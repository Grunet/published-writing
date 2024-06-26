# Making a Totally Free Uptime Monitor using a Worker Runtime and OpenTelemetry

## Table of Contents

- [What is an Uptime Monitor and When to Use One?](#what-is-an-uptime-monitor-and-when-to-use-one)
- [Traditional Options](#traditional-options)
- [Using a Worker Runtime and OpenTelemetry](#using-a-worker-runtime-and-opentelemetry)
   * [The High-Level Solution](#the-highlevel-solution)
   * [The High-Level Setup Steps](#the-highlevel-setup-steps)
   * [Comparison to the Other Options](#comparison-to-the-other-options)
- [Takeaway](#takeaway)

## What is an Uptime Monitor and When to Use One?

An uptime monitor is a tool that periodically (e.g. every minute) checks your application or API to gauge if it’s up and healthy.

If you have true observability and are using SLOs effectively you probably don’t need to use one. But if you’re not at that level yet, an uptime monitor can be a valuable information source regarding the reliability of your application or API.

## Traditional Options

There are a number of ways to run an uptime monitor. For example,

- Running a cron job on a server/VM and using bash, curl, and webhooks
- Setting up an Eventbridge cron with Container/Lambda targets and webhooks
- Paying for a 3rd party service (e.g. Pingdom)

Each of them comes with their own downsides though

- Maintenance (e.g. security patching, keeping away from end-of-life states)
- Complexity (e.g. setting up IaC, CI/CD)
- Cost

Is there an option that avoids these downsides?

## Using a Worker Runtime and OpenTelemetry

I contend there is using a [worker runtime](https://workers.js.org/) and [OpenTelemetry](https://opentelemetry.io/).  

### The High-Level Solution

The solution maps out at a high-level as follows

1. Use a cron from a worker runtime
2. Have the worker hit the application or API endpoint
3. Gather instrumentation about the network call with OpenTelemetry
4. Send that OpenTelemetry instrumentation to an observability backend
5. Use the observability backend to alert on unhealthy traffic 

### The High-Level Setup Steps

These steps will use Cloudflare Workers for the worker runtime, but something similar can be done with Deno Deploy as well.

1. [Create a free Cloudflare account](https://dash.cloudflare.com/sign-up/workers-and-pages)
2. [Create a worker](https://developers.cloudflare.com/workers/get-started/guide/) with the following code and the [Node.js compatibility flag](https://developers.cloudflare.com/workers/runtime-apis/nodejs/#enable-nodejs-from-the-cloudflare-dashboard)

    ```js
    import { instrument } from '@microlabs/otel-cf-workers'

    const handler = { 
	    async scheduled(event, env, ctx) {
		    await fetch(env.ENDPOINT_TO_MONITOR)
	    }
    }

    const config = (env, _trigger) => {
	    return {
		    exporter: {
			    url: 'https://api.honeycomb.io/v1/traces',
			    headers: { 'x-honeycomb-team': env.HONEYCOMB_API_KEY },
		    },
		    service: { name: env.ENDPOINT_NAME },
	    }
    }

    export default instrument(handler, config)
    ```

3. [Add an environment variable](https://developers.cloudflare.com/workers/configuration/environment-variables/#add-environment-variables-via-the-dashboard) named “ENDPOINT_TO_MONITOR” with the endpoint to check and add another environment variable named “ENDPOINT_NAME” with a friendly name for the endpoint
4. [Create a free Honeycomb account](https://www.notion.so/Making-a-Totally-Free-Uptime-Monitor-using-a-Worker-Runtime-and-OpenTelemetry-0a4636936b3c40f38dd8c4a474145aec?pvs=21)
5. Create an environment named “Uptime Monitors” and [create an ingest key](https://docs.honeycomb.io/get-started/configure/environments/manage-api-keys/)
6. Back in Cloudflare, take that ingest key and copy-paste it into a [Cloudflare Workers secret](https://developers.cloudflare.com/workers/configuration/secrets/#via-the-dashboard) named “HONEYCOMB_API_KEY”
7. [Add a cron](https://developers.cloudflare.com/workers/configuration/cron-triggers/#via-the-dashboard) of “* * * * *” to the worker
8. (Confirm that traces are appearing every minute in Honeycomb)
9. In Honeycomb, [create a trigger](https://docs.honeycomb.io/notify/alert/triggers/create/) (alert) based on the query

    ```jsx
    COUNT > 0 where http.response.status_code >= 400 
    ```

10. Route the trigger’s notifications as needed (e.g. to Slack)

You should now have a functioning uptime monitor for your endpoint.

### Comparison to the Other Options

Compared to the other options outlined before, this solution has

- Minimal maintenance (just a single npm package and its dependencies to monitor for security vulnerabilities)
- Minimal complexity (just the steps outlined above)
- Totally free (the usage is very much within the [Cloudflare Workers free tier](https://developers.cloudflare.com/workers/platform/pricing/#workers) and [Honeycomb free tier](https://www.honeycomb.io/pricing))

## Takeaway

Paying for an uptime monitor service is probably preferable to this (if you’re able to).

The real takeaway is that there is this newer form of compute (worker runtimes) with a cost model that can be taken advantage of for situations similar to this.