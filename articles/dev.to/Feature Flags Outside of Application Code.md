# Feature Flags Outside of Application Code

## Table of Contents

1. [What are Feature Flags and How Can They be Helpful?](#what-are-feature-flags-and-how-can-they-be-helpful)
2. [Can I Use Them for Configuration Too?](#can-i-use-them-for-configuration-too)
    - [Updating a npm Dependency](#updating-a-npm-dependency)
        1. [Why Update Dependencies](#why-update-dependencies)
        2. [The Traditional Approach to Updating a Dependency](#the-traditional-approach-to-updating-a-dependency)
            1. [Pre-Production Work ](#preproduction-work)
            2. [Production Work](#production-work)
                (choose [Happy Path](#happy-path) or [Sad Path](#sad-path))
        3. [The Critical Problem with the Traditional Approach to Updating a Dependency](#the-critical-problem-with-the-traditional-approach-to-updating-a-dependency)
        4. [The Inability to Use Feature Flags when Updating a Dependency](#the-inability-to-use-feature-flags-when-updating-a-dependency)
    - [Responding to an Alert while On-Call](#responding-to-an-alert-while-oncall)
        1. [Some Background on Horizontal Auto Scaling](#some-background-on-horizontal-auto-scaling)
        2. [Where Simple Horizontal Auto Scaling Fails](#where-simple-horizontal-auto-scaling-fails)
        3. [Using an On-Call Human to Make Up for the Horizontal Auto Scaling Limitation](#using-an-oncall-human-to-make-up-for-the-horizontal-auto-scaling-limitation)
        4. [The Critical Problem with Using an On-Call Human to Manually Change Infrastructure Quickly](#the-critical-problem-with-using-an-oncall-human-to-manually-change-infrastructure-quickly)
        5. [The Inability to Use Feature Flags when Manually Changing Infrastructure Quickly](#the-inability-to-use-feature-flags-when-manually-changing-infrastructure-quickly)
3. [What if I Turn my Configuration into Code?](#what-if-i-turn-my-configuration-into-code)
    - [Using Feature Flags for Dependency Upgrades with In-Code Dependency Imports](#using-feature-flags-for-dependency-upgrades-with-incode-dependency-imports)
    - [Using Feature Flags with CDK and CDK-TF](#using-feature-flags-with-cdk-and-cdktf)
4. [Why is this so Frustrating?](#why-is-this-so-frustrating)
5. [Will this be Less Frustrating in the Future?](#will-this-be-less-frustrating-in-the-future)

## What are Feature Flags and How Can They be Helpful?

A feature flag is a context-aware configuration point.

Depending on the context and the feature flag’s definition, it can return different values (e.g. return 1 if the user’s name is Bob and CPU usage is below 90%, otherwise return 0). An ordinary configuration point (e.g. an environment variable) returns a fixed value regardless of the context.

Some feature flag vendors allow for near-instantaneous updates to feature flag definitions being used in production code. This unlocks many interesting operational possibilities, including 

1. Only letting a small fraction of users see a code change at first, and also being able to immediately turn off the change if it proves problematic
2. Temporarily disabling one part of your application under attack so another more critical part relying on a fragile shared dependency can continue to operate without interruption

(If this doesn’t match your definition of what a “feature flag” is, check out [https://docs.openfeature.dev/docs/reference/intro](https://docs.openfeature.dev/docs/reference/intro) for a disambiguation.)

## Can I Use Them for Configuration Too?

By default, no typically.

These two examples illustrate why it’s not typically possible, as well as why it would be extremely helpful were it possible.

- Updating a npm Dependency
- Responding to an Alert while On-Call

### Updating a npm Dependency

#### Why Update Dependencies

To react faster to security vulnerabilities in a product’s dependencies, those dependencies need to be kept appropriately up-to-date (e.g. to avoid being on an older version no longer receiving security support patches)

For a Node.js app, this includes keeping 3rd party npm dependencies updated.

But bumping a dependency by one or more major versions can also be dangerous, as the dependency may have introduced breaking changes affecting the app’s usage of it.

#### The Traditional Approach to Updating a Dependency

##### Pre-Production Work 

Static analysis tools (e.g. Typescript) and pre-production tests that exercise the app’s usage of the dependency can help catch post-upgrade regressions early on. 

Scrutinizing the release notes and/or changelog can also help build confidence in the upgrade.

However, often this isn’t enough to build a high-level of confidence in an upgrade.

An example is where the dependency’s usage is scattered around a large codebase in a huge number of places (as opposed to being behind a central adapter). It’s typically not possible to expect fully exhaustive usage coverage through the dependency in this case.

##### Production Work

###### Happy Path

Nevertheless, once diminishing returns in confidence have been hit in pre-production work, the upgrade needs to be deployed to production.

For a Node.js app deployed in ordinary ways (e.g. rolling on Fargate on AWS Elastic Container Service behind an AWS Application Load Balancer) this will end up meaning that the upgrade will go from affecting 0% of active users before the deploy to 100% of active users after the deploy.

If no issues appear to have occurred for a while after the deploy (either according to production telemetry or user reports) the upgrade is completed successfully.

###### Sad Path

However if an issue does arise in production, then up to 100% of active users could be affected by it at once. 

Once the issue is detected (from telemetry or user reports) the only option is to revert the upgrade commit and re-deploy the app.

This may take upwards of 10s of minutes depending on how quickly your CI/CD pipeline runs. In the meantime more and more of your active users will encounter the issue.

#### The Critical Problem with the Traditional Approach to Updating a Dependency

In the sad path above, the critical problem is not that an issue happened in production (assuming the product is not safety critical, like a component of an airplane).

The critical problem is that after the issue was detected, the issue continued to impact users for a long period of time.

Ideally, as soon as the issue was detected, it should have been instantaneously fixed.

#### The Inability to Use Feature Flags when Updating a Dependency

Being able to wrap the upgrade behind a feature flag would allow for the change to be instantaneously shut off in production as soon as any issue was detected.

However, this is impossible due to how the code change for a dependency upgrade in a Node.js app works. Rather than the change happening in a Javascript file where most of the app’s source code lives, it happens in a separate package-lock.json JSON file. 

The changes to the package-lock.json file are autogenerated, and even if they weren’t, Node.js feature flagging libraries (e.g. [OpenFeature](https://docs.openfeature.dev/docs/reference/technologies/server/javascript)) can’t be used inside of a JSON file (they only work inside of Javascript files).

This sucks, it should be as easy to turn off problematic dependency upgrades as it is to turn off problematic code changes.


![In an anime, a black haired girl with purple eyes sits with arms crossed across her knees having just raised her head from them. Tears stream down from her eyes, and her face beneath her eyes is slightly red. She looks at her classmate who has reddish brown hair and asks "Aren't you upset?" The emotional context is that their middle school band just lost in regionals and they won't be going to nationals, but the reddish brown haired girl isn't that upset about it.](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/q84m95ot4vu87m9ewn6t.png)

### Responding to an Alert while On-Call

#### Some Background on Horizontal Auto Scaling

Some products receive expected but unpredictable changes in their usage (e.g. after a marketing campaign suddenly gains traction many new users start to use the product, but only a few end up becoming recurring users afterwards).

When a spike in traffic like this occurs, an app can become overwhelmed and start to break down.

One technique for dealing with this is to automatically horizontally scale the app. This creates additional identical copies of the app and then distributes the traffic among them, reducing the amount of traffic sent to any given copy (e.g. using AWS Application Auto Scaling with a containerized app built on AWS Elastic Container Service behind an AWS Application Load Balancer).

The simplest auto scaling logic works by continuously watching 1 metric produced by the system that indicates how overwhelmed it is. 

When that metric exceeds some chosen threshold, auto scaling kicks in and tries to create more copies of the app. The hope is that will cause the metric to go back down below the threshold, implying the system is no longer overwhelmed and doesn’t need any more intervention. 

Picking a metric like “average number of requests each copy of the app is currently handling” will likely help deal with spikes in usage leading to spikes in requests. However, this comes with the tradeoff that auto scaling will do nothing at all in other problematic situations.

#### Where Simple Horizontal Auto Scaling Fails

Imagine a legacy Node.js app where it’s known that certain rare single requests (e.g. “make me a report of all the things”) from certain large customers will monopolize the single-threaded Javascript event loop with long unbroken CPU work (e.g. 20+ seconds of it) interrupted by some network I/O to a database, and with that pattern repeating for an extremely long time (e.g. 20+ minutes). 

When a request like that is being handled by a copy of the app, the vast majority of requests from other customers to that copy of the app will get very, very slow (since they have to wait for those unbroken CPU intervals to finish before they can start to be processed).

That isn’t good, but what’s worse is that it’s fairly easy for that large customer to accidentally “lock up” your entire system just by re-sending that same request a few more times to other copies of the app. Then any request from any customer may start to see the massive slowdowns too. (I swear this is a fictional story)

What is auto scaling based on the “average number of requests each copy of the app is currently handling” metric doing during this? Not much. It only cares when a sudden large spike in requests occurs. With only a handful of requests in flight, it just stands by and watches as things burn.

![The "This is fine" popular internet meme. A two panel comic. The first panel shows an anthropomorphized dog sitting on a chair next to a table with a coffee mug on it. The room the dog is in is covered in flames and smoke covers the ceiling. The dog's eyes do not seem to betray any fear of the situation. In the second panel we zoom in on the dog's face where the dog says this is fine in a speech bubble that appears above their face. ](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bqsy0rh0iv3xzb5m7gqp.jpg)

#### Using an On-Call Human to Make Up for the Horizontal Auto Scaling Limitation

Rather than attempt to fix the performance issue plaguing the legacy Node.js app, an organization may decide instead to have their on-call human be alerted when the bad situation starts. That way they can intervene and manually trigger the horizontal scaling.

That could play out as follows

1. On-call human receives a high CPU usage alert for the app and just a little later a p95 latency alert for the app, and opens up the runbook for each
2. The runbook for both says to check what a certain dashboard looks like to see if the cause is probably the known bad situation (described above)
3. It’s a match, and the runbooks direct the on-call human to quickly make a pull request (PR) in a certain repository to highly increase the default number of copies of the app (i.e. manually horizontally scaling out)
4. The on-call human makes the PR, and then tracks down another human to approve the PR 
5. The PR is merged, and the CI/CD system applies the Infrastructure as Code changes, highly increasing the default number of copies of the app
6. The on-call human looks back at the dashboard to confirm the horizontal scaling did occur and that the latency metrics seemed to have improved. They drill in to confirm via tracing that the requests with better latencies are the ones segregating through the newer copies of the app.

#### The Critical Problem with Using an On-Call Human to Manually Change Infrastructure Quickly

In the scenario above, the main problem is not that the organization decided not to address the underlying performance issue, nor in that the response was not fully automated. Both of these are reasonable decisions in this day and age.

The main problem is that once the on-call human knew what needed to be done, the change wasn’t made instantaneously.

Depending on how long some of the steps took, it could have been several minutes between them knowing what needing to happen and the IaC tool making the relevant API calls to their cloud provider.

During those several minutes, customers would have still been experiencing pain they didn’t need to experience.

#### The Inability to Use Feature Flags when Manually Changing Infrastructure Quickly

Being able to keep a permanent feature flag that held the value for the default number of copies of the app likely could have erased those several minutes of customer pain. 

The on-call human would have only needed to jump to a link to the flag’s web UI from the runbook, change the number, and hit Save. This would then immediately make the required cloud provider API calls.

Now imagine more complex scenarios where multiple IaC files needed adjusting, or the change wasn’t as simple as just changing a number (e.g. for toggling a maintenance page for a load balancer, or for quickly toggling expensive infrastructure-emitted logs on/off). Creating PRs for these in a high pressure situation seems even more unappetizing.

With declarative IaC tools like CloudFormation or Terraform, there’s no straightforward way to make this work as far as I’m aware. There just isn’t a way to use feature flag libraries in YAML, JSON, or HCL files. They’re fully incompatible.

This sucks, it should be possible to adjust infrastructure as fast as it’s possible to adjust application code flows.

![In an anime, a black haired girl with shoulder length hair and purple eyes is standing up. Her face is slightly red beneath her eyes. She is wearing a greyish button downed vest on top of a white undergarment with a ribbon tied at the neck, and a grey skirt. Her arms are straight by her side and her fists are balled up. Her eyes stare intently at her classmate with reddish brown her as she says I'm upset. The emotional context is that their middle school band just lost in regionals and they won't be going to nationals, but the reddish brown haired girl isn't that upset about it.](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/k2dvgiuucq5we9qynjyf.png)

## What if I Turn my Configuration into Code?

In some cases, declarative configuration can be replaced by code that executes in a runtime. In these cases, feature flags can be used, now that there’s a runtime they can be evaluated in.

The two above examples can actually be converted in this way, with some caveats

- Using Feature Flags for Dependency Upgrades with In-Code Dependency Imports
- Using Feature Flags for Changing Infrastructure with CDK and CDK-TF

### Using Feature Flags for Dependency Upgrades with In-Code Dependency Imports

In certain Javascript runtimes (including all versions of Deno, and experimental support in Node.js 18), dependencies don’t have to be declared in separate package.json and package-lock.json files. Instead, they can be included directly in source code files, as in this Deno example

```jsx
import axios from "npm:axios@0.27.2";
```

This means that when upgrading a dependency, both the old and the new versions can actually be imported at the same time

```jsx
import axiosOld from "npm:axios@0.27.2";
import axiosNew from "npm:axios@1.0.0";
```

Then as long as the dependency is centralized behind an adapter or dependency injection container, the upgrade can be easily put behind a feature flag.

```jsx
import axiosOld from "npm:axios@0.27.2";
import axiosNew from "npm:axios@1.0.0";

async function getAxios() {
	let axios;
	if (await featureFlagClient.getBooleanValue('Axios_v1_Upgrade', false)) {
		axios = axiosNew;
	} else {
		axios = axiosOld;
	}

	return axios;
}

export { getAxios }
```

### Using Feature Flags with CDK and CDK-TF

Cloud Development Kit (CDK) and Cloud Development Kit for Terraform (CDK-TF) are a set of libraries for various programming languages and runtimes (e.g. Typescript with Node.js, Python) that let you write ordinary code to generate CloudFormation and Terraform files, respectively (in actuality they go beyond this, but that’s all that’s relevant for this topic)

Since it’s ordinary code, you can use any ordinary feature flagging library in your code to conditionally control what ends up in the outputted Cloudformation or Terraform files.

(I’ve never actually used either but that’s my understanding)

However, you’re still on the hook for finding a way for your feature flag vendor to trigger webhooks whenever feature flag state changes and trigger your CI/CD provider to re-deploy the infrastructure.

## Why is this so Frustrating?

In general, declarative configuration isn’t context-aware. It can’t even try to be on its own, it’s just static data at the end of the day.

Also in general, declarative configuration is extremely highly coupled to the things that interpret it (Node.js and AWS APIs in the above 2 examples). There’s no “space” to insert an adapter where feature flags could control how they’re exposed to their consumers.

That’s why things work when configuration becomes true code evaluated in a runtime. Because that allows for the extra “space” needed for feature flags to slot into.

## Will this be Less Frustrating in the Future?

I doubt it. 

It seems like there will always be a tradeoff between the simplicity of using declarative configuration and the flexibility of using true runtime-based code.

That this tradeoff applies to how feature flags can be applied to configuration is something I’m going to try to remember moving forward.