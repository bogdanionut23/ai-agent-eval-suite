# TC05 - Test cases: Password reset by email

## Requirement

A user who forgot the password can request a reset link by email. The link is valid for 60 minutes and can be used only once. The new password must have at least 10 characters.

## Test cases

### Positive scenarios

**TC05-01 - Request a reset link for a registered email**
- Preconditions: A user account exists with a known registered email address. The user is logged out.
- Steps:
  1. Open the "Forgot password" page.
  2. Enter the registered email address.
  3. Submit the form.
- Expected result: The system confirms that a reset link has been sent. An email containing a reset link arrives at the registered address.

**TC05-02 - Reset password using a valid link within 60 minutes**
- Preconditions: TC05-01 has been executed. The reset email was received less than 60 minutes ago and has not been used.
- Steps:
  1. Open the reset link from the email.
  2. Enter a new password with 10 or more characters.
  3. Enter the same password in the confirmation field, if present.
  4. Submit the form.
- Expected result: The password is changed. The user sees a success message and can log in with the new password. The old password no longer works.

**TC05-03 - Login with new password after reset**
- Preconditions: TC05-02 completed successfully.
- Steps:
  1. Go to the login page.
  2. Log in with the email and the new password.
- Expected result: Login succeeds.

**TC05-04 - Reset link works at the 59th minute**
- Preconditions: A reset link was generated at time T.
- Steps:
  1. At T + 59 minutes, open the link and submit a valid new password.
- Expected result: The reset succeeds.

### Negative scenarios

**TC05-05 - Request reset for an unregistered email**
- Preconditions: The email address is not associated with any account.
- Steps:
  1. Open the "Forgot password" page.
  2. Enter the unregistered email address.
  3. Submit the form.
- Expected result: The response does not reveal whether the account exists. It shows the same message as TC05-01. No reset email is sent. (Behaviour depends on the open question in Q1.)

**TC05-06 - Request reset with malformed email**
- Preconditions: None.
- Steps:
  1. Enter `user@` or `userexample.com` in the email field.
  2. Submit the form.
- Expected result: Client-side or server-side validation rejects the input. No email is sent.

**TC05-07 - Use reset link after 60 minutes have passed**
- Preconditions: A reset link was generated at time T and not used.
- Steps:
  1. At T + 60 minutes and 1 second, open the link.
  2. Enter a valid new password and submit.
- Expected result: The link is rejected as expired. The password is not changed. The user is offered a way to request a new link.

**TC05-08 - Use reset link a second time**
- Preconditions: TC05-02 has been completed with a link.
- Steps:
  1. Open the same link again within the 60-minute window.
  2. Enter a valid new password and submit.
- Expected result: The link is rejected as already used. The password is not changed again.

**TC05-09 - Reset with password of 9 characters**
- Preconditions: A valid, unused, unexpired reset link.
- Steps:
  1. Open the link.
  2. Enter a password with exactly 9 characters (for example `Abcdefgh1`).
  3. Submit.
- Expected result: The form is rejected with a message stating the minimum length of 10 characters. The password is not changed. The link remains usable (see Q5).

**TC05-10 - Reset with empty password**
- Preconditions: A valid reset link.
- Steps:
  1. Leave the password field empty.
  2. Submit.
- Expected result: Validation error. The password is not changed.

**TC05-11 - Confirmation mismatch**
- Preconditions: A valid reset link. The form has a confirmation field.
- Steps:
  1. Enter a 12-character password.
  2. Enter a different value in the confirmation field.
  3. Submit.
- Expected result: Validation error. The password is not changed.

**TC05-12 - Tampered or invalid token**
- Preconditions: A valid reset link exists.
- Steps:
  1. Change one character of the token in the URL.
  2. Open the URL and submit a valid new password.
- Expected result: The link is rejected as invalid. The password is not changed. No error details of the token format are exposed.

**TC05-13 - Link from one account used for another**
- Preconditions: Two accounts A and B exist. A reset link was issued for account A.
- Steps:
  1. Modify the URL so that it refers to account B's identifier while keeping A's token.
  2. Submit a new password.
- Expected result: The request is rejected. Neither account's password is changed.

**TC05-14 - Previous link invalidated by a newer request**
- Preconditions: Two reset requests were made for the same account, link 1 first, then link 2. (Behaviour is an open question, see Q4.)
- Steps:
  1. Open link 1 within 60 minutes and submit a valid password.
- Expected result: Depends on Q4. If old links are invalidated, the request is rejected. If not, the reset succeeds and link 2 is then unusable (single use applies to the token, not the account).

**TC05-15 - Reset while logged in on another session**
- Preconditions: The user is logged in on device X. A valid reset link is available.
- Steps:
  1. Complete a password reset from device Y.
  2. Check the session on device X.
- Expected result: Depends on Q6. Requirement is silent; document the actual behaviour.

### Boundary scenarios

**TC05-16 - Password of exactly 10 characters**
- Preconditions: A valid reset link.
- Steps:
  1. Enter exactly 10 characters, for example `Abcdefghi1`.
  2. Submit.
- Expected result: The password is accepted and changed.

**TC05-17 - Password of 11 characters**
- Preconditions: A valid reset link.
- Steps:
  1. Enter 11 characters and submit.
- Expected result: The password is accepted and changed.

**TC05-18 - Password of 10 spaces**
- Preconditions: A valid reset link. (Whether spaces count is open question Q2.)
- Steps:
  1. Enter 10 space characters and submit.
- Expected result: Depends on Q2. If spaces are counted and no other rule applies, the password is accepted. Document the actual result.

**TC05-19 - Password of 10 characters with leading/trailing whitespace**
- Preconditions: A valid reset link.
- Steps:
  1. Enter ` abcdefgh ` (8 letters plus one space on each side, 10 characters total) and submit.
- Expected result: Depends on Q2 (whitespace trimming). Document the actual result. The stored password must match what the user typed, or the trimmed value, consistently with login.

**TC05-20 - Very long password**
- Preconditions: A valid reset link.
- Steps:
  1. Enter a password of 1000 characters and submit.
- Expected result: Depends on Q3 (maximum length). The system either accepts it or rejects it with a clear message. It must not crash or truncate silently.

**TC05-21 - Link validity at exactly 60 minutes**
- Preconditions: A reset link was generated at time T.
- Steps:
  1. At T + 60 minutes exactly, open the link and submit a valid password.
- Expected result: Depends on Q7 (whether the boundary is inclusive). Document the actual behaviour and make it consistent with the requirement.

**TC05-22 - Link validity at 60 minutes and 1 second**
- Covered by TC05-07. Listed here for the boundary pair with TC05-04 and TC05-21.

**TC05-23 - Unicode characters count as characters**
- Preconditions: A valid reset link.
- Steps:
  1. Enter 10 characters including multi-byte characters, for example `ÄÖÜßéèàçñ` plus two digits.
  2. Submit.
- Expected result: The password is counted by characters, not bytes, and is accepted.

### Security and non-functional scenarios

**TC05-24 - Reset link is not reusable after a successful login with the old password**
- Not in the requirement. Open question Q8. Do not execute until defined.

**TC05-25 - Reset email is sent once per request**
- Preconditions: A registered email.
- Steps:
  1. Submit the reset request once.
  2. Check the inbox.
- Expected result: Exactly one reset email is received.

**TC05-26 - Rate limiting on reset requests**
- Preconditions: Not defined in requirement. See Q9.
- Steps:
  1. Submit reset requests for the same email repeatedly, for example 20 times in one minute.
- Expected result: Depends on Q9.

**TC05-27 - Link transport security**
- Preconditions: A reset email is received.
- Steps:
  1. Inspect the link URL.
- Expected result: The link uses HTTPS. The token is random, not guessable, and is not logged in plain text.

## Open questions

These points are not defined in the requirement. The related test cases document the behaviour to be confirmed and do not assume it.

1. **Unregistered email response.** Should the system show the same confirmation for unknown emails (to avoid account enumeration) or an error? Affects TC05-05.
2. **Password character rules.** Does "at least 10 characters" count spaces, Unicode characters and leading/trailing whitespace? Is trimming applied? Affects TC05-18, TC05-19, TC05-23.
3. **Maximum password length.** Is there an upper limit? Affects TC05-20.
4. **Multiple outstanding links.** When a new reset is requested, are earlier links invalidated? Affects TC05-14.
5. **Failed validation and link consumption.** If the new password is rejected (for example 9 characters), is the link still valid, or is it consumed? Affects TC05-09.
6. **Existing sessions.** After a successful reset, are other active sessions for the account terminated? Affects TC05-15.
7. **Expiry boundary.** Is the 60-minute window inclusive or exclusive? Measured from link generation or from email send time? Which time zone applies if the server and client differ? Affects TC05-21.
8. **Password reuse.** Must the new password differ from the old one or from recent passwords? Not in the requirement.
9. **Rate limiting and lockout.** Is there a limit on reset requests per account or IP address, and what is the response when it is exceeded? Affects TC05-26.
10. **Password complexity.** Beyond length, are there requirements for character classes, a blocklist of common passwords, or a strength meter?
11. **Single-use scope.** Does "used only once" apply only to successful resets, or also to link opens? Does a link opened but not submitted count as used?
12. **Confirmation field.** Is a second password field required? Not stated in the requirement.
13. **Error messages on expired or used links.** What message and next step does the user get? Not stated.
14. **Email delivery time.** What is the maximum delay for the email to arrive? Does the 60-minute clock start before or after delivery?
