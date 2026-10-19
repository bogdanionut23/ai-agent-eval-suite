# TC06 - Test cases: cart quantity (1 to 10, limited by stock)

Requirement: A customer can add between 1 and 10 units of the same product to the cart. The quantity cannot exceed the stock available at that moment.

Notation: S = stock available at the moment of the action. Effective maximum = min(10, S). Where the requirement is silent, the expected result is marked "per OQ-n" and I do not assume behaviour.

## Test cases

### Positive

**TC06-01 - Add minimum quantity (1)**
- Preconditions: Product P in stock, S = 50. Cart empty. Customer logged in (or guest, if allowed).
- Steps: 1. Open P. 2. Set quantity 1. 3. Click Add to cart.
- Expected: P is in the cart with quantity 1. No error.

**TC06-02 - Add maximum quantity (10) with ample stock**
- Preconditions: S = 50. Cart empty.
- Steps: Set quantity 10, Add to cart.
- Expected: Cart contains P x 10. No error.

**TC06-03 - Add a mid-range quantity (5)**
- Preconditions: S = 50. Cart empty.
- Steps: Set quantity 5, Add to cart.
- Expected: Cart contains P x 5.

**TC06-04 - Add quantity exactly equal to stock (S = 7, qty 7)**
- Preconditions: S = 7. Cart empty.
- Steps: Set quantity 7, Add to cart.
- Expected: Cart contains P x 7. Accepted because quantity does not exceed stock.

**TC06-05 - Stock of exactly 1, add 1**
- Preconditions: S = 1. Cart empty.
- Steps: Add quantity 1.
- Expected: Cart contains P x 1.

### Boundary

**TC06-06 - Quantity 0**
- Preconditions: S = 50.
- Steps: Enter 0, Add to cart.
- Expected: Rejected with a validation message (minimum is 1). Cart unchanged.

**TC06-07 - Quantity 11 (one above maximum)**
- Preconditions: S = 50.
- Steps: Enter 11, Add to cart.
- Expected: Rejected (or blocked at the input) with a message that the maximum is 10. Cart unchanged.

**TC06-08 - Quantity 9 and 10 with S = 10**
- Preconditions: S = 10. Cart empty.
- Steps: Add 9, then in a separate run add 10.
- Expected: Both accepted (single add each).

**TC06-09 - Stock one below the request (S = 5, qty 6)**
- Preconditions: S = 5. Cart empty.
- Steps: Enter 6, Add to cart.
- Expected: Rejected; message indicates only 5 are available (exact wording per OQ-5). Cart unchanged.

**TC06-10 - Stock below 10 limits the effective maximum (S = 3)**
- Preconditions: S = 3.
- Steps: Add 3 (accepted), then in another run add 4.
- Expected: 3 accepted; 4 rejected.

**TC06-11 - Stock above 10 does not raise the cap (S = 100, qty 11)**
- Preconditions: S = 100.
- Steps: Enter 11, Add to cart.
- Expected: Rejected; the 10-unit cap applies regardless of stock.

**TC06-12 - Stock exactly 10, qty 10; stock exactly 11, qty 11**
- Preconditions: S = 10, then S = 11.
- Steps: Add 10 when S = 10. Add 11 when S = 11.
- Expected: 10 accepted. 11 rejected due to the cap of 10.

### Negative

**TC06-13 - Out of stock (S = 0)**
- Preconditions: S = 0.
- Steps: Attempt to add quantity 1 (via UI if the control is enabled, and via API).
- Expected: Not added; out-of-stock indication shown. Cart unchanged. Add-to-cart disabled in UI per OQ-6.

**TC06-14 - Negative quantity (-1)**
- Steps: Submit quantity -1 (UI input and direct API call).
- Expected: Rejected with validation error. Cart unchanged.

**TC06-15 - Non-numeric quantity ("abc", "", " ")**
- Steps: Enter each value and submit.
- Expected: Rejected with validation error; no server error (no 500).

**TC06-16 - Decimal quantity (2.5)**
- Steps: Enter 2.5 and submit.
- Expected: Rejected (or handled per OQ-7); not silently rounded without defined behaviour.

**TC06-17 - Very large or malformed number (99999999999, 1e3, 0010)**
- Steps: Submit each value via API.
- Expected: Values over 10 rejected; no overflow or crash. Handling of "0010" per OQ-7.

**TC06-18 - API bypass of UI limit**
- Preconditions: S = 50.
- Steps: Send an add-to-cart API request with quantity 11 directly.
- Expected: Server rejects with a 4xx error and a clear message. Validation is enforced server-side, not only in the UI.

### Cumulative quantity in the cart (depends on OQ-1)

**TC06-19 - Adding to an existing cart line stays within 10**
- Preconditions: S = 50. Cart has P x 6.
- Steps: Add 4 more of P.
- Expected: Cart has P x 10.

**TC06-20 - Adding to an existing line would exceed 10**
- Preconditions: S = 50. Cart has P x 6.
- Steps: Add 5 more of P.
- Expected: Per OQ-1: either rejected, or capped at 10 with a message. Cart total must never exceed 10.

**TC06-21 - Cumulative quantity exceeds stock**
- Preconditions: S = 8. Cart has P x 6.
- Steps: Add 3 more.
- Expected: Total 9 would exceed stock of 8; rejected or capped per OQ-1. Cart never exceeds 8.

**TC06-22 - Cart quantity edit (update in cart)**
- Preconditions: Cart has P x 3, S = 50.
- Steps: Change quantity to 10 (accepted), then to 11, then to 0.
- Expected: 10 accepted; 11 rejected; 0 handled per OQ-8 (rejected or line removed).

**TC06-23 - Same product, different variant (size/color)**
- Preconditions: P has variants A and B, each S = 50. Cart has A x 10.
- Steps: Add B x 1.
- Expected: Per OQ-2 (is the limit per product or per variant/SKU).

### Stock changes ("at that moment")

**TC06-24 - Stock drops after the product page was loaded**
- Preconditions: S = 10 when page loaded. Another user purchases 8 so S = 2.
- Steps: On the stale page, add 5.
- Expected: Server checks current stock (2); request rejected with message showing 2 available.

**TC06-25 - Two customers race for the last units**
- Preconditions: S = 3. Two sessions, each with empty cart.
- Steps: Both add 3 at (nearly) the same time.
- Expected: Behaviour per OQ-3 (reservation or no reservation). At no point is more than the allowed stock committed if stock is reserved on add. No server error.

**TC06-26 - Stock reduced while item already in cart**
- Preconditions: Cart has P x 5. Stock then drops to 3 (inventory adjustment).
- Steps: Open cart, proceed to checkout.
- Expected: Per OQ-4: user is informed and quantity is adjusted or blocked. Cart is not silently left in an invalid state.

**TC06-27 - Stock increases after a rejection**
- Preconditions: S = 2. Add 5 rejected. Stock replenished to 20.
- Steps: Retry add 5.
- Expected: Accepted.

### Other

**TC06-28 - Add via different entry points**
- Steps: Add quantity 11 from product page, quick-add on listing page, wishlist, and re-order flow (those that exist).
- Expected: The same limits apply on every entry point.

**TC06-29 - Rapid repeated clicks on Add to cart**
- Preconditions: S = 50, quantity 6.
- Steps: Double-click Add quickly.
- Expected: Total in cart follows OQ-1 and never exceeds 10 (no duplicate-submit bypass).

**TC06-30 - Guest vs logged-in cart merge**
- Preconditions: Guest cart P x 7. Logged-in saved cart P x 6 (if supported).
- Steps: Log in so the carts merge.
- Expected: Merged quantity does not exceed 10 or stock; behaviour per OQ-9.

## Open questions (not assumed)

1. OQ-1: Does the 1 to 10 limit apply per single add action, or to the total quantity of that product in the cart? If total, what happens when an add would exceed it: reject the whole add, or cap at the maximum and warn?
2. OQ-2: Is "same product" the product, or the variant/SKU (size, color)? Is the limit per variant or across variants?
3. OQ-3: Is stock reserved when an item is added to cart, or only checked at add time and again at checkout? Who wins in a race for the last units?
4. OQ-4: If stock falls below the cart quantity after the item was added, what should happen (auto-reduce, warning, block checkout, remove)?
5. OQ-5: Exact error messages and UI behaviour (disabled button, input clamped, inline message), and whether the message states the available quantity.
6. OQ-6: For S = 0, is the Add button hidden/disabled, and is a back-in-stock notification expected?
7. OQ-7: Input handling for decimals, leading zeros, whitespace, and pasted values (reject vs normalize).
8. OQ-8: Changing quantity to 0 in the cart: validation error or removal of the line?
9. OQ-9: Rules for guest/logged-in cart merge, multiple devices, and saved carts.
10. OQ-10: Is "stock available" net of items in other customers' carts, pending orders, or safety stock? Is it per warehouse/location?
11. OQ-11: Can the 10-unit cap be overridden for specific products, customer groups (e.g., B2B) or promotions?
12. OQ-12: What is the expected behaviour when stock is unlimited or untracked (digital goods, backorder/pre-order)?
13. OQ-13: Is the rule enforced at checkout and order-creation too, or only when adding to cart? Are there non-functional expectations (response time, concurrency)?
