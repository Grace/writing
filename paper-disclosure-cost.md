---
title: "Settability and Observability: What an Adverse Action Notice Actually Leaks"
description: "Five measurements on whether explanation mandates and model-extraction risk actually conflict. The cost turns out to depend on whether the requester can set the quantities an explanation names."
permalink: /settability-and-observability/
---

# Settability and Observability: What an Adverse Action Notice Actually Leaks

**Grace** · grace@gracefulco.de · *Preprint, September 2026*

---

## Abstract

Regulation increasingly compels explanation of automated adverse decisions,
while a security literature shows explanations enable model extraction. The
apparent conflict has produced defences that degrade explanation quality to buy
security. We argue the conflict has been mismeasured, and that the relevant
variable is not the presence of an explanation but two properties of the
quantities it names: whether the requester can **set** them, and whether the
requester can **observe** them.

We report five measurements on synthetic decision systems. Against an adversary
who controls every input, explanation disclosure is worth a factor of
1 + (log₂ R + 2)/(d + 2) in our rule-list setting, where R is threshold
resolution and d feature count — about two at the values we happened to choose,
but 1.15× to 3× across their range — and, on a continuous model, a small
positive gain for counterfactual explanations whose apparent magnitude depends
heavily on the metric used to state it. Unbudgeted per-query perturbation of
the explanation is dominated: at σ = 1.5 it costs 71 percentage points of
disclosure accuracy for at most one point of extraction resistance, because
independent noise averages out over a few hundred queries. We do not evaluate a
composed differential-privacy mechanism and claim nothing about one. As inputs
move outside the adversary's control the picture inverts, and full disclosure
recovers 55 percentage points more than outcome-only. Within that regime an
adversary who filters its own observations for attributability recovers as much
without a reason code as with one: what attribution buys is convergence rate,
not reach.

We derive a disclosure rule and apply it to Regulation B Appendix C Form C-1,
the adverse action notice the CFPB publishes. Most listed reasons are free to
disclose; the costly ones are thresholds on quantities the applicant did not
supply. The rule turns out to be close to what Regulation B already requires —
the Official Interpretations relieve a creditor of describing how or why a
factor operated, and forbid a bare qualifying-score statement — so the security
analysis and the compliance obligation point the same way. Where they do not is
narrower and, we think, the more useful finding: naming the responsible feature
without its threshold, which the specificity requirement mandates and our rule
permits, still reveals the existence of 61% of decision rules against 9% for
outcome-only disclosure. That is the measured price of a specificity mandate.
An open-source implementation enforces the rule as a declarative contract.

**Status: preliminary.** Synthetic targets, low dimensionality, single author.
The limitations section is longer than the results and should be read first by
anyone inclined to cite this.

---

## 1. Introduction

Two bodies of work have reached opposite conclusions about the same artifact.

Consumer protection law treats the explanation of an adverse automated decision
as a right. The Equal Credit Opportunity Act and Regulation B require creditors
to give specific, accurate principal reasons for a denial: the statement "must
be specific and indicate the principal reason(s) for the adverse action"
(12 CFR 1002.9(b)(2)), and the reasons disclosed must relate to and accurately
describe the factors actually considered. CFPB Circular 2022-03 took the
position that "ECOA and Regulation B do not permit creditors to use complex
algorithms when doing so means they cannot provide the specific and accurate
reasons for adverse actions" (87 FR 35864, 35865).

That circular and Circular 2023-03 were **withdrawn effective 12 May 2025**
(90 FR 20084), and this paper does not rely on them. The regulation and the
Official Interpretations on which they rested remain in force and are the
operative authority throughout; where a circular is quoted it is quoted as a
statement of a position once taken, not of current guidance.

EU AI Act Article 86 gives a person subject to a decision taken by a deployer
on the basis of an Annex III high-risk system, excluding those under point 2,
the right to obtain "clear and meaningful explanations of the role of the AI
system in the decision-making procedure and the main elements of the decision
taken." Creditworthiness evaluation is Annex III 5(b), with a carve-out for
fraud detection. The right is subsidiary to other Union law and its scope — the
system's *role* — is materially narrower than ECOA's specific-reasons duty. It
requires nothing R1 would forbid.

Security research treats the same explanation as attack surface. Counterfactual
explanations have been used to extract high-fidelity surrogate models in
hundreds of queries. The mitigation most commonly proposed applies differential
privacy to the explanation generator, degrading explanation quality to reduce
leakage.

Stated together these produce a bind: the law compels the disclosure security
says is dangerous, and the published defence works by breaking the compliance.

We set out to propose a resolution and failed. What the measurements produced
instead is a reframing: **explanation is not the unit of analysis.** The
relevant properties belong to the quantities an explanation names.

### Contributions

1. **Two axes — settability and observability** — that predict disclosure cost,
   and evidence that they, rather than model complexity, determine it (§3, §5).
2. **A negative result on unbudgeted explanation perturbation** — the defence
   practitioners actually deploy — which is dominated because independent
   per-query noise averages out. This is not a result about differential
   privacy, which requires a sensitivity bound and a composition accountant
   that our mechanism does not have (§5.1).
3. **A calibration of the full-input-control regime**: against an adversary who
   supplies every feature, disclosure is worth a factor set by threshold
   resolution over dimensionality, invariant to rule count by construction
   (§5.2, §5.3).
4. **A negative result on our own prior claim**: we initially reported that
   attribution rather than observation makes disclosure costly. A stronger
   outcome-only adversary, described in §5.5, erases the gap (§5.5).
5. **A two-part disclosure rule** and its application to the CFPB's own notice
   template, including an observation about FCRA §615(a) (§6, §7).
6. **An implementation** — a declarative disclosure contract with a linter that
   enforces the rule (§7.3).

---

## 2. Background

### 2.1 Disclosure mandates

ECOA and Regulation B §1002.9 require a statement of specific principal reasons
for adverse action. The regulation does not mandate how many: comment
9(b)(2)-1 provides that it "does not mandate that a specific number of reasons
be disclosed," only that more than four is unlikely to help the applicant.
Appendix C provides sample notification forms; Form C-1 carries a checklist of
twenty-three reasons plus a free-text "Other, specify."

The checklist is a default, not a closed set. Appendix C comment 3 states the
sample forms "are illustrative and may not be appropriate for all creditors,"
and instructs that where a creditor's common reasons are absent, it "should
modify the checklist by substituting or adding other reasons" — the regulator's
own examples being "inadequate down payment" and "no deposit relationship with
us." The second is an internal, relationship-derived reason stated without a
threshold, which is worth noting against the argument in §7.2.

FCRA §615(a) imposes a separate obligation. A creditor that used a credit score
must disclose the score, the range, the date, the source, and **up to four key
factors that adversely affected the score** — five where number of inquiries is
among them. The CFPB is explicit that satisfying this does not satisfy the ECOA
duty; they are distinct disclosures.

EU AI Act Article 86 grants a right to explanation of individual
decision-making for Annex III high-risk systems, running against the
**deployer** rather than the model provider.

### 2.2 Explanations as attack surface

Model extraction reconstructs a target from query access. Counterfactual
explanations are unusually informative for this purpose because a counterfactual
is, by construction, a point on the decision boundary. Reported results include
high-fidelity surrogates from roughly 500 queries where the adversary has
partial knowledge of the data distribution. Proposed mitigations centre on
perturbing the explanation generator.

### 2.3 What is missing

Stakeholder-tailored explanation is a mature literature, but it is motivated by
comprehension and trust; it does not ask what an audience being adversarial
costs. Extraction defence is also mature — query budgets, rate limiting, output
perturbation, watermarking — but it treats requesters as interchangeable and
asks how many queries, never who is owed an answer.

Neither literature distinguishes explanations by the **provenance of the
quantities they name**, which is what we find to be decisive.

---

## 3. Threat model

An adversary submits applications to a decision system and observes the
response. They seek to recover the system's parameters: which features are
used, and at what thresholds.

We characterise each feature by two independent properties.

**Settability.** Whether the adversary can choose the value. A declared income
is settable; a bureau score is not. A settable feature can be binary-searched:
the threshold falls in O(log range) submissions regardless of what the notice
says. An unsettable feature cannot be *bisected*, because the adversary cannot
place a probe. It does not follow that it cannot be learned: where its
realisations vary across submissions, statistical estimation remains open, and
§5.5 measures that route.

**Observability.** Whether the adversary can see the value that was used.
Observable-but-unsettable features admit a weaker attack — passive bracketing,
in which the adversary accumulates (value, outcome) pairs and narrows the
threshold from whichever side the evidence is unambiguous on.

This yields three regimes:

| | Settable | Unsettable, observable | Unsettable, unobservable |
|---|---|---|---|
| Attack available | binary search | passive bracketing | none deterministic |
| Cost | O(log range) | O(range / resolution) | statistical only |
| What disclosure adds | a constant factor | speed | everything |

The prediction that follows, and that §5 tests, is that disclosure cost is
concentrated in the right-hand columns. We initially predicted further that in
the middle column the binding requirement would be attribution — knowing which
rule an observation belongs to — rather than observation itself. §5.5 tests
that and rejects it: an adversary who reasons about which observations are
unambiguous needs no attribution, and what the notice supplies is convergence
rate. The middle column's entry above reflects the corrected result.

We note what this threat model leaves unspecified, because it is more than it
should be: query budgets vary from 600 to 400,000 across the experiments
without justification; the adversary is granted the target's model family; and
in every experiment it samples queries from the same distribution on which
fidelity is evaluated, which is a strong and unmodelled advantage over the
adversaries in the work we are arguing with.

---

## 4. Method

We evaluate five synthetic decision-system configurations against a query-based
adversary. All harnesses are implemented in Go using only the standard library,
with fixed seeds; each is a single file of under 350 lines and the full set
runs in under twenty seconds.

The adversary knows the target's model family. We previously described it as
"assumed competent," which §5.5 shows we were not entitled to say — the
adversary is as competent as we made it, and no more.

Two metrics recur:

- **Extraction** — surrogate agreement with the target on held-out inputs, or
  for rule lists the share of rules whose feature *and* threshold are recovered.
  These are different quantities and we should not have given them one name.
- **Reason fidelity** — the share of denials for which the disclosed top-3
  principal reasons match the true top-3 contributions. This is a measurable
  proxy for Reg B's specificity requirement, **not** the regulatory standard:
  the regulation fixes no number of reasons (comment 9(b)(2)-1), and for
  scoring systems comment 9(b)(2)-5 specifies comparative selection against
  near-threshold approvals rather than ranking by absolute contribution. A
  notice scoring zero here is not "0% compliant"; it is a violation. We use the
  metric only to detect *degradation* under a defence.

A defence that improves one and degrades the other has not resolved anything,
which is why both are always reported.

Trial counts, which an earlier draft omitted from every table but §5.2:

| Section | Target | n | Aggregation |
|---|---|---|---|
| 5.1 | 6-feature linear | **1** | single run |
| 5.2 | 12-feature rule list | 25 | median |
| 5.3 | 8-dim tanh, 12 hidden | 7 | median |
| 5.4 | 12-feature, 8 rules | 40 | mean |
| 5.5 | 12-feature, 8 rules | 40 | mean |

No table in this paper reports dispersion. That is a defect and it is not yet
fixed.

---

## 5. Results

### 5.1 Linear scorecard: unbudgeted perturbation is dominated

Six-feature linear model, threshold decision, 600-query budget. **Single target,
single adversary seed** — see §8.

| Policy | Reason fidelity | Extraction |
|---|---|---|
| Uniform full disclosure | 100% | 100% |
| Unbudgeted Gaussian (σ 0.5) | 62% | 100% |
| Unbudgeted Gaussian (σ 1.5) | 29% | 99% |
| Outcome only | 0% | 93% |
| Entitlement-scoped | 100% | 100% |

Independent per-query noise averages out across a few hundred samples, so the
adversary recovers the clean signal regardless. At σ 1.5 the defence costs 71
points of compliance for at most one point of extraction resistance.

**This is not a result about differential privacy**, and an earlier draft
labelled it as one. Our mechanism adds i.i.d. Gaussian noise at a fixed
standard deviation with no sensitivity bound, no δ, and no composition
accountant across the 600 releases. Noise added without a composed privacy
budget is not a defence, it is a tax on the notice. A mechanism that composes a
budget across the explanation sequence is not subject to the averaging attack
described here, we have not evaluated one, and nothing in this section bears on
it. What the row does measure is the perturbation shape that gets deployed in
practice, which is worth knowing separately.

Outcome-only disclosure still reaches 93%: for a model this simple the decision
boundary alone gives it up, and explanations are not the binding leak.

We also tested *entitlement-scoped* disclosure — faithful notices to
authenticated subjects, outcomes to bulk requesters — on the theory that the
legally entitled party cannot mount the attack. Ten manufactured identities
sufficed for full reconstruction.

### 5.2 Rule lists: the advantage is a flat constant

Knockout rule lists over twelve features, thresholds on 0–4096, adversary must
recover every rule's feature and threshold. Median of 25 lists.

| Rules | Full | Reason named | Outcome only | Ratio |
|---|---|---|---|---|
| 1 | 16 | 29 | 30 | 1.9× |
| 2 | 27 | 53 | 55 | 2.0× |
| 4 | 54 | 106 | 110 | 2.0× |
| 8 | 111 | 215 | 223 | 2.0× |
| 12 | 169 | 325 | 337 | 2.0× |

The ratio does not grow with rule count — but that is entailed rather than
discovered, and the constant is an accident. Cost is additive per rule under
every policy, because with k ≤ d and features drawn without replacement no two
rules ever share a feature, so rule count cancels. Reading the columns:
`outcome − full` is exactly 14 per rule at every k, and `outcome − reason`
exactly 1. So

> ratio = 1 + (log₂ R + 2) / (d + 2)

and the observed 2.0 is what you get when log₂(4096) + 2 = 14 happens to equal
d + 2 = 14. Two unrelated constants we chose and did not sweep. The ratio grows
without bound in threshold resolution and decays toward 1 in dimensionality;
we have not run those sweeps and the abstract's range is derived from the
closed form, not measured.

Note also that *reason* and *outcome* differ by exactly one query per rule.
Naming the feature buys almost nothing here because our adversary's probe
differs from its base vector in exactly one coordinate, so identifying that
coordinate costs one query by construction. An adversary probing several
coordinates at once, or facing rules that share features, would pay more.

### 5.3 Continuous model: counterfactuals add little

Eight-dimensional tanh network, twelve hidden units, matched-architecture
surrogate. Counterfactuals are supplied free of query cost, which is generous.

| Queries | Outcome only | Counterfactual | Gain |
|---|---|---|---|
| 50 | 81% | 83% | +1.8 pp |
| 250 | 88% | 88% | −0.1 pp |
| 1000 | 92% | 94% | +1.4 pp |
| 2000 | 93% | 96% | +3.0 pp |

The gain grows with budget, which is the direction the literature predicts.

We previously described its magnitude as small. That description depends
entirely on the metric. Percentage points of agreement compress the top of the
range — 93% to 96% is a halving of the error rate — and the extraction
literature reports query efficiency rather than agreement. Stated that way the
same numbers say something different: the disclosed-explanation adversary
reaches at 2,000 queries what the outcome-only adversary does not reach at
several times that budget. We report the percentage-point framing above because
it is what we measured, and we flag it as the framing that makes the gain look
smallest.

Two further caveats belong with the number rather than in §8. The gain is a
difference of medians over seven targets where the trials are paired and a
paired estimator is the correct one, and our surrogate has the target's
architecture with hyperparameters fixed across every budget. Both choices work
against the disclosed arm.

### 5.4 Attested inputs: the picture inverts

Eight rules over twelve features, sweeping the fraction of features supplied
outside the adversary's control and not shown to them. *Recovered* means feature
and threshold known; *discovered* means the rule is known to exist.

| Attested | full rec/disc | reason rec/disc | outcome rec/disc |
|---|---|---|---|
| 0% | 100% / 100% | 100% / 100% | 100% / 100% |
| 25% | 88% / 88% | 60% / 87% | 44% / 44% |
| 50% | 81% / 81% | 36% / 82% | 22% / 22% |
| 75% | 64% / 64% | 15% / 61% | 9% / 9% |

At 0% the policies are indistinguishable, reproducing §5.1–5.3. At 75% full
disclosure recovers 55.6 percentage points more than outcome-only (64.4 ± 18.9
against 8.8 ± 11.4, n = 40). We report the difference rather than a ratio
because outcome-only recovers nothing in 21 of 40 trials, so a per-trial ratio
is undefined for most of the sample; where it is defined it averages 4.7×.

A threshold on a quantity the adversary cannot set is unreachable *by
bisection* at any budget, and a notice that prints it transfers it in one
query. It is not unreachable in general: where the quantity's realisations vary
across submissions it remains estimable statistically, which is what §5.5
measures.

The **discovered** column carries a separate and, we now think, more important
result. Naming the feature without its threshold — which is what Regulation B
requires and what our own R1 permits — still reveals the existence of 61% of
rules at 75% attestation, against 9% for outcome-only. That gap is the measured
cost of a specificity mandate, and it is the one cost in this paper that no
disclosure policy compatible with the regulation can avoid.

### 5.5 Attribution buys convergence rate, not reach

Fifty percent attested, adversary observes the attested values that arrived
with their own applications. Threshold counted recovered within 1% of scale,
40 trials per cell.

**This section retracts a claim.** We first reported that observation alone
confers nothing at any budget while observation plus an attributing reason code
confers eleven points, and concluded that attribution rather than observation
is what makes disclosure costly. That conclusion was an artifact of our own
adversary, and the corrected result is below.

The original outcome-only adversary charged every decline to every attested
feature, on the reasoning that a decline does not say which attested rule
fired. True, but it discards the other half of the evidence. A one-sided
threshold rule passes an *interval*. So if a feature has been seen taking both
its largest and its smallest approving value, every value between them also
passed — whatever direction the rule runs, and without being told the rule
exists. Certify each attested value against its own observed approval interval,
and where exactly one feature fails to certify, the decline belongs to it. No
reason code, no extra queries, no knowledge the outcome-only adversary does not
already hold in its observation log.

| Budget | full | reason | outcome (naive) | outcome (self-certifying) |
|---|---|---|---|---|
| 200 | 76% | 19.1% | 14.1% | 15.9% |
| 1,000 | 76% | 20.3% | 13.8% | 19.4% |
| 5,000 | 76% | 23.1% | 13.8% | 22.8% |
| 20,000 | 76% | 25.0% | 13.8% | **25.0%** |
| 100,000 | 76% | 25.0% | 13.8% | **25.0%** |

**Observability is worth eleven points; attribution is worth time.** The naive
adversary is stuck at 13.8% forever. The competent one reaches the same 25.0%
as the reason-code adversary by 20,000 queries and stays there. What the reason
code buys is the approach: 3.1 points at 200 queries, 0.9 at 1,000, nothing at
all by 20,000.

This should not have been surprising. Recovering an axis-aligned threshold from
approvals alone is the tightest-bounding-box learner, PAC-learnable from
positive examples only. Approvals never needed attribution; only declines do,
and an adversary who can wait does not need declines.

Two caveats on the rest of the table. The `full` column is constant by
construction — observability does not change any decision, and full disclosure
never consults the adversary's observations — so its 76% ceiling is rule
masking rather than a saturation of the attack. And the hidden-attestation
columns of the earlier draft were a code identity rather than a measurement:
that harness had no distinct branch for `reason`, so it executed the
outcome-only path twice. They are removed rather than corrected.

---

## 6. A disclosure rule

> **R1.** A notice must not print a threshold on a quantity the requester
> cannot set. It is unreachable by bisection at any budget, so the notice is
> the cheapest route to it by a wide margin.
>
> **R2** *(provisional, and see below).* Where that quantity is observable to
> the requester, naming the rule accelerates recovery by roughly two orders of
> magnitude in queries. It does not change what is eventually recoverable.

**R1 and R2 do not have equal standing, and the difference is not a matter of
degree.**

R1 is compatible with Regulation B, and on inspection is close to what the
regulation already requires. Comment 9(b)(2)-3 provides that a creditor "need
not describe the mechanics of adverse impact," offering as its own example that
a notice may say "length of residence" rather than "too short a period of
residence." §1002.9(b)(2) goes further and forbids the leaky formulation
outright: a statement that the applicant failed to achieve a qualifying score
on the creditor's scoring system is *insufficient*. A notice printing a
behavioural score and its floor is not an over-compliant notice. It is a
non-compliant one. R1 is therefore a security-side derivation of a rule the
regulator adopted for comprehension reasons, which we take as mild evidence
that the rule is right.

R2 is a different matter. §1002.9(b)(2) requires the notice to indicate the
principal reason, and comment 9(b)(2)-4 provides that no factor that was a
principal reason may be excluded from disclosure. R2 asks a creditor to
withhold precisely such a factor. **In the US consumer credit setting R2 is
unlawful**, and we state it as a measurement rather than a recommendation. Its
application is to settings without a specific-reasons mandate — internal risk
platforms, B2B decisioning — and, more usefully, as a way of pricing what the
specificity mandate costs. §5.4's discovered column puts that price at 61% of
rules revealed as existing against 9% for outcome-only.

R1 remains separately checkable and separately enforceable, and it requires a
disclosure mechanism to scope **rules** and **facts** independently: a system
with a single verbosity dial cannot express "name the reason but not the
cutoff," which is the operating point both the measurements and the regulation
point at.

---

## 7. Case study: Regulation B Form C-1

### 7.1 Classification

Applying settability and observability to the twenty-three listed reasons:

- **Eight are free.** The applicant supplied the input — income, employment
  length, residence length, references, completeness. Binary-searchable, so
  disclosure saves a constant factor.
- **Nine are near-free.** Binary facts with no threshold behind them —
  bankruptcy, judgment, foreclosure, garnishment, unable-to-verify. Naming them
  tells the applicant something they already knew about themselves.
- **Six carry real cost**, and cluster in one category: numeric thresholds on
  quantities the applicant did not supply. Excessive obligations relative to
  income; limited credit experience; delinquent obligations with others; number
  of recent inquiries; collateral sufficiency; and *poor credit performance with
  us* — the only listed reason that is neither settable nor observable.

Two of those six are contested and the count is sensitive to them. *Excessive
obligations in relation to income* is a ratio of two applicant-supplied
quantities, and declared income is fully settable, so an adversary sweeps
income against fixed debts and bisects the cutoff — which would make it free,
and is hard to reconcile with our classifying *income insufficient for the
amount of credit requested* as free. *Value or type of collateral not
sufficient* conflates two things: the applicant chooses the collateral type,
and for first-lien dwelling-secured applications §1002.14 entitles them to the
appraisal, so only the creditor's valuation is unsettable and it is not
unobservable. Under the alternative coding the costly count is four. We report
the split we think is right and flag that it is a judgment, not a measurement.

A further caveat on the column headings. "Applicant sets it" is used here in
the sense that matters for extraction — *the adversary can choose the value
submitted* — not in the sense of whether an applicant can change the fact about
their life. A bureau score is unsettable in the first sense and, by design of
the FCRA disclosure, very much settable in the second. The two come apart and
only the first is measured.

### 7.2 Two observations about the statutory scheme

**The FCRA §615(a) score disclosure is unsettable and observable, but it is not
the maximal-cost cell, and it may not be our threat model at all.** An earlier
draft claimed it instantiated the maximal-cost configuration exactly. Two
corrections, both of which narrow the claim.

First, §5.5's substantial cost sits in the `full` column, which prints the
*cutoff*. FCRA §615(a)(2) requires the score, the range, the date, the source,
and up to four key factors — five where number of inquiries is among them — but
it does not require the creditor's cutoff, and Form C-1 does not carry one.
That places the disclosure in the observed cell at 25%, not the printed-cutoff
cell at 76%. And since attribution turns out to buy convergence rate rather
than reach (§5.5), the four key factors are not what makes it costly; the
observability of the value is.

Second, and more seriously: FCRA key factors attribute to the **consumer
reporting agency's scoring model**, not to the creditor's underwriting model.
15 U.S.C. 1681g(f)(2)(B) defines them as reasons "adversely affecting the
credit score," listed in order of their effect on that score. Our threat model
throughout is an adversary attacking the system they are submitting to. The
FCRA claim therefore only holds if the extraction target is the bureau's model
— which is a defensible and arguably more interesting claim, but a different
one, and it has to contend with the fact that bureau scoring models are already
the most reverse-engineered systems in consumer finance.

We are not arguing the disclosure is wrong. It exists because opacity in credit
scoring was a serious harm and the remedy was hard-won, and we would rather the
cost of it be measured than asserted: unmeasured costs are the ones that get
claimed at inflated magnitude by whoever wants the disclosure narrowed. Our
number is small.

**A specificity mandate has a real price, and it is not the one we first
identified.** We previously argued that guidance pressing creditors toward
accuracy pushes disclosure into the costly category. That argument does not
survive contact with the regulation.

The premise was that a model keyed on an internal quantity has no checklist
entry, so complying accurately means writing the quantity into free text. But
Appendix C comment 3 gives creditors a second option the argument ignored —
modify the checklist, adding a properly worded reason, as the regulator's own
"no deposit relationship with us" example does. And comment 9(b)(2)-3 states
directly that a creditor need not describe how or why a factor operated. The
"designable middle" we proposed — naming the reason at the granularity the
applicant needs to act on, without printing the cutoff — is not a compromise we
invented. It is what §1002.9 already prescribes, and §1002.9(b)(2) makes the
leaky alternative insufficient on its face.

The genuine tension is elsewhere, and our own data locates it. Regulation B
requires the notice to name the specific principal reason. §5.4 measures what
naming the feature costs even when the threshold is withheld: 61% of rules
revealed as existing at 75% attestation, against 9% for outcome-only. **That is
the price of the specificity mandate, it is unavoidable for anyone complying
with ECOA, and to our knowledge it has not previously been quantified.** It is
a smaller and more defensible claim than the one it replaces.

### 7.3 Implementation

We encode the classification as a declarative contract in
[warden](https://github.com/Grace/warden), an open-source guard between a rules
engine and its consumers. Engine terms are declared as either **facts** (about
the case) or **parameters** (about the policy), each with an audience. The
distinction is exactly settability, made explicit rather than inferred: a
previous version guessed it from words like "limit" and "floor."

A linter enforces R1 mechanically. On the compliant Form C-1 contract it
reports clean; on a variant that prints the risk tier and its floor it reports
two findings and exits non-zero. That variant is not a more accurate notice —
§1002.9(b)(2) makes a qualifying-score statement insufficient on its face — so
the linter catches a notice that is simultaneously leakier and less compliant,
which is the common case and the more useful one to detect. The classification
is therefore mechanically checkable rather than advisory, and the check runs in
CI.

This is an existence proof, not an evaluation. We have not built a labelled
corpus of contracts and cannot report precision or recall.

---

## 8. Limitations

These matter more than the results and are stated at length deliberately.

**Dimensionality.** All targets are six to twelve features. The extraction
literature's strong results occur in higher dimensions, where random sampling
covers the space far less efficiently and a boundary point is correspondingly
more valuable. This is the most likely explanation for the small counterfactual
gain in §5.3.

**Architectural prior.** The §5.3 surrogate has the same architecture as the
target. A real adversary usually does not know the model family, and removing
this prior should help the disclosed-explanation adversary more than the
outcome-only one.

**Adversary input control.** §5.1–5.3 grant total control, which §5.4 shows is
the decisive assumption. It is realistic for a credit applicant and wrong
wherever inputs are attested. Most published extraction results assume it too,
which is the point.

**Synthetic targets.** No production model, no real notice, no real applicant
population. The Form C-1 analysis is of the published sample form, not of any
institution's implementation.

**Adversary strategies are hand-written and not optimised.** This is the
limitation that has already bitten us. Every claim of the form "policy X
confers nothing" is a statement about one algorithm, and §5.5 documents a case
where a modest improvement to that algorithm erased a headline finding. Results
here upper-bound what a disclosure policy protects, never lower-bound it.

**Experiment 1 is a single target and a single adversary seed.** The 71-point
compliance figure is stable across seeds; the one-point extraction figure is
not, and should be read as "at most one point" rather than as a measurement.

**Attested values are redrawn independently on every submission.** This models
a quantity that varies per application, not a stable per-identity attribute
such as a bureau score, and it assigns zero correlation between attested and
submitted features. Real internal tiers and bureau scores are functions of
submitted data and are therefore partially inferable through their correlates,
which is exactly what R1 denies. The independence is the strongest possible
assumption in R1's favour and we have not relaxed it.

**Threshold recovery is not verified against ground truth** in §5.2 or §5.4.
The bisection result is discarded and the rule is marked recovered once a
bisection is attempted, so those experiments measure the cost of a procedure
assumed to succeed.

**No independent replication.** The harnesses are seeded, single-file and
dependency-free; each runs in under a minute and the full set in under twenty
seconds. Replication and correction are invited.

The honest summary is not that the extraction literature is wrong. It is that
**disclosure risk appears to be a property of dimensionality, input control, and
architectural uncertainty rather than of explanation as such** — a claim we have
partially tested and not established.

---

## 9. Related work

Model extraction from explanations, particularly counterfactual explanations,
is well studied, as are defences based on query budgets, rate limiting, output
perturbation, and watermarking. Differential privacy applied to explanation
generators is the mitigation most directly comparable to §5.1.

Stakeholder-tailored explainability establishes that audiences differ in what
they need, with taxonomies distinguishing developers, owners, users, regulators
and society. That work is motivated by comprehension; to our knowledge it does
not treat audience as an access-control principal or ask what an adversarial
audience costs.

**Settability is not new and we should not have implied it was.** The
algorithmic recourse literature has partitioned features into actionable,
immutable and conditionally immutable since Ustun, Spangher and Liu (2019), and
Kothari, Kulynych, Weng and Ustun (ICLR 2024) formalise exactly the object we
need under the name *reachable sets*, in consumer finance, with a verification
procedure that is a close relative of the linter in §7.3. Strategic
classification (Hardt et al. 2016; Miller, Milli and Hardt 2020) partitions on
manipulability for the same underlying reason. Barocas, Selbst and Raghavan
(2020) interrogate Regulation B principal reasons and counterfactual
explanations directly, which is §7's subject.

Our contribution is not the partition. It is the adversarial reading of it:
that literature asks whether actionability is sufficient for the *subject* of a
decision, and we ask what it implies for an *adversary*, and find the two in
tension. That is a narrower claim than the one an earlier draft made and it is
the one the evidence supports.

We also note a terminology collision we should own rather than ignore:
settability and observability are Kalman's controllability and observability,
and the access axis in the extraction literature is already named label-only
versus confidence-vector (Tramèr et al. 2016).

This section still lacks citations to the extraction literature it argues with
— Tramèr et al. 2016, Milli et al. 2019, Aïvodji et al. 2020, Wang et al. 2022,
Jagielski et al. 2020 — and to Patel, Shokri and Zick (2022), whose composed
mechanism is the defence §5.1 should have been measured against. Writing that
survey is the largest single piece of work outstanding.

---

## 10. Conclusion

The conflict between explanation mandates and extraction risk is smaller and
differently located than it appears. Against an adversary who controls the
inputs — the case most studied — explanation is worth a constant factor set by
threshold resolution over dimensionality, and the naive perturbation defence is
dominated. The cost is concentrated in quantities the requester cannot set.

That yields a rule narrow enough to enforce, and the most useful thing we can
say about it is that Regulation B appears to have got there first: the
Interpretations already relieve a creditor of printing the mechanics, and
already make a bare score-threshold statement insufficient. Independent
arrival at the same rule from a security argument is weak evidence, but it is
evidence, and it means the compliant notice and the low-leak notice are
substantially the same notice.

Where they diverge is the specificity requirement itself, which mandates naming
the responsible feature and which we measure at 61% of rules revealed against
9%. That price is real, unavoidable under ECOA, and as far as we can tell
unpriced until now.

The larger suggestion is that the field has been optimising the wrong variable:
not how much to explain, but which quantities an explanation is permitted to
name — and, on the evidence of §5.5, how strong an adversary you assumed when
you decided.

---

## Availability

Harnesses: `warden/experiment/{oracle,disclosure,scaling,continuous,attested,observable}`.
Contract and linter: [github.com/Grace/warden](https://github.com/Grace/warden).
Go, standard library only, seeded. Corrections and replication welcome —
**grace@gracefulco.de**.
