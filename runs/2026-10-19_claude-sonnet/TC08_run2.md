# Test Cases: Registration with Age Restriction (18+)

**Requirement:** To register, a user enters a date of birth. Only users aged 18 or older can create an account.

**Assumptions (kept minimal, pending the open questions below):** "Today" is the date on the server at submission time. Age is calculated in completed years. A DOB entered as a valid calendar date in the format the UI accepts. For the examples below, assume today = 2026-10-19, so the 18th-birthday cutoff is DOB = 2008-10-19. If the actual answers to the open questions differ, expected results marked (*) must be re-confirmed.

---

## 1. Positive scenarios

### TC-REG-001: Adult user (well above 18) can register
- **Preconditions:** Registration page is open. All other required fields are valid (email, password etc.). Email is not already registered.
- **Steps:**
  1. Enter DOB 1990-05-15.
  2. Fill the remaining required fields with valid data.
  3. Submit the form.
- **Expected result:** Account is created. User receives the normal post-registration outcome (success message, confirmation email or redirect, as defined for the product). No age-related error is shown.

### TC-REG-002: User turning exactly 18 today can register (boundary)
- **Preconditions:** As TC-REG-001. Today = 2026-10-19.
- **Steps:**
  1. Enter DOB 2008-10-19 (18th birthday is today).
  2. Fill remaining required fields validly.
  3. Submit.
- **Expected result:** Account is created. Aged 18 or older is satisfied on the birthday itself. (*)

### TC-REG-003: User who turned 18 yesterday can register (boundary)
- **Preconditions:** As TC-REG-001.
- **Steps:**
  1. Enter DOB 2008-10-18.
  2. Fill remaining fields validly and submit.
- **Expected result:** Account is created.

### TC-REG-004: User aged 18 with a birthday later in the calendar year can register (boundary on month/day logic)
- **Preconditions:** As TC-REG-001. Today = 2026-10-19.
- **Steps:**
  1. Enter DOB 2008-01-01 (birthday earlier in the year, age 18).
  2. Submit with valid data.
- **Expected result:** Account is created.

### TC-REG-005: Leap-day birth date for an adult
- **Preconditions:** As TC-REG-001.
- **Steps:**
  1. Enter DOB 2000-02-29.
  2. Submit with valid data.
- **Expected result:** DOB is accepted as a valid date, age is computed as 26, account is created.

### TC-REG-006: Very old but plausible DOB
- **Preconditions:** As TC-REG-001.
- **Steps:**
  1. Enter DOB 1930-01-01 (age 96).
  2. Submit with valid data.
- **Expected result:** Account is created (subject to open question Q7 on a maximum age or earliest date). (*)

---

## 2. Negative scenarios

### TC-REG-101: Underage user (clearly under 18) is rejected
- **Preconditions:** As TC-REG-001.
- **Steps:**
  1. Enter DOB 2015-06-10 (age 11).
  2. Submit with otherwise valid data.
- **Expected result:** Account is NOT created. A clear message explains that users must be 18 or older. No user record, no confirmation email.

### TC-REG-102: User who turns 18 tomorrow is rejected (boundary)
- **Preconditions:** As TC-REG-001. Today = 2026-10-19.
- **Steps:**
  1. Enter DOB 2008-10-20.
  2. Submit with valid data.
- **Expected result:** Account is NOT created. Age restriction message shown.

### TC-REG-103: User aged 17 years 364 days is rejected (boundary)
- **Preconditions:** As TC-REG-102.
- **Steps:**
  1. Enter DOB 2008-10-20 (one day short of 18).
  2. Submit.
- **Expected result:** Rejected, as in TC-REG-102. Verifies that rounding (e.g. dividing days by 365) does not let this user through.

### TC-REG-104: Age is computed in calendar years, not by year subtraction
- **Preconditions:** As TC-REG-001. Today = 2026-10-19.
- **Steps:**
  1. Enter DOB 2008-12-31 (2026 minus 2008 = 18, but the birthday has not occurred this year; real age 17).
  2. Submit.
- **Expected result:** Rejected. Age is 17.

### TC-REG-105: Birth date in the future is rejected
- **Preconditions:** As TC-REG-001.
- **Steps:**
  1. Enter DOB one day after today (2026-10-20).
  2. Submit.
- **Expected result:** Rejected with a validation error indicating an invalid date (not necessarily the age message). No account created.

### TC-REG-106: Date of birth left empty
- **Preconditions:** As TC-REG-001.
- **Steps:**
  1. Leave the DOB field empty.
  2. Fill other fields validly and submit.
- **Expected result:** Submission blocked with a "date of birth is required" message. No account created.

### TC-REG-107: Non-existent calendar dates
- **Preconditions:** As TC-REG-001.
- **Steps:** Try each DOB value in turn and submit: 2001-02-30, 2001-13-01, 2001-00-10, 2001-04-31, 1999-02-29 (1999 is not a leap year).
- **Expected result:** Each is rejected with an invalid-date message. No account created, no server error (no HTTP 5xx).

### TC-REG-108: Non-date input
- **Preconditions:** As TC-REG-001.
- **Steps:** Enter each of: "abc", "00/00/0000", "18", a string of special characters, `<script>alert(1)</script>`, an SQL fragment such as `' OR 1=1 --`. Submit.
- **Expected result:** Rejected with a validation error. Input is not executed or reflected unescaped. No account created.

### TC-REG-109: Wrong date format
- **Preconditions:** As TC-REG-001. Know the accepted format (see Q4).
- **Steps:**
  1. Enter an adult DOB in the opposite format, e.g. 05/15/1990 vs 15/05/1990 where the product expects the other one; also 1990/5/15 and 15-5-90.
  2. Submit.
- **Expected result:** Either rejected with a format hint, or interpreted exactly as the documented format. It must never be silently misinterpreted (e.g. 03/04/2010 read as 4 March vs 3 April, which changes eligibility near the cutoff). (*)

### TC-REG-110: Two-digit year handling
- **Preconditions:** As TC-REG-001.
- **Steps:**
  1. Enter DOB with a two-digit year, e.g. 15/05/90 and 15/05/10.
  2. Submit.
- **Expected result:** Rejected as ambiguous, or interpreted per a documented rule. Must not create an account for someone born in 2010 by treating "10" as 1910. (*)

### TC-REG-111: Client-side check bypassed (server-side enforcement)
- **Preconditions:** Ability to send requests directly to the registration API (e.g. via API client or browser dev tools).
- **Steps:**
  1. Submit a registration request with an underage DOB (2015-06-10) directly to the endpoint, skipping the UI validation.
  2. Repeat with the DOB field omitted, null, and an empty string.
- **Expected result:** The server rejects every request with a 4xx validation error. No account created.

### TC-REG-112: Age rule cannot be bypassed by editing the DOB after validation
- **Preconditions:** As TC-REG-001.
- **Steps:**
  1. Enter an adult DOB, and let the form validate.
  2. Change the DOB to an underage value without leaving the field (or via browser back/forward, or autofill) and submit.
- **Expected result:** The final submitted value is validated; the underage submission is rejected.

### TC-REG-113: Retry after rejection with an adult DOB
- **Preconditions:** A user was just rejected for being underage (TC-REG-101), same session.
- **Steps:**
  1. On the same form, change the DOB to an adult DOB.
  2. Submit.
- **Expected result:** Behaviour per Q6 (retry allowed or blocked). Until confirmed, record observed behaviour; if retry is allowed, account is created normally. (*)

### TC-REG-114: Rejected underage attempt leaves no data behind
- **Preconditions:** Access to the database or admin view/API.
- **Steps:**
  1. Attempt registration with an underage DOB and a unique email.
  2. Check user store, email logs, and any partially created records.
  3. Then register that same email with an adult DOB.
- **Expected result:** No account, partial profile, or confirmation email exists after the rejected attempt. The email is still available afterwards for a valid registration (unless Q6 says otherwise).

---

## 3. Boundary and date-edge scenarios

### TC-REG-201: Leap-day birthday, 18th anniversary in a non-leap year (see Q3)
- **Preconditions:** Server date = 2026-02-28 (set via test clock).
- **Steps:**
  1. Enter DOB 2008-02-29 is invalid (2008 IS a leap year, so use DOB 2008-02-29, valid). Submit on 2026-02-28 and again on 2026-03-01.
- **Expected result:** On 2026-02-28 the user is 17, so rejected. On 2026-03-01 the user is 18, so accepted. Behaviour for 2026-02-28 vs 2026-03-01 is the documented rule for leap-day births (Q3); the 02-28 result is not confirmed by the requirement. (*)

### TC-REG-202: Registration crossing midnight
- **Preconditions:** Test clock controllable. User's 18th birthday is "tomorrow".
- **Steps:**
  1. Load the form at 23:59:30 on the day before the 18th birthday with DOB equal to the 18th birthday's date.
  2. Submit after the clock passes 00:00:00.
- **Expected result:** Age is evaluated at submission time, so the account is created. Submitting before midnight is rejected. (*)

### TC-REG-203: Time zone boundary
- **Preconditions:** Test clock and ability to set the user's browser time zone different from the server's. DOB's 18th birthday = today in server time but still yesterday in user time zone (or vice versa).
- **Steps:**
  1. Enter that DOB and submit.
  2. Repeat with browser time zone set ahead of and behind the server time zone.
- **Expected result:** Outcome follows the single documented reference time zone (Q2). Result is consistent regardless of the user's browser time zone. (*)

### TC-REG-204: Earliest and latest representable dates
- **Preconditions:** As TC-REG-001.
- **Steps:** Submit DOB 0001-01-01, 1900-01-01, 1899-12-31, and 9999-12-31.
- **Expected result:** Far-future dates are rejected as invalid. Far-past dates follow the rule defined in Q7; no crash, overflow or server error in any case. (*)

### TC-REG-205: Date input via date picker and via typing give identical results
- **Preconditions:** UI has a date picker and a text entry (if both exist).
- **Steps:**
  1. Select DOB 2008-10-19 with the picker, submit; repeat typing it.
  2. Repeat with 2008-10-20.
- **Expected result:** Both input methods give the same result: accepted for 2008-10-19, rejected for 2008-10-20.

---

## 4. Non-functional and UX checks (derived only from the stated requirement)

### TC-REG-301: Error message wording and accessibility
- **Preconditions:** As TC-REG-001.
- **Steps:**
  1. Submit an underage DOB.
  2. Inspect the message (visual, screen reader, keyboard focus).
- **Expected result:** The message is clear, associated with the DOB field, announced to assistive technology, and does not disclose internal details. Exact wording to be confirmed (Q8).

### TC-REG-302: DOB is not exposed in URLs or logs
- **Preconditions:** Access to network traces and server logs.
- **Steps:**
  1. Register with a valid DOB.
  2. Inspect the request URL, query string and logs.
- **Expected result:** DOB is sent in the request body over HTTPS, not in the URL. Handling in logs per privacy policy (Q9).

---

## 5. Open questions (requirement is ambiguous or incomplete)

1. **Q1: How is "aged 18 or older" calculated?** Completed years on the day of registration is assumed, with the user turning 18 on their birthday (inclusive). Please confirm the birthday itself counts as 18.
2. **Q2: Which time zone and clock define "today"?** Server UTC, the company's local time, or the user's local time? This changes the outcome for users within a day of their 18th birthday.
3. **Q3: When does a person born on 29 February turn 18 in a non-leap year?** On 28 Feb or 1 Mar?
4. **Q4: What date format(s) are accepted, and is it a date picker, free text, or three separate fields (day/month/year)?** Is there any localisation?
5. **Q5: What exactly happens to a rejected user?** Which message text is shown, is the form kept, is the attempt logged, and is any data (email, DOB) retained or discarded (privacy/GDPR/COPPA considerations)?
6. **Q6: Can a rejected user retry with a different DOB, immediately or after a lockout?** Is there any control against repeated attempts to find a passing date (rate limiting, device or IP block, cookie)?
7. **Q7: Is there a minimum date or maximum age (e.g. reject DOB before 1900 or age over 120)?** What should happen for implausible but valid dates?
8. **Q8: What are the exact required error messages, and are they to be localised?** Different messages for underage, invalid date, future date, missing date?
9. **Q9: Is the DOB stored, and if so is it stored as a date or only as an "over 18" flag?** Retention, encryption, and visibility to the user afterward?
10. **Q10: Is age verified in any way beyond self-declaration** (ID check, parental consent for minors)? The requirement implies self-declared only; please confirm.
11. **Q11: Does the rule apply to all registration channels** (web, mobile apps, API, social login/SSO, admin-created accounts, invitations)? With social login the DOB may come from a third party or be absent.
12. **Q12: Should age be re-checked at later events** (login, profile DOB edit, existing accounts created before this rule)? Can a user edit their DOB later to a value that makes them underage or adult?
13. **Q13: Is the minimum age jurisdiction dependent** (e.g. 16 or 21 in some regions)? The requirement states a flat 18.
14. **Q14: Is DOB mandatory for all users, and what is the behaviour when it is missing** (required-field error vs treated as underage)?

---

## Coverage summary

| Area | Test cases |
|---|---|
| Positive | TC-REG-001 to 006 |
| Negative (invalid age, invalid input, bypass, state) | TC-REG-101 to 114 |
| Boundary / date edges | TC-REG-002, 003, 102, 103, 104, 201 to 205 |
| Non-functional | TC-REG-301, 302 |

Cases marked (*) depend on answers to the open questions and must be re-confirmed before execution. All dates assume today = 2026-10-19 and must be recalculated, or run with a controllable test clock, on any other day.
