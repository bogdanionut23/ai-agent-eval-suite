# TC06 - Test cases: add product units to cart (1-10, limited by stock)

## Requirement

A customer can add between 1 and 10 units of the same product to the cart. The quantity cannot exceed the stock available at that moment.

## Assumptions used for the test design

These are the minimum assumptions needed to write concrete steps. Each one is also listed under Open questions. Change the expected results if the answers differ.

- "Add to cart" is the action that takes a quantity of one product. The quantity is sent as a whole number.
- The 1-10 limit applies to a single add action. It is not a cumulative limit on the cart line (see Open questions, Q1).
- "Stock available" means the sellable stock for that product at the moment of the add, and a quantity above it is rejected.
- A rejected add leaves the cart unchanged and shows an error to the customer.

## Test cases

### Positive scenarios

**TC06-01 - Add the minimum quantity of 1 unit**
- Preconditions: Product P is in stock with at least 1 unit available. The customer is logged in or a guest, with an empty cart.
- Steps:
  1. Open the product detail page for P.
  2. Set the quantity to 1.
  3. Click "Add to cart".
- Expected result: P is added to the cart with quantity 1. The cart count and the cart total update. No error is shown.

**TC06-02 - Add a mid-range quantity**
- Preconditions: P has at least 5 units in stock. The cart is empty.
- Steps:
  1. Open the product detail page for P.
  2. Set the quantity to 5.
  3. Click "Add to cart".
- Expected result: P is added with quantity 5. The cart shows 5 units of P.

**TC06-03 - Add the maximum quantity of 10 units**
- Preconditions: P has at least 10 units in stock. The cart is empty.
- Steps:
  1. Open the product detail page for P.
  2. Set the quantity to 10.
  3. Click "Add to cart".
- Expected result: P is added with quantity 10. No error is shown.

**TC06-04 - Add a quantity equal to the available stock**
- Preconditions: P has exactly 7 units in stock. The cart is empty.
- Steps:
  1. Open the product detail page for P.
  2. Set the quantity to 7.
  3. Click "Add to cart".
- Expected result: P is added with quantity 7. The stock limit is inclusive, so a quantity equal to the stock is allowed.

**TC06-05 - Add the last remaining unit**
- Preconditions: P has exactly 1 unit in stock. The cart is empty.
- Steps:
  1. Open the product detail page for P.
  2. Set the quantity to 1.
  3. Click "Add to cart".
- Expected result: P is added with quantity 1. The add succeeds because 1 is both the minimum and the stock.

### Boundary scenarios

**TC06-06 - Quantity 0 is rejected (below minimum)**
- Preconditions: P has stock of at least 1. The cart is empty.
- Steps:
  1. Open the product detail page for P.
  2. Set the quantity to 0.
  3. Click "Add to cart".
- Expected result: The add is rejected. An error message says the quantity must be between 1 and 10. The cart stays empty.

**TC06-07 - Quantity 11 is rejected (above maximum, stock is sufficient)**
- Preconditions: P has at least 11 units in stock. The cart is empty.
- Steps:
  1. Open the product detail page for P.
  2. Set the quantity to 11.
  3. Click "Add to cart".
- Expected result: The add is rejected because the quantity is above 10, even though the stock allows it. An error message is shown. The cart stays empty.

**TC06-08 - Quantity 10 with stock of 9 is rejected (stock is the lower limit)**
- Preconditions: P has exactly 9 units in stock. The cart is empty.
- Steps:
  1. Open the product detail page for P.
  2. Set the quantity to 10.
  3. Click "Add to cart".
- Expected result: The add is rejected because 10 is more than the 9 units in stock. An error message is shown. The cart stays empty.

**TC06-09 - Quantity one above the stock is rejected**
- Preconditions: P has exactly 7 units in stock. The cart is empty.
- Steps:
  1. Open the product detail page for P.
  2. Set the quantity to 8.
  3. Click "Add to cart".
- Expected result: The add is rejected. An error message is shown that names the available stock (7), if the UI supports this. The cart stays empty.

**TC06-10 - Quantity 2 with stock of 1 is rejected**
- Preconditions: P has exactly 1 unit in stock. The cart is empty.
- Steps:
  1. Open the product detail page for P.
  2. Set the quantity to 2.
  3. Click "Add to cart".
- Expected result: The add is rejected. The cart stays empty.

### Negative and invalid-input scenarios

**TC06-11 - Negative quantity is rejected**
- Preconditions: P has stock of at least 1. The cart is empty.
- Steps:
  1. Open the product detail page for P.
  2. Set the quantity to -1.
  3. Click "Add to cart".
- Expected result: The add is rejected with a validation error. The cart stays empty and the stock is not changed.

**TC06-12 - Non-integer quantity is rejected**
- Preconditions: P has stock of at least 3. The cart is empty.
- Steps:
  1. Open the product detail page for P.
  2. Set the quantity to 2.5.
  3. Click "Add to cart".
- Expected result: The add is rejected with a validation error. The cart stays empty. (Assumes whole units only. See Open questions, Q3.)

**TC06-13 - Non-numeric quantity is rejected**
- Preconditions: P has stock of at least 1. The cart is empty.
- Steps:
  1. Open the product detail page for P.
  2. Enter "abc" in the quantity field, or try to submit with a non-numeric value through the API.
  3. Click "Add to cart".
- Expected result: The add is rejected with a validation error. The cart stays empty. The application does not crash.

**TC06-14 - Empty quantity is rejected or defaulted**
- Preconditions: P has stock of at least 1. The cart is empty.
- Steps:
  1. Open the product detail page for P.
  2. Clear the quantity field so it is empty.
  3. Click "Add to cart".
- Expected result: Either the add is rejected with a validation error, or the quantity defaults to 1 and the add succeeds. The behaviour must be the one the product owner chooses (see Open questions, Q4). In both cases, the cart must not contain an item with an empty or null quantity.

**TC06-15 - Product with zero stock cannot be added**
- Preconditions: P has 0 units in stock (sold out). The cart is empty.
- Steps:
  1. Open the product detail page for P.
  2. Set the quantity to 1.
  3. Click "Add to cart".
- Expected result: The add is rejected because 1 is more than the 0 units in stock. An error message says the product is out of stock. The cart stays empty.

### Stock timing scenarios ("at that moment")

**TC06-16 - Stock is reduced by another customer before the add is submitted**
- Preconditions: P has 5 units in stock when the customer loads the product page. A second customer buys 4 units, so stock falls to 1 before the first customer submits. The cart is empty.
- Steps:
  1. Customer A loads the product page for P, which shows stock 5.
  2. Customer B buys 4 units of P, so stock is now 1.
  3. Customer A sets the quantity to 3 and clicks "Add to cart".
- Expected result: The add is rejected because the check is made against the stock at the moment of the add (1), not the stock shown on the page. An error is shown. The cart stays empty. (The exact message and whether the page is refreshed are open questions, Q6.)

**TC06-17 - Two customers race for the last unit**
- Preconditions: P has exactly 1 unit in stock. Customers A and B each have an empty cart.
- Steps:
  1. Customer A and customer B open the product page for P at the same time.
  2. Both set the quantity to 1.
  3. Both click "Add to cart" at the same time.
- Expected result: Only one add succeeds. The other is rejected with the out-of-stock or stock-limit error. Stock never goes below 0, and the total of units in carts does not exceed the stock.

**TC06-18 - Stock check at add time is not based on an earlier cart state**
- Preconditions: P has 10 units in stock. Customer A has already added 4 units of P to the cart (see Open questions, Q1).
- Steps:
  1. Customer A sets the quantity to 6 and clicks "Add to cart".
- Expected result: The add succeeds only if the combined cart quantity (4 + 6 = 10) is within the 1-10 limit and the stock. The result is 10 units in the cart. The exact rule depends on Q1 and is not yet defined. This case is blocked until Q1 is answered.

## Open questions

These points are not stated in the requirement. The test cases above do not depend on a guess for them, except where noted.

1. **Cumulative limit.** If a customer adds 6 units, then adds 6 more of the same product, is the limit of 10 applied to the single add (6 is allowed twice, giving 12 in the cart) or to the total in the cart line (the second add is rejected or capped at 4)? Does the same rule apply to the stock check (stock minus units already in the cart)?
2. **Editing the quantity in the cart.** Is changing the quantity of a cart line (not adding) in scope? If yes, does the same 1-10 and stock limit apply there, and what happens when the new quantity is above stock?
3. **Units type.** Are products sold only in whole units? If fractional units are valid for some products (for example by weight), what is the rule? TC06-12 assumes whole units only.
4. **Empty or missing quantity.** Should an empty quantity be rejected or default to 1? TC06-14 accepts either, but the product owner must pick one.
5. **Stock that goes down after the add.** If stock falls below the cart quantity after the item was added (for example, another customer buys first), is the customer blocked at checkout, shown a warning in the cart, or is the item removed? The requirement covers only the moment of the add.
6. **Error behaviour and messages.** What is the exact error text for each rejection (quantity out of range, quantity above stock, out of stock)? Should the message show the available stock number? Should the product page refresh the stock value after a failed add?
7. **Reservation of stock.** Does adding to the cart reserve stock for the customer, or is stock only reduced at order placement? This decides whether two carts can both hold units that together exceed the stock (TC06-17 assumes that they cannot).
8. **Stock source and freshness.** Is "stock available at that moment" the value read from the database at the moment of the request, or a cached value? What is the maximum allowed delay for stock updates?
9. **Products with no stock tracking.** Do some products have unlimited stock (for example digital goods)? If so, is the 1-10 limit still applied, and what happens to the stock check?
10. **Guest and logged-in carts.** Do guest carts and logged-in carts follow the same rule? Does merging a guest cart into a customer account at login re-check the quantities against stock?
11. **Multiple variants.** If a product has variants (size, colour), is the stock check made per variant? The requirement says "the same product", so the variant rule must be confirmed.
12. **Limit as a fixed value.** Is 10 a fixed business rule or a setting that can change per product or per market? TC06-03 and TC06-07 assume a fixed value of 10.
