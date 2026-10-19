# TC13 - Test cases: shipping cost and free shipping threshold

**Requirement:** Shipping costs 15 EUR. Orders of 100 EUR or more ship for free within the country.

**Assumptions used for the test cases below (to be confirmed, see Open questions):**
- "Within the country" means the shipping destination is the store's home country (called DOMESTIC below).
- The threshold is checked against the merchandise subtotal, after discounts, before shipping. Tax handling is not specified.
- "100 EUR or more" is inclusive: exactly 100.00 EUR qualifies.
- Shipping is charged once per order, not per item.

## Positive scenarios

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC13-P01 | Order below threshold pays 15 EUR shipping | Customer in DOMESTIC country; cart subtotal 50.00 EUR; shipping is available | 1. Add items totalling 50.00 EUR. 2. Go to checkout. 3. Read the shipping line. | Shipping shows 15.00 EUR. Order total = 50.00 + 15.00 = 65.00 EUR. |
| TC13-P02 | Order at high value ships free | Customer in DOMESTIC country; cart subtotal 250.00 EUR | 1. Add items totalling 250.00 EUR. 2. Go to checkout. 3. Read the shipping line. | Shipping shows 0.00 EUR (free). Order total = 250.00 EUR. |
| TC13-P03 | Shipping recalculates when cart crosses threshold upward | Customer in DOMESTIC country; cart subtotal 80.00 EUR, shipping 15.00 EUR shown | 1. Increase quantity so subtotal is 100.00 EUR. 2. Observe checkout summary. | Shipping changes from 15.00 EUR to 0.00 EUR. Total updates to 100.00 EUR. |
| TC13-P04 | Shipping recalculates when cart drops below threshold | Customer in DOMESTIC country; cart subtotal 120.00 EUR, shipping free | 1. Remove an item so subtotal is 90.00 EUR. 2. Observe checkout summary. | Shipping changes from 0.00 EUR to 15.00 EUR. Total updates to 105.00 EUR. |
| TC13-P05 | Shipping is charged once per order, not per item | Customer in DOMESTIC country; cart has 3 different items totalling 60.00 EUR | 1. Add the 3 items. 2. Go to checkout. | Shipping is 15.00 EUR once, not 45.00 EUR. |
| TC13-P06 | Free shipping shown consistently across checkout artefacts | Customer in DOMESTIC country; subtotal 100.00 EUR, order placed | 1. Place the order. 2. Compare the order confirmation page, confirmation email and invoice. | All three show shipping 0.00 EUR and total 100.00 EUR. |
| TC13-P07 | 15 EUR shipping shown consistently across checkout artefacts | Customer in DOMESTIC country; subtotal 40.00 EUR, order placed | 1. Place the order. 2. Compare confirmation page, email and invoice. | All three show shipping 15.00 EUR and total 55.00 EUR. |

## Negative scenarios

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC13-N01 | Subtotal just below threshold still pays shipping | Customer in DOMESTIC country; subtotal 99.99 EUR | 1. Add items totalling 99.99 EUR. 2. Go to checkout. | Shipping is 15.00 EUR. Free shipping is not granted. |
| TC13-N02 | Shipping fee is not counted toward the threshold | Customer in DOMESTIC country; subtotal 85.00 EUR, shipping 15.00 EUR | 1. Add items totalling 85.00 EUR. 2. Go to checkout. | Shipping is 15.00 EUR. The 100 EUR check uses 85.00, not the 100.00 total including shipping. |
| TC13-N03 | Order over threshold to another country does not ship free | Customer in a non-DOMESTIC country; subtotal 150.00 EUR | 1. Add items totalling 150.00 EUR. 2. Set the shipping address to a non-DOMESTIC country. 3. Go to checkout. | Shipping is NOT 0.00 EUR. Actual amount is blocked by Open question 1. Do not mark as pass until it is defined. |
| TC13-N04 | Order under threshold to another country | Customer in a non-DOMESTIC country; subtotal 30.00 EUR | 1. Add items totalling 30.00 EUR. 2. Set a non-DOMESTIC shipping address. 3. Go to checkout. | Shipping is not 15.00 EUR by default. Actual amount is blocked by Open question 1. |
| TC13-N05 | Empty cart cannot be checked out with a shipping charge | Customer in DOMESTIC country; cart is empty (0.00 EUR) | 1. Open checkout with an empty cart. | Checkout is blocked, or no shipping fee is applied. A 15.00 EUR fee on a 0.00 EUR order must not occur. Behaviour is partly assumed, see Open question 8. |
| TC13-N06 | Removing items below threshold does not keep free shipping | Customer in DOMESTIC country; cart subtotal 100.00 EUR, shipping free, checkout open | 1. Remove an item worth 0.01 EUR so subtotal is 99.99 EUR. 2. Observe summary. | Shipping changes to 15.00 EUR. Free shipping is not kept from the earlier state. |
| TC13-N07 | Shipping not free when threshold is met only by a discount that is later reversed | Customer in DOMESTIC country; subtotal 110.00 EUR before a 15.00 EUR coupon, which is applied and then removed | 1. Add items totalling 110.00 EUR. 2. Apply a 15.00 EUR discount code. 3. Remove the code. | Shipping is free while the code is applied only if the threshold is pre-discount (Open question 2). After removal, shipping must match the rule for the resulting subtotal. Final state must be consistent. |

## Boundary scenarios

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC13-B01 | Subtotal exactly at threshold (100.00 EUR) ships free | Customer in DOMESTIC country; subtotal 100.00 EUR | 1. Add items totalling 100.00 EUR. 2. Go to checkout. | Shipping is 0.00 EUR (free). Inclusive boundary assumed (Open question 4). |
| TC13-B02 | One cent below threshold (99.99 EUR) pays shipping | Customer in DOMESTIC country; subtotal 99.99 EUR | 1. Add items totalling 99.99 EUR. 2. Go to checkout. | Shipping is 15.00 EUR. (Same as TC13-N01, kept at boundary level.) |
| TC13-B03 | One cent above threshold (100.01 EUR) ships free | Customer in DOMESTIC country; subtotal 100.01 EUR | 1. Add items totalling 100.01 EUR. 2. Go to checkout. | Shipping is 0.00 EUR (free). |
| TC13-B04 | Zero subtotal | Customer in DOMESTIC country; subtotal 0.00 EUR | 1. Attempt checkout with 0.00 EUR subtotal. | Same as TC13-N05: no 15.00 EUR charge on a zero order. Behaviour assumed. |
| TC13-B05 | Threshold is checked on the subtotal, not on the number of items | Customer in DOMESTIC country; one item priced 100.00 EUR | 1. Add 1 item priced 100.00 EUR. 2. Go to checkout. | Shipping is 0.00 EUR. The rule depends on value, not count. |
| TC13-B06 | Many small items reach threshold | Customer in DOMESTIC country; 20 items at 5.00 EUR each (subtotal 100.00 EUR) | 1. Add 20 items at 5.00 EUR each. 2. Go to checkout. | Shipping is 0.00 EUR. |
| TC13-B07 | Quantity change exactly on the threshold | Customer in DOMESTIC country; subtotal 95.00 EUR, item unit price 5.00 EUR | 1. Increase quantity by 1 so subtotal is 100.00 EUR. | Shipping goes from 15.00 EUR to 0.00 EUR. |
| TC13-B08 | Large order with high value | Customer in DOMESTIC country; subtotal 99999.99 EUR | 1. Add items totalling 99999.99 EUR. 2. Go to checkout. | Shipping is 0.00 EUR. No overflow or formatting error. |

## Open questions

1. **Destination outside the country:** The requirement says free shipping applies "within the country" but gives no rule for other countries. What is the shipping fee for cross-border orders (15 EUR, a different fee, or not offered)? Does the threshold apply there? (Affects TC13-N03, TC13-N04.)
2. **What "country" means:** Is it the shipping destination country or the seller's country? Are there several domestic regions (for example islands or territories) that are excluded?
3. **Threshold basis:** Is the 100 EUR measured on the subtotal before or after discounts and coupons? Before or after VAT (gross or net)? Does it include gift cards or store credit? (Affects TC13-N07 and TC13-P01/P02 under discounts.)
4. **Inclusive or exclusive:** Is an order of exactly 100.00 EUR free? The text says "100 EUR or more", which implies inclusive. This is assumed in TC13-B01.
5. **Per order or per parcel:** If an order is split into several parcels or shipments, is 15 EUR charged for each parcel? Does the threshold apply to each parcel or to the whole order?
6. **Cart changes after checkout starts:** Must shipping be recalculated live when the cart changes? Is the threshold checked again at payment time?
7. **Changes after the order is placed:** If items are removed, returned or partly refunded after payment and the value falls below 100 EUR, is shipping refunded, charged or left as is?
8. **Empty or zero-value orders:** Should checkout be blocked, or should shipping be 0 EUR? Is any fee charged on orders made only of free or gift items? (Affects TC13-N05, TC13-B04.)
9. **Exclusions:** Do digital goods, gift cards, or other non-shippable items count toward the threshold? Do they have a shipping fee at all? Are there any product categories excluded from free shipping?
10. **Currency:** Is the store only in EUR? If it sells in other currencies, what is the threshold and fee in each?
11. **Shipping methods:** Is 15 EUR the only option, or are there express, pickup, or other methods with different prices and thresholds?
12. **Promotions:** Can a promotion or coupon waive or change shipping on its own, outside this rule? If so, which rule wins?
13. **Display rules:** How should free shipping be shown: "0.00 EUR", "Free", or hidden? Is the threshold shown to the customer (for example "Add 20 EUR more for free shipping")?
14. **Rounding and tax on the fee:** Is the 15 EUR fee gross (including VAT) or net? Is the amount shown before or after tax?
