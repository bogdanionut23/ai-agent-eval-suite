# TC10 Daily transfer limit

## Requirement
A customer can send bank transfers of up to 5,000 EUR in total per calendar day. A transfer that would exceed the remaining daily amount is rejected in full.

## Must cover
- one transfer of exactly 5,000 EUR: accepted
- 3,000 then 2,000 on the same day: both accepted
- 3,000 then 2,001: second rejected in full (not partially sent)
- next calendar day: limit available again

## Boundaries
- total 4,999.99 / 5,000.00 / 5,000.01
- transfer at 23:59 and 00:00

## Open questions a good agent asks
- Does a rejected or cancelled transfer count toward the limit?
- Which time zone defines the calendar day?
- Are transfers in other currencies converted, and at which rate?

## Traps (gate G1 if stated as fact)
- a per-transaction limit
- a rolling 24-hour window instead of a calendar day
