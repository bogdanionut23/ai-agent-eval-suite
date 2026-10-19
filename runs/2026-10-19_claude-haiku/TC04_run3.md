# TC04 - Test cases: date range filter on the orders page

## Assumptions (used to write the cases)

These are my working interpretations. Each one is also listed under Open questions.

- "Both dates are included" means From and To are inclusive. An order dated exactly on From or exactly on To is shown.
- "Longer than 31 days" is counted inclusively. A range is `To - From + 1` calendar days. From = 1 Jan and To = 31 Jan is 31 days and is allowed. From = 1 Jan and To = 1 Feb is 32 days and is rejected.
- "To cannot be earlier than From" allows To = From. A single-day range is valid.
- The filter applies to the order date field.

## Test cases

### Positive

**TC04-01 - Filter with a valid range returns orders inside the range**
- Preconditions: User is logged in and has orders dated on several days, including days before, inside and after the intended range.
- Steps:
  1. Open the orders page.
  2. Enter From = 05.03.2026.
  3. Enter To = 10.03.2026.
  4. Apply the filter.
- Expected result: The list shows only orders dated 05.03.2026 through 10.03.2026. No error message is shown. The From and To values stay in the fields.

**TC04-02 - Orders dated exactly on From and To are included (inclusive bounds)**
- Preconditions: Orders exist on 05.03.2026 (one order) and on 10.03.2026 (one order). Orders also exist on 04.03.2026 and 11.03.2026.
- Steps:
  1. Open the orders page.
  2. Enter From = 05.03.2026, To = 10.03.2026.
  3. Apply the filter.
- Expected result: Orders from 05.03.2026 and 10.03.2026 are both listed. Orders from 04.03.2026 and 11.03.2026 are not listed.

**TC04-03 - Single-day range (From = To) is accepted**
- Preconditions: Orders exist on 12.03.2026.
- Steps:
  1. Enter From = 12.03.2026, To = 12.03.2026.
  2. Apply the filter.
- Expected result: No validation error. Only orders from 12.03.2026 are listed.

**TC04-04 - Range of exactly 31 days is accepted (boundary, maximum)**
- Preconditions: Orders exist on the first and last day of the range and on days outside it.
- Steps:
  1. Enter From = 01.03.2026, To = 31.03.2026 (31 days inclusive).
  2. Apply the filter.
- Expected result: No validation error. Orders from 01.03.2026 through 31.03.2026 are listed, including the boundary days.

**TC04-05 - Filter can be cleared and the full list returns**
- Preconditions: A date range filter is applied and the list is filtered.
- Steps:
  1. Clear both From and To.
  2. Apply the filter (or use the reset control, if one exists).
- Expected result: The unfiltered orders list is shown. No error is shown.

**TC04-06 - Filter works together with other filters and pagination**
- Preconditions: Other filters (for example status) exist on the orders page. More orders exist than fit on one page.
- Steps:
  1. Apply a valid date range.
  2. Apply another filter (for example status).
  3. Go to page 2 of the results.
- Expected result: Results match both the date range and the other filter. Pagination counts reflect the filtered set only.

### Negative

**TC04-07 - To earlier than From is rejected**
- Preconditions: User is on the orders page.
- Steps:
  1. Enter From = 10.03.2026, To = 05.03.2026.
  2. Apply the filter.
- Expected result: The filter is not applied. A validation message says that To cannot be earlier than From. The message is shown next to the To field or as a clear form error. The order list is not changed or reset to an unexpected state.

**TC04-08 - Range of 32 days is rejected (boundary, just over the limit)**
- Preconditions: User is on the orders page.
- Steps:
  1. Enter From = 01.03.2026, To = 01.04.2026 (32 days inclusive).
  2. Apply the filter.
- Expected result: The filter is not applied. A validation message says the range cannot be longer than 31 days.

**TC04-09 - Range far above the limit is rejected**
- Preconditions: User is on the orders page.
- Steps:
  1. Enter From = 01.01.2025, To = 31.12.2026.
  2. Apply the filter.
- Expected result: The filter is not applied. The 31-day validation message is shown. No request returns a multi-year result set.

**TC04-10 - Only From is filled**
- Preconditions: User is on the orders page.
- Steps:
  1. Enter From = 05.03.2026. Leave To empty.
  2. Apply the filter.
- Expected result: Depends on open question Q2. Either the filter is rejected with a message that To is required, or it is applied as "from this date onward". The behaviour must be the one that the specification defines.

**TC04-11 - Only To is filled**
- Preconditions: User is on the orders page.
- Steps:
  1. Leave From empty. Enter To = 10.03.2026.
  2. Apply the filter.
- Expected result: Depends on open question Q2. The behaviour must match the specification.

**TC04-12 - Invalid date text is rejected**
- Preconditions: User is on the orders page.
- Steps:
  1. Enter From = 31.02.2026 (not a real date). Enter To = 10.03.2026.
  2. Apply the filter.
- Expected result: The filter is not applied. A message says the date is invalid. The range check does not run on the invalid value and no crash or blank page appears.

**TC04-13 - Non-date text in a date field is rejected**
- Preconditions: User is on the orders page.
- Steps:
  1. Type "abc" into From. Enter a valid To.
  2. Apply the filter.
- Expected result: The field is rejected or blocked. A validation message is shown. No request with an invalid date is sent.

**TC04-14 - Valid range with no matching orders shows an empty state**
- Preconditions: No orders exist in the chosen range.
- Steps:
  1. Enter From = 01.01.2020, To = 05.01.2020.
  2. Apply the filter.
- Expected result: No error. The page shows an empty state (for example "No orders found"). The filter values stay in the fields.

**TC04-15 - Error from a previous invalid attempt is cleared after a valid correction**
- Preconditions: TC04-07 has been run and the error message is shown.
- Steps:
  1. Change To to 15.03.2026 so the range is valid.
  2. Apply the filter.
- Expected result: The error message disappears. The filtered list is shown.

### Boundary and edge cases

**TC04-16 - Range that crosses a month and a leap-year February**
- Preconditions: Orders exist around 28-29 February 2028 (leap year).
- Steps:
  1. Enter From = 25.02.2028, To = 03.03.2028.
  2. Apply the filter.
- Expected result: Range is valid (7 days). Orders on 29.02.2028 are included.

**TC04-17 - 31-day range across a year boundary**
- Preconditions: Orders exist on 15.12.2026 and on 14.01.2027.
- Steps:
  1. Enter From = 15.12.2026, To = 14.01.2027 (31 days inclusive).
  2. Apply the filter.
- Expected result: Range is accepted. Orders on both boundary dates are included.

**TC04-18 - One day over the limit across a year boundary**
- Preconditions: Same as TC04-17.
- Steps:
  1. Enter From = 15.12.2026, To = 15.01.2027 (32 days inclusive).
  2. Apply the filter.
- Expected result: Range is rejected with the 31-day validation message.

**TC04-19 - Orders near midnight are assigned to the correct day**
- Preconditions: An order is created at 23:59:59 on 10.03.2026 in the user's time zone. Another order is created at 00:00:01 on 11.03.2026.
- Steps:
  1. Enter From = 10.03.2026, To = 10.03.2026.
  2. Apply the filter.
- Expected result: The 23:59:59 order is listed. The 00:00:01 order on 11.03.2026 is not listed.

**TC04-20 - Daylight saving time change inside the range**
- Preconditions: The range includes the DST change day for the user's time zone. Orders exist on both sides of the change.
- Steps:
  1. Enter a range that spans the DST change (for example 25.03.2026 to 30.03.2026 in Europe).
  2. Apply the filter.
- Expected result: The range length is still counted in calendar days. The range is accepted (6 days). No order is dropped or duplicated because of the time shift.

**TC04-21 - Dates in the future**
- Preconditions: Today is 19.10.2026.
- Steps:
  1. Enter From = 20.10.2026, To = 25.10.2026.
  2. Apply the filter.
- Expected result: Depends on open question Q6. Either an empty list is shown without error, or the future date is rejected. The behaviour must be the one that the specification defines.

**TC04-22 - Range where From equals To on the 31-day limit is not a special case**
- Preconditions: none.
- Steps:
  1. Enter From = To = 01.01.2026.
  2. Apply the filter.
- Expected result: Accepted. Orders dated 01.01.2026 are listed. Confirms the minimum range (1 day) is valid.

**TC04-23 - Date picker and typed input give the same result**
- Preconditions: Orders exist on 05.03.2026 and 10.03.2026.
- Steps:
  1. Set From and To by typing the dates.
  2. Apply and note the list.
  3. Reset. Set the same dates with the date picker.
  4. Apply and compare.
- Expected result: Both methods produce the same list.

**TC04-24 - Filter values survive a page reload or back navigation (if this is required)**
- Preconditions: A valid range is applied.
- Steps:
  1. Reload the orders page, or navigate to another page and then back.
- Expected result: Depends on open question Q8. Either the filter is kept, or the page opens unfiltered. The behaviour must match the specification.

### Security and access

**TC04-25 - Date range does not expose orders of other users**
- Preconditions: User A and User B both have orders on the same date. User A is logged in.
- Steps:
  1. As User A, apply a date range that includes the shared date.
- Expected result: Only User A's orders are listed. No order of User B appears.

**TC04-26 - Date range cannot be bypassed by a direct request**
- Preconditions: The filter parameters are visible in the request (for example in the URL or API call).
- Steps:
  1. Send a request with To earlier than From.
  2. Send a request with a 32-day range.
- Expected result: The server rejects both requests with a clear error (for example HTTP 400). The client-side check is not the only control. No orders are returned for the invalid request.

## Open questions

- **Q1 - Counting of 31 days.** Is the limit inclusive (From to To counted as 31 days, so 01.03 to 31.03 is OK) or exclusive (To - From <= 31, so 01.03 to 01.04 is OK)? The spec says both dates are included, so I used inclusive. Please confirm. This changes TC04-04, TC04-08 and TC04-18.
- **Q2 - Open-ended ranges.** Is From without To allowed (open to today or to the future)? Is To without From allowed? Or are both required? (TC04-10, TC04-11)
- **Q3 - Which date field is filtered?** Order creation date, order date shown in the list, shipping date, or another field? Is it the same field the list is sorted by?
- **Q4 - Time zone.** Which time zone is used to decide the day of an order: the user's, the server's, or UTC? (TC04-19, TC04-20)
- **Q5 - Error presentation.** Where exactly is the validation message shown (next to the field, above the list, as a toast)? Is the Apply button disabled, or does the error appear after clicking Apply? What is the exact message text?
- **Q6 - Future dates.** Are future dates allowed? (TC04-21)
- **Q7 - Date format and locale.** Which date format is used in the input (DD.MM.YYYY, MM/DD/YYYY, locale-based)? Is typed input supported, or only a date picker?
- **Q8 - Persistence.** Should the filter be kept after reload, navigation or in the URL? (TC04-24)
- **Q9 - Interaction with other filters, sorting and pagination.** Does changing the date range reset the page number to 1? Are other filters combined with AND?
- **Q10 - Limits on the date picker.** Should the picker itself prevent choosing a To date more than 31 days after From, or only show a message after applying?
- **Q11 - Date range on the server.** Is there an upper bound on the date itself (for example, no dates before the company founding or before data retention)?
- **Q12 - Empty input.** Is clicking Apply with both fields empty a valid "no filter" action or an error?
