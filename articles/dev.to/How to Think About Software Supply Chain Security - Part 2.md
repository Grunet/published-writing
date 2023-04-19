# How to Think About Software Supply Chain Security - Part 2 

(If you haven’t already, read or skim [Part 1 of this series](https://dev.to/grunet/how-to-think-about-software-supply-chain-security-28eg) first for background.)

## Tracking Confidence and Risk

![It looks like a histogram of a narrow bell curve, rotated 90 degrees clockwise so the base is at the far left and the tip is at the far right. Inside the bell curve near the left is the text High Confidence in a large font size. Inside the tip near the right is the text Low Confidence in a small font size. To the right of each bar of the histogram is a phrase. The phrases are the heading level threes in the article below. Each one is a software supply chain security risk, and the idea is they're chipping away at confidence. There's an arrow at the bottom indicating the risks to the left occur closer to the development phase, whereas the risks to the right occur closer to the deployment to production phase, roughly speaking.](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/nr06vaxcwz3d5tbmqwjs.png)

There are an extremely large number of software supply chain security risks. Each one of these risks can reduce confidence in the security of the software development process. 

When someone has an idea for a software change, it’s at its most secure. Peoples’ brains cannot be infected or manipulated that easily (by software).

However the change then has to go through design, then development, then validation, and ultimately deployment or release. At each stage there are a multitude of software supply chain security risks that can erode confidence. 

If left unmitigated, these risks can add up to a complete loss of confidence in the integrity of the final product and the security of the process itself.

## Diving Deeper Into Individual Risks

What follows is a brief exposition of each of the risks included in the above diagram.

Note that what the diagram covers is only a small sample of all software supply chain security risks.

### Lack of 2FA on Github

If your Github password gets compromised, an attacker can now act as you.

For example, they might write a Github workflow to exfiltrate all of your build-time secrets.

Enforcing 2FA on all user accounts mitigates this risk.

### Long-Lived Github Personal Access Tokens

If a PAT (personal access token) gets compromised, an attacker now has access to all of the allowed permissions of the token and the Github account the PAT is from, regardless of 2FA.

For example, they could use the PAT to steal all of your confidential source code, and then use that information in a subsequent attack.

There is no general mitigation for this (that I can think of) outside of avoiding use of PATs.

### Signed Commits Not Required

If signed commits aren’t required in your Github repository or aren't in use by your team, an attacker who has already compromised some Github account can modify your commits after they’ve been made.

For example, they might modify a commit you had previously made on a PR and that a reviewer had already reviewed, sneaking in subtle runtime secrets exfiltration code.

Requiring signed commits in your repository eliminates this risk. And if you use Github Codespaces, your commits will automatically be signed.

### Main Branch Not Protected

If the main branch of your repository isn’t protected, an attacker who has already compromised some Github account can directly commit changes to it without anyone noticing.

For example, they could add in some subtle runtime secrets exfiltration code.

Protecting the main branch eliminates this risk.

### Code Review Not Required

If code review is not required in your repository, an attacker who has already compromised some Github account can make a PR (pull request) and merge it into main all by themselves.

For example, they could add in some subtle runtime secrets exfiltration code via the PR.

Requiring code review on all PRs eliminates this risk.

### Builds Not Fully Automated using a CI Service

If builds are not automated in an ephemeral environment, then malware lingering in the environment can infect the builds.

For example, if a container image is built on someone’s computer, existing malware running on that computer could modify what’s included in the container image, inserting a backdoor for when it’s running in production later on.

Using a build service like Github Actions prevents this possibility from happening, since the build is run on a new, clean VM (virtual machine) each time.

### Dependencies’ Versions Not Pinned

A dependency could be compromised by an attacker and a new malicious version of the dependency published. 

If dependencies aren’t pinned, the next build will pull in the malicious version of the dependency.

Pinning the dependency ensures the same code is used each time unless someone explicitly chooses to change it.

### Dependencies Neither Cached in CI Nor Vendored

If every build fetches dependencies from the internet, then that increases the chances of pulling in an existing version of a dependency that’s been corrupted.

Caching dependencies in CI helps reduce the number of times fetching from the internet is required. Vendoring dependencies (i.e. including their code in your source code) erases this problem altogether.

### Dependencies Updated Too Often

If you update your dependencies anytime a dependency publishes a new version, you’re at increased risk that one of those dependency updates has been compromised and you'll now be pulling in its malicious code.

Keeping your dependencies up-to-date only whenever there’s a new major version, while also taking all security patches is one way to achieve a safer balance.

### Secrets Not Restricted to Protected Branches

If you have Github Secrets that are accessible outside of protected branches, anyone (e.g. a disgruntled employee) can write a Github Workflow in a throwaway branch to exfiltrate those secrets.

Environment-based secrets in Github Actions can restrict secrets to protected branches and eliminate this risk.

### Secrets Not Least Permissioned

If secrets include things like AWS access keys, and the permissions behind them are very broad (e.g. Adminstrator-level on an AWS account) if/when the secrets are exfiltrated, an attacker will have wide access to your AWS accounts.

Restricting these kinds of secrets to the least permissions required to perform their functions (e.g. only enough for pushing to a container registry) is one mitigation.

Another (imo easier) mitigation if the tool supports it is to use ephemeral access keys via Github’s OIDC provider. These can be configured so the access keys only exist for a few minutes, so even if they are exfiltrated they are hard to abuse.

### Outbound Requests Made during CI

Disallowing outbound requests during CI prevents any malicious code that’s already infiltrated your CI environment from exfiltrating any secrets. (this is similar to the idea of “air gapping” a build)

In practice this can be difficult to pull off, or even monitor for.

### Deploys/Releases Not Automated

If deployments or releases aren’t automated from an ephemeral environment, then malware lingering in the environment can affect the deployment or release.

For example, if changes to your cloud IAC (infrastructure as code) are done from someone’s computer, existing malware on their computer could include extra cloud resources into changesets (e.g. cryptominers).

Outside of fully automating deploys or releases, one mitigation for this is to do the process from a clean machine (e.g. Cloud Shell in AWS or GCP) every time.

### Production Access Outside of Deployment/Release Branch

This is the same as “Secrets Not Restricted to Protected Branches” for the special (and much worse) case that the secrets contains access credentials to your production environments.

The same mitigation about using Environments in Github Actions to restrict the branches the secrets are accessible from applies here.

## Takeaways

Supply chain security is hard to grok because it requires you to be skeptical about things you also have to trust heavily. This tension is difficult to grapple with.

Thinking about things from a risk-first (or equivalently confidence-first) perspective has proven useful to me in dealing with this tension.

To take a deeper dive into the world of software supply chain security, check out https://slsa.dev/