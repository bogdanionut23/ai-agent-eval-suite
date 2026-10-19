# TC04: Test cases for the orders page date range filter

Requirement: On the orders page a user can filter orders by a date range (From, To). Both dates are included. The range cannot be longer than 31 days. To cannot be earlier than From.

## Open questions

These are listed separately. I did not invent behaviour for them. Test cases that depend on an answer are marked "Depends on Q#".

1. **How is "31 days" counted?** There are two readings.
   - (a) To minus From is at most 31, so Jan 1 to Feb 1 is allowed. That is 32 calendar days when both ends are included.
   - (b) The inclusive count of days is at most 31, so Jan 1 to Jan 31 is allowed and Jan 1 to Feb 1 is rejected.
   - Since both dates are included, this is the main ambiguity. The boundary tests (TC-B01 to TC-B04) depend on it.
2. **What happens when only one date is filled in?** Is From alone "from this date onward"? Is To alone "up to this date"? Or are both required?
3. **What is the validation behaviour?**
   - Is the invalid input blocked in the date picker (dates disabled)?
   - Is an inline error shown?
   - Is the Apply button disabled?
   - Is an existing result list kept or cleared when the range is invalid?
   - What is the exact error text for each rule?
4. **Can the range be in the future?** Can From or To be later than today? Is there a minimum date, for example before the first order?
5. **What is the date field format and locale?** Are typed input and picker both supported? What format applies (DD.MM.YYYY, MM/DD/YYYY or ISO)? How is invalid text such as "31/02/2025" handled?
6. **Which timezone defines the day boundaries?** Is it the user's, the server's or UTC? Is the order timestamp compared at 00:00:00 of From through 23:59:59.999 of To?
7. **What is the default state?** Is there a default range on page load, or is the list unfiltered? Is there a Clear/Reset action, and does it reset the data too?
8. **Does the filter persist?** Is it kept across reload, pagination, sorting, other filters and navigation (URL parameters)?
9. **What happens when From is changed after To is set?** Is To auto-adjusted, cleared or flagged when the new From makes the range invalid?
10. **What is shown when no orders match?** Is there an empty state message?
11. **Is there a performance or volume expectation** for large result sets (paging, limits)?

## Assumptions used in the cases below

- Dates are entered in the date fields and applied with an "Apply" (or equivalent) action.
- Where the 31-day limit matters, I use reading (a), where the To - From difference is at most 31. Reading (b) shifts the boundary by one day. Both are written out in the boundary cases so they can be confirmed either way once Q1 is answered.
- Test data is as follows.
  - User U1 has orders on these dates: 2025-01-01, 2025-01-15, 2025-01-31, 2025-02-01, 2025-02-02, 2025-03-01 and 2025-03-02.
  - Two of the orders are on 2025-01-15 (one at 00:00:00, one at 23:59:59).

## Positive cases

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-P01 | Filter by a valid range within one month | Logged in as U1, orders page open | 1. Set From = 2025-01-10. 2. Set To = 2025-01-20. 3. Apply. | No validation error. Only orders dated 2025-01-10 to 2025-01-20 inclusive are listed, including both orders on 2025-01-15. |
| TC-P02 | From date is included | U1 has an order on 2025-01-15 | 1. From = 2025-01-15, To = 2025-01-20. 2. Apply. | Orders dated 2025-01-15 (both, including the one at 00:00:00) are in the list. |
| TC-P03 | To date is included | U1 has an order on 2025-01-31 (one at 23:59:59 would be ideal) | 1. From = 2025-01-20, To = 2025-01-31. 2. Apply. | Orders dated 2025-01-31, including late-evening ones, are in the list. |
| TC-P04 | Orders outside the range are excluded | Same data | 1. From = 2025-01-10, To = 2025-01-20. 2. Apply. | Orders on 2025-01-01, 2025-01-31 and later are not listed. |
| TC-P05 | Single-day range (From = To) | Two orders on 2025-01-15 | 1. From = 2025-01-15, To = 2025-01-15. 2. Apply. | Allowed (To is not earlier than From). Both orders of 2025-01-15 are shown and no others. |
| TC-P06 | Range crossing a month boundary | Orders on 2025-01-31 and 2025-02-01 | 1. From = 2025-01-31, To = 2025-02-01. 2. Apply. | Both orders are listed. |
| TC-P07 | Range crossing a year boundary | Orders on 2024-12-31 and 2025-01-01 | 1. From = 2024-12-30, To = 2025-01-02. 2. Apply. | Orders from both years are listed. |
| TC-P08 | Range including 29 Feb in a leap year | Orders on 2024-02-28, 2024-02-29, 2024-03-01 | 1. From = 2024-02-28, To = 2024-03-01. 2. Apply. | Allowed. All three orders are listed. 29 Feb is treated as a valid date. |
| TC-P09 | Change the range after a first filter | A filter is already applied | 1. Change To to a different valid date. 2. Apply. | The list refreshes to match the new range and the previous result does not remain. |
| TC-P10 | Range with no matching orders | U1 has no orders in 2025-04-01 to 2025-04-10 | 1. Set that range. 2. Apply. | No validation error. An empty result or empty state is shown (see Q10) and no orders from other dates leak in. |
| TC-P11 | Filter only shows the user's own orders | Users U1 and U2 both have orders in the range | 1. As U1, apply a range that covers both users' orders. | Only U1's orders are listed. |

## Boundary cases (31-day limit)

All of these depend on Q1. The two columns show the expected result under each reading.

| ID | Title | Steps | Expected under reading (a): To - From at most 31 | Expected under reading (b): inclusive count at most 31 |
|---|---|---|---|---|
| TC-B01 | Inclusive count of 30 days | From = 2025-01-01, To = 2025-01-30 | Accepted | Accepted |
| TC-B02 | Inclusive count of exactly 31 days | From = 2025-01-01, To = 2025-01-31 | Accepted | Accepted |
| TC-B03 | Inclusive count of 32 days (difference of 31) | From = 2025-01-01, To = 2025-02-01 | Accepted | Rejected with an error |
| TC-B04 | Inclusive count of 33 days (difference of 32) | From = 2025-01-01, To = 2025-02-02 | Rejected with an error | Rejected with an error |
| TC-B05 | 31-day boundary across February (non-leap) | From = 2025-02-01, To = 2025-03-03 (inclusive count 31) and then To = 2025-03-04 (count 32) | Count 31 accepted; count 32 accepted (difference 31); To = 2025-03-05 rejected | Count 31 accepted; count 32 rejected |
| TC-B06 | 31-day boundary across a leap February | From = 2024-02-01, To = 2024-03-02 (count 31) and To = 2024-03-03 (count 32) | Same pattern as TC-B05, shifted by one day | Same pattern as TC-B05, shifted by one day |
| TC-B07 | 31-day boundary across a year end | From = 2024-12-15, To = 2025-01-14 (count 31) and To = 2025-01-15 (count 32) | 01-14 and 01-15 accepted; 01-16 rejected | 01-14 accepted; 01-15 rejected |
| TC-B08 | 31-day boundary across a DST change (user timezone with DST) | From and To around the DST switch, with a count of 31 and then 32 | Result is the same as the same boundary away from DST. The DST hour does not make a valid range fail or an invalid range pass. | Same |
| TC-B09 | To one day before From | From = 2025-01-10, To = 2025-01-09 | Rejected with an error: To is earlier than From | Same |
| TC-B10 | To same day as From | From = 2025-01-10, To = 2025-01-10 | Accepted (see TC-P05) | Accepted |
| TC-B11 | Earliest and latest supported dates | Use the minimum and maximum dates the picker allows (see Q4) | Behaviour matches the documented limits | Same |

## Negative cases

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-N01 | Range clearly longer than 31 days | Orders page open | 1. From = 2025-01-01, To = 2025-06-30. 2. Apply. | Rejected. An error message about the maximum range is shown (text per Q3). The filter is not applied. |
| TC-N02 | To earlier than From | Orders page open | 1. From = 2025-02-10, To = 2025-02-01. 2. Apply. | Rejected. An error says To cannot be earlier than From. The filter is not applied. |
| TC-N03 | Both rules broken at once (To before From, and the span is more than 31 days) | None | 1. From = 2025-06-30, To = 2025-01-01. 2. Apply. | Rejected. A clear error is shown, and the message is not misleading (see Q3 on which message wins). |
| TC-N04 | Change From so that an existing valid range becomes invalid | From = 2025-01-10, To = 2025-01-20 is applied | 1. Change From to 2025-01-25. | The invalid state is handled consistently (see Q9): the error is shown or To is adjusted. No stale or wrong results are shown silently. |
| TC-N05 | Change To so that the range becomes longer than 31 days | A valid range is applied | 1. Change To to a date more than 31 days after From. 2. Apply. | Rejected with the max-range error. The previous valid results are handled per Q3. |
| TC-N06 | Invalid calendar date typed in | Typing is supported (Q5) | 1. Type 2025-02-30 in From. 2. Apply. | Rejected. A format or invalid-date error is shown and the filter is not applied. |
| TC-N07 | Non-date text | Typing is supported | 1. Type "abc" in From. 2. Apply. | Rejected with an error. No server error and no crash. |
| TC-N08 | Wrong date format | Typing is supported | 1. Type a date in a format other than the expected one. 2. Apply. | Rejected, or interpreted exactly as documented (see Q5). It is never silently interpreted as a different date. |
| TC-N09 | Only From filled in | None | 1. From = 2025-01-10, To empty. 2. Apply. | Behaviour per Q2 (not specified). Whatever happens, there is no server error and no unhandled exception. |
| TC-N10 | Only To filled in | None | 1. From empty, To = 2025-01-10. 2. Apply. | Behaviour per Q2. No error page. |
| TC-N11 | Both empty | None | 1. Leave both empty. 2. Apply. | Behaviour per Q2 and Q7 (for example the unfiltered list). No error page. |
| TC-N12 | Bypass the UI limit by calling the API directly | API/test tool available | 1. Send the orders request with a range of 60 days (or To before From). | The server rejects it with a validation error (4xx) too. The limit is not enforced in the UI only. |
| TC-N13 | Injection in a date parameter | API/test tool available | 1. Send `from=2025-01-01'; DROP TABLE orders;--`. | Rejected with a validation error. No server error and no data change. |
| TC-N14 | Extreme dates | Typing or API | 1. From = 0001-01-01 and To = 9999-12-31. | Rejected (over 31 days). No overflow or crash. |

## Non-functional and UI behaviour

| ID | Title | Steps | Expected result |
|---|---|---|---|
| TC-U01 | The picker prevents or flags invalid choices | 1. Pick From, then open the To picker. | Depends on Q3: dates before From and more than 31 days after From are disabled, or are accepted and then flagged. Behaviour is consistent. |
| TC-U02 | Error message is clear and accessible | 1. Trigger each error (TC-N01, TC-N02). | The message names the rule broken, is visible next to the fields, and is read by a screen reader. The invalid field is marked. |
| TC-U03 | Error clears after correction | 1. Trigger an error. 2. Fix the dates. 3. Apply. | The error disappears and the correct results are shown. |
| TC-U04 | Keyboard use | 1. Use only the keyboard to set both dates and apply. | The whole flow works and the focus order is logical. |
| TC-U05 | Filter and pagination | Range returns more than one page of orders | 1. Apply the range. 2. Go to page 2. | Page 2 keeps the same range. The total count matches the range. |
| TC-U06 | Filter and sorting | A range is applied | 1. Change the sort order. | The range stays applied. |
| TC-U07 | Persistence on reload | A range is applied | 1. Reload the page. | Behaviour per Q8. It is consistent and does not show a mismatch between the fields and the list. |
| TC-U08 | Day boundaries and timezone | Orders at 23:59 and 00:01 around midnight, and a user in a timezone other than the server's | 1. Filter on exactly that day. | Orders are grouped by the day defined in Q6, and an order is never shown on a different day than the one displayed in its own row. |
| TC-U09 | Responsive layout | Open on a mobile viewport | 1. Set the range and apply. | Fields, picker and error are usable and not cut off. |
| TC-U10 | Response time for a full 31-day range | Large order history | 1. Apply a maximum-length range. | Results load within the agreed time (to be set, see Q11). |

## Coverage summary

- Positive: 11 cases (TC-P01 to TC-P11).
- Boundary: 11 cases (TC-B01 to TC-B11). The 31-day boundary is tested at 30, 31, 32 and 33 days, and across leap-year, year-end and DST edges.
- Negative: 14 cases (TC-N01 to TC-N14).
- UI and non-functional: 10 cases (TC-U01 to TC-U10).
- Open questions: 11. Q1 (how the 31 days are counted) should be answered first because it changes the expected result of the key boundary tests.
