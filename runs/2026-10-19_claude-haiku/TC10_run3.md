# TC10 - Test cases: daily bank transfer limit

**Requirement:** A customer can send bank transfers of up to 5,000 EUR in total per calendar day. A transfer that would exceed the remaining daily amount is rejected in full.

## Assumptions used for the test design

These are not stated in the requirement. Each test case below assumes them; the open questions at the end should confirm them.

- "Up to 5,000 EUR" is inclusive: a total of exactly 5,000.00 EUR is allowed.
- The limit counts successful transfers only, and the amount counted is the transfer amount (no fees).
- A rejected transfer does not count toward the daily total.
- Amounts have two decimal places (cents).

## Test cases

### TC10-01 - Single transfer well below the limit is accepted

- **Type:** Positive
- **Preconditions:** Customer is active and has sufficient balance. No transfers have been made today (daily total = 0 EUR).
- **Steps:**
  1. Submit a transfer of 1,000.00 EUR.
- **Expected result:** The transfer is accepted and executed. The remaining daily amount becomes 4,000.00 EUR.

### TC10-02 - Multiple transfers that together stay within the limit are all accepted

- **Type:** Positive
- **Preconditions:** Customer is active with sufficient balance. Daily total = 0 EUR.
- **Steps:**
  1. Submit a transfer of 1,500.00 EUR.
  2. Submit a transfer of 2,000.00 EUR.
  3. Submit a transfer of 1,000.00 EUR.
- **Expected result:** All three transfers are accepted. The daily total is 4,500.00 EUR and the remaining daily amount is 500.00 EUR.

### TC10-03 - Transfer that brings the daily total exactly to 5,000 EUR is accepted (upper boundary, inclusive)

- **Type:** Boundary
- **Preconditions:** Customer is active with sufficient balance. Daily total = 4,000.00 EUR (from earlier transfers today).
- **Steps:**
  1. Submit a transfer of 1,000.00 EUR.
- **Expected result:** The transfer is accepted. The daily total is 5,000.00 EUR and the remaining daily amount is 0.00 EUR.

### TC10-04 - Single transfer of exactly 5,000.00 EUR on a day with no prior transfers is accepted

- **Type:** Boundary
- **Preconditions:** Customer is active with sufficient balance (at least 5,000.00 EUR). Daily total = 0 EUR.
- **Steps:**
  1. Submit a transfer of 5,000.00 EUR.
- **Expected result:** The transfer is accepted. The remaining daily amount is 0.00 EUR.

### TC10-05 - Single transfer of 5,000.01 EUR is rejected in full (boundary, just above limit)

- **Type:** Boundary / Negative
- **Preconditions:** Customer is active with sufficient balance. Daily total = 0 EUR.
- **Steps:**
  1. Submit a transfer of 5,000.01 EUR.
- **Expected result:** The transfer is rejected in full. No money is debited and no partial transfer is executed. The customer receives a rejection message indicating the daily limit. The daily total remains 0 EUR.

### TC10-06 - Transfer that would exceed the remaining daily amount is rejected in full

- **Type:** Negative
- **Preconditions:** Customer is active with sufficient balance. Daily total = 4,000.00 EUR.
- **Steps:**
  1. Submit a transfer of 1,000.01 EUR.
- **Expected result:** The transfer is rejected in full. No amount is partially transferred (e.g., 1,000.00 EUR is not sent). The customer receives a rejection message. The daily total remains 4,000.00 EUR.

### TC10-07 - Rejected transfer does not reduce the remaining daily amount; a later transfer within the remaining amount is still accepted

- **Type:** Negative / Positive
- **Preconditions:** Customer is active with sufficient balance. Daily total = 4,000.00 EUR.
- **Steps:**
  1. Submit a transfer of 1,500.00 EUR (exceeds remaining 1,000.00 EUR). Observe rejection.
  2. Submit a transfer of 1,000.00 EUR.
- **Expected result:** Step 1 is rejected in full. Step 2 is accepted. The daily total is 5,000.00 EUR and the remaining daily amount is 0.00 EUR.

### TC10-08 - Transfer submitted when the daily total is already exactly 5,000 EUR is rejected

- **Type:** Boundary / Negative
- **Preconditions:** Customer is active with sufficient balance. Daily total = 5,000.00 EUR (limit reached).
- **Steps:**
  1. Submit a transfer of 0.01 EUR.
- **Expected result:** The transfer is rejected in full. The daily total remains 5,000.00 EUR.

### TC10-09 - Smallest valid transfer amount is accepted when remaining amount allows it (lower boundary)

- **Type:** Boundary
- **Preconditions:** Customer is active with sufficient balance. Daily total = 4,999.99 EUR (remaining 0.01 EUR).
- **Steps:**
  1. Submit a transfer of 0.01 EUR.
- **Expected result:** The transfer is accepted. The daily total is 5,000.00 EUR and the remaining daily amount is 0.00 EUR.
- **Note:** Depends on the minimum transfer amount, which is not specified (see open questions).

### TC10-10 - Transfer of 0.01 EUR is rejected when remaining amount is 0.00 EUR

- **Type:** Boundary / Negative
- **Preconditions:** Customer is active with sufficient balance. Daily total = 5,000.00 EUR.
- **Steps:**
  1. Submit a transfer of 0.01 EUR.
- **Expected result:** Rejected in full (same as TC10-08; included to confirm the cent-level boundary).
- **Note:** Redundant with TC10-08 if the rejection rule is applied uniformly. Keep only if the cent-level precision is a stated concern.

### TC10-11 - Daily total resets at the start of the next calendar day

- **Type:** Boundary (time)
- **Preconditions:** Customer is active with sufficient balance. Customer made transfers totalling 5,000.00 EUR on day D (daily total = 5,000.00 EUR on day D).
- **Steps:**
  1. On day D, submit a transfer of 1 EUR and observe rejection.
  2. On day D+1 at 00:00:00 (in the time zone used by the system's definition of "calendar day"), submit a transfer of 5,000.00 EUR.
- **Expected result:** Step 1 is rejected in full. Step 2 is accepted. The daily total for day D+1 is 5,000.00 EUR.

### TC10-12 - Transfer just before midnight is counted against the current day

- **Type:** Boundary (time)
- **Preconditions:** Customer is active with sufficient balance. Daily total = 4,000.00 EUR on day D.
- **Steps:**
  1. On day D at 23:59:59, submit a transfer of 1,000.00 EUR.
  2. Observe the daily total immediately after step 1 and immediately after 00:00:00 on day D+1.
- **Expected result:** Step 1 is accepted on day D and the day D total becomes 5,000.00 EUR. The day D+1 total starts at 0 EUR.

### TC10-13 - Multiple transfers submitted concurrently cannot together exceed the daily limit

- **Type:** Negative / Concurrency
- **Preconditions:** Customer is active with sufficient balance. Daily total = 4,000.00 EUR (remaining 1,000.00 EUR).
- **Steps:**
  1. Submit two transfers of 600.00 EUR each at the same time (e.g., two parallel requests from two channels).
- **Expected result:** Exactly one transfer is accepted and the other is rejected in full. The daily total is 4,600.00 EUR. The total never exceeds 5,000.00 EUR.

### TC10-14 - Transfer that is rejected for the daily limit does not debit the account balance

- **Type:** Negative
- **Preconditions:** Customer has a known account balance of X EUR. Daily total = 4,500.00 EUR.
- **Steps:**
  1. Record the account balance X.
  2. Submit a transfer of 600.00 EUR (exceeds remaining 500.00 EUR).
  3. Check the account balance.
- **Expected result:** The transfer is rejected in full. The account balance is still X EUR. No fee or partial debit is applied.

### TC10-15 - Limit is per customer, not shared between customers

- **Type:** Positive
- **Preconditions:** Customer A has a daily total of 5,000.00 EUR. Customer B is a different active customer with a daily total of 0 EUR.
- **Steps:**
  1. Customer A submits a transfer of 1 EUR (expected rejection).
  2. Customer B submits a transfer of 5,000.00 EUR.
- **Expected result:** Customer A's transfer is rejected in full. Customer B's transfer is accepted.

### TC10-16 - Limit applies across all bank transfers, regardless of destination or account

- **Type:** Positive / Negative
- **Preconditions:** Customer has two accounts or sends to two different beneficiaries. Daily total = 3,000.00 EUR from transfers to beneficiary P.
- **Steps:**
  1. Submit a transfer of 2,000.00 EUR to beneficiary Q (from a different account, if available).
  2. Submit a transfer of 0.01 EUR to beneficiary P.
- **Expected result:** Step 1 is accepted (daily total 5,000.00 EUR). Step 2 is rejected in full.
- **Note:** Assumes the limit is per customer across accounts and beneficiaries. See open questions.

### TC10-17 - Transfer with a zero amount is rejected

- **Type:** Negative (invalid input)
- **Preconditions:** Customer is active. Daily total = 0 EUR.
- **Steps:**
  1. Submit a transfer of 0.00 EUR.
- **Expected result:** The transfer is rejected by input validation. The daily total remains 0 EUR. The rejection reason is not specific to the daily limit.
- **Note:** Expected result depends on the behaviour for invalid amounts, which is not specified.

### TC10-18 - Transfer with a negative amount is rejected

- **Type:** Negative (invalid input)
- **Preconditions:** Customer is active. Daily total = 0 EUR.
- **Steps:**
  1. Submit a transfer of -100.00 EUR.
- **Expected result:** The transfer is rejected by input validation. The daily total is not changed (e.g., it does not reduce the total by 100 EUR).

### TC10-19 - Transfer with more than two decimal places is handled consistently

- **Type:** Boundary (format)
- **Preconditions:** Customer is active. Daily total = 4,999.99 EUR.
- **Steps:**
  1. Submit a transfer of 0.011 EUR.
- **Expected result:** The system either rejects the input as invalid or rounds it consistently. The daily total is never silently exceeded (e.g., 0.011 rounded up to 0.02 would be rejected since it exceeds the remaining 0.01 EUR).
- **Note:** Expected behaviour is not specified (see open questions).

### TC10-20 - Transfer in a non-EUR currency is handled according to the rule for the limit

- **Type:** Negative / Boundary
- **Preconditions:** Customer is active. Daily total = 0 EUR. The system supports transfers in at least one non-EUR currency.
- **Steps:**
  1. Submit a transfer of an amount in another currency whose EUR equivalent is 5,000.01 EUR or more.
- **Expected result:** Not determinable from the requirement. See open questions.

### TC10-21 - Scheduled or future-dated transfers are counted on the day they are executed

- **Type:** Positive / Boundary
- **Preconditions:** Customer is active. Daily total = 0 EUR. A transfer of 5,000.00 EUR is scheduled for tomorrow.
- **Steps:**
  1. Today, submit a transfer of 1 EUR.
  2. Tomorrow, when the scheduled transfer executes, check whether it is accepted.
- **Expected result:** Step 1 is accepted. The scheduled transfer is evaluated against tomorrow's daily total at execution time, so it is accepted or rejected in full based on that day's total.
- **Note:** Assumes the limit applies at execution time. See open questions.

## Open questions

1. **Inclusive or exclusive limit:** Is exactly 5,000.00 EUR allowed in total? ("Up to" suggests inclusive, but this is not stated.)
2. **What counts toward the total:** Does the total include only successful transfers, or also pending, scheduled or failed ones? Are fees included in the amount counted?
3. **Rejected transfers:** Does a rejected transfer count toward the daily total in any way? (The requirement says it is rejected in full, which implies no. Confirm.)
4. **Definition of "calendar day":** Which time zone is used (customer's local time, the bank's time zone, or UTC)? What exact time does the day reset?
5. **Scope of the limit:** Is the limit per customer, per account, or per beneficiary? Does it apply across all transfer types (domestic, SEPA, international, own-account transfers)?
6. **Currency:** Are transfers in currencies other than EUR allowed? If yes, how is the EUR amount determined (exchange rate, date, fees)?
7. **Minimum and maximum single-transfer amounts:** Is there a minimum amount? Is there a separate per-transfer maximum? The requirement only covers the daily total.
8. **Amount precision:** Are amounts limited to two decimal places? How are amounts with more decimals handled?
9. **Error handling and messages:** What rejection message or error code does the customer see for a limit breach? Is the customer notified in a specific channel?
10. **Scheduled and future-dated transfers:** When is the limit checked: at submission time or at execution time? Which day's total applies?
11. **Concurrency:** If two transfers are submitted at the same time, what is the required outcome? (The requirement implies the total must never exceed 5,000 EUR.)
12. **Limit changes:** Can the limit be changed per customer (e.g., raised for premium customers or reduced for risk reasons)? Is the 5,000 EUR figure fixed?
13. **Partial execution:** The requirement says "rejected in full". Confirm that no partial execution, split or queued processing happens in any case.
14. **Failed or reversed transfers:** If a transfer is later reversed or fails after acceptance, is the daily total reduced? Is the reduction immediate?
15. **Authorization or staff override:** Can an operator or staff member override the limit? If so, how is it recorded?
