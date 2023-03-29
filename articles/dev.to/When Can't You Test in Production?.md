# When Can’t You Test in Production?

## Table of Contents

- [What is Testing in Production?](#what-is-testing-in-production)
- [Why Test in Production?](#why-test-in-production)
  * [There’s Only 1 Production Environment](#theres-only-1-production-environment)
  * [The Majority of Users Only Interact with Production](#the-majority-of-users-only-interact-with-production)
- [When Can’t You Test In Production?](#when-cant-you-test-in-production)
  * [Can Bugs Cause Significant Harm to Human Life?](#can-bugs-cause-significant-harm-to-human-life)
  * [Is the System Highly Coupled to Integrations with IRL Things?](#is-the-system-highly-coupled-to-integrations-with-irl-things)
  * [Is Usage Fully Anonymous and Software Changes Can’t be Correlated with Business Outcomes?](#is-usage-fully-anonymous-and-software-changes-cant-be-correlated-with-business-outcomes)
- [Summary](#summary)

## What is Testing in Production?

Testing in production is the practice of generating information on how users and software systems behave in production.

> Note specifically the term “generating”, as opposed to “gathering”. This is to distinguish testing in production from the related area of observability.
> 

Common activities that fall into this bucket include

- A/B testing, canarying, ring deployments
- Synthetic Monitoring, manual testing in production
- Chaos engineering, fault injection, load testing

## Why Test in Production?

Testing in production has 2 key benefits that are otherwise difficult to get.

### There’s Only 1 Production Environment

Testing anywhere else will come with tradeoffs in how representative the results will be for production.

For example, load testing done on a container run locally may generate different findings than load testing done against that container deployed in AWS Elastic Container Service.

### The Majority of Users Only Interact with Production

Generating information anywhere else will come with the tradeoff of not being representative of the majority of user behaviors.

For example, user interviews done with a small sample of users may indicate a strong need for a feature. But, it may turn out later post-development that it’s rarely used in production since the sample wasn’t representative of most users.

## When Can’t You Test In Production?

There are 3 key dimensions for deciding whether or not testing in production is viable for a given context.

### Can Bugs Cause Significant Harm to Human Life?

If yes, testing in production is a no-go.

An example of this is a medical device company that makes pacemakers. There is no room for software error in production in this case.

Instead, all efforts need to be focused on finding damaging problems prior to production.

### Is the System Highly Coupled to Integrations with IRL Things?

If yes, testing in production is a no-go.

It’s impossible to fake a social security number, and strenuous to try and use a company credit card for payment flows. Don’t even get me started on Captcha.

To enable testing in production, the software needs to be adjusted so that authenticated internal employees can use test doubles instead of the integrations.

For example, an internal employee might be targeted via a feature flag, giving them access to perform a payment flow that’s using a test API key from Stripe in place of the real one.

The key tradeoff here is that the surface area for abuse or security breaches widens. There are now workflows that allow you to fake credit cards, fake social security numbers, bypass Captcha, etc… that are 1 misconfiguration away from an exploit.

### Is Usage Fully Anonymous and Software Changes Can’t be Correlated with Business Outcomes?

If so, testing in production is a no-go.

Traditional A/B testing in this case is pointless, since it’s impossible to tell if the change made a difference or not.

For example, the owners of an HTML-only blog site might want to experiment with ways to increase how long people stay on the site. However, this is impossible to measure with server-side telemetry alone, so they can’t actually perform any A/B tests.

## Summary

Testing in production is an option if

- Your software can’t severely harm people, and it avoids high coupling with IRL things
- Your software can’t severely harm people, and you have some way to correlate changes in the software to changes in business outcomes

Otherwise, it’s testing in pre-production that will deliver the most value.


![The words Cautionary Tales in the top center. To the lower left it says With Tim Harford. To the lower right it says Pushkin. All the text is white and overlaid over top a picture of books of different colors standing next to each other.](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/l4grft2tjwkjhgob8c1e.png)
