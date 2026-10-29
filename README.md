## Step 1 — Understanding the Business Problem

### 1. What exactly is being predicted?
Whether an **existing credit card client** will default on (fail to pay) their next month's payment.

### 2. When is the prediction supposed to be made?
The prediction is made at the end of a client's most recent billing cycle, using their trailing history up to that point — **not** before the card is issued. It's about flagging existing cardholders, not loan-approval decisions.

### 3. What information is available before the next-month outcome?
Everything in the dataset except the target itself:
- Client's credit limit
- Demographic details
- Repayment status, bill amounts, and payment amounts for the past 6 months

All of these describe periods strictly *before* the month being predicted, so none of them leak the outcome.

### 4. What does `default = 1` and `default = 0` mean?

| Value | Meaning |
|-------|---------|
| `1` | Client **did not** pay their next month's bill (defaulted) |
| `0` | Client **paid** on time |

### 5. What business action could follow a high-risk prediction?
The risk team could flag the client for **early intervention** before the client actually misses a payment.

### 6. What could cause data leakage?
Initially suspected the `PAY_` columns created leakage, but tracing the timeline below shows none of the `PAY_`, `BILL_AMT`, or `PAY_AMT` columns describe the target month — they're all from *before* it, so the raw columns don't leak.

PAY_6 → PAY_5 → PAY_4 → PAY_3 → PAY_2 → PAY_0 → [target month]

The real leakage risk is in **preprocessing**: fitting a scaler on train + test combined (instead of splitting first) lets test-set information leak into how training data is represented, making evaluation scores look better than they really are. In production, a new client's data is only ever transformed using the mean/std learned during training — there's no "whole dataset" to re-fit on.
