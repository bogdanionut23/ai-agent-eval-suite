# Test Cases: Account Lockout on Failed Logins

**Requirement:** A user account is locked after 5 consecutive failed login attempts. A locked account is unlocked automatically after 15 minutes. A successful login resets the failed-attempt counter.

**Assumptions (only the minimum needed to make tests executable; see Open Questions):**
- Test user `u1` exists, is active, and has a known valid password.
- Time can be controlled (clock mock or test environment with adjustable time), so 15-minute waits are not real-time.
- "Failed attempt" means a login with a valid username and wrong password.

---

## A. Positive scenarios

### TC-001: Account is not locked after 4 consecutive failures
- **Preconditions:** `u1` active, failed counter = 0.
- **Steps:**
  1. Submit wrong password 4 times in a row.
  2. Submit the correct password.
- **Expected result:** Steps 1 each return an authentication-failure response. Step 2 logs in successfully. The account is not locked.

### TC-002: Account is locked on the 5th consecutive failure
- **Preconditions:** `u1` active, counter = 0.
- **Steps:**
  1. Submit wrong password 5 times in a row.
  2. Check account status.
- **Expected result:** Account status is locked after the 5th failure.

### TC-003: Correct password is rejected while the account is locked
- **Preconditions:** `u1` locked (TC-002 completed), less than 15 minutes elapsed.
- **Steps:**
  1. Submit the correct password.
- **Expected result:** Login is denied. The response indicates the account is locked (or a generic failure per the open question on messaging). No session is created.

### TC-004: Account unlocks automatically after 15 minutes
- **Preconditions:** `u1` locked at time T.
- **Steps:**
  1. Advance time to T + 15 min 1 s.
  2. Submit the correct password.
- **Expected result:** Login succeeds. Account status is unlocked. No manual action is needed.

### TC-005: Successful login resets the counter
- **Preconditions:** `u1` active, counter = 0.
- **Steps:**
  1. Submit wrong password 4 times.
  2. Submit the correct password (success).
  3. Log out.
  4. Submit wrong password 4 times again.
  5. Submit the correct password.
- **Expected result:** The account is never locked. Step 5 succeeds. This shows the counter restarted at 0 after the successful login in step 2 (otherwise the 8 total failures would have locked the account).

### TC-006: Lockout re-triggers after reset
- **Preconditions:** `u1` had a successful login after earlier failures (counter = 0).
- **Steps:**
  1. Submit wrong password 5 times.
- **Expected result:** Account is locked after the 5th failure.

### TC-007: Counter is reset after post-unlock successful login
- **Preconditions:** `u1` was locked and has auto-unlocked.
- **Steps:**
  1. Log in successfully.
  2. Log out.
  3. Submit wrong password 4 times.
  4. Submit the correct password.
- **Expected result:** Step 4 succeeds. The counter started at 0 after the unlock-and-login, not at 5.

---

## B. Negative scenarios

### TC-008: Non-consecutive failures do not lock the account
- **Preconditions:** `u1` active, counter = 0.
- **Steps:**
  1. Fail 2 times.
  2. Succeed once.
  3. Fail 2 times.
  4. Succeed once.
  5. Fail 2 times.
- **Expected result:** Account is never locked (6 total failures, never 5 consecutive).

### TC-009: Account stays locked before 15 minutes elapse
- **Preconditions:** `u1` locked at time T.
- **Steps:**
  1. Advance time to T + 10 min.
  2. Submit the correct password.
- **Expected result:** Login denied; account remains locked.

### TC-010: Failed attempts on a locked account do not shorten the lock
- **Preconditions:** `u1` locked at T.
- **Steps:**
  1. At T + 5 min, submit the correct password (denied).
  2. At T + 15 min 1 s, submit the correct password.
- **Expected result:** Step 1 denied. Step 2 succeeds (unlock time is measured from the lock event, behavior per open question Q3 if attempts during lock are expected to extend it).

### TC-011: Failed login for a non-existent username does not lock any account
- **Preconditions:** Username `ghost` does not exist.
- **Steps:**
  1. Submit 6 failed logins for `ghost`.
  2. Attempt login as `u1` with the correct password.
- **Expected result:** `u1` is unaffected and logs in. No account is created or locked for `ghost`. Response does not reveal whether the user exists (see Q7).

### TC-012: Failures on one account do not lock another
- **Preconditions:** Users `u1` and `u2` active, counters = 0.
- **Steps:**
  1. Submit wrong password 5 times for `u1`.
  2. Log in as `u2` with the correct password.
- **Expected result:** `u1` is locked. `u2` logs in successfully.

### TC-013: Empty or malformed credentials
- **Preconditions:** `u1` active, counter = 0.
- **Steps:**
  1. Submit an empty password for `u1` once.
  2. Check the counter.
- **Expected result:** Request rejected. Whether the counter increments is defined by Q5; the test records the observed behavior and flags a defect if it contradicts the clarified rule.

### TC-014: Locked state persists across server restart / new session
- **Preconditions:** `u1` locked at T.
- **Steps:**
  1. Restart the application (or switch to another node/client).
  2. Within 15 minutes, attempt a correct login.
- **Expected result:** Login denied; lock is persisted, not held only in memory.

### TC-015: Lock is not bypassed by a different client or IP
- **Preconditions:** `u1` locked.
- **Steps:**
  1. Attempt a correct login from a different device/IP/browser.
- **Expected result:** Login denied.

---

## C. Boundary scenarios

### TC-016: Exactly 4 failures (lower boundary, not locked)
- **Preconditions:** counter = 0.
- **Steps:** Fail 4 times, then check status.
- **Expected result:** Not locked; counter = 4.

### TC-017: Exactly 5 failures (boundary, locked)
- **Preconditions:** counter = 0.
- **Steps:** Fail 5 times, then check status.
- **Expected result:** Locked.

### TC-018: 6th attempt after lock (upper boundary)
- **Preconditions:** `u1` locked by 5 failures.
- **Steps:** Submit a 6th wrong password.
- **Expected result:** Denied as locked. The counter does not cause errors or overflow. Lock state is unchanged (lock timer behavior per Q3).

### TC-019: Success on the 5th attempt (just before locking)
- **Preconditions:** counter = 4 (4 prior failures).
- **Steps:** Submit the correct password as the 5th attempt.
- **Expected result:** Login succeeds, account not locked, counter reset to 0.

### TC-020: Unlock at 14 min 59 s (just before boundary)
- **Preconditions:** `u1` locked at T.
- **Steps:** At T + 14:59, submit the correct password.
- **Expected result:** Denied; account still locked.

### TC-021: Unlock at exactly 15 min 00 s
- **Preconditions:** `u1` locked at T.
- **Steps:** At T + 15:00, submit the correct password.
- **Expected result:** Behavior depends on Q4 (inclusive or exclusive). Record observed result; the expected value must be set once Q4 is answered.

### TC-022: Unlock at 15 min 01 s (just after boundary)
- **Preconditions:** `u1` locked at T.
- **Steps:** At T + 15:01, submit the correct password.
- **Expected result:** Login succeeds.

### TC-023: First wrong attempt after auto-unlock
- **Preconditions:** `u1` auto-unlocked after 15 minutes, no successful login yet.
- **Steps:** Submit one wrong password, then check status and counter.
- **Expected result:** Per Q2: either counter = 1 (reset on unlock) and account not locked, or counter = 6 and account immediately re-locked. Record observed behavior and compare to the clarified rule.

### TC-024: Rapid / concurrent failed attempts
- **Preconditions:** counter = 0.
- **Steps:** Submit 10 wrong-password requests concurrently.
- **Expected result:** Account is locked. Counter is consistent (no lost updates that leave the account unlocked after 5 or more failures). No server errors.

### TC-025: Clock and time-zone edge cases
- **Preconditions:** `u1` locked shortly before a DST change or midnight rollover.
- **Steps:** Advance across the DST change/midnight and attempt login at 15 minutes elapsed.
- **Expected result:** Unlock timing is based on elapsed time (monotonic or UTC), not local wall-clock, so the 15 minutes is unaffected.

---

## Open Questions (ambiguities and gaps; no behavior was invented for these)

1. **What counts as a "failed login attempt"?** Wrong password only, or also empty fields, malformed input, expired password, failed MFA/second factor, or a disabled account?
2. **Counter state at auto-unlock:** Is the counter reset to 0 when the account unlocks after 15 minutes, or does it stay at 5 so the next single failure re-locks it? The requirement only says a *successful login* resets it.
3. **Attempts during the lock:** Do login attempts (failed or correct) while locked extend or restart the 15-minute timer, or is the unlock time fixed from the moment of locking?
4. **Exact boundary:** Is the account unlocked at exactly 15:00 (inclusive) or only after more than 15:00?
5. **"Consecutive" definition:** Is it consecutive per account regardless of source IP, device or session? Is there a time window within which failures must occur, or do failures from days apart count?
6. **Does a correct password during the lock count?** Is it simply rejected, and does it affect the counter?
7. **User messaging and enumeration:** What must the user see when locked (explicit "account locked" vs. generic error)? Is revealing the lock state acceptable from a security standpoint? Should the remaining time be shown?
8. **Non-existent usernames:** Are failures for unknown usernames tracked in any way (e.g., for rate limiting)?
9. **Other unlock paths:** Can an administrator or the user (password reset, email link) unlock earlier? Does a password reset clear the lock and counter?
10. **Notifications and audit:** Is the user notified on lock? Must lock/unlock events be logged for audit?
11. **Scope:** Does the rule apply to all roles, including admins and service accounts? Does it apply to API/token logins as well as the UI?
12. **Persistence and distribution:** Must the counter and lock state survive restarts and be shared across multiple servers?
13. **Time source:** Server time (UTC) assumed for the 15-minute window; confirm.
14. **Is there a repeated-lock escalation** (e.g., longer lock after repeated lockouts)? Not stated, so not tested.
15. **Non-functional:** Any expected behavior for concurrent attempts (race conditions) and rate limiting beyond the lock?

Test cases TC-010, TC-013, TC-021 and TC-023 depend directly on the answers to questions 1 to 4 and should be finalized once those are resolved.
