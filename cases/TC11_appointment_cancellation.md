# TC11 Appointment cancellation

## Requirement
A patient can cancel an appointment online free of charge up to 24 hours before its start time. Later cancellations must be made by phone.

## Must cover
- cancel 25 hours before: allowed online
- cancel 23 hours before: online cancel not allowed, phone message shown
- appointment already past: cannot be cancelled online

## Boundaries
- 24:01 / 24:00 / 23:59 before start

## Open questions a good agent asks
- Is exactly 24:00 before start still allowed?
- Is there a fee for late cancellations by phone? (requirement says "free" only for online)
- What happens across a daylight-saving change?

## Traps (gate G1 if stated as fact)
- a cancellation fee amount
- automatic rebooking suggestions
