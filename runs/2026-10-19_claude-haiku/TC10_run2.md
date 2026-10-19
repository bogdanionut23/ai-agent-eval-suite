# TC10: Daily transfer limit (5,000 EUR per calendar day)

## Requirement

A customer can send bank transfers of up to 5,000 EUR in total per calendar day. A transfer that would exceed the remaining daily amount is rejected in full.

## Working assumptions

These are stated so the tests can run. Each one is also listed as an open question below.

- The limit is 5,000.00 EUR per customer per calendar day, and "up to" means 5,000.00 itself is allowed (Q3, Q11).
- Only transfers that were accepted count toward the total. Rejected attempts do not (Q4).
- Amounts are exact decimals in EUR with two decimal places.
- Default precondition for every test: customer C1 is active, has a funded EUR account with enough balance so the balance is never the reason for a rejection, and has no other transfers today unless the test says otherwise.

## Test cases

### Positive

**TC10-01: Single transfer well within the limit is accepted**
- Preconditions: C1 has used 0.00 EUR today.
- Steps: 1. Submit a transfer of 100.00 EUR to a valid beneficiary.
- Expected result: The transfer is accepted and 100.00 EUR is debited. Today's used total is 100.00 EUR and the remaining amount is 4,900.00 EUR.

**TC10-02: Several transfers that together stay within the limit are all accepted**
- Preconditions: C1 has used 0.00 EUR today.
- Steps: 1. Submit 1,000.00 EUR. 2. Submit 2,000.00 EUR. 3. Submit 1,500.00 EUR. All on the same day.
- Expected result: All three transfers are accepted. Today's used total is 4,500.00 EUR and the remaining amount is 500.00 EUR.

**TC10-03: Transfer that fills the limit to exactly 5,000.00 EUR is accepted**
- Preconditions: C1 has used 4,500.00 EUR today.
- Steps: 1. Submit 500.00 EUR.
- Expected result: The transfer is accepted. Today's used total is 5,000.00 EUR and the remaining amount is 0.00 EUR.

**TC10-04: A single transfer of exactly 5,000.00 EUR is accepted when nothing has been used (upper boundary)**
- Preconditions: C1 has used 0.00 EUR today.
- Steps: 1. Submit a transfer of 5,000.00 EUR.
- Expected result: The transfer is accepted. Remaining amount is 0.00 EUR.

**TC10-05: Transfer that is exactly the remaining amount is accepted (boundary)**
- Preconditions: C1 has used 4,500.00 EUR today, so 500.00 EUR remains.
- Steps: 1. Submit a transfer of 500.00 EUR.
- Expected result: The transfer is accepted. Remaining amount is 0.00 EUR.

**TC10-06: Limit is tracked to the cent, and the last cent is still usable (boundary)**
- Preconditions: C1 has used 4,999.98 EUR today (three accepted transfers of 1,666.66 EUR each).
- Steps: 1. Submit 0.02 EUR. 2. Submit 0.01 EUR.
- Expected result: Step 1 is accepted; remaining becomes 0.00 EUR. Step 2 is rejected in full (see TC10-07 for the rejection rules).

**TC10-07: After a rejected transfer, a smaller transfer that fits is still accepted**
- Preconditions: C1 has used 4,500.00 EUR today.
- Steps: 1. Submit 600.00 EUR (rejected). 2. Submit 500.00 EUR.
- Expected result: Step 1 is rejected. Step 2 is accepted. Remaining amount is 0.00 EUR.

**TC10-08: Limit resets when the next calendar day starts**
- Preconditions: C1 has used 5,000.00 EUR on day D. Day boundary is defined as in Q1.
- Steps: 1. On day D shortly before midnight, submit 1.00 EUR. 2. Just after midnight on day D+1, submit 5,000.00 EUR.
- Expected result: Step 1 is rejected in full. Step 2 is accepted. Today's used total for D+1 is 5,000.00 EUR.

**TC10-09: Transfers from the previous day do not reduce today's limit**
- Preconditions: On day D-1, C1 had 4,900.00 EUR of accepted transfers. Today is day D and C1 has used 0.00 EUR.
- Steps: 1. On day D, submit 4,900.00 EUR.
- Expected result: The transfer is accepted. Today's used total is 4,900.00 EUR and the remaining amount is 100.00 EUR.

**TC10-10: Limits are independent per customer**
- Preconditions: C1 has used 5,000.00 EUR today. C2 is a different active customer with used 0.00 EUR today.
- Steps: 1. C2 submits 5,000.00 EUR.
- Expected result: C2's transfer is accepted. C1's used total is unchanged at 5,000.00 EUR.

**TC10-11: Limit applies across all accounts of the same customer (depends on Q3)**
- Preconditions: C1 has two EUR accounts, A and B. C1 has already sent 3,000.00 EUR from account A today.
- Steps: 1. From account B, submit 2,500.00 EUR. 2. From account B, submit 2,000.00 EUR.
- Expected result: Step 1 is rejected in full because only 2,000.00 EUR remains. Step 2 is accepted. Remaining amount is 0.00 EUR.

### Negative

**TC10-12: Transfer that exceeds the remaining amount is rejected in full (no partial execution)**
- Preconditions: C1 has used 4,500.00 EUR today.
- Steps: 1. Submit a transfer of 501.00 EUR.
- Expected result: The transfer is rejected. No money is debited, and no partial transfer of 500.00 EUR is made or queued. Used total stays at 4,500.00 EUR and remaining stays at 500.00 EUR.

**TC10-13: Transfer that exceeds the remaining amount by one cent is rejected in full (boundary)**
- Preconditions: C1 has used 4,500.00 EUR today.
- Steps: 1. Submit a transfer of 500.01 EUR.
- Expected result: The transfer is rejected in full. Used total stays at 4,500.00 EUR.

**TC10-14: A single transfer of 5,000.01 EUR is rejected in full (upper boundary, just above)**
- Preconditions: C1 has used 0.00 EUR today.
- Steps: 1. Submit a transfer of 5,000.01 EUR.
- Expected result: The transfer is rejected. No money is debited, and no partial transfer of 5,000.00 EUR is made. Used total stays at 0.00 EUR.

**TC10-15: Rejected attempts do not consume the daily limit (depends on Q4)**
- Preconditions: C1 has used 0.00 EUR today.
- Steps: 1. Submit a transfer of 5,000.01 EUR (rejected). 2. Submit a transfer of 5,000.00 EUR.
- Expected result: Step 1 is rejected. Step 2 is accepted, because the rejected attempt did not count toward the total.

**TC10-16: When two transfers are submitted at the same time and together exceed the limit, only one is accepted (depends on Q13)**
- Preconditions: C1 has used 0.00 EUR today. Two transfers of 3,000.00 EUR each are prepared.
- Steps: 1. Submit both transfers at the same moment, using two sessions or two parallel API calls.
- Expected result: Exactly one transfer is accepted and the other is rejected in full. The total accepted for today is 3,000.00 EUR, never more than 5,000.00 EUR. Remaining amount is 2,000.00 EUR.

**TC10-17: Rejection tells the customer that the daily limit was exceeded (depends on Q8)**
- Preconditions: C1 has used 4,500.00 EUR today.
- Steps: 1. Submit a transfer of 501.00 EUR.
- Expected result: The transfer is rejected. The customer sees a message or error code that identifies the daily limit as the reason. The exact wording and whether the remaining amount is shown are as defined in Q8.

## Open questions

1. **Time zone and day boundary.** Which time zone defines the "calendar day": the customer's local time, the bank's time zone, or UTC? Exactly when does the day start and end? This affects TC10-08 and TC10-09.
2. **Point of counting.** Does a transfer count toward the total when it is submitted, when it is validated, when it is executed or settled, or on its value date? This matters for scheduled or future-dated transfers.
3. **Scope of the limit.** Is the limit per customer across all accounts, or per account? TC10-11 assumes per customer. The requirement says "a customer", so this needs confirming.
4. **Rejected, failed or errored attempts.** Do rejected transfers, or transfers that fail on a technical error after the limit check, count toward the total? TC10-15 assumes they do not.
5. **Cancelled or reversed transfers.** Does cancelling or reversing an accepted transfer on the same day return its amount to the daily limit?
6. **Non-EUR transfers.** Does the limit apply to transfers in other currencies? If so, is the check on the original amount or on the EUR equivalent, and which exchange rate and time is used?
7. **Fees.** Are transfer fees counted toward the 5,000 EUR?
8. **Rejection message and error code.** What does the customer see, and what error code does each channel (online, mobile, API, branch) return? Should the message show the remaining amount? TC10-17 depends on this.
9. **Future-dated transfers.** If a transfer is scheduled for a future date, is it counted on the day it is submitted or on its execution day? Can a customer schedule more than 5,000 EUR across several days?
10. **Own-account and internal transfers.** Are transfers between a customer's own accounts, or other internal transfers, covered by this limit?
11. **Boundary and validation rules.** The requirement says "up to 5,000 EUR", which I read as 5,000.00 being allowed. Please confirm. Please also confirm the minimum amount, how zero and negative amounts are handled, and the maximum number of decimal places.
12. **Order of rejection reasons.** If a transfer breaks the daily limit and also fails another check (for example, insufficient balance or a blocked beneficiary), which reason is reported?
13. **Concurrent submissions.** When transfers are submitted at the same time, how is the order decided (submission time, processing order, or something else)? TC10-16 assumes the limit check is atomic, so the total can never exceed 5,000.00 EUR.
14. **Channel consistency.** Does the same limit apply on every channel (online, mobile, branch, API), and is usage shared across channels?
15. **Configurability.** Can the limit differ by customer segment, or be changed or overridden by an operator? If so, what happens to usage already counted when the limit changes during the day?
16. **Other limits.** Is there a per-transaction maximum or any other limit that interacts with the daily limit, and which check runs first?
