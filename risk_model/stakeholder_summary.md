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
| Platform loyalty | Time as a Clair member, number of pay cycles used | Longer track record = more confidence |

The model combines these signals and outputs a **default probability score** between 0 and 1. A score of 0.05 means "5% chance this user defaults." A score of 0.60 means "60% chance."

---

## How lending decisions are made

We set a **decision threshold** — a cutoff probability above which we decline the request. For example, with a threshold of 0.35:

- Score < 0.35 → **Approve**
- Score ≥ 0.35 → **Decline**

The threshold is not arbitrary. We calibrated it by weighing two types of mistakes:

- **Approving a user who defaults** → we lose the full advance amount
- **Declining a user who would have repaid** → we lose the fee and damage the user relationship

We assumed a missed default costs roughly **5× more** than a declined good user. The threshold was chosen to minimise total expected cost under this assumption. If the business changes its view of the cost ratio, the threshold can be adjusted with a single parameter — no retraining needed.

---

## Expected business benefits

**1. Reduced default losses**  
By catching high-risk requests before they're approved, we avoid a meaningful share of defaults. The model's accuracy (ROC AUC) comfortably exceeds random chance, meaning it reliably identifies risky users better than a coin flip — or a simple rule like "decline anyone with a late payment."

**2. Better user experience for low-risk users**  
Because the model is more precise than a blanket rule, fewer good users are unfairly declined. Users with clean histories get faster, frictionless approvals.

**3. Consistent, auditable decisions**  
Every decision is backed by a score and a clear rationale. This makes the process easier to audit, explain to regulators, and improve over time.

**4. Scalability**  
The model runs in milliseconds on any new advance request. It requires no human review for the majority of decisions.

---

## Bonus: How much should we offer? (Safe Amount Estimation)

For approved users, instead of always granting the full requested amount, we can offer a **personalised safe amount**:

> **Safe Amount = Requested Amount × Repayment Confidence × Liquidity Factor**

- **Repayment Confidence** = 1 minus the default probability. A user with a 5% default risk has a 95% repayment confidence.
- **Liquidity Factor** = how much available cash the user has relative to their typical paycheck. If their bank account already covers a full paycheck, this factor is 1 (no reduction). If they're nearly empty, it scales down the offer.

**Example:**
- User requests $200
- Model gives them a 10% default probability → Repayment confidence = 0.90
- Available balance is half their typical paycheck → Liquidity factor = 0.5
- Safe amount = $200 × 0.90 × 0.50 = **$90**

This approach lets us say "yes" to more users (even borderline ones) while limiting our exposure by offering a smaller amount. It turns a binary approve/decline into a graduated, risk-proportional product.

---

## What this model is not

- It is not a credit score. It's specific to Clair wage advance repayment behavior.
- It is not final. Model performance should be monitored monthly and retrained as user behavior and economic conditions evolve.
- It does not replace human judgment for edge cases. Complex situations (disputes, fraud signals) should still involve manual review.

---

## Next steps

1. **Shadow mode deployment**: Run the model alongside existing decisions for 4–8 weeks to validate predictions against real outcomes before going live.
2. **Monitoring dashboard**: Track model accuracy, approval rate, and default rate weekly to catch drift early.
3. **Feedback loop**: As new repayment outcomes come in, retrain the model quarterly to keep it current.
