# Test Cases: Registration Age Check (18+)

Requirement: To register, a user enters a date of birth. Only users aged 18 or older can create an account.

Assumption used only for expected results: age is counted in completed years on the day of registration. Where behaviour is not stated (messages, timezone, input format), the expected result is marked "per open question" and is not invented. See the Open Questions section.

## Positive

**TC-001: Adult well above the threshold can register**
- Preconditions: Registration form is open. All other required fields are valid. Today is 2026-10-19.
- Steps: 1. Enter date of birth 1990-05-15 (age 36). 2. Complete the other fields. 3. Submit.
- Expected: Account is created. User is not blocked on the date of birth.

**TC-002: User turns exactly 18 today (boundary)**
- Preconditions: As above. Today is 2026-10-19.
- Steps: 1. Enter date of birth 2008-10-19. 2. Submit.
- Expected: Account is created, because the user is 18 today.

**TC-003: User turned 18 yesterday**
- Preconditions: As above.
- Steps: 1. Enter date of birth 2008-10-18. 2. Submit.
- Expected: Account is created.

**TC-004: Very old but plausible date of birth**
- Preconditions: As above.
- Steps: 1. Enter date of birth 1930-01-01 (age 96). 2. Submit.
- Expected: Account is created, unless an upper limit is defined (see OQ-6).

**TC-005: Leap day birthday, adult**
- Preconditions: As above.
- Steps: 1. Enter date of birth 2000-02-29. 2. Submit.
- Expected: Account is created. The leap day is accepted as a valid date.

## Negative

**TC-006: User turns 18 tomorrow (boundary)**
- Preconditions: As above. Today is 2026-10-19.
- Steps: 1. Enter date of birth 2008-10-20. 2. Submit.
- Expected: Registration is rejected and no account is created. A message is shown (text per OQ-3).

**TC-007: Clearly underage user**
- Preconditions: As above.
- Steps: 1. Enter date of birth 2015-03-10 (age 11). 2. Submit.
- Expected: Registration is rejected and no account is created.

**TC-008: Born today**
- Preconditions: As above.
- Steps: 1. Enter date of birth 2026-10-19. 2. Submit.
- Expected: Rejected, no account created.

**TC-009: Date of birth in the future**
- Preconditions: As above.
- Steps: 1. Enter date of birth 2030-01-01. 2. Submit.
- Expected: Rejected as invalid or underage, no account created.

**TC-010: Date of birth left empty**
- Preconditions: As above.
- Steps: 1. Leave date of birth blank. 2. Fill the other fields. 3. Submit.
- Expected: Rejected, no account created. The date of birth field is flagged as required.

**TC-011: Non-existent calendar date**
- Preconditions: As above.
- Steps: 1. Enter 2001-02-30. 2. Submit.
- Expected: Rejected as an invalid date, no account created.

**TC-012: Feb 29 in a non-leap year**
- Preconditions: As above.
- Steps: 1. Enter 2003-02-29. 2. Submit.
- Expected: Rejected as an invalid date.

**TC-013: Malformed input**
- Preconditions: As above.
- Steps: 1. Enter each of: "abc", "2001/13/45", "01-01-01", "0000-00-00", "' OR 1=1 --", "<script>alert(1)</script>". 2. Submit after each.
- Expected: Each is rejected with a format or validation error. No account is created, no server error, no script runs.

**TC-014: Underage user retries with a different date**
- Preconditions: TC-007 was just performed with the same email.
- Steps: 1. Re-enter the same email with an adult date of birth. 2. Submit.
- Expected: Behaviour per OQ-5 (retry allowed or blocked). Record the actual result and raise it if the requirement stays silent.

**TC-015: Server-side enforcement (bypass client validation)**
- Preconditions: Registration API is reachable.
- Steps: 1. Send the registration request directly (for example with curl) with an underage date of birth, skipping the UI. 2. Also try editing the DOM or disabling JavaScript in the form.
- Expected: The server rejects the request. No account is created.

## Boundary and Date Edge Cases

**TC-016: Birthday later this month, same year offset**
- Preconditions: Today is 2026-10-19.
- Steps: 1. Enter 2008-12-31. 2. Submit.
- Expected: Rejected (age 17, turns 18 in about 2 months).

**TC-017: Birthday earlier in the year**
- Preconditions: Today is 2026-10-19.
- Steps: 1. Enter 2008-01-01. 2. Submit.
- Expected: Accepted (age 18).

**TC-018: Born on Feb 29, 18th birthday on a non-leap year**
- Preconditions: Test clock set to 2026-02-28, then 2026-03-01. Date of birth is 2008-02-29.
- Steps: 1. Register on 2026-02-28. 2. Register on 2026-03-01.
- Expected: Result per OQ-4, because the requirement does not say whether the user is 18 on Feb 28 or Mar 1. Record actual behaviour on both days. Both must be consistent with the stated rule.

**TC-019: Day rollover at midnight**
- Preconditions: Test clock set just before midnight on the day before the 18th birthday.
- Steps: 1. Submit at 23:59:59. 2. Submit again at 00:00:01 the next day.
- Expected: First attempt rejected, second accepted. Timezone basis per OQ-2.

**TC-020: Timezone difference at the boundary**
- Preconditions: User's local date differs from the server date (for example the user is in UTC+13 and the server is in UTC). Date of birth makes the user exactly 18 in one zone only.
- Steps: 1. Submit the registration.
- Expected: Result per OQ-2. Record which timezone the system uses and confirm it is applied consistently.

**TC-021: Earliest sensible date**
- Preconditions: As above.
- Steps: 1. Enter 1900-01-01. 2. Enter 1800-01-01. 3. Enter 0001-01-01.
- Expected: Behaviour per OQ-6. No crash or overflow, and a clear accept or reject for each.

## Data Handling

**TC-022: Date of birth is stored correctly**
- Preconditions: Account created through TC-002.
- Steps: 1. Check the stored value of the date of birth (DB or API).
- Expected: Stored date equals the entered date, with no timezone shift to the previous or next day.

**TC-023: No account or partial data after rejection**
- Preconditions: TC-007 performed.
- Steps: 1. Check the user table, the verification emails sent and the session state.
- Expected: No user record, no confirmation email, no logged-in session. Retention of the rejected date of birth per OQ-7.

**TC-024: Different input formats and locales**
- Preconditions: Forms in locales with different date formats (MM/DD/YYYY, DD/MM/YYYY).
- Steps: 1. Enter 03/04/2008 in each locale. 2. Submit.
- Expected: The date is interpreted according to the displayed format (March 4 or April 3), and the age decision matches that interpretation. The format is shown clearly to the user.

## Open Questions (not assumed in the tests above)

1. OQ-1: How is "18 or older" computed? Completed years on the registration date is assumed, but this is not stated. Is the 18th birthday itself accepted (the wording "18 or older" suggests yes)?
2. OQ-2: Which timezone defines "today": user's local time, server time or UTC? This affects users near midnight.
3. OQ-3: What exact message is shown on rejection? Does it reveal the age rule? Is it localised?
4. OQ-4: For users born on Feb 29, on which date do they turn 18 in a non-leap year (Feb 28 or Mar 1)?
5. OQ-5: After an underage rejection, may the user retry with another date of birth? Is there any lockout, cookie or device or email block to stop trial and error?
6. OQ-6: Is there a maximum age or earliest accepted date (for example 120 years) to reject implausible dates?
7. OQ-7: Is the date of birth stored for rejected attempts? Are there privacy or legal requirements (GDPR, COPPA, local age of majority) for handling data of minors?
8. OQ-8: What is the input format (free text, date picker, three dropdowns) and the accepted format(s)?
9. OQ-9: Is the date of birth verified in any way (ID check), or is it self-declared only? Is there any other registration path (social login, API, admin creation, invitations) that must also enforce the rule?
10. OQ-10: Does the age rule differ by country or region (some places set the legal age at 19, 20 or 21)?
11. OQ-11: Can the date of birth be edited after registration, and if so, can a user change it to become underage or to become eligible?
12. OQ-12: Are existing accounts affected, and is there a re-check at a later step (for example email confirmation or first login)?
