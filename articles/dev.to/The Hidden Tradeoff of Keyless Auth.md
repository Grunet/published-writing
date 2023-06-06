# The Hidden Tradeoff of Keyless Auth

## What is Keyless Auth and Why Should I Care?

Keyless auth refers to being able to authenticate to a system without using any long-lived credentials.

This means getting access to a non-public system without a username/password, a public/private key pair, an access key, etc… while (somewhat magically) maintaining security.

Here are a few places using it today

- Getting AWS access from a Github Actions workflow
- SSH-ing into VMs using Teleport
- Signing artifacts using cosign and sigstore

You should care because any long-lived credential you can get rid of is 1 less target for attackers to compromise.

## How do Keyless Auth Systems Work?

There are usually 3 parties involved in an auth interaction

- The Requestor (e.g. a Github Actions workflow run)
- The Identity Provider (e.g. Github’s OIDC provider)
- The Resource Provider (e.g. an AWS account)

The flow then goes something like this

1. The Requestor wants to access the Resource Provider
2. The Requestor asks the Identity Provider for a token capturing the identity of the Requestor
3. The Identity Provider vends it a token
4. The Requestor sends the token to the Resource Provider
5. The Resource Provider then sends the token back to the Identity Provider, asking if this is a valid request
6. The Identity Provider confirms it just made that token and it’s expected
7. The Resource Provider allows the Requestor time-limited access via temporary credentials

The critical part here is the Identity Provider (e.g. Github’s OIDC provider) and the Resource Provider (e.g. an AWS account) have already previously established a trust relationship via configuration inside the Resource Provider. That’s what enables the Resource Provider to trust that the token isn’t malicious.

But there’s a catch to this that no one seems to talks about. 

## The Hidden, Unspoken Tradeoff

Imagine that Github’s OIDC provider were to get compromised (it’s not inconceivable. It’s a massive target just like LastPass or CircleCI were). This would then mean that malicious actors could also get access to any AWS accounts configured for keyless auth from Github Actions.

The same exploit is not necessarily possible if you’re managing your own long-lived AWS access keys. You can make securing them fully independent of the security position of Github’s OIDC provider.

So the tradeoff in general is putting trust in the Identity Provider’s security at the expense of losing control of the security surface for your Resource Provider.

## Takeaways

I will continue to use keyless auth solutions as I think the tradeoff is almost always worth it.

However, I will now think twice about the vendors involved before jumping for it.