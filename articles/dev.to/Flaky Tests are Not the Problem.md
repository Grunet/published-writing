# Flaky Tests are Not the Problem

## Table of Contents

- [Flaky Tests are Fundamentally Misunderstood](#flaky-tests-are-fundamentally-misunderstood)
- [Flake in Scientific Research](#flake-in-scientific-research)
    - [Analogies Between Software Testing and Scientific Research](#analogies-between-software-testing-and-scientific-research)
        - [“Test” is like “Measurement” or “Observation”](#test-is-like-measurement-or-observation)
        - [“Unit Test” is like “Controlled Experiment”](#unit-test-is-like-controlled-experiment)
        - [“Integration Test” is like “Uncontrollable Observation”](#integration-test-is-like-uncontrollable-observation)
    - [Analogies Between Flaky Tests and Measurement Noise](#analogies-between-flaky-tests-and-measurement-noise)
- [Flake in Production](#flake-in-production)
- [Flaky Tests are Noisy Information Sources](#flaky-tests-are-noisy-information-sources)
    - [The Ideal Lifecycle of a Flaky Test](#the-ideal-lifecycle-of-a-flaky-test)
    - [Analysis of the Ideal Lifecycle of a Flaky Test](#analysis-of-the-ideal-lifecycle-of-a-flaky-test)
        - [Taking a Statistics-First Approach](#taking-a-statistics-first-approach)
        - [Sample Size Matters](#sample-size-matters)
        - [Improved Precision-Recall Optimization](#improved-precision-recall-optimization)
- [Why is this Not a More Common Perspective?](#why-is-this-not-a-more-common-perspective)
- [How We can Unlock the Full Potential of Flaky Tests](#how-we-can-unlock-the-full-potential-of-flaky-tests)

## Flaky Tests are Fundamentally Misunderstood

Flaky tests (i.e. automated tests that fail intermittently even when code hasn’t changed) are universally loathed. No one likes to see random failures in test suites that are unrelated to the changes they’re working on.

However, I believe flaky tests don’t deserve the hate they get, and are actually fundamentally misunderstood. In fact, I think flaky tests can actually be more useful than non-flaky tests.

To lay the groundwork for that hot take, we first need to talk about flake in two other contexts:

- Scientific Research
- Production

## Flake in Scientific Research

Software testing and scientific research have a lot in common, but they use different terms to refer to the same or similar things.

Before we can understand flake in the context of scientific research, we need to better understand both sets of terms.

### Analogies Between Software Testing and Scientific Research

#### “Test” is like “Measurement” or “Observation”

You can test an API by sending a request to it and inspecting the response.

You can increase the voltage input to a circuit and measure the resulting changes in current.

In both cases you’re trying to gain information about a system.

#### “Unit Test” is like “Controlled Experiment”

Imagine a function that reads from a database and filters the results. A unit test will pass inputs to the function and inspect the outputs, but will also replace the database by an in-memory mock.

Imagine trying to make accurate estimates of the constant of gravitational acceleration near the earth’s surface. You could create a machine to hold an object, drop it, then time how long it takes to hit the floor, but in a room where the air has been pumped out to reduce air resistance.

In both cases there is a variable that is being explicitly controlled to prevent it from influencing the results.

#### “Integration Test” is like “Uncontrollable Observation”

An automated test might query an API you’ve hosted in AWS (the integration being between your code and AWS). AWS sometimes has outages, causing your test to fail despite your code not having changed.

You could be trying to observe weak electrical signals in the face of [Johnson-Nyquist noise](https://en.wikipedia.org/wiki/Johnson%E2%80%93Nyquist_noise) and other forms of intrinsic noise that cannot be removed.

In both cases there is a variable that is impossible to control that is also influencing the results.

### Analogies Between Flaky Tests and Measurement Noise

The last example above establishes the analogy between flaky tests and measurement noise. Specifically, it highlights the important case where it is fundamentally impossible to remove the flake or noise from the system.

In scientific research, even when removing noise from a system is possible, it’s not always needed. Significant results can still be deduced in the presence of noise. 

A famous example of this is the [Luria-Delbrück fluctuation assay experiment](https://en.wikipedia.org/wiki/Luria–Delbrück_experiment). It was actually an analysis of the noise of the results (and that the noise was greater than the competing hypothesis would have predicted) that led to the critical deduction. The noise was the signal in this case.

<figure>
![The text Induced mutations above 4 diagrams of petri dishes. Each dish has 2 red cells in it, where cells are represented as ovals. Beneath them is a horizontal dividing line and then the text Spontaneous mutations above 4 diagrams of petri dishes. The first dish has 1 red cell in it, the second has 4, the third has 0, and the fourth has 2. The 2nd and 4th are notable because above the dish is a diagram of the ancestry of the cells currently in the dish. In both cases it shows the progenitor of the trait of redness spontaneously arose several generations in the past, and then duplicated each generation to create the red descendants we currently see in the petri dishes. This behavior was not present in the Induced mutations petri dishes at all.](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/o8sqluln4ukx6sm7e5lj.png)
<figcaption>
Don't stare at this too much. But if you do, notice how the counts of cells in the petri dishes on top are always the same, whereas the ones on the bottom are more variable. This single train of thought won Luria and Delbrück Nobel Prizes. It was a simpler time.
</figcaption>
</figure>

Another perspective on this came from an experimental physicist who once said 

> Error is just physics you don’t understand yet

Embracing the idea that measurement noise is as much a fundamental part of the systems you investigate as the actual measurements themselves.

Software testing stands in stark contrast, where flake is seen as something to quarantine, eradicate, or even wage war on.

## Flake in Production

Flake also exists in other parts of the software lifecycle, in particular when it comes to operating software in production.

Here are several relevant areas that all have to do with extracting information from production and operationalizing it.

- Alerting and On-Call
- Synthetic Monitoring
- Error Tracking
- Performance Monitoring

In each of these cases, noise can be rampant and will always exist no matter how much it’s tuned due to intrinsic, irremovable precision-recall tradeoffs (i.e. favoring false-positives vs favoring false-negatives). 

However, noise is seen as a natural and expected part of these areas. Something to intentionally and thoughtfully account for.

Contrast this with software testing, where flake is seen as an abnormal state, one to avoid or eliminate as much as possible.

<figure>
![A box of Corn Flakes cereal, tilted slightly so you can see 1 side as well as the front. The front says Kellogg's Corn Flakes next to a green rooster drawn out of simple curved lines, sort of like abstract art. An image of the cereal is shown beneath them.](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/st477rqo44pjfapznoh9.jpg)
<figcaption>
I also try to avoid or eliminate Corn Flakes as much as possible.
</figcaption>
</figure>

## Flaky Tests are Noisy Information Sources

Just like their synthetic monitoring cousins in production, flaky tests should be perceived as noisy information sources.

Not only that, flaky tests’ noisiness should be fully embraced, just as is done in scientific research.

To more concretely illustrate how this perspective would fit into software testing, let’s examine a hypothetical life cycle of a flaky test in this new paradigm.

### The Ideal Lifecycle of a Flaky Test

Imagine you’re a developer on a team building a web service.

Your dev environment is cloud native and serverless (e.g. AWS API Gateway, Lambda, and DynamoDB) as is CI, to more closely match production.

You’re just finishing up a new feature, and you

1. Write an automated test against your dev environment
2. After your PR is merged in, the test is run against 1000 fresh copies of the cloud native CI environment
3. The counts of passing runs (900) vs failing runs (100) are recorded in a system somewhere
4. Every time a new change is merged in afterwards, the test is re-run against 1000 fresh copies, and the passing/failing counts are recorded
5. After a few merged changes, the test’s counts sits at 90010 passing vs 9990 failing
6. Someone else merges a change in, but this time when the test is run it registers 200 passing runs and 800 failing runs
7. The system infers that this is very, very unlikely to have happened due to chance, and declares the CI build broken by the latest change
8. It turns out the latest change caused a regression that impacted the flaky test you wrote

## Analysis of the Ideal Lifecycle of a Flaky Test

There are 3 key aspects of the above scenario that are worth taking a closer look at.

#### Taking a Statistics-First Approach

The 1st critical aspect of the above scenario is that we’re no longer thinking of tests as only needing to exist in the 2 states of

- Passing 100% of the time
- Passing 0% of the time

It’s completely fine if a test only passes 90% of the time (or any fraction thereof), because that still carries a lot of information. More than enough to detect regressions with.

Thinking of tests as noisy sources of information requires that we handle them with statistical methods.

#### Sample Size Matters

The 2nd critical aspect is that we’re running the test on large numbers of independent and identical copies of the system in parallel. 

That’s what enables us to immediately get high confidence after the initial merge that the test’s pass rate is somewhere around 90%. Then subsequent runs don’t deviate significantly from that and further add data to increase confidence around that estimate. Then when the rate changes dramatically on a run, we can be highly confident some real change with the system’s behavior has occurred.

#### Improved Precision-Recall Optimization

The 3rd subtler aspect is that the flaky test likely has a better precision-recall tradeoff than non-flaky tests. A non-flaky test is always highly precision-optimized (i.e. it minimizes false positives at the cost of increased false negatives). But a flaky test aggregated over multiple runs should be able to compare favorably on precision while doing much better on recall (i.e. false positives are still low while false negatives are also reduced).

The intuition here is that in order to make a test non-flaky, you have to control for a lot of variables that account for important aspects of system behavior (i.e. you have to bring it closer to a unit test). This in turn reduces its ability to catch regressions because it’s ignoring a lot of phenomena.

Statistically analyzing a flaky test allows you to keep those important variables uncontrolled, while still managing the noise they can introduce. This increases its ability to catch regressions.

<figure>
![The front of a box of Kellogg's Frosted Flakes cereal. It says Frosted Flakes above an anthroporphic tiger named Tony gesturing energetically with 1 finger pointing towards the sky and his other hand clenched into a fist while saying They're Great but rolling the R](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6ljx7hmc4ed2bqguv9so.jpg)
<figcaption>
You think Tony would do a better job of shilling for flaky tests? Don't answer that.
</figcaption>
</figure>

## Why is this Not a More Common Perspective?

To be completely honest I’m not entirely sure. The only vague thoughts that have come to mind are

- Lack of tooling (e.g. common tools don’t easily support analyzing results this way)
- Cost (e.g. extra test runner costs, extra infrastructure costs, extra maintenance costs, etc…)
- History (e.g. things are the way they’ve always been)

## How We can Unlock the Full Potential of Flaky Tests

From a technical point of view, here’s the bare minimum I can envision at a high-level

1. Test frameworks would need to make it dead simple to run N copies of a test at once (potentially distributed in execution, in various senses)
2. CI frameworks and providers would need to facilitate keeping all test data from all N copies of a run every time
3. Something would need to be analyzing that data (e.g. via a simple Bayesian framework), detecting shifts in distribution, and breaking the CI build accordingly

But mostly I’m just hoping this resonates with other folks on some level. That more than anything seems key to transforming how we perceive and leverage flaky tests.