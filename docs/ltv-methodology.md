# LTV Methodology — How We Made Different Offer Types Comparable

One of the hardest problems in building a CTR × LTV ranking system is the LTV calculation itself.

You cannot multiply CTR by LTV until every offer has an LTV expressed in the same units, on the same time horizon, with the same discount rate. This sounds obvious. In practice it requires aligning finance, product, and data science across teams that have never had to agree on a single number before.

Here's how LTV was defined for each offer type in the Meridian Commerce personalization system.

---

## The problem: structurally different revenue models

Each offer type generates revenue through a completely different mechanism:

| Offer | Revenue mechanism | Time horizon | Certainty |
|-------|-----------------|--------------|-----------|
| Gift card | One-time margin + breakage | Immediate | High |
| Shop with points | Reduced churn, loyalty engagement | 12 months | Medium |
| Pay with Card | Interchange share | Per transaction | High |
| Split Pay (BNPL) | Merchant fee + repeat usage | 6 months | Medium |
| Meridian Visa signup | Acquisition fee + spend lift + membership | 24 months | Lower |
| Meridian Visa referral | Same as signup, peer-acquired | 24 months | Lower |

Comparing these directly is like comparing apples to present-value-discounted-apple-futures. You need a framework.

---

## The framework: present value of incremental revenue at 12 months

We defined LTV as: **the expected incremental revenue attributable to a customer accepting this offer, measured at 12 months, discounted to present value.**

Key decisions baked into this definition:

**12 months** — long enough to capture meaningful downstream behavior for credit card products, short enough to limit forecast uncertainty. 24-month LTVs for credit card products would be higher but less reliable.

**Incremental** — the counterfactual matters. If a customer was going to shop regardless, the gift card's LTV is only the margin on the card itself, not their total spend. For the co-branded card, incrementality is higher because the cashback benefit demonstrably drives spend that wouldn't otherwise have happened.

**Present value** — a dollar of revenue in 12 months is worth less than a dollar today. We used a standard discount rate agreed with finance.

---

## LTV by offer type

### Gift card: LTV = 1.0× (baseline)

Revenue: card margin (approximately 2-3% of face value) + breakage (cards never redeemed, typically 10-15% of volume)

Why it's the baseline: immediate, certain, simple to calculate. Everything else is expressed as a multiple of gift card LTV.

### Shop with points: LTV = 0.8×

Points redemption is slightly below baseline because it represents redemption of a liability (outstanding points) rather than new revenue. The benefit is churn reduction — customers who engage with points stay on the platform longer. But the direct revenue impact is negative (we're paying out points), with the offset being retention.

### Pay with Card: LTV = 1.2×

Interchange share per transaction. Predictable, per-use revenue. Higher than gift card because the relationship is ongoing — a customer who saves their card will use it across many future purchases.

### Split Pay (BNPL): LTV = 1.8×

Merchant fee per transaction (the BNPL provider pays a referral fee) + cart size lift (customers spend more when they can split payments) + repeat usage over the following 6 months. The cart size lift is the largest component.

### Meridian Visa signup: LTV = 4.0×

Three revenue streams:

1. **Card acquisition fee** — a one-time fee paid by the card-issuing bank for each approved application. This is the largest single component.

2. **Spend lift** — cardholders with 5% cashback on Meridian purchases spend measurably more than they did before the card, and more than comparable non-cardholders. This is calculated from cohort analysis: matched pairs of customers who got the card vs. didn't, controlling for pre-card spend level.

3. **Membership retention** — customers upgrade to or maintain premium membership to unlock the full 5% cashback rate (vs 2% without premium). This creates a compounding flywheel: card → premium → more cashback → more spend → more premium renewal.

The 4× LTV versus gift card reflects these three streams compounding over a 12-month window.

### Meridian Visa referral: LTV = 3.5×

Same revenue streams as direct signup, but applied to the referred customer. Slightly lower than direct signup for two reasons: (1) referred customers have slightly lower average spend than directly-acquired customers in the first year, and (2) referral programs have variable conversion rates from referral to actual signup.

---

## The governance process

Getting all six LTV numbers agreed required:

- **Finance** to validate the discount rate, the incremental spend attribution methodology, and the acquisition economics for each offer type
- **Data science** to run the cohort analyses for spend lift and retention
- **Product** to agree on the 12-month horizon and the definition of "incremental"
- **Risk** to confirm that approval rate assumptions embedded in the LTV calculations were consistent with their models

This process took longer than building the model. It was also the most important part — because without agreed LTV numbers, the objective function is just CTR with a multiplier that nobody trusts.

The output was a shared document, reviewed quarterly, that defined the LTV for each offer type and the methodology behind each number. When the objective function produced a ranking that surprised any team, the document was the reference point for the conversation.

---

*This document describes methodology used in the Meridian Commerce demo system. Numbers are illustrative. Frameworks reflect real patterns from production personalization systems.*
