---
title: "What does an explanation actually leak?"
description: "Five reproducible measurements on the assumed conflict between explanation mandates and model extraction. Two negative results, and one positive result narrower than the question."
permalink: /explanation-disclosure/
---

# What does an explanation actually leak?

*Five measurements on the assumed conflict between explanation mandates and
model-extraction risk. Two negative results, and one positive result that is
narrower than the question but holds.*

**Status: preliminary.** Reproducible harnesses, small synthetic targets, one
author. Every number below comes from code in
[warden/experiment](https://github.com/Grace/warden). Read the limitations
before citing anything.

---

## The assumed conflict

Regulation increasingly compels explanation of automated adverse decisions.
ECOA and Regulation B require creditors to give specific, accurate principal
reasons for a denial, and CFPB Circulars 2022-03 and 2023-03 make clear this
binds regardless of model complexity — the Bureau's position is that a creditor
unable to identify the specific reasons for an AI's decision *cannot lawfully
use that model* for adverse credit decisions. EU AI Act Article 86 gives
affected persons a right to clear and meaningful explanations of decisions made
on the basis of Annex III high-risk systems.

Meanwhile a security literature shows explanations leak the model.
Counterfactual explanations have been used to build high-fidelity surrogates in
roughly 500 queries; the proposed mitigation is differential privacy applied to
the explanation generator.

Stated together these produce an apparent bind: the law requires the disclosure
that security says is attack surface, and the published defence works by
degrading the disclosure. I set out to measure the tradeoff, expecting to
propose a resolution. The measurements did not support the resolution, and two
of them did not support the premise either.

---

## Experiment 1 — a linear scorecard

Six-feature linear model with a decision threshold. Compliance is measured as
the share of denials whose disclosed top-3 principal reasons match the true top
three, which is Reg B's obligation stated as a number. Extraction is surrogate
agreement with the target on held-out applicants after 600 queries.

| Policy | Compliance | Extraction |
|---|---|---|
| Uniform full disclosure | 100% | 100% |
| DP-noised (ε 0.5) | 62% | 100% |
| DP-noised (ε 1.5) | 29% | 99% |
| Outcome only | 0% | 93% |
| Entitlement-scoped | 100% | 100% |

**Two findings.**

*Differential privacy on explanations is a bad trade here.* At ε 1.5 it
destroys 71 points of compliance and removes one point of extraction
resistance. The mechanism is unremarkable: independent per-query noise averages
out, so an adversary with a few hundred samples recovers the clean signal.
Noise added without a composed privacy budget is not a defence, it is a tax on
the notice.

*Explanations were not the binding leak.* Outcome-only disclosure still yields
93% agreement. For a model this simple the decision boundary gives it up
regardless of what the notice says.

I also tested entitlement-scoped disclosure — a faithful notice to the
authenticated subject, outcome only to bulk requesters — on the theory that the
parties the law entitles to explanations cannot mount the attack. Ten
manufactured identities fully reconstructed the model. Ten is not a security
parameter.

## Experiment 2 — rule lists, sweeping complexity

The obvious rescue is that six parameters is too few. So: knockout rule lists
over twelve features with thresholds on a 0–4096 scale, which is what
underwriting actually looks like. The adversary must recover every rule's
feature *and* threshold. Cost is queries, median of 25 random rule lists.

| Rules | Full | Reason named | Outcome only | Ratio |
|---|---|---|---|---|
| 1 | 16 | 29 | 30 | 1.9× |
| 2 | 27 | 53 | 55 | 2.0× |
| 4 | 54 | 106 | 110 | 2.0× |
| 8 | 111 | 215 | 223 | 2.0× |
| 12 | 169 | 325 | 337 | 2.0× |

**The ratio is flat.** Extraction cost grows linearly in rule count under every
policy, and disclosure buys a constant factor of two — never more.

The `reason` and `outcome` columns are nearly identical, and that is the
mechanistic explanation. An adversary who controls the application inputs can
vary one feature at a time and identify the responsible dimension themselves.
Naming it in the notice tells them what they already know. All disclosure
actually saves is the threshold bisection: log₂(4096) ≈ 12 queries per rule,
which is the 2×.

## Experiment 3 — a continuous model with counterfactuals

Rule lists have axis-aligned boundaries an adversary can probe coordinatewise,
which may be what makes disclosure cheap. So: an eight-dimensional tanh network
with twelve hidden units, and the strongest disclosure there is — a
counterfactual, which is a point *on the decision boundary*, handed over on
request. Surrogate has matched architecture.

| Queries | Outcome only | Counterfactual | Gain |
|---|---|---|---|
| 50 | 81% | 83% | +1.8 pp |
| 100 | 86% | 86% | −0.5 pp |
| 250 | 88% | 88% | −0.1 pp |
| 500 | 90% | 91% | +1.3 pp |
| 1000 | 92% | 94% | +1.4 pp |
| 2000 | 93% | 96% | +3.0 pp |

Counterfactual disclosure is worth between zero and three points. The gain
grows with budget rather than shrinking, which is the right direction for the
literature's claim, but the magnitude is nothing like an order of magnitude.

---

## Experiment 4 — varying what the adversary can reach

Every experiment so far let the adversary set every input. Real decisions do
not work that way: bureau scores, internal risk tiers, fraud model outputs and
computed margins are supplied by something outside the requester's reach, and
usually not shown to them either.

So: eight knockout rules over twelve features, sweeping the fraction of
features that are attested rather than submitted. Two metrics, because they
come apart — *recovered* means the adversary knows the rule's feature and its
threshold; *discovered* means they know the rule exists at all.

| Attested | full: rec / disc | reason: rec / disc | outcome: rec / disc |
|---|---|---|---|
| 0% | 100% / 100% | 100% / 100% | 100% / 100% |
| 25% | 88% / 88% | 59% / 87% | 44% / 44% |
| 50% | 81% / 81% | 34% / 78% | 22% / 22% |
| 75% | 64% / 64% | 12% / 48% | 9% / 9% |

**At 0% attested the policies are indistinguishable**, reproducing experiments
1–3 exactly: an adversary who controls every input recovers the system whatever
the notice says.

**At 75% attested, full disclosure gives up seven times as much as outcome
only** — 64% against 9%. A threshold on a quantity the requester cannot set is
not findable by search at any budget, and a notice that prints it hands it over
in a single query. That is the whole of what disclosure costs, and it is
invisible in any experiment where the adversary controls everything, which is
every experiment above and most of the ones in the literature.

The `reason` column separates the two metrics. Naming a feature without its
threshold still discovers 48% of rules at 75% attestation, against outcome
only's 9%. Knowing that an internal tier exists is actionable even without its
value: it names the quantity to go and influence by other means.

## Experiment 5 — observable but not settable

Experiment 4 assumed the requester neither sets nor sees the attested values.
That is right for an internal risk tier and wrong for a bureau score, which an
applicant can pull themselves. An adversary who *observes* an attested value
cannot binary-search it, but they can wait: submit repeatedly, record what
arrived with each application, and bracket the threshold between the highest
approving observation and the lowest declining one.

Fifty percent attested, eight rules, threshold counted recovered within 1% of
scale.

| Budget | HIDDEN: full / reason / outcome | OBSERVED: full / reason / outcome |
|---|---|---|
| 200 | 76% / 14% / 14% | 76% / 19% / 14% |
| 1,000 | 76% / 14% / 14% | 76% / 20% / 14% |
| 5,000 | 76% / 14% / 14% | 76% / 23% / 14% |
| 20,000 | 76% / 14% / 14% | 76% / 25% / 14% |
| 100,000 | 76% / 14% / 14% | 76% / 25% / 14% |

**Observability alone buys nothing.** Outcome-only sits at 14% at every budget
including 100,000 queries. The adversary sees the score on every application
and cannot use it, because a decline does not say which attested rule fired.
Every observation has to be charged to all of them, and the brackets poison
each other.

**Observability plus a reason code buys eleven points**, 14% to 25%, plateauing
by twenty thousand queries. The reason code is what makes an observation
attributable, and attribution is what makes it usable.

Full disclosure is 76% at every budget under both conditions. Printing a
threshold costs the same whether or not the requester could ever have found it.

---

## What survived

The claim is narrow, and now it is measured rather than anecdotal.

> Disclosure does not meaningfully protect dimensions an adversary can
> manipulate — they find those regardless, and the notice saves them a constant
> factor of about two. What disclosure protects is dimensions the adversary
> **cannot reach through the inputs**: derived quantities, internal risk tiers,
> margin floors, bureau scores, thresholds on facts the submitter does not
> control.
>
> For those, the notice is the only leak. Withholding it is not a slowdown; it
> is the difference between known and unknown, and the gap runs from 1× to 7×
> as inputs move out of the requester's reach.

This first appeared as an accident in experiment 1: a rule keyed on carrier
margin — computed from declared value and destination — was unrecoverable under
every policy, because probing it tripped the value ceiling first. Yet raw
disclosure printed it in the notice as a labelled fact. Experiment 4 is that
observation turned into a controlled sweep.

It is also actionable, which the original thesis was not. The audit question is
not "do you explain your decisions" but:

> **1. Does your notice print a threshold on a quantity the requester cannot
> set?** That value is unreachable by search at any budget, so the notice is
> the only way they get it.
>
> **2. If that quantity is observable to them, does your notice name the rule?**
> Naming it is what converts their passive observations into an attributable
> search. A reason code is safe when the underlying quantity is hidden and
> unsafe when it is visible — which is not a distinction anyone currently draws.

Both are answerable from a decision contract without touching the model, and
the second one is the reason a disclosure control needs to scope rules and
facts separately rather than having a single verbosity dial.


One positive finding, from the first experiment and robust across all three.

## What did not survive

**The reconciliation framework.** I proposed that entitlement-scoped disclosure
resolves the compliance/security conflict, because the legally entitled party
is structurally incapable of the attack. The measurements say the conflict is
not big enough to need resolving: disclosure is a constant-factor advantage,
and where it is cheap to extract without explanations, scoping them changes
nothing.

**The premise, partly.** Where the adversary controls the inputs, explanations
are not the dominant leak — the decision itself is. The premise holds only in
proportion to how much of the input is out of their hands, which is a condition
neither the regulation nor the security literature currently states.

---

## Limitations

These matter more than the results.

**All three targets are low-dimensional** — six to twelve features. The
extraction literature's strong results are in higher dimensions where random
sampling covers the space far less efficiently, which is precisely the
condition that makes a boundary point valuable.

**The surrogate has matched capacity and, in experiment 3, matched
architecture.** A real adversary usually does not know the model family. That
assumption helps the outcome-only adversary considerably and is the most likely
reason my counterfactual gain is small.

**The adversary has full control of every input.** True for a credit applicant
submitting an application, false where inputs are partly observed or attested.
Where the adversary controls less, disclosure should matter more — this is the
same mechanism as the margin-floor finding, generalised.

**Synthetic targets, no real model, no real notices.** Nothing here has touched
a production system.

Any of these could be what separates my numbers from the published ones. The
honest summary is not "the literature is wrong" but **"disclosure risk appears
to be a property of dimensionality, adversary input control, and architectural
uncertainty, rather than of explanation as such"** — and that is a testable
claim I have not yet tested.

## What I would test next

Input control was the third item on this list and became experiment 4. What
remains:

1. **Sweep dimensionality directly.** If the counterfactual gain grows with
   dimension, that locates the regime and reconciles experiment 3 with the
   published extraction results.
2. **Remove the architectural prior** — a surrogate from a different family
   than the target, which is the realistic case and the one most likely to be
   suppressing the gap in experiment 3.
Observability became experiment 5. What is left after that:

3. **Attribution under partial disclosure.** Experiment 5 shows attribution is
   the binding constraint, not observation. A notice that names a *category* of
   rule rather than a specific one should sit between reason and outcome, and
   where it sits is worth knowing — it is the design space a real contract
   occupies.
4. **Real notices.** Everything here is synthetic. The obvious next step is to
   take a published adverse-action notice template and ask which of its fields
   name quantities the applicant cannot set.

---

*Harnesses: `warden/experiment/{oracle,disclosure,scaling,continuous,attested,observable}`. Go,
standard library only, seeded. Corrections welcome —
**grace@gracefulco.de**.*
