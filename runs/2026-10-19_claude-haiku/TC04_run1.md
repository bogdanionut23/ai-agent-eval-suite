# TC04 Run 1: Test cases for the orders page date range filter

## Requirement (as given)

On the orders page a user can filter orders by a date range (From, To). Both dates are included. The range cannot be longer than 31 days. To cannot be earlier than From.

## Working assumptions

These assumptions are used to write the expected results. Each one is also listed under open questions. Please confirm or correct them before the cases are used.

- A. Range length is counted in calendar days, inclusive of both ends: length = (To - From) + 1. So From 2026-10-01 to To 2026-10-31 is 31 days (allowed), and To 2026-11-01 is 32 days (rejected).
- B. Dates are date-only (no time part) and are compared in the user's local time zone.
- C. The date filter applies to the order creation date.
- D. Invalid input shows an inline error next to the field and the filter is not applied. The previous result list stays unchanged.
- E. Both From and To are required for the filter to run.

## Test cases

### Positive

**TC04-001: Orders inside the range are shown, with both ends included**
- Type: Positive
- Preconditions: Logged in as a user with orders dated 2026-09-30, 2026-10-01, 2026-10-15, 2026-10-31, 2026-11-01. The orders page is open with no filter set.
- Steps:
  1. Enter From = 2026-10-01.
  2. Enter To = 2026-10-31.
  3. Apply the filter.
- Expected result: The list shows the orders dated 2026-10-01, 2026-10-15 and 2026-10-31. The orders dated 2026-09-30 and 2026-11-01 are not shown. The range length is 31 days, which is within the limit.

**TC04-002: Single-day range (From equals To) returns that day's orders**
- Type: Positive (boundary of "To not earlier than From")
- Preconditions: Orders exist on 2026-10-15 and on the days before and after it.
- Steps:
  1. Enter From = 2026-10-15.
  2. Enter To = 2026-10-15.
  3. Apply the filter.
- Expected result: The filter is accepted with no error. Only orders dated 2026-10-15 are shown.

**TC04-003: Range of 30 days is accepted**
- Type: Positive (just under the limit)
- Preconditions: Orders exist on the first and last day of the range and on the days just outside it.
- Steps:
  1. Enter From = 2026-10-01.
  2. Enter To = 2026-10-30.
  3. Apply the filter.
- Expected result: The filter is accepted. Orders on 2026-10-01 through 2026-10-30 are shown. No error is shown.

**TC04-004: Dates entered with the date picker and by typing give the same result**
- Type: Positive (input methods)
- Preconditions: Same data as TC04-001.
- Steps:
  1. Set From and To by using the date picker. Apply and note the result list.
  2. Clear the filter. Type the same two dates in the input fields. Apply.
- Expected result: Both methods produce the same list, with the same orders and the same count.

**TC04-005: Clearing the filter restores the full order list**
- Type: Positive
- Preconditions: A valid range filter is applied (TC04-001 steps 1 to 3).
- Steps:
  1. Clear both From and To (or use the reset/clear control).
  2. Apply or let the list refresh.
- Expected result: All orders are shown again, with no date restriction and no error.

**TC04-006: Date filter works together with pagination**
- Type: Positive
- Preconditions: More orders in the range than fit on one page (for example, 45 orders in 2026-10-01 to 2026-10-31 with a page size of 20).
- Steps:
  1. Apply the valid range filter.
  2. Go to page 2.
  3. Go to page 3.
- Expected result: Every page shows only orders inside the range. No order appears twice across pages. The total count matches the number of orders in the range.

### Boundary

**TC04-007: Range of exactly 31 days is accepted**
- Type: Boundary (at the limit, inclusive)
- Preconditions: Orders exist on 2026-10-01 and 2026-10-31.
- Steps:
  1. Enter From = 2026-10-01.
  2. Enter To = 2026-10-31.
  3. Apply the filter.
- Expected result: The filter is accepted with no error. The orders on both 2026-10-01 and 2026-10-31 are shown.

**TC04-008: Range of 32 days is rejected**
- Type: Boundary (just over the limit)
- Preconditions: Orders exist on 2026-10-01 and 2026-11-01.
- Steps:
  1. Enter From = 2026-10-01.
  2. Enter To = 2026-11-01.
  3. Apply the filter.
- Expected result: An error says the range cannot be longer than 31 days. The filter is not applied. The previous list stays unchanged.

**TC04-009: Leap year boundary (31 days across February 2028)**
- Type: Boundary
- Preconditions: Orders exist on 2028-01-31 and 2028-03-01.
- Steps:
  1. Enter From = 2028-01-31 and To = 2028-03-01. Apply.
  2. Change To to 2028-03-02 (32 days). Apply.
- Expected result: Step 1 is accepted (31 days, Feb 29 counted correctly). Step 2 is rejected with the "longer than 31 days" error.

**TC04-010: Range that crosses a year boundary**
- Type: Boundary
- Preconditions: Orders exist on 2025-12-25 and 2026-01-10.
- Steps:
  1. Enter From = 2025-12-25 and To = 2026-01-10 (17 days). Apply.
- Expected result: The filter is accepted. Orders from both years in the range are shown, and none outside it.

**TC04-011: To one day before From is rejected**
- Type: Boundary (just outside "To not earlier than From")
- Preconditions: Orders exist on 2026-10-09 and 2026-10-10.
- Steps:
  1. Enter From = 2026-10-10.
  2. Enter To = 2026-10-09.
  3. Apply the filter.
- Expected result: An error says To cannot be earlier than From. The filter is not applied.

**TC04-012: Order at the first and last second of the range is included; orders just outside are excluded**
- Type: Boundary (time of day)
- Preconditions: Orders exist at 2026-10-01 00:00:00, 2026-10-31 23:59:59, 2026-09-30 23:59:59 and 2026-11-01 00:00:00 (local time).
- Steps:
  1. Enter From = 2026-10-01 and To = 2026-10-31. Apply.
- Expected result: Orders at 2026-10-01 00:00:00 and 2026-10-31 23:59:59 are shown. Orders at 2026-09-30 23:59:59 and 2026-11-01 00:00:00 are not shown.

**TC04-013: Order near midnight is grouped by the user's local date**
- Type: Boundary (time zone)
- Preconditions: The user's browser time zone is set to UTC+2. An order was placed at 2026-10-31 23:30 local time.
- Steps:
  1. Enter From = 2026-10-01 and To = 2026-10-31. Apply.
- Expected result: The order is shown, because its local date is 2026-10-31. (Depends on open question Q2. Adjust the expected result if the business uses another time zone.)

### Negative

**TC04-014: Non-date text in From is rejected**
- Type: Negative
- Preconditions: Orders page is open with no filter set.
- Steps:
  1. Type "abc" in From.
  2. Enter To = 2026-10-31.
  3. Apply.
- Expected result: An inline validation error is shown on the From field. The filter is not applied. The list is unchanged.

**TC04-015: Non-existent calendar date is rejected**
- Type: Negative
- Preconditions: Orders page is open.
- Steps:
  1. Enter From = 2026-02-30 (typed).
  2. Enter To = 2026-03-05. Apply.
- Expected result: An error says the From date is not a valid date. The filter is not applied.

**TC04-016: Incomplete date is rejected**
- Type: Negative
- Preconditions: Orders page is open.
- Steps:
  1. Enter From = 2026-10 (incomplete).
  2. Enter To = 2026-10-31. Apply.
- Expected result: An error shows that the From date is not valid. The filter is not applied.

**TC04-017: Range much longer than 31 days is rejected**
- Type: Negative
- Preconditions: Orders exist across 2026.
- Steps:
  1. Enter From = 2026-01-01 and To = 2026-12-31. Apply.
- Expected result: An error says the range cannot be longer than 31 days. The filter is not applied.

**TC04-018: Reversed range shows only the "To before From" error**
- Type: Negative (error precedence)
- Preconditions: Orders page is open.
- Steps:
  1. Enter From = 2026-11-20 and To = 2026-10-01. Apply.
- Expected result: Only the "To cannot be earlier than From" error is shown. The "longer than 31 days" error is not shown, because the length of a reversed range is not meaningful.

**TC04-019: Error clears after the range is corrected**
- Type: Negative (recovery)
- Preconditions: TC04-011 has produced the reversed-range error.
- Steps:
  1. Change To to 2026-10-20 (which is after From = 2026-10-10).
  2. Apply.
- Expected result: The error disappears. The filter is applied and the orders in 2026-10-10 to 2026-10-20 are shown.

**TC04-020: Moving From after To shows an error and does not silently change To**
- Type: Negative
- Preconditions: From = 2026-10-01 and To = 2026-10-20 are set and applied.
- Steps:
  1. Change From to 2026-10-25 without changing To.
- Expected result: The constraint is enforced: an error shows that To cannot be earlier than From. The value of To is not changed without the user's action. (Depends on open question Q6.)

**TC04-021: Only one of the two dates is entered**
- Type: Negative (incomplete input)
- Preconditions: Orders page is open.
- Steps:
  1. Enter From = 2026-10-01 and leave To empty. Apply.
  2. Clear From, enter To = 2026-10-31 and leave From empty. Apply.
- Expected result (per assumption E): Each attempt shows a validation message that asks for the missing date, and the filter is not applied. (Depends on open question Q3.)

**TC04-022: Range with no matching orders shows an empty state, not an error**
- Type: Negative (no results)
- Preconditions: No orders exist between 2026-03-01 and 2026-03-10.
- Steps:
  1. Enter From = 2026-03-01 and To = 2026-03-10. Apply.
- Expected result: The range is accepted. No error is shown. The page shows an empty-state message such as "No orders in this date range" and the count is 0.

**TC04-023: Range filter combined with another filter (for example, status)**
- Type: Negative (filter interaction)
- Preconditions: Orders exist in the range with statuses "shipped" and "cancelled".
- Steps:
  1. Apply the range 2026-10-01 to 2026-10-31.
  2. Apply the status filter "shipped".
- Expected result: Only orders that are both in the range and "shipped" are shown (AND logic). (Depends on open question Q8.)

### Usability and accessibility

**TC04-024: Date fields can be used with the keyboard and read by a screen reader**
- Type: Positive (accessibility)
- Preconditions: Orders page is open.
- Steps:
  1. Tab to the From field, type a date, tab to To, type a date, and press Enter to apply.
  2. Use a screen reader to read the fields and the error text.
- Expected result: Both fields are reachable with the keyboard and have visible labels "From" and "To". Errors are announced by the screen reader.

## Open questions

Q1. How is the 31-day limit counted? Is the length inclusive (To - From + 1, so 31 days is allowed) or the plain difference (To - From, so 31 days is allowed and 32 days is the first rejected value)? The requirement says "both dates are included", which suggests inclusive, but the exact limit needs to be confirmed. TC04-007 and TC04-008 depend on this.

Q2. Which time zone defines the order date? Is the user's local time used, or the server or business time zone? Are orders stored with a time part, or are they date-only? TC04-012 and TC04-013 depend on this.

Q3. What happens when only one of the two dates is entered? Is the filter blocked with a message, or is the other side treated as open-ended (for example, "From only" means everything on or after From)? If open-ended ranges are allowed, the 31-day limit may not apply to them, and TC04-021 would change.

Q4. What happens when both fields are empty and Apply is pressed? Is the full list shown, or is a validation message shown?

Q5. Are future dates allowed? Is there a minimum date (for example, the earliest order date or a fixed year)? Is there any limit on how far back a user can filter?

Q6. When From is moved after To, is To auto-corrected, or is an error shown? Is the Apply button disabled while the range is invalid, or is the error only shown after Apply? TC04-020 and TC04-011 depend on this.

Q7. What is the exact error text for each rule (empty field, invalid format, longer than 31 days, To before From)? Are these messages final, and do they need translation?

Q8. Does the date filter combine with other filters using AND? Is the date filter kept when the user changes a different filter or changes pages? Is the filter kept in the URL or session so it survives a reload? TC04-006 and TC04-023 depend on this.

Q9. Is the date picker allowed to show or select dates that the rules reject (for example, dates more than 31 days after From, or dates before From)? Or should the picker block them?

Q10. Does the date range apply to the order creation date, or to another date (order date, shipping date, last updated)? The requirement says "orders" without naming the field. TC04-001 and TC04-012 assume the creation date.
