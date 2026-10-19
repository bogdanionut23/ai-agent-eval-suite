# TC02: Checkout discount code - test cases

## Requirement (as given)

At checkout a customer can enter one discount code. A valid code gives 10% off orders of 50 EUR or more. Codes expire at 23:59 on their end date. Codes are not case-sensitive.

## Assumptions used for the test cases

Only the rules stated in the requirement are tested. Where the requirement is silent, the expected result is marked "TBD (see Q-n)" and the matching open question is listed below. No behaviour is invented.

- Discount = 10% of the order value, the threshold is 50.00 EUR inclusive (50.00 qualifies, 49.99 does not).
- A code with end date D is valid up to and including 23:59:59 on D, and invalid from 00:00:00 on D+1.
- "Case-insensitive" means `save10`, `SAVE10` and `Save10` are the same code.

## Test cases

### Positive

**TC02-01 Valid code on an order of exactly 50.00 EUR (lower bound)**
- Preconditions: Code `SAVE10` exists, is active, end date is in the future. Cart subtotal is 50.00 EUR.
- Steps: 1. Open checkout. 2. Enter `SAVE10`. 3. Apply.
- Expected: Code is accepted. Discount line shows -5.00 EUR. Order total is 45.00 EUR (before shipping and tax, see Q-3).

**TC02-02 Valid code on a larger order**
- Preconditions: As TC02-01. Cart subtotal is 100.00 EUR.
- Steps: 1. Open checkout. 2. Enter `SAVE10`. 3. Apply.
- Expected: Code accepted. Discount is -10.00 EUR. Total is 90.00 EUR (before shipping and tax, see Q-3).

**TC02-03 Lowercase entry of an uppercase code**
- Preconditions: Code `SAVE10` exists and is valid. Cart subtotal is 60.00 EUR.
- Steps: 1. Enter `save10`. 2. Apply.
- Expected: Code accepted with the same 10% discount as `SAVE10` (-6.00 EUR).

**TC02-04 Mixed-case entry**
- Preconditions: As TC02-03.
- Steps: 1. Enter `SaVe10`. 2. Apply.
- Expected: Code accepted, discount -6.00 EUR.

**TC02-05 Code applied on the end date before 23:59**
- Preconditions: Code `SAVE10` has end date 2026-10-19. Current time is 2026-10-19 23:58:00. Cart subtotal is 80.00 EUR.
- Steps: 1. Enter `SAVE10`. 2. Apply.
- Expected: Code accepted, discount -8.00 EUR.

**TC02-06 Code is still accepted at 23:59:59 on the end date (time boundary, inclusive)**
- Preconditions: As TC02-05, time is 2026-10-19 23:59:59. Cart subtotal is 80.00 EUR.
- Steps: 1. Enter `SAVE10`. 2. Apply.
- Expected: Code accepted, discount -8.00 EUR. (Depends on Q-4 for timezone; the test assumes the store timezone.)

**TC02-07 Order is placed after the code was applied, still above threshold**
- Preconditions: `SAVE10` applied to a 70.00 EUR cart. Discount -7.00 EUR shown.
- Steps: 1. Complete payment. 2. Open the order confirmation.
- Expected: Order stores discount -7.00 EUR and total 63.00 EUR (before shipping and tax, see Q-3).

### Negative

**TC02-08 Order just below the threshold (49.99 EUR)**
- Preconditions: `SAVE10` is valid. Cart subtotal is 49.99 EUR.
- Steps: 1. Enter `SAVE10`. 2. Apply.
- Expected: Code is rejected with a message stating the minimum order value (exact wording TBD, see Q-9). No discount is applied. Total stays 49.99 EUR.

**TC02-09 Code entered on a 49.99 EUR order, then the cart is raised to 50.00 EUR**
- Preconditions: Code rejected at 49.99 EUR (TC02-08).
- Steps: 1. Add an item so the subtotal becomes 50.00 EUR. 2. Re-enter `SAVE10`. 3. Apply.
- Expected: Code accepted, -5.00 EUR. The rejection does not block a later valid attempt.

**TC02-10 Expired code, one minute after end of end date**
- Preconditions: Code `SAVE10` has end date 2026-10-18. Current time is 2026-10-19 00:00:00. Cart subtotal is 80.00 EUR.
- Steps: 1. Enter `SAVE10`. 2. Apply.
- Expected: Code rejected with an "expired" message. No discount applied.

**TC02-11 Expired code, one second after the expiry time**
- Preconditions: Code end date 2026-10-19. Current time is 2026-10-20 00:00:00 (the day after). Cart subtotal is 80.00 EUR.
- Steps: 1. Enter `SAVE10`. 2. Apply.
- Expected: Code rejected as expired. No discount.

**TC02-12 Code that was valid at the start of checkout expires during checkout**
- Preconditions: Code end date 2026-10-19. Time is 23:59:30 when the customer enters the code. Customer is on payment page at 00:00:10 on 2026-10-20.
- Steps: 1. Enter `SAVE10` at 23:59:30 and apply. 2. Wait until 00:00:10. 3. Submit payment.
- Expected: TBD (see Q-5). The test records whether the order is accepted with the discount or rejected.

**TC02-13 Unknown code**
- Preconditions: No code `NOTACODE` exists. Cart subtotal is 80.00 EUR.
- Steps: 1. Enter `NOTACODE`. 2. Apply.
- Expected: Code rejected with an "invalid code" message. No discount. The total does not change.

**TC02-14 Empty input**
- Preconditions: Discount field is empty. Cart subtotal is 80.00 EUR.
- Steps: 1. Click apply without entering a code.
- Expected: No discount is applied. The field shows a validation message (wording TBD). The checkout is not blocked.

**TC02-15 Only whitespace entered**
- Preconditions: As TC02-14.
- Steps: 1. Enter three spaces. 2. Apply.
- Expected: Treated as empty input (TC02-14 result). Whether leading or trailing spaces are trimmed for valid codes is open (see Q-7).

**TC02-16 Second code entered when one is already applied**
- Preconditions: `SAVE10` applied to an 80.00 EUR cart. Second code `OTHER20` exists and is valid.
- Steps: 1. Enter `OTHER20`. 2. Apply.
- Expected: The requirement allows one code per checkout, so `OTHER20` must not be stacked on top of `SAVE10`. Whether it replaces `SAVE10` or is rejected is TBD (see Q-6). In all cases the total must not reflect both discounts.

**TC02-17 Code with special characters or markup**
- Preconditions: Cart subtotal is 80.00 EUR.
- Steps: 1. Enter `SAVE10' OR 1=1 --` or `<script>alert(1)</script>`. 2. Apply.
- Expected: Code rejected as invalid. No error page, no script execution, no discount.

**TC02-18 Code removed after applying, cart falls below threshold**
- Preconditions: `SAVE10` applied to a 55.00 EUR cart. Discount -5.50 EUR.
- Steps: 1. Remove an item so the subtotal becomes 45.00 EUR. 2. Review the order summary.
- Expected: TBD (see Q-8). The test records whether the discount is removed automatically, kept, or the customer is warned.

### Boundary

**TC02-19 Threshold at 50.00 vs 49.99 (pair)**
- Run TC02-01 and TC02-08 together as a boundary pair. Both results must be consistent with the same threshold rule: 50.00 accepted, 49.99 rejected.

**TC02-20 Threshold at 50.01 (just above)**
- Preconditions: `SAVE10` valid. Cart subtotal is 50.01 EUR.
- Steps: 1. Enter `SAVE10`. 2. Apply.
- Expected: Code accepted. Discount is 10% of 50.01 = 5.001 EUR, rounded to 5.00 EUR (rounding rule TBD, see Q-2).

**TC02-21 Rounding of a discount that is not a whole cent**
- Preconditions: `SAVE10` valid. Cart subtotal is 55.55 EUR.
- Steps: 1. Enter `SAVE10`. 2. Apply.
- Expected: Discount is 5.555 EUR before rounding. Result is 5.55 or 5.56 EUR per the rounding rule (TBD, see Q-2). The total is consistent with the displayed discount to the cent.

**TC02-22 Expiry boundary at 23:59:59 vs 00:00:00 (pair)**
- Run TC02-06 (23:59:59 accepted) and TC02-11 (00:00:00 next day rejected) together. They must share one consistent cut-off.

**TC02-23 Case-insensitive boundary: non-ASCII letters**
- Preconditions: A code containing an accented or non-ASCII letter exists (only if the code format allows it, see Q-10).
- Steps: 1. Enter the code with a different letter case. 2. Apply.
- Expected: TBD (see Q-10). Case folding for non-ASCII characters is not specified.

## Open questions

1. **Threshold base.** Is the 50 EUR threshold checked on the subtotal before discount, or after? Does it include shipping and tax?
2. **Rounding.** How is the 10% discount rounded to cents (round half up, round half even, or truncate)?
3. **Discount base.** Is the 10% applied to the subtotal only, or also to shipping, tax, and fees? Is the 50 EUR threshold measured on the same base?
4. **Time zone.** Is "23:59" in the store's time zone, the customer's local time zone, or UTC? What is the store's time zone?
5. **Expiry during checkout.** If a code expires after it is applied but before the order is placed, is the order accepted with the discount or rejected?
6. **Second code.** When a code is already applied and a second valid code is entered, does the new code replace the old one or is it rejected? Is the rule "one code per checkout" meant to block stacking only, or to replace too?
7. **Input normalisation.** Are leading and trailing spaces trimmed before validation? Are internal spaces or hyphens allowed or removed?
8. **Threshold after apply.** If the cart drops below 50 EUR after a code is applied, is the discount removed automatically, kept, or does the customer see a warning?
9. **Messages.** What are the exact customer-facing messages for below-threshold, expired, unknown, and empty input? Are they translated?
10. **Case folding.** Which characters are allowed in codes? Is case-insensitivity defined for non-ASCII letters (for example, Turkish dotted and dotless i, or German sharp s)?
11. **Start date.** Does a code have a start date? If so, is it inclusive, and what time zone applies? The requirement only mentions the end date.
12. **Scope.** Does the 10% apply to every product, or are some products or categories excluded? Does it stack with other promotions, such as sale prices?
13. **Usage limits.** Can a code be used once per customer, a fixed number of times in total, or unlimited? The requirement is silent.
14. **Currency.** Is the 50 EUR threshold applied only to EUR orders? What happens for orders in other currencies or with a converted price?
15. **Empty cart.** Can a code be entered when the cart is empty or the total is 0.00 EUR?
16. **Code format.** Maximum length and allowed character set for codes are not stated.
