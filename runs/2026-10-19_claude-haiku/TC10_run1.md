# Test cases: Daily bank transfer limit (5,000 EUR per customer per calendar day)

Requirement: A customer can send bank transfers of up to 5,000 EUR in total per calendar day. A transfer that would exceed the remaining daily amount is rejected in full.

Working assumptions used in the cases below (these are not stated in the requirement, so each one is also listed as an open question):
- The limit is per customer, and it counts only accepted transfers.
- "Calendar day" means midnight-to-midnight in one reference time zone (TBD, see open questions).
- "Remaining amount" = 5,000 EUR minus the sum of accepted transfers already made on that day.

---

## Positive scenarios

### TC-01: Single transfer well below the limit is accepted
- **Preconditions:** Customer has an active account with sufficient balance. No transfers made today (remaining daily amount = 5,000.00 EUR).
- **Steps:**
  1. Submit a transfer of 1,000.00 EUR.
- **Expected result:** Transfer is accepted and executed. Remaining daily amount becomes 4,000.00 EUR.

### TC-02: Several transfers whose sum stays below the limit are all accepted
- **Preconditions:** Customer has sufficient balance. No transfers made today.
- **Steps:**
  1. Submit a transfer of 1,000.00 EUR. Expect acceptance.
  2. Submit a transfer of 2,000.00 EUR. Expect acceptance.
  3. Submit a transfer of 1,500.00 EUR. Expect acceptance.
- **Expected result:** All three transfers are accepted (total 4,500.00 EUR). Remaining daily amount is 500.00 EUR.

### TC-03: Single transfer of exactly 5,000.00 EUR is accepted
- **Preconditions:** Customer has sufficient balance. No transfers made today.
- **Steps:**
  1. Submit a transfer of 5,000.00 EUR.
- **Expected result:** Transfer is accepted. Remaining daily amount is 0.00 EUR.

### TC-04: Transfers that add up to exactly 5,000.00 EUR are all accepted
- **Preconditions:** Customer has sufficient balance. No transfers made today.
- **Steps:**
  1. Submit a transfer of 3,000.00 EUR. Expect acceptance.
  2. Submit a transfer of 2,000.00 EUR. Expect acceptance.
- **Expected result:** Both transfers are accepted. The second one lands exactly on the limit. Remaining daily amount is 0.00 EUR.

### TC-05: Transfer that exactly fills the remaining amount is accepted
- **Preconditions:** Customer has already made accepted transfers totalling 4,000.00 EUR today (remaining = 1,000.00 EUR).
- **Steps:**
  1. Submit a transfer of 1,000.00 EUR.
- **Expected result:** Transfer is accepted. Remaining daily amount is 0.00 EUR.

---

## Negative scenarios

### TC-06: Single transfer above 5,000.00 EUR is rejected in full
- **Preconditions:** Customer has sufficient balance. No transfers made today.
- **Steps:**
  1. Submit a transfer of 5,000.01 EUR.
- **Expected result:** Transfer is rejected. Nothing is debited from the account and no partial amount is sent. Remaining daily amount stays 5,000.00 EUR. The customer is told the transfer was rejected.

### TC-07: Transfer that exceeds the remaining amount is rejected in full (no partial execution)
- **Preconditions:** Customer has accepted transfers totalling 4,000.00 EUR today (remaining = 1,000.00 EUR). Customer has sufficient balance.
- **Steps:**
  1. Submit a transfer of 1,500.00 EUR.
- **Expected result:** Transfer is rejected in full. Account balance is unchanged. No 1,000.00 EUR partial transfer is executed. Remaining daily amount stays 1,000.00 EUR.

### TC-08: Rejected transfer does not reduce the remaining daily amount
- **Preconditions:** Same state as TC-07 (remaining = 1,000.00 EUR).
- **Steps:**
  1. Submit a transfer of 1,500.00 EUR. Expect rejection.
  2. Submit a transfer of 1,000.00 EUR.
- **Expected result:** Step 1 is rejected. Step 2 is accepted, because the full 1,000.00 EUR remaining amount is still available. Remaining daily amount becomes 0.00 EUR.

### TC-09: Any transfer is rejected once the daily limit is fully used
- **Preconditions:** Customer has accepted transfers totalling 5,000.00 EUR today (remaining = 0.00 EUR).
- **Steps:**
  1. Submit a transfer of 0.01 EUR.
- **Expected result:** Transfer is rejected in full. Nothing is debited. Remaining daily amount stays 0.00 EUR.

### TC-10: Two transfers submitted at the same time cannot together exceed the daily limit
- **Preconditions:** Customer has accepted transfers totalling 2,000.00 EUR today (remaining = 3,000.00 EUR). Customer has sufficient balance.
- **Steps:**
  1. Submit transfer A of 2,000.00 EUR and transfer B of 2,000.00 EUR at the same moment (two parallel requests, for example from two sessions or a double-click).
- **Expected result:** Exactly one transfer is accepted and the other is rejected in full. Accepted total for the day is 4,000.00 EUR, which is at most 5,000.00 EUR. Remaining daily amount is 1,000.00 EUR. Repeat several times to check for race conditions.

### TC-11: Accepted transfers are counted per customer, not across customers
- **Preconditions:** Customer A has used 5,000.00 EUR today (remaining = 0.00 EUR). Customer B is a different customer with no transfers today.
- **Steps:**
  1. Customer A submits a transfer of 1.00 EUR.
  2. Customer B submits a transfer of 5,000.00 EUR.
- **Expected result:** Step 1 is rejected. Step 2 is accepted. Customer B's limit is not affected by Customer A's usage.

---

## Boundary scenarios

### TC-12: Remaining amount of 0.01 EUR is usable, and the next cent is not
- **Preconditions:** Customer has accepted transfers totalling 4,999.99 EUR today (remaining = 0.01 EUR).
- **Steps:**
  1. Submit a transfer of 0.01 EUR. Expect acceptance.
  2. Submit a transfer of 0.01 EUR.
- **Expected result:** Step 1 is accepted, and remaining daily amount becomes 0.00 EUR. Step 2 is rejected in full.

### TC-13: Transfer of 0.01 EUR more than the remaining amount is rejected
- **Preconditions:** Customer has accepted transfers totalling 4,999.99 EUR today (remaining = 0.01 EUR).
- **Steps:**
  1. Submit a transfer of 0.02 EUR.
- **Expected result:** Transfer is rejected in full. Remaining daily amount stays 0.01 EUR.

### TC-14: Cumulative total of 5,000.00 EUR is accepted, and 5,000.01 EUR is rejected
- **Preconditions:** No transfers made today. Customer has sufficient balance.
- **Steps:**
  1. Submit a transfer of 4,999.99 EUR. Expect acceptance.
  2. Submit a transfer of 0.02 EUR. Expect rejection (4,999.99 + 0.02 = 5,000.01 EUR).
  3. Submit a transfer of 0.01 EUR. Expect acceptance (total exactly 5,000.00 EUR).
- **Expected result:** Steps 1 and 3 are accepted. Step 2 is rejected in full. Final accepted total is 5,000.00 EUR.

### TC-15: Limit resets at the start of the next calendar day
- **Preconditions:** Customer used the full 5,000.00 EUR on day D. The reference time zone is known (see open question OQ-2).
- **Steps:**
  1. On day D, at 23:59:59 in the reference time zone, submit a transfer of 0.01 EUR. Expect rejection.
  2. On day D+1, at 00:00:00 in the reference time zone, submit a transfer of 5,000.00 EUR.
- **Expected result:** Step 1 is rejected. Step 2 is accepted, because the new day starts with the full 5,000.00 EUR available.

### TC-16: Transfer just before midnight counts toward the current day
- **Preconditions:** No transfers made on day D. Reference time zone known (OQ-2).
- **Steps:**
  1. On day D at 23:59:59, submit a transfer of 3,000.00 EUR.
  2. On day D+1 at 00:00:01, submit a transfer of 3,000.00 EUR.
- **Expected result:** Step 1 is accepted and counts toward day D (remaining on D becomes 2,000.00 EUR). Step 2 is accepted and counts toward day D+1 (remaining on D+1 becomes 2,000.00 EUR).

### TC-17: Transfer submitted at the daily reset does not count toward the previous day
- **Preconditions:** Customer has 4,000.00 EUR accepted on day D. Reference time zone known (OQ-2).
- **Steps:**
  1. At 00:00:00 on day D+1, submit a transfer of 5,000.00 EUR.
- **Expected result:** Transfer is accepted in full and counts only toward day D+1. It is not rejected against day D's usage.

---

## Open questions

These questions are not answered by the requirement. No behaviour is assumed for them in the test cases above beyond the marked assumptions. Each one needs an answer before the related test case can be finalised.

- **OQ-1 Fees and charges:** Does the 5,000 EUR limit apply to the transfer amount only, or to the amount plus any transfer fees?
- **OQ-2 Time zone of the "calendar day":** Which time zone defines the day boundary (customer's local time, bank's time zone, or UTC)? Does this matter for customers who move between time zones?
- **OQ-3 Multiple accounts:** If a customer has several accounts or a joint account, is the 5,000 EUR limit shared across all of them, or per account? Does it apply per customer or per account holder?
- **OQ-4 Currency:** Are transfers in currencies other than EUR allowed? If so, how is the amount converted for the limit (which exchange rate, and when)?
- **OQ-5 Which transfers count:** Do pending, scheduled, or future-dated transfers reserve part of the daily amount when they are created, or only when they execute? Which day counts for a future-dated transfer?
- **OQ-6 Failed or cancelled transfers:** Does a transfer that fails technically after acceptance, or is later cancelled, release its amount back to the daily limit? Does a transfer rejected by the bank (for example for insufficient funds) affect the limit?
- **OQ-7 Rejection message:** What must the customer see when a transfer is rejected for exceeding the limit? Should the message show the remaining daily amount? What error code or status is returned to API clients?
- **OQ-8 Amount validation:** What are the minimum and maximum amounts per single transfer? Is a zero or negative amount rejected, and with what message? How are amounts with more than two decimal places handled?
- **OQ-9 Limit on customer-initiated versus other transfers:** Do standing orders, direct debits, refunds, or transfers initiated by bank staff count toward the customer's limit?
- **OQ-10 Limit changes:** Can the 5,000 EUR limit be changed per customer (for example by a higher-tier product or a manual override)? Does a change apply immediately to the current day?
- **OQ-11 Concurrency expectation:** TC-10 assumes the system must never exceed the limit under parallel requests. Is this the expected behaviour, and which request wins when two are submitted at the same time (first received, or first processed)?
