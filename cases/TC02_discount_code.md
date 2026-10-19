# TC02 Discount code

## Requirement
At checkout a customer can enter one discount code. A valid code gives 10% off orders of 50 EUR or more. Codes expire at 23:59 on their end date. Codes are not case-sensitive.

## Must cover
- valid code on an order of exactly 50 EUR: 10% off
- valid code on 49.99 EUR: rejected or no discount
- expired code: rejected
- code typed in lower case and mixed case: accepted
- second code: not allowed

## Boundaries
- order total 49.99 / 50.00 / 50.01
- time 23:59 / 00:00 next day on the end date

## Open questions a good agent asks
- Is 50 EUR before or after VAT and shipping?
- Which time zone defines 23:59?
- What message does the customer see on rejection?

## Traps (gate G1 if stated as fact)
- a maximum discount amount
- codes being single-use per customer
- leading/trailing spaces being trimmed
