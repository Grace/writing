---
title: "Your observability tool cannot be your evidence"
description: "Honeycomb said it themselves in 2023: observability and compliance workloads are orthogonal. Sampling is the product; retention is 60 days. Three years on, the answer for the other workload is still 'build a data lake' — here is what actually finishing that takes."
permalink: /observability-is-not-evidence/
---

# Your observability tool cannot be your evidence

*Honeycomb said it themselves in 2023. Three years on, the answer for the other
workload is still "build a data lake" — and the assignment is harder than it
looks.*

Here is the moment you find out. An auditor asks for a record of a model
interaction from the first week of the period. Your period is a year. Your AI
traces are in your observability platform, where they are excellent, queryable,
and thirty to sixty days old at most. The evidence for ten months of a
twelve-month Type II does not exist, and no change you make today recovers it.

This is not a vendor failing. It is a design boundary, and the clearest
statement of it I have found was written by the vendor.

## Honeycomb wrote it down first

From [Infinite Retention with OpenTelemetry and
Honeycomb](https://www.honeycomb.io/blog/infinite-retention-opentelemetry-honeycomb),
Mike Terhar, September 2023:

> The needs of observability workloads can sometimes be orthogonal to the needs
> of compliance workloads… Honeycomb is designed for software developers to
> quickly fix problems in production, where reducing 100% data completeness to
> 99.99% is acceptable to receive immediate answers.

That is exactly right, and it is a statement about design rather than about
roadmap. Two things follow from it that are easy to miss.

**Sampling is the product, not a limitation.**
[Refinery](https://github.com/honeycombio/refinery) is a trace-aware tail-based
sampling proxy whose entire job is to decide which traces you do not need. That
is correct engineering for debugging: you want the errors, the slow ones, the
weird ones, and one in a thousand of the boring ones, because keeping all the
boring ones costs money and answers nothing. Nobody is going to remove it.

**Retention is a business model, not an oversight.** Sixty days standard, longer
by arrangement. Storage that supports sub-second queries over high-cardinality
data is not the storage you want to hold seven years of anything.

Both choices are right for the workload they were made for. Both are
disqualifying for the other one, where completeness *is* the claim and the
obligations run from six months (EU AI Act
[Art. 19](https://artificialintelligenceact.eu/article/19/)) to six years
(HIPAA §164.316(b)(2), FINRA Rule 4511).

## The assignment

Terhar's post does not leave you stranded. Its recommendation is to use
OpenTelemetry's S3 exporter to fan trace data out to object storage, and query it
with Glue and Athena. Debugging stays fast; the archive holds everything.

That is the right shape. It is also a homework assignment, and it has been open
for three years. Having now done a version of it, here is what the data lake
still does not answer.

**Is the archive complete, or did you archive the sampled stream?** This is the
one that matters most and it is invisible once you have got it wrong. If your
sampler sits upstream of the exporter that writes to object storage, your
compliance archive contains exactly the traces that survived sampling. The
pipeline runs, the bucket fills, the dashboards look right, and the archive is
a subset of unknown shape. **Where you put the sampler decides whether your
archive is evidence**, and nothing about the running system tells you which side
of it you are on. Go and look at your collector config; it is a five-minute
check and I have never once regretted running it.

**Can you prove nobody edited it?** A bucket you control is a bucket you can
rewrite. Object Lock in compliance mode fixes this and is genuinely good — and
it is off by default, it has to be set at bucket creation, and "we have S3" is
not the same sentence as "we have Object Lock." An archive whose integrity rests
on the access controls of the team being audited is an assertion, not evidence.

**Can you say what rules were in force?** A trace records what happened. It does
not record the model roster, the redaction rules, the tool grants or the rate
limits that were in effect when it happened — and "was this allowed under the
policy at the time" is the question a disputed decision actually turns on, six
months later, when the configuration has moved on four times.

**Was anything refused?** Only if you instrumented refusals. Most traces record
the calls that happened, because that is what a trace is for. The tool call your
gateway *declined* is the highest-value event in the whole record — it is either
an attack that was stopped or a permission somebody needs and lacks — and it
leaves no span at all unless something deliberately emitted one.

**Did the export itself fail?** OTLP is fire-and-forget by design, and correctly
so: a collector that is briefly down must not take your application with it. But
"the pipeline dropped some" and "the record is complete" cannot both be true, and
a tier that is allowed to lose data cannot be the tier that proves nothing was
lost.

None of these are hard to fix individually. Together they are the difference
between a pile of retained telemetry and something an auditor will accept, and
the gap between the two is where every one of these projects stalls.

## What the second tier actually needs

Five properties, and each of them is a design decision rather than a storage
decision.

**Completeness, enforced.** Not "we keep everything" but "a request whose record
cannot be written is refused." That inverts the usual availability trade
deliberately, and it is the only version of completeness you can put in front of
somebody, because the alternative is a floor rather than a total.

**Integrity that does not depend on you.** Hash-chain the entries so alteration,
deletion and reordering are detectable by the recipient, without trusting the
person who handed over the file. State the limit too: an intact chain does not
prove nothing was removed from the *end*, because a truncated prefix is itself
an intact chain. Closing that needs an anchor held by somebody else.

**The rules, not just their name.** Fingerprint the decision-affecting
configuration, stamp the digest on every entry, and keep the document the digest
names. A hash whose document nobody kept is a citation to a missing source: it
proves a change happened and leaves you unable to say what changed.

**Who authorised them.** A configuration cannot tell you who agreed to it. If a
model roster or a tool grant can be edited by anyone who can write a JSON file,
the record shows what the rules became without showing that anybody signed off —
and change control that covers application code routinely does not cover the
config that decides what a model is allowed to do.

**Retention you chose.** Six months, six years, whatever your longest obligation
is. Not whatever the query engine can afford.

## Two tiers, one instrumentation point

The mistake would be to treat this as a competition with the observability
platform. It is not: the two answer different questions, and you want both
answered.

| | Observability | Evidence |
|---|---|---|
| Question | what is happening | what happened, and can you prove it |
| Completeness | sampled, by design | every request, or the request is refused |
| Retention | 60 days | your obligation |
| Integrity | a vendor record | verifiable without the vendor |
| Latency | sub-second | slow is fine |
| Content | never leaves | redacted, under a retention policy |

The thing that makes this practical rather than doubling everyone's work is
that a gateway is already in the request path. One place sees every call, so one
place can write the chained record *and* emit the wide event, and no application
needs an SDK, a wrapper, or a change.

The event is the record minus content:

```
gen_ai.request.model            claude-sonnet
gen_ai.response.model           anthropic.claude-sonnet-4-6-20260501-v1:0
switchboard.backend             bedrock
switchboard.team                search
gen_ai.usage.input_tokens       1204
gen_ai.usage.output_tokens      388
gen_ai.usage.cache_read_tokens  11402
switchboard.tools.offered       [search_tickets, wire_transfer]
switchboard.tools.refused       1
switchboard.policy              4f4c581392f8
switchboard.audit.id            01JB2K…
switchboard.audit.recorded      true
```

Two of those fields are the join, and they are what make the tiers one system
rather than two piles.

`switchboard.policy` is the fingerprint of the rules that request was judged
under. The event will expire; the fingerprint resolves against an archived,
self-verifying copy of the configuration long after the event is gone. Given a
trace from four months ago, you can print the exact rules behind it and check
that the document hashes to the name the entry cited.

`switchboard.audit.recorded` is `false` when the completion did not reach the
log. That is the gap the evidence tier exists to close, and it belongs in the
tool people watch every day rather than only in the one they open during an
audit. Alert on it.

Two more worth stealing whatever you build this on. **Cached tokens go in their
own fields** — they bill at roughly a tenth of the input rate, so a cost chart
that folds them into input is wrong by close to an order of magnitude for
exactly the deployments with a large stable system prompt. And **the refusal
count is its own attribute**, not something to derive by unpacking nested
events, because that is the field somebody builds an alert on and a computed one
never gets built.

## What never leaves

Prompts, completions and tool call arguments do not go on the span. Not by
configuration — in the implementation below there is no field on the exported
shapes able to carry them, and a test asserts that no future edit adds one.

The reason is the chokepoint. Content is redacted on its way into the record,
and telemetry goes to a collector you do not control and which has no redaction
step of its own. A span carrying an argument to `transfer_funds` routes around
the one control the whole design depends on. Tool *names* export, because a name
is metadata and is the finding. Arguments are content, and there should be
nowhere to put them.

## Go and check the sampler

If you take one thing from this: find out whether your long-retention export
sits upstream or downstream of your sampler. It is a five-minute check, the
answer determines whether you have an archive or a souvenir, and nothing in the
running system will ever tell you.

Then, if you want to see the second tier written out — chained records, a policy
fingerprint resolvable to the archived configuration, signed approval of
configuration changes, and OTLP emission into whatever you already run — the
code is at
[github.com/Grace/switchboard](https://github.com/Grace/switchboard), with the
two-tier write-up in
[docs/honeycomb.md](https://github.com/Grace/switchboard/blob/main/docs/honeycomb.md).

I would genuinely like to hear from anyone who has finished the data lake
version, and particularly what it cost to keep it complete —
**grace@gracefulco.de**.
