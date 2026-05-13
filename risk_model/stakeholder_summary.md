# Clair Wage Advance Risk Model — Business Summary

**Prepared by:** Jeremy Thaller  
**Audience:** Non-technical stakeholders

---

## What problem are we solving?

When a Clair user requests a wage advance, we face a decision: approve or decline. Approving users who won't repay creates a direct financial loss. Declining users who would repay costs us a good customer relationship and fee revenue.

Today, that decision may rely on simple rules. This model gives us a data-driven score for every request so we can make smarter, more consistent decisions at scale.

---

## How the model works (in plain English)

The model looks at signals we already know about the user at the moment they request an advance:

| Signal type | What we use | Why it matters |
|---|---|---|
| Payment history | How often they've repaid on time vs. late | Past behavior predicts future behavior |
| Income stability | Average paycheck, whether it's growing or shrinking | A steady, rising income = lower risk |
| Debt load | Advance amount relative to their typical paycheck | Borrowing 20% of a paycheck is safer than 90% |
| Job tenure | Days since hire date | Longer tenure → more stable employment |
| Liquidity | Available bank balance at time of request | Cash on hand is a direct safety buffer |
| Platform loyalty | Time as a Clair member, number of pay cycles completed | Longer track record = more confidence |
| Account age | How many pay cycles of history we have on record | New users with limited history are treated more cautiously |

The model combines these signals and outputs a **default probability score** between 0 and 1. A score of 0.05 means "5% chance this user defaults." A score of 0.60 means "60% chance."

**An important step: probability calibration.** Raw model scores need to be translated into meaningful probabilities before they can drive decisions. Think of it like a weather forecast — "70% chance of rain" is only useful if, historically, it actually rains 70% of the time when that forecast is issued. We apply the same logic to our model scores, ensuring that a score of 0.10 genuinely reflects a 10% default risk. Without this step, the scores can be misleading — and in our initial testing, using uncalibrated scores would have produced a threshold that let through over 90% of actual defaults undetected.

---

## How lending decisions are made

We set a **decision threshold** — a cutoff probability above which we decline the request. With our calibrated model and current cost assumptions, that threshold is **0.16**:

- Calibrated score < 0.16 → **Approve**
- Calibrated score ≥ 0.16 → **Decline**

The threshold is not arbitrary. We chose it by weighing two types of mistakes:

- **Approving a user who defaults** → we lose the full advance amount
- **Declining a user who would have repaid** → we lose the fee and damage the user relationship

We assumed a missed default costs roughly **5× more** than a declined good user. The threshold was chosen to minimise total expected cost under this assumption.

**The threshold is a business lever, not a permanent setting.** If the business wants to be more aggressive in growth, we raise the threshold slightly (approve more users, accept a marginally higher loss rate). If default rates are climbing, we lower it. Either way, no retraining is needed — we just move the cutoff.

---

## How we validated the model

Before trusting the model's scores, we ran two important checks:

**1. Time-based validation.** We trained the model on older loans and tested it on the most recent loans — simulating exactly what will happen in production, where we always predict on requests more recent than our training data. The model performed just as well on the newer loans (AUC 0.76) as it did in cross-validation (AUC 0.74), giving us confidence that it generalises to future requests and isn't just memorising the past.

**2. Calibration check.** We verified that the model's probability scores actually match real-world outcomes. A model that predicts "20% default risk" for a group of users should see roughly 20% of them default. We confirmed this holds, and applied a calibration correction where needed.

---

## Expected business benefits

**1. Reduced default losses — with concrete ROI**  
At the current threshold, the model catches approximately 435 defaults out of every ~120,000 advance requests. At an average advance of ~$158, that represents **~$69,000 in prevented losses**. The cost of declining the ~1,470 good users who are incorrectly flagged is approximately **~$11,000 in lost fee revenue**, yielding a **net benefit of ~$58,000** — a roughly 6:1 return.

The model identifies risky users better than any single rule — "decline anyone with a late payment" would be both too broad (hurting good users) and too narrow (missing first-time defaulters with no late payment history yet).

**2. Better user experience for low-risk users**  
Because the model is more precise than a blanket rule, fewer good users are unfairly declined. Users with clean histories and stable income get faster, frictionless approvals.

**3. Consistent, auditable decisions**  
Every decision is backed by a score and a clear rationale. This makes the process easier to audit, explain to regulators, and improve over time.

**4. Scalability**  
The model runs in milliseconds on any new advance request and requires no human review for the vast majority of decisions.

---

## Bonus: How much should we offer? (Safe Amount Estimation)

For approved users, instead of always granting the full requested amount, we can offer a **personalised safe amount**:

> **Safe Amount = Requested Amount × Repayment Confidence × Liquidity Factor**

- **Repayment Confidence** = 1 minus the *calibrated* default probability. Because we've calibrated our scores, this number is meaningful — a user at 10% default risk genuinely has a 90% repayment confidence.
- **Liquidity Factor** = how much available cash the user has relative to their typical paycheck. If their bank account already covers a full paycheck, the factor is 1 (no reduction). If they're cash-poor, it scales the offer down.

**Example:**
- User requests $200
- Calibrated model gives them a 10% default probability → Repayment confidence = 0.90
- Available balance is half their typical paycheck → Liquidity factor = 0.50
- Safe amount = $200 × 0.90 × 0.50 = **$90**

**What we observed in the data:** Among approved users, average safe amounts are similar across outcomes — but the formula's value is at the tails. The riskiest approved users (high default probability, low bank balance) receive meaningfully smaller offers than the safest users (low default probability, healthy balance). This graduated approach limits our exposure on borderline approvals where the binary approve/decline decision is least certain.

This approach lets us say "yes" to more users (including borderline cases) while limiting our exposure by offering a smaller amount. It turns a binary approve/decline into a graduated, risk-proportional product.

---

## What this model is not

- **Not a credit score.** It is specific to Clair wage advance repayment behavior, using signals from our own platform that traditional credit bureaus don't have.
- **Not final.** Model performance should be monitored regularly and the model retrained as user behavior, economic conditions, and our product evolve.
- **Not a replacement for human judgment** on edge cases. Complex situations (disputes, suspected fraud, account anomalies) should still involve manual review.
- **Not exempt from fair lending review.** Before using this model in production decisions, it should be audited to ensure it does not produce disparate outcomes for any groups protected under applicable consumer credit law.

---

## Next steps

1. **Shadow mode deployment**: Run the model alongside existing decisions for 4–8 weeks, scoring every request without acting on the score. Compare predictions against real outcomes before the model affects any lending decision.
2. **Monitoring dashboard**: Track model accuracy, approval rate, and default rate on a weekly basis, with alerts if any metric moves outside expected bounds.
3. **Feedback loop**: As new repayment outcomes come in, retrain the model on a regular cadence to keep it current with evolving user behavior.
4. **Fair lending audit**: Assess whether any model features produce disparate impact across demographic groups before production deployment.
