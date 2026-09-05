---
title: "AI Controls for SOC 2 Type II"
description: "Thirty-two controls for systems that call large language models, mapped to the 2017 Trust Services Criteria, each with a test procedure written for a period rather than a moment."
permalink: /ai-controls-soc2/
---

# AI Controls for SOC 2 Type II

**A control mapping for systems that call large language models.**
Thirty-two controls against the 2017 Trust Services Criteria, each with a
test procedure written for a period rather than a point in time.

---

## How to use this

There is no AI criterion in the Trust Services Criteria. The operative standard
is the 2017 TSC with the 2022 revised Points of Focus, and practitioners are
folding AI systems into the criteria that already exist — principally CC3
(risk assessment), CC6 (access), CC7 (monitoring) and CC9 (vendor management).

That means **somebody has to write the mapping, and your auditor is expecting
you to be that somebody.** This document is that mapping. Each control below
names the criterion it satisfies, why it maps there, the evidence that
satisfies it, and how to test it across a period.

Mappings are defensible, not authoritative. Your auditor may place a control
under a different criterion; that is a conversation, not an error. What matters
is that every control has *a* home and none of your AI system is unmapped.

**Criterion text verified against the AICPA source.** Every CC3, CC4, CC5, CC6
and CC7 criterion cited here was checked against *2017 Trust Services Criteria
(With Revised Points of Focus — 2022)*, TSP Section 100. Where a criterion's
published points of focus directly support a mapping, this document says so —
CC3.4's *"Assesses Changes in Vendor and Business Partner Relationships"* is
why B3 sits there, and CC7.1's *"Implements Change-Detection Mechanisms"* is
why E2 has a second home.

### The four states

Most control checklists have two states. That is the wrong number, and it is
why they fail in a Type II.

| State | Meaning |
|---|---|
| **Met** | Evidenced across the whole period |
| **Partial** | Evidenced, with a gap inside the period |
| **Not addressed** | The control is absent, deliberately or otherwise |
| **Unknown** | **The source cannot tell you** |

*Unknown* is not a soft failure. It is the honest answer when your evidence
source is silent — a provider console that shows current configuration but not
history, a log that starts in June for a period beginning in January. Recording
it as "no" understates you; recording it as "yes" is the thing that gets found.

### Period, not moment

A configuration screen shows today. A Type II opinion covers a period, and a
control that ran for eleven months of twelve **failed the month it did not**.

Every test procedure below asks the same underlying question: *can you
demonstrate this was true for the entire period, and if not, when was it not?*
Where you cannot answer that, the status is Partial and the exception belongs
in your description of the system — written by you, before the auditor finds
it.

---

## A. Governance and risk assessment

### A1 — AI systems are inventoried
**CC3.2** · *Identifies risks to the achievement of objectives*

An inventory of every system that sends data to, or acts on output from, a
language model. Each entry: owner, model provider and version, data
classification of inputs, whether output reaches a customer or takes an action.

**Evidence.** The inventory itself, dated, with change history.

**Test over a period.** Compare the inventory at period start and period end
against provider billing records. A model endpoint that appears in an invoice
and not in the inventory is the finding. *This is the test auditors run first
and the one most teams fail*, because shadow usage accrues through developer
API keys.

> **Honest limit.** An inventory is a statement about what you know about. It
> does not detect a team calling a provider on a personal card. Note also that
> the bill names models the way billing names them, not the way your inventory
> does — see E1 — so this comparison needs that mapping before it produces
> either a finding or a clean result you can rely on.

### A2 — AI-specific risks are assessed, and reassessed on change
**CC3.2, CC3.4** · *Identifies and assesses changes that could significantly impact internal control*

A documented risk assessment covering at minimum: prompt injection, sensitive
data in prompts, output used without review, provider outage, provider model
change, and cost runaway.

**Evidence.** The assessment, with review dates.

**Test over a period.** For each AI system added or materially changed during
the period, show the risk assessment was revisited. An assessment dated once,
at the start, with three new systems launched since, is Partial.

### A3 — Accountability for each AI system is assigned to a named person
**CC1.3** · *Establishes structures, reporting lines, authorities and responsibilities*

**Evidence.** Inventory owner field, tied to a real employee, not a team alias.

**Test over a period.** Sample three systems. For each, confirm the owner of
record was employed and in that role for the whole period, or that reassignment
was recorded.

---

## B. Model provider and vendor management

### B1 — Each model provider is assessed before use and periodically after
**CC9.2** · *Assesses and manages risks associated with vendors and business partners*

**Evidence.** A completed vendor assessment per provider, covering security
posture, subprocessor list, incident history, and the terms in B2.

**Test over a period.** Every provider in the A1 inventory has a current
assessment. One added mid-period without one is an exception dated to the day
it was added, not to the day you noticed.

### B2 — Provider data-handling terms are documented and match your commitments
**CC9.2, C1.1** · *Identifies and maintains confidential information*

Specifically: whether inputs are used for training, how long the provider
retains them, where they are processed, and whether a zero-retention or
enterprise tier is in force.

**Evidence.** The executed agreement or the console setting, captured with a
date. Not the marketing page.

**Test over a period.** Confirm the tier in force for the *whole* period.
Providers change defaults, and an account that moved onto a retention-free tier
in August was not on one in March.

> **Honest limit.** This is a contractual control. It evidences what the
> provider agreed to, never what the provider did.

### B3 — Provider model version changes are detected
**CC3.4, CC9.2**

A model identifier that silently repoints — an alias like `-latest`, or a
provider-side update to a pinned name — changes your system's behaviour with no
change on your side.

**The obvious test does not work, and this is worth being precise about.**
Comparing the distinct model names in your logs catches a *caller* adopting a
new model. It does not catch a provider repointing a name you already use,
because the name is unchanged — and most gateways record the identifier the
caller *asked for*, not the one the provider *resolved to*. Against a silent
repoint, a comparison of requested names is blind by construction.

Three mechanisms, in ascending strength:

1. **Record the identifier the provider returns**, not the one you sent. On the
   Anthropic and OpenAI APIs the response carries a model field, and for an
   alias it resolves to a dated snapshot. **Not every platform offers one:**
   Amazon Bedrock's inference responses do not report which model served the
   request, so the strongest record available there is the identifier *you
   sent* — which evidences your own routing and is not an attestation from the
   provider. Do not populate the field from your own configuration and present
   it as the provider's answer; that converts a gap into a false attestation,
   and a false attestation is worse than a blank because nobody goes looking to
   disprove good news. If your log stores only the name the caller requested,
   this control cannot be evidenced at all and the honest status is *Unknown*.
2. **Record the provider's build fingerprint** where one is offered — some APIs
   expose a field for exactly this purpose — so a backend change under a stable
   snapshot name is still visible.
3. **Run a behavioural canary.** A fixed prompt set on a schedule, outputs
   hashed, drift alerted. This is the only mechanism that catches a repoint
   reporting an unchanged name *and* an unchanged fingerprint.

**Evidence.** Log schema showing the resolved identifier is captured, plus the
review comparing resolved versions observed against versions approved.

**Test over a period.** Extract the set of distinct *resolved* identifiers seen
across the period and reconcile against the inventory. Then confirm your log
schema recorded the resolved value for the whole period — a field added in
month seven leaves months one through six unevidenced.

> **Honest limit.** Mechanisms 1 and 2 depend on the provider telling you the
> truth about what served your request. They are attestations, not
> measurements. Only the canary observes behaviour directly.

### B4 — AI subprocessors are disclosed downstream
**CC9.2**

If you process customer data through a model provider, that provider is a
subprocessor and your customer agreements likely require disclosure.

**Evidence.** Public subprocessor list including model providers, with the date
each was added, and evidence of customer notification where contracts require it.

**Test over a period.** Each provider's addition date on the list is on or
before its first appearance in the logs.

---

## C. Access control

### C1 — Callers are authenticated before reaching a model
**CC6.1, CC6.2** · *Logical access security measures; registration and authorization of users*

**Evidence.** Identity provider configuration, or the credential roster if
authentication is by shared key.

**Test over a period.** Pull the authentication configuration at three points
across the period. Shared static keys are **Partial at best** — they name a
team, not a person, and revocation is a manual edit.

### C2 — Unauthenticated requests are denied
**CC6.1**

**Evidence.** Configuration showing the default-deny posture, plus a negative
test: an unauthenticated request receives a refusal.

**Test over a period.** Search the logs for any served request with no
associated principal. One is an exception.

### C3 — Provider credentials are scoped, not pooled
**CC6.3** · *Least privilege*

Where every call reaches the provider under one shared identity, the provider's
own logs and bill cannot distinguish your callers — and neither can an
investigation.

**Evidence.** Credential configuration showing per-team or per-caller scoping.

**Test over a period.** Confirm the provider-side identity count matches the
caller count. One shared credential across twelve teams is Partial.

### C4 — Records attribute actions to a person
**CC6.1, CC7.2**

**Evidence.** Sample log entries showing a subject identifier resolvable to an
individual.

**Test over a period.** Sample twenty entries spread across the period. Entries
carrying only a team or service name are Partial; this is the most common
finding in this section and the hardest to remediate retroactively.

---

## D. Data protection

### D1 — Sensitive data is not written to prompt and completion logs
**CC6.7, C1.1** · *Restricts transmission and movement of information*

**Evidence.** Redaction rule set, plus sampled log entries demonstrating
redaction applied.

**Test over a period.** Sample entries from the earliest and latest weeks of
the period. Rules added in month six do not cover months one through five, and
that is an exception with a knowable start date.

> **Honest limit.** Pattern-based redaction catches structured identifiers. It
> does not catch a name, or a medical condition described in prose. State this
> in your system description rather than letting an auditor infer coverage you
> do not have.

### D2 — Redaction is applied at a chokepoint, not per caller
**CC6.7, CC5.2** · *Control activities over technology*

A rule each application opts into is a convention. A rule applied where no call
site can skip it is a control.

**Evidence.** Architecture showing redaction in the request path, not in a
client library.

**Test over a period.** Identify every code path that reaches a provider.
Any that bypasses the chokepoint is a finding regardless of whether it was used.

### D3 — Encryption in transit
**CC6.7**

**Evidence.** TLS configuration on every listener, including internal hops.

**Test over a period.** Certificate validity dates must cover the period with
no gap. An expired-and-renewed certificate is a gap unless renewal was
seamless, and the renewal timestamp proves which.

### D4 — Prompt and completion data is disposed of on schedule
**C1.2** · *Disposes of confidential information*

**Evidence.** Retention configuration and evidence of actual deletion — not the
policy, the deletion.

**Test over a period.** Query for records older than the stated retention. Any
result is a finding. *Retention policies that were never implemented are common
and trivially detected.*

---

## E. Logging and accountability

### E1 — Every model interaction is recorded
**CC7.2** · *Monitors system components for anomalies*

Minimum fields: timestamp, principal, model identifier and version, token
counts, latency, outcome, and any tools offered or called.

**Evidence.** Log schema and sampled entries.

**Test over a period.** Reconcile logged **token** counts against provider
billing records, month by month, at the model grain. **A discrepancy is either
unlogged traffic or unbilled usage, and both are findings.** This is the single
strongest test in this document because it uses a source you do not control.

**Tokens, not requests.** Billing records do not carry request counts. AWS
states it plainly: *"CUR does not contain per-request line items ... neither
carries a per-`requestId` identifier"* — usage is aggregated by usage type over
an hour or a day. Any request count derived from a bill is inferred rather than
observed, so a test written against request counts cannot be run at all.

**Count all four token types.** Providers bill input, output, cache-write and
cache-read separately, at unit prices differing by close to an order of
magnitude. AWS names this as the most common source of reconciliation gaps: sum
only input and output and your totals will not match. The error grows with
prompt caching, so it is largest in exactly the deployments most likely to have
a stable system prompt — and it fails in the direction that looks like a pass.
If your log does not split cached tokens out, this test cannot be run and the
honest status is *Unknown*.

**Do not reconcile currency.** One model bills at different unit prices by
service tier and by cross-region routing, so no rate card of yours reproduces
the invoice. Compare the usage behind it.

> **Honest limit.** The bill names models the way billing names them, not the
> way your API calls do — AWS bills `Claude4.6Sonnet` for the model invoked as
> `anthropic.claude-sonnet-4-6-...`. Someone who knows both has to write that
> mapping down. Do not infer it from resemblance: an unmapped line is a question
> you can answer, and a wrongly matched one reconciles two different models
> against each other and reports a clean month that nobody will re-examine.

### E2 — Records are protected from modification
**CC7.2, CC7.1** · *CC7.1's points of focus name change-detection mechanisms —
"file integrity monitoring tools to alert personnel to unauthorized
modifications of critical system files, configuration files, or content files."
Log tamper-evidence maps cleanly to either; cite both and let your auditor
choose.*

**Evidence.** The mechanism — hash chaining, object-lock storage, a WORM tier —
named specifically. "Append-only" is not tamper evidence if an administrator
can rewrite the store.

**Test over a period.** Run the integrity verification across the full period,
not a sample. Where the mechanism is object-lock, evidence the retention lock
was in force from period start.

### E3 — Logging cannot fail silently
**CC7.2**

If a completion whose record fails to write is served anyway, your log is a
best-effort artifact and every count derived from it is a floor, not a total.

**Evidence.** Configuration showing fail-closed behaviour, plus a test
demonstrating a request is refused when the log is unwritable.

**Test over a period.** Review error rates for log-write failures. Any period
with failures and no corresponding request refusals is an exception.

### E4 — Logs are reviewed on a defined interval
**CC4.1, CC7.2** · *Evaluations to ascertain whether controls are functioning*

**Evidence.** Review records with dates, reviewer, and findings.

**Test over a period.** Reviews occur at the stated cadence with no gap
exceeding the interval. A record nobody re-reads is a record nobody would
notice being edited.

### E5 — Retention meets the longest applicable obligation
**CC7.2, C1.2**

**This is the control most likely to fail, and the failure is structural.**

Observability platforms retain 15 to 60 days by default. Compliance obligations
run far longer:

| Obligation | Floor |
|---|---|
| EU AI Act Art. 19 (log retention) | 6 months |
| HIPAA §164.316(b)(2) | 6 years |
| FINRA Rule 4511 | 6 years |
| SEC Rule 17a-4 | 3–6 years depending on record |
| Your own customer commitments | Read them |

If your AI logs live only in your APM tool, **the evidence for month one of a
twelve-month period no longer exists**, and no configuration change made today
recovers it.

**Evidence.** Retention configuration at each tier, plus proof that records
from period start are still retrievable.

**Test over a period.** Retrieve a record from the first week of the period.
If you cannot, the control is Unmet for the portion already aged out, and the
exception is dated to the aging boundary, not to today.

---

## F. Change management

### F1 — Model, prompt, and policy changes are authorized before deployment
**CC8.1** · *Authorizes, designs, develops, tests, approves and implements changes*

Prompts and model selections are configuration that changes system behaviour.
They belong under change control whether or not they live in application code.

**Evidence.** Change tickets or pull requests covering prompt and model
changes, with approval.

**Test over a period.** Sample changes to prompt templates and model
identifiers across the period. **Prompt changes made through a console rather
than a repository are the common gap**, because nothing forces them through the
process.

### F2 — The policy in force at the time of a decision is recoverable
**CC8.1, CC7.2**

When a decision is questioned six months later, you need to show what rules,
prompt, and model produced it — not what is deployed now.

**Evidence.** A version identifier or content hash of the effective
configuration, recorded on each interaction, resolvable to the configuration
itself.

**Test over a period.** Take a logged interaction from early in the period and
reconstruct the exact prompt and model settings that produced it. *Most
organisations cannot do this and do not discover it until a dispute.*

---

## G. Model behaviour and agency

### G1 — Actions taken by a model are authorized before they take effect
**CC6.3** · *Least privilege*

Where a model can call tools, each call is checked against the caller's grant
before execution — not logged after.

**Evidence.** Authorization configuration and the grant model.

**Test over a period.** Sample tool calls and confirm each has a corresponding
grant. Confirm refusals exist; a system that has never refused anything has
either perfect callers or no enforcement.

> **Honest limit.** On streaming responses, withholding the completing frame
> stops a client that waits for the finish reason and does not stop one acting
> on partial output. Where this must be a control rather than a signal, do not
> offer tools on streaming requests.

### G2 — Tool calls and refusals are recorded
**CC7.2**

**Evidence.** Log entries naming every tool requested, in order, with refusals
and reasons.

**Test over a period.** Confirm refusals are written before the request fails,
so a stopped call cannot be lost along with it.

### G3 — Per-caller resource limits are enforced
**A1.1**, or **CC5.2** where Availability is out of scope · *Maintains and
monitors capacity; selects and develops general control activities over
technology*

**A1.1 sits in the Availability category, which is optional, and most SOC 2
reports are Security-only.** Mapped there alone, this control has no home in the
majority of engagements — against this document's own rule that every control
needs *a* home. Where Availability is not in scope, cite **CC5.2**: a per-caller
limit is a general control activity over technology whether or not availability
is being reported on. With metered inference it is also a spend control rather
than only a capacity one, which is the reading that survives a Security-only
scope — MITRE ATLAS **AML.M0004** treats limiting model queries as the
denial-of-wallet defence.

**Evidence.** Rate, concurrency and token budget configuration per caller.

**Test over a period.** Identify the highest-consuming caller per month.
Consumption exceeding the stated limit means the limit was not enforced.

### G4 — Each agent is a distinct identity with a named accountable owner
**CC6.2, CC1.3** · *Registers and authorizes before issuing credentials*

An agent acting for employees is not a service account and should not share
one. It needs its own principal, and a human who answers for what it does.

**Evidence.** Agent inventory with one identity per agent and a named owner,
not a team alias.

**Test over a period.** For each agent, pull the thread: who is accountable? If
nobody can be named, that is the finding, and it is a finding regardless of how
good the logs are.

### G5 — The delegation chain is recorded end to end
**CC6.1, CC7.2**

Three separate facts, and a token tied to an automation identity is none of
them: **which agent** acted, **who initiated** it, and **on whose behalf** it
held authority. Collapsing them into one service-account name is the
accountability gap — the employee says the agent acted independently, and the
log cannot contradict them.

**Evidence.** Log schema carrying agent identity, initiating principal, and
subject-on-whose-behalf as distinct fields.

**Test over a period.** Take an agent action from mid-period and name all three.
If the log yields only the automation identity, this is Unmet, and no access
review built on it means anything.

### G6 — Delegated authority is scoped and time-bounded
**CC6.3** · *Least privilege*

Duration is a dimension of privilege and is usually the one nobody sets.
Standing authority granted once and never expiring is the normal state, and it
is the one that makes "duration of access" unanswerable.

**Evidence.** Grant configuration showing scope *and* expiry per agent.

**Test over a period.** For each agent, state when its authority began and
ended. Any grant with no expiry is Partial at best.

> **An agent needs tighter least privilege than a person, not looser.** Broad
> scopes get granted because narrowing them is tedious and slows a rollout —
> and then the log cannot distinguish *the agent did its job* from *the agent
> had access to do something strange*. Scope narrowness is an evidentiary
> property, not only a security one.

### G7 — What a grant was used for is evidenced separately from the grant itself
**CC6.3, CC7.2**

**This is the control most likely to be assumed rather than held.** Grant-layer
telemetry is usually fine: an OAuth grant event records client, scopes,
timestamp and authorizing user, and reconstructs *who authorized what, when*.

It cannot reconstruct what happened next. The same scope granted to a
fixed-purpose application and to an agent is byte-identical in that log — but
the application's behaviour was fixed by the vendor's code, and the agent's is
decided at inference time, turn by turn, from prompts nobody reviewed. **The
delegated task and the action performed do not live in the grant.** They live
in activity records, and they have to be captured and reviewed as a separate
control.

**Evidence.** Activity records tied to the grant that produced them, showing
each action taken under it.

**Test over a period.** Pick a grant. Enumerate every action taken under it
during the period. If you can produce the authorization event but not the
activity, the honest status is Unmet for this control even where G1 and G2 are
Met — those describe the boundary, and this describes what crossed it.

> **Where practitioners disagree, and both positions are defensible.** One
> school holds that accountability is assigned by policy: the employee who owns
> an automation is answerable for everything it does, and reconstruction from
> logs is a convenience rather than the basis of the claim. The other holds
> that accountability which cannot be evidenced will not survive a disputed
> incident. The first is cheaper and is what most policies actually say. The
> second is what an auditor tests. Adopting the first does not relieve you of
> G5 — it changes who the record has to convince.

---

## H. Monitoring and response

### H1 — Anomalous model behaviour is monitored
**CC7.1, CC7.2**

At minimum: error rate, refusal rate, latency distribution, token consumption
per caller, and distinct model versions observed.

**Evidence.** Dashboards or alert definitions, with alert history.

**Test over a period.** Alert definitions existed for the whole period. An
alert added after an incident does not cover the months before it.

### H2 — AI incidents are handled under the incident response process
**CC7.3, CC7.4** · *Evaluates security events; responds to incidents*

Your IR process should name AI-specific scenarios: prompt injection resulting
in data disclosure, sensitive data sent to a provider, harmful output reaching
a customer, provider breach.

**Evidence.** IR plan with AI scenarios, plus records of any incidents.

**Test over a period.** Where AI incidents occurred, the process was followed.
Where none occurred, the plan existed and was reviewed.

### H3 — Adversarial testing is performed and results retained
**CC4.1**

**Evidence.** Red-team or automated probe results, dated. `garak`, `promptfoo`,
PyRIT and Giskard all produce attachable output.

**Test over a period.** Testing occurred at the stated cadence and covered
systems in production during the period, not only the flagship one.

> **Honest note.** Absence of testing is *Unmet*, not *Unknown*. Nothing about
> your configuration is silent on this — either the results exist or they do not.

---

## Scoring sheet

| # | Control | Criterion | Status | Exception dates |
|---|---|---|---|---|
| A1 | AI systems inventoried | CC3.2 | | |
| A2 | Risks assessed and reassessed | CC3.2, CC3.4 | | |
| A3 | Named accountability | CC1.3 | | |
| B1 | Provider assessed | CC9.2 | | |
| B2 | Provider data terms documented | CC9.2, C1.1 | | |
| B3 | Model version changes detected | CC3.4, CC9.2 | | |
| B4 | Subprocessors disclosed | CC9.2 | | |
| C1 | Callers authenticated | CC6.1, CC6.2 | | |
| C2 | Unauthenticated denied | CC6.1 | | |
| C3 | Provider credentials scoped | CC6.3 | | |
| C4 | Records attribute to a person | CC6.1, CC7.2 | | |
| D1 | Sensitive data not logged | CC6.7, C1.1 | | |
| D2 | Redaction at a chokepoint | CC6.7, CC5.2 | | |
| D3 | Encryption in transit | CC6.7 | | |
| D4 | Disposal on schedule | C1.2 | | |
| E1 | Every interaction recorded | CC7.2 | | |
| E2 | Records tamper-evident | CC7.2 | | |
| E3 | Logging fails closed | CC7.2 | | |
| E4 | Logs reviewed | CC4.1, CC7.2 | | |
| E5 | Retention meets obligation | CC7.2, C1.2 | | |
| F1 | Changes authorized | CC8.1 | | |
| F2 | Effective policy recoverable | CC8.1, CC7.2 | | |
| G1 | Model actions authorized | CC6.3 | | |
| G2 | Tool calls recorded | CC7.2 | | |
| G3 | Resource limits enforced | A1.1, or CC5.2 | | |
| G4 | Agent identity with named owner | CC6.2, CC1.3 | | |
| G5 | Delegation chain recorded end to end | CC6.1, CC7.2 | | |
| G6 | Delegated authority scoped and time-bounded | CC6.3 | | |
| G7 | Grant exercise evidenced separately | CC6.3, CC7.2 | | |
| H1 | Anomalous behaviour monitored | CC7.1, CC7.2 | | |
| H2 | AI incidents in the IR process | CC7.3, CC7.4 | | |
| H3 | Adversarial testing performed | CC4.1 | | |

Thirty-two rows; G-section controls are *Not addressed* rather than *Unmet*
where no model in scope can call tools. Say so explicitly — an auditor reading
a blank is entitled to assume the worse of the two.

---

*This mapping is a working document, not authoritative guidance. It reflects
the 2017 Trust Services Criteria as applied to language-model systems in the
absence of an AI-specific criterion. Your auditor's judgment governs.*
