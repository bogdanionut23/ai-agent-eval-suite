# TC13 - Test cases: shipping cost and free-shipping threshold

Requirement: "Shipping costs 15 EUR. Orders of 100 EUR or more ship for free within the country."

## Assumptions (kept to the literal text only)
- Shipping fee is 15.00 EUR for orders below 100.00 EUR.
- Free shipping applies when order value >= 100.00 EUR AND the destination is inside the country.
- Behaviour for international orders is NOT defined (see open questions), so tests for it only check that the behaviour is clarified, not a specific outcome.
- "Order value" is assumed to be the amount compared to the threshold; which exact amount (before/after discount, VAT, etc.) is an open question. Test cases below use plain item subtotal with no discounts or taxes unless stated.

## Positive test cases

### TC13-01 - Domestic order below threshold pays shipping
- Preconditions: Shipping address is in the home country. Cart is empty. No discounts or vouchers.
- Steps: 1. Add items totalling 50.00 EUR. 2. Go to checkout. 3. Review the shipping line and total.
- Expected: Shipping = 15.00 EUR. Total = 65.00 EUR.

### TC13-02 - Domestic order well above threshold ships free
- Preconditions: Home-country address. No discounts.
- Steps: 1. Add items totalling 250.00 EUR. 2. Go to checkout. 3. Review shipping and total.
- Expected: Shipping = 0.00 EUR (shown as free). Total = 250.00 EUR.

### TC13-03 - Domestic order exactly at threshold ships free
- Preconditions: Home-country address. No discounts.
- Steps: 1. Add items totalling exactly 100.00 EUR. 2. Go to checkout.
- Expected: Shipping = 0.00 EUR. Total = 100.00 EUR.

### TC13-04 - Shipping cost updates when cart crosses the threshold (upwards)
- Preconditions: Home-country address. Cart contains 80.00 EUR of items; shipping shows 15.00 EUR.
- Steps: 1. Add an item of 25.00 EUR (subtotal 105.00 EUR). 2. Observe the shipping line without leaving checkout.
- Expected: Shipping changes to 0.00 EUR; total = 105.00 EUR.

### TC13-05 - Shipping cost updates when cart crosses the threshold (downwards)
- Preconditions: Home-country address. Cart contains 110.00 EUR; shipping shows 0.00 EUR.
- Steps: 1. Remove an item of 20.00 EUR (subtotal 90.00 EUR). 2. Observe shipping and total.
- Expected: Shipping changes to 15.00 EUR; total = 105.00 EUR.

## Boundary test cases

### TC13-06 - Just below threshold (99.99 EUR)
- Preconditions: Home-country address. No discounts.
- Steps: 1. Build a cart of 99.99 EUR. 2. Go to checkout.
- Expected: Shipping = 15.00 EUR. Total = 114.99 EUR.

### TC13-07 - Just above threshold (100.01 EUR)
- Preconditions: Home-country address. No discounts.
- Steps: 1. Build a cart of 100.01 EUR. 2. Go to checkout.
- Expected: Shipping = 0.00 EUR. Total = 100.01 EUR.

### TC13-08 - Smallest possible order value
- Preconditions: Home-country address. A 0.01 EUR item exists (or the cheapest available item is used).
- Steps: 1. Add the cheapest item. 2. Go to checkout.
- Expected: Shipping = 15.00 EUR; total = item price + 15.00 EUR.

### TC13-09 - Rounding at the threshold
- Preconditions: Home-country address. Prices with 3+ decimals or per-unit prices that produce rounding (e.g. 3 x 33.333 EUR) can be configured.
- Steps: 1. Build a cart whose mathematical sum is 99.999 EUR / displayed 100.00 EUR. 2. Go to checkout.
- Expected: Threshold comparison uses the same value that is displayed and charged (no mismatch between shown subtotal and shipping decision). Record actual behaviour; see open question on rounding.

### TC13-10 - Very large order
- Preconditions: Home-country address.
- Steps: 1. Build a cart of 99,999.99 EUR (or system maximum). 2. Go to checkout.
- Expected: Shipping = 0.00 EUR; no overflow or formatting errors.

## Negative test cases

### TC13-11 - Empty cart
- Preconditions: Cart empty.
- Steps: 1. Open the cart/checkout page.
- Expected: No shipping cost is charged or displayed; checkout cannot be completed with an empty cart.

### TC13-12 - Free shipping is not applied to a sub-threshold order via UI tampering
- Preconditions: Home-country address. Cart of 50.00 EUR.
- Steps: 1. Using browser dev tools or an API client, modify the request to send shipping = 0 or a forged subtotal of 100.00 EUR. 2. Submit the order.
- Expected: Server recalculates; shipping = 15.00 EUR; the forged value is ignored or the request is rejected.

### TC13-13 - Negative or zero quantity cannot lower the cart to cross the rule incorrectly
- Preconditions: Cart with 120.00 EUR of items.
- Steps: 1. Attempt to set an item quantity to -1 or 0 via UI/API.
- Expected: Invalid quantity is rejected (or item removed for 0); shipping is recalculated on the valid cart only.

### TC13-14 - Currency other than EUR
- Preconditions: Shop supports (or can be forced to) another display currency, if applicable.
- Steps: 1. Switch currency. 2. Add items equal to 100 EUR in the other currency. 3. Check shipping.
- Expected: Threshold is evaluated against EUR value as defined by the business (see open question); behaviour is consistent and documented. Skip if only EUR is supported.

### TC13-15 - Order outside the country
- Preconditions: Shipping address is in another country. Cart of 150.00 EUR.
- Steps: 1. Enter the foreign address. 2. Go to checkout.
- Expected: Free shipping is NOT automatically granted solely by the 100 EUR rule, since it applies "within the country". Actual fee/availability must follow the business answer to open question 1. Record actual result.

### TC13-16 - Address changed from domestic to foreign after cart reaches threshold
- Preconditions: Home-country address; cart 150.00 EUR; shipping 0.00 EUR.
- Steps: 1. Change the shipping country to a foreign one.
- Expected: Shipping is recalculated and no longer shows free shipping under the domestic rule; result follows business decision.

## Open questions (not assumed above)
1. International orders: the requirement says free shipping applies "within the country" but does not define the cost (or availability) of shipping abroad. Is it 15 EUR, different rates by country, or not offered?
2. Which amount is compared with the 100 EUR threshold: items only, or after discounts/vouchers, before or after VAT, including gift cards? Does shipping itself count toward the threshold?
3. Which country is "the country" (the shop's home country)? How is it determined: shipping address, billing address, or customer account?
4. Rounding: how are amounts with more than two decimals handled, and is the threshold compared before or after rounding?
5. Do discounts applied after reaching 100 EUR (e.g. a voucher bringing the total to 95 EUR) remove free shipping?
6. Do all products qualify (e.g. digital goods, oversized/heavy items, marketplace sellers, excluded categories)? Is shipping charged for digital-only orders?
7. Is the 15 EUR flat per order, or per parcel/item/shipment when an order is split into multiple parcels?
8. Are there shipping methods (express, pickup, delivery to parcel locker) and do they have different prices? Does free shipping apply to the standard method only?
9. Is the 15 EUR inclusive or exclusive of VAT, and how is it displayed to the customer?
10. How should the cost be displayed (e.g. "Free" vs "0.00 EUR"), and should the customer be informed how much more they need to spend to get free shipping?
11. Multi-currency: if other currencies are supported, is the threshold converted, and at what exchange rate?
12. Partial returns/cancellations: if an order of 105 EUR ships free and the customer returns a 20 EUR item, is the shipping fee retroactively charged?
13. Which order statuses and channels (web, app, phone, B2B) does the rule apply to?
