# Test cases: checkout discount code

Requirement: At checkout a customer can enter one discount code. A valid code gives 10% off orders of 50 EUR or more. Codes expire at 23:59 on their end date. Codes are not case-sensitive.

Assumptions (only what the requirement states): the discount is 10% of the order amount; the threshold is inclusive ("50 EUR or more"); expiry is 23:59 on the end date. Everything else is in the open questions.

Common preconditions unless stated otherwise: customer is on the checkout page with a non-empty cart. Test codes: SAVE10 (valid, end date D, 10%), OLD10 (end date in the past), FAKE99 (does not exist).

## Positive

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-P01 | Valid code on order above threshold | Cart total 80.00 EUR; SAVE10 valid | 1. Enter SAVE10. 2. Apply. | Code accepted. 10% discount of 8.00 EUR shown. Order total 72.00 EUR. |
| TC-P02 | Code with lowercase input | Cart 80.00 EUR; SAVE10 valid | Enter "save10", apply. | Accepted. Same 10% discount as TC-P01. |
| TC-P03 | Code with mixed case input | Cart 80.00 EUR | Enter "SaVe10", apply. | Accepted. 10% discount applied. |
| TC-P04 | Code used on its end date before 23:59 | System time is end date D, 12:00; cart 80.00 EUR | Enter SAVE10, apply. | Accepted, 10% discount applied. |
| TC-P05 | Discounted total carried to order confirmation | Cart 100.00 EUR | Apply SAVE10, place the order. | Confirmation, payment amount and order record all show 90.00 EUR and the code used. |
| TC-P06 | Code applied then cart total shown correctly after page reload | Cart 100.00 EUR, SAVE10 applied | Reload checkout page. | Behaviour is consistent: the code is still applied with 90.00 EUR, or the page clearly shows it is not applied (see OQ-9). |

## Boundary

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-B01 | Order exactly 50.00 EUR | Cart total 50.00 EUR | Apply SAVE10. | Accepted (threshold inclusive). Discount 5.00 EUR, total 45.00 EUR. |
| TC-B02 | Order just below threshold | Cart total 49.99 EUR | Apply SAVE10. | Rejected with a message that the minimum order is 50 EUR. No discount. |
| TC-B03 | Order just above threshold | Cart total 50.01 EUR | Apply SAVE10. | Accepted. Discount 5.00 EUR (0.005 rounded per OQ-3), no negative or zero-cent errors. |
| TC-B04 | Zero or empty cart | Empty cart, or total 0.00 EUR | Try to apply SAVE10. | Rejected or checkout not reachable. No discount. |
| TC-B05 | Expiry at 23:59:00 on end date | System time D 23:59:00 | Apply SAVE10. | Accepted (code still valid at 23:59). |
| TC-B06 | Expiry at 23:59:59 on end date | System time D 23:59:59 | Apply SAVE10. | Result depends on OQ-4. Record actual; expected to be defined by the answer. |
| TC-B07 | One minute after expiry | System time D+1 00:00:00 | Apply SAVE10. | Rejected as expired. No discount. |
| TC-B08 | Code expires between applying and paying | Cart 80 EUR; apply SAVE10 at D 23:58; pay at D+1 00:01 | Apply, wait, place order. | Behaviour per OQ-5. Must not be silently inconsistent between displayed and charged amount. |
| TC-B09 | Code with leading/trailing spaces | Cart 80 EUR | Enter " SAVE10 " (spaces). | Per OQ-7. Record actual; ideally trimmed and accepted. |
| TC-B10 | Very large order | Cart 1,000,000.00 EUR | Apply SAVE10. | 10% (100,000.00 EUR) discount computed correctly, no overflow. |
| TC-B11 | Rounding on odd cents | Cart 50.05 EUR | Apply SAVE10. | Discount rounded per OQ-3 (5.005 becomes 5.00 or 5.01). Total consistent in UI, payment and order record. |
| TC-B12 | Time zone boundary | Customer in a different time zone from the server; time near 23:59 on D | Apply SAVE10. | Expiry evaluated in the time zone defined by OQ-4. |
| TC-B13 | Maximum code length | Code of the maximum allowed length | Enter and apply. | Handled without error (accepted if it exists, otherwise "invalid code"). |

## Negative

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-N01 | Non-existent code | Cart 80 EUR | Enter FAKE99, apply. | Rejected with an "invalid code" message. Total unchanged. |
| TC-N02 | Expired code | Cart 80 EUR | Enter OLD10, apply. | Rejected with an "expired" message. Total unchanged. |
| TC-N03 | Empty code | Cart 80 EUR | Leave field blank, press Apply. | Validation message, no discount, no error page. |
| TC-N04 | Whitespace-only code | Cart 80 EUR | Enter spaces, apply. | Treated as empty. Validation message. |
| TC-N05 | Second code while one is applied | SAVE10 applied; a second valid code OTHER5 exists | Enter OTHER5, apply. | Only one code allowed: second rejected, or it replaces the first (OQ-1). Never both applied. |
| TC-N06 | Two codes in one field | Cart 80 EUR | Enter "SAVE10 OTHER5" or "SAVE10,OTHER5". | Rejected as invalid. No stacked discount. |
| TC-N07 | Same code re-applied | SAVE10 applied | Apply SAVE10 again. | Discount stays 10%, not 19% or 20%. |
| TC-N08 | Special characters / SQL / script injection | Cart 80 EUR | Enter `' OR 1=1 --` and `<script>alert(1)</script>`. | Rejected as invalid, input escaped, no error, no discount. |
| TC-N09 | Below-threshold order, then add items | Cart 40 EUR | Apply SAVE10 (rejected). Add items to reach 60 EUR. Apply again. | First attempt rejected. Second accepted, 6.00 EUR off. |
| TC-N10 | Remove items after applying code | Cart 60 EUR, SAVE10 applied | Remove items so the total is 40 EUR. | Discount removed or code invalidated with a message (OQ-6). Never 10% on an order under 50 EUR. |
| TC-N11 | Discount applied to a below-threshold order via direct request | Cart 30 EUR | Send the apply-code request directly to the API. | Server-side validation rejects it. No discount. |
| TC-N12 | Expired code via API / tampered client time | Client clock set to before D | Apply OLD10. | Rejected: validation uses server time. |
| TC-N13 | Repeated failed attempts | Cart 80 EUR | Enter 20 wrong codes quickly. | System stays stable. Throttling or lockout if specified (OQ-10). |
| TC-N14 | Code removal | SAVE10 applied | Remove the code. | Discount removed, total returns to original. Another code can now be entered. |
| TC-N15 | Unicode / similar-looking characters | Cart 80 EUR | Enter "SAVE１０" (full-width digits) or Turkish dotless i variants of a code. | Not treated as equal to SAVE10 unless defined. Case-insensitivity must not break on locale (OQ-8). |

## Open questions

1. Does the discount code replace the first when a second is entered, or is the second rejected? The requirement says "one code" but not what happens on a second attempt.
2. Is the 50 EUR threshold measured before or after other price changes: shipping, tax (VAT), gift cards, other promotions, items already on sale? "Orders of 50 EUR or more" does not say which amount.
3. Does the 10% apply to the whole order or only eligible items, and does it include shipping and taxes? What is the rounding rule (half-up, half-even, down) and to how many decimals?
4. Which time zone is "23:59" evaluated in (server, customer, merchant)? What happens during 23:59:01 to 23:59:59 and 00:00, since "expire at 23:59" could mean valid through 23:59:59 or only until 23:59:00?
5. If a code is valid when applied but expires before payment, is the discount honoured, re-validated, or removed?
6. If the cart changes after the code is applied (total drops below 50 EUR), is the code removed automatically, with what message?
7. Should leading/trailing whitespace be trimmed? Are there allowed characters and a maximum length for codes?
8. "Not case-sensitive": does this include non-ASCII letters and locale-specific casing (e.g. Turkish i)?
9. Is the applied code persisted across page reloads, sessions and devices, and across guest vs. logged-in checkout?
10. Are there usage limits (per customer, total redemptions, one use per account) or throttling on failed attempts? The word "valid" implies other validity rules that are not listed.
11. Is there a start date, and what happens before it?
12. Can the code combine with other discounts, loyalty points or sale prices?
13. What are the exact error messages for: invalid, expired, below minimum, empty? Are they localised?
14. Does the discount appear as a separate line on the receipt, invoice and in refunds? On a partial refund, is the discount prorated?
15. Currency: is the threshold only in EUR? What happens for customers paying in another currency?
