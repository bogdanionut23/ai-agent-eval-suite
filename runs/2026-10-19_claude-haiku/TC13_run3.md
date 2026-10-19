# TC13 - Test cases: Shipping cost and free shipping threshold

**Requirement:** Shipping costs 15 EUR. Orders of 100 EUR or more ship for free within the country.

**Scope note:** Test cases below cover only what the requirement states. Where the requirement is silent (cross-border shipping, threshold base, returns, currency, and similar), the case is marked **BLOCKED - see Open question Qn** and no expected behaviour is invented. The open questions are listed at the end.

---

## 1. Positive scenarios

### TC13-01 - Order below threshold is charged 15 EUR shipping
- **Preconditions:** Shop is open. Customer has a delivery address in the country. Cart subtotal is 50.00 EUR (items only).
- **Steps:**
  1. Proceed to checkout with the 50.00 EUR cart.
  2. Enter the in-country delivery address.
  3. Review the order summary.
- **Expected result:** Shipping line shows 15.00 EUR. Order total = 50.00 + 15.00 = 65.00 EUR (before any tax or discount rules defined in Q2/Q4).

### TC13-02 - Order well above threshold ships free
- **Preconditions:** Shop is open. Customer has an in-country delivery address. Cart subtotal is 150.00 EUR.
- **Steps:**
  1. Proceed to checkout.
  2. Enter the in-country delivery address.
  3. Review the order summary.
- **Expected result:** Shipping line shows 0.00 EUR (free). Order total = 150.00 EUR.

### TC13-03 - Multi-item cart reaching threshold ships free
- **Preconditions:** Cart contains three items: 40.00 EUR, 35.00 EUR, 25.00 EUR (subtotal 100.00 EUR). In-country address.
- **Steps:**
  1. Proceed to checkout.
  2. Review the order summary.
- **Expected result:** Shipping is 0.00 EUR. The threshold is evaluated on the combined cart subtotal, not per item.

### TC13-04 - Single low-value item is charged 15 EUR shipping
- **Preconditions:** Cart contains one item at 9.99 EUR. In-country address.
- **Steps:**
  1. Proceed to checkout.
  2. Review the order summary.
- **Expected result:** Shipping is 15.00 EUR. Shipping is not reduced or waived because the item is cheap.

---

## 2. Boundary scenarios

### TC13-05 - Subtotal just below threshold (99.99 EUR) is charged shipping
- **Preconditions:** Cart subtotal is exactly 99.99 EUR. In-country address.
- **Steps:**
  1. Proceed to checkout.
  2. Review the order summary.
- **Expected result:** Shipping is 15.00 EUR.

### TC13-06 - Subtotal exactly at threshold (100.00 EUR) ships free
- **Preconditions:** Cart subtotal is exactly 100.00 EUR. In-country address.
- **Steps:**
  1. Proceed to checkout.
  2. Review the order summary.
- **Expected result:** Shipping is 0.00 EUR. The requirement says "100 EUR or more", so the threshold is inclusive.

### TC13-07 - Subtotal just above threshold (100.01 EUR) ships free
- **Preconditions:** Cart subtotal is exactly 100.01 EUR. In-country address.
- **Steps:**
  1. Proceed to checkout.
  2. Review the order summary.
- **Expected result:** Shipping is 0.00 EUR.

### TC13-08 - Threshold reached by cent-level summation (no floating-point drift)
- **Preconditions:** Cart contains three items at 33.33 EUR each (99.99 EUR) plus one item at 0.01 EUR (total 100.00 EUR). In-country address.
- **Steps:**
  1. Proceed to checkout.
  2. Review the order summary.
- **Expected result:** Shipping is 0.00 EUR. Subtotal is computed exactly to the cent; a result such as 99.99999999 from binary float arithmetic must not block free shipping.

### TC13-09 - Smallest positive order (0.01 EUR) is charged shipping
- **Preconditions:** Cart subtotal is 0.01 EUR. In-country address. (Feasibility of a 0.01 EUR order depends on the shop's minimum-order rule, see Q12.)
- **Steps:**
  1. Proceed to checkout.
  2. Review the order summary.
- **Expected result:** Shipping is 15.00 EUR.

### TC13-10 - Shipping amount is exactly 15.00 EUR at the charged boundary
- **Preconditions:** Cart subtotal is 99.99 EUR. In-country address.
- **Steps:**
  1. Proceed to checkout.
  2. Read the shipping amount in the summary and in the payment total.
- **Expected result:** Shipping is shown and charged as 15.00 EUR in both places, with no rounding to another value.

### TC13-11 - Zero-value order
- **Preconditions:** Cart subtotal is 0.00 EUR (for example, an item priced at 0.00 EUR).
- **Steps:**
  1. Attempt checkout.
- **Expected result:** **BLOCKED - see Q12.** The requirement does not say whether a zero-value order is allowed or whether 15 EUR applies to it.

---

## 3. Negative and edge scenarios

### TC13-12 - Cart drops below threshold after item removal
- **Preconditions:** Cart subtotal is 120.00 EUR (free shipping shown). Cart contains a 30.00 EUR item and a 90.00 EUR item. In-country address.
- **Steps:**
  1. Proceed to checkout and confirm shipping is 0.00 EUR.
  2. Remove the 30.00 EUR item. Subtotal becomes 90.00 EUR.
  3. Review the order summary.
- **Expected result:** Shipping recalculates to 15.00 EUR before payment. The total updates accordingly.

### TC13-13 - Cart rises above threshold after adding an item
- **Preconditions:** Cart subtotal is 80.00 EUR (shipping 15.00 EUR shown). In-country address.
- **Steps:**
  1. Add an item priced 20.00 EUR.
  2. Review the order summary.
- **Expected result:** Subtotal is 100.00 EUR and shipping recalculates to 0.00 EUR.

### TC13-14 - Foreign (out-of-country) delivery address with subtotal at or above 100 EUR
- **Preconditions:** Cart subtotal is 150.00 EUR. Delivery address is in a different country.
- **Steps:**
  1. Proceed to checkout and enter the foreign address.
  2. Review the order summary.
- **Expected result:** The domestic free-shipping rule must not be applied. The shipping charge for a foreign address is **BLOCKED - see Q1**; the requirement does not define it.

### TC13-15 - Foreign address with subtotal below 100 EUR
- **Preconditions:** Cart subtotal is 50.00 EUR. Delivery address is in a different country.
- **Steps:**
  1. Proceed to checkout and enter the foreign address.
- **Expected result:** **BLOCKED - see Q1.** The requirement does not say whether 15 EUR applies outside the country.

### TC13-16 - Client-side tampering with shipping value
- **Preconditions:** Cart subtotal is 50.00 EUR (shipping 15.00 EUR). Access to browser dev tools or an API client.
- **Steps:**
  1. Modify the request that submits the order so that shipping is 0.00 EUR.
  2. Submit the order.
- **Expected result:** The server recalculates shipping from the subtotal and address. The order is charged 15.00 EUR shipping, or the request is rejected. The tampered 0.00 EUR value is not accepted.

### TC13-17 - No delivery address entered
- **Preconditions:** Cart subtotal is 50.00 EUR. No address entered.
- **Steps:**
  1. Proceed to checkout and stop before entering an address.
  2. Look at the shipping line.
- **Expected result:** Shipping is not finalised. The UI shows a pending or "enter address" state, or the default-country rule applies. **Confirm with Q1.** No free-shipping decision is made without a country.

### TC13-18 - Invalid or unknown country code
- **Preconditions:** Cart subtotal is 150.00 EUR. Country field has an unsupported value.
- **Steps:**
  1. Enter an unsupported country and proceed.
- **Expected result:** Checkout shows a validation error. Free shipping is not granted on an unresolved address.

### TC13-19 - Discount or coupon applied to a cart near the threshold
- **Preconditions:** Cart items total 110.00 EUR. A 15.00 EUR coupon is available. In-country address.
- **Steps:**
  1. Apply the coupon. Subtotal becomes 95.00 EUR.
  2. Review shipping.
- **Expected result:** **BLOCKED - see Q2.** The requirement does not say whether the threshold is checked before or after discounts. Record the observed result and raise it against Q2.

### TC13-20 - Post-payment item removal or partial refund
- **Preconditions:** Order placed with subtotal 120.00 EUR and free shipping. Order is paid.
- **Steps:**
  1. Partially refund or cancel an item so the remaining merchandise total is 80.00 EUR.
- **Expected result:** **BLOCKED - see Q6.** The requirement does not say whether shipping is retroactively charged or refunded.

### TC13-21 - Tax-inclusive threshold check
- **Preconditions:** Prices shown including VAT. Items net 92.00 EUR, VAT 8.00 EUR, gross 100.00 EUR. In-country address.
- **Steps:**
  1. Proceed to checkout.
  2. Review shipping.
- **Expected result:** **BLOCKED - see Q3.** The requirement does not say whether the 100 EUR threshold is measured gross or net. Record the observed result against Q3.

---

## 4. Open questions

These are not answered by the requirement. No behaviour is assumed for them above; the related test cases are blocked.

- **Q1 - "Within the country":** Which country is meant? Is the shop single-country? What shipping rule applies to addresses outside it (fee, not offered, different threshold)? Does a missing or invalid address block the shipping calculation? (TC13-14, TC13-15, TC13-17, TC13-18)
- **Q2 - Threshold base:** Is the 100 EUR threshold measured on merchandise subtotal before or after discounts and coupons? (TC13-19)
- **Q3 - VAT:** Is the 100 EUR threshold checked gross (VAT-inclusive) or net? Is the 15 EUR fee VAT-inclusive? (TC13-21)
- **Q4 - Other fees and totals:** Are there other charges in the total (handling, packaging, payment fees) that count towards or are excluded from the threshold?
- **Q5 - Scope of "order":** Is shipping charged once per order or per parcel/shipment? How are split shipments (partially in stock, multiple warehouses) handled against the threshold?
- **Q6 - Changes after order placement:** If an order drops below 100 EUR after cancellation, return, partial refund or edit, is shipping recharged, refunded or left unchanged? (TC13-20)
- **Q7 - Excluded products:** Are any products excluded from the threshold or from free shipping (gift cards, digital goods, oversized items, specific categories)?
- **Q8 - Currency:** Is the shop EUR-only? If prices are shown in other currencies, which amount is compared to 100 EUR, and at what exchange rate and rounding?
- **Q9 - Rounding:** What rounding applies to line items, tax and totals? Is the comparison done on the rounded or unrounded subtotal?
- **Q10 - Shipping methods:** Do alternative methods (express, pickup point, same-day) exist, and do they have their own price or threshold? Does the 15 EUR rule apply to all of them?
- **Q11 - Threshold comparison:** Is the threshold "100 EUR or more" inclusive as written (assumed in TC13-06), and is it evaluated at checkout only or continuously as the cart changes? (TC13-12, TC13-13)
- **Q12 - Minimum and zero-value orders:** Is there a minimum order value? Are zero-value or near-zero orders permitted, and does 15 EUR apply to them? (TC13-09, TC13-11)
- **Q13 - Display:** Must the shop show the free-shipping threshold to the customer (for example, "Add 20 EUR more for free shipping")? This is not stated in the requirement and is not tested here.

---

## Summary

- Positive: 4 cases (TC13-01 to TC13-04)
- Boundary: 7 cases (TC13-05 to TC13-11), of which TC13-11 is blocked
- Negative and edge: 10 cases (TC13-12 to TC13-21), of which TC13-14, TC13-15, TC13-17, TC13-19, TC13-20 and TC13-21 are blocked or depend on open questions
- Open questions: 13 (Q1 to Q13)
