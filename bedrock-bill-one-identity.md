---
title: "Your gateway is why the Bedrock bill has one line"
description: "Amazon solved per-team attribution for callers that reach Bedrock directly. Putting a gateway in front is how you give that up, and the gateway is the only thing that can give it back."
permalink: /bedrock-attribution/
---

# Your gateway is why the Bedrock bill has one line

*Amazon solved per-team attribution for callers that talk to Bedrock directly.
Putting a gateway in front is how you give that up — and the gateway is the only
thing that can give it back.*

Here is the moment you notice. Finance asks which team is responsible for the
model spend, you open Cost Explorer expecting to group by principal, and every
request in the account arrives under one identity: the gateway's service role.
Not wrong, exactly. Just useless.

## This is structural, not a bug

Bedrock attributes inference cost to the IAM principal that made the call. That
is a good design and it works exactly as intended — right up until something
sits in the path.

A gateway authenticates its callers at its own layer. It knows perfectly well
that this request came from the search team and that one from billing. Then it
calls Bedrock under a single service role, and every one of those requests
reaches AWS wearing the same face. The distinction you wanted on the bill is
precisely the distinction the proxy just erased.

Nothing here is anyone's mistake. It is what proxying *means*. But it produces a
specific irony worth sitting with: **the component that destroyed the identity is
the only component still holding it.** By the time the request reaches AWS the
information is gone. By the time it reaches your gateway it is still right
there, in a header.

## The answers that don't work

**Tag the requests.** Bedrock doesn't bill on request tags. You can attach all
the metadata you like to an invocation; the Cost and Usage Report groups by
principal.

**One AWS account per team.** This does work, and if you already have it, use
it. But account-per-team is an organizational decision with blast radius far
beyond your model spend, and "our gateway can't split a bill" is a poor reason
to make it.

**Parse your own logs and reconcile.** Now you have a second billing system,
maintained by you, that has to agree with the first one. It will drift. Someone
will ask which number is authoritative and you will not enjoy the conversation.

**A role per team, wired by hand.** Closer — this is actually the right shape —
but done manually it means every team's application holds credentials for its
own role, which is most of the reason you built a gateway.

## What AWS actually documents

The supported answer for exactly this case: the gateway assumes a role per
caller, passing the caller as the `RoleSessionName` and its attributes as
**session tags**. Tagged sessions surface in Cost and Usage Report 2.0 under an
`iamPrincipal/` prefix, once you activate the tag as a cost allocation tag in
Billing.

It is documented. It is also, in my experience, not widely known — I have read a
lot of "we built an LLM gateway" writeups and I do not recall one that mentions
session tags.

## Doing it per request

[switchboard](https://github.com/Grace/switchboard) is an LLM gateway I've been
building — one OpenAI-compatible endpoint in front of Bedrock and on-device
inference. As of v0.1.0 it does this per request.

```json
{
  "attribution": {
    "enabled": true,
    "role_arn": "arn:aws:iam::123456789012:role/switchboard-caller",
    "tag_key": "team",
    "session_duration": "15m",
    "require_caller": true
  },
  "teams": [
    { "name": "search",  "keys": ["sk-switchboard-search-…"] },
    { "name": "billing", "keys": ["sk-switchboard-billing-…"] }
  ]
}
```

A caller presents its key as a bearer token. switchboard resolves the key to a
team, assumes the role with `RoleSessionName` set to that team and a session tag
of `team=<name>`, and makes the Bedrock call with those credentials. The request
now arrives at the provider wearing an identity the bill can split on.

Credentials are cached per team and refresh ahead of expiry, so a busy team
costs one STS call per `session_duration` rather than one per request.

One consequence worth stating plainly: **attribution and authentication become
the same feature.** A gateway cannot invent an identity it was never given, so
the team list is simultaneously the key roster and the chargeback roster. If you
want the bill split, you have to know who is calling. There is no version of this
that works anonymously.

`require_caller: true` refuses unkeyed requests with a 401. Leave it off and
unattributed traffic bills to the gateway's own role, exactly as it does today —
visible, but nobody's. Fail closed if the bill is the point.

## Three ways this silently doesn't work

**Missing `sts:TagSession`.** The trust policy needs it alongside
`sts:AssumeRole`. Without it the assume *succeeds*, the tag is silently dropped,
and everything bills to one identity again. This is the usual first failure and
it produces no error anywhere.

**The cost allocation tag isn't activated.** Until you turn it on in Billing,
your sessions are tagged and your bill does not group by them.

**You looked too early.** Cost Explorer lags 24–48 hours.

## What I have not verified

The key resolution, team validation, fail-closed behavior, and the assumed
credentials reaching the backend are unit-tested. What tests cannot cover is
whether AWS bills the way this expects — that needs a real account and a full
CUR cycle.

So verify it once, deliberately, before you trust it: send traffic as two teams,
wait for the report, confirm the `iamPrincipal/` prefix splits the way you
expect. If it doesn't, it's one of the three above.

Also not built: **spend caps** (you get visibility, not enforcement — this tells
you spend climbed, it does not stop it) and **chargeback reporting** (the data
lands in CUR, not in anything finance can read without an engineer).

## Why bother

Attribution is the least glamorous thing a gateway does and close to the first
thing anyone asks it for. It's also a decent test of whether a gateway is
actually a platform component or just a shared HTTP handler: a platform
component gives back the things it took away.

Code is at [github.com/Grace/switchboard](https://github.com/Grace/switchboard),
with [the full write-up](https://github.com/Grace/switchboard/blob/main/docs/cost-attribution.md)
including the IAM setup.

If you're running Bedrock for several teams and solving this some other way, I'd
genuinely like to hear how — **grace@gracefulco.de**.
