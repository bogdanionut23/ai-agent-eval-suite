# TC04 - Test cases: orders page date range filter

## Assumptions used for the test cases

These are not stated in the requirement. The cases below assume them; the open questions at the end cover them.

- "Both dates are included" means From and To are inclusive. Orders dated exactly From and exactly To are returned.
- "Cannot be longer than 31 days" is counted inclusively, so From = D1 and To = D1 + 30 days (31 calendar days) is the maximum allowed range. A range of 32 calendar days is rejected.
- "To cannot be earlier than From" means To = From is allowed; To < From is rejected.
- The filter applies to the order date. Dates are calendar dates (no time component), in the user's displayed timezone.

## Test cases

### Positive

**TC04-01 - Filter returns orders inside a valid range, including both boundary dates**
- Preconditions: User is logged in and can open the orders page. Orders exist dated 2026-03-09 (day before range), 2026-03-10 (From), 2026-03-15 (inside), 2026-03-20 (To), 2026-03-21 (day after range).
- Steps:
  1. Open the orders page.
  2. Set From = 2026-03-10.
  3. Set To = 2026-03-20.
  4. Apply the filter.
- Expected result: The list shows the orders dated 2026-03-10, 2026-03-15 and 2026-03-20. The orders dated 2026-03-09 and 2026-03-21 are not shown. No validation error is shown.

**TC04-02 - Single-day range (From = To) is accepted**
- Preconditions: Orders exist on 2026-03-15 and on 2026-03-16.
- Steps:
  1. Set From = 2026-03-15 and To = 2026-03-15.
  2. Apply the filter.
- Expected result: The filter is accepted. Only orders dated 2026-03-15 are shown.

**TC04-03 - Range of exactly 31 days (inclusive) is accepted**
- Preconditions: Orders exist on 2026-01-01 and on 2026-01-31.
- Steps:
  1. Set From = 2026-01-01 and To = 2026-01-31 (31 calendar days, inclusive).
  2. Apply the filter.
- Expected result: The filter is accepted with no error. Orders dated 2026-01-01 and 2026-01-31 are both shown.

**TC04-04 - Leap-year range of exactly 31 days is accepted**
- Preconditions: Orders exist on 2028-02-15 and 2028-03-16.
- Steps:
  1. Set From = 2028-02-15 and To = 2028-03-16. This is 15 days in February (including 29 Feb) plus 16 days in March, 31 days in total.
  2. Apply the filter.
- Expected result: The filter is accepted with no error. Orders on both dates are shown.

**TC04-05 - Orders just outside the range are excluded**
- Preconditions: Orders exist on 2026-04-04 (From - 1 day) and 2026-04-11 (To + 1 day), with at least one order inside the range.
- Steps:
  1. Set From = 2026-04-05 and To = 2026-04-10.
  2. Apply the filter.
- Expected result: Neither the 2026-04-04 order nor the 2026-04-11 order is shown. Orders inside the range are shown.

**TC04-06 - Valid range with no matching orders shows the empty state**
- Preconditions: No orders exist between 2026-06-01 and 2026-06-03.
- Steps:
  1. Set From = 2026-06-01 and To = 2026-06-03.
  2. Apply the filter.
- Expected result: No validation error. The page shows the empty-results message (exact text per the UI spec). No orders are listed.

**TC04-07 - Clearing the filter restores the full order list**
- Preconditions: A valid date range filter is applied and the list is narrowed.
- Steps:
  1. Apply From = 2026-03-10 and To = 2026-03-20.
  2. Clear both date fields (or use the reset control) and apply.
- Expected result: The full, unfiltered order list is shown again.

### Negative

**TC04-08 - To earlier than From is rejected**
- Preconditions: Orders exist in March 2026.
- Steps:
  1. Set From = 2026-03-20.
  2. Set To = 2026-03-10.
  3. Apply the filter.
- Expected result: The filter is not applied. A validation error is shown on the To field (or the filter form) stating that To cannot be earlier than From. The previously displayed list is unchanged.

**TC04-09 - To one day before From is rejected (lower boundary of the ordering rule)**
- Preconditions: Orders exist on 2026-03-14 and 2026-03-15.
- Steps:
  1. Set From = 2026-03-15.
  2. Set To = 2026-03-14.
  3. Apply the filter.
- Expected result: Rejected with the same error as TC04-08. No orders are filtered.

**TC04-10 - Range of 32 days (inclusive) is rejected (upper boundary of the length rule)**
- Preconditions: Orders exist on 2026-01-01 and 2026-02-01.
- Steps:
  1. Set From = 2026-01-01 and To = 2026-02-01 (32 calendar days, inclusive).
  2. Apply the filter.
- Expected result: Rejected with a validation error stating that the range cannot be longer than 31 days. No filter is applied, and the list is not narrowed to this range.

**TC04-11 - Range far beyond the limit is rejected**
- Preconditions: Orders exist across several years.
- Steps:
  1. Set From = 2024-01-01 and To = 2026-12-31.
  2. Apply the filter.
- Expected result: Rejected with the "longer than 31 days" error. No partial or truncated results are shown.

**TC04-12 - Invalid date text is rejected**
- Preconditions: None.
- Steps:
  1. Enter From = 2026-02-30 (non-existent date). Apply the filter.
  2. Enter From = "abc" (non-date text). Apply the filter.
  3. Repeat with To set to an invalid value and From valid.
- Expected result: Each input is rejected with a format or validity error on the offending field. No filter is applied.

**TC04-13 - Range that is both too long and reversed shows a single, clear error**
- Preconditions: None.
- Steps:
  1. Set From = 2026-03-31 and To = 2026-01-01 (reversed and more than 31 days apart).
  2. Apply the filter.
- Expected result: The filter is rejected. At least one clear error is shown that names the problem. Which message appears first is an open question (see Q6); the test checks only that the filter is not applied and an error is visible.

**TC04-14 - Changing From so that the current To becomes earlier is handled**
- Preconditions: From = 2026-03-10 and To = 2026-03-20 are applied.
- Steps:
  1. Change From to 2026-03-25 without changing To.
  2. Apply the filter.
- Expected result: The filter is rejected because To (2026-03-20) is earlier than From (2026-03-25). The error is shown. The state of the previously applied list is consistent with the UI spec (see Q8).

### Boundary

**TC04-15 - Order timestamped 23:59 on the To date is included**
- Preconditions: An order exists with timestamp 2026-03-20 23:59 (user's timezone). Another order exists at 2026-03-21 00:00.
- Steps:
  1. Set From = 2026-03-10 and To = 2026-03-20.
  2. Apply the filter.
- Expected result: The 2026-03-20 23:59 order is shown. The 2026-03-21 00:00 order is not shown.

**TC04-16 - Order timestamped 00:00 on the From date is included**
- Preconditions: An order exists with timestamp 2026-03-10 00:00 and another at 2026-03-09 23:59.
- Steps:
  1. Set From = 2026-03-10 and To = 2026-03-20.
  2. Apply the filter.
- Expected result: The 2026-03-10 00:00 order is shown. The 2026-03-09 23:59 order is not shown.

**TC04-17 - Boundary of the length limit across a month end**
- Preconditions: Orders exist on 2026-04-01 and 2026-05-01.
- Steps:
  1. Set From = 2026-04-01 and To = 2026-05-01 (31 days inclusive: 30 days in April plus 1 day in May).
  2. Apply the filter.
- Expected result: Accepted. Orders on both dates are shown.

**TC04-18 - Boundary of the length limit at 31 days with a 1-day-later To**
- Preconditions: Orders exist on 2026-01-01 and 2026-02-01.
- Steps: Same as TC04-10 but as a test of the boundary pair: TC04-03 (31 days) must pass and TC04-10 (32 days) must fail when run together.
- Expected result: Both results match the expectations of TC04-03 and TC04-10.

## Open questions

These points are not defined in the requirement. Test cases that depend on them must not be treated as final until they are answered.

1. **Length counting:** Is the 31-day limit inclusive (From to To counts both days, so 31 calendar days is the maximum) or exclusive (To - From <= 31, so 32 calendar days is allowed)? The cases above assume inclusive. TC04-03, TC04-04, TC04-10, TC04-11, TC04-17 and TC04-18 depend on this.
2. **Open-ended ranges:** Can From or To be left empty? If only From is set, is To treated as today, as no upper bound, or as a required field? If only To is set, is From treated as no lower bound or as required? No test covers this yet, because the requirement is silent.
3. **Timezone and time component:** Are dates compared as calendar dates, or as timestamps? Which timezone applies (user's local, server, or the order's timezone)? TC04-15 and TC04-16 assume user-local calendar dates.
4. **Which date is filtered:** Order placed date, paid date, shipped date, or another date field? The requirement says "orders" only.
5. **Date input and validation details:** Accepted date format and locale (for example 2026-03-10 vs 10/03/2026)? Is there a minimum or maximum allowed date (for example no dates in the future, or no dates before a system launch date)?
6. **Error precedence:** When a range is both reversed and longer than 31 days (TC04-13), which error is shown? Is it one message or both? Where is the error shown (on the field, or as a form-level message)?
7. **Validation timing:** Is validation run when a field changes, or only when the filter is applied? Can an invalid filter be applied at all?
8. **Field-dependent re-validation:** When From is changed to a date after To (TC04-14), is To automatically adjusted, cleared, or does it stay and show an error? Is the error shown immediately or on apply?
9. **Invalid input behavior:** When a range is rejected, is the previously applied list kept, or is the list cleared? (TC04-08 and TC04-10 assume the list is kept.)
10. **Equal dates:** TC04-02 assumes To = From is valid, because "cannot be earlier" allows equality. Please confirm.
11. **Persistence:** Should the filter survive page reload, navigation away and back, or pagination? Is it stored in the URL?
12. **Combination with other filters and sorting:** Does the date range combine with other filters (status, search) using AND? Does changing the range reset pagination to page 1?
13. **Empty-state text and behaviour:** Exact wording of the empty-results message is not given.
14. **Accessibility and input method:** Is a date picker required, text entry allowed, or both? Does the picker prevent invalid dates from being selected (for example, To before From being disabled)? If so, TC04-08 and TC04-09 would test the picker differently.
15. **Maximum result size:** Is there a maximum number of orders returned for a range, and how is pagination handled for long ranges?
