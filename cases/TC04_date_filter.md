# TC04 Date range filter

## Requirement
On the orders page a user can filter orders by a date range (From, To). Both dates are included. The range cannot be longer than 31 days. To cannot be earlier than From.

## Must cover
- orders on the From date and on the To date: included
- range of 31 days: accepted
- range of 32 days: rejected
- To earlier than From: rejected
- From equal to To: accepted (single day)

## Boundaries
- range length 30 / 31 / 32 days
- order timestamps at 00:00 and 23:59 on the edge dates

## Open questions a good agent asks
- Is "31 days" counted inclusively (1 Jan to 31 Jan = 31 days)?
- What happens if only one date is filled in?
- Which time zone are order timestamps in?

## Traps (gate G1 if stated as fact)
- a default range (e.g. last 7 days)
- future dates being blocked
