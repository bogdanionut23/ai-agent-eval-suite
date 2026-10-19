# Test Cases: Daily Transfer Limit (5,000 EUR per calendar day)

Requirement: A customer can send bank transfers of up to 5,000 EUR in total per calendar day. A transfer that would exceed the remaining daily amount is rejected in full.

Interpretation used (only what the text states): the sum of accepted transfers in one calendar day may reach 5,000.00 EUR inclusive ("up to"). A transfer that would push the total above 5,000.00 EUR is rejected entirely (no partial execution). Anything else is listed under Open Questions and is not assumed.

Common preconditions (unless overridden): customer is authenticated, has an active EUR account with sufficient balance (at least 10,000 EUR), the destination account is valid, and no other limits (per-transaction, account-balance, fraud) interfere. Daily total starts at 0.00 EUR.

## Positive

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-01 | Single transfer well below limit | Daily total 0.00 | 1. Send 100.00 EUR | Transfer accepted and executed. Remaining daily amount = 4,900.00 EUR. |
| TC-02 | Several transfers within limit | Daily total 0.00 | 1. Send 1,000.00. 2. Send 1,500.00. 3. Send 2,000.00 | All three accepted. Total 4,500.00; remaining 500.00. |
| TC-03 | Transfer exactly equals the limit (single) | Daily total 0.00 | 1. Send 5,000.00 EUR | Accepted. Remaining = 0.00 EUR. |
| TC-04 | Transfers sum exactly to the limit | Daily total 0.00 | 1. Send 3,000.00. 2. Send 2,000.00 | Both accepted. Total 5,000.00; remaining 0.00. |
| TC-05 | Transfer exactly equals remaining amount | Daily total 4,200.00 (remaining 800.00) | 1. Send 800.00 | Accepted. Remaining = 0.00. |
| TC-06 | Remaining amount shown correctly | Daily total 1,250.50 | 1. View the remaining daily limit (if displayed) | Shows 3,749.50 EUR. |
| TC-07 | Limit resets on next calendar day | Daily total 5,000.00 on day D | 1. On day D+1 send 5,000.00 | Accepted; the new day starts at 0.00 total. |
| TC-08 | Different customers have independent limits | Customer A used 5,000.00 today; customer B used 0.00 | 1. Customer B sends 3,000.00 | Accepted for B; A's usage does not affect B. |

## Negative

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-09 | Single transfer above limit | Daily total 0.00 | 1. Send 5,000.01 EUR | Rejected in full. No funds debited. Daily total stays 0.00. Clear error message. |
| TC-10 | Transfer exceeds remaining amount | Daily total 4,200.00 (remaining 800.00) | 1. Send 800.01 | Rejected in full (no partial 800.00 executed). Total stays 4,200.00; balance unchanged. |
| TC-11 | Transfer after limit fully used | Daily total 5,000.00 | 1. Send 0.01 EUR | Rejected. Total unchanged. |
| TC-12 | Rejected transfer does not consume limit | Daily total 4,000.00 | 1. Send 1,500.00 (rejected). 2. Send 1,000.00 | Step 1 rejected; step 2 accepted. Total 5,000.00. |
| TC-13 | Rejected transfer leaves no trace in balance or history as executed | Daily total 4,500.00 | 1. Send 600.00 | Rejected; balance unchanged; no executed transaction created (a rejected/failed record may be logged per policy, see Q9). |
| TC-14 | Zero amount | Daily total 0.00 | 1. Send 0.00 | Rejected by validation (not a valid transfer); limit unaffected. See Q6. |
| TC-15 | Negative amount | Daily total 0.00 | 1. Send -100.00 | Rejected by validation; limit not increased or decreased. |
| TC-16 | Non-numeric amount | Daily total 0.00 | 1. Enter "abc" as amount | Rejected by validation; no effect on limit. |
| TC-17 | Failed transfer (insufficient balance) does not count | Balance 100.00; daily total 0.00 | 1. Send 500.00 (fails on balance). 2. Send 5,000.00 with sufficient balance after top-up | Step 1 fails for balance reasons and does not consume limit; step 2 accepted. (Subject to Q8.) |
| TC-18 | Unauthenticated request | No session | 1. Submit a transfer via API without auth | Rejected (401/403); no limit calculation side effects. |

## Boundary

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-19 | Exactly at limit (5,000.00) | Total 0.00 | Send 5,000.00 | Accepted. |
| TC-20 | One cent above limit | Total 0.00 | Send 5,000.01 | Rejected. |
| TC-21 | One cent below limit | Total 0.00 | Send 4,999.99 | Accepted; remaining 0.01. |
| TC-22 | Use last remaining cent | Total 4,999.99 | Send 0.01 | Accepted; remaining 0.00. |
| TC-23 | Exceed by one cent with remaining 0.01 | Total 4,999.99 | Send 0.02 | Rejected in full. |
| TC-24 | Minimum valid amount | Total 0.00 | Send 0.01 | Accepted (if minimum transfer is 0.01; see Q6). |
| TC-25 | More than 2 decimals | Total 0.00 | Send 4,999.999 | Rejected or handled per currency precision rules (see Q7); must not silently round to exceed 5,000.00. |
| TC-26 | Just before midnight | Total 5,000.00 at 23:59:59 day D | Send 100.00 at 23:59:59 | Rejected (still day D). |
| TC-27 | Just after midnight | Same as above | Send 100.00 at 00:00:00 day D+1 | Accepted (new day). Time zone per Q1. |
| TC-28 | Leap day / month / year boundary | Total 5,000.00 on 31 Dec | Send 100.00 on 1 Jan; also repeat for 28/29 Feb to 1 Mar | Accepted on the new day each time. |
| TC-29 | DST change days (23h and 25h days) | Total 5,000.00 earlier that day | Send 100.00 shortly after the clock change, and across the transition | Day boundary behaves consistently with Q1; no double or lost reset. |

## Concurrency / robustness

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-30 | Two simultaneous transfers exceeding limit together | Total 3,000.00 | Submit two 1,500.00 transfers at the same moment from two sessions | At most one accepted per limit: if the first brings total to 4,500, the second (1,500) is rejected only if it exceeds 5,000. Here 3,000+1,500+1,500 = 6,000, so exactly one is accepted and one rejected. Final total never exceeds 5,000.00. |
| TC-31 | Parallel requests at the exact limit | Total 0.00 | Submit 5 parallel 1,000.00 transfers, then 1 more | First 5 accepted (total 5,000.00); sixth rejected. No race allows more than 5,000.00. |
| TC-32 | Retry/duplicate submission (idempotency) | Total 0.00 | Submit the same transfer (same idempotency key) 2 times, 3,000.00 | Executed once; total 3,000.00 (subject to Q10). |
| TC-33 | Multiple channels share one limit | Total 0.00 | Send 3,000.00 via web, then 2,500.00 via mobile | Second rejected if the limit is shared across channels (see Q5); otherwise per rule. |
| TC-34 | Multiple accounts of one customer | Customer has 2 accounts; 3,000.00 sent from A | Send 2,500.00 from account B | Outcome depends on Q4 (limit per customer vs per account). |

## Open Questions (not assumed above)

1. **Calendar day definition:** which time zone (customer, bank/CET, UTC)? When exactly does the day roll over, and how are DST days handled?
2. **"Up to 5,000 EUR":** is 5,000.00 inclusive? (Assumed yes; "exceed" implies a total strictly above 5,000 is rejected.)
3. **Which transfers count:** only executed, or also pending, scheduled, or future-dated ones? For scheduled transfers, is the limit evaluated at submission or at execution date?
4. **Scope of "customer":** per customer across all accounts, or per account? What about joint accounts and business vs. private customers?
5. **Channels and types:** do web, mobile, API, standing orders, SEPA Instant, SEPA Credit, and international transfers all count toward the same limit? Do transfers between the customer's own accounts count?
6. **Minimum amount and zero/negative values:** what is the minimum transfer and expected validation message?
7. **Currency and precision:** how are non-EUR transfers converted (which rate, when)? How many decimals are allowed, and what is the rounding rule?
8. **Failed, reversed, cancelled, or recalled transfers:** do they free up the limit again (e.g., a return after execution, or cancellation of a pending transfer)? Do fees count toward the limit?
9. **Rejection behaviour:** exact error message/code, whether the remaining amount is shown to the customer, and whether a rejected attempt is logged or notified.
10. **Concurrency and idempotency:** required guarantees when parallel requests arrive; handling of duplicates.
11. **Limit changes:** can the limit be raised/lowered per customer (e.g., verified customers, bank staff override)? What happens if the limit changes mid-day?
12. **Remaining amount visibility:** is the remaining daily amount required to be displayed to the customer or exposed via API?
13. **Multi-step authorisation:** for transfers needing approval (e.g., business accounts), is the limit consumed at initiation or at approval?
