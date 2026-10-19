# Test cases: daily bank transfer limit (5,000 EUR per calendar day)

Requirement: A customer can send bank transfers of up to 5,000 EUR in total per calendar day. A transfer that would exceed the remaining daily amount is rejected in full.

## Assumptions (kept minimal, see open questions)

- "Up to 5,000 EUR" is read as inclusive: a day's total of exactly 5,000.00 EUR is allowed.
- "Rejected in full" is read as no partial execution: nothing is sent and the remaining daily amount is unchanged.
- Amounts are in EUR with 2 decimals.
- Which time zone defines "calendar day" and which transfers count toward the limit are not specified (see open questions). The cases below that depend on them are marked **[BLOCKED by OQ-x]**. Their expected results cannot be fixed until the question is answered.

## Positive cases

### TC-01: Single transfer well below the limit
- Preconditions: Customer is logged in, has sufficient balance (at least 1,000 EUR), and has sent no transfers today.
- Steps: 1. Create a transfer of 1,000.00 EUR to a valid beneficiary. 2. Confirm it.
- Expected: The transfer is accepted and executed. Remaining daily amount becomes 4,000.00 EUR.

### TC-02: Several transfers whose total stays below the limit
- Preconditions: Sufficient balance (at least 4,000 EUR), no transfers today.
- Steps: 1. Send 1,500.00 EUR. 2. Send 1,500.00 EUR. 3. Send 1,000.00 EUR.
- Expected: All three are accepted. Total sent is 4,000.00 EUR and the remaining amount is 1,000.00 EUR.

### TC-03: Accumulated transfers reach exactly the limit
- Preconditions: Sufficient balance (at least 5,000 EUR), no transfers today.
- Steps: 1. Send 3,000.00 EUR. 2. Send 2,000.00 EUR.
- Expected: Both are accepted. Total is 5,000.00 EUR and the remaining amount is 0.00 EUR.

## Boundary cases

### TC-04: Single transfer of exactly 5,000.00 EUR
- Preconditions: Balance at least 5,000 EUR, no transfers today.
- Steps: 1. Send 5,000.00 EUR.
- Expected: Accepted (limit is inclusive, per assumption). Remaining amount is 0.00 EUR.

### TC-05: Single transfer of 5,000.01 EUR (one cent over)
- Preconditions: Balance at least 5,001 EUR, no transfers today.
- Steps: 1. Send 5,000.01 EUR.
- Expected: Rejected in full with a clear limit-exceeded message. No money is moved and the remaining amount stays 5,000.00 EUR.

### TC-06: Transfer of 4,999.99 EUR, then 0.01 EUR
- Preconditions: Sufficient balance, no transfers today.
- Steps: 1. Send 4,999.99 EUR. 2. Send 0.01 EUR.
- Expected: Both accepted. Total is exactly 5,000.00 EUR and the remaining amount is 0.00 EUR.

### TC-07: Transfer of 4,999.99 EUR, then 0.02 EUR
- Preconditions: Sufficient balance, no transfers today.
- Steps: 1. Send 4,999.99 EUR. 2. Send 0.02 EUR.
- Expected: Step 1 accepted. Step 2 rejected in full (it would bring the total to 5,000.01). Remaining amount stays 0.01 EUR.

### TC-08: Transfer exactly equal to the remaining amount
- Preconditions: Customer already sent 3,250.50 EUR today.
- Steps: 1. Send 1,749.50 EUR.
- Expected: Accepted. Remaining amount is 0.00 EUR.

### TC-09: Transfer one cent above the remaining amount
- Preconditions: Customer already sent 3,250.50 EUR today (remaining 1,749.50).
- Steps: 1. Send 1,749.51 EUR.
- Expected: Rejected in full. No partial transfer of 1,749.50 is executed. Remaining stays 1,749.50 EUR.

### TC-10: Minimum transfer amount
- Preconditions: No transfers today.
- Steps: 1. Send 0.01 EUR.
- Expected: Accepted if the system's minimum transfer is 0.01, and the remaining amount becomes 4,999.99 EUR. (The system minimum is not defined, see OQ-8.)

## Negative cases

### TC-11: Rejection leaves state untouched
- Preconditions: Customer already sent 4,000.00 EUR today, with a known balance B.
- Steps: 1. Attempt to send 1,500.00 EUR (exceeds remaining 1,000.00). 2. Check balance, transaction history and remaining daily amount.
- Expected: Transfer rejected in full. Balance is still B, no pending or partial transaction appears, remaining amount is still 1,000.00 EUR.

### TC-12: Rejected transfer does not consume the limit
- Preconditions: Same as TC-11.
- Steps: 1. Attempt 1,500.00 EUR (rejected). 2. Send 1,000.00 EUR.
- Expected: Step 1 rejected. Step 2 accepted, because the rejected amount was not counted.

### TC-13: Smaller transfer still allowed after a rejection
- Preconditions: Customer already sent 4,500.00 EUR today.
- Steps: 1. Attempt 600.00 EUR (rejected). 2. Send 500.00 EUR.
- Expected: Step 1 rejected in full. Step 2 accepted.

### TC-14: Any further transfer after the limit is fully used
- Preconditions: Customer has sent exactly 5,000.00 EUR today.
- Steps: 1. Attempt to send 0.01 EUR.
- Expected: Rejected in full with the limit-exceeded message.

### TC-15: Zero and negative amounts
- Preconditions: Logged in, no transfers today.
- Steps: 1. Attempt 0.00 EUR. 2. Attempt -100.00 EUR.
- Expected: Both rejected as invalid amounts (validation error, not a limit error). The remaining daily amount is unchanged. (Exact message is not specified in the requirement.)

### TC-16: Insufficient account balance but within limit
- Preconditions: Balance 200 EUR, no transfers today.
- Steps: 1. Attempt to send 1,000.00 EUR.
- Expected: Rejected for insufficient funds. The attempt does not reduce the remaining daily amount (still 5,000.00 EUR).

### TC-17: Amount with more than 2 decimal places
- Preconditions: No transfers today.
- Steps: 1. Attempt to send 100.005 EUR.
- Expected: Rejected by input validation or rounded per system rules. The limit calculation must not be skewed by sub-cent amounts. (Rounding behaviour not specified, see OQ-8.)

## Concurrency and consistency cases

### TC-18: Two simultaneous transfers that together exceed the limit
- Preconditions: Customer already sent 2,000.00 EUR today (remaining 3,000.00). Two sessions (for example web and mobile) logged in as the same customer.
- Steps: 1. In both sessions at the same moment, submit a 2,000.00 EUR transfer.
- Expected: Exactly one is accepted and the other is rejected in full. Total for the day never exceeds 5,000.00 EUR (no race condition).

### TC-19: Double submission of the same transfer
- Preconditions: Remaining amount 5,000.00 EUR.
- Steps: 1. Submit a 3,000.00 EUR transfer and click or tap confirm twice quickly.
- Expected: Behaviour follows the system's idempotency rules. Either only one 3,000.00 EUR transfer executes, or the second is rejected by the limit. The daily total never goes above 5,000.00 EUR.

### TC-20: Remaining amount shown to customer is consistent
- Preconditions: Customer sent 1,234.56 EUR today.
- Steps: 1. View the remaining daily amount in the app or web, then in another channel if available.
- Expected: Shows 3,765.44 EUR everywhere. **[Display of the remaining amount is itself not stated in the requirement, see OQ-9.]**

## Calendar-day cases (depend on open questions)

### TC-21: Limit resets on the next calendar day **[BLOCKED by OQ-1]**
- Preconditions: Customer has used 5,000.00 EUR on day D.
- Steps: 1. After the day boundary on D+1, send 5,000.00 EUR.
- Expected: Accepted, because the new day has a fresh 5,000.00 EUR allowance. The exact reset moment depends on the defined time zone.

### TC-22: Transfers just before and just after midnight **[BLOCKED by OQ-1]**
- Preconditions: Customer has used 5,000.00 EUR before midnight (in the defined time zone).
- Steps: 1. Send 23:59:59 on day D: 1.00 EUR (expect rejection). 2. Send 00:00:00 on D+1: 1.00 EUR.
- Expected: Step 1 rejected. Step 2 accepted. Boundary timestamp assignment (request time versus execution time) depends on OQ-2.

### TC-23: Daylight-saving change day **[BLOCKED by OQ-1]**
- Preconditions: Day with a 23-hour or 25-hour length in the defined time zone.
- Steps: 1. Send transfers around the DST change and across midnight.
- Expected: The day is still counted as one calendar day with one 5,000.00 EUR limit. Exact behaviour depends on the definition of "day".

### TC-24: Scheduled (future-dated) transfer **[BLOCKED by OQ-2]**
- Preconditions: Customer schedules a 4,000.00 EUR transfer today for tomorrow.
- Steps: 1. Check the remaining amount today. 2. Check it tomorrow after execution.
- Expected: Unknown whether the amount counts toward today (creation date) or tomorrow (execution date).

### TC-25: Transfer in a currency other than EUR **[BLOCKED by OQ-4]**
- Preconditions: Customer has a non-EUR account.
- Steps: 1. Send an amount that is just above and just below 5,000 EUR at the current rate.
- Expected: Unknown which exchange rate and rounding apply.

### TC-26: Failed or reversed transfer after acceptance **[BLOCKED by OQ-5]**
- Preconditions: A transfer was accepted but later failed or was returned by the receiving bank.
- Steps: 1. Check the remaining daily amount.
- Expected: Unknown whether the amount is restored.

## Open questions

- OQ-1: What defines a "calendar day": which time zone (customer's, bank's, UTC, CET/CEST) and what is the exact reset time? How are DST changes handled?
- OQ-2: Which date is used for counting: the time the customer submits the transfer, or the execution/settlement date? How are scheduled, future-dated and standing orders treated?
- OQ-3: Is the limit per customer (across all their accounts and cards) or per account? Do joint accounts and business accounts share it?
- OQ-4: Which transfers count toward the limit? Only bank transfers in EUR, or also SEPA instant, international, foreign-currency, own-account (internal) transfers, card payments and ATM withdrawals? If other currencies count, which exchange rate and rounding are used?
- OQ-5: If a transfer later fails, is cancelled, is recalled or is returned, is the amount restored to the remaining daily limit? What about pending or unconfirmed transfers (for example awaiting strong authentication)?
- OQ-6: Is "up to 5,000 EUR" inclusive (exactly 5,000.00 allowed)? The wording suggests yes, but it should be confirmed.
- OQ-7: What should the customer see when a transfer is rejected: required message text, error code, whether the remaining amount is displayed, and whether the rejection is logged or notified?
- OQ-8: What are the minimum transfer amount, the amount precision (2 decimals), and the rounding rule for inputs with more decimals?
- OQ-9: Must the remaining daily amount be visible to the customer (UI or API)? Is there an API contract for the error response?
- OQ-10: Can the limit be changed by the customer, by support, or per customer segment (for example higher limits, temporary increases, regulatory or fraud-related reductions)? Is 5,000 EUR a fixed default?
- OQ-11: How must concurrent requests be handled (strictness of the guarantee), and what is the expected idempotency behaviour for repeated submissions?
- OQ-12: Are there exemptions (for example transfers to the customer's own accounts, tax or payroll payments, bank-initiated transfers)?
