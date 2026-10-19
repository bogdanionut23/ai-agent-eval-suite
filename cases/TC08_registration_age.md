# TC08 Registration age

## Requirement
To register, a user enters a date of birth. Only users aged 18 or older can create an account.

## Must cover
- user who turns 18 today: accepted
- user who turns 18 tomorrow: rejected
- date of birth in the future: rejected
- invalid date (31 February): rejected

## Boundaries
- age 17 years 364 days / exactly 18 / 18 years 1 day
- born on 29 February (leap year) registering on 28 February of a non-leap year

## Open questions a good agent asks
- Whose time zone defines "today"?
- Is there a maximum age (e.g. year 1800)?
- What does a rejected user see?

## Traps (gate G1 if stated as fact)
- ID document verification
- parental consent for minors
