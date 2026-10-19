# Test Cases: Password Reset via Email Link

Requirement: A user who forgot the password can request a reset link by email. The link is valid for 60 minutes and can be used only once. The new password must have at least 10 characters.

Test data convention: `user@example.com` is a registered, active account. `unknown@example.com` is not registered. T0 is the moment the reset link is issued.

## 1. Test cases

### A. Requesting a reset link

| ID | Title | Type | Preconditions | Steps | Expected result |
|---|---|---|---|---|---|
| TC-PR-001 | Request reset link for a registered email | Positive | Account `user@example.com` exists and is active. Mailbox is accessible. | 1. Open the login page. 2. Click "Forgot password". 3. Enter `user@example.com`. 4. Submit. | A confirmation message is shown. An email arrives at `user@example.com` containing a reset link. |
| TC-PR-002 | Reset email content | Positive | As TC-PR-001, request already made. | 1. Open the received email. | The email contains a working, clickable reset link, comes from the expected sender, and states the validity period (60 minutes) if the product chooses to show it. |
| TC-PR-003 | Request with an unregistered email | Negative | `unknown@example.com` is not registered. | 1. Open the forgot-password form. 2. Enter `unknown@example.com`. 3. Submit. | No email is sent. The response shown to the user is defined by open question Q1 (neutral message vs. error). No stack trace or system error is exposed. |
| TC-PR-004 | Request with an empty email field | Negative | On the forgot-password form. | 1. Leave the email field empty. 2. Submit. | Validation error is shown. No request is processed, no email is sent. |
| TC-PR-005 | Request with a malformed email | Negative | On the forgot-password form. | 1. Enter `user@`, `user.example.com`, `user@@example.com`, `a b@example.com` (one per attempt). 2. Submit. | Each attempt is rejected with a format validation error. No email is sent. |
| TC-PR-006 | Email address case and whitespace handling | Boundary | Account `user@example.com` exists. | 1. Enter `USER@Example.com` and submit. 2. Repeat with leading and trailing spaces around the address. | Behaviour matches the product's email normalisation rule (see Q8). At minimum no error page appears and the result is consistent with TC-PR-001 or TC-PR-003 as defined. |
| TC-PR-007 | Request reset for a disabled, locked or unverified account | Negative | Account exists but is disabled, locked or email-unverified. | 1. Submit the forgot-password form with that email. | Behaviour per open question Q2. No unintended account state change. |
| TC-PR-008 | Repeated requests for the same account | Boundary | Account `user@example.com` exists. | 1. Request a reset link (link A). 2. Request again immediately (link B). 3. Try link A. 4. Try link B. | Behaviour per open question Q3 (is link A invalidated or do both stay valid?). Whatever the rule, it must be consistent and each link must still be single-use. |
| TC-PR-009 | Rate limiting on request endpoint | Negative | Account exists. | 1. Submit the form many times in a short period (for example 20 requests in 1 minute). | Behaviour per open question Q4. System must not send an unbounded number of emails. |
| TC-PR-010 | Request link while already logged in | Positive | User is logged in on another session. | 1. Open the forgot-password page. 2. Request a reset. | Behaviour per Q7 (page accessible or redirect). Request, if allowed, works as in TC-PR-001. |

### B. Link validity (60 minutes)

| ID | Title | Type | Preconditions | Steps | Expected result |
|---|---|---|---|---|---|
| TC-PR-011 | Use link immediately | Positive | Link issued at T0. | 1. Open the link at T0 + 1 min. | The reset-password form is displayed. |
| TC-PR-012 | Use link shortly before expiry | Boundary | Link issued at T0. Time can be controlled (test clock or wait). | 1. Open the link at T0 + 59 min 59 s. 2. Submit a valid new password. | Form is shown and the password is changed successfully. |
| TC-PR-013 | Use link at exactly 60 minutes | Boundary | Link issued at T0. | 1. Open the link at exactly T0 + 60 min 00 s. | Behaviour per open question Q5 (is the 60-minute boundary inclusive or exclusive?). Result must be deterministic. |
| TC-PR-014 | Use link just after expiry | Boundary | Link issued at T0. | 1. Open the link at T0 + 60 min 01 s. | An "expired link" message is shown. The reset form is not usable. Password is unchanged. |
| TC-PR-015 | Use link long after expiry | Negative | Link issued more than 24 hours ago. | 1. Open the link. | Expired message is shown. User is offered a way to request a new link. |
| TC-PR-016 | Expiry applies if form is opened in time but submitted late | Boundary | Link issued at T0. | 1. Open the link at T0 + 55 min (form shown). 2. Wait until T0 + 61 min. 3. Submit a valid new password. | Behaviour per open question Q6. The submission must not succeed silently in a way that contradicts the 60-minute rule. |
| TC-PR-017 | Expiry is measured from issue time, not from request-form time | Positive | Link issued at T0. | 1. Open the forgot-password page at T0 - 10 min and submit the request at T0. 2. Use the link at T0 + 55 min. | Link is accepted (validity counted from issue). |
| TC-PR-018 | Expiry independent of client clock and time zone | Negative | Link issued at T0. | 1. Change the device clock forward or backward by 2 hours. 2. Open the link at T0 + 30 min real time. | Validity is decided by server time. Link works at T0 + 30 min real time irrespective of the client clock. |

### C. Single use

| ID | Title | Type | Preconditions | Steps | Expected result |
|---|---|---|---|---|---|
| TC-PR-019 | Link works once | Positive | Valid, unused link. | 1. Open the link. 2. Enter a valid new password and confirm. 3. Submit. | Password is changed. Success message is shown. |
| TC-PR-020 | Reuse of a consumed link | Negative | Link already used successfully in TC-PR-019. | 1. Open the same link again. | "Link already used or invalid" message is shown. No password form. Password is unchanged by this action. |
| TC-PR-021 | Reuse via direct POST | Negative | Link already used. | 1. Send the password-change request with the used token directly to the API. | Request is rejected (4xx). Password unchanged. |
| TC-PR-022 | Opening the link without submitting does not consume it | Boundary | Valid unused link. | 1. Open the link, see the form. 2. Close the browser without submitting. 3. Open the link again within 60 min. | Behaviour per open question Q9 (is the link consumed on open or on successful submit?). The default expected reading is that the link is still usable. |
| TC-PR-023 | Failed submission (weak password) does not consume the link | Boundary | Valid unused link. | 1. Open the link. 2. Submit a 5-character password (rejected). 3. Submit a valid password on the same form. | Per Q9. Expected reading: the link remains valid until a successful reset. |
| TC-PR-024 | Concurrent use of the same link | Negative | Valid unused link opened in two browsers. | 1. Submit valid passwords in both browsers at nearly the same time. | Exactly one submission succeeds. The other is rejected as already used. Final password equals the one from the successful submission. |
| TC-PR-025 | Tampered or random token | Negative | None. | 1. Modify one character of the token in a valid link. 2. Open it. 3. Try a completely random token and an empty token. | Each is rejected with a generic "invalid link" message. No information leak about existing accounts or tokens. |
| TC-PR-026 | Token belongs to a different user | Negative | Two accounts, A and B, each with a link. | 1. Use A's token while attempting to change B's account (for example by altering a user-id parameter, if present). | Request is rejected. B's password is unchanged. |

### D. New password rule (minimum 10 characters)

| ID | Title | Type | Preconditions | Steps | Expected result |
|---|---|---|---|---|---|
| TC-PR-027 | Password of 10 characters (lower boundary) | Boundary | Valid link opened. | 1. Enter `Abcdef1234` (10 chars) in both fields. 2. Submit. | Accepted. Password is changed. (Assumes no complexity rule, see Q10.) |
| TC-PR-028 | Password of 9 characters | Boundary | Valid link opened. | 1. Enter `Abcdef123` (9 chars). 2. Submit. | Rejected with a message about minimum length. Password unchanged. |
| TC-PR-029 | Password of 11 characters | Boundary | Valid link opened. | 1. Enter `Abcdef12345` (11 chars). 2. Submit. | Accepted. |
| TC-PR-030 | Very short password | Negative | Valid link opened. | 1. Enter `a` (1 char). 2. Submit. | Rejected with length error. |
| TC-PR-031 | Empty password | Negative | Valid link opened. | 1. Leave the field empty. 2. Submit. | Rejected with a required-field error. |
| TC-PR-032 | Long password | Boundary | Valid link opened. | 1. Enter passwords of 64, 128 and 256 characters, one per attempt. | Behaviour per open question Q11 (maximum length). At least 64 characters must be accepted and must work for later login. No truncation silently occurs. |
| TC-PR-033 | Spaces only, 10 spaces | Negative | Valid link opened. | 1. Enter 10 space characters. 2. Submit. | Behaviour per Q10 (whether this counts as 10 characters). Must be handled consistently. |
| TC-PR-034 | Special characters and Unicode | Positive | Valid link opened. | 1. Enter `Pässwörd!#€ñ` and an emoji-containing 10-character password. 2. Submit. 3. Log in with the new password. | Length counted in characters per Q12. Accepted passwords work at login without corruption. |
| TC-PR-035 | Leading and trailing spaces in password | Boundary | Valid link opened. | 1. Enter ` Abcdefghi ` (spaces at both ends, 9 visible chars). 2. Submit. | Per Q13 (trimming). Must be consistent between reset and login. |
| TC-PR-036 | Confirmation field mismatch | Negative | Valid link opened, form has confirm field. | 1. Enter valid password in field 1 and a different one in field 2. 2. Submit. | Rejected with mismatch message. Password unchanged. Link not consumed (see Q9). |
| TC-PR-037 | New password equals the old password | Negative | Valid link opened. | 1. Enter the current password if it has at least 10 characters. 2. Submit. | Behaviour per open question Q14. |
| TC-PR-038 | Client-side bypass of length rule | Negative | Valid link opened. | 1. Disable client validation or send the API request directly with a 5-character password. | Server rejects the request. Server-side validation is enforced. |

### E. Post-reset behaviour and security

| ID | Title | Type | Preconditions | Steps | Expected result |
|---|---|---|---|---|---|
| TC-PR-039 | Log in with new password | Positive | Reset completed. | 1. Log in with the new password. | Login succeeds. |
| TC-PR-040 | Old password no longer works | Negative | Reset completed. | 1. Log in with the old password. | Login fails. |
| TC-PR-041 | Other active sessions after reset | Positive | User logged in elsewhere before reset. | 1. Complete a reset. 2. Use the other session. | Behaviour per open question Q15. |
| TC-PR-042 | Confirmation email after reset | Positive | Reset completed. | 1. Check the mailbox. | Behaviour per open question Q16. |
| TC-PR-043 | Other outstanding links after reset | Negative | Two links issued (see Q3), one used. | 1. Use the second link after the first reset. | Behaviour per Q3. Expectation to confirm: no outstanding link can be abused after a successful reset. |
| TC-PR-044 | Token not exposed in logs, referrer or URL after load | Negative | Valid link. | 1. Open the link. 2. Inspect network requests, referrer header on any outbound link on the page, and browser history. | Token is not leaked to third parties. Page sets no-referrer or equivalent. Token is not stored in plain text (inspect server storage if accessible). |
| TC-PR-045 | Reset link over HTTPS only | Negative | None. | 1. Replace `https` with `http` in the link. | Request is redirected to HTTPS or refused. |

## 2. Open questions (not assumed in the test cases above)

| # | Question | Why it matters |
|---|---|---|
| Q1 | When the email is not registered, should the UI show an error or a neutral message ("If the account exists, an email was sent")? | Security (account enumeration) versus usability. Drives TC-PR-003. |
| Q2 | What should happen for disabled, locked or unverified accounts? | Not covered by the requirement. |
| Q3 | If a user requests several links, is only the newest valid, or are all valid until each expires? Are outstanding links invalidated after a successful reset? | Affects TC-PR-008 and TC-PR-043. |
| Q4 | Is there rate limiting or a cooldown on requests, and what are the limits and messages? | Abuse and email flooding. |
| Q5 | Is the 60-minute limit inclusive or exclusive at exactly 60:00? What clock source is used (server UTC)? | Boundary test TC-PR-013. |
| Q6 | Is expiry checked when the form is opened, when it is submitted, or both? | TC-PR-016. |
| Q7 | Can a logged-in user use the forgot-password flow? | TC-PR-010. |
| Q8 | Is the email lookup case-insensitive, and is whitespace trimmed? | TC-PR-006. |
| Q9 | Is the link "used" when opened, or only when the password is changed successfully? Does a failed validation consume it? | "Used only once" is ambiguous. TC-PR-022, 023, 036. |
| Q10 | Beyond 10 characters, are there complexity rules (upper/lower case, digits, symbols), a blacklist of common passwords, or rules on whitespace-only passwords? | The requirement states only a minimum length. |
| Q11 | Is there a maximum password length? | Boundary at the upper end. TC-PR-032. |
| Q12 | How is length counted for Unicode (code points, UTF-16 units, graphemes, bytes)? | Emoji and accented characters. TC-PR-034. |
| Q13 | Are leading and trailing spaces in passwords trimmed or kept? | TC-PR-035. |
| Q14 | Must the new password differ from the current one or from previous passwords (history)? | TC-PR-037. |
| Q15 | Are other active sessions and remember-me tokens terminated after a reset? | Security expectation not stated. |
| Q16 | Is a notification email sent after the password is changed? | TC-PR-042. |
| Q17 | Is a confirm-password field required? | Affects TC-PR-036. |
| Q18 | What exact messages, languages and redirects apply (success page, expired page, auto-login after reset)? | Expected texts are not specified. |
| Q19 | Is there a defined email delivery time (for example within 1 or 5 minutes)? Does the 60 minutes start at request time or at email delivery? | Affects timing tests and the usability of the window. |
| Q20 | Does the reset flow support MFA or any additional verification? | Not mentioned. |

## 3. Notes

- Time-based cases (TC-PR-012 to TC-PR-016) should use a controllable test clock or a configurable token lifetime in a test environment rather than real waiting.
- Cases that depend on an open question list the expectation as "per Q#". They must be completed once the question is answered, and should not be executed with an invented expectation.
