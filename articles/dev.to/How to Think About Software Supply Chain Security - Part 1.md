# How to Think About Software Supply Chain Security

## How Software Supply Chain Security Differs from Normal Security

With normal security, the concern is primarily with malicious, external actors probing your software looking for direct exploits.

With software supply chain security, the concern is more about malicious actors exploiting backdoors in the creation of your software.

The terminology is different too. Whereas it makes sense to talk about "trust" in the context of normal security, that concept loses usefulness in the context of software supply chain security.

## What to be Concerned about When it Comes to Software Supply Chain Security

There are 2 main areas to be concerned about

1. Artifact Integrity
2. Exfiltration

### Artifact Integrity

Artifact integrity has to do with trying to make sure that the software that was delivered and/or is running in production is actually what was intended to be made.

An example of when this would fail is when a malicious actor is able to modify the source code used to build a container, including a backdoor that lets them collect sensitive user information.

### Exfiltration

Exfiltration in this context is concerned with trying to make sure sensitive parts of the supply chains themselves aren't stolen (e.g. source code, build secrets).

An example of when this has happened is the [CircleCI security incident](https://circleci.com/blog/jan-4-2023-incident-report/) from a few months ago, where all customer build secrets were compromised by a malicious actor.

## How to Think About the Techniques That Address The Concerns

There are many techniques available to address these two concerns, but one helpful way to categorize them is how they impact the risks involved. The 3 most prominent categories are

1. Risk Elimination
2. Risk Mitigation
3. Risk Awareness

### Risk Elimination

These techniques eviscerate certain classes of risk altogether. For example,

- Signing all Git operations (e.g. commits, tags)
- Automating builds and running them in an isolated, ephemeral environment

The former prevents malicious actors from impersonating valid contributors.

The latter prevents any long-lived malicious software from living in the build environment.

### Risk Mitigation

These techniques reduce the chances of certain classes of risk. For example,

- Peer review of code changes
- Using a dedicated build service

The former mitigates the case of 1 disgruntled employee trying to submit malicious code, but it does nothing for the case of 2 disgruntled employees colluding to submit malicious code.

The latter will generally improve the security of the secrets kept inside the build service. However, as the CircleCI incident showed, all build service platforms are still fallible when it comes to secrets exfiltration.

### Risk Awareness

These techniques give you more insight into the risk profile of certain classes of risk. For example,

- Gathering all manifests (e.g. Software Bills of Material, SBOMs) of all of your dependencies
- Checking Reddit before updating a dependency in case there's a well-known compromise in flight

The former helps increase awareness of all the pieces comprising the software, as well as their individual security vulnerabilities (notice how this overlaps with normal security concerns). 

The latter will help you become aware before merging a Dependabot dependency update PR that may contain malicious code.

## Takeaways

Risk is the primitive to use when thinking about software supply chain security.

Thinking about the risk a technique or tool impacts can make it easier to reason about.

For more on this, including more practical examples, check out [Part 2 of this series](https://dev.to/grunet/how-to-think-about-software-supply-chain-security-part-2-964).

To take a deeper dive into the world of software supply chain security, check out https://slsa.dev/