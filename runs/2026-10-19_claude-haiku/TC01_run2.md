# Test cases: Account lockout after failed login attempts

## Requirement

A user account is locked after 5 consecutive failed login attempts. A locked account is unlocked automatically after 15 minutes. A successful login resets the failed-attempt counter.

## Conventions

- "Lock time" (T0) is the moment the 5th consecutive failure is recorded.
- Unlock time is T0 + 15 minutes.
- Where timing is tested, the clock must be controlled (mocked or fast-forwarded) rather than waited on in real time.
- Where a test depends on an open question below, it says so in its Note.

---

## Positive scenarios

### TC-LOCK-001: Account locks on the 5th consecutive failed login

- **Type:** Positive
- **Preconditions:** Account `alice` exists, is active, is not locked, and has a failed-attempt counter of 0. Password is `Correct#1`.
- **Steps:**
  1. Submit login for `alice` with a wrong password (attempt 1).
  2. Repeat step 1 for attempts 2, 3 and 4.
  3. Submit login for `alice` with a wrong password (attempt 5).
  4. Submit login for `alice` with the correct password `Correct#1`.
- **Expected result:** Attempts 1-4 are rejected with the standard invalid-credentials response. Attempt 5 is rejected and the account is now locked. Step 4 is refused with the locked-account response; no session is created.

### TC-LOCK-002: Locked account unlocks automatically after 15 minutes

- **Type:** Positive
- **Preconditions:** Account `alice` is locked by 5 consecutive failures at T0. Clock is controlled.
- **Steps:**
  1. Advance the clock to T0 + 15 min.
  2. Submit login for `alice` with the correct password.
- **Expected result:** Login succeeds and a session is created. No manual action is needed.

### TC-LOCK-003: Successful login resets the counter (4 failures, success, then 4 failures does not lock)

- **Type:** Positive
- **Preconditions:** Account `bob` is active, not locked, counter at 0.
- **Steps:**
  1. Submit 4 wrong-password logins for `bob`.
  2. Submit 1 correct-password login for `bob`. Log out.
  3. Submit 4 wrong-password logins for `bob`.
  4. Submit 1 correct-password login for `bob`.
- **Expected result:** Step 2 succeeds. Step 3 produces 4 standard invalid-credentials responses and no lock. Step 4 succeeds. The account is never locked.

### TC-LOCK-004: Successful login resets the counter, then 5 new failures lock the account

- **Type:** Positive
- **Preconditions:** Account `carol` is active, not locked, counter at 0.
- **Steps:**
  1. Submit 3 wrong-password logins for `carol`.
  2. Submit 1 correct-password login for `carol`. Log out.
  3. Submit 5 consecutive wrong-password logins for `carol`.
- **Expected result:** Step 2 succeeds. The 5th failure in step 3 locks the account. The counter restarts from zero after the successful login, so the lock happens after 5 new failures, not after 2.

### TC-LOCK-005: Successful first login does not change the counter

- **Type:** Positive
- **Preconditions:** Account `dan` is active, not locked, counter at 0.
- **Steps:**
  1. Submit correct-password login for `dan`. Log out.
  2. Submit 4 wrong-password logins for `dan`.
- **Expected result:** Step 1 succeeds. Step 2 produces 4 standard invalid-credentials responses. The account is not locked.

---

## Negative scenarios

### TC-LOCK-006: Four consecutive failures do not lock the account

- **Type:** Negative
- **Preconditions:** Account `erin` is active, not locked, counter at 0. Password is `Correct#1`.
- **Steps:**
  1. Submit 4 wrong-password logins for `erin`.
  2. Submit a correct-password login for `erin`.
- **Expected result:** The 4 wrong-password attempts return standard invalid-credentials responses. The correct-password login in step 2 succeeds.

### TC-LOCK-007: Correct password is refused while the account is locked

- **Type:** Negative
- **Preconditions:** Account `frank` was locked at T0. Clock is at T0 + 1 min.
- **Steps:**
  1. Submit login for `frank` with the correct password.
- **Expected result:** Login is refused with the locked-account response. No session is created. The lock is not lifted.
- **Note:** The requirement says the account is "locked", which this test reads as refusing all logins, including correct ones. See open question Q6 for the exact response.

### TC-LOCK-008: Wrong password during lock is refused and does not unlock the account

- **Type:** Negative
- **Preconditions:** Account `grace` was locked at T0. Clock is at T0 + 5 min.
- **Steps:**
  1. Submit login for `grace` with a wrong password.
- **Expected result:** Login is refused with the locked-account response. The account stays locked.

### TC-LOCK-009: Attempts during lock do not unlock the account early

- **Type:** Negative
- **Preconditions:** Account `heidi` was locked at T0. Clock is controlled.
- **Steps:**
  1. At T0 + 14 min, submit login with the correct password.
  2. Submit login with the correct password again at T0 + 14 min 59 s.
- **Expected result:** Both logins are refused with the locked-account response. Nothing shortens the lock.

### TC-LOCK-010: Lock applies only to the affected account

- **Type:** Negative
- **Preconditions:** Accounts `ivan` and `judy` are both active and not locked. Counters are at 0.
- **Steps:**
  1. Submit 5 wrong-password logins for `ivan`. Confirm `ivan` is locked.
  2. Submit a correct-password login for `judy`.
- **Expected result:** `ivan` is locked. `judy`'s login succeeds.

### TC-LOCK-011: Failures on a locked account do not affect another account

- **Type:** Negative
- **Preconditions:** Accounts `kate` (locked) and `leo` (active, not locked, counter at 0).
- **Steps:**
  1. Submit 3 wrong-password logins for `kate`, which is locked.
  2. Submit 4 wrong-password logins for `leo`.
  3. Submit a correct-password login for `leo`.
- **Expected result:** `leo` still has 4 failures and is not locked. Step 3 succeeds.

### TC-LOCK-012: Lock is enforced across sessions and clients

- **Type:** Negative
- **Preconditions:** Account `mike` was locked at T0 through client A (for example, the web app). Client B is a separate browser or device.
- **Steps:**
  1. From client B, submit login for `mike` with the correct password.
- **Expected result:** Login is refused with the locked-account response, because the lock is stored on the account and not on the client.
- **Note:** The requirement does not say whether the lock applies to every channel (web, mobile, API, SSO). See open question Q8. This test assumes it does.

---

## Boundary scenarios

### TC-LOCK-013: Four failures is one below the threshold (boundary below)

- **Type:** Boundary
- **Preconditions:** Account `nina` is active, not locked, counter at 0.
- **Steps:**
  1. Submit exactly 4 wrong-password logins for `nina`.
  2. Check the account status.
- **Expected result:** Status is "not locked". The counter reads 4 (if observable through an admin view or logs).

### TC-LOCK-014: Five failures is exactly the threshold (boundary at)

- **Type:** Boundary
- **Preconditions:** Account `oscar` is active, not locked, counter at 0.
- **Steps:**
  1. Submit exactly 5 wrong-password logins for `oscar`.
  2. Check the account status and submit a correct-password login.
- **Expected result:** After the 5th failure the account is locked. The correct-password login is refused.

### TC-LOCK-015: Sixth attempt after the lock is refused, not counted as a new cycle

- **Type:** Boundary
- **Preconditions:** Account `peggy` was locked at T0. Clock is at T0 + 1 min.
- **Steps:**
  1. Submit 1 wrong-password login for `peggy`.
  2. Advance the clock to T0 + 15 min and submit a correct-password login.
- **Expected result:** Step 1 is refused with the locked-account response. Step 2 succeeds, so the lock lasted the full 15 minutes.
- **Note:** This test assumes attempts during a lock do not extend it. See open question Q3.

### TC-LOCK-016: Unlock boundary, one second before 15 minutes

- **Type:** Boundary
- **Preconditions:** Account `quinn` locked at T0. Clock is controlled.
- **Steps:**
  1. Advance the clock to T0 + 14 min 59 s.
  2. Submit a correct-password login.
- **Expected result:** Login is refused with the locked-account response.

### TC-LOCK-017: Unlock boundary, exactly 15 minutes

- **Type:** Boundary
- **Preconditions:** Account `ruth` locked at T0. Clock is controlled.
- **Steps:**
  1. Advance the clock to T0 + 15 min 00 s.
  2. Submit a correct-password login.
- **Expected result:** Login succeeds.
- **Note:** The requirement does not say whether 15:00 is inclusive. This test assumes the account is unlocked at exactly 15:00. See open question Q4.

### TC-LOCK-018: Unlock boundary, one second after 15 minutes

- **Type:** Boundary
- **Preconditions:** Account `sam` locked at T0. Clock is controlled.
- **Steps:**
  1. Advance the clock to T0 + 15 min 01 s.
  2. Submit a correct-password login.
- **Expected result:** Login succeeds.

### TC-LOCK-019: Success on the 5th attempt (last allowed attempt) does not lock

- **Type:** Boundary
- **Preconditions:** Account `tina` is active, not locked, counter at 0.
- **Steps:**
  1. Submit 4 wrong-password logins for `tina`.
  2. Submit the correct-password login (5th attempt).
- **Expected result:** The 5th attempt succeeds. The account is not locked. The counter is reset to 0.

### TC-LOCK-020: Counter is reset to zero after success, not decremented

- **Type:** Boundary
- **Preconditions:** Account `uma` is active, not locked, counter at 0.
- **Steps:**
  1. Submit 4 wrong-password logins for `uma`.
  2. Submit the correct-password login (succeeds).
  3. Submit 4 wrong-password logins for `uma`.
  4. Submit the correct-password login.
- **Expected result:** Step 3 produces 4 invalid-credentials responses and no lock. Step 4 succeeds. A counter that was only decremented would reach 5 across steps 1-3 and lock, so this confirms a full reset.

### TC-LOCK-021: Five parallel wrong-password logins lock the account without losing a count

- **Type:** Boundary (concurrency)
- **Preconditions:** Account `victor` is active, not locked, counter at 0.
- **Steps:**
  1. Send 5 wrong-password login requests for `victor` at the same time (within the same second).
  2. Submit a correct-password login for `victor`.
- **Expected result:** The account is locked after the batch. The correct-password login is refused. Every failure is counted (no lost updates from concurrent writes).

### TC-LOCK-022: Lock survives a page reload and a new browser session

- **Type:** Boundary (state persistence)
- **Preconditions:** Account `wendy` was locked at T0. Clock is at T0 + 2 min.
- **Steps:**
  1. Reload the login page and clear cookies and local storage.
  2. Submit a correct-password login for `wendy` from a fresh browser session.
- **Expected result:** Login is refused with the locked-account response. Lock state is stored server-side, not in the browser.

---

## Open questions

These are not tested above as fixed behaviour. Each needs an answer before the related test can be finalised.

- **Q1. Counter scope.** Is the failure counter per account only, or per account plus IP address, device or client? The requirement says "user account", which suggests per account, but this is not stated.
- **Q2. Definition of "consecutive" and time window.** Do failures spread over days still count toward the 5? Does any inactivity window reset the counter, or does only a successful login reset it?
- **Q3. Attempts during the lock.** Do failed attempts made while the account is locked extend the lock, or are they ignored? Does the 15-minute timer run from T0 only, or restart with each attempt? (TC-LOCK-009, TC-LOCK-015 assume the timer is fixed from T0.)
- **Q4. Exact unlock moment.** Is the account unlocked at exactly T0 + 15 min (inclusive), or only after it (exclusive)? Is the timer checked at login time or by a background job, and what is the precision?
- **Q5. Counter after auto-unlock.** After automatic unlock, does the counter reset to 0? If it does not, is the next single wrong password enough to lock again? This is the main unspecified case for repeated lockouts.
- **Q6. Response to the user.** What message is shown when the account is locked, and is it the same for a correct and a wrong password? Does it reveal that the account exists or is locked (account enumeration)?
- **Q7. Non-existent usernames.** Do failed attempts for usernames that do not exist increase any counter? Does the response differ from a real account being locked?
- **Q8. Channels.** Does the lock apply to all login paths: web, mobile, API, SSO, and any "remember me" or token refresh flow? Can an existing session stay active after the account is locked?
- **Q9. What counts as a failure.** Does an empty password, a malformed request, an MFA failure or an expired-password prompt count as a failed attempt?
- **Q10. Manual intervention.** Can an administrator or support agent unlock an account before 15 minutes? Is there a manual lock or unlock that resets the counter?
- **Q11. Notifications and logging.** Should the user be notified (for example by email) when the account is locked or unlocked? Should lock events be logged or audited?
- **Q12. Password reset.** Does a successful password reset clear the lock and the counter? Can a locked user still reset their password?
- **Q13. Case sensitivity.** Is the username matched case-insensitively when counting failures? Could `Alice` and `alice` have separate counters?
- **Q14. Concurrency expectation.** Is an exact count required under concurrent requests (TC-LOCK-021), or is it acceptable for the lock to occur slightly late?
- **Q15. Clock source.** Is the lock timer based on server time? How is clock skew across multiple app instances handled?
