# Test cases: Orders page date range filter

Requirement: On the orders page a user can filter orders by a date range (From, To). Both dates are included. The range cannot be longer than 31 days. To cannot be earlier than From.

## Assumptions (taken literally from the requirement, nothing else invented)

- "Range longer than 31 days" is interpreted in the way that is the most natural reading but is NOT confirmed (see open questions Q1). For the test cases I use the inclusive-count reading: a range of From = 2026-01-01 and To = 2026-01-31 is 31 days (inclusive), so it is valid; From = 2026-01-01 and To = 2026-02-01 is 32 days inclusive, so it is invalid. If the answer to Q1 is different, TC-B02/B03/B04 must be re-checked.
- Exact wording of error messages and UI behaviour on invalid input is not specified, so expected results for negative cases say "the filter is not applied and the user is informed that the input is invalid" and the exact text is to be confirmed (Q4).
- Test data: the user has orders on every date from 2025-12-25 to 2026-03-10, at least one per day, plus one order at 00:00:00 and one at 23:59:59 on 2026-01-10 (time zone of the user and server are the same unless stated otherwise).

## Positive scenarios

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-P01 | Filter by a normal range inside one month | User is logged in, orders page open, test data present | 1. Enter From = 2026-01-10. 2. Enter To = 2026-01-20. 3. Apply the filter. | Only orders dated 2026-01-10 to 2026-01-20 are listed. |
| TC-P02 | Both boundary dates are included | Same as above | 1. From = 2026-01-10, To = 2026-01-20. 2. Apply. 3. Check the first and last listed orders. | Orders dated exactly 2026-01-10 and exactly 2026-01-20 are shown. No order from 2026-01-09 or 2026-01-21 is shown. |
| TC-P03 | Single-day range (From = To) | Same as above | 1. From = 2026-01-10, To = 2026-01-10. 2. Apply. | Only orders dated 2026-01-10 are shown, including the ones at 00:00:00 and 23:59:59. No error shown. |
| TC-P04 | Range spanning two months | Same as above | 1. From = 2026-01-25, To = 2026-02-05. 2. Apply. | Orders from 2026-01-25 to 2026-02-05 are shown. |
| TC-P05 | Range spanning two years | Same as above | 1. From = 2025-12-28, To = 2026-01-05. 2. Apply. | Orders from 2025-12-28 to 2026-01-05 are shown. |
| TC-P06 | Range including 29 February (leap year) | Test data covers 2028-02-20 to 2028-03-05 | 1. From = 2028-02-25, To = 2028-03-02. 2. Apply. | Orders including 2028-02-29 are shown correctly. |
| TC-P07 | Change an applied filter | A filter is already applied (TC-P01) | 1. Change To to 2026-01-25. 2. Apply. | List refreshes and now shows 2026-01-10 to 2026-01-25. |
| TC-P08 | Range with no matching orders | A date range with no orders exists, e.g. 2030-01-01 to 2030-01-10 | 1. Enter that range. 2. Apply. | An empty result is shown (no error). Exact empty-state text to be confirmed. |
| TC-P09 | Entering dates with the date picker and by typing | Same as TC-P01 | 1. Select dates with the picker and apply. 2. Repeat, typing the same dates in the accepted format. | Both methods give the same result. |

## Boundary scenarios (range length, 31 days)

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-B01 | 30-day range (just under the limit) | Test data present | 1. From = 2026-01-01, To = 2026-01-30. 2. Apply. | Filter accepted, orders for 30 days shown. |
| TC-B02 | Exactly 31-day range (maximum allowed) | Test data present | 1. From = 2026-01-01, To = 2026-01-31. 2. Apply. | Filter accepted, orders for all 31 days shown. (Depends on Q1.) |
| TC-B03 | 32-day range (just over the limit) | Test data present | 1. From = 2026-01-01, To = 2026-02-01. 2. Apply. | Filter is rejected, user is informed that the range is too long, list is not filtered with this range. (Depends on Q1.) |
| TC-B04 | Difference of exactly 31 days between dates (32 days inclusive) | Test data present | 1. From = 2026-01-01, To = 2026-02-01 (date difference = 31). | Same as TC-B03 under the inclusive reading; under the "difference" reading it would be accepted. Result must match the answer to Q1. |
| TC-B05 | 31-day limit across February in a non-leap year | Test data present | 1. From = 2026-02-01, To = 2026-03-03 (31 days inclusive). 2. Apply. | Accepted. Then To = 2026-03-04 (32 days) is rejected. |
| TC-B06 | 31-day limit across February in a leap year | Test data for 2028 | 1. From = 2028-02-01, To = 2028-03-02 (31 days inclusive). 2. Apply. 3. Then change To = 2028-03-03. | Step 2 accepted, step 3 rejected as too long. |
| TC-B07 | Boundary at month end | Test data present | 1. From = 2026-01-31, To = 2026-03-02 (31 days inclusive). 2. Apply. | Accepted (a "one calendar month" implementation would wrongly behave differently). |
| TC-B08 | To = From + 1 day (smallest multi-day range) | Test data present | 1. From = 2026-01-10, To = 2026-01-11. | Accepted, orders of both days shown. |
| TC-B09 | To one day before From | Test data present | 1. From = 2026-01-10, To = 2026-01-09. 2. Apply. | Rejected, user informed that To cannot be earlier than From. |
| TC-B10 | Orders at the time edges of the boundary dates | Orders at 00:00:00 and 23:59:59 on 2026-01-10 | 1. From = To = 2026-01-10. 2. Apply. | Both edge-time orders are included (see Q3 on time zones). |
| TC-B11 | Earliest and latest supported dates | Depends on system limits (Q6) | 1. Use the oldest and newest dates the system allows. | Behaviour defined once Q6 is answered. |

## Negative scenarios

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-N01 | To earlier than From | Orders page open | 1. From = 2026-01-20, To = 2026-01-10. 2. Apply. | Filter rejected, user informed, no orders filtered with this range. |
| TC-N02 | Range far longer than 31 days | Orders page open | 1. From = 2026-01-01, To = 2026-06-30. 2. Apply. | Filter rejected, user informed that the range is too long. |
| TC-N03 | Range of one full year | Orders page open | 1. From = 2025-01-01, To = 2025-12-31. 2. Apply. | Rejected, same as TC-N02. |
| TC-N04 | Invalid date value | Orders page open | 1. Type 2026-02-30 in From. 2. Enter a valid To. 3. Apply. | Input rejected as an invalid date, no filter applied, no server error. |
| TC-N05 | Invalid date format | Orders page open | 1. Type "abc" or "10/40/2026" in From. 2. Apply. | Input rejected, no filter applied, no crash. |
| TC-N06 | Invalid 29 February in a non-leap year | Orders page open | 1. From = 2026-02-29. 2. Apply. | Rejected as an invalid date. |
| TC-N07 | Special characters / script in the date field | Orders page open | 1. Enter `<script>alert(1)</script>` and `'; DROP TABLE orders;--` in From. 2. Apply. | Input rejected, nothing executed, no server error, nothing stored. |
| TC-N08 | Invalid range sent directly to the API/URL | Access to the request layer | 1. Call the orders endpoint (or edit the URL query) with a 60-day range. 2. Repeat with To earlier than From. | Server validates too: request rejected with a client error (not 500), no data of the invalid range returned. The rule must not be enforced only in the UI. |
| TC-N09 | Rejected input does not overwrite the previous valid result | A valid filter is already applied (TC-P01) | 1. Change To to make a 60-day range. 2. Apply. | Error shown. Behaviour of the list (kept as before or reset) must match Q4. |
| TC-N10 | Fixing an invalid input | After TC-N01 error | 1. Correct To to 2026-01-25. 2. Apply. | Error disappears and the filter is applied normally. |
| TC-N11 | User without access to some orders | User with limited rights | 1. Apply a valid range. | Only orders the user is allowed to see are shown, the date filter does not widen access. |

## Missing-input scenarios (behaviour not defined, see questions)

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-M01 | Only From filled | Orders page open | 1. From = 2026-01-10, To empty. 2. Apply. | Not defined by the requirement (Q2). To be confirmed before execution. |
| TC-M02 | Only To filled | Orders page open | 1. To = 2026-01-10, From empty. 2. Apply. | Not defined (Q2). |
| TC-M03 | Both empty | Orders page open | 1. Leave both empty. 2. Apply. | Not defined (Q2), probably all orders shown. |
| TC-M04 | Clearing a filter | Filter applied | 1. Clear both fields (or use reset if exists). | Not defined (Q2, Q5). |

## Open questions

1. How is "range longer than 31 days" counted? Inclusive (1 Jan to 31 Jan = 31 days, valid) or by difference (1 Jan to 1 Feb = 31 days, valid)? This changes the exact boundary (TC-B02, B03, B04).
2. Are From and To both mandatory? What happens if only one is filled, or both are empty? Is an open-ended range allowed, and does the 31-day rule then apply?
3. Which date does the filter use: order creation date, payment date, delivery date? And which time zone (user's, browser's or server's) defines the day boundaries for "both dates included" (an order at 23:30 UTC can be the next day for the user)?
4. What should the user see for an invalid input: which error messages, shown where, and is the Apply button disabled or does validation happen on submit? Does the previous list stay or reset after an invalid attempt?
5. Is there a default range when the page opens (for example the last 30 days)? Is there a reset/clear option, and is the filter kept after reload, navigation or in the URL?
6. What are the earliest and latest dates allowed (e.g. future dates, dates before the first order)? Is a future range valid?
7. What date format is expected and which locales are supported (DD/MM/YYYY, MM/DD/YYYY, ISO)? Can the user type the dates or only use a picker?
8. Does the filter combine with other filters (status, customer, search) and with sorting and pagination? Does pagination reset when the range changes?
9. Is the rule also enforced on the API/back end, and what response is expected for violations?
10. Are there performance expectations for the maximum range (31 days) on large data volumes?
11. Accessibility and platform: must the filter work with keyboard only, screen readers, and on mobile browsers?

## Coverage summary

- Positive: 9 cases (P01 to P09)
- Boundary: 11 cases (B01 to B11)
- Negative: 11 cases (N01 to N11)
- Dependent on open questions: 4 cases (M01 to M04), plus B02, B03, B04, B10, B11, N09 whose expected result must be confirmed against the answers above.
