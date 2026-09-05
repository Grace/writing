---
title: "Decision-layer incidents: a proposed taxonomy"
description: "Five classes of supply-chain incident involving automated decision systems that conventional incident taxonomies have no row for, and the instrumentation that would make them visible."
permalink: /decision-layer-incidents/
---

# Decision-layer incidents: a proposed category for supply chain cybersecurity observatories

*Draft taxonomy — Grace, September 2026. Offered for discussion, not published.*

---

## The gap

Supply chain cybersecurity incident taxonomies classify well on the systems
layer: ransomware against a WMS, credential compromise at a 3PL, an OT
intrusion halting a line, third-party breach at a shared provider. These share a
useful property — something was *breached*, so there is a moment to point at.

A growing class of disruption has no such moment. The automated systems that
decide what ships, by which carrier, under which customs classification, and
whether a counterparty is eligible can be manipulated, can leak, and can drift,
using nothing but authorised traffic. No malware, no intrusion, no credential
theft. The physical effect is real — misrouted freight, evaded screening,
halted procurement — but nothing in a conventional incident taxonomy has a row
for it.

Call this the **decision layer**: rules engines, optimisers, and increasingly
LLM agents, sitting between operational data and physical action.

The claim worth testing is not that these incidents are common. It is that we
would not know if they were, because **most of them are structurally
unobservable with the instrumentation operators currently deploy.** An
observatory that catalogues them is the way to find out.

---

## Proposed classes

### D1 — Decision-oracle disclosure

An automated decision system returns reasons specific enough to be inverted.
An actor who can submit requests and read why they failed learns the thresholds
by search — the same shape as a padding oracle, one bit per attempt.

- **Vector:** authorised submissions; no access beyond a normal counterparty's
- **Physical effect:** routing around controls — hazmat classification, customs
  eligibility, sanctions screening, carrier restrictions, credit limits
- **Why it is missed:** every request is well-formed and authorised. There is no
  anomaly to detect, only a volume of ordinary rejections
- **Evidence needed:** per-counterparty rejection-reason logs, retained long
  enough to see a search pattern. Rarely retained at all

### D2 — Instruction-surface compromise

The configuration an automated agent obeys is modified persistently. For LLM
agents this is a literal file — `CLAUDE.md`, `AGENTS.md`, an MCP server
definition. For rules engines it is a ruleset or decision table.

- **Vector:** dependency postinstall, cloned repository, plugin marketplace,
  compromised configuration sync
- **Physical effect:** whatever the agent is authorised to do — procurement,
  scheduling, routing, carrier selection
- **Why it is missed:** nothing in a normal toolchain diffs these files, and a
  human reviewing one cannot see a zero-width space or a homoglyph
- **Evidence needed:** integrity baselines over instruction surfaces. MITRE
  ATLAS names the control `AML.M0031` (Memory Hardening); adoption is near zero

### D3 — Unrecorded policy change

Not adversarial. A ruleset or prompt changes with no durable record of what was
in force when a decision was made, so a later dispute cannot be resolved and a
systematic error cannot be bounded in time.

- **Vector:** ordinary change management, absent decision-level provenance
- **Physical effect:** systematic misrouting or misclassification, discovered
  late and unscopeable
- **Why it is missed:** each decision looks locally correct
- **Evidence needed:** a fingerprint of the decision-affecting policy carried on
  every decision record

### D4 — Unattributed autonomous action

An agent acts and the record does not say on whose behalf, under what policy, or
against which model version. The action may be entirely legitimate; it is the
irreducibility that is the incident.

- **Vector:** shared service credentials; a gateway that calls downstream under
  one identity and erases the caller
- **Physical effect:** disruption that cannot be attributed, scoped, or
  prevented from recurring
- **Why it is missed:** it only surfaces during investigation, by which point
  the evidence was never written
- **Evidence needed:** per-caller identity preserved through the hop, and a
  tamper-evident record

### D5 — Shared decision-provider concentration

Many operators route decisions through the same small set of model or
optimisation providers. A provider's outage, silent version change, or altered
behaviour propagates simultaneously across unrelated supply chains.

- **Vector:** none required — a vendor-side change is sufficient
- **Physical effect:** correlated, cross-operator disruption with no local cause
- **Why it is missed:** each operator sees an isolated anomaly; the correlation
  is only visible in aggregate, which is precisely what an observatory can see
  and an operator cannot
- **Evidence needed:** model and version pinned on decision records, comparable
  across operators

---

## Record fields

For each catalogued incident, beyond an observatory's existing fields:

| Field | Values |
|---|---|
| Decision system | rules engine · optimiser · ML model · LLM agent · hybrid |
| Autonomy | advisory · human-approved · autonomous with review · autonomous |
| Class | D1–D5 |
| Physical operation | warehousing · transport · customs · procurement · manufacturing · last-mile |
| Detection source | operator · counterparty · auditor · regulator · provider · not detected (inferred later) |
| Time to detection | hours · days · weeks · months · unknown |
| **Evidence availability** | sufficient · partial · **absent — reconstructed from secondary sources** |

The last field is the one that makes the dataset a research instrument rather
than a list. If most D1–D5 entries record *absent*, the finding is not "these
incidents are rare" but "our instrumentation cannot see them," which is a
different and more actionable conclusion — and it is testable against the
existing corpus by re-coding known incidents against these classes.

---

## What would make this real

1. **Re-code a sample of existing incidents** against D1–D5. If none fit, the
   category is wrong and that is worth knowing cheaply.
2. **Check the evidence-availability distribution.** A skew toward *absent* is
   the publishable result.
3. **Name the minimum instrumentation** that would move a class from absent to
   partial. For D2 that is a file-integrity baseline; for D3 a policy
   fingerprint on each decision; for D4 caller identity preserved through the
   gateway. All three are cheap, and none are common.

---

## Prior work I can contribute

Two public, reproducible artifacts, both MIT-licensed:

- **[injection-study](https://github.com/Grace/injection-study)** — a
  stdlib-only, deterministic harness measuring whether hash-based detection can
  carry prompt-injection defence. The sensitivity result: similarity collapses
  past a single-word rewrite, and false positives fire at the default
  threshold. Relevant to D2 — it is the argument for integrity controls over
  content detection.
- **[warden](https://github.com/Grace/warden)** — a guard that translates a
  rules-engine decision into a published contract scoped by audience, so a
  decline reason stops being a queryable oracle. Directly a D1 mitigation.

Neither is offered as a product. They are the reason I think these classes are
real, and the measurement behind D1 and D2.
