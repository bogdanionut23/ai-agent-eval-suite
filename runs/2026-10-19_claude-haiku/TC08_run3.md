# TC08: Test cases for the date-of-birth age gate

Requirement: To register, a user enters a date of birth. Only users aged 18 or older can create an account.

Working assumptions used for expected results (each one is also listed under Open questions):
- "Aged 18 or older" means the user has reached their 18th birthday on or before the current date.
- The check runs at registration time, against the current date.
- Under-18 attempts create no account.

---

## Positive scenarios

### TC08-01: Adult user registers with a DOB well over 18
- **Preconditions:** Registration form is open. No account exists for the test email.
- **Steps:**
  1. Enter a valid email and password.
  2. Enter date of birth 1990-05-15.
  3. Submit the form.
- **Expected result:** Account is created. The user is taken to the next step (for example, confirmation or dashboard). No age-related error is shown.

### TC08-02: User who turned 18 the day before registration is accepted
- **Preconditions:** Current date is 2026-10-19.
- **Steps:**
  1. Enter date of birth 2008-10-18 (18 years and 1 day old).
  2. Complete the other required fields and submit.
- **Expected result:** Account is created.

---

## Boundary scenarios

### TC08-03: Registration on the 18th birthday is accepted
- **Preconditions:** Current date is 2026-10-19.
- **Steps:**
  1. Enter date of birth 2008-10-19 (exactly 18 years before today).
  2. Complete the other required fields and submit.
- **Expected result:** Account is created. The user is 18 on that day, so the "18 or older" rule is met.

### TC08-04: Registration one day before the 18th birthday is rejected
- **Preconditions:** Current date is 2026-10-19.
- **Steps:**
  1. Enter date of birth 2008-10-20 (turns 18 tomorrow).
  2. Complete the other required fields and submit.
- **Expected result:** Account is not created. An age-related validation error is shown on the date-of-birth field. The user stays on the registration form.

### TC08-05: Leap-day birth date, adult in the current year
- **Preconditions:** Current date is 2026-10-19.
- **Steps:**
  1. Enter date of birth 2008-02-29.
  2. Complete the other required fields and submit.
- **Expected result:** Account is created. The 18th birthday (in 2026, a non-leap year, under any reasonable convention) has already passed.

### TC08-06: Leap-day birth date on the non-leap-year boundary (open question)
- **Preconditions:** Use a test clock or mocked date set to 2026-02-28, then 2026-03-01.
- **Steps:**
  1. With the date set to 2026-02-28, enter date of birth 2008-02-29 and submit.
  2. Repeat with the date set to 2026-03-01.
- **Expected result:** Depends on OQ-5. Under the "1 March" convention, the first attempt is rejected and the second is accepted. Under the "28 February" convention, the first attempt is accepted. Record the rule the team confirms.

### TC08-07: Date of birth one day after today (age 0) is rejected
- **Preconditions:** Current date is 2026-10-19.
- **Steps:**
  1. Enter date of birth 2026-10-19 (born today, age 0).
  2. Submit.
- **Expected result:** Account is not created. Age-related validation error shown.

### TC08-08: Maximum age boundary (open question)
- **Preconditions:** Current date is 2026-10-19.
- **Steps:**
  1. Enter date of birth 1900-01-01.
  2. Submit.
- **Expected result:** Under the literal requirement, the user is 18 or older, so the account is created. Confirm with OQ-4 whether an upper bound applies.

### TC08-09: Timezone boundary at the 18th birthday (open question)
- **Preconditions:** A user is in a timezone where it is already their 18th birthday, while the server clock (for example UTC) still shows the day before. Set up the clock for both.
- **Steps:**
  1. Enter that user's 18th birthday as the date of birth and submit.
- **Expected result:** Depends on OQ-1 (which timezone counts as "today"). Record the agreed behaviour.

---

## Negative scenarios

### TC08-10: Clearly under-18 user is rejected
- **Preconditions:** Current date is 2026-10-19. No account exists for the test email.
- **Steps:**
  1. Enter valid email and password.
  2. Enter date of birth 2015-06-01 (about 11 years old).
  3. Submit.
- **Expected result:** Account is not created. An age-related validation error is shown. No welcome or confirmation email is sent.

### TC08-11: 17 years and 11 months old is rejected
- **Preconditions:** Current date is 2026-10-19.
- **Steps:**
  1. Enter date of birth 2008-10-20 (already covered by TC08-04, kept here as the "just under" case with the same DOB, so reviewers can read the pair together).
  2. Submit.
- **Expected result:** Same as TC08-04. Account is not created.

### TC08-12: Required field left empty
- **Preconditions:** Registration form is open.
- **Steps:**
  1. Leave the date-of-birth field empty.
  2. Fill the other fields with valid data and submit.
- **Expected result:** Account is not created. A "date of birth is required" error is shown. The age rule is not evaluated as a pass.

### TC08-13: Impossible calendar date is rejected
- **Preconditions:** Registration form is open.
- **Steps:**
  1. Enter date of birth 2000-02-31.
  2. Submit.
- **Expected result:** Account is not created. An invalid-date error is shown. The system does not roll the date over to 2000-03-02.

### TC08-14: Non-date text is rejected
- **Preconditions:** Registration form is open.
- **Steps:**
  1. Enter "abc" in the date-of-birth field.
  2. Submit.
- **Expected result:** Account is not created. An invalid-date error is shown.

### TC08-15: Partial date is rejected
- **Preconditions:** Registration form is open.
- **Steps:**
  1. Enter "1990-05" (no day).
  2. Submit.
- **Expected result:** Account is not created. An invalid-date error is shown.

### TC08-16: Future date of birth is rejected
- **Preconditions:** Current date is 2026-10-19.
- **Steps:**
  1. Enter date of birth 2030-01-01.
  2. Submit.
- **Expected result:** Account is not created. A validation error is shown. The exact message depends on OQ-3. The age rule must not treat a negative age as adult.

### TC08-17: Under-18 attempt leaves no data behind
- **Preconditions:** Current date is 2026-10-19. Record the state of the account table and outgoing email queue before the test.
- **Steps:**
  1. Submit a valid form with date of birth 2015-06-01 (see TC08-10).
  2. Check the database and email queue.
- **Expected result:** No account row is created. No verification email is queued. Whether a rejection log entry is written depends on OQ-8.

### TC08-18: Age gate is enforced server-side, not only in the UI
- **Preconditions:** Access to the registration API directly, for example with an HTTP client.
- **Steps:**
  1. Send a registration request with date of birth 2015-06-01, bypassing the front-end validation.
  2. Inspect the response.
- **Expected result:** The request is rejected with an age-related error. No account is created. (Assumes the age check is required on the server. See OQ-2.)

### TC08-19: Age is evaluated at submit time, not at page load
- **Preconditions:** Registration form is open in a browser tab. Simulate the clock moving past the user's 18th birthday while the tab stays open (or use a test clock).
- **Steps:**
  1. Enter a DOB that is 18 at page-load time but not at submit time.
  2. Submit after the date passes.
- **Expected result:** Behaviour depends on OQ-1 and OQ-2. Under the stated assumption (checked at registration time), the submit should be rejected because the user is under 18 at that moment.

---

## Open questions

These points are not specified in the requirement. Test expectations above state the assumption used, and each item needs a decision before the cases can be finalised.

- **OQ-1 (timezone):** Which timezone defines "today" for the age calculation: the user's local time, the server's time (for example UTC), or a fixed company timezone? This affects TC08-09 and TC08-19.
- **OQ-2 (enforcement point):** Is the age check required server-side, client-side, or both? Should the server re-check even if the client already validated? This affects TC08-18.
- **OQ-3 (future and invalid dates):** What is the expected error message for a future date of birth, and what should the user see for an impossible date like 2000-02-31?
- **OQ-4 (upper bound):** Is there a maximum age or a minimum year for date of birth (for example, 120 years)? Is 1900-01-01 valid? This affects TC08-08.
- **OQ-5 (leap-day rule):** On which date does a user born on 29 February become 18 in a non-leap year: 28 February or 1 March? This affects TC08-06.
- **OQ-6 (input format and locale):** Which date formats are accepted (for example DD/MM/YYYY vs. MM/DD/YYYY)? Is the field a date picker only, or free text? Does the format change by locale?
- **OQ-7 (error message content):** Should the error say the user is too young, or state the minimum age (18)? Should it be generic to avoid revealing the rule? Can the user retry with a different date of birth in the same session?
- **OQ-8 (logging and retention):** Is a rejected attempt logged? Is the entered date of birth stored for rejected attempts? Are there retention rules, for example under GDPR or children's data rules?
- **OQ-9 (rate limiting):** Is there a limit on how many times a user can retry the date of birth after a rejection, to prevent trying different dates until one passes?
- **OQ-10 (other registration paths):** Does the age gate also apply to SSO, social login, invitations, and admin-created accounts? TC08-01 to TC08-19 cover only the standard form.
- **OQ-11 (jurisdiction):** Is 18 a single global threshold, or do some regions require a different age? The requirement states 18 without qualification.
- **OQ-12 (changing date of birth later):** Can the stored date of birth be edited after registration? If so, can an under-18 user change it to pass? Is the check repeated on edit?
- **OQ-13 (stored date of birth):** Is the date of birth stored after a successful registration? If yes, for what purpose, and is it visible to other users?
