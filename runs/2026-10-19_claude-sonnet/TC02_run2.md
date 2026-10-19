# Test cases: checkout discount code

Requirement: At checkout a customer can enter one discount code. A valid code gives 10% off orders of 50 EUR or more. Codes expire at 23:59 on their end date. Codes are not case-sensitive.

Assumptions used for test data (not invented behaviour, only fixtures): code SAVE10 exists, end date D, 10% discount. "Order value" is treated as the cart amount the threshold applies to; see open questions OQ1 and OQ2 for what exactly is compared.

## Positive

**TC-01: Valid code on order well above threshold**
- Preconditions: SAVE10 is valid, end date in the future. Cart total 80.00 EUR.
- Steps: 1. Go to checkout. 2. Enter `SAVE10`. 3. Apply.
- Expected: Code accepted. 10% discount (8.00 EUR) applied. Total shows 72.00 EUR.

**TC-02: Order exactly at the threshold (boundary, inclusive)**
- Preconditions: As TC-01. Cart total exactly 50.00 EUR.
- Steps: Enter `SAVE10`, apply.
- Expected: Code accepted. Discount 5.00 EUR. Total 45.00 EUR ("50 EUR or more" is inclusive).

**TC-03: Lowercase code**
- Preconditions: As TC-01, cart 60.00 EUR.
- Steps: Enter `save10`, apply.
- Expected: Accepted, 6.00 EUR discount.

**TC-04: Mixed case code**
- Preconditions: As TC-03.
- Steps: Enter `SaVe10`, apply.
- Expected: Accepted, same discount as TC-03.

**TC-05: Code used on its end date before 23:59**
- Preconditions: SAVE10 end date is today. Time set to 14:00.
- Steps: Cart 50+ EUR, enter code, apply.
- Expected: Accepted, 10% off.

**TC-06: Code used at the last valid minute (boundary)**
- Preconditions: SAVE10 end date is today. System time 23:59:00.
- Steps: Cart 50+ EUR, enter code, apply.
- Expected: Accepted, 10% off.

**TC-07: Code with multiple items reaching the threshold**
- Preconditions: Cart of 3 items totalling 50.00 EUR.
- Steps: Enter `SAVE10`, apply.
- Expected: Accepted, 5.00 EUR off.

## Negative

**TC-08: Order just below threshold (boundary)**
- Preconditions: SAVE10 valid. Cart 49.99 EUR.
- Steps: Enter `SAVE10`, apply.
- Expected: Code rejected, clear message that the minimum order is 50 EUR. No discount, total 49.99 EUR.

**TC-09: Order far below threshold**
- Preconditions: Cart 10.00 EUR.
- Steps: Enter `SAVE10`, apply.
- Expected: Rejected as in TC-08.

**TC-10: Expired code (day after end date)**
- Preconditions: SAVE10 end date was yesterday. Cart 80 EUR.
- Steps: Enter code, apply.
- Expected: Rejected with an "expired" message. No discount.

**TC-11: Code just after expiry (boundary)**
- Preconditions: End date is today. System time 00:00 the next day (or 23:59:59 / 24:00, see OQ4).
- Steps: Enter code, apply.
- Expected: Rejected as expired. Behaviour for 23:59:01 to 23:59:59 is open (OQ4).

**TC-12: Non-existent code**
- Steps: Enter `NOTACODE`, apply.
- Expected: Rejected with an "invalid code" message. Cart unchanged.

**TC-13: Empty code field**
- Steps: Leave the field empty, click Apply.
- Expected: No discount, validation message or no action. No error or crash.

**TC-14: Whitespace-only input**
- Steps: Enter spaces only, apply.
- Expected: Treated as empty or invalid. No discount.

**TC-15: Second code when one is already applied**
- Preconditions: SAVE10 applied to a 80 EUR cart. A second valid code OTHER5 exists.
- Steps: Try to enter `OTHER5`.
- Expected: Second code not stacked. Either the input is blocked or the second code is refused (or replaces the first, see OQ5). Total never receives both discounts.

**TC-16: Same code entered twice**
- Preconditions: SAVE10 applied.
- Steps: Enter `SAVE10` again, apply.
- Expected: Discount applied only once (still 10%).

**TC-17: Cart drops below threshold after code applied**
- Preconditions: Cart 55 EUR, SAVE10 applied (5.50 EUR off).
- Steps: Remove items so the cart is 40 EUR.
- Expected: Discount removed or code invalidated with message; no 10% on a sub-50 order (see OQ2).

**TC-18: Code with special characters / injection**
- Steps: Enter `' OR 1=1 --` and `<script>alert(1)</script>`.
- Expected: Rejected as invalid, input safely handled, no error page, no script execution.

**TC-19: Code with leading/trailing spaces**
- Steps: Enter ` SAVE10 `.
- Expected: Undefined by requirement (OQ6). Must not error. Record actual behaviour.

**TC-20: Very long input**
- Steps: Enter a 1000-character string.
- Expected: Rejected or truncated gracefully, no server error.

## Boundary and rounding

**TC-21: Rounding of discount**
- Preconditions: Cart 50.05 EUR.
- Steps: Apply `SAVE10`.
- Expected: Discount 5.005 rounded per defined rule (OQ3). Total has exactly two decimals and is consistent between cart, checkout and invoice.

**TC-22: Cart just above threshold**
- Preconditions: Cart 50.01 EUR.
- Expected: Accepted, discount 5.00 EUR (rounded, OQ3).

**TC-23: Time zone boundary**
- Preconditions: End date today. Customer and server in different time zones, local time before 23:59 but server time after (or the reverse).
- Expected: Expiry evaluated in the single defined time zone (OQ4).

**TC-24: Case-insensitivity with locale-specific characters**
- Preconditions: Code `ÄBC10` or `ISTANBUL10` exists.
- Steps: Enter lowercase variants, including Turkish dotless i.
- Expected: Matches case-insensitively without locale errors.

## Persistence and flow

**TC-25: Discount survives page refresh and navigation**
- Steps: Apply code, refresh, go back and forward.
- Expected: Discount and code state stay consistent.

**TC-26: Code expires between apply and payment**
- Preconditions: Code applied at 23:58, payment submitted at 00:01.
- Expected: Behaviour defined by OQ7; at minimum the charged total matches what the customer was shown, or the customer is told of the change.

**TC-27: Order total and confirmation**
- Steps: Complete an order with SAVE10.
- Expected: Confirmation, email and order record show the discounted total and the code used.

## Open questions (not assumed above)

1. OQ1: Is the 50 EUR threshold checked on the subtotal before discount, or including shipping, taxes or other fees? Are items already on sale included?
2. OQ2: What happens if the cart falls below 50 EUR after the code is applied (remove code, block checkout, keep the discount)?
3. OQ3: Rounding rule for the 10% discount (to the cent, half-up, per line or on the total)?
4. OQ4: Which time zone is used for 23:59? Is 23:59:30 still valid (is the limit 23:59:00, 23:59:59 or end of day)? Is the end date inclusive?
5. OQ5: When a second code is entered, is it refused or does it replace the first? What message is shown?
6. OQ6: Are leading/trailing spaces trimmed? Are there allowed characters or max length for codes?
7. OQ7: Is validity checked at apply time only, or again at payment submission?
8. OQ8: Is there a start date, usage limit per code or per customer, or restriction to certain products, customers or currencies? Are guest customers allowed?
9. OQ9: Does "10% off" apply to the whole order or only eligible items? Does it also reduce shipping?
10. OQ10: What are the exact error messages for expired, invalid and below-minimum cases?
11. OQ11: For orders in a currency other than EUR, how is the 50 EUR threshold converted?
12. OQ12: Are refunds or partial returns of a discounted order handled proportionally?
