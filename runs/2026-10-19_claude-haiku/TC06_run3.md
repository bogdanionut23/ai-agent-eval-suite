# TC06 - Test cases: add to cart with quantity limits

**Requirement:** A customer can add between 1 and 10 units of the same product to the cart. The quantity cannot exceed the stock available at that moment.

**Working assumptions (used only to write the cases below; each is listed as an open question):**
- "Between 1 and 10" is inclusive: 1 and 10 are valid.
- The 1-10 limit applies to a single add action. Cumulative handling is covered in the open questions and in TC-17 to TC-19.
- Stock is checked at the moment of the add action. Stock changes after the add are not covered by this requirement.
- A rejected add leaves the cart unchanged (no partial add).

---

## Positive scenarios

### TC-01 Add 1 unit (lower boundary, typical stock)
- **Preconditions:** Product P has stock 20. Cart is empty. Customer is logged in.
- **Steps:**
  1. Open the product detail page for P.
  2. Set quantity to 1.
  3. Click "Add to cart".
- **Expected result:** The add succeeds. The cart shows P with quantity 1. Available stock for P is still 20 (or displays 20, depending on whether stock is reserved at add time; see open questions).

### TC-02 Add a mid-range quantity
- **Preconditions:** Product P has stock 20. Cart is empty.
- **Steps:**
  1. Set quantity to 5.
  2. Click "Add to cart".
- **Expected result:** The add succeeds. The cart shows P with quantity 5.

### TC-03 Add 10 units (upper boundary, stock well above limit)
- **Preconditions:** Product P has stock 50. Cart is empty.
- **Steps:**
  1. Set quantity to 10.
  2. Click "Add to cart".
- **Expected result:** The add succeeds. The cart shows P with quantity 10.

### TC-04 Add quantity equal to stock (stock equals limit)
- **Preconditions:** Product P has stock exactly 10. Cart is empty.
- **Steps:**
  1. Set quantity to 10.
  2. Click "Add to cart".
- **Expected result:** The add succeeds. The cart shows P with quantity 10.

### TC-05 Add 1 unit when only 1 is in stock
- **Preconditions:** Product P has stock exactly 1. Cart is empty.
- **Steps:**
  1. Set quantity to 1.
  2. Click "Add to cart".
- **Expected result:** The add succeeds. The cart shows P with quantity 1.

### TC-06 Stock above limit, quantity at stock-minus-one boundary
- **Preconditions:** Product P has stock 11. Cart is empty.
- **Steps:**
  1. Set quantity to 10.
  2. Click "Add to cart".
- **Expected result:** The add succeeds because 10 is within both the 1-10 limit and the stock of 11. The cart shows P with quantity 10.

### TC-07 Two different products are limited independently
- **Preconditions:** Product A and product B each have stock 20. Cart is empty.
- **Steps:**
  1. Add 10 units of A.
  2. Add 10 units of B.
- **Expected result:** Both adds succeed. The cart shows A with quantity 10 and B with quantity 10. The limit for one product does not affect the other.

---

## Negative scenarios

### TC-08 Quantity above 10 with ample stock
- **Preconditions:** Product P has stock 50. Cart is empty.
- **Steps:**
  1. Set quantity to 11.
  2. Click "Add to cart".
- **Expected result:** The add is rejected with a validation message that states the maximum is 10. The cart does not contain P.

### TC-09 Quantity far above 10 (large value)
- **Preconditions:** Product P has stock 1,000,000. Cart is empty.
- **Steps:**
  1. Set quantity to 1000000.
  2. Click "Add to cart".
- **Expected result:** The add is rejected with the same maximum-quantity message as TC-08. No overflow or wrong total occurs. The cart does not contain P.

### TC-10 Quantity of zero
- **Preconditions:** Product P has stock 20. Cart is empty.
- **Steps:**
  1. Set quantity to 0.
  2. Click "Add to cart".
- **Expected result:** The add is rejected with a validation message that the minimum is 1. The cart does not contain P. (If the UI disables the button for 0, the expected result is that the add cannot be submitted.)

### TC-11 Negative quantity
- **Preconditions:** Product P has stock 20. Cart is empty.
- **Steps:**
  1. Enter -1 in the quantity field.
  2. Click "Add to cart".
- **Expected result:** The add is rejected with a validation message. The cart is unchanged. The cart total does not decrease.

### TC-12 Non-integer quantity
- **Preconditions:** Product P has stock 20. Cart is empty. Whether P is sold in fractional units is not specified (see open questions).
- **Steps:**
  1. Enter 2.5 in the quantity field.
  2. Click "Add to cart".
- **Expected result:** The add is rejected with a validation message, or the input is blocked. The cart does not contain P. (Confirm expected behaviour once the open questions are answered.)

### TC-13 Non-numeric quantity
- **Preconditions:** Product P has stock 20. Cart is empty.
- **Steps:**
  1. Enter "abc" in the quantity field, or try to type letters into it.
  2. Click "Add to cart".
- **Expected result:** The input is blocked or the add is rejected with a validation message. The cart does not contain P.

### TC-14 Empty quantity field
- **Preconditions:** Product P has stock 20. Cart is empty.
- **Steps:**
  1. Clear the quantity field so it is empty.
  2. Click "Add to cart".
- **Expected result:** The add is rejected with a validation message, or the default value is used and the rule applies to that value. The behaviour must match the open question on the default quantity.

### TC-15 Quantity exceeds stock (stock below 10)
- **Preconditions:** Product P has stock 4. Cart is empty.
- **Steps:**
  1. Set quantity to 5.
  2. Click "Add to cart".
- **Expected result:** The add is rejected with a message that the requested quantity exceeds available stock. The cart does not contain P. The message should not show the wrong maximum (10) as the limit.

### TC-16 Stock is the binding limit (stock lower than 10, quantity within 1-10)
- **Preconditions:** Product P has stock 9. Cart is empty.
- **Steps:**
  1. Set quantity to 10.
  2. Click "Add to cart".
- **Expected result:** The add is rejected. Quantity 10 is within the 1-10 range, but it exceeds stock 9. The cart does not contain P.

### TC-17 Out of stock (stock 0)
- **Preconditions:** Product P has stock 0. Cart is empty.
- **Steps:**
  1. Set quantity to 1.
  2. Click "Add to cart".
- **Expected result:** The add is rejected with an out-of-stock or stock-exceeded message. The cart does not contain P. (The "Add to cart" control may be disabled instead; either is acceptable if the cart stays unchanged.)

### TC-18 Quantity exceeds stock by one
- **Preconditions:** Product P has stock 5. Cart is empty.
- **Steps:**
  1. Set quantity to 6.
  2. Click "Add to cart".
- **Expected result:** The add is rejected. The cart does not contain P. Quantity 5 with the same stock must succeed (covered by TC-04/TC-05 pattern).

### TC-19 Rejected add does not partially change the cart
- **Preconditions:** Product P has stock 20. The cart already contains 3 units of P.
- **Steps:**
  1. Set quantity to 11.
  2. Click "Add to cart".
- **Expected result:** The add is rejected. The cart still shows P with quantity 3. It is not changed to 10 or 11.

---

## Cumulative and cart-state scenarios

These depend on open questions 1 and 2. The expected results assume the rule applies to the total quantity of the product in the cart. Confirm before running.

### TC-20 Cumulative total reaches exactly 10
- **Preconditions:** Product P has stock 50. The cart already contains 4 units of P.
- **Steps:**
  1. Add 6 units of P.
- **Expected result:** The add succeeds. The cart shows P with quantity 10.

### TC-21 Cumulative total exceeds 10
- **Preconditions:** Product P has stock 50. The cart already contains 6 units of P.
- **Steps:**
  1. Add 5 units of P.
- **Expected result:** The add is rejected (total would be 11). The cart still shows P with quantity 6.

### TC-22 Cumulative total exceeds stock
- **Preconditions:** Product P has stock 7. The cart already contains 5 units of P.
- **Steps:**
  1. Add 3 units of P.
- **Expected result:** The add is rejected (total would be 8, which exceeds stock 7). The cart still shows P with quantity 5.

### TC-23 Stock reduced after the item is already in the cart
- **Preconditions:** Product P has stock 20. The cart contains 5 units of P. Stock for P is then reduced to 3 by another order or by an admin change.
- **Steps:**
  1. Try to add 1 more unit of P.
- **Expected result:** The add is rejected because the new total (6) exceeds the current stock (3). The cart is unchanged. (Behaviour of the existing 5 units at checkout is an open question.)

### TC-24 Change quantity in cart to above limit
- **Preconditions:** Product P has stock 50. The cart contains 2 units of P.
- **Steps:**
  1. Open the cart. Change the quantity of P to 11 directly in the cart.
  2. Save or update the cart.
- **Expected result:** Expected behaviour is not specified (see open questions). If the rule applies to cart edits too, the change is rejected and the quantity stays at 2.

---

## Concurrency and repeated actions

### TC-25 Two customers compete for the last unit
- **Preconditions:** Product P has stock 1. Customer A and customer B are both logged in. Cart is empty for both.
- **Steps:**
  1. Customer A and customer B each set quantity to 1.
  2. Both click "Add to cart" at (nearly) the same time.
- **Expected result:** Exactly one add succeeds. The other is rejected with a stock message. Stock is not reduced below 0. Neither cart shows a quantity greater than the available stock.

### TC-26 Repeated click on Add to cart
- **Preconditions:** Product P has stock 20. Cart is empty.
- **Steps:**
  1. Set quantity to 10.
  2. Double-click "Add to cart" quickly.
- **Expected result:** The cart shows a single line for P. The quantity is either 10 (duplicate submit ignored) or the cumulative behaviour applies (see open questions), but it must never be above 10 unless the cumulative rule allows it, and never above stock.

---

## Open questions

1. **Per add or cumulative?** Does the 1-10 limit apply to one add action, or to the total quantity of the product in the cart? Example: the cart holds 6 units and the customer adds 5. Is that allowed?
2. **Stock vs. limit precedence.** If the requested quantity is above both 10 and the stock, which message is shown? Is the message the same for both reasons?
3. **Boundaries.** Are 1 and 10 inclusive? Confirmed in the working assumptions, but please check.
4. **Non-integer quantities.** Can products be sold in fractional units (for example, 2.5 kg)? If yes, is 0.5 a valid quantity, and does the 1-10 rule apply to fractions?
5. **Zero and empty input.** Is 0 treated as a validation error, or as "remove from cart"? What is the default quantity when the field is empty (1, or blank with an error)?
6. **Stock timing.** Is stock checked only at add time, or also at cart view and checkout? Is stock reserved when added to the cart, or only at order placement?
7. **Stock changes after add.** If stock drops below the quantity already in the cart, what happens? Is the cart item blocked, auto-reduced, or kept until checkout?
8. **Cart edits.** Does the 1-10 rule also apply when the customer changes the quantity inside the cart (not only when adding)?
9. **Partial add.** When the requested quantity is more than stock but stock is greater than 0, is the add rejected entirely, or is the quantity clamped to available stock? The requirement does not say; this document assumes rejection.
10. **Same product definition.** Do product variants (for example, size or colour) count as the same product for the 1-10 limit and the stock check?
11. **Concurrency.** How is a race for the last unit handled? Is first-come-first-served expected, and what message does the losing customer see?
12. **Out-of-stock behaviour.** Is the "Add to cart" button hidden, disabled, or shown and then rejected when stock is 0?
13. **Error messages.** What exact wording and channel (inline, toast, modal) is expected for each rejection? Are there localisation requirements?
14. **Guest customers.** Does the rule apply the same way to guest (not logged-in) customers?
15. **Maximum for B2B or bulk accounts.** Is the 10-unit cap the same for all customer types?
