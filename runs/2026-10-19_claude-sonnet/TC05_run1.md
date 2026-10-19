# TC05 - Password reset by email: test cases

Requirement: A user who forgot the password can request a reset link by email. The link is valid for 60 minutes and can be used only once. The new password must have at least 10 characters.

Common preconditions (unless a test says otherwise): application reachable; a registered, active user U1 with email u1@example.com; test mailbox accessible; system clock controllable or time can be waited/mocked.

## A. Requesting a reset link

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC05-01 | Request reset link with registered email (positive) | U1 exists | 1. Open the login page, choose "Forgot password". 2. Enter u1@example.com. 3. Submit. | Confirmation message is shown. An email with a reset link arrives in U1's mailbox. |
| TC05-02 | Reset link email content | As TC05-01 | 1. Request a reset. 2. Open the received email. | Email is addressed to U1, contains a working reset link (HTTPS), and states the link validity (if the product shows it). The old password is not included. |
| TC05-03 | Request with unregistered email (negative) | u-unknown@example.com not registered | 1. Request a reset for this email. | No email is sent. The user-facing response does not reveal whether the account exists (see open question Q3); no error/crash. |
| TC05-04 | Request with empty email (negative) | None | 1. Open the forgot-password form. 2. Leave email empty. 3. Submit. | Validation error; no request is created, no email is sent. |
| TC05-05 | Request with malformed email (negative) | None | 1. Enter "user@", "user.example.com", "a b@x.com". 2. Submit each. | Validation error for each; no email sent. |
| TC05-06 | Email case-insensitivity | U1 registered as u1@example.com | 1. Request a reset using U1@EXAMPLE.COM. | Behaviour consistent with login email matching (see Q4); expected: link is sent. |
| TC05-07 | Leading/trailing spaces in email | U1 exists | 1. Enter " u1@example.com ". 2. Submit. | Input is trimmed and the link is sent, or a clear validation error is shown (see Q4). |
| TC05-08 | Request reset twice in a row | U1 exists | 1. Request a reset. 2. Request again immediately. 3. Try link 1, then link 2. | Behaviour matches the defined rule for multiple links (see Q5). No crash; no email-sending error. |
| TC05-09 | Request reset for locked/disabled/unverified account | U1 disabled (or unverified) | 1. Request a reset. | Behaviour as defined (see Q6). Must not reveal account status to an anonymous user. |
| TC05-10 | SQL/script injection in email field (negative, security) | None | 1. Enter `' OR 1=1 --` and `<script>alert(1)</script>@x.com`. 2. Submit. | Input rejected or safely handled; no script executed, no DB error, no email sent. |

## B. Link validity: 60 minutes (boundary)

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC05-11 | Use link immediately | Link L1 issued at T0 | 1. Open L1 at T0+1 min. 2. Set a valid new password. | Reset form is shown and password is changed. |
| TC05-12 | Use link at 59 minutes (boundary, inside) | L1 issued at T0 | 1. Open L1 at T0+59:00. 2. Submit a valid new password. | Accepted; password changed. |
| TC05-13 | Use link at exactly 60:00 (boundary) | L1 issued at T0 | 1. Open/submit L1 at T0+60:00. | Outcome depends on inclusive/exclusive definition (see Q1). Result must be deterministic and documented. |
| TC05-14 | Use link at 60:01 (boundary, outside) | L1 issued at T0 | 1. Open L1 at T0+60:01. | Link rejected with an "expired" message; offer to request a new link; password unchanged. |
| TC05-15 | Use link long after expiry (e.g. 24 h) | L1 issued 24 h ago | 1. Open L1. | Rejected as expired; password unchanged. |
| TC05-16 | Link opened before expiry, form submitted after expiry | L1 issued at T0 | 1. Open L1 at T0+55 min (form shown). 2. Submit the new password at T0+65 min. | Submission rejected as expired (see Q2); password unchanged. |
| TC05-17 | Expiry independent of time zone/DST | Server and client in different time zones, or DST change within the window | 1. Request link. 2. Use it at T0+59 min across the change. | Validity is measured in absolute elapsed time (60 min), not affected by time zone. |

## C. Single use

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC05-18 | Link works once | L1 valid | 1. Open L1, set valid new password, submit. | Password changed, success message. |
| TC05-19 | Reuse link after successful reset (negative) | L1 already used in TC05-18 | 1. Open L1 again. | Rejected: link already used/invalid; no form shown. |
| TC05-20 | Reuse link by POSTing directly | L1 used | 1. Replay the reset request (token + new password) via API/HTTP client. | Rejected; password unchanged. |
| TC05-21 | Open link but do not submit, then reopen | L1 valid | 1. Open L1, close the page without submitting. 2. Open L1 again within 60 min. | Link still works (not consumed by viewing only) - confirm in Q7. |
| TC05-22 | Submit invalid password, then retry with same link | L1 valid | 1. Open L1. 2. Submit a 5-char password (rejected). 3. Submit a valid password. | Link is not consumed by the failed attempt; second submission succeeds (confirm in Q7). |
| TC05-23 | Concurrent use of same link | L1 valid; two browsers | 1. Open L1 in both. 2. Submit valid, different passwords in both nearly simultaneously. | Exactly one succeeds; the other is rejected. Final password equals the successful one. |
| TC05-24 | Old password after reset | L1 used successfully | 1. Log in with old password. 2. Log in with new password. | Old password fails; new password works. |
| TC05-25 | Tampered/invalid token | None | 1. Open the reset URL with a modified token, an empty token, a random string. | Rejected with a generic invalid-link message; no information leak. |
| TC05-26 | Link of user A cannot reset user B | Two users A and B | 1. Take A's link and alter any user identifier in the URL to B. | Rejected; B's password unchanged. |

## D. New password length: minimum 10 characters (boundary)

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC05-27 | Password with exactly 10 characters (boundary, valid) | L1 valid | 1. Open L1. 2. Enter "Abcdef1234" in both fields. 3. Submit. | Accepted; password changed. |
| TC05-28 | Password with 9 characters (boundary, invalid) | L1 valid | 1. Enter "Abcdef123" (9). 2. Submit. | Rejected with a message stating the minimum length; password unchanged; link still usable (Q7). |
| TC05-29 | Password with 11 characters | L1 valid | 1. Enter "Abcdef12345". | Accepted. |
| TC05-30 | Empty password | L1 valid | 1. Submit with both fields empty. | Validation error; password unchanged. |
| TC05-31 | Long password (e.g. 128 and 1000 chars) | L1 valid | 1. Enter a very long password. | Accepted or rejected according to a defined maximum (see Q8); no crash/truncation without notice. |
| TC05-32 | 10 spaces only | L1 valid | 1. Enter 10 space characters. | Behaviour per Q9 (whitespace handling); expected to be rejected or clearly defined. |
| TC05-33 | Length counted in characters, not bytes | L1 valid | 1. Enter 10 characters including non-ASCII (e.g. "éèàùçöüñßø"). 2. Then 9 such characters. | 10 characters accepted, 9 rejected (confirm Q9). |
| TC05-34 | Special characters and emoji | L1 valid | 1. Enter "P@ss w0rd!#" and a password containing emoji. | Behaviour per Q9; user can subsequently log in with the exact password. |
| TC05-35 | Password confirmation mismatch (if confirm field exists) | L1 valid | 1. Enter different values in password and confirmation. | Rejected with mismatch error; password unchanged. |
| TC05-36 | New password equals old password | L1 valid | 1. Enter the current password (10+ chars). | Behaviour per Q10 (not specified). |
| TC05-37 | Leading/trailing spaces preserved | L1 valid | 1. Set " Abcdef123 " (spaces included). 2. Log in with and without spaces. | Behaviour per Q9; must be consistent between reset and login. |

## E. Security and side effects

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC05-38 | Existing sessions after reset | U1 logged in on another device | 1. Complete a reset. 2. Use the other session. | Per Q11; recommended: other sessions invalidated. |
| TC05-39 | Notification email after password change | Reset done | 1. Check mailbox. | Per Q12 (not required by the requirement). |
| TC05-40 | Token not exposed | Reset flow | 1. Inspect page for referrer leakage, logs, URL after redirect. | Token not leaked to third parties; page not cached. |
| TC05-41 | Rate limiting of reset requests | None | 1. Submit 20 requests in one minute for the same email / same IP. | Per Q13; expect throttling and no email flood. |
| TC05-42 | Email delivery failure | Mail service down | 1. Request a reset. | Graceful behaviour; no crash; no token exposed to the user. |
| TC05-43 | UI usability and accessibility | None | 1. Walk the flow with keyboard only and a screen reader; check mobile width. | Fields labelled, errors announced, flow usable on mobile. |

## Open questions (requirement is ambiguous or incomplete; behaviour not invented)

1. Q1: Is the 60-minute limit inclusive or exclusive at exactly 60:00? And is it measured from request time or email sent time?
2. Q2: Is validity checked when the link is opened, when the form is submitted, or both?
3. Q3: What should the response be for an unregistered email? Same generic message (anti-enumeration) or an explicit "not found"?
4. Q4: Is email matching case-insensitive and are surrounding spaces trimmed?
5. Q5: If several links are requested, do older links become invalid, or are all valid until used/expired?
6. Q6: What happens for disabled, locked or unverified accounts, and for accounts that sign in only via SSO/social login?
7. Q7: Does "used" mean the link is consumed when opened, when a password submit is attempted, or only on a successful password change? Is a rejected (too short) password attempt counted as use?
8. Q8: Is there a maximum password length?
9. Q9: Other password rules besides the 10-character minimum (complexity, whitespace handling, Unicode and how length is counted, forbidden/breached passwords)?
10. Q10: May the new password equal the current or a previous password?
11. Q11: Should existing sessions/tokens be invalidated after a reset? Is the user logged in automatically afterwards?
12. Q12: Is a confirmation/notification email sent after the change?
13. Q13: Rate limits or lockouts for reset requests and for token guessing?
14. Q14: What are the exact UI messages, link format and email content/language (localization)?
15. Q15: Does a successful login with the old password count as "remembering" the password and should it cancel outstanding reset links?

## Coverage summary

- Positive: TC05-01, 02, 11, 12, 18, 24, 27, 29
- Negative: TC05-03, 04, 05, 09, 10, 14, 15, 16, 19, 20, 25, 26, 28, 30, 35
- Boundary: TC05-12, 13, 14, 27, 28, 29, 33
- Tests dependent on open questions are marked with the question number and should be finalized once answered.
