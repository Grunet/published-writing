# Operational Challenges for SCIM Servers 

## Table of Contents


- [What is SCIM?](#what-is-scim)
- [The Key Operational Challenges](#the-key-operational-challenges)
  * [No Load Limits](#no-load-limits)
  * [All Requests Must be Synchronously Handled](#all-requests-must-be-synchronously-handled)
- [Downstream Consequences when Faced with “Scale”](#downstream-consequences-when-faced-with-scale)
  * [Performance Problems with 3rd Party SCIM Libraries](#performance-problems-with-3rd-party-scim-libraries)
  * [ORM Problems Leading to Database Performance Problems Leading to Other ORM Problems](#orm-problems-leading-to-database-performance-problems-leading-to-other-orm-problems)
  * [Inability to Throttle Requests](#inability-to-throttle-requests)
  * [Inability to Queue Requests](#inability-to-queue-requests)
  * [Inability to Horizontally Scale](#inability-to-horizontally-scale)
- [Takeaways](#takeaways)
  * [Targeted Avoidance of the ORM-to-SCIM-to-ORM Pattern is Valuable](#targeted-avoidance-of-the-orm-to-scim-to-orm-pattern-is-valuable)
  * [Threading is Potentially Valuable, Maybe](#threading-is-potentially-valuable-maybe)
  * [Pre-Production Load Testing of SCIM Servers is Valuable](#pre-production-load-testing-of-scim-servers-is-valuable)
  * [Recording Failed SCIM Requests in Production Telemetry is Valuable](#recording-failed-scim-requests-in-production-telemetry-is-valuable)

## What is SCIM?

SCIM (System for Cross-domain Identity Management) is an open standard for user provisioning.

For example, it allows an organization that is a customer of a SaaS product to easily sync all of their users’ identity information into the SaaS’s databases.

The organization’s users’ identity information will often be stored in a 3rd party identity provider like Okta or Azure Active Directory (Azure AD). The identity provider will act as a SCIM client, sending requests with provisioning data to a SCIM server managed by the SaaS product.

Building a conformant and operational SCIM server however is a non-trivial task.

## The Key Operational Challenges

There are 2 inherent operational difficulties all SCIM server implementations must face.

### No Load Limits

A SCIM client has no restrictions on how quickly it can send requests to your SCIM server.

In theory, this means that your SCIM server needs to be able to handle arbitrary requests at arbitrary concurrency.

In practice, this means that your SCIM server needs to be able to handle the load of “the worst” SCIM client out there (Azure AD’s SCIM client has been observed to send somewhere just shy of 1000 requests per minute at peak load)

### All Requests Must be Synchronously Handled

SCIM clients rely on the status code of responses to determine whether or not to retry a request (e.g. a client may retry all 500-level errors until it succeeds).

This means that all request processing has to be done synchronously (i.e. it can’t be offloaded to consumers of a separate queue).

## Downstream Consequences when Faced with “Scale”

When faced with increased load and larger customers, several different types of problems can arise for a SCIM server implementation.

### Performance Problems with 3rd Party SCIM Libraries

Handling SCIM requests means writing the logic for applying a request’s changes to a SCIM resource (e.g. PATCH-ing a group).

Depending on your runtime, there may be existing libraries that already have this logic ready for re-use (e.g. [scim-patch](https://www.npmjs.com/package/scim-patch) for PATCH requests in Node.js). However, a library’s code may not necessarily be optimized for performance in every case. 

For example, a library function’s execution time may sometimes scale quadratically with the size of the request and/or SCIM resource (e.g. for groups with a large number of users). This can drastically slow down response times (think 10s of seconds) and hog server CPU (a lethal problem for single-threaded runtimes like Node.js).

### ORM Problems Leading to Database Performance Problems Leading to Other ORM Problems

If you’re using an ORM (Object Relational Mapping), all of your PATCH or PUT SCIM endpoints may work as follows

1. Load the ORM’s representation of the SCIM resource from the database
2. Transform the ORM’s representation into a SCIM representation expected by the SCIM library
3. Apply the request to the SCIM representation using the SCIM library
4. Transform the updated SCIM representation back into an ORM representation of the resource
5. Save the ORM representation back to the database

The problem with this can occur in Step 5, as the ORM has no idea of what exactly changed and so can in certain cases end up recreating the database records from scratch.

For example, consider PATCH-ing a very large group (10,000 or more users) to add 1 user to the group, where in the database users in a group are represented in a join table between the Groups and Users tables.  In Step 5, the ORM isn’t aware that 1 user was added, so instead it generates SQL to delete all the existing users from the group and then re-insert them all plus the 1 new user.

This can become extremely problematic when these requests happen at high concurrency (e.g. Azure AD sending a huge number of near parallel requests to add 1 user at a time to a group). Every request requires an exclusive lock on the join table because of the deletes, so the transactions end up being processed 1 at a time at the database-level, leading to very slow query times.

Because of these delays in database processing, ORM operations will start to back up in their queue and time out. ORM database connection pools will become maxed out as well. This will cause a massive percentage (e.g. 95+%) of the requests to fail. So many that no amount of retrying from the SCIM client will fix things.

![The "This is fine" popular internet meme. A two panel comic. The first panel shows an anthropomorphized dog sitting on a chair next to a table with a coffee mug on it. The room the dog is in is covered in flames and smoke covers the ceiling. The dog's eyes do not seem to betray any fear of the situation. In the second panel we zoom in on the dog's face where the dog says this is fine in a speech bubble that appears above their face. ](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/bqsy0rh0iv3xzb5m7gqp.jpg)

### Inability to Throttle Requests

Because of the severe slowdown and serialization in request processing under this high concurrency, throttling is ineffective as it will just lead to timeouts at the gateway instead (e.g. 504s from the load balancer fronting the SCIM server replicas) .

### Inability to Queue Requests

Because SCIM requires that requests be synchronous, putting the requests onto a separate queue (e.g. an SQS queue) to avoid all of the aforementioned problems won’t work because if a queue consumer fails to process a request for some reason there’s no way to indicate that to the SCIM client so it knows to retry.

### Inability to Horizontally Scale

SCIM clients’ barrage of requests can start suddenly and end just as quickly (think minutes) which isn’t enough time for traditional autoscaling tools to respond by creating more replicas of the SCIM servers.

Also in the case of the database lock contention issue mentioned before, more replicas (and hence more available database connections) can actually make things worse, as the line of concurrent transactions waiting on the database lock will grow even longer.

## Takeaways

Stepping back, there are a few high-level things to take away from the previous pessimism. 

### Targeted Avoidance of the ORM-to-SCIM-to-ORM Pattern is Valuable

In the large group scenario from before, if all of the requests happen to be adding or removing 1 user from a group, making that 1 change directly to the database (via the ORM) rather than creating an intermediate SCIM representation of the group that the ORM then has to save back to the database can avoid all of the aforementioned problems.

### Threading is Potentially Valuable, Maybe

As an alternative to using a separate queue, using separate runtime threads and an in-memory request queue may help avoid saturation in the specific case of a server CPU bottleneck coming from request processing.

### Pre-Production Load Testing of SCIM Servers is Valuable

Simulating the type and concurrency of requests that SCIM clients may deliver should be a valuable exercise, as it may unearth several bottlenecks in the SCIM server that may not become otherwise apparent until production.

You might try forking https://github.com/wso2-incubator/scim2-compliance-test-suite and adjusting it to be able to send requests at high concurrency towards this end.

### Recording Failed SCIM Requests in Production Telemetry is Valuable

The first step in addressing a novel operational problem happening with your SCIM servers in production is usually to develop some understanding of it. 

On top of your usual sources of telemetry (e.g. OpenTelemetry, Application Performance Monitoring tools, RDS Performance Monitoring) recording the raw SCIM request data of failed requests in your telemetry (e.g. logs, traces) can be very helpful in figuring out what exactly is going on.