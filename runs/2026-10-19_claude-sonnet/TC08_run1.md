# Test cases: registration age gate (18+)

Requirement: To register, a user enters a date of birth. Only users aged 18 or older can create an account.

Assumptions (stated, not invented behaviour): age is calculated as full years completed on the day of registration, against a single reference date and time zone (see Q2 and Q3). "T" below means the registration date. Where the expected result depends on an open question, the case is marked [OQ-n] and the expected result is the most likely reading, to be confirmed.

## Positive

**TC-01 - Adult user registers successfully**
- Preconditions: Registration form is open. All other mandatory fields are valid. Email is not already registered.
- Steps: 1. Enter a DOB giving age 30 (T minus 30 years, minus 1 day). 2. Complete the other fields. 3. Submit.
- Expected: Account is created. User gets the normal success flow. DOB is stored.

**TC-02 - Exactly 18 today (birthday is T)**
- Preconditions: As TC-01.
- Steps: 1. Enter DOB = T minus 18 years (same day and month). 2. Submit.
- Expected: Account is created (18 or older is allowed). [OQ-2]

**TC-03 - 18 years and 1 day old**
- Steps: Enter DOB = T minus 18 years minus 1 day. Submit.
- Expected: Account is created.

**TC-04 - Very old but plausible age**
- Steps: Enter DOB giving age 90. Submit.
- Expected: Account is created. [OQ-6]

**TC-05 - Leap-day birthday, adult**
- Steps: Enter 29 Feb of a leap year at least 19 years ago (for example 29-02-2000 when T is after 2018). Submit.
- Expected: Date is accepted and account is created.

## Negative

**TC-06 - Minor is rejected**
- Steps: Enter DOB giving age 12. Submit.
- Expected: Account is not created. Clear message that the user must be 18 or older. No user record is stored. [OQ-4, OQ-5]

**TC-07 - One day short of 18 (birthday is T+1)**
- Steps: Enter DOB = T minus 18 years plus 1 day. Submit.
- Expected: Registration is rejected (age 17).

**TC-08 - Age 17 years, 364 days**
- Steps: Same as TC-07 (equivalent partition). Confirm the rule uses the full date, not just the year.
- Expected: Rejected. Guards against the "current year minus birth year" bug: for example, a person born 31 Dec, with T on 1 Jan 18 years later, is 17 and must be rejected only if their birthday has not yet occurred in the year of T.

**TC-09 - Birth year gives 18 but birthday later this year**
- Steps: Enter DOB with year = current year minus 18 and a month and day after T.
- Expected: Rejected.

**TC-10 - Future date of birth**
- Steps: Enter DOB = T plus 1 day, then T plus 1 year.
- Expected: Validation error (invalid date). Account is not created. [OQ-7]

**TC-11 - DOB left empty**
- Steps: Leave DOB blank. Submit.
- Expected: Mandatory-field error. Account is not created.

**TC-12 - Invalid calendar date**
- Steps: Enter 31-04-2000, 30-02-2000, 29-02-2001 (non-leap year), month 13, day 0.
- Expected: Each is rejected as an invalid date. No account is created.

**TC-13 - Wrong format and non-date input**
- Steps: Enter "abc", "2000/13/45", "01-01-00", special characters, a SQL/script string.
- Expected: Rejected with a format error. Input is safely handled, with no server error and no script execution.

**TC-14 - Minor tries to bypass by going back and changing the DOB**
- Steps: 1. Submit a DOB aged 17 and get rejected. 2. Immediately resubmit with an adult DOB.
- Expected: Behaviour follows the product decision. [OQ-8] Record whether a retry is allowed.

**TC-15 - Server-side enforcement (bypass the UI)**
- Steps: Send the registration request directly to the API with an under-18 DOB, with client-side validation skipped. Also try a modified request that omits the DOB field.
- Expected: API returns a validation error (4xx). No account is created.

## Boundary

**TC-16 - Day boundary (T-1, T, T+1 around the 18th birthday)**
- Steps: Run TC-03, TC-02 and TC-07 together as a boundary triple.
- Expected: Rejected only when the 18th birthday is after T; accepted on and after the birthday.

**TC-17 - Leap-day birthday, 18th birthday year is not a leap year**
- Preconditions: User born 29-02-2008 (18th birthday falls in 2026, not a leap year).
- Steps: Register on 28-02-2026, then on 01-03-2026.
- Expected: Behaviour follows the stated rule for leap-day birthdays. [OQ-3] Most common conventions: 18 on 28-02 or on 01-03. Confirm one and test both days.

**TC-18 - Leap-day birthday, adult year is a leap year**
- Preconditions: User born 29-02-2004 (18th birthday on 29-02-2022 does not exist, since 2022 is not a leap year).
- Steps: Register on 28-02-2022 and 01-03-2022.
- Expected: As TC-17.

**TC-19 - Time zone around midnight**
- Steps: Register at 23:30 local time for a user whose 18th birthday is "today" in the user's zone but "tomorrow" in UTC (and the reverse).
- Expected: Result follows the defined reference zone. [OQ-2] Both cases must be consistent and documented.

**TC-20 - Date range extremes**
- Steps: Enter 01-01-1900, 01-01-1800, 01-01-0001, and 31-12-9999.
- Expected: Plausible lower bound handling is defined and enforced; future dates rejected. [OQ-6, OQ-7]

**TC-21 - Date at system boundaries**
- Steps: Enter 01-01-1970 and 31-12-1969 (epoch boundary); 19-01-2038 (32-bit time boundary) as a future-type date.
- Expected: No date overflow or parsing errors; valid pasts are accepted and futures rejected.

**TC-22 - Input channels give the same result**
- Steps: Enter the same boundary DOB (exactly 18 today) via the date picker, by typing, and by paste, on desktop and mobile.
- Expected: Identical result in every case.

**TC-23 - Date input formats and locales**
- Steps: Enter the same date in the DD/MM/YYYY and MM/DD/YYYY locales (for example 05/06/2008 vs 06/05/2008).
- Expected: Date is interpreted unambiguously per the displayed format. No day/month swap changes the age result. [OQ-9]

## Open questions

1. Is "18 or older" measured in completed years? We assume yes (the user turns 18 on their birthday, and is 18 on that whole day).
2. Which date and time zone is the reference: user's local date, server date, or UTC? When exactly does a user become eligible on their birthday (00:00 in which zone)?
3. How are 29 February birthdays treated in non-leap years: is the birthday 28 Feb or 1 Mar?
4. What is the exact user-facing message for under-18 rejection, and in which languages? Should it reveal the reason?
5. Is the DOB stored for rejected users? Is any data of a rejected minor kept (privacy, GDPR/COPPA-type obligations)?
6. Is there an upper age or earliest accepted year (for example reject DOBs implying age over 120)?
7. How are future dates handled: a generic invalid-date error or a specific one?
8. Can a rejected user retry immediately with a different DOB? Is there a lockout, cooldown, cookie or device flag to stop trial-and-error?
9. What input format is used (date picker, three fields, free text), and is the format locale dependent?
10. Is the age re-checked at later steps (email verification, first login, social sign-up/SSO where the DOB may come from the provider)?
11. Do other registration paths exist (API, admin-created accounts, imports, social login) and does the age rule apply to them?
12. Is DOB editable after registration? What happens if an existing user later changes it to an under-18 date?
13. Is the DOB verified in any way (ID check)? If not, we only test the declared DOB and not real age.
14. Does the 18 threshold vary by country or jurisdiction, and what is the user's country?
15. Is there parental-consent or other alternative flow for minors, or is the answer always a hard block?
