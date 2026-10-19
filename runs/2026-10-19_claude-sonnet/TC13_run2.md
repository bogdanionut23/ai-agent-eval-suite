# Test Cases: Shipping Cost and Free Shipping Threshold

Requirement: "Shipping costs 15 EUR. Orders of 100 EUR or more ship for free within the country."

## Assumptions (taken literally from the requirement, nothing else invented)
- The threshold is inclusive: 100.00 EUR and above is free.
- Free shipping applies only to domestic orders. Behaviour for international orders is not defined (see open questions).
- Currency is EUR, amounts have 2 decimals.

## Positive test cases

**TC-POS-01: Domestic order below threshold is charged shipping**
- Preconditions: Shipping address is in the home country. Cart is empty.
- Steps: 1. Add items totaling 50.00 EUR. 2. Go to checkout. 3. Select a domestic address.
- Expected: Shipping line shows 15.00 EUR. Order total is 65.00 EUR.

**TC-POS-02: Domestic order above threshold ships free**
- Preconditions: Domestic address. Empty cart.
- Steps: 1. Add items totaling 150.00 EUR. 2. Go to checkout.
- Expected: Shipping shows 0.00 EUR (free). Order total is 150.00 EUR.

**TC-POS-03: Domestic order exactly at threshold ships free**
- Preconditions: Domestic address. Empty cart.
- Steps: 1. Add items totaling exactly 100.00 EUR. 2. Go to checkout.
- Expected: Shipping is free. Total is 100.00 EUR.

**TC-POS-04: Shipping updates when cart crosses the threshold**
- Preconditions: Domestic address, cart with 80.00 EUR.
- Steps: 1. Observe shipping (15.00 EUR). 2. Add an item worth 25.00 EUR (subtotal 105.00 EUR).
- Expected: Shipping changes from 15.00 EUR to 0.00 EUR without a page reload issue; total is 105.00 EUR.

## Boundary test cases

**TC-BND-01: Just below threshold (99.99 EUR)**
- Preconditions: Domestic address.
- Steps: Add items totaling 99.99 EUR; open checkout.
- Expected: Shipping 15.00 EUR. Total 114.99 EUR.

**TC-BND-02: Exactly 100.00 EUR** (see TC-POS-03; run with a single item priced 100.00 EUR)
- Expected: Free shipping.

**TC-BND-03: Just above threshold (100.01 EUR)**
- Preconditions: Domestic address.
- Steps: Add items totaling 100.01 EUR; open checkout.
- Expected: Free shipping. Total 100.01 EUR.

**TC-BND-04: Lowest possible order value (0.01 EUR)**
- Preconditions: A product priced 0.01 EUR exists (if the catalogue allows).
- Steps: Add it; open checkout with domestic address.
- Expected: Shipping 15.00 EUR. Total 15.01 EUR.

**TC-BND-05: Threshold reached by multiple items (e.g. 3 x 33.34 EUR = 100.02 EUR)**
- Steps: Add quantity 3 of a 33.34 EUR item; domestic address.
- Expected: Free shipping. No rounding error changes the result.

**TC-BND-06: Rounding at the boundary (e.g. items summing to 99.995 before rounding, if percentage prices are possible)**
- Expected: Behaviour follows the documented rounding rule (see open question Q4). Result must be consistent between cart, checkout and invoice.

## Negative test cases

**TC-NEG-01: Order value drops below threshold after removing an item**
- Preconditions: Domestic address, cart 120.00 EUR with two items (70.00 and 50.00).
- Steps: Remove the 50.00 EUR item.
- Expected: Subtotal 70.00 EUR, shipping changes from free to 15.00 EUR, total 85.00 EUR.

**TC-NEG-02: Empty cart**
- Steps: Open checkout with an empty cart.
- Expected: No shipping cost shown; checkout is not possible (no 15.00 EUR charge on an empty order).

**TC-NEG-03: Quantity reduced below threshold**
- Preconditions: Domestic address, 2 x 60.00 EUR (120.00 EUR).
- Steps: Reduce quantity to 1.
- Expected: Shipping changes to 15.00 EUR; total 75.00 EUR.

**TC-NEG-04: Invalid quantity / negative value is rejected**
- Steps: Try to set the quantity to -1 or 0 via UI or API.
- Expected: Request rejected; shipping is not computed from a negative subtotal.

**TC-NEG-05: Tampering with shipping cost through the API**
- Steps: Submit an order via API with subtotal 50.00 EUR and shipping 0.00 EUR.
- Expected: Server recalculates shipping and ignores the client value (15.00 EUR charged), or rejects the request.

**TC-NEG-06: Non-domestic address (behaviour only partly defined)**
- Preconditions: Cart 150.00 EUR, address in another country.
- Steps: Go to checkout.
- Expected: Free shipping must NOT be applied automatically on the strength of this requirement alone; actual result to be confirmed (see Q1). Test is blocked until clarified.

**TC-NEG-07: Address changes from domestic to foreign after cart is built**
- Steps: Cart 150.00 EUR with domestic address (free). Change address to a foreign one.
- Expected: Shipping is recalculated according to the rule for non-domestic orders (see Q1).

## Open questions (not assumed, to be answered by the product owner)

1. What is the shipping cost for orders outside the country? The requirement says 15 EUR shipping and free only "within the country", but does not state whether 15 EUR applies abroad or whether there are other rates, or no delivery at all.
2. What counts toward the 100 EUR: the subtotal before or after discounts and vouchers? Does it include VAT? Are gift cards, previous refunds or shipping itself part of the amount?
3. Is the 15 EUR flat per order, or per parcel/item/shipment? What if an order is split into several shipments?
4. What is the rounding rule and the decimal precision for amounts (e.g. 99.995)?
5. How is "within the country" determined: shipping address, billing address, or seller country? Which country is "the country" (single-market shop or multi-country)? Are islands, territories or PO boxes included?
6. Are there product exclusions (e.g. digital goods, heavy or oversize items, gift cards) with different shipping rules?
7. Does a discount code applied after the threshold is reached, which brings the total below 100 EUR, remove free shipping?
8. Is shipping cost subject to VAT, and is the 15 EUR gross or net?
9. Do other shipping methods exist (express, pickup) and does the free rule apply to them?
10. Is the threshold configurable, and should the rule apply to other currencies if the shop supports them?
11. How must the shipping cost be displayed (cart, checkout, invoice, emails), and should the user see how much more is needed to reach free shipping?
12. How are returns/partial refunds handled when a refund brings a free-shipping order below 100 EUR?
