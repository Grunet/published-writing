# Leveraging OpenTelemetry in Deno

## Table of Contents

- [Background](#background)
- [Goal](#goal)
- [A Minimal Interesting Example](#a-minimal-interesting-example)
  * [The Boilerplate](#the-boilerplate)
      + [Autoinstrumentation](#autoinstrumentation)
      + [Span Processing and Exporting](#span-processing-and-exporting)
          - [Exporting to the Console and to a Tracing Vendor ](#exporting-to-the-console-and-to-a-tracing-vendor)
  * [The App Code](#the-app-code)
      + [OpenTelemetry’s API Surface](#opentelemetrys-api-surface)
      + [Handling Concurrent Requests and the Need for Async Context](#handling-concurrent-requests-and-the-need-for-async-context)
- [Future Directions](#future-directions)
  * [Adding Structured Data to Spans](#adding-structured-data-to-spans)
  * [To Deploy and Beyond](#to-deploy-and-beyond)
  * [Actually Knowing CPU Time per Request](#actually-knowing-cpu-time-per-request)
  * [Autoinstrumenting Deno-specific APIs and Libraries](#autoinstrumenting-deno-specific-apis-and-libraries)
  * [Putting the “Distributed” in Distributed Tracing](#putting-the-distributed-in-distributed-tracing)
  * [All the Pillars](#all-the-pillars)
- [Parting Thoughts](#parting-thoughts)
- [References](#references)
- [Console Output](#console-output)

## Background

Deno is a runtime for JavaScript, TypeScript, and WebAssembly that is based on the V8 JavaScript engine and the Rust programming language.

OpenTelemetry (OTEL) is a collection of tools, APIs, and SDKs. It's used to instrument, generate, collect, and export telemetry data (metrics, logs, and traces) to help you analyze your software’s performance and behavior.

Until relatively recently, it wasn’t possible to bring the power of OpenTelemetry to bear on Deno. All of us were missing out on the information OpenTelemetry can gather, in particular for tracing.

## Goal

This article will go over 1 simplified example in depth of using OpenTelemetry for tracing in Deno. 

The aim is primarily to serve as an introduction to OpenTelemetry concepts for folks already somewhat familiar with Deno.

## A Minimal Interesting Example

Here is a visualization of 1 trace emitted by the example code (taken from Honeycomb, an observability vendor).

![3 rows, with 1 horizontal bar in each of varying lengths. The top bar is the longest and it says 16.34ms inside the bar. The 2nd bar says 12.00 ms inside the bar, and it starts a little after the top bar and ends a little after it. The 3rd bar says 0.1948ms and is very short, starting and ending just before the end of the top bar. To the left of each row there's a name given to each bar. The top bar is named handler. The second bar is named HTTP GET. The third bar is named construct body. There is a tree structure that indicates that the HTTP GET row and the contruct body row are both children of the handler row.](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/loahutlkxrpum3p6xjn4.png)

> The "HTTP GET" span ending after its parent is due to [a small bug in otel-js](https://github.com/open-telemetry/opentelemetry-js/issues/3719). For now, just pretend it ended before its parent did.

Note the following things you can tell without even looking at the code

- Different parts of the code have their duration measured
- Outgoing HTTP requests are captured
- The structure of the code is probably reflected in the structure of the diagram

And now here is the code that was used to generate that telemetry.

```tsx
import { registerInstrumentations } from "npm:@opentelemetry/instrumentation";
import { FetchInstrumentation } from 'npm:@opentelemetry/instrumentation-fetch';

import { NodeTracerProvider } from "npm:@opentelemetry/sdk-trace-node";
import { Resource } from "npm:@opentelemetry/resources";
import { SemanticResourceAttributes } from "npm:@opentelemetry/semantic-conventions";
import { BatchSpanProcessor, ConsoleSpanExporter } from "npm:@opentelemetry/sdk-trace-base";
import { OTLPTraceExporter } from "npm:@opentelemetry/exporter-trace-otlp-proto";

import opentelemetry from "npm:@opentelemetry/api";
import { serve } from "https://deno.land/std@0.180.0/http/server.ts";

// autoinstrumentation.ts

registerInstrumentations({
  instrumentations: [new FetchInstrumentation()],
});

// Monkeypatching to get past FetchInstrumentation's dependence on sdk-trace-web, which has runtime dependencies on some browser-only constructs. See https://github.com/open-telemetry/opentelemetry-js/issues/3413#issuecomment-1496834689 for more details
// Specifically for this line - https://github.com/open-telemetry/opentelemetry-js/blob/main/packages/opentelemetry-sdk-trace-web/src/utils.ts#L310
globalThis.location = {}; 

// tracing.ts

const resource =
  Resource.default().merge(
    new Resource({
      [SemanticResourceAttributes.SERVICE_NAME]: "deno-demo-service",
      [SemanticResourceAttributes.SERVICE_VERSION]: "0.1.0",
    })
  );

const provider = new NodeTracerProvider({
    resource: resource,
});

const consoleExporter = new ConsoleSpanExporter();
provider.addSpanProcessor(new BatchSpanProcessor(consoleExporter));

const traceExporter = new OTLPTraceExporter();
provider.addSpanProcessor(new BatchSpanProcessor(traceExporter));

provider.register();

// Application code

const tracer = opentelemetry.trace.getTracer(
  'deno-demo-tracer'
);

const port = 8080;

const handler = async (request: Request): Promise<Response> => {
  // This call will be autoinstrumented
  await fetch("http://www.example.com");

  const span = tracer.startSpan(`constructBody`);
  const body = `Your user-agent is:\n\n${request.headers.get("user-agent") ?? "Unknown"}`;
  span.end();

  return new Response(body, { status: 200 });
};

await serve(instrument(handler), { port });

// Helper code

function instrument(handler) {

  async function instrumentedHandler(request) {
    let response;
    await tracer.startActiveSpan('handler', async (span) => {

      response = await handler(request);

      span.end();
    });

    return response;
  }

  return instrumentedHandler;
}
```

It’s a lot! But let’s take a look at it piece-by-piece.

### The Boilerplate

#### Autoinstrumentation

This is the code for the fetch autoinstrumentation

```tsx
import { registerInstrumentations } from "npm:@opentelemetry/instrumentation";
import { FetchInstrumentation } from 'npm:@opentelemetry/instrumentation-fetch';

...

registerInstrumentations({
  instrumentations: [new FetchInstrumentation()],
});
```

This monkeypatches the global `fetch` so that all network calls made with it are instrumented (i.e. have data on them recorded in telemetry).

It saves you the trouble of needing to instrument every `fetch` call in application code yourself. And it also instruments `fetch` calls your dependencies are making, which may have been otherwise impossible to track.

#### Span Processing and Exporting

This is the code for setting up the span processing and exporting pipelines.

```tsx
import { BatchSpanProcessor, ConsoleSpanExporter } from "npm:@opentelemetry/sdk-trace-base";
import { OTLPTraceExporter } from "npm:@opentelemetry/exporter-trace-otlp-proto";

...

const consoleExporter = new ConsoleSpanExporter();
provider.addSpanProcessor(new BatchSpanProcessor(consoleExporter));

const traceExporter = new OTLPTraceExporter();
provider.addSpanProcessor(new BatchSpanProcessor(traceExporter));
```

Once a span has ended, it will be sent to a span processor, which will decide what to do with it and eventually pass it on to an exporter.

In this case `BatchSpanProcessor` is being used, meaning that spans are queued up in-memory and [flushed in batches](https://github.com/open-telemetry/opentelemetry-js/blob/main/packages/opentelemetry-sdk-trace-base/src/export/BatchSpanProcessorBase.ts#L218) via a `setTimeout` [every 5 seconds](https://github.com/open-telemetry/opentelemetry-js/blob/main/packages/opentelemetry-core/src/utils/environment.ts#L153).

##### Exporting to the Console and to a Tracing Vendor 

The first “endpoint” spans are exported to is the console, via the `ConsoleSpanExporter`. This is useful to have for debugging purposes, especially when you’re not seeing traces show up in your vendor but you are seeing them in the console.

The second endpoint spans are exported to is your tracing vendor (e.g. Honeycomb, NewRelic, etc...), via the `OTLPTraceExporter`. It will depend on the vendor, but specifying your vendor’s remote endpoint and auth credentials as environment variables should be enough

```shell
export OTEL_EXPORTER_OTLP_ENDPOINT=<your vendor HTTP OTLP ingest endpoint>
export OTEL_EXPORTER_OTLP_HEADERS=<vendor specific auth credentials>
```

This should enable the exporter code to send telemetry via OTLP (OpenTelemetry Line Protocol) over HTTP to your tracing vendor.

### The App Code

#### OpenTelemetry’s API Surface

Everything so far has been part of the OpenTelemetry SDK, and should not be touched directly by application code.

Application code should only ever have to interact with the OpenTelemetry API. The API then hooks up to the SDK behind the scenes to do all of the things previously discussed.

To create spans and other instrumentation, application code should use the OpenTelemetry API, as shown below.

```tsx
import opentelemetry from "npm:@opentelemetry/api";

...

const tracer = opentelemetry.trace.getTracer(
  'deno-demo-tracer'
);

...

const span = tracer.startSpan(`constructBody`);
const body = `Your user-agent is:\n\n${request.headers.get("user-agent") ?? "Unknown"}`;
span.end();

...

async function instrumentedHandler(request) {
    let response;
    await tracer.startActiveSpan('handler', async (span) => {

      response = await handler(request);

      span.end();
    });

    return response;
  }
```

A key, subtle difference here is between `startSpan` and `startActiveSpan.`

- `startSpan`
    - Creates a span, finds the currently “active” span, and adds the newly created span as a child of it
- `startActiveSpan`
    - Creates a span, finds the currently “active” span, and adds the newly created span as a child of it.
    - Makes the new span the “active” span, so all spans created in the function passed in the 2nd parameter will be added as child spans of it

The difference is that `startActiveSpan` makes a new parent by default, whereas `startSpan` does not.

In order to have a span created by `startSpan` become “active” and the parent of any child spans, you have to do this

```tsx
const ctx = opentelemetry.trace.setSpan(
  opentelemetry.context.active(),
  wantsToBeAParentSpan
);
```

Then calling `startSpan` afterwards will create the new spans as children of `wantsToBeAParentSpan`.

#### Handling Concurrent Requests and the Need for Async Context

Imagine 2 requests (say request A and request B) come in near simultaneously. Both create their own parent spans (say parent span A and parent span B) and both end up waiting on the asynchronous fetch call to resolve at the same time.

Say request A’s fetch call finishes first. How does the `tracer.startSpan` call know to attach itself to parent span A and not parent span B?

```tsx
const handler = async (request: Request): Promise<Response> => {

  await fetch("http://www.example.com");

  const span = tracer.startSpan(`constructBody`);
  const body = `Your user-agent is:\n\n${request.headers.get("user-agent") ?? "Unknown"}`;
  span.end();

  return new Response(body, { status: 200 });
};
```

We need some way to keep the context of the request across asynchronous events, so that `tracer.startSpan` can know that this is still for request A, and it should make the new span a child of parent span A.

OpenTelemetry-JS handles this differently based on the situation

- Browser - Uses zone.js to keep track of these asynchronous contexts
- Node.js - Uses AsyncLocalStorage to keep track of asynchronous contexts

Thanks to Deno’s Node.js compatibility efforts all that’s needed to benefit from the Node.js async context management approach is to use the Node SDK

```tsx
import { NodeTracerProvider } from "npm:@opentelemetry/sdk-trace-node";

...

const provider = new NodeTracerProvider({
    resource: resource,
});
```
This will automatically make sure `tracer.startSpan` attaches the span it creates to the correct parent span in the above situation.

## Future Directions

This example is just scratching the surface of using OpenTelemetry in Deno. Here are some other ways to take it moving forward.

### Adding Structured Data to Spans

One thing the example didn't highlight is the ability to add structured data to spans, just like you would with logs ([this OTEL doc](https://opentelemetry.io/docs/instrumentation/js/instrumentation/#attributes) covers how to do this in more detail).

In certain use cases, you could potentially get away with only using traces and not using logs at all.

### To Deploy and Beyond

As of this writing, Deno Deploy doesn’t support `npm:` specifiers so it’s not possible to use the OTEL example above there.

But once that lands it should be good to go (!)

### Actually Knowing CPU Time per Request

Deno Deploy (and other edge function services) limit or bill based on the milliseconds of CPU time your functions use.

However, there’s no easy way to profile or measure this today (as far as I’m aware).

With OpenTelemetry traces, you should be able to drop in spans in CPU-intensive parts of your code to zone in on what’s eating up CPU time.

And with autoinstrumentation, you could even measure the underlying framework you’re using too.

### Autoinstrumenting Deno-specific APIs and Libraries

There are many [Deno APIs](https://deno.land/api@v1.31.1) outside of `fetch` that could probably benefit from being autoinstrumented (e.g. Cache), in particular the ones that don’t already have a browser autoinstrumentation package available (i.e. for Deno-specific APIs, like filesystem ones).

There are also a wide variety of Deno-specific libraries (e.g. oak, Fresh) that could possibly benefit from being autoinstrumented too, either via a separate autoinstrumentation library or by getting built-in to the library itself.

### Putting the “Distributed” in Distributed Tracing

This example was only of 1 service, but you could imagine running several different distributed services as well.

In that case, trace context propagation across the network (where Service B is aware of the parent span Service A created before sending Service B the request, so Service B can add its spans to the correct parent) is critical in creating 1 unbroken distributed trace across all services.

I haven't yet tried to see if this just works out-of-the-box, but I’m guessing it may need some effort to get firing on all cylinders.

### All the Pillars

This example was just of tracing, but OpenTelemetry also covers metrics and logging as well (enabling cool things like finding trace exemplars that are contributing to a metric).

It would be interesting to see if those work out-of-the-box for Deno too.

## Parting Thoughts

I personally didn't anticipate that the Node.js compat work done by the Deno team would impact OpenTelemetry support, so this came as a pleasant surprise to me. I have no idea what that involved but hats off to the Deno folks for their efforts.

And I hope there is more to come in making Deno the best server-side JS runtime around for production workloads!

## References

1. https://en.wikipedia.org/wiki/Deno_(software) is where I got the definition of Deno from
2. https://opentelemetry.io/ is where I got the definition of OpenTelemetry from
3. https://opentelemetry.io/docs/instrumentation/js/instrumentation/ and the remaining OTEL-JS docs have a lot of information on what more you can do with spans and how to go about doing it
4. For more tips on troubleshooting beyond ConsoleSpanExporter, including turning on OpenTelemetry-JS's diagnostic Debug-level logging, see https://opentelemetry.io/docs/instrumentation/js/getting-started/nodejs/#troubleshooting
5. [As of this writing, Deno supports Node.js's AsyncLocalStorage API for async context management except for setTimeout](https://github.com/open-telemetry/opentelemetry-js/issues/2293#issuecomment-1485395549) per Luca Casonato
6. [This older article I wrote collects some notes on the history of async context management in JS](https://dev.to/grunet/can-you-measure-the-duration-of-a-promise-3a6h)

## Console Output

For the curious, here is the output that the `ConsoleSpanExporter` generates after the server handles a request

```tsx
{
  traceId: "9a9a625e1562e9847ffe97e09fcf1bea",
  parentId: "f80a96f14dc334d8",
  traceState: undefined,
  name: "constructBody",
  id: "5a1974974e18fcf6",
  kind: 0,
  timestamp: 1680829418679000,
  duration: 195,
  attributes: {},
  status: { code: 0 },
  events: [],
  links: []
}
{
  traceId: "9a9a625e1562e9847ffe97e09fcf1bea",
  parentId: undefined,
  traceState: undefined,
  name: "handler",
  id: "f80a96f14dc334d8",
  kind: 0,
  timestamp: 1680829418663000,
  duration: 16340,
  attributes: {},
  status: { code: 0 },
  events: [],
  links: []
}
{
  traceId: "9a9a625e1562e9847ffe97e09fcf1bea",
  parentId: "f80a96f14dc334d8",
  traceState: undefined,
  name: "HTTP GET",
  id: "7ec3993d6f40671c",
  kind: 2,
  timestamp: 1680829418669000,
  duration: 12000,
  attributes: {
    component: "fetch",
    "http.method": "GET",
    "http.url": "http://www.example.com/",
    "http.status_code": 200,
    "http.status_text": "OK",
    "http.host": "www.example.com",
    "http.scheme": "http",
    "http.user_agent": "Deno/1.32.1"
  },
  status: { code: 0 },
  events: [],
  links: []
}
{
  traceId: "5086d5f4412c11a533d81044998ac7d6",
  parentId: undefined,
  traceState: undefined,
  name: "HTTP POST",
  id: "698b36fbc35e09c9",
  kind: 2,
  timestamp: 1680829423694000,
  duration: 94000,
  attributes: {
    component: "fetch",
    "http.method": "POST",
    "http.url": "https://api.honeycomb.io/v1/traces",
    "http.status_code": 200,
    "http.status_text": "OK",
    "http.host": "api.honeycomb.io",
    "http.scheme": "https",
    "http.user_agent": "Deno/1.32.1"
  },
  status: { code: 0 },
  events: [],
  links: []
}
```