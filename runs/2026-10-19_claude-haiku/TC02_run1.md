# TC02 Run 1: Test cases for checkout discount code

## Requirement

At checkout a customer can enter one discount code. A valid code gives 10% off orders of 50 EUR or more. Codes expire at 23:59 on their end date. Codes are not case-sensitive.

## Assumptions used for expected results

These are not in the requirement. Each test that depends on one is marked with the assumption ID. The open questions below must be answered before these tests are final.

- A1: "50 EUR or more" is inclusive (50.00 qualifies, 49.99 does not), as the wording "or more" implies.
- A2: The 50 EUR threshold is checked against the order subtotal before the discount is applied.
- A3: The 10% is applied to the whole order subtotal.
- A4: Expiry is evaluated in the store's local time zone, and the code is valid through 23:59:59 on its end date.
- A5: Only one code applies per checkout. A second code is rejected and the first code stays in place.
- A6: Codes are matched after trimming surrounding whitespace.

## Test cases

### Positive

**TC02-01: Valid code on an order of exactly 50.00 EUR (lower boundary)**
- Preconditions: Code SUMMER10 exists, is active, and ends in the future. Cart subtotal is 50.00 EUR.
- Steps:
  1. Go to checkout.
  2. Enter `SUMMER10` in the discount field.
  3. Click Apply.
- Expected result: Code is accepted. Discount line shows -5.00 EUR. Total is 45.00 EUR (plus shipping and tax if applicable, per A2).

**TC02-02: Valid code on a larger order**
- Preconditions: Code SUMMER10 is active. Cart subtotal is 100.00 EUR.
- Steps:
  1. Enter `SUMMER10` and click Apply.
- Expected result: Discount is -10.00 EUR. Total is 90.00 EUR.

**TC02-03: Valid code on an order of 50.01 EUR (just above threshold)**
- Preconditions: Code SUMMER10 is active. Cart subtotal is 50.01 EUR.
- Steps:
  1. Enter `SUMMER10` and click Apply.
- Expected result: Code is accepted. Discount is 5.00 EUR (5.001 rounded to the cent, see OQ-6). Total is 45.01 EUR.

**TC02-04: Code is case-insensitive, all lower case**
- Preconditions: Code SUMMER10 is active. Cart subtotal is 60.00 EUR.
- Steps:
  1. Enter `summer10` and click Apply.
- Expected result: Code is accepted. Discount is -6.00 EUR. Total is 54.00 EUR.

**TC02-05: Code is case-insensitive, mixed case**
- Preconditions: Same as TC02-04.
- Steps:
  1. Enter `SuMmEr10` and click Apply.
- Expected result: Same as TC02-04.

**TC02-06: Code accepted on its end date before 23:59**
- Preconditions: Code SUMMER10 ends on 2026-10-19. Cart subtotal is 60.00 EUR. System time is 2026-10-19 23:58 in the store's time zone (A4).
- Steps:
  1. Enter `SUMMER10` and click Apply.
- Expected result: Code is accepted. Discount is -6.00 EUR.

**TC02-07: Code accepted at 23:59:59 on its end date (A4)**
- Preconditions: Code SUMMER10 ends on 2026-10-19. Cart subtotal is 60.00 EUR. System time is 2026-10-19 23:59:59 in the store's time zone.
- Steps:
  1. Enter `SUMMER10` and click Apply.
- Expected result: Code is accepted (depends on OQ-2). If the requirement is read as "expires at 23:59:00", expect rejection instead.

### Negative

**TC02-08: Code rejected at 00:00 on the day after its end date**
- Preconditions: Code SUMMER10 ends on 2026-10-19. Cart subtotal is 60.00 EUR. System time is 2026-10-20 00:00 in the store's time zone.
- Steps:
  1. Enter `SUMMER10` and click Apply.
- Expected result: Code is rejected with an "expired" (or equivalent) message. No discount is applied. Total is 60.00 EUR.

**TC02-09: Expired code entered in a different letter case**
- Preconditions: Code SUMMER10 ended on 2026-10-01. Cart subtotal is 60.00 EUR.
- Steps:
  1. Enter `summer10` and click Apply.
- Expected result: Rejected as expired. No discount. Case-insensitivity does not change the expiry check.

**TC02-10: Code rejected on an order of 49.99 EUR (upper boundary below threshold)**
- Preconditions: Code SUMMER10 is active. Cart subtotal is 49.99 EUR.
- Steps:
  1. Enter `SUMMER10` and click Apply.
- Expected result: Rejected with a message that the order minimum is 50 EUR (wording to be confirmed). No discount. Total is 49.99 EUR.

**TC02-11: Code rejected on an empty cart (0.00 EUR)**
- Preconditions: Code SUMMER10 is active. Cart is empty or subtotal is 0.00 EUR.
- Steps:
  1. Enter `SUMMER10` and click Apply.
- Expected result: Rejected because the minimum is not met. No discount. No negative total is shown.

**TC02-12: Unknown code**
- Preconditions: Cart subtotal is 60.00 EUR. No code `NOSUCHCODE` exists.
- Steps:
  1. Enter `NOSUCHCODE` and click Apply.
- Expected result: Rejected with an "invalid code" message. No discount. Total unchanged.

**TC02-13: Empty input submitted**
- Preconditions: Cart subtotal is 60.00 EUR.
- Steps:
  1. Leave the discount field empty and click Apply.
- Expected result: No request is made, or a validation message is shown. No discount. Total unchanged.

**TC02-14: Whitespace only input**
- Preconditions: Cart subtotal is 60.00 EUR.
- Steps:
  1. Enter three spaces and click Apply.
- Expected result: Treated as empty. Validation message shown. No discount (A6).

**TC02-15: Code with surrounding whitespace**
- Preconditions: Code SUMMER10 is active. Cart subtotal is 60.00 EUR.
- Steps:
  1. Enter ` SUMMER10 ` (leading and trailing spaces) and click Apply.
- Expected result: Accepted after trimming (A6). Discount is -6.00 EUR. If the system does not trim, expect rejection; this is covered by OQ-5.

**TC02-16: Second code while one is already applied**
- Preconditions: Code SUMMER10 is applied to a cart of 60.00 EUR. Code SAVE5 is active and valid for this order.
- Steps:
  1. Enter `SAVE5` and click Apply.
- Expected result (A5): Second code is rejected with a message that only one code can be used. SUMMER10 remains applied. Discount is still -6.00 EUR.

**TC02-17: Attempt to apply the same code twice**
- Preconditions: Code SUMMER10 is applied to a cart of 60.00 EUR.
- Steps:
  1. Enter `SUMMER10` again and click Apply.
- Expected result: No double discount. Discount is still -6.00 EUR, not -12.00 EUR. The user sees either a confirmation or an "already applied" message.

**TC02-18: Discount no longer valid after cart is reduced below 50 EUR**
- Preconditions: Code SUMMER10 is applied to a cart of 60.00 EUR (discount -6.00 EUR). User removes an item so the subtotal becomes 45.00 EUR.
- Steps:
  1. Observe the checkout summary after the item is removed.
- Expected result (assumed behaviour, see OQ-4): The discount is removed or the user is told the code no longer qualifies. The total is 45.00 EUR, not 40.50 EUR.

**TC02-19: Code valid, but discounted total falls below 50 EUR**
- Preconditions: Code SUMMER10 is active. Cart subtotal is 55.00 EUR.
- Steps:
  1. Enter `SUMMER10` and click Apply.
- Expected result: Under A2, the code is accepted and the discount is -5.50 EUR, giving a total of 49.50 EUR. This test exists because the requirement does not say whether the threshold applies before or after the discount (OQ-3). Mark as blocked until OQ-3 is answered.

**TC02-20: Expired code with a qualifying order**
- Preconditions: Code SUMMER10 ended on 2026-09-30. Cart subtotal is 200.00 EUR.
- Steps:
  1. Enter `SUMMER10` and click Apply.
- Expected result: Rejected as expired. No discount. Qualifying order size does not override expiry.

### Boundary and rounding

**TC02-21: Rounding of 10% on an odd-cent subtotal**
- Preconditions: Code SUMMER10 is active. Cart subtotal is 75.05 EUR.
- Steps:
  1. Enter `SUMMER10` and click Apply.
- Expected result: Discount is 7.51 or 7.50 EUR depending on the rounding rule (OQ-6). The total must match the discount line exactly, with no cent lost or added.

**TC02-22: Discount applies only to the subtotal, not shipping (A3)**
- Preconditions: Code SUMMER10 is active. Subtotal is 60.00 EUR. Shipping is 4.90 EUR.
- Steps:
  1. Enter `SUMMER10` and click Apply.
- Expected result (assumed, see OQ-1): Discount is -6.00 EUR. Shipping remains 4.90 EUR. Total is 58.90 EUR.

## Open questions

These points are not defined in the requirement. Each one must be answered before the tests above are final. I did not invent the answers.

- **OQ-1: Scope of the discount.** Does 10% apply to the whole order, to subtotal only, or to eligible items only? Are shipping, tax, fees and gift cards included or excluded?
- **OQ-2: Exact expiry moment.** Is the code valid through 23:59:59 on the end date, or does it stop at 23:59:00? Is the end date itself included?
- **OQ-3: Threshold basis.** Is the 50 EUR minimum checked on the subtotal before the discount, or on the amount after the discount? Is it checked before or after shipping and tax?
- **OQ-4: Threshold after cart changes.** If a discount is applied and the cart later drops below 50 EUR, is the discount removed silently, removed with a message, or kept?
- **OQ-5: Input handling.** Are leading and trailing spaces trimmed? Are internal spaces or special characters allowed in codes? Is the code length limited?
- **OQ-6: Rounding.** How is the 10% rounded to the cent (half up, half even, or down)?
- **OQ-7: Time zone.** Which time zone defines 23:59: the store, the customer's browser, or UTC? What happens to a customer checking out near midnight across zones?
- **OQ-8: Multiple codes.** Is it certain that a second code is always rejected? Does a new code replace the old one, or is the old one kept? Can a customer remove an applied code?
- **OQ-9: Start date and usage limits.** Does a code have a start date? Is there a maximum number of uses, or a per-customer limit? The requirement only defines an end date.
- **OQ-10: Error messages.** What are the exact texts for invalid, expired and below-minimum codes? Is the error shown inline on the discount field?
- **OQ-11: Currency.** Does the 50 EUR threshold apply only to EUR orders? What happens if a customer pays in another currency?
- **OQ-12: Stacking with other promotions.** Can this code be combined with product-level or automatic discounts? If not, which one wins?
- **OQ-13: Minimum value of 50 EUR.** Is the rule "50 EUR or more" measured in gross (with tax) or net amounts?
