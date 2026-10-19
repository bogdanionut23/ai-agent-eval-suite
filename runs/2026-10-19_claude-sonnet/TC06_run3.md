# TC06 - Test cases: cart quantity (1-10) limited by available stock

Requirement: A customer can add between 1 and 10 units of the same product to the cart. The quantity cannot exceed the stock available at that moment.

Notation: S = stock available at the moment of the action. Effective maximum allowed = min(10, S). Where behaviour is not defined by the requirement, the expected result refers to an open question (OQ) instead of inventing behaviour.

## Test cases

### Positive

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC06-01 | Add a single unit (lower boundary) | Product P, S = 50. Cart empty. | 1. Open P. 2. Set quantity 1. 3. Click Add to cart. | P is in the cart with quantity 1. Add is accepted. |
| TC06-02 | Add a mid-range quantity | P, S = 50. Cart empty. | 1. Set quantity 5. 2. Add to cart. | Cart contains P x 5. |
| TC06-03 | Add exactly 10 units (upper boundary) | P, S = 50. Cart empty. | 1. Set quantity 10. 2. Add to cart. | Cart contains P x 10. Accepted. |
| TC06-04 | Quantity equals stock when stock is below 10 | P, S = 4. Cart empty. | 1. Set quantity 4. 2. Add to cart. | Cart contains P x 4. Accepted (quantity = S). |
| TC06-05 | Quantity equals stock when stock is exactly 10 | P, S = 10. Cart empty. | 1. Set quantity 10. 2. Add. | Cart contains P x 10. Accepted. |
| TC06-06 | Stock of exactly 1 | P, S = 1. Cart empty. | 1. Set quantity 1. 2. Add. | Cart contains P x 1. Accepted. |
| TC06-07 | Different products are counted separately | P1 and P2, each S = 50. Cart has P1 x 10. | 1. Add P2 x 10. | P2 x 10 is accepted. The limit of 10 applies per product, not to the whole cart. |
| TC06-08 | Cumulative add up to the limit | P, S = 50. Cart has P x 6. | 1. Add P x 4 (same product again). | Cart has P x 10 (assuming repeat adds accumulate; see OQ-3). |

### Negative

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC06-09 | Quantity 0 | P, S = 50. | 1. Set quantity 0. 2. Add. | Rejected. Nothing added. A validation message is shown (wording: OQ-6). |
| TC06-10 | Negative quantity | P, S = 50. | 1. Enter -1. 2. Add. | Rejected. Nothing added. |
| TC06-11 | Quantity 11 (above max, stock sufficient) | P, S = 50. Cart empty. | 1. Enter 11. 2. Add. | Rejected (or not accepted as entered; see OQ-5). Cart does not contain 11. |
| TC06-12 | Large quantity far above max | P, S = 500. | 1. Enter 1000. 2. Add. | Rejected. Cart unchanged. |
| TC06-13 | Quantity exceeds stock, stock below 10 | P, S = 4. | 1. Enter 5. 2. Add. | Rejected. Cart unchanged. Message indicates insufficient stock. |
| TC06-14 | Quantity exceeds stock by one | P, S = 7. | 1. Enter 8. 2. Add. | Rejected. Cart unchanged. |
| TC06-15 | Product out of stock | P, S = 0. | 1. Try to add 1 unit. | Rejected. Cart unchanged. (UI treatment such as disabled button: OQ-7.) |
| TC06-16 | Non-numeric input | P, S = 50. | 1. Enter "abc". 2. Add. | Rejected. Nothing added. No server error. |
| TC06-17 | Decimal input | P, S = 50. | 1. Enter 2.5. 2. Add. | Rejected or not accepted as a non-integer (OQ-8). Cart does not contain 2.5 units. |
| TC06-18 | Empty quantity field | P, S = 50. | 1. Clear the quantity field. 2. Add. | Rejected with a validation message. Nothing added. |
| TC06-19 | Cumulative add exceeds 10 | P, S = 50. Cart has P x 8. | 1. Add P x 3. | Total would be 11. Not allowed to reach 11 (outcome for the whole request vs. partial add: OQ-3). |
| TC06-20 | Cumulative add exceeds stock | P, S = 5. Cart has P x 3. | 1. Add P x 3. | Total 6 > S. Not allowed to reach 6 (OQ-3, OQ-4). |
| TC06-21 | Stock drops below requested quantity before add | P, S = 8. Customer is on the product page with quantity 6 selected. | 1. In another session/admin, reduce S to 5. 2. Click Add in the first session. | Add is rejected (stock is checked at the moment of the action). Cart does not contain 6. |
| TC06-22 | Race: last units bought by another customer | P, S = 2. Customers A and B both view P. | 1. A adds 2 and completes the add. 2. B adds 1. | B is rejected because available stock is now 0 (or reduced by A's reservation, depending on OQ-1). |
| TC06-23 | Concurrent adds for the last unit | P, S = 1. Two customers send the add request at the same time. | 1. Both click Add for 1 unit simultaneously. | At most one add succeeds in a way that is consistent with S = 1 (subject to OQ-1). No overselling. |
| TC06-24 | Bypass UI validation via direct request | P, S = 50. | 1. Send the add-to-cart request (API) with quantity 11 or 0 or -3. | Server rejects. Limits are enforced server-side, not only in the UI. |
| TC06-25 | Direct request exceeding stock | P, S = 4. | 1. Send the add request via API with quantity 5. | Server rejects. Cart unchanged. |

### Boundary summary (value analysis)

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC06-26 | Lower boundary sweep: 0, 1, 2 | P, S = 50. | Attempt to add 0, then 1, then 2 (clean cart each time). | 0 rejected, 1 accepted, 2 accepted. |
| TC06-27 | Upper boundary sweep: 9, 10, 11 | P, S = 50. | Attempt to add 9, 10, 11 (clean cart each time). | 9 accepted, 10 accepted, 11 rejected. |
| TC06-28 | Stock boundary sweep, S below 10: S-1, S, S+1 | P, S = 6. | Attempt to add 5, 6, 7 (clean cart each time). | 5 accepted, 6 accepted, 7 rejected. |
| TC06-29 | Stock boundary sweep, S above 10: 10, 11 | P, S = 11. | Attempt to add 10, then 11. | 10 accepted. 11 rejected (the cap of 10 applies although stock allows 11). |
| TC06-30 | Stock boundary at S = 10 and S = 11 | P with S = 10, then S = 11. | Attempt to add 10 and 11 each time. | S = 10: 10 accepted, 11 rejected. S = 11: 10 accepted, 11 rejected. |
| TC06-31 | Stock changes during UI interaction (increase) | P, S = 3. Customer sees max 3. | 1. Stock is replenished to 20. 2. Customer adds 3. | Accepted. Adding 4 to 10 would be accepted after refresh (UI refresh behaviour: OQ-9). |
| TC06-32 | Quantity adjustment in cart (if supported) | P, S = 6. Cart has P x 2. | 1. Change quantity in cart to 6. 2. Change to 7. 3. Change to 11. 4. Change to 0. | 6 accepted. 7 and 11 rejected. 0 behaviour depends on OQ-10 (remove vs. reject). |

## Open questions (not assumed in the tests above)

1. Stock reservation: Is stock "available" counted as total on-hand stock, or on-hand minus units in other customers' carts (reservation)? Does adding to a cart reserve stock, and for how long?
2. Stock definition: Does "stock available at that moment" include inbound or backordered stock, or only physically available/sellable stock? Are variants (size, colour) separate products for the 10-unit limit?
3. Repeat adds: If P is already in the cart, does the limit of 10 apply to the cart total for that product (cumulative) or only to each single add action? If the new total would exceed the limit, is the whole request rejected, or is the quantity clamped to the allowed maximum?
4. Is the stock check applied to cart quantity cumulative with what is already in the cart, or only to the quantity being added?
5. Behaviour when quantity exceeds the limit: Does the UI block input (for example max attribute or a stepper that stops at the max), show an error, or silently clamp to the maximum? What should the API return (status code, error code)?
6. Exact wording, language and placement of error messages for each case (below 1, above 10, above stock, out of stock).
7. Out-of-stock presentation: Is the Add button disabled, hidden, replaced by "notify me", or enabled with an error on click?
8. Are non-integer quantities (2.5) and non-numeric input to be rejected, rounded or truncated? Is whitespace or a leading zero ("05") allowed?
9. If stock changes while the customer has the page open (or the item is in the cart), when is it re-validated: at add, at cart view, at checkout, or all three? What happens to a cart item whose quantity now exceeds stock (auto-reduce, warning, block checkout)?
10. Cart quantity edits and removal: Do the same rules apply when editing the quantity in the cart? Is setting the quantity to 0 treated as removal?
11. Guest vs. logged-in customers and multiple devices or sessions: is the limit applied to the combined cart of the same customer?
12. Is the limit of 10 configurable per product (for example, lower limits for restricted items)? Does any customer type (wholesale, VIP) have a different limit?
13. Concurrency policy: when two customers compete for the last units, who wins and is the outcome guaranteed (no overselling) at add time or only at checkout?
14. Is stock shown to the customer (for example "only 3 left")? If so, must it match the enforced limit exactly?
15. Are there performance or rate-limiting expectations for repeated add requests?
