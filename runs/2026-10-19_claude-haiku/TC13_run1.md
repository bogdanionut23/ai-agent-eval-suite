# TC13 - Test cases: Shipping cost rule

Requirement: "Shipping costs 15 EUR. Orders of 100 EUR or more ship for free within the country."

Assumptions used for the test cases (each one is also listed as an open question below):
- The order value is the merchandise subtotal in EUR, before shipping.
- "Within the country" means the shipping address is in the seller's home country.
- Prices are shown and charged in EUR.

## Positive scenarios

### TC13-01 - Domestic order below threshold is charged 15 EUR shipping
- Preconditions: Seller ships from/within the home country. Customer has a domestic shipping address. Cart subtotal is 50.00 EUR.
- Steps:
  1. Add products with a total of 50.00 EUR to the cart.
  2. Proceed to checkout and enter a domestic shipping address.
  3. Review the order summary.
- Expected result: Shipping line shows 15.00 EUR. Order total = subtotal + 15.00 EUR (65.00 EUR).

### TC13-02 - Domestic order well above threshold ships free
- Preconditions: Domestic shipping address. Cart subtotal is 150.00 EUR.
- Steps:
  1. Add products with a total of 150.00 EUR.
  2. Proceed to checkout with a domestic address.
  3. Review the order summary.
- Expected result: Shipping line shows 0.00 EUR (free). Order total = 150.00 EUR.

### TC13-03 - Free shipping is applied when the subtotal is exactly 100.00 EUR
- Preconditions: Domestic shipping address.
- Steps:
  1. Add products with a total of exactly 100.00 EUR.
  2. Proceed to checkout with a domestic address.
- Expected result: Shipping is 0.00 EUR. Order total = 100.00 EUR.

### TC13-04 - Free shipping is applied to a domestic order that is one cent above threshold
- Preconditions: Domestic shipping address.
- Steps:
  1. Add products with a total of 100.01 EUR.
  2. Proceed to checkout with a domestic address.
- Expected result: Shipping is 0.00 EUR. Order total = 100.01 EUR.

### TC13-05 - Shipping cost is added to the total when threshold is not reached (summary vs. payment)
- Preconditions: Domestic address. Subtotal 60.00 EUR.
- Steps:
  1. Complete checkout up to the payment step.
  2. Compare the amount shown on the payment step with the summary.
- Expected result: Both show 75.00 EUR (60.00 + 15.00). The amount charged equals the amount shown.

### TC13-06 - Order confirmation shows the same shipping decision as checkout
- Preconditions: Domestic address. Subtotal 120.00 EUR, free shipping applied at checkout.
- Steps:
  1. Place the order.
  2. Open the order confirmation (and the confirmation email, if sent).
- Expected result: Shipping shows 0.00 EUR in confirmation and email. Total is 120.00 EUR.

## Negative scenarios

### TC13-07 - Shipping is not waived for a subtotal just below threshold
- Preconditions: Domestic address.
- Steps:
  1. Add products with a total of 99.99 EUR.
  2. Proceed to checkout with a domestic address.
- Expected result: Shipping is charged at 15.00 EUR. Free shipping is not applied.

### TC13-08 - Shipping cost does not count toward the 100 EUR threshold
- Preconditions: Domestic address. Subtotal 85.00 EUR. (Open question Q4 - this test assumes the threshold is on the merchandise subtotal only.)
- Steps:
  1. Add products with a total of 85.00 EUR.
  2. Proceed to checkout. Observe the shipping fee (15.00 EUR).
  3. Verify the free-shipping threshold is not reached with the shipping fee included (85 + 15 = 100).
- Expected result: Shipping is still charged 15.00 EUR. The 15 EUR fee is not used to reach the threshold.

### TC13-09 - Cross-border order is not given the domestic free-shipping rule
- Preconditions: Shipping address is in a country other than the home country. Subtotal 150.00 EUR. (Expected fee for foreign destinations is an open question Q2.)
- Steps:
  1. Add products with a total of 150.00 EUR.
  2. Proceed to checkout with a foreign shipping address.
- Expected result: Shipping is NOT 0.00 EUR under the domestic rule. The amount shown must match the rate defined for that destination (to be confirmed, see Q2).

### TC13-10 - Cross-border order below 100 EUR
- Preconditions: Foreign shipping address. Subtotal 40.00 EUR.
- Steps:
  1. Add products with a total of 40.00 EUR.
  2. Proceed to checkout with a foreign address.
- Expected result: Behaviour not defined by the requirement (see Q2). Test is blocked until the foreign rate is known. Shipping must not be 0.00 EUR silently.

### TC13-11 - Empty cart cannot reach checkout with a shipping charge
- Preconditions: Cart is empty (0.00 EUR).
- Steps:
  1. Open checkout directly (URL or back button).
- Expected result: Checkout is blocked or redirected to cart. No order can be placed with 0.00 EUR merchandise plus shipping.

### TC13-12 - Missing or invalid shipping country
- Preconditions: Subtotal 120.00 EUR.
- Steps:
  1. Proceed to checkout.
  2. Leave the country empty, or enter an unsupported country.
- Expected result: Shipping is not finalised. The user gets a validation message. No free-shipping decision is made from an unknown country.

### TC13-13 - Subtotal falls below threshold after removing an item
- Preconditions: Domestic address. Cart subtotal 130.00 EUR (free shipping shown).
- Steps:
  1. In the cart, remove a 40.00 EUR item (new subtotal 90.00 EUR).
  2. Proceed to checkout.
- Expected result: Shipping is recalculated to 15.00 EUR. Total = 105.00 EUR. The previous free shipping does not carry over.

### TC13-14 - Subtotal reaches threshold after adding an item
- Preconditions: Domestic address. Cart subtotal 90.00 EUR (shipping 15.00 EUR).
- Steps:
  1. Add a 10.00 EUR item to the cart.
  2. Proceed to checkout.
- Expected result: Shipping is 0.00 EUR. Total = 100.00 EUR. The 15 EUR fee disappears.

### TC13-15 - Discount that drops subtotal below threshold
- Preconditions: Domestic address. Cart subtotal 105.00 EUR. A 10% discount code exists. (Basis for threshold after discount is open question Q3.)
- Steps:
  1. Apply the discount code. Discounted subtotal = 94.50 EUR.
  2. Proceed to checkout.
- Expected result: Depends on Q3. If threshold is applied after discount: shipping is 15.00 EUR. If before discount: shipping is 0.00 EUR. The result must be consistent with the rule chosen and must not change between cart, checkout and confirmation.

## Boundary scenarios

### TC13-16 - Threshold boundary at 99.99 / 100.00 / 100.01 (three-value check)
- Preconditions: Domestic address. Three separate carts.
- Steps:
  1. Cart A subtotal 99.99 EUR, go to checkout.
  2. Cart B subtotal 100.00 EUR, go to checkout.
  3. Cart C subtotal 100.01 EUR, go to checkout.
- Expected result: A = 15.00 EUR shipping. B = 0.00 EUR shipping. C = 0.00 EUR shipping.

### TC13-17 - Rounding at the threshold with fractional cent amounts
- Preconditions: Domestic address. Two items: 49.995 EUR and 50.005 EUR (sum 100.00 EUR). Behaviour of sub-cent prices is open question Q6.
- Steps:
  1. Add both items to the cart.
  2. Proceed to checkout.
- Expected result: The subtotal is rounded to cents before the comparison. Result is free shipping if the rounded subtotal is 100.00 EUR. Confirm the rule (Q6).

### TC13-18 - Very large order
- Preconditions: Domestic address. Subtotal 100 000.00 EUR (e.g., bulk order).
- Steps:
  1. Add items totalling 100 000.00 EUR.
  2. Proceed to checkout.
- Expected result: Shipping 0.00 EUR. Total shows 100 000.00 EUR with correct formatting. No overflow or rounding error.

### TC13-19 - Zero-value and minimum order
- Preconditions: Domestic address.
- Steps:
  1. Try to add a 0.00 EUR item (e.g., a free sample) and check out.
  2. Try an order of 0.01 EUR.
- Expected result: A 0.00 EUR order is either blocked or charged 15.00 EUR shipping. A 0.01 EUR order is charged 15.00 EUR shipping. Neither is free.

### TC13-20 - Refund or partial cancellation after a free-shipping order
- Preconditions: Domestic order placed with subtotal 120.00 EUR (free shipping). Customer returns a 30.00 EUR item after delivery. (Refund of shipping is open question Q7.)
- Steps:
  1. Process the return of the 30.00 EUR item.
  2. Check the order value and any refund amount.
- Expected result: Not defined by the requirement (see Q7). The refund is calculated per the agreed policy and the shipping decision is not retroactively changed silently.

## Open questions

Q1. Country definition: Which country is "the country"? Is it the seller's home country only, or a list of countries? Is it based on shipping address or billing address?

Q2. Shipping fee outside the country: The requirement only gives the domestic fee. What is the shipping fee for foreign destinations, and does 15 EUR apply to them when the order is below 100 EUR? Is free shipping ever available abroad?

Q3. Threshold basis: Is the 100 EUR threshold checked against the subtotal before or after discounts and coupons? Does a gift card or store credit count toward it?

Q4. Shipping cost in the threshold: Does the 15 EUR fee count toward the 100 EUR (e.g., 85 + 15)? The tests assume it does not.

Q5. VAT / tax basis: Is the 100 EUR threshold compared to the gross amount (VAT included) or the net amount? Is the 15 EUR shipping charge gross or net?

Q6. Rounding: How are sub-cent amounts and rounding handled before the comparison with 100 EUR?

Q7. Returns and cancellations: If an order is partially returned or cancelled and drops below 100 EUR, is shipping charged retroactively, refunded, or left as is?

Q8. Re-evaluation: Is the free-shipping decision recalculated on every cart change, or only once when checkout starts?

Q9. Split shipments: If one order is split into several parcels, is the threshold applied to the whole order or per parcel?

Q10. Currency: Are prices always in EUR? Is the threshold applied to a converted amount if a customer views the shop in another currency?

Q11. Product exclusions: Are any products excluded from the threshold or from shipping charges (for example digital goods, gift cards, oversized items)?

Q12. Shipping methods: Does the 15 EUR apply to every shipping method (for example express or pickup)? Is pickup in a store ever charged?

Q13. Display: Must the storefront show a message about the free-shipping threshold (for example "Add 10 EUR more for free shipping") in cart and checkout? This is not stated in the requirement and is not tested here.
