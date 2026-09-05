---
title: "Auditing Form C-1"
description: "Applying a security disclosure rule to the adverse action notice the CFPB actually publishes. What leaks, what does not, and where Regulation B already drew the line."
permalink: /form-c1-audit/
---

# Auditing Form C-1

*Applying the disclosure rule to the adverse action notice the CFPB actually
publishes. What leaks, what doesn't, and why the regulator's push for accuracy
points at the leaky half.*

**Not legal advice.** This is a security analysis of a compliance artifact. The
classification below is arguable institution by institution, and that is the
point — it is a question to ask, not an answer to adopt.

---

## The rule being applied

From five measurements ([write-up](explanation-disclosure-measurements.md)):

> **1.** A notice must not print a threshold on a quantity the requester cannot
> set. That value is unreachable by search at any budget, so the notice is the
> only route to it — 7× more disclosure than outcome-only at 75% attestation.
>
> **2.** Where that quantity *is* observable to them, the notice must not name
> the rule either. Observation alone bought nothing at 100,000 queries;
> observation plus an attributing reason code bought eleven points. Attribution
> is the unlock.

Disclosure of anything the applicant sets themselves is close to free — they can
find it by resubmitting, and the notice saves them a constant factor of about
two.

## The checklist, classified

Regulation B Appendix C, Form C-1, "Principal Reason(s) for Credit Denial."

| Reason | Applicant sets it? | Applicant sees it? | Leak |
|---|---|---|---|
| Credit application incomplete | yes | yes | none |
| Insufficient number of credit references | yes | yes | none |
| Unacceptable type of credit references | yes | yes | none |
| Temporary or irregular employment | yes | yes | none |
| Length of employment | yes | yes | none |
| Income insufficient for amount requested | yes | yes | none |
| Length of residence | yes | yes | none |
| Temporary residence | yes | yes | none |
| Unable to verify credit references | no | no | low — binary, no threshold |
| Unable to verify employment | no | no | low — binary, no threshold |
| Unable to verify income | no | no | low — binary, no threshold |
| Unable to verify residence | no | no | low — binary, no threshold |
| No credit file | no | yes | low — binary |
| Bankruptcy | no | yes | low — binary |
| Foreclosure or repossession | no | yes | low — binary |
| Garnishment or attachment | no | yes | low — binary |
| Collection action or judgment | no | yes | low — binary |
| Delinquent obligations with others | no | yes | **rule 2** — observable, attributed |
| Limited credit experience | no | yes | **rule 2** — threshold on an observable |
| Excessive obligations in relation to income | partly | partly | **rule 2** — DTI cutoff |
| Number of recent inquiries on report | no | yes | **rule 2** — countable, observable, thresholded |
| Value or type of collateral not sufficient | no | partly | **rule 1** — appraisal cutoff |
| **Poor credit performance with us** | no | **no** | **rule 1** — internal, unobservable |
| **Other, specify: ___** | — | — | **see below** |

Eight of twenty-three are free to disclose. Nine are near-free — binary facts
with no threshold behind them, where naming the reason tells an adversary
something they already knew about themselves.

Six carry real disclosure cost, and they cluster in one place: **numeric
thresholds on quantities the applicant did not supply.**

## Two findings

### The mandated credit score disclosure is the maximally leaky configuration

FCRA §615(a) requires a creditor using a credit score to disclose **the score
itself**, and **up to four key factors that adversely affected it** — five if
inquiries is among them. Form C-1 carries this alongside the ECOA reasons, and
the two obligations are distinct: the CFPB is explicit that disclosing key
score factors does *not* satisfy the ECOA duty to give specific reasons.

Read that against experiment 5. The score is:

- **not settable** by the applicant,
- **observable** to them — the notice discloses it, and they can buy it anyway,
- and accompanied by **named attributing factors**, which is precisely the
  thing that converted 14% recovery into 25% in the measurement.

Observable, not settable, with attribution supplied. That is the exact
configuration my harness identifies as the only one where disclosure carries
real cost, and it is not an oversight — it is the statute.

I am not arguing the disclosure is wrong. It exists because opacity in credit
scoring was a serious harm and the remedy was hard-won. The point is narrower:
**this is the one field in the notice where the security cost is real, and
nobody appears to have priced it.**

### The push for accuracy points at the leaky half

CFPB Circular 2023-03 says a creditor may not check the closest box when it
does not accurately name the actual principal reason, and Circular 2022-03 says
this binds regardless of model complexity. If neither applies, the guidance is
to use **"Other, specify"** and say what really happened.

Now consider a model keyed on a derived or internal quantity — a behavioural
score, an internal risk tier, a margin floor, a fraud model output. The
checklist has no box for it. "Poor credit performance with us" is the only
internal reason on the form, and it is the one row I classified as rule 1: not
settable, not observable, maximum leak.

So complying accurately means writing the internal quantity into "Other,
specify." **The regulator's correct insistence on accuracy pushes disclosure
toward exactly the category where disclosure is expensive**, and the more novel
your model, the further it pushes you.

That is a real tension and it has a resolution that does not require breaking
either rule: state the reason at the **granularity the applicant needs to act**
without printing the **cutoff**. "Your recent payment history with us" is
accurate and actionable. "Behavioural score 412, threshold 450" is accurate,
actionable, and hands over a number no applicant could have computed.

Whether that satisfies §1002.9 is a question for counsel, not for me. But it is
a designable middle, and the measurement says the middle is where nearly all of
the security cost sits.

## What this makes possible

The classification above is a **contract**, in warden's sense: per-reason,
per-audience, with facts scoped separately from the rule that produced them.
That separation is not a design preference — experiment 5 is the argument for
it. Naming the rule and printing its threshold are different disclosures with
different costs, and a system with a single verbosity dial cannot express the
difference.

Concretely, the next artifact is `form-c1.json`: the twenty-four reasons as a
warden contract, with the six flagged rows carrying `audience: internal` on
their thresholds and `audience: public` on their reason codes. Then
`warden lint` on a real institution's mapping becomes an audit anyone can run.

## What I have not done

Not looked at a real institution's notice, only the published sample. Not
tested whether the "granularity without cutoff" middle survives a §1002.9
analysis. Not checked whether FCRA's key-factor codes (the standardised reason
codes the bureaus emit) are themselves thresholded — if they are, that is a
second instance of the same finding and a larger one.

---

*Companion to [explanation-disclosure-measurements.md](explanation-disclosure-measurements.md).
Sources: [Reg B Appendix C](https://www.consumerfinance.gov/rules-policy/regulations/1002/c/) ·
[§1002.9](https://www.consumerfinance.gov/rules-policy/regulations/1002/9/) ·
[Circular 2022-03](https://www.federalregister.gov/documents/2022/06/14/2022-12729/consumer-financial-protection-circular-2022-03-adverse-action-notification-requirements-in) ·
[Circular 2023-03](https://www.federalregister.gov/documents/2024/04/17/2024-08003/consumer-financial-protection-circular-2023-03-adverse-action-notification-requirements-and-proper)*
