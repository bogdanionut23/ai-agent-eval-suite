# TC15 Quiet hours

## Requirement
Users can set quiet hours (for example 22:00 to 07:00). During quiet hours no push notifications are sent; they are delivered when quiet hours end.

## Must cover
- notification at 23:00 with quiet hours 22:00–07:00: held, delivered at 07:00
- notification at 21:59: sent immediately
- notification at 07:00: sent immediately
- range crossing midnight works

## Boundaries
- 21:59 / 22:00 / 06:59 / 07:00

## Open questions a good agent asks
- Is the end time included or excluded?
- In which time zone: the user's device or the account setting?
- Are urgent notifications (security alerts) exempt?
- If 50 notifications pile up, are they all delivered or grouped?

## Traps (gate G1 if stated as fact)
- grouping held notifications into one
- email or SMS being affected
