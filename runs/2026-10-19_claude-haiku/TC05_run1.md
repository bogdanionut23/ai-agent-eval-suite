# TC05 - Test cases: password reset by email

Requirement: A user who forgot the password can request a reset link by email. The link is valid for 60 minutes and can be used only once. The new password must have at least 10 characters.

Assumed test data: a registered account `user@example.com` with a known current password `OldPass!2026`. A second account `other@example.com` is used for isolation checks. Inbox access is available for the tester (or a test mail catcher).

## Positive scenarios

### TC05-01 - Request a reset link for a registered email
- Preconditions: `user@example.com` is a registered, active account. No reset request is pending.
- Steps:
  1. Open the login page and choose "Forgot password".
  2. Enter `user@example.com` and submit.
  3. Open the inbox of `user@example.com`.
- Expected result: The confirmation message is shown. An email with a reset link arrives in the inbox.

### TC05-02 - Reset the password using a valid link within 60 minutes
- Preconditions: TC05-01 done. Reset email received less than 60 minutes ago and not used.
- Steps:
  1. Click the reset link in the email.
  2. Enter a new password `NewPass!2026X` (14 characters) in both fields.
  3. Submit the form.
  4. Log in with `user@example.com` and `NewPass!2026X`.
- Expected result: The password is changed and a success message is shown. Login with the new password succeeds. Login with `OldPass!2026` fails.

### TC05-03 - New password of exactly 10 characters is accepted (lower boundary)
- Preconditions: Valid, unused reset link, less than 60 minutes old.
- Steps:
  1. Open the reset link.
  2. Enter `Abcdefg1!x` (exactly 10 characters) in both fields and submit.
  3. Log in with the new password.
- Expected result: The password is accepted and changed. Login succeeds.

### TC05-04 - Reset link works at 59 minutes (inside the validity window)
- Preconditions: Reset link generated; the clock or timestamp is such that 59 minutes have passed since generation.
- Steps:
  1. Open the link at T+59 minutes.
  2. Submit a valid new password.
- Expected result: The reset form is shown and the password is changed.

### TC05-05 - Repeated reset requests: the latest link is valid
- Preconditions: Two reset requests made for the same account, the second after the first.
- Steps:
  1. Open the link from the second email and reset the password.
- Expected result: The reset succeeds. (See open question Q3 about whether the first link must stop working.)

## Negative scenarios

### TC05-06 - Request with an unregistered email
- Preconditions: `nobody@example.com` is not registered.
- Steps:
  1. Choose "Forgot password" and enter `nobody@example.com`.
  2. Submit.
- Expected result: The response is the same as for a registered email (no account-existence disclosure, see Q1). No reset email is sent to that address. No error reveals whether the account exists.

### TC05-07 - Request with a malformed email address
- Preconditions: None.
- Steps:
  1. Enter `user@` (and separately `user.example.com`) in the forgot-password field and submit.
- Expected result: Validation message is shown, no request is sent, no email is sent.

### TC05-08 - Link expired after 60 minutes
- Preconditions: Reset link generated at T. Current time is T+60 minutes plus 1 second.
- Steps:
  1. Open the reset link.
  2. Enter a valid new password and submit.
- Expected result: The link is rejected with an "expired link" message. Password is not changed. User is offered to request a new link.

### TC05-09 - Link reused after a successful reset (single use)
- Preconditions: TC05-02 completed with link L.
- Steps:
  1. Open link L again.
  2. Enter a new valid password and submit.
- Expected result: Link is rejected as already used. Password is not changed by this attempt.

### TC05-10 - Link used a second time while still within 60 minutes, after a failed attempt
- Preconditions: Link L generated. A submit with an invalid password (9 characters) was made with L.
- Steps:
  1. Open link L again and submit a valid 10-character password.
- Expected result: Behaviour must be defined (see Q5). If the link is considered consumed only on successful reset, the reset succeeds. Test records the actual result against the open question.

### TC05-11 - New password of 9 characters is rejected (just below boundary)
- Preconditions: Valid, unused link.
- Steps:
  1. Open the reset link.
  2. Enter `Abcdefg1!` (9 characters) in both fields and submit.
- Expected result: Form shows an error stating the minimum length is 10. Password is not changed. Link remains usable (see Q5).

### TC05-12 - New password of 0 characters
- Preconditions: Valid, unused link.
- Steps:
  1. Open the reset link.
  2. Leave the password fields empty and submit.
- Expected result: Required-field error shown. Password not changed.

### TC05-13 - New password of 1 character and of 5 characters
- Preconditions: Valid, unused link.
- Steps:
  1. Enter `a` and submit. Repeat with `abcde`.
- Expected result: Length error shown for both. Password not changed.

### TC05-14 - Password and confirmation do not match
- Preconditions: Valid, unused link.
- Steps:
  1. Enter `NewPass!2026X` in the first field and `NewPass!2026Y` in the second. Submit.
- Expected result: Mismatch error shown. Password not changed. (Only if the form has a confirmation field; see Q6.)

### TC05-15 - Reset link for account A used to change account B
- Preconditions: Reset link L generated for `user@example.com`.
- Steps:
  1. Tamper with the link: replace the account identifier or token with that of `other@example.com`.
  2. Open the tampered link and submit a new password.
- Expected result: Request is rejected. Neither account is changed.

### TC05-16 - Tampered or random token
- Preconditions: None.
- Steps:
  1. Open the reset URL with the token altered by one character.
  2. Open the URL with an empty token and with a random 64-character string.
- Expected result: Each is rejected with an invalid-link message. No password form is shown or the submit fails. No error stack trace is shown.

### TC05-17 - Link is not usable for login
- Preconditions: Valid reset link.
- Steps:
  1. Try to use the reset link's token as a password or in the login form.
- Expected result: Login fails. Token is not a valid credential.

### TC05-18 - Old password still works if reset is never completed
- Preconditions: Reset requested, email received, link never opened.
- Steps:
  1. Log in with `OldPass!2026`.
- Expected result: Login succeeds. A pending reset request does not change the password by itself.

### TC05-19 - Reset with the account's current password (same password)
- Preconditions: Valid, unused link. Account password is `OldPass!2026`.
- Steps:
  1. Open the link and enter `OldPass!2026` in both fields. Submit.
- Expected result: Behaviour depends on policy (see Q7). Test records the actual result.

### TC05-20 - Rapid repeated reset requests (abuse)
- Preconditions: Account exists.
- Steps:
  1. Submit the forgot-password form 20 times in one minute for the same address.
- Expected result: Rate limit applies after the defined threshold (threshold per Q4). Responses do not reveal account existence. Excess requests do not send further emails.

## Boundary scenarios

### TC05-21 - Validity boundary at 60:00
- Preconditions: Link generated at T.
- Steps:
  1. Open the link at T+59:59 and submit a valid password. Repeat with a fresh link at T+60:00 and with another at T+60:01.
- Expected result: T+59:59 accepted. T+60:01 rejected. T+60:00 is accepted or rejected consistently with the agreed rule (see Q2); the rule must be documented and matched.

### TC05-22 - Password length boundary: 9, 10, 11 characters
- Preconditions: Valid, unused link for each run.
- Steps:
  1. Submit `Abcdefg1!` (9). Expect rejection.
  2. Submit `Abcdefg1!x` (10). Expect acceptance.
  3. Submit `Abcdefg1!xy` (11). Expect acceptance.
- Expected result: As stated in each step.

### TC05-23 - Length counted in characters, not bytes
- Preconditions: Valid, unused link.
- Steps:
  1. Submit a 10-character password containing multi-byte characters, e.g. `ÄÖÜßéèàçñ` plus `ab` (10 characters, more than 10 bytes).
  2. Submit a 9-character password of multi-byte characters, e.g. `ÄÖÜßéèàçñ` plus `a` (9 characters).
- Expected result: First is accepted, second rejected. Length is measured in characters (see Q8).

### TC05-24 - Leading and trailing spaces count toward length
- Preconditions: Valid, unused link.
- Steps:
  1. Submit `  Abcdefg1` (2 leading spaces + 8 characters = 10). Expect the stored value to be checked for length including spaces.
  2. Submit `         ` (9 spaces + 1 character) .
- Expected result: Spaces are counted as characters (or trimmed, per Q8). Outcome matches the agreed rule. Password is not silently trimmed without rule.

### TC05-25 - Very long password
- Preconditions: Valid, unused link.
- Steps:
  1. Submit a password of 128 characters. Then 1000 characters.
- Expected result: Either accepted, or rejected with a clear maximum-length message (maximum not specified, see Q9). No server error (5xx), no truncation without notice.

### TC05-26 - Link valid at the exact moment of first open
- Preconditions: Link generated at T.
- Steps:
  1. Open the link at T+1 second.
- Expected result: Link opens the reset form.

## Open questions

These points are not defined in the requirement. Test expectations for them are not invented above; they must be answered before the tests are final.

- Q1. Account enumeration: for an unregistered email, must the response be identical to the registered case? The requirement does not say. TC05-06 assumes yes.
- Q2. Exact expiry rule: is the 60-minute limit measured from link creation or from the email being sent? Is 60:00 itself valid or expired (TC05-21)?
- Q3. Multiple requests: does a new reset request invalidate earlier links? Requirement is silent. TC05-05 only checks the latest link.
- Q4. Rate limiting: is there a limit on reset requests per address or per IP, and what is it? (TC05-20 cannot have a pass/fail threshold without it.)
- Q5. Failed submits: is the link consumed by an attempt that fails validation (e.g. 9-character password), or only by a successful reset? Affects TC05-10 and TC05-11.
- Q6. Confirmation field: does the form have a second "confirm password" field? TC05-14 depends on this.
- Q7. Password reuse: may the new password equal the old one? Is there a history rule? TC05-19 depends on this.
- Q8. Length rules: is length counted in characters or bytes; are leading/trailing spaces trimmed before counting; is Unicode allowed?
- Q9. Maximum length and character classes: is there an upper bound? Are complexity rules (upper/lower/digit/symbol) required beyond the 10-character minimum? The requirement states only a minimum.
- Q10. Session effects: after a successful reset, must existing sessions/tokens of the account be logged out? Is the user logged in automatically after reset?
- Q11. Email delivery: what is the expected delivery time? Is the link single-domain or can it be sent to any address the user typed? What is the sender and link format?
- Q12. Link token: is the link bound to the browser/device that requested it, or usable anywhere? TC05-02 assumes it works in a different browser; this needs confirmation.
- Q13. Inactive or locked accounts: can a locked or unverified account reset its password? Requirement is silent.
- Q14. Error messages: exact wording for expired, used, invalid and too-short cases is not specified; tests check the category, not the text.
