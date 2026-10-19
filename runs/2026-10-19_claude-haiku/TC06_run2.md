# TC06 - Test cases: add product quantity to cart (1-10 units, bounded by stock)

## Requirement

A customer can add between 1 and 10 units of the same product to the cart. The quantity cannot exceed the stock available at that moment.

## Assumptions (stated, not invented)

- "Add" means a single add-to-cart action for one product, with a quantity field or selector.
- "Stock available at that moment" means the stock value read at the time of the add action.
- Limits are inclusive: 1 and 10 are valid, 0 and 11 are invalid.
- Where the requirement is silent, the test cases below mark the expected result as "TBD (see open questions)" and do not assume a behaviour.

## Test cases

### Positive scenarios

**TC06-P01: Add a quantity of 1 (lower boundary, valid)**
- Preconditions: Product P is in stock with stock >= 1. Customer is logged in or a guest session is active. Cart does not contain P.
- Steps:
  1. Open the product detail page for P.
  2. Set quantity to 1.
  3. Click "Add to cart".
- Expected result: P is added to the cart with quantity 1. The cart count and cart total update. No error message is shown.

**TC06-P02: Add a quantity of 10 (upper boundary, valid)**
- Preconditions: Product P has stock >= 10. Cart does not contain P.
- Steps:
  1. Open the product detail page for P.
  2. Set quantity to 10.
  3. Click "Add to cart".
- Expected result: P is added with quantity 10. No error is shown.

**TC06-P03: Add a mid-range quantity (e.g. 5)**
- Preconditions: Product P has stock >= 5. Cart does not contain P.
- Steps:
  1. Set quantity to 5.
  2. Click "Add to cart".
- Expected result: P is added with quantity 5.

**TC06-P04: Quantity equal to stock (valid, stock boundary)**
- Preconditions: Product P has stock exactly 3 (use 3 <= 10 so the stock limit is the binding one). Cart does not contain P.
- Steps:
  1. Set quantity to 3.
  2. Click "Add to cart".
- Expected result: P is added with quantity 3. Remaining stock is reported as 0 (or the product is shown as unavailable for further adds, per the stock display rule).

**TC06-P05: Stock larger than 10, quantity capped only by the 10-unit rule**
- Preconditions: Product P has stock 50. Cart does not contain P.
- Steps:
  1. Set quantity to 10.
  2. Click "Add to cart".
- Expected result: P is added with quantity 10. The 10-unit limit applies even though stock is higher.

### Negative scenarios

**TC06-N01: Quantity of 0 is rejected**
- Preconditions: Product P is in stock. Cart does not contain P.
- Steps:
  1. Set quantity to 0 (or leave the field empty and enter 0).
  2. Click "Add to cart".
- Expected result: The item is not added. A validation message is shown stating the minimum is 1. Cart content is unchanged.

**TC06-N02: Quantity of 11 is rejected**
- Preconditions: Product P has stock >= 11. Cart does not contain P.
- Steps:
  1. Set quantity to 11.
  2. Click "Add to cart".
- Expected result: The item is not added. A message states the maximum per add is 10. Cart content is unchanged.

**TC06-N03: Quantity above stock is rejected**
- Preconditions: Product P has stock 4. Cart does not contain P.
- Steps:
  1. Set quantity to 5 (within 1-10, above stock).
  2. Click "Add to cart".
- Expected result: The item is not added. A message states that only 4 units are available. Cart content is unchanged. Stock is not decremented.

**TC06-N04: Product with zero stock cannot be added**
- Preconditions: Product P has stock 0.
- Steps:
  1. Attempt to set quantity to 1 and click "Add to cart" (or observe that the control is disabled).
- Expected result: The item is not added. The customer sees an out-of-stock message. Cart content is unchanged.

**TC06-N05: Negative quantity is rejected**
- Preconditions: Product P is in stock.
- Steps:
  1. Enter -1 in the quantity field.
  2. Click "Add to cart".
- Expected result: The item is not added. A validation message is shown. The field cannot be submitted with a negative value.

**TC06-N06: Non-integer quantity is rejected**
- Preconditions: Product P is in stock.
- Steps:
  1. Enter 2.5 in the quantity field.
  2. Click "Add to cart".
- Expected result: The item is not added. A validation message is shown. (Whether units are whole-number only is an open question; see Q1.)

**TC06-N07: Non-numeric input is rejected**
- Preconditions: Product P is in stock.
- Steps:
  1. Enter "abc" (or special characters) in the quantity field.
  2. Click "Add to cart".
- Expected result: The field rejects the input or the add fails with a validation message. No item is added and no error page or unhandled exception is shown.

**TC06-N08: Empty quantity field**
- Preconditions: Product P is in stock.
- Steps:
  1. Clear the quantity field.
  2. Click "Add to cart".
- Expected result: The item is not added. A validation message is shown, or the empty value defaults per the open question Q2.

### Boundary scenarios

**TC06-B01: Quantity 0 vs 1 (lower edge)**
- Preconditions: Product P is in stock with stock >= 1.
- Steps: Run TC06-N01 (qty 0) and TC06-P01 (qty 1) back to back.
- Expected result: 0 is rejected, 1 is accepted. The two results differ.

**TC06-B02: Quantity 10 vs 11 (upper edge)**
- Preconditions: Product P has stock >= 11.
- Steps: Run TC06-P02 (qty 10) and TC06-N02 (qty 11) back to back.
- Expected result: 10 is accepted, 11 is rejected.

**TC06-B03: Stock 1, quantity 1 (stock equals minimum)**
- Preconditions: Product P has stock exactly 1.
- Steps: Set quantity to 1 and click "Add to cart".
- Expected result: Added with quantity 1. Stock becomes 0.

**TC06-B04: Stock 1, quantity 2 (stock just below requested)**
- Preconditions: Product P has stock exactly 1.
- Steps: Set quantity to 2 and click "Add to cart".
- Expected result: Rejected. Message states only 1 unit is available. Cart unchanged.

**TC06-B05: Stock 10, quantity 10 (both limits coincide)**
- Preconditions: Product P has stock exactly 10.
- Steps: Set quantity to 10 and click "Add to cart".
- Expected result: Added with quantity 10. Stock becomes 0.

**TC06-B06: Stock 10, quantity 11 (stock allows it, rule does not)**
- Preconditions: Product P has stock exactly 10 and... (stock limit would allow 10; the 10-unit rule is the limit to check). Use stock 20.
- Steps: Set quantity to 11 and click "Add to cart".
- Expected result: Rejected because of the 10-unit maximum, not stock. Message states the maximum per add is 10.

**TC06-B07: Stock changes between page load and add**
- Preconditions: Product P shows stock 5 on the page. Another session reduces stock to 2 after the page loaded but before the add.
- Steps:
  1. Load the product page (stock displayed as 5).
  2. In another session, buy 3 units of P so stock is 2.
  3. In the first session, set quantity to 4 and click "Add to cart".
- Expected result: Rejected. The check uses the stock at the moment of the add, so the add fails with a message that only 2 units are available. The stale displayed value of 5 does not allow the add.

**TC06-B08: Stock reduced after item is in cart**
- Preconditions: Customer has P (qty 3) in cart. Stock is then reduced to 1 by other orders.
- Steps:
  1. Open the cart.
  2. Attempt to increase P to 3 (total stays within 1-10 and the cart line is 3).
- Expected result: The increase is rejected. The rule applies to the quantity being added. Behaviour of the existing cart line is an open question (see Q4).

**TC06-B09: Adding in several steps reaching the limit**
- Preconditions: Product P has stock 20. Cart does not contain P.
- Steps:
  1. Add quantity 6.
  2. Add quantity 5 more.
- Expected result: Depends on Q3. If the 10-unit limit is per cart line, the second add is rejected and the cart holds 6. If it is per add action, the cart may hold 11. Record the actual result and check against the decision on Q3.

## Open questions

1. **Whole units only?** Is the quantity always an integer, or are fractional units allowed for some products (weight, length)? TC06-N06 depends on this.
2. **Empty and default quantity.** When the quantity field is empty, should it be rejected, or default to 1? Is a default selected in the UI?
3. **Limit scope.** Does "between 1 and 10 units" apply to a single add action, or to the total of this product in the cart? If total, what happens when the cart already holds 8 and the customer adds 3: reject, or add 2 and show a message? TC06-B09 depends on this.
4. **Existing cart lines and stock drops.** If stock drops below the quantity already in the cart, should the cart line be reduced, flagged, or left alone until checkout? Is the stock check also repeated at checkout?
5. **Stock check timing and concurrency.** Is stock reserved on add, or only checked? If two customers add the last unit at the same moment, which one succeeds? (TC06-B07 assumes a check at add time with no reservation.)
6. **Message wording and limits display.** What are the exact validation messages? Should the UI show the max (10) or the available stock before the customer enters a value?
7. **Quantity greater than 10 with stock above 10.** Should the UI cap the selector at min(10, stock), or allow entry and reject on submit? This affects TC06-N02 and TC06-B06.
8. **Zero-stock display.** When stock is 0, should the control be hidden, disabled, or shown with a notice? TC06-N04 assumes disabled or rejected.
9. **Stock meaning.** Does "stock available" mean physical stock only, or stock net of other customers' carts and reservations?
10. **Guest vs logged-in.** Do the limits and stock checks behave the same for guest sessions?
11. **Variants.** If a product has variants (size, colour), is the limit and stock per variant or per product? The requirement says "the same product", which is ambiguous for variants.
12. **Units of measure.** Is "unit" always one item, or can a unit be a pack?
