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

The model combines these signals and outputs a **default probability score** between 0 and 1. A score of 0.05 means "5% chance this user defaults." A score of 0.60 means "60% chance." We apply a calibration step to ensure these scores match real-world default rates — so a 10% score genuinely means 10% risk, not just a relative ranking.

---

## How lending decisions are made

Rather than a binary approve/deny, we use a **three-tier decision framework**:

| Tier | Who | Action |
|---|---|---|
| **Full approval** | Low-risk users | Grant the requested amount |
| **Capped approval** | Borderline users | Approve at a reduced cap (e.g. $50) |
| **Deny** | High-risk users | Decline with specific reasons |

**Why tiers instead of approve/deny?** Denying a financially stressed worker outright carries high churn risk — they're unlikely to come back. But offering them $50 instead of $200 keeps them on the platform, limits our exposure to at most $50 if they default, and gives them a chance to build repayment history toward full approval. The economics are dramatically better than a binary approach: in our modeling, the tiered approach produces **~$73K net benefit** vs. **negative ROI** for binary approve/deny under realistic churn assumptions.

| Scenario | Binary net ROI | Tiered net ROI | Improvement |
|---|---|---|---|
| Low churn (15% deny / 3% cap) | -$161K | $117K | +$278K |
| **Mid churn (30% deny / 5% cap)** | **-$168K** | **$73K** | **+$241K** |
| High churn (50% deny / 10% cap) | -$194K | -$126K | +$69K |

Under the recommended mid-churn scenario: ~58% of users get full approval, ~41% get a capped offer, and only ~2% are denied outright.

**Three business levers drive every decision.**
 (1) The risk model itself — scores improve as we retrain on new repayment data. 
 (2) Churn sensitivity — the gap between churn rates for denied vs. capped users determines how aggressively to offer reduced amounts instead of denials. 
 (3) Lifetime value (LTV) estimates — higher LTV makes churn costlier, shifting thresholds toward approval. Both thresholds (cap and deny) can be adjusted without retraining; the key unknowns to measure during shadow deployment are actual churn rates and per-user LTV.

**What happens when a user is denied?** If wage advances are classified as credit, regulations (ECOA/Reg B) require that we tell declined users the specific reasons — for example, "your advance amount was high relative to your recent paychecks." The model supports this: it produces a ranked list of the top factors behind each individual decision. Beyond compliance, this transparency gives users a clear path back — their score updates with every new paycheck and on-time repayment.

---

## How we validated the model

We tested the model three ways: (1) ensuring users with multiple loans are never split across training and test sets, so accuracy reflects performance on genuinely new users; (2) training on older loans and testing on the most recent — the model performed equally well, confirming it generalises forward in time; (3) verifying that predicted probabilities match actual default rates, so the scores are trustworthy enough to drive dollar-value decisions.

---

## Expected business benefits

**1. Better economics through tiered decisions**  
The tiered framework produces ~$73K net benefit per ~120K requests under mid-churn assumptions — a $241K improvement over binary approve/deny. The key driver: capping borderline users at $50 instead of denying them preserves fee revenue and customer relationships while limiting default exposure.

**2. Near-zero denial rate**  
Only ~2% of users are denied outright. The remaining borderline users (~41%) receive a capped $50 offer instead of the full amount — a better experience that still protects the business.

**3. Natural onboarding ramp**  
Capped users who repay build history toward full approval, creating a self-reinforcing cycle: good behavior → higher limit → more engagement → more fee revenue.

**4. Consistent, auditable decisions**  
Every decision is backed by a score and a clear rationale — easier to audit, explain to regulators, and improve over time.

---

## Next steps

1. **Shadow mode + fair lending audit** (parallel, 4–8 weeks): Score every request without acting on it; simultaneously audit for disparate impact across demographic groups. Both must pass before the model affects any lending decision.
2. **Measure churn rates**: Track churn for capped vs. denied vs. approved users — the gap between cap-churn and deny-churn is the entire economic justification for the tiered approach.
3. **Test cap amount sensitivity**: $50 is a starting point, but user response to different cap levels ($25/$50/$75) likely varies by risk tier. Compliance-aware A/B testing would reveal how cap amount affects churn, default rates, and satisfaction — the single highest-leverage experiment for optimizing this framework.
4. **Monitoring dashboard**: Weekly accuracy, approval rate, and default rate tracking with drift alerts.
5. **Feedback loop**: Retrain on a regular cadence as new repayment outcomes come in.
