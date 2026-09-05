---
title: "The strongest test in your AI controls checklist doesn't run"
description: "Reconcile logged request counts against provider invoices, month by month. It is the best test in every AI control mapping, including the one I wrote. AWS billing data contains no request counts."
permalink: /invoice-reconciliation/
---

# The strongest test in your AI controls checklist doesn't run

*Reconcile logged request counts against provider invoices, month by month. It
is the best test in every AI control mapping — including the one I wrote. AWS
billing data contains no request counts.*

Almost every control in a SOC 2 assessment is tested against evidence you
produce. Your log, your configuration, your policy document, your screenshot.
An auditor is not being rude when they discount that; they are doing the job. A
control evidenced only by the thing it controls has no independent witness.

Which is why one test in an AI control mapping is worth more than the rest of
them together: **reconcile what your logs say you sent against what your
provider charged you for.** The invoice is the only record of those events
that nobody in your organisation can edit. Agreement is evidence rather than
assertion. A discrepancy is either traffic your log never saw or usage your
provider never billed, and both are findings.

I put it in my own control mapping as the single strongest test in the
document. Then I sat down to build the tool that runs it, and it doesn't run.

## What AWS actually says

From Amazon's own documentation on [reading Bedrock data in the Cost and Usage
Report](https://docs.aws.amazon.com/bedrock/latest/userguide/cost-mgmt-understanding-cur-data.html):

> **CUR does not contain per-request line items.** Both classic CUR and CUR 2.0
> aggregate Amazon Bedrock cost by usage type, operation, and pricing/resource
> over an hour or a day; neither carries a per-`requestId` identifier.

There is no request count on the bill. There is no request *anything* on the
bill. Usage arrives aggregated by usage type over an hour or a day, and the
identifier that would let you count requests was never in the export.

This is not an AWS deficiency and there is no setting that changes it. Billing
data is billing data — it exists to be summed, not to be joined to your
application traces. But it means a test written against request counts cannot
be run at all, and a control mapping that specifies one is asking practitioners
to produce a number that does not exist.

The failure mode is quietly bad. Nobody reports "this test is unrunnable." They
approximate — divide tokens by an average, or count the invoice's line items,
or reconcile something adjacent and call it done — and the strongest control in
the document becomes the one with the softest evidence behind it.

## What you can reconcile

Tokens, at the model and month grain. AWS says so in the same document, in the
sentence immediately after:

> To reconcile dollars to your invoice while keeping the per-prompt detail in
> logs, **join logs to CUR at the model and usage-type grain.**

That is the test. It is very nearly as strong as the one you wanted — it still
uses a source you do not control, it still catches traffic your gateway never
saw, and it still catches a log that stopped writing. What it gives up is the
ability to say *how many calls*, which was never the interesting part. The
interesting part is *did anything reach the provider that we have no record
of*, and tokens answer that.

But there are three ways to run it and still be wrong, and the first one is
the reason most reconciliations that "balance" don't.

## Trap one: four token types, not two

Providers bill input, output, cache-write and cache-read separately, at unit
prices that differ by close to an order of magnitude. AWS is unusually blunt
about what happens if you forget:

> All four token types must be accounted for when reconciling usage to spend.
> **If you only sum input and output tokens, your totals will not match your
> bill.** This is the most common source of reconciliation gaps, particularly
> for workloads that use prompt caching heavily.

Sit with the last clause. The error is *largest* in deployments with a big
stable system prompt — which is to say, in the deployments most likely to be
worth auditing. And it fails in the direction that looks like a pass: your log
shows fewer tokens than the bill, you shrug at a rounding difference, and the
gap you just waved away was cached traffic rather than a discrepancy.

The corollary matters more than the fix. **If your log does not record cached
tokens separately, this test cannot be run and the honest status is Unknown**,
not Met. And it is not retroactively recoverable. Rates change and can be
reapplied to history; what a provider said it served from cache at 09:14 on a
Tuesday is observable exactly once. A log that didn't capture it has lost it.

## Trap two: do not reconcile currency

The obvious shortcut is to price your logged tokens with a rate card and compare
dollars to dollars. It does not work, and AWS names why:

> Apply the correct rate for each routing type. In-region and cross-region
> inference have different unit prices.

Service tier does the same thing. One model, one name, several unit prices
depending on whether the request went standard, priority or flex, and whether
it was routed cross-region. No rate card you maintain reproduces the invoice,
and the moment your comparison is in dollars you are reconciling your
assumptions against their arithmetic rather than your usage against theirs.

Compare tokens. The invoice remains the authority on what is owed; you are not
auditing the arithmetic, you are auditing whether the same events are on both
sides.

## Trap three: the names don't match, and guessing is worse than stopping

Here is what a Bedrock line item actually looks like:

```
USE1-Claude4.6Sonnet-input-tokens
USE1-Claude4.6Sonnet-cache-read-input-token-count
USE1-Nova2.0Lite-input-tokens-flex
USE1-openai.gpt-oss-120b-mantle-input-tokens-standard
```

The model segment is a billing name. It is not the model id your code sent —
`anthropic.claude-sonnet-4-6-20260501-v1:0` — and it is not whatever you call
that model internally. Three names for one thing, and nothing derives any of
them from the others.

So somebody who knows both has to write the mapping down. The temptation is to
match on resemblance, because `Claude4.6Sonnet` and `claude-sonnet-4-6` are
obviously the same model to a human being. Resist it, and notice why: the cost
of being wrong is asymmetric.

An **unmapped** line produces a question — *what is this and why is it on our
bill?* — which somebody answers in a minute. A **wrongly matched** line
produces a reconciliation that balances between two different models, and
reports a clean month. Nobody re-examines a clean month. The failure that
invents a finding gets corrected; the failure that erases one does not, because
nobody goes looking to disprove good news.

Do the same thing with unmapped lines, incidentally, that you do with a failed
export: report them as *unreconciled*, never as *reconciled to zero*. A model
on the bill that your configuration cannot name is either a mapping nobody
wrote down or a system running in your account that has never touched your
gateway, and only a person can say which.

## What a discrepancy actually means

Both directions are findings, and they are not equally interesting.

**The bill holds tokens your log does not.** Either traffic reached the provider
without passing through your gateway, or entries are missing from the record.
The first is the one to lose sleep over: a route around the gateway is a route
around every control attached to it — the redaction, the tool grants, the rate
limits, the log itself. This is shadow AI observed from outside, and it is the
one thing no amount of reading your own logs will ever produce. Your logs
cannot tell you about the traffic that didn't reach them.

**Your log holds tokens the bill does not.** Rarer, usually less alarming, and
still a disagreement between two accounts of the same month. Often it is a
month your export didn't fully cover, which is why the tooling has to keep
*"asked, and there was none"* apart from *"could not ask"*. A month your export
failed on looks exactly like a month the provider charged nothing for, and only
one of those is a finding about your gateway.

## What the test still cannot tell you

Four things it will report as gaps that aren't, if you let it.

**A month at the edge of your log's coverage.** If the log starts on the 20th,
that month is short by construction. Flag it and exclude it. But allow some
grace: a log covering the whole of January still begins a few minutes after
midnight on the first, and that is a clock rather than a coverage gap.

**A month the export is silent about.** An export that stopped short and a
provider that billed nothing are indistinguishable from the log's side. Report
it once as a question, not as a never-billed finding against every model in it.

**A month the invoice covers and you didn't ask the log about.** Comparing it
reports your own query window as a finding.

**Every month off by the same round factor.** Denomination conventions differ.
A bill drawn in thousands of tokens against a log counting tokens disagrees by
exactly a thousand in every row at once — that is a unit, not a gap, and
reporting it as missing traffic sends somebody hunting an application that does
not exist. One month off by a thousand is a finding. All of them at once is a
convention.

Every one of those, reported as a discrepancy, teaches the reader to skip the
section that holds the real ones.

## While you're here: your logs probably can't evidence a model change either

A related discovery from the same work, since it lands in the same place.

The standard advice for detecting a provider silently repointing a model is to
record the identifier the provider *returns* rather than the one you sent —
the response carries a model field, and for an alias it resolves to a dated
snapshot. That is true of the Anthropic and OpenAI APIs.

**Bedrock's inference responses do not report which model served the request.**
So on the platform most enterprise AI runs on, the strongest record available
is the identifier you *sent*, which evidences your own routing and is not an
attestation from anybody.

That is a real limitation and the honest thing is to write it down. The
tempting thing is to fill the field from your own configuration and let it read
as the provider's answer, and that is strictly worse than leaving it blank: it
converts a known gap into a false attestation, and nobody audits good news.

## The corrected control

If you have my control mapping, this is E1's test as it should read:

> **Test over a period.** Reconcile logged **token** counts against provider
> billing records, month by month, at the model grain. A discrepancy is either
> unlogged traffic or unbilled usage, and both are findings. Count all four
> token types — input, output, cache-write and cache-read — because they bill
> separately and summing only the first two understates any deployment using a
> prompt cache. If your log does not split cached tokens out, this test cannot
> be run and the honest status is *Unknown*.

The mapping itself is at [AI Controls for SOC 2 Type
II]({{ '/ai-controls-soc2/' | relative_url }}), corrected.

## Running it

[switchboard](https://github.com/Grace/switchboard) is an LLM gateway I've been
building; `switchboard reconcile` does the version described above. It reads a
CUR export or a four-column CSV, compares at the model and month grain across
all four token types, refuses to guess a name mapping, and reports the four
non-findings above as what they are rather than as gaps. Where the export
carries a cost allocation tag it also checks whether the bill splits by team the
way the gateway says it should, which is the only way to find out that your
session tags were never reaching the invoice.

There is a `scripts/aws-invoice.sh` that pulls the export — all GETs, and it
keeps *"asked and there was none"* apart from *"could not ask"*, because that
distinction is the whole difference between a finding and a hole. Cost Explorer
charges a cent per request, so a year of months costs twelve cents. Full
write-up in
[docs/reconciliation.md](https://github.com/Grace/switchboard/blob/main/docs/reconciliation.md).

## Why this one is worth the trouble

Every other control in an AI assessment is you marking your own homework. This
is the one where somebody else already wrote down what happened, has no reason
to flatter you, and will hand you the document for free.

It is also the one nobody runs, and the reason is mundane: getting the numbers
out is a morning's work and the test as commonly written doesn't survive contact
with the export. Both of those are fixable. The finding underneath — *is there
traffic reaching our provider that our gateway never saw* — is not one you can
reach any other way.

If you've run this reconciliation against a real bill and found something, or
found that it balanced, I'd genuinely like to hear which —
**grace@gracefulco.de**.
