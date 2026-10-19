# Test cases: Orders page date range filter

## Assumptions (taken directly from the requirement)
- Filter has two fields: From and To.
- Both dates are inclusive.
- Max range length is 31 days.
- To must not be earlier than From (To = From is therefore allowed).

"31 days" is read inclusively: From = 1 Jan, To = 31 Jan is 31 days (see Open Questions 1). Test data below assumes one order exists at 00:00 and one at 23:59 on key dates, in a non-leap-year context unless stated.

## Positive

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-01 | Filter by a valid multi-day range | User logged in; orders exist on 2026-03-01 to 2026-03-20 | 1. Open orders page. 2. Set From = 2026-03-05. 3. Set To = 2026-03-10. 4. Apply filter. | Only orders dated 5 to 10 March are listed. No validation error. |
| TC-02 | From date is included | Order exists on 2026-03-05 | Filter From = 2026-03-05, To = 2026-03-10 | The 5 March order is shown. |
| TC-03 | To date is included | Order exists on 2026-03-10, including one at 23:59 | Filter From = 2026-03-05, To = 2026-03-10 | All 10 March orders, including the 23:59 one, are shown. |
| TC-04 | Single-day range (From = To) | Orders exist on 2026-03-07 and neighbouring days | Filter From = To = 2026-03-07 | Only 7 March orders are shown; no error. |
| TC-05 | Range with no matching orders | No orders between 2026-01-01 and 2026-01-10 | Filter that range | Empty result with the normal "no orders" state; no validation error. |
| TC-06 | Range spanning month and year boundary | Orders on 2025-12-30 to 2026-01-02 | Filter From = 2025-12-30, To = 2026-01-02 | Orders from all four days are shown. |
| TC-07 | Change a previously applied range | TC-01 filter applied | Change To to 2026-03-12 and apply | List updates to 5 to 12 March. |
| TC-08 | Clear the filter | A range is applied | Clear/reset the filter | Full unfiltered list returns; fields empty. |

## Boundary

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-09 | Maximum allowed range (31 days inclusive) | Orders exist across January 2026 | From = 2026-01-01, To = 2026-01-31 | Accepted; orders for all 31 days shown. |
| TC-10 | One day over the maximum (32 days inclusive) | As above | From = 2026-01-01, To = 2026-02-01 | Rejected with a clear error about max range; list not filtered (or stays on previous result). Depends on OQ-1. |
| TC-11 | 30-day range (just under limit) | As above | From = 2026-01-01, To = 2026-01-30 | Accepted. |
| TC-12 | To one day before From | None | From = 2026-03-10, To = 2026-03-09 | Rejected with error that To cannot be earlier than From. |
| TC-13 | To equals From | None | From = To = 2026-03-10 | Accepted (same as TC-04). |
| TC-14 | 31-day range across February (non-leap) | None | From = 2026-02-01, To = 2026-03-03 (31 days) | Accepted. |
| TC-15 | 31-day range containing 29 Feb (leap year) | None | From = 2028-02-01, To = 2028-03-02 (31 days) | Accepted; leap day handled correctly. |
| TC-16 | 32-day range containing 29 Feb | None | From = 2028-02-01, To = 2028-03-03 | Rejected (over 31 days). |
| TC-17 | 31-day range over a DST change | User time zone with DST | From = 2026-03-20, To = 2026-04-19 (spans DST switch) | Accepted; day count not distorted by the 23/25-hour day. |
| TC-18 | Earliest/latest supported dates | Minimum and maximum supported dates known (see OQ-5) | Use the earliest valid From with a 31-day range; use the latest valid To | Accepted without error or overflow. |

## Negative

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-19 | Far-too-long range | None | From = 2026-01-01, To = 2026-12-31 | Rejected with max-range error. |
| TC-20 | To earlier than From by a large gap | None | From = 2026-06-01, To = 2026-01-01 | Rejected with To-before-From error. |
| TC-21 | Only From provided | None | From = 2026-03-01, To empty, apply | Behaviour per OQ-2; no crash or unhandled error. |
| TC-22 | Only To provided | None | From empty, To = 2026-03-10, apply | Behaviour per OQ-2; no crash. |
| TC-23 | Both empty | None | Apply with both empty | Behaviour per OQ-2 (likely unfiltered list); no error. |
| TC-24 | Invalid date text | Fields accept typing | Type "abc" in From | Input rejected or validation error; no filter applied. |
| TC-25 | Impossible calendar date | Fields accept typing | From = 2026-02-30 | Validation error; no filter applied. |
| TC-26 | Wrong date format | Fields accept typing | From = 03/15/2026 when the expected format is different | Validation error or correct parse per locale; never a silent wrong date. |
| TC-27 | Error clears after correction | TC-10 error showing | Fix To so range is 31 days and apply | Error disappears; filter applies. |
| TC-28 | Editing From makes the existing range invalid | From = 2026-01-01, To = 2026-01-31 applied | Change From to 2025-12-01 | Validation error shown (range now over 31 days); no stale/incorrect result. |
| TC-29 | Editing To makes the existing range invalid | From = 2026-03-10, To = 2026-03-15 | Change To to 2026-03-05 | To-before-From error shown. |
| TC-30 | Injection / special characters | Fields accept typing | Enter `'; DROP TABLE orders;--` in From | Rejected as invalid date; no server error. |
| TC-31 | Direct request bypassing the UI | API/URL access | Call the orders endpoint (or edit URL parameters) with a 60-day range or To < From | Server-side validation rejects it with an error; UI validation is not the only guard. |
| TC-32 | Date picker restrictions | Date picker used | Open the To picker after choosing From | Behaviour per OQ-6 (dates before From or beyond +30 days disabled, or error on selection). |

## Other

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-33 | Filter combined with pagination | More than one page of orders in range | Apply range, go to page 2 | Page 2 respects the range; counts correct. |
| TC-34 | Filter persists on reload/navigation | Range applied | Refresh page / navigate away and back | Behaviour per OQ-7; consistent and not broken. |
| TC-35 | Order near midnight and time zones | User time zone differs from server; order created 23:30 local on 10 Mar | Filter To = 2026-03-10 | Order is shown according to the defined time zone rule (OQ-4). |
| TC-36 | Future dates | None | From/To in the future | Behaviour per OQ-3; no crash, likely empty result. |
| TC-37 | Keyboard and accessibility | None | Operate fields and error messages by keyboard and screen reader | Fields reachable, errors announced and associated with fields. |

## Open questions (not assumed in the tests above)

1. **How is "31 days" counted?** Inclusive (1 Jan to 31 Jan is 31 days) or as a difference (1 Jan to 1 Feb is 31 days)? This decides whether TC-09/TC-10 boundaries are correct.
2. **Are both dates mandatory?** What happens with only From, only To, or neither? Is there a default range on page load?
3. **Are future dates allowed?** Is there a minimum date (e.g. data retention limit)?
4. **Which date and time zone is compared?** Order creation date, payment date or update date? User local time, shop time or UTC? Are orders at 23:59 / 00:00 on the boundary handled by date or timestamp?
5. **What are the supported date limits and format?** Locale-dependent format? Can users type dates or only use a picker?
6. **How does the date picker behave?** Should it disable invalid dates, or allow them and show an error?
7. **Is the filter state persisted** (URL, session, saved between visits)? Does it combine with other filters and sorting?
8. **When is validation triggered?** On change, on blur, or on Apply? Is the Apply button disabled while invalid?
9. **What are the exact error messages**, and where are they shown?
10. **Is the rule also enforced server-side** (API), and what error code is returned?
11. **Which orders are in scope** (all statuses, including cancelled/archived)? Does the user see only their own orders?
12. **What happens to the previously shown list** when an invalid range is submitted: unchanged or cleared?
