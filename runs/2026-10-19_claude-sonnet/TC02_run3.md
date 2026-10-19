# Test Cases: Checkout Discount Code

Requirement: At checkout a customer can enter one discount code. A valid code gives 10% off orders of 50 EUR or more. Codes expire at 23:59 on their end date. Codes are not case-sensitive.

Assumptions (marked where they matter): "order value" in the 50 EUR threshold is taken to mean the cart subtotal, but this is an open question (see Q1). Test data uses a code `SAVE10` with end date D, and a code `OTHER5` that is also valid. Expected results that depend on an open question are marked [OQ].

## Positive

### TC-01: Valid code on an order well above the threshold
- Preconditions: Code `SAVE10` is active, end date is in the future. Cart subtotal is 100.00 EUR.
- Steps: 1. Go to checkout. 2. Enter `SAVE10`. 3. Apply.
- Expected: Code is accepted. 10.00 EUR discount shown. Order total is 90.00 EUR.

### TC-02: Order exactly at the threshold (50.00 EUR)
- Preconditions: `SAVE10` active. Cart subtotal is exactly 50.00 EUR.
- Steps: 1. Enter `SAVE10`. 2. Apply.
- Expected: Code accepted ("50 EUR or more" includes 50). Discount 5.00 EUR. Total 45.00 EUR.

### TC-03: Lowercase code
- Preconditions: `SAVE10` active. Subtotal 80.00 EUR.
- Steps: 1. Enter `save10`. 2. Apply.
- Expected: Accepted. Discount 8.00 EUR. Total 72.00 EUR.

### TC-04: Mixed-case code
- Preconditions: As TC-03.
- Steps: 1. Enter `SaVe10`. 2. Apply.
- Expected: Accepted, same result as TC-03.

### TC-05: Code used on its end date before 23:59
- Preconditions: `SAVE10` end date is today. System time is 12:00. Subtotal 60.00 EUR.
- Steps: 1. Enter `SAVE10`. 2. Apply.
- Expected: Accepted. Discount 6.00 EUR.

### TC-06: Code starting before expiry and still valid at the start of the end date
- Preconditions: `SAVE10` end date is today. System time is 00:00:00. Subtotal 60.00 EUR.
- Steps: 1. Enter and apply `SAVE10`.
- Expected: Accepted.

### TC-07: Discount rounding
- Preconditions: `SAVE10` active. Subtotal 50.05 EUR.
- Steps: 1. Apply `SAVE10`.
- Expected: Discount is rounded to cents per the agreed rule [OQ Q6]. With round-half-up: 5.01 EUR (5.005), total 45.04 EUR.

### TC-08: Remove code and re-apply
- Preconditions: `SAVE10` applied to a 100.00 EUR order.
- Steps: 1. Remove the code. 2. Check total. 3. Re-enter `SAVE10`.
- Expected: After removal total is 100.00 EUR with no discount. Re-applying gives 90.00 EUR.

## Negative

### TC-09: Order just below the threshold
- Preconditions: `SAVE10` active. Subtotal 49.99 EUR.
- Steps: 1. Enter and apply `SAVE10`.
- Expected: No discount applied. A clear message says the minimum order is 50 EUR [OQ Q3 on message wording]. Total remains 49.99 EUR.

### TC-10: Expired code (end date yesterday)
- Preconditions: `SAVE10` end date is yesterday. Subtotal 100.00 EUR.
- Steps: 1. Enter and apply `SAVE10`.
- Expected: Rejected with an "expired" message. No discount.

### TC-11: Code after 23:59 on the end date
- Preconditions: `SAVE10` end date is today. System time is 23:59:30 (or 00:00:00 next day).
- Steps: 1. Apply `SAVE10`.
- Expected: Behaviour per Q2. For 00:00:00 the next day, the code is rejected as expired. For 23:59:30 see Q2.

### TC-12: Non-existent code
- Preconditions: No code `ABCXYZ` exists. Subtotal 100.00 EUR.
- Steps: 1. Enter and apply `ABCXYZ`.
- Expected: Rejected with an "invalid code" message. No discount.

### TC-13: Empty code
- Preconditions: Subtotal 100.00 EUR.
- Steps: 1. Leave the field empty. 2. Click Apply.
- Expected: No discount, validation message or disabled button. No error or crash.

### TC-14: Whitespace-only code
- Steps: 1. Enter spaces only. 2. Apply.
- Expected: Treated as empty or invalid. No discount, no server error.

### TC-15: Second code while one is applied
- Preconditions: `SAVE10` applied to a 100.00 EUR order. `OTHER5` is also valid.
- Steps: 1. Try to enter `OTHER5`.
- Expected: Only one code allowed. Either the input is blocked, the second code is rejected with a message, or it replaces the first [OQ Q4]. Discounts are never stacked (total is never 80.00 or 85.00 EUR).

### TC-16: Code with special characters / injection attempt
- Steps: 1. Enter `' OR 1=1 --` and `<script>alert(1)</script>`. 2. Apply.
- Expected: Rejected as invalid. No script runs, no server error, input is safely handled.

### TC-17: Removing items drops the order below the threshold after applying
- Preconditions: `SAVE10` applied to a 60.00 EUR order.
- Steps: 1. Remove items so the subtotal is 40.00 EUR. 2. View total.
- Expected: Discount is removed or the code is flagged as no longer eligible. Total is 40.00 EUR, no discount [OQ Q3].

### TC-18: Code expires between applying and placing the order
- Preconditions: `SAVE10` end date is today. Applied at 23:55.
- Steps: 1. Wait until after 23:59. 2. Place the order.
- Expected: Validity is re-checked at order placement. Behaviour per Q2.

## Boundary

### TC-19: Subtotal 50.01 EUR
- Preconditions: `SAVE10` active.
- Steps: 1. Apply `SAVE10`.
- Expected: Accepted. Discount 5.00 EUR (5.001 rounded). Total 45.01 EUR.

### TC-20: Subtotal 49.995 EUR equivalent / rounding at threshold
- Preconditions: Order whose pre-rounding subtotal is 49.995 EUR (for example from per-unit pricing with tax or weights), displayed as 50.00 EUR.
- Steps: 1. Apply `SAVE10`.
- Expected: Depends on whether the threshold is checked on the rounded or unrounded amount [OQ Q1, Q6].

### TC-21: Last minute of the end date (23:59:00)
- Preconditions: `SAVE10` end date is today. System time 23:59:00.
- Steps: 1. Apply `SAVE10`.
- Expected: Accepted ("expire at 23:59" read as valid until 23:59 inclusive) [OQ Q2].

### TC-22: 23:59:59 on the end date
- Preconditions: As TC-21, time 23:59:59.
- Steps: 1. Apply `SAVE10`.
- Expected: Depends on Q2 (expired if the cut-off is 23:59:00, valid if 23:59:59 is inclusive).

### TC-23: 00:00:00 the day after the end date
- Expected: Rejected as expired.

### TC-24: Very large order
- Preconditions: Subtotal 999,999.99 EUR (or system maximum).
- Steps: 1. Apply `SAVE10`.
- Expected: Discount is exactly 10%, no overflow or precision error.

### TC-25: Code length and case boundaries
- Steps: 1. Enter the minimum and maximum permitted code length, in upper, lower and mixed case. 2. Enter a code one character over the maximum.
- Expected: Valid-length codes behave the same regardless of case. Over-length input is rejected or truncated per the spec [OQ Q7].

### TC-26: Leading and trailing spaces
- Steps: 1. Enter ` SAVE10 ` (with spaces). 2. Apply.
- Expected: Per Q8; most likely trimmed and accepted. Must not produce a server error.

### TC-27: Time zone boundary
- Preconditions: Code end date is today; customer is in a different time zone than the server.
- Steps: 1. Apply the code at a time that is before 23:59 in one zone and after in the other.
- Expected: Per Q2 (which time zone defines 23:59).

## Open questions (not assumed in the cases above)

1. What counts as the "order of 50 EUR"? Subtotal before discount, including or excluding shipping, taxes, and items already discounted? Is the check on the rounded or unrounded amount?
2. Which time zone defines "23:59"? Is the code valid through 23:59:00, through 23:59:59, or through the end of the day? Is the end date inclusive? The wording "expire at 23:59" suggests a cut-off at the start or end of that minute, which is ambiguous.
3. What happens when the cart changes after the code is applied (subtotal drops below 50 EUR)? Is the discount removed automatically or is checkout blocked? What error messages are required for each rejection reason (below minimum, expired, invalid)?
4. If a code is already applied, what happens when the customer enters a second one: blocked, rejected, or replaced?
5. Does the 10% apply to the whole order including shipping and tax, or only to item value? Does it combine with other promotions or sale prices?
6. What is the rounding rule for the discount amount (half-up, half-even, truncate) and the currency precision?
7. What are the valid code format and length, and is there a limit on failed attempts (brute-force protection)?
8. Should leading/trailing whitespace be trimmed?
9. Is a code single-use or multi-use per customer or overall? Is there a usage limit? Can guests use codes?
10. Is there a start date, or is a code valid from creation? The requirement only defines an end date.
11. Is the discount re-validated when the order is placed (see TC-18), and what is shown to the customer if it now fails?
12. How should orders in currencies other than EUR be handled?
13. How are refunds or partial returns handled for discounted orders (out of scope or not)?
