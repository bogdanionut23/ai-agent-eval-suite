# Test cases: Shipping cost and free-shipping threshold

Requirement: "Shipping costs 15 EUR. Orders of 100 EUR or more ship for free within the country."

## Assumptions (only what the text states)
- Standard shipping fee = 15.00 EUR.
- Free shipping applies when order value >= 100.00 EUR AND the destination is within the country.
- What counts as "order value" is not defined (see open questions). Unless stated otherwise, the test cases below use the cart merchandise subtotal, with no discounts or taxes in play, so they do not depend on the unresolved points.
- Expected results for scenarios the requirement does not define (international orders, rounding, discounts) are deliberately marked "TBD" and linked to open questions, not invented.

## Positive test cases

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC13-01 | Domestic order below threshold pays shipping | Customer logged in; delivery address in the home country; cart subtotal 50.00 EUR | 1. Go to checkout. 2. Review the order summary. | Shipping shown as 15.00 EUR; total = 65.00 EUR. |
| TC13-02 | Domestic order well above threshold ships free | Domestic address; cart subtotal 250.00 EUR | 1. Go to checkout. 2. Review the order summary. | Shipping shown as 0.00 EUR (free); total = 250.00 EUR. |
| TC13-03 | Domestic order exactly at threshold ships free | Domestic address; cart subtotal exactly 100.00 EUR | 1. Go to checkout. 2. Review the order summary. | Shipping is free (0.00 EUR); total = 100.00 EUR ("100 EUR or more" is inclusive). |
| TC13-04 | Placed order stores the shipping charge | As TC13-01 | 1. Complete the purchase. 2. Open the order confirmation, confirmation email and order history. | Shipping of 15.00 EUR and total 65.00 EUR are identical in all three places. |
| TC13-05 | Placed free-shipping order stores no charge | As TC13-02 | 1. Complete the purchase. 2. Open confirmation, email and order history. | Shipping 0.00 EUR or marked "free" consistently everywhere; total 250.00 EUR. |

## Boundary test cases

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC13-06 | Just below threshold (99.99) | Domestic address; subtotal 99.99 EUR | Go to checkout; review summary. | Shipping 15.00 EUR; total 114.99 EUR. |
| TC13-07 | Just above threshold (100.01) | Domestic address; subtotal 100.01 EUR | Go to checkout; review summary. | Shipping free; total 100.01 EUR. |
| TC13-08 | Lowest possible order value | Domestic address; cheapest item in cart (e.g. 0.01 EUR if allowed) | Go to checkout; review summary. | Shipping 15.00 EUR (fee is not capped by item price); no negative or zero total. |
| TC13-09 | Crossing threshold by adding an item | Domestic address; cart 90.00 EUR showing 15.00 EUR shipping | 1. Add an item of 10.00 EUR. 2. Review the cart/summary. | Subtotal 100.00 EUR; shipping changes from 15.00 EUR to free immediately, total recalculated to 100.00 EUR. |
| TC13-10 | Dropping below threshold by removing an item | Domestic address; cart 100.00 EUR with free shipping | 1. Remove an item worth 0.01 EUR or more so subtotal is below 100.00. 2. Review the summary. | Shipping switches to 15.00 EUR; total recalculated. |
| TC13-11 | Quantity change crossing threshold | Domestic address; one item at 40.00 EUR, quantity 2 (80.00) | Increase quantity to 3 (120.00). | Shipping changes from 15.00 EUR to free. |
| TC13-12 | Empty cart | Domestic address; empty cart | Open cart/checkout. | No shipping charge shown and checkout not possible (or per product rules); 15.00 EUR is not charged on an empty cart. |
| TC13-13 | Very large order | Domestic address; subtotal 1,000,000.00 EUR (or the system maximum) | Go to checkout. | Shipping free; no overflow or formatting error. |
| TC13-14 | Rounding at the threshold | Domestic address; items priced so the subtotal is 99.995 EUR before rounding (if sub-cent prices or percent calculations exist) | Go to checkout. | Result follows the system rounding rule: TBD, see OQ-05. |

## Negative test cases

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC13-15 | Order above threshold shipped abroad | Delivery address in another country; subtotal 150.00 EUR | Go to checkout. | The "free" rule must NOT apply on the basis of the stated text; shipping amount for abroad is TBD (see OQ-01). Verify at minimum that the free-shipping message is not shown incorrectly. |
| TC13-16 | Order below threshold shipped abroad | Foreign address; subtotal 50.00 EUR | Go to checkout. | Shipping amount TBD (OQ-01); verify a clear, defined price is shown or shipping is refused with a message. |
| TC13-17 | Change country from domestic to foreign at checkout | Domestic address; subtotal 150.00 EUR with free shipping | Change the delivery address to a foreign country. | Shipping is recalculated; free shipping is withdrawn or replaced per the foreign rule (OQ-01); total updates. |
| TC13-18 | Change country from foreign to domestic at checkout | Foreign address; subtotal 150.00 EUR | Change delivery address to a domestic one. | Shipping becomes free; total updates to 150.00 EUR. |
| TC13-19 | Tampering with the shipping value client-side | Domestic address; subtotal 50.00 EUR | Intercept the checkout request and set shipping to 0 (or the subtotal to 100). | Server recalculates; order is rejected or charged 15.00 EUR; client values are not trusted. |
| TC13-20 | Negative or zero item quantities/prices | Domestic address | Attempt to set quantity to 0 or -1, or submit a negative price via the API. | Request rejected with a validation error; shipping not computed from invalid data. |
| TC13-21 | Shipping not shown for missing address | No delivery address entered; subtotal 50.00 EUR | Open checkout before entering an address. | Behaviour TBD (OQ-02): either shipping is hidden until the address is known, or a default is shown and clearly marked as an estimate. Must not silently assume free shipping. |
| TC13-22 | Non-EUR currency display | Shop supports another currency (if applicable); subtotal equals 100 EUR equivalent | Go to checkout. | Threshold and fee are evaluated per OQ-06; no off-by-rounding error from currency conversion. |

## Discount, tax and other interactions (depend on open questions)

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC13-23 | Discount code drops subtotal below threshold | Domestic address; cart 105.00 EUR; code giving 10 EUR off | Apply the code. | TBD (OQ-03): shipping is 15.00 EUR if the threshold is checked after discount, free if checked before. Result must be consistent in cart, checkout and order. |
| TC13-24 | Discount code with order still above threshold | Domestic address; cart 150.00 EUR; code giving 10 EUR off | Apply the code. | Subtotal 140.00 EUR; shipping free. |
| TC13-25 | Free-shipping threshold with tax-inclusive pricing | Domestic address; items 100.00 EUR including VAT | Go to checkout. | TBD (OQ-04): whether the 100 EUR is measured including or excluding tax. |
| TC13-26 | Gift card / voucher payment | Domestic address; cart 100.00 EUR paid fully by gift card | Complete checkout. | Shipping decision unaffected by payment method (confirm in OQ-03). |
| TC13-27 | Order with a virtual / non-shippable item only | Domestic address; cart has only a digital product worth 50.00 EUR | Go to checkout. | TBD (OQ-07): no shipping charge expected for non-physical goods; confirm. |
| TC13-28 | Mixed cart: physical + digital | Domestic address; physical item 60.00 EUR + digital item 50.00 EUR | Go to checkout. | TBD (OQ-07): whether digital value counts toward the threshold. |
| TC13-29 | Refund/return of an item after free shipping | Order placed at 100.00 EUR with free shipping; one item then returned so remainder < 100 EUR | Process the return. | TBD (OQ-08): whether the 15.00 EUR shipping is deducted from the refund. |
| TC13-30 | Cancellation of an order that paid shipping | Order of 50.00 EUR + 15.00 EUR shipping, not yet shipped | Cancel the order. | TBD (OQ-08): whether shipping is refunded in full; at minimum no inconsistent amounts. |

## Display / formatting checks

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC13-31 | Shipping amount formatting | Domestic address; subtotal 50.00 EUR | View cart, checkout, confirmation, email, invoice. | Shown as 15.00 EUR (locale-appropriate format, e.g. 15,00 EUR) with the correct currency symbol everywhere. |
| TC13-32 | Free shipping label | Domestic address; subtotal 120.00 EUR | View cart and checkout. | Shows "free" or 0.00 EUR (wording TBD), not blank or a hidden line that could look like an error. |
| TC13-33 | Progress hint toward free shipping (only if the product has it) | Domestic address; subtotal 70.00 EUR | View cart. | If a hint exists, it says 30.00 EUR remaining; otherwise not applicable (requirement does not mention it). |
| TC13-34 | Totals arithmetic | Several carts: 0.01, 50, 99.99, 100, 100.01, 500 EUR | Check summary for each. | Total = subtotal + shipping in every case, to the cent. |

## Open questions (no behaviour was invented for these)

| ID | Question | Why it matters |
|---|---|---|
| OQ-01 | What is the shipping cost for orders outside the country (any amount)? Is there international shipping at all? "Free within the country" implies a different rule abroad but none is given. | TC13-15, 16, 17. |
| OQ-02 | When is shipping calculated: before or after the address is known? What is shown in the cart when no address exists yet? | TC13-21. |
| OQ-03 | Is the 100 EUR threshold checked before or after discounts, coupons, vouchers and gift cards? | TC13-23, 26. |
| OQ-04 | Is the threshold measured on prices including or excluding VAT/tax? Are shipping fees themselves tax-inclusive? | TC13-25. |
| OQ-05 | What is the rounding rule and the number of decimals for amounts (relevant for percentage discounts, currency conversion)? | TC13-14. |
| OQ-06 | Is EUR the only currency? If not, is the threshold converted, and at what rate and when? | TC13-22. |
| OQ-07 | Do digital or non-shippable items count toward the threshold, and is shipping charged on digital-only orders? | TC13-27, 28. |
| OQ-08 | On returns, partial refunds and cancellations, is shipping refunded, and does the threshold get re-evaluated afterwards? | TC13-29, 30. |
| OQ-09 | Does the 15 EUR fee depend on weight, size, delivery speed or region, or is it a single flat rate for every order below 100 EUR? | Affects all cases below 100 EUR. |
| OQ-10 | What does "within the country" mean exactly: shipping address, billing address, or seller location? What about territories, islands, PO boxes and parcel lockers? | TC13-15 to 18. |
| OQ-11 | Does the rule apply to all customers and channels (web, app, B2B, marketplace), and can it be overridden by promotions or customer groups? | Scope of testing. |
| OQ-12 | Should the free-shipping state be displayed anywhere (banner, product page, cart hint), and what is the exact wording? Which languages? | TC13-32, 33. |
| OQ-13 | Is the maximum or minimum order value limited (e.g., minimum order, maximum cart value)? | TC13-08, 13. |
| OQ-14 | Which country is "the country" (single-market shop or configurable per store)? | Test data setup. |

## Coverage summary
- Positive: TC13-01 to 05
- Boundary: TC13-06 to 14 (99.99 / 100.00 / 100.01, threshold crossing both ways, empty and very large carts, rounding)
- Negative: TC13-15 to 22 (foreign destination, address changes, tampering, invalid input, missing address, currency)
- Interaction and lifecycle: TC13-23 to 30 (blocked or partly blocked until open questions are answered)
- Display: TC13-31 to 34
