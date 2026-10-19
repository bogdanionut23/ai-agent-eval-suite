# TC06 – Test cases: cart quantity (1–10, limited by stock)

Requirement: A customer can add between 1 and 10 units of the same product to the cart. The quantity cannot exceed the stock available at that moment.

Notation: S = stock available at the moment of the action; Q = requested quantity. The effective maximum is min(10, S).

## Test cases

### Positive

**TC06-P01 – Add minimum quantity (1)**
- Preconditions: Product A exists, S = 50. Cart is empty.
- Steps: 1. Open product A. 2. Set quantity to 1. 3. Click "Add to cart".
- Expected: Cart contains product A x 1. No error is shown.

**TC06-P02 – Add a mid-range quantity (5)**
- Preconditions: Product A, S = 50. Cart is empty.
- Steps: 1. Set quantity to 5. 2. Add to cart.
- Expected: Cart contains product A x 5.

**TC06-P03 – Add maximum quantity (10) with ample stock**
- Preconditions: Product A, S = 50. Cart is empty.
- Steps: 1. Set quantity to 10. 2. Add to cart.
- Expected: Cart contains product A x 10. No error.

**TC06-P04 – Quantity equal to stock, stock below 10**
- Preconditions: Product B, S = 3. Cart is empty.
- Steps: 1. Set quantity to 3. 2. Add to cart.
- Expected: Cart contains product B x 3. No error.

**TC06-P05 – Stock exactly 10, add 10**
- Preconditions: Product C, S = 10. Cart empty.
- Steps: 1. Set quantity to 10. 2. Add to cart.
- Expected: Cart contains product C x 10.

**TC06-P06 – Different products are counted separately**
- Preconditions: Products A and D, S = 50 each. Cart contains A x 10.
- Steps: 1. Add D x 10.
- Expected: Cart contains A x 10 and D x 10. The limit applies per product.

### Boundary

**TC06-B01 – Quantity 0**
- Preconditions: Product A, S = 50.
- Steps: 1. Enter quantity 0. 2. Add to cart.
- Expected: Rejected with a message that the minimum is 1. Cart unchanged.

**TC06-B02 – Quantity 11 (one above max) with ample stock**
- Preconditions: Product A, S = 50. Cart empty.
- Steps: 1. Enter quantity 11. 2. Add to cart.
- Expected: Rejected (or blocked by the control) with a message that the maximum is 10. Cart unchanged.

**TC06-B03 – Quantity = stock + 1, stock below 10**
- Preconditions: Product B, S = 3.
- Steps: 1. Enter quantity 4. 2. Add to cart.
- Expected: Rejected with a message that only 3 are available. Cart unchanged.

**TC06-B04 – Stock = 1**
- Preconditions: Product E, S = 1. Cart empty.
- Steps: 1. Add quantity 1. 2. Try again with quantity 2 (new cart).
- Expected: Quantity 1 is accepted. Quantity 2 is rejected with an availability message.

**TC06-B05 – Stock = 11, request 10 and 11**
- Preconditions: Product F, S = 11.
- Steps: 1. Add 10. 2. In a new empty cart, try 11.
- Expected: 10 is accepted. 11 is rejected because of the maximum of 10 (not stock).

**TC06-B06 – Stock = 9, request 10**
- Preconditions: Product G, S = 9.
- Steps: 1. Enter quantity 10. 2. Add to cart.
- Expected: Rejected, stock is the limiting factor (max 9).

### Negative

**TC06-N01 – Negative quantity**
- Preconditions: Product A, S = 50.
- Steps: 1. Enter -1. 2. Add to cart.
- Expected: Rejected with a validation message. Cart unchanged.

**TC06-N02 – Non-numeric input**
- Preconditions: Product A, S = 50.
- Steps: 1. Enter "abc" (and separately "1e1", "5,5"). 2. Add to cart.
- Expected: Input rejected or ignored with a validation message. Nothing added.

**TC06-N03 – Decimal quantity**
- Preconditions: Product A, S = 50.
- Steps: 1. Enter 2.5. 2. Add to cart.
- Expected: Rejected (units are whole numbers). Cart unchanged.

**TC06-N04 – Empty quantity field**
- Preconditions: Product A, S = 50.
- Steps: 1. Clear the quantity field. 2. Add to cart.
- Expected: Validation message, or a defined default. Nothing invalid is added.

**TC06-N05 – Out-of-stock product (S = 0)**
- Preconditions: Product H, S = 0.
- Steps: 1. Open product H. 2. Try to add 1.
- Expected: Add is blocked or disabled with an out-of-stock message. Cart unchanged.

**TC06-N06 – Cumulative additions exceed 10**
- Preconditions: Product A, S = 50. Cart contains A x 8.
- Steps: 1. Add 3 more of A.
- Expected: Rejected (total would be 11). Cart still A x 8. (See open question Q1 on exact behaviour.)

**TC06-N07 – Cumulative additions exceed stock**
- Preconditions: Product B, S = 5. Cart contains B x 4.
- Steps: 1. Add 2 more of B.
- Expected: Rejected (total 6 > 5). Cart still B x 4.

**TC06-N08 – Cumulative additions reaching the limit exactly**
- Preconditions: Product A, S = 50. Cart contains A x 8.
- Steps: 1. Add 2 more of A.
- Expected: Accepted. Cart contains A x 10.

**TC06-N09 – Stock decreases between page load and add (race)**
- Preconditions: Product B, S = 5. Customer 1 has the page open showing 5 available.
- Steps: 1. Customer 2 purchases 3 units (S becomes 2). 2. Customer 1 adds 4.
- Expected: Customer 1's request is rejected based on current stock (2), with a message. Server-side check, not only UI.

**TC06-N10 – Two customers add the last units concurrently**
- Preconditions: Product E, S = 1. Two customers have it open.
- Steps: 1. Both click "Add to cart" at the same time.
- Expected: At most the available stock is allocated; one request fails with an availability message. No oversell (subject to Q3 on whether the cart reserves stock).

**TC06-N11 – Bypass UI limits via API**
- Preconditions: Product A, S = 50. Authenticated session.
- Steps: 1. Send an add-to-cart API request with quantity 11, 0, -1, and a value above stock directly.
- Expected: Each request is rejected with a client error; the server enforces the rules independently of the UI.

**TC06-N12 – Quantity change in the cart (increase/decrease)**
- Preconditions: Cart contains A x 5, S = 50.
- Steps: 1. Increase to 10. 2. Increase to 11. 3. Decrease to 0.
- Expected: 10 accepted. 11 rejected. 0 handled per defined behaviour (see Q5). Same limits as on the add action.

**TC06-N13 – Stock drops below cart quantity after adding**
- Preconditions: Cart contains B x 5, S = 5. Stock then drops to 3 (e.g. other purchase or stock adjustment).
- Steps: 1. Open the cart / proceed to checkout.
- Expected: The user is informed and quantity is corrected or blocked per defined behaviour (see Q4).

## Open questions (not assumed in the cases above)

1. When an add would push the cumulative cart quantity of a product above 10 (or above stock), is the whole request rejected, or is the quantity clamped to the allowed maximum? Is the limit on the single add action or on the total in the cart?
2. Is the limit of 10 per cart line, per cart, or per order? Do different variants (size, colour) of one product count as the same product?
3. Does adding to the cart reserve stock (and for how long), or is stock only checked at add time and again at checkout? Does "available" exclude units reserved in other customers' carts?
4. What happens if stock falls below the quantity already in the cart (auto-reduce, warning, block at checkout)? Is stock re-validated at checkout?
5. What is the behaviour for quantity 0 in the cart: is it a removal, or a validation error?
6. If stock is below 10, what is the maximum shown to the user, and what exact error messages/wording and UI control (free text, stepper, dropdown) are expected?
7. How should out-of-stock products and backorder/pre-order items be treated? Is stock 0 an add-block or a notification option?
8. Does the rule apply equally to guests and logged-in customers, to all channels (web, app, API), and to B2B/wholesale customers?
9. Are decimal-unit products (weight-based) in scope, or only whole units?
10. What are the performance and concurrency expectations (acceptable staleness of stock data)?
