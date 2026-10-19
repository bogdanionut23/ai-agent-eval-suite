# TC02 Run 3: Test cases for checkout discount code

## Requirement (as given)

At checkout a customer can enter one discount code. A valid code gives 10% off orders of 50 EUR or more. Codes expire at 23:59 on their end date. Codes are not case-sensitive.

## Common preconditions (used by the test cases unless stated otherwise)

- Customer is on the checkout page with a cart in EUR and a discount code field visible.
- Code `VALID10` exists, is active, and has an end date in the future (for example 2026-12-31).
- Code `EXPIRED10` exists and its end date was 2026-10-18.
- Code `NOTLIVE`, `TODAYEND10`, and `SAVE10` are described in the individual tests where used.
- Test dates are set with a controlled clock, not the real system time.

## Test cases

### Positive

**TC02-01: Valid code gives 10% off an order above the threshold**
- Preconditions: Common preconditions. Cart subtotal is 100.00 EUR.
- Steps:
  1. Enter `VALID10` in the discount code field.
  2. Apply the code.
- Expected result: The discount line shows -10.00 EUR. The order total is reduced by 10.00 EUR (to 90.00 EUR, before shipping and tax; see OQ-03).

**TC02-02: Order of exactly 50.00 EUR qualifies**
- Preconditions: Common preconditions. Cart subtotal is exactly 50.00 EUR.
- Steps:
  1. Enter `VALID10`.
  2. Apply the code.
- Expected result: Code is accepted. Discount is -5.00 EUR and the order total is 45.00 EUR (see OQ-01 for the base the threshold is checked against).

**TC02-03: Large order receives 10% off**
- Preconditions: Common preconditions. Cart subtotal is 1,000.00 EUR.
- Steps:
  1. Enter `VALID10` and apply it.
- Expected result: Discount is -100.00 EUR and the order total is 900.00 EUR.

**TC02-04: Code is accepted on the day before its end date**
- Preconditions: Clock set to 2026-12-30 12:00. Code `VALID10` has end date 2026-12-31.
- Steps:
  1. Enter `VALID10` and apply it on an order of 100.00 EUR.
- Expected result: Code is accepted and 10.00 EUR discount is applied.

**TC02-05: Code is accepted just before expiry on its end date**
- Preconditions: Clock set to 2026-12-31 23:58:59. Code `VALID10` has end date 2026-12-31.
- Steps:
  1. Enter `VALID10` and apply it on an order of 100.00 EUR.
- Expected result: Code is accepted and 10.00 EUR discount is applied.

**TC02-06: Lowercase entry is accepted**
- Preconditions: Common preconditions. Order of 100.00 EUR.
- Steps:
  1. Enter `valid10` and apply it.
- Expected result: Code is accepted and the same 10.00 EUR discount is applied as for `VALID10`.

**TC02-07: Mixed-case entry is accepted**
- Preconditions: Common preconditions. Order of 100.00 EUR.
- Steps:
  1. Enter `VaLiD10` and apply it.
- Expected result: Code is accepted and 10.00 EUR discount is applied.

**TC02-08: Case-insensitive match when the stored code is mixed case**
- Preconditions: Code `Save10` exists, active, with a future end date. Order of 100.00 EUR.
- Steps:
  1. Enter `SAVE10` and apply it.
  2. Separately, remove the code, then enter `save10` and apply it.
- Expected result: Both entries are accepted and each gives a 10.00 EUR discount.

**TC02-09: Discount is applied once when the same code is entered twice**
- Preconditions: Common preconditions. Order of 100.00 EUR.
- Steps:
  1. Enter `VALID10` and apply it.
  2. Enter `VALID10` again and apply it.
- Expected result: Total discount is 10.00 EUR, not 20.00 EUR. The second entry does not stack.

**TC02-10: Removing an applied code restores the full price**
- Preconditions: Common preconditions. Order of 100.00 EUR with `VALID10` applied (total 90.00 EUR).
- Steps:
  1. Remove the discount code.
- Expected result: Discount line disappears and the order total returns to 100.00 EUR.

### Negative

**TC02-11: Unknown code is rejected**
- Preconditions: Common preconditions. Order of 100.00 EUR.
- Steps:
  1. Enter `NOTLIVE` (a code that does not exist) and apply it.
- Expected result: Code is rejected. No discount is applied and the total stays at 100.00 EUR. An error message is shown (wording: see OQ-09).

**TC02-12: Code with a single-character typo is rejected**
- Preconditions: Common preconditions. Order of 100.00 EUR.
- Steps:
  1. Enter `VALID1O` (letter O instead of zero) and apply it.
- Expected result: Code is rejected. No discount is applied.

**TC02-13: Empty code submission is rejected**
- Preconditions: Common preconditions. Order of 100.00 EUR.
- Steps:
  1. Leave the code field empty and apply it.
- Expected result: No discount is applied. The customer gets a validation message and the total stays at 100.00 EUR.

**TC02-14: Code with an internal space is rejected**
- Preconditions: Common preconditions. Order of 100.00 EUR.
- Steps:
  1. Enter `VALID 10` and apply it.
- Expected result: Code is rejected and no discount is applied. (Whether spaces are trimmed or treated as part of the code is OQ-07. This test assumes an internal space does not match `VALID10`.)

**TC02-15: Expired code is rejected**
- Preconditions: Clock set to 2026-10-19 10:00. Code `EXPIRED10` has end date 2026-10-18. Order of 100.00 EUR.
- Steps:
  1. Enter `EXPIRED10` and apply it.
- Expected result: Code is rejected with an expired message. No discount is applied.

**TC02-16: Expired code is rejected regardless of case**
- Preconditions: Same as TC02-15.
- Steps:
  1. Enter `expired10` and apply it.
- Expected result: Code is rejected as expired. No discount is applied. Case does not change the result.

**TC02-17: Valid code on an order below 50 EUR is rejected**
- Preconditions: Common preconditions. Cart subtotal is 40.00 EUR.
- Steps:
  1. Enter `VALID10` and apply it.
- Expected result: Code is not applied. No discount is shown and the total stays at 40.00 EUR. A message says the order must be at least 50 EUR (wording: OQ-09).

**TC02-18: Second code cannot be added while one is applied**
- Preconditions: Common preconditions. Order of 100.00 EUR with `VALID10` applied.
- Steps:
  1. Enter a second valid code `SAVE10` and apply it.
- Expected result: Only one discount is in effect. Total discount is not 20.00 EUR. The second code is rejected, or it replaces `VALID10`, as set by OQ-05. In either case, the total never shows more than one code's discount.

**TC02-19: Input of excessive length is handled safely**
- Preconditions: Common preconditions. Order of 100.00 EUR.
- Steps:
  1. Enter a string of 1,000 characters in the code field and apply it.
- Expected result: Code is rejected cleanly. No error page, server error, or discount. Maximum input length is OQ-10.

**TC02-20: Code with special characters is rejected**
- Preconditions: Common preconditions. Order of 100.00 EUR.
- Steps:
  1. Enter `VALID10;DROP` (or `VALID10<script>`) and apply it.
- Expected result: Code is rejected. No discount is applied. No script runs and no error page appears.

### Boundary

**TC02-21: Subtotal just below threshold, 49.99 EUR**
- Preconditions: Common preconditions. Cart subtotal is 49.99 EUR.
- Steps:
  1. Enter `VALID10` and apply it.
- Expected result: Code is rejected. No discount is applied (threshold is 50.00 EUR).

**TC02-22: Subtotal just above threshold, 50.01 EUR**
- Preconditions: Common preconditions. Cart subtotal is 50.01 EUR.
- Steps:
  1. Enter `VALID10` and apply it.
- Expected result: Code is accepted. Discount is -5.00 EUR (5.001 rounded to cents, see OQ-04) and total is 45.01 EUR.

**TC02-23: Expiry at 23:59:00 exactly on the end date**
- Preconditions: Clock set to 2026-12-31 23:59:00. Code `VALID10` has end date 2026-12-31. Order of 100.00 EUR.
- Steps:
  1. Enter `VALID10` and apply it.
- Expected result: Depends on OQ-02. If 23:59 is the expiry instant, the code is rejected as expired. If expiry is end of the minute, the code is accepted. Record the actual result and check it against the decision in OQ-02.

**TC02-24: Expiry just after 23:59 on the end date**
- Preconditions: Clock set to 2026-12-31 23:59:30. Code `VALID10` has end date 2026-12-31. Order of 100.00 EUR.
- Steps:
  1. Enter `VALID10` and apply it.
- Expected result: Code is rejected as expired under the reading that the code expires at 23:59 (see OQ-02). No discount is applied.

**TC02-25: Code is rejected on the day after its end date**
- Preconditions: Clock set to 2027-01-01 00:00:00. Code `VALID10` has end date 2026-12-31. Order of 100.00 EUR.
- Steps:
  1. Enter `VALID10` and apply it.
- Expected result: Code is rejected as expired. No discount is applied.

**TC02-26: Cart is reduced from above to below the threshold after the code is applied**
- Preconditions: Common preconditions. Cart is 60.00 EUR with `VALID10` applied (total 54.00 EUR).
- Steps:
  1. Remove items so the subtotal becomes 45.00 EUR.
- Expected result: The discount no longer applies. The discount line is removed (or the code is flagged as not applicable), and the total is 45.00 EUR. Exact behaviour depends on OQ-06.

**TC02-27: Cart is raised from below to above the threshold after the code is entered**
- Preconditions: Common preconditions. Cart is 40.00 EUR and `VALID10` was rejected.
- Steps:
  1. Add items so the subtotal becomes 60.00 EUR.
  2. Check the checkout summary before paying.
- Expected result: Depends on OQ-06. If the code is retained, the discount applies (-6.00 EUR, total 54.00 EUR). If the code was dropped, the customer must enter it again. Record which behaviour occurs and confirm it matches the decision.

**TC02-28: Rounding of the discount to the cent**
- Preconditions: Common preconditions. Cart subtotal is 55.55 EUR.
- Steps:
  1. Enter `VALID10` and apply it.
- Expected result: The discount is 5.555 EUR before rounding. The rounding rule (round half up, round half even, or round down) is not defined. See OQ-04. The total must be consistent with the rule once it is decided.

**TC02-29: Timezone of the expiry time (customer and store in different zones)**
- Preconditions: Store time zone and customer time zone differ. Code `VALID10` has end date 2026-12-31. Clock set so that 23:59 in one zone falls on a different date in the other zone.
- Steps:
  1. Enter `VALID10` at the moment the two zones disagree.
- Expected result: Not defined. Expiry is evaluated in the zone decided in OQ-02. Do not create the final expected result until OQ-02 is answered.

## Open questions

These are not tested with an invented expected result. Each test that depends on one says so.

- **OQ-01: Threshold base.** Is "50 EUR or more" checked against the subtotal before or after the discount? Does it include or exclude shipping, taxes, and fees? Does it include items already on promotion?
- **OQ-02: Expiry time and timezone.** Is the expiry 23:59:00 (an instant, so 23:59:00 is already expired) or the end of the 23:59 minute (23:59:59)? Is the time evaluated in the store's timezone, the customer's, or UTC?
- **OQ-03: Discount base.** Does the 10% apply to the item subtotal only, or also to shipping and fees? Is it applied before or after VAT/tax?
- **OQ-04: Rounding.** How is 10% rounded to the cent (for example 5.555 EUR)? Is rounding per line or on the order total?
- **OQ-05: Second code.** When a code is already applied and the customer enters another, is the new code rejected, or does it replace the first?
- **OQ-06: Threshold changes after applying.** If the cart drops below 50 EUR after the code is applied, is the code removed, kept, or kept with a message? If the cart rises above 50 EUR after a rejected code, is it re-checked automatically?
- **OQ-07: Whitespace and formatting.** Are leading or trailing spaces trimmed? Are internal spaces or hyphens significant? Is there a fixed code format or length?
- **OQ-08: Unicode case-insensitivity.** Is "not case-sensitive" meant for ASCII only? Do special cases such as the German sharp s, or characters that case-fold to ASCII letters, match? Is Unicode normalisation applied?
- **OQ-09: Messages.** What are the exact texts for unknown, expired, below-threshold, and duplicate-code errors? Are they translated?
- **OQ-10: Input limits.** What is the maximum length of the code input?
- **OQ-11: Usage limits.** Can a code be used once, or many times? Is there a limit per customer or in total? (None is stated in the requirement.)
- **OQ-12: Eligibility.** Are codes limited to certain products, categories, customers, or channels? Can a code be used with other promotions or with items already discounted?
- **OQ-13: Start date.** Is there a start date for codes, or are codes valid from creation until expiry? (The requirement only states an end date.)
- **OQ-14: Currency.** What happens to a code when the cart is not in EUR? Is the 50 EUR threshold converted, or is the code limited to EUR carts?
- **OQ-15: After-sale changes.** What happens to the discount if the order is later refunded, partly returned, or edited after payment? Is the discount re-applied or reversed?
- **OQ-16: Validation point.** Is the code validated when it is entered, when the order is placed, or both? If the code expires between entry and payment, which applies?
- **OQ-17: Case-insensitivity scope.** Does case-insensitivity also apply to the code stored in the database and in reports, or only at checkout?
