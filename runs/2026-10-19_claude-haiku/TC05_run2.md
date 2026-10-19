# TC05 Run 2: Test cases for the password reset requirement

## Requirement

A user who forgot the password can request a reset link by email. The link is valid for 60 minutes and can be used only once. The new password must have at least 10 characters.

## Test cases

### Positive

**TC-01: Request a reset link for a registered email**
- Preconditions: A user account exists with a registered email address. The user is logged out.
- Steps:
  1. Open the "Forgot password" page.
  2. Enter the registered email address.
  3. Submit the form.
  4. Open the reset email in the mailbox.
- Expected result: The form confirms that a reset link was sent. An email arrives at the registered address containing a reset link.

**TC-02: Reset the password with a valid link within 60 minutes**
- Preconditions: TC-01 completed. The reset link was generated less than 60 minutes ago and has not been used.
- Steps:
  1. Click the reset link in the email.
  2. Enter a new password of at least 10 characters (for example, `NewPass1234`) in both fields.
  3. Submit the form.
  4. Log in with the email address and the new password.
- Expected result: The password is changed. The reset succeeds and the login with the new password succeeds.

**TC-03: Reset link still works at 59 minutes**
- Preconditions: A reset link was generated and has not been used. Time is controlled or measured precisely.
- Steps:
  1. Wait until 59 minutes have passed since the link was generated (or move the server clock forward to that point).
  2. Click the link and submit a valid new password of at least 10 characters.
- Expected result: The link is accepted and the password is changed.

**TC-04: New password of exactly 10 characters is accepted**
- Preconditions: A valid, unused reset link.
- Steps:
  1. Open the reset link.
  2. Enter exactly 10 characters as the new password (for example, `Abcdefghi1`) in both fields.
  3. Submit the form.
- Expected result: The password is accepted and changed.

**TC-05: New password longer than 10 characters is accepted**
- Preconditions: A valid, unused reset link.
- Steps:
  1. Open the reset link.
  2. Enter a password of 11 or more characters.
  3. Submit the form.
- Expected result: The password is accepted and changed.

### Negative

**TC-06: Reset link is rejected after it is used once**
- Preconditions: TC-02 completed. The same link is still available to the tester.
- Steps:
  1. Open the same reset link a second time.
  2. Enter a new password of at least 10 characters and submit.
- Expected result: The request is refused with a clear message that the link has already been used. The password is not changed.

**TC-07: Reset link is rejected after 60 minutes**
- Preconditions: A reset link was generated and has not been used.
- Steps:
  1. Wait more than 60 minutes after the link was generated (or move the server clock forward past 60 minutes; for example, 60 minutes and 1 second).
  2. Click the link and submit a valid new password.
- Expected result: The link is refused with a message that it has expired. The password is not changed. The user is offered a way to request a new link.

**TC-08: New password with 9 characters is rejected**
- Preconditions: A valid, unused reset link.
- Steps:
  1. Open the reset link.
  2. Enter a password of exactly 9 characters (for example, `Abcdefgh1`) in both fields.
  3. Submit the form.
- Expected result: The form is refused with a validation message stating the minimum length of 10 characters. The password is not changed. The link is handled as described in the open questions (see Q-04).

**TC-09: Empty new password is rejected**
- Preconditions: A valid, unused reset link.
- Steps:
  1. Open the reset link.
  2. Leave the password fields empty and submit.
- Expected result: The form is refused with a required-field message. The password is not changed.

**TC-10: Password and confirmation do not match**
- Preconditions: A valid, unused reset link.
- Steps:
  1. Enter a 12-character password in the first field and a different 12-character password in the second field.
  2. Submit the form.
- Expected result: The form is refused with a mismatch message. The password is not changed.

**TC-11: Request a reset for an unregistered email**
- Preconditions: No account exists with the email address used.
- Steps:
  1. Open the "Forgot password" page.
  2. Enter the unregistered email address and submit.
- Expected result: Depends on the security policy, which the requirement does not define (see Q-01). At minimum, no reset link is sent to any other address, and the account state is not exposed.

**TC-12: Malformed or tampered reset link is rejected**
- Preconditions: A valid reset link has been generated.
- Steps:
  1. Change one character of the token in the link.
  2. Open the modified link and submit a valid new password.
  3. Separately, remove the token from the link and open it.
- Expected result: Both links are refused with a generic invalid-link message. The password is not changed.

**TC-13: Password change is blocked for a link generated for another user**
- Preconditions: Two accounts, A and B, each have a valid, unused reset link.
- Steps:
  1. Take the reset link of account A.
  2. Change the email or user identifier in the URL to account B, if the URL carries one.
  3. Submit a new password.
- Expected result: The request is refused. Neither account password is changed through the altered link.

### Boundary

**TC-14: Link at 60 minutes exactly**
- Preconditions: A reset link was generated and has not been used. Time is controlled.
- Steps:
  1. Advance the clock to exactly 60 minutes after generation.
  2. Open the link and submit a valid new password.
- Expected result: Depends on the boundary rule, which the requirement does not define (see Q-03). The test records the actual behaviour and the team confirms the expected one.

**TC-15: Link at 60 minutes and one second**
- Preconditions: As TC-14.
- Steps:
  1. Advance the clock to 60 minutes and 1 second after generation.
  2. Open the link and submit a valid new password.
- Expected result: The link is refused as expired.

**TC-16: Password length boundary values (9, 10, 11)**
- Preconditions: A new valid, unused reset link is generated for each run.
- Steps:
  1. Submit a password of 9 characters. Check the result.
  2. Submit a password of 10 characters with a new link. Check the result.
  3. Submit a password of 11 characters with a new link. Check the result.
- Expected result: 9 characters is refused. 10 and 11 characters are accepted.

**TC-17: Password length is counted in characters, not bytes**
- Preconditions: A valid, unused reset link.
- Steps:
  1. Enter exactly 10 non-ASCII characters (for example, `ááááááááá` with one more letter, or `ąęćśżźńółĄĘĆŚ`) and submit.
- Expected result: The password is counted by characters. A password of 10 characters is accepted even if its byte length is larger. The result is consistent with the length rule (see Q-06).

**TC-18: Password of 10 spaces**
- Preconditions: A valid, unused reset link.
- Steps:
  1. Enter 10 space characters in both fields and submit.
- Expected result: Depends on whether whitespace is trimmed or allowed (see Q-06). Recommended expectation: refused, as the password is not meaningful. The team confirms the rule.

**TC-19: Single-use under concurrency**
- Preconditions: A valid, unused reset link.
- Steps:
  1. Open the same link in two browser sessions.
  2. Submit a valid new password in both sessions at the same moment.
- Expected result: Exactly one request succeeds. The other is refused as already used. The stored password matches only the successful submission.

**TC-20: Failed submission does not consume the link**
- Preconditions: A valid, unused reset link.
- Steps:
  1. Submit a 9-character password (TC-08).
  2. Without requesting a new link, submit a valid 10-character password on the same link.
- Expected result: Depends on the rule (see Q-04). If the link is consumed on any submission, the second submit is refused. The team confirms the expected behaviour.

**TC-21: Requesting a second link**
- Preconditions: A reset link (link 1) was requested and has not been used.
- Steps:
  1. Request a second reset link for the same email address (link 2).
  2. Open link 1 and submit a valid password.
  3. If link 1 is refused, open link 2 and submit a valid password.
- Expected result: Depends on the rule (see Q-02). The test records whether link 1 is invalidated when link 2 is issued.

## Open questions

- **Q-01: Unregistered email.** The requirement does not say what happens when the email is not registered. Should the form show the same confirmation as for a registered email (to prevent account enumeration), or show an error? Should any email be sent?
- **Q-02: Multiple reset requests.** If the user requests a second link, is the first link invalidated? Is there a limit on how many links can be requested per time period?
- **Q-03: Exact 60-minute boundary.** Is a link valid at exactly 60:00 after generation, or is it refused from that moment on? Is the 60 minutes counted from link generation or from the moment the email is sent or received?
- **Q-04: Link use on failed validation.** Does a submission that fails validation (for example, a 9-character password) consume the link? The requirement says the link "can be used only once" but does not define what "use" means.
- **Q-05: Session after reset.** Does a successful reset log the user in automatically, or does the user need to log in again? Are existing sessions on other devices ended?
- **Q-06: Password rules detail.** Does "at least 10 characters" count characters or bytes? Are leading and trailing spaces trimmed? Is there a maximum length? Are other rules (complexity, reuse of the old password) in scope?
- **Q-07: Old password.** Once the reset succeeds, the old password must stop working. This is not stated in the requirement. Confirm that it is expected.
- **Q-08: Expiry messaging and retry.** What message is shown on an expired or used link? Is the user offered a new link directly from that page?
- **Q-09: Email delivery time.** The requirement does not say how long delivery may take. If the email arrives after 60 minutes, is the link already expired? Should the 60 minutes be counted from generation or from sending?
- **Q-10: Link transport and channel.** Is the link single-use also when the user clicks it on a different device? Is the token only valid in the browser that requested it? This is not defined.
