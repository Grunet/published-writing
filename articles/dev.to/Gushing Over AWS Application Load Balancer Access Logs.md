## Table of Contents

- [What They Are](#what-they-are)
- [Why They Are Great](#why-they-are-great)
   * [Chock Full of Details](#chock-full-of-details)
   * [Close to End-User Behavior and Pain](#close-to-end-user-behavior-and-pain)
   * [Non-Invasive to Enable](#non-invasive-to-enable)
   * [Supported by Vendors](#supported-by-vendors)
- [Some Frustrations](#some-frustrations)
   * [Batching](#batching)
   * [Poor Integrations](#poor-integrations)
- [Takeaway](#takeaway)

## What They Are

Auto-generated logs that capture details on each request that passes through an Application Load Balancer (ALB) and onward to a backend target. ([Here is their main AWS doc page](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-access-logs.html))

## Why They Are Great

There are multiple reasons to get excited about these logs.

![An anime character with mouth open wide and eyes widened in fascination of something. The character is Boji from Ranking of Kings.](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/la2pg6giyus13kozvzj1.jpg)

### Chock Full of Details

ALB access logs include a huge amount of information on each request, for example

- Request Method
- Request Path (including query parameters)
- Client Ip Address
- User Agent
- Request Start Time
- Request End Time
- Request Duration
- Load Balancer Status Code
- Target Status Code
- Target Internal Ip Address

And that’s just to mention a few! ([Here is the full list of attributes](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-access-logs.html#access-log-entry-syntax))

With just these attributes you can do things like

- Look for requests coming from a particular ip address
- Look for requests blocked by the Web Application Firewall (403s for the load balancer status code with no target status code)
- Look for a service that temporarily went down (502s for the load balancer status code with no target status code)

### Close to End-User Behavior and Pain

Access logs from a public-facing ALB give the closest representation of what end-users are doing with your application (client-side instrumentation aside) since they record every single request (no sampling).

They also give the best representation of the pain your end-users are facing (e.g. looking at 5xx’s the load balancer is returning).

Backend instrumentation alone will always be missing part of the picture (e.g. when a backend service is hard down, sampling).

### Non-Invasive to Enable

No application code changes or instrumentation with 3rd party agents/libraries are required to turn on ALB access logs.

Just [enable them](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/enable-access-logging.html) like you configure the other parts of your infrastructure via IaC, the CLI, or ClickOps. Then watch the log files start to show up in S3.

### Supported by Vendors

Most observability vendors will offer a solution for ingesting ALB access log files into their platform (e.g. offering a lambda that can trigger off the log files’ S3 bucket new object created event) for querying there.

Alternatively, Athena can be used to query them from within AWS.

## Some Frustrations

To be honest these logs aren't perfect and have a few notable downsides.

![A slightly chubby yellow creature that looks like a duck without a neck standing upright on its back flippers. It's holding its front flippers to the sides of its head. It's Psyduck the pokemon having a headache.](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2jzvs04m7utffe9m8foy.png)

### Batching

An ALB will batch all the access logs it generates and send it to the S3 bucket containing them once every 5 minutes. This means that it’s not suitable for situations where human response times to changes need to be faster than that. This is one area where backend instrumentation wins out.

(I believe GCP’s analogs don’t have this limitation fyi.)

### Poor Integrations

Web Application Firewalls (WAF) can be configured to record their own logs of every request sent to a load balancer configured to use a firewall, but as far as I know there’s no way to integrate them with ALB access logs. They are a totally separate source of info.

Also ALB access logs can’t take part in OpenTelemetry tracing as far as I know (though it would be pretty cool if they did)

## Takeaway

If you’re stranded on an island and you can only have 1 piece of telemetry, I’d recommend picking ALB access logs. They have their limitations, but they deliver the most value for the least investment in my opinion.