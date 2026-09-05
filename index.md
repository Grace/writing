---
title: "Grace — AI governance and compliance engineering"
description: "Primary-source research and control mappings for systems that call large language models: SOC 2 evidence, EU AI Act obligations, and what an adverse action notice actually leaks."
permalink: /
---

# Writing

Control mappings, disclosure measurement, and evidence tooling for systems that
call large language models. Mostly primary-source: the regulation, the criteria
document, and code you can run.

## Compliance and evidence

**[AI Controls for SOC 2 Type II](/ai-controls-soc2/)**
Twenty-five controls mapped to the 2017 Trust Services Criteria, each with a
test procedure written for a period rather than a moment. There is no AI
criterion in the TSC, so somebody has to write the mapping, and your auditor
expects it to be you.

**[You cannot filter your way to compliance](/structural-controls/)**
What the EU AI Act actually asks of an LLM deployment, and why the control most
vendors sell you is the one least able to deliver it.

**[Your gateway is why the Bedrock bill has one line](/bedrock-attribution/)**
Amazon solved per-team attribution for callers that reach Bedrock directly. A
gateway is how you give that up — and the only thing that can give it back.

## Disclosure and extraction

**[Settability and Observability](/settability-and-observability/)**
Whether explanation mandates and model-extraction risk really conflict. Five
measurements say the cost depends on whether the requester can *set* the
quantities an explanation names — and that Regulation B had already drawn the
line.

**[What does an explanation actually leak?](/explanation-disclosure/)**
The measurements in full, including the two that came out negative and the one
I later had to retract.

**[Auditing Form C-1](/form-c1-audit/)**
The disclosure rule applied to the adverse action notice the CFPB publishes,
row by row.

## Systems

**[Decision-layer incidents](/decision-layer-incidents/)**
Five classes of supply-chain incident involving automated decision systems that
conventional taxonomies have no row for. The claim is not that they are common
— it is that we would not know if they were.

---

Code: [github.com/Grace](https://github.com/Grace) — `switchboard` (LLM gateway
and control assessor), `warden` (disclosure contracts and linter), `paladin`
(instruction-file integrity).

Corrections welcome, and several of these have needed them.
