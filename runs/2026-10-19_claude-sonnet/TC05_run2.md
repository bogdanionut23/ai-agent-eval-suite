# Test Cases: Password Reset by Email Link

Requirement: A user who forgot the password can request a reset link by email. The link is valid for 60 minutes and can be used only once. The new password must have at least 10 characters.

Assumptions (kept minimal, nothing beyond the requirement): "60 minutes" is counted from the moment the link is issued. "Used" means the link was submitted successfully with a valid new password. Anything not stated is listed under Open Questions, and the related test cases are marked "(pending OQ-x)".

## Test Cases

### A. Requesting the reset link

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-01 | Request reset link with registered email (positive) | Registered user with a working mailbox. | 1. Open the "Forgot password" page. 2. Enter the registered email. 3. Submit. | The request is accepted and a confirmation message is shown. An email containing a reset link arrives. |
| TC-02 | Link in the email opens the reset form (positive) | A reset email was received (TC-01). | 1. Open the link from the email. | The page to set a new password is displayed. |
| TC-03 | Request with unregistered email (negative) | The email is not registered. | 1. Open "Forgot password". 2. Enter the unregistered email. 3. Submit. | No reset email is sent. The message shown follows the product decision in OQ-1 (pending OQ-1). |
| TC-04 | Request with empty email (negative) | None. | 1. Leave the email field empty. 2. Submit. | A validation error is shown. No request is created and no email is sent. |
| TC-05 | Request with malformed email (negative) | None. | 1. Enter "user@" (and separately "user.example.com"). 2. Submit. | A validation error is shown. No email is sent. |
| TC-06 | Email case and surrounding spaces (boundary) | Registered email is user@example.com. | 1. Enter " USER@Example.com " (uppercase, leading and trailing spaces). 2. Submit. | Behaviour matches the product decision in OQ-2 (pending OQ-2). |
| TC-07 | Two requests in a row (positive/boundary) | Registered user. | 1. Request a reset link. 2. Request again without using the first link. | Expected state of the first link is defined by OQ-3 (pending OQ-3). The newest link works. |

### B. Link validity: 60 minutes

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-08 | Use link right after receiving it (positive) | Fresh link issued at T0. | 1. Open the link at about T0 + 1 min. 2. Set a valid new password. 3. Submit. | The password is changed successfully. |
| TC-09 | Use link just before expiry (boundary) | Link issued at T0. Time can be controlled (test clock or time mock). | 1. Open the link and submit a valid new password at T0 + 59 min 59 s. | The password is changed successfully. |
| TC-10 | Use link exactly at 60 minutes (boundary) | Link issued at T0. Controllable time. | 1. Submit a valid new password at exactly T0 + 60 min 00 s. | Result depends on whether the limit is inclusive (pending OQ-4). Must be consistent with the documented rule. |
| TC-11 | Use link just after expiry (boundary, negative) | Link issued at T0. Controllable time. | 1. Submit a valid new password at T0 + 60 min 01 s. | The change is rejected with an "expired link" message. The password is unchanged. |
| TC-12 | Use link long after expiry (negative) | Link issued more than 24 h ago. | 1. Open the link. | An "expired link" message is shown. No password form that can succeed. The old password still works. |
| TC-13 | Link opened before expiry but form submitted after expiry (boundary) | Link issued at T0. Controllable time. | 1. Open the link at T0 + 55 min. 2. Wait until T0 + 61 min. 3. Submit a valid new password. | Expiry is checked at submission time (pending OQ-5). If so, the change is rejected. |
| TC-14 | Expiry is not extended by activity (negative) | Link issued at T0. | 1. Open the link several times between T0 and T0 + 50 min. 2. Try to use it at T0 + 61 min. | Rejected as expired. |
| TC-15 | Server and user time zones differ (boundary) | User device set to a different time zone or wrong clock. | 1. Request a link. 2. Change the device clock or time zone. 3. Use the link within 60 minutes real time. | Expiry is based on server time. The link works within the real 60 minutes, regardless of the device clock. |

### C. Single use

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-16 | Link cannot be reused after success (negative) | Link used successfully once. | 1. Open the same link again. | An "already used / invalid link" message is shown. The form is not usable. |
| TC-17 | Reuse attempt via direct submit (negative) | Link used successfully once. | 1. Replay the earlier password-change request (same token) with another valid password. | Rejected. The password stays the one set by the first use. |
| TC-18 | Link remains valid after only opening it (positive) | Fresh link. | 1. Open the link but do not submit. 2. Close the page. 3. Open the link again and submit a valid password. | Behaviour per OQ-6 (pending OQ-6). |
| TC-19 | Link remains valid after a failed attempt (boundary) | Fresh link. | 1. Open the link and submit a too-short password (rejected). 2. Submit a valid password with the same link. | Behaviour per OQ-6 (pending OQ-6). A rejected password must not be treated as a successful use unless defined otherwise. |
| TC-20 | Concurrent use of the same link (negative) | Fresh link, two browser sessions. | 1. Open the link in both sessions. 2. Submit valid passwords in both at about the same time. | Exactly one succeeds. The other is rejected as used. The final password is the one from the successful request. |
| TC-21 | Tampered or random token (negative) | None. | 1. Open the reset URL with a modified or random token. | An "invalid link" message is shown. No information about the account is leaked. |
| TC-22 | Link belongs to a specific user (negative) | Two users A and B with reset links. | 1. Take the token from A's link and combine it with B's identifier in the URL (if the URL contains one). | Rejected. B's password is not changed. |

### D. Password length (minimum 10 characters)

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-23 | Password with exactly 10 characters (boundary, positive) | Valid link. | 1. Enter "Abcdef1234" in both fields (new and confirm, if present). 2. Submit. | Accepted. Password changed. |
| TC-24 | Password with 9 characters (boundary, negative) | Valid link. | 1. Enter "Abcdef123". 2. Submit. | Rejected with a message about the minimum length. Password unchanged. The link is still usable per OQ-6. |
| TC-25 | Password with 11 characters (boundary, positive) | Valid link. | 1. Enter "Abcdef12345". 2. Submit. | Accepted. |
| TC-26 | Long password (positive/boundary) | Valid link. | 1. Enter a 64-character password. 2. Submit. | Accepted unless a maximum length exists (pending OQ-7). |
| TC-27 | Empty password (negative) | Valid link. | 1. Leave the field empty. 2. Submit. | Rejected with a validation message. |
| TC-28 | Spaces only, 10 spaces (negative/boundary) | Valid link. | 1. Enter 10 spaces. 2. Submit. | Behaviour per OQ-8 (pending OQ-8). |
| TC-29 | Password with special and Unicode characters, 10 characters in total (boundary) | Valid link. | 1. Enter a password such as "Pä$$wörd!1". 2. Submit. | Length is counted in characters. Accepted if allowed (pending OQ-8, OQ-9). |
| TC-30 | Emoji counted as characters (boundary) | Valid link. | 1. Enter 9 letters plus 1 emoji. 2. Submit. | Behaviour per OQ-9 (pending OQ-9). |
| TC-31 | Confirmation does not match (negative) | Valid link, form has a confirm field (pending OQ-10). | 1. Enter valid password in the first field and a different one in the confirm field. 2. Submit. | Rejected. Password unchanged. |

### E. After the reset

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-32 | Log in with the new password (positive) | Password just reset. | 1. Log in with the new password. | Login succeeds. |
| TC-33 | Old password no longer works (positive) | Password just reset. | 1. Log in with the old password. | Login fails. |
| TC-34 | Reset does not affect other accounts (negative) | Two users. | 1. Reset the password of user A. 2. Log in as user B with B's own password. | B is unaffected. |
| TC-35 | Unused pending link after another reset (negative) | Two links issued, one used. | 1. Use the first link. 2. Try the second. | Behaviour per OQ-3 (pending OQ-3). |

## Open Questions

The requirement does not define these points. No behaviour has been assumed for them.

1. OQ-1: What should the page show for an email that is not registered? A neutral message (same as for registered emails, to avoid revealing which accounts exist) or an explicit error?
2. OQ-2: Is the email compared case-insensitively? Are leading and trailing spaces trimmed?
3. OQ-3: When a user requests a new link while an older one is still valid, is the older link invalidated? Are several active links allowed at once? After a successful reset, are all other outstanding links invalidated?
4. OQ-4: Is the 60-minute limit inclusive or exclusive at exactly 60:00? From which moment is it counted (request, sending, or email delivery)?
5. OQ-5: Is expiry checked when the link is opened, when the form is submitted, or both?
6. OQ-6: What counts as "used"? Opening the link, submitting any password, or only a successful change? Does a rejected password (for example too short) consume the link?
7. OQ-7: Is there a maximum password length?
8. OQ-8: Are there other password rules besides length (uppercase, digits, special characters, spaces-only, common or previously used passwords, same as the old password)?
9. OQ-9: How are characters counted for Unicode, combined characters and emoji (code points, grapheme clusters, bytes)?
10. OQ-10: Does the form have a "confirm password" field?
11. OQ-11: Is there a limit on how many reset requests can be made per user, email or IP in a period (rate limiting)? What happens above it?
12. OQ-12: Are existing sessions and tokens logged out after a reset? Is the user notified by email that the password was changed?
13. OQ-13: Are locked, disabled or unverified accounts allowed to request a reset?
14. OQ-14: Who is the user-facing wording for expired, used and invalid links? Should the three cases be distinguished or shown with one generic message?
15. OQ-15: Is there a time limit for email delivery, and what should the user see if no email arrives (resend option)?
16. OQ-16: Is the new password allowed to equal the old one?

## Coverage Summary

- Positive: TC-01, 02, 08, 09, 23, 25, 26, 32, 33
- Negative: TC-03 to 05, 11, 12, 14, 16, 17, 20 to 22, 24, 27, 31, 34, 35
- Boundary: TC-06, 09 to 11, 13, 15, 19, 23 to 25, 28 to 30
- Cases marked "pending OQ-x" must not be given a fixed expected result until the question is answered.
