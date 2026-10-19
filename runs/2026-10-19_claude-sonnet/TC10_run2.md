# Test cases: Daily bank transfer limit (5,000 EUR per calendar day)

Requirement: A customer can send bank transfers of up to 5,000 EUR in total per calendar day. A transfer that would exceed the remaining daily amount is rejected in full.

## Assumptions used only to make the cases executable (to be confirmed, see open questions)
- The limit is evaluated on the sum of accepted transfers of one customer in one calendar day.
- "Up to 5,000 EUR" is inclusive: a total of exactly 5,000.00 EUR is allowed.
- Amounts are in EUR with two decimals.
- Rejected transfers do not consume any of the daily amount.

Unless stated otherwise, the preconditions include: customer is authenticated, has an active EUR account with sufficient balance (at least 10,000 EUR), and no transfers have been sent yet on the current calendar day.

## Positive cases

**TC-01: Single transfer well below the limit**
- Preconditions: Daily total sent = 0.00 EUR.
- Steps: 1. Submit a transfer of 100.00 EUR to a valid beneficiary.
- Expected: Transfer is accepted and executed. Remaining daily amount becomes 4,900.00 EUR.

**TC-02: Multiple transfers whose total stays below the limit**
- Preconditions: Daily total sent = 0.00 EUR.
- Steps: 1. Send 1,000.00 EUR. 2. Send 1,500.00 EUR. 3. Send 2,000.00 EUR.
- Expected: All three are accepted. Total = 4,500.00 EUR; remaining = 500.00 EUR.

**TC-03: Single transfer exactly equal to the limit**
- Preconditions: Daily total sent = 0.00 EUR.
- Steps: 1. Send 5,000.00 EUR.
- Expected: Accepted. Remaining daily amount = 0.00 EUR.

**TC-04: Several transfers summing exactly to the limit**
- Preconditions: Daily total sent = 0.00 EUR.
- Steps: 1. Send 2,000.00 EUR. 2. Send 2,000.00 EUR. 3. Send 1,000.00 EUR.
- Expected: All accepted. Total = 5,000.00 EUR; remaining = 0.00 EUR.

**TC-05: Transfer equal to the exact remaining amount**
- Preconditions: Daily total sent = 3,250.50 EUR.
- Steps: 1. Send 1,749.50 EUR.
- Expected: Accepted. Remaining = 0.00 EUR.

**TC-06: Limit resets on the next calendar day**
- Preconditions: Daily total sent = 5,000.00 EUR on day D.
- Steps: 1. On day D+1 (after the day boundary), send 5,000.00 EUR.
- Expected: Accepted; the limit is calculated anew for D+1.

**TC-07: Limit is per customer**
- Preconditions: Customer A has sent 5,000.00 EUR today. Customer B has sent 0.00 EUR.
- Steps: 1. As customer B, send 3,000.00 EUR.
- Expected: Accepted. Customer A's usage does not affect customer B.

## Negative cases

**TC-08: Single transfer above the limit**
- Preconditions: Daily total sent = 0.00 EUR.
- Steps: 1. Send 5,000.01 EUR.
- Expected: Rejected in full, with a clear message that the daily limit would be exceeded. No money is moved, no partial amount is sent, daily total stays 0.00 EUR.

**TC-09: Transfer exceeding the remaining amount is rejected in full (not partially executed)**
- Preconditions: Daily total sent = 4,000.00 EUR (remaining 1,000.00 EUR).
- Steps: 1. Send 1,500.00 EUR.
- Expected: Rejected entirely. The account is not debited, not even by 1,000.00 EUR. Remaining stays 1,000.00 EUR. Error message is shown.

**TC-10: Transfer rejected when limit already fully used**
- Preconditions: Daily total sent = 5,000.00 EUR.
- Steps: 1. Send 0.01 EUR.
- Expected: Rejected; remaining amount is 0.00 EUR.

**TC-11: Rejected transfer does not consume the limit**
- Preconditions: Daily total sent = 4,000.00 EUR.
- Steps: 1. Send 2,000.00 EUR (rejected). 2. Send 1,000.00 EUR.
- Expected: Step 1 rejected; step 2 accepted. Remaining = 0.00 EUR at the end.

**TC-12: Rejected transfer followed by smaller valid transfer**
- Preconditions: Daily total sent = 4,500.00 EUR.
- Steps: 1. Send 600.00 EUR (rejected). 2. Send 500.00 EUR.
- Expected: Step 1 rejected in full; step 2 accepted.

**TC-13: Invalid amounts**
- Preconditions: Daily total sent = 0.00 EUR.
- Steps: Try amounts 0.00, -100.00, and non-numeric "abc".
- Expected: All rejected with validation errors (not with a limit error); daily total unchanged. (Exact behaviour for 0 is an open question.)

**TC-14: Concurrent transfers racing for the same remaining amount**
- Preconditions: Daily total sent = 3,000.00 EUR (remaining 2,000.00 EUR). Two sessions of the same customer.
- Steps: 1. Simultaneously submit 1,500.00 EUR from session 1 and 1,500.00 EUR from session 2.
- Expected: Exactly one is accepted and one is rejected in full. Total never exceeds 5,000.00 EUR.

**TC-15: Limit cannot be bypassed via other channels**
- Preconditions: Customer has sent 4,000.00 EUR via web banking today.
- Steps: 1. Send 1,500.00 EUR via mobile app (and via API, if available).
- Expected: Rejected on every channel, if the limit is per customer across channels (see open question 4).

## Boundary cases

**TC-16: Just below the limit**
- Preconditions: Daily total sent = 0.00 EUR.
- Steps: 1. Send 4,999.99 EUR.
- Expected: Accepted; remaining = 0.01 EUR.

**TC-17: Exactly at the limit** (see TC-03)
- Expected: 5,000.00 EUR accepted.

**TC-18: Just above the limit** (see TC-08)
- Expected: 5,000.01 EUR rejected.

**TC-19: Minimal remaining amount**
- Preconditions: Daily total sent = 4,999.99 EUR.
- Steps: 1. Send 0.01 EUR. 2. Send 0.01 EUR again.
- Expected: First accepted (remaining 0.00); second rejected.

**TC-20: One cent over the remaining amount**
- Preconditions: Daily total sent = 2,500.00 EUR.
- Steps: 1. Send 2,500.01 EUR.
- Expected: Rejected in full.

**TC-21: Day boundary just before midnight**
- Preconditions: Daily total sent = 0.00 EUR; time 23:59:59 (in the timezone used for the limit).
- Steps: 1. Send 5,000.00 EUR at 23:59:59. 2. Send 5,000.00 EUR at 00:00:00 of the next day.
- Expected: Both accepted, as they belong to different calendar days.

**TC-22: Transfers on both sides of midnight do not accumulate**
- Preconditions: Daily total sent = 0.00 EUR on day D.
- Steps: 1. At 23:59 send 3,000.00 EUR. 2. At 00:01 (D+1) send 4,000.00 EUR.
- Expected: Both accepted; the limit counts per calendar day only.

**TC-23: Decimal precision**
- Preconditions: Daily total sent = 0.00 EUR.
- Steps: 1. Send 0.10 EUR thirty times in a row, then 4,997.00 EUR in total using further transfers (e.g. 4,997.00 EUR as one transfer). 2. Send 0.01 EUR.
- Expected: Total reaches exactly 5,000.00 EUR with no floating-point rounding error; step 2 rejected.

**TC-24: Amount with more than two decimals**
- Preconditions: Daily total sent = 0.00 EUR.
- Steps: 1. Send 4,999.999 EUR.
- Expected: Behaviour per rounding/validation rule (open question 7); must not be silently accepted as above-limit-bypassing or rounded up past the limit.

**TC-25: Daylight saving time change day**
- Preconditions: Day with a 23-hour or 25-hour clock change.
- Steps: 1. Send transfers across the DST switch totalling 5,000.00 EUR, then 0.01 EUR.
- Expected: The day boundary behaves per defined timezone; the daily limit is applied to one calendar day only.

## Open questions (requirement is ambiguous or incomplete)

1. Is "up to 5,000 EUR" inclusive (5,000.00 allowed)? Test cases TC-03/04/05 assume yes.
2. Which timezone defines the "calendar day" (customer's, bank's, UTC)? When exactly does it roll over, and how does it work on DST change days?
3. Does the limit apply per customer, per account (for customers with several accounts) or per card/device? What about joint accounts?
4. Which channels count (web, mobile, API, branch, standing orders, scheduled transfers)? Are all channels summed together?
5. Which transfer types are included: SEPA credit transfer, instant transfers, internal transfers between own accounts, international transfers, card payments, direct debits?
6. Foreign currency: how are transfers in other currencies converted (which rate, which time)? Is the limit applied to the EUR equivalent?
7. Amount precision and rounding: how are amounts with more than two decimals handled? Is the minimum transfer amount defined? Is a 0.00 transfer valid?
8. Scheduled/future-dated transfers: is the amount counted on the day of ordering or on the execution day?
9. Pending, failed, reversed or cancelled transfers: do they consume or release the daily amount? What about transfers that are later returned by the beneficiary bank?
10. Fees: are they included in the counted amount?
11. Batch or bulk payments: is a file with several payments rejected as a whole or per payment if the sum exceeds the remaining amount?
12. What is the exact error message, error code and user feedback on rejection? Should the remaining daily amount be shown to the user before sending?
13. Can the limit be changed (higher limits for VIP/business customers, temporary increases, strong customer authentication)? Is it configurable?
14. How are concurrent requests handled (locking/atomicity), and what is the required behaviour on system errors or timeouts (is the amount reserved)?
15. Is the rejection logged/audited and is the customer notified?
