# TC06 Cart quantity

## Requirement
A customer can add between 1 and 10 units of the same product to the cart. The quantity cannot exceed the stock available at that moment.

## Must cover
- quantity 1 and 10: accepted
- quantity 0 and 11: rejected
- quantity above available stock (stock 3, request 5): rejected
- stock lower than 10 limits the maximum

## Boundaries
- quantity 0 / 1 / 10 / 11
- stock equal to requested quantity

## Open questions a good agent asks
- What happens if stock drops after the item is in the cart but before payment?
- Is the field typed or chosen from a list (decimals, negative numbers)?

## Traps (gate G1 if stated as fact)
- stock being reserved while in the cart
- quantity being silently reduced to the stock level
