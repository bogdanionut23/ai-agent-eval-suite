# Test Cases: Account Lockout After Failed Logins

**Requirement:** A user account is locked after 5 consecutive failed login attempts. A locked account is unlocked automatically after 15 minutes. A successful login resets the failed-attempt counter.

## Assumptions (only what the requirement states)
- N = 5 failed attempts triggers lock. Lock duration = 15 minutes. Success resets the counter.
- Behaviour not stated in the requirement is listed under Open Questions. Test cases that depend on it are marked **[Blocked by OQ-x]** and need confirmation before execution.

Common precondition for all cases unless stated otherwise: user account U exists, is active, has a known valid password, and has a failed-attempt counter of 0 and is not locked.

---

## A. Positive scenarios

### TC-LOCK-001: Successful login with no prior failures
- **Preconditions:** Counter = 0.
- **Steps:** 1. Log in with valid credentials.
- **Expected:** Login succeeds. Counter remains 0.

### TC-LOCK-002: Counter is incremented on each failed attempt
- **Preconditions:** Counter = 0.
- **Steps:** 1. Submit a wrong password. 2. Check the counter (DB/admin tool/API).
- **Expected:** Login rejected with a generic failure message. Counter = 1. Account not locked.

### TC-LOCK-003: Account locks on the 5th consecutive failure
- **Preconditions:** Counter = 0.
- **Steps:** 1. Submit a wrong password 5 times in a row. 2. Observe the response to the 5th attempt. 3. Check account status.
- **Expected:** Attempts 1-4 are rejected as invalid credentials. After the 5th, the account is in the locked state with a lock timestamp recorded. **[Blocked by OQ-3 for the exact message on the 5th attempt]**

### TC-LOCK-004: Valid password is rejected while the account is locked
- **Preconditions:** Account locked via TC-LOCK-003 (less than 15 minutes ago).
- **Steps:** 1. Log in with the correct password.
- **Expected:** Login is denied. No session is created. The user is informed the account is locked (or receives a message per OQ-3).

### TC-LOCK-005: Automatic unlock after 15 minutes
- **Preconditions:** Account locked at time T (record T).
- **Steps:** 1. Wait until T + 15:00 plus a small tolerance (or advance the system clock or use a test hook). 2. Log in with the correct password.
- **Expected:** Login succeeds with no manual intervention. Account status is unlocked. **[Blocked by OQ-4 for the counter state after unlock]**

### TC-LOCK-006: Successful login resets the counter
- **Preconditions:** Counter = 0.
- **Steps:** 1. Submit wrong password 3 times (counter = 3). 2. Log in with the correct password. 3. Check the counter.
- **Expected:** Login succeeds. Counter = 0.

### TC-LOCK-007: Counter reset means a full 5 new failures are needed to lock
- **Preconditions:** Counter = 0.
- **Steps:** 1. Fail 4 times. 2. Log in successfully. 3. Log out. 4. Fail 4 more times. 5. Attempt a valid login.
- **Expected:** After step 4 the account is not locked (counter = 4), and step 5 succeeds. Demonstrates failures before the success are not carried over.

---

## B. Boundary scenarios

### TC-LOCK-010: 4 failures do not lock the account (N-1)
- **Preconditions:** Counter = 0.
- **Steps:** 1. Fail 4 times. 2. Log in with the correct password.
- **Expected:** Account is not locked after 4 failures. Valid login succeeds and the counter resets to 0.

### TC-LOCK-011: Exactly 5 failures lock the account (N)
- **Preconditions:** Counter = 0.
- **Steps:** 1. Fail 5 times. 2. Log in with the correct password immediately.
- **Expected:** Account is locked; the correct password is rejected.

### TC-LOCK-012: 6th attempt (N+1) while locked
- **Preconditions:** Account locked after 5 failures.
- **Steps:** 1. Submit a 6th wrong password. 2. Check the counter and lock timestamp.
- **Expected:** Login rejected. Account remains locked. **[Blocked by OQ-5: does this extend the lock, or increment the counter?]**

### TC-LOCK-013: Just before 15 minutes (14:59)
- **Preconditions:** Account locked at T.
- **Steps:** 1. At T + 14:59 (use controllable clock) log in with the correct password.
- **Expected:** Login denied; account still locked.

### TC-LOCK-014: Exactly at 15:00
- **Preconditions:** Account locked at T.
- **Steps:** 1. At exactly T + 15:00 log in with the correct password.
- **Expected:** Behaviour is **undefined by the requirement** ("after 15 minutes" could mean >= or >). **[Blocked by OQ-6]** Record the observed behaviour and raise it as a clarification.

### TC-LOCK-015: Just after 15 minutes (15:01)
- **Preconditions:** Account locked at T.
- **Steps:** 1. At T + 15:01 log in with the correct password.
- **Expected:** Login succeeds.

### TC-LOCK-016: Failures are counted per account, not globally
- **Preconditions:** Accounts U1 and U2 both with counter 0.
- **Steps:** 1. Fail 3 times on U1 and 2 times on U2. 2. Log in to each with correct passwords.
- **Expected:** Neither account is locked. Both log in successfully. Counters are independent.

---

## C. Negative scenarios

### TC-LOCK-020: Non-consecutive failures do not lock (success in between)
- **Preconditions:** Counter = 0.
- **Steps:** 1. Fail 3 times. 2. Succeed once. 3. Fail 3 times. 4. Log in with the correct password.
- **Expected:** Total failures = 6 but never 5 consecutive. Account is never locked. Final login succeeds.

### TC-LOCK-021: Failed attempts on a non-existent username
- **Preconditions:** Username "nouser" does not exist.
- **Steps:** 1. Submit 5+ failed logins for "nouser". 2. Compare responses with those for a real account that is locked.
- **Expected:** No account is created or locked. Responses should not reveal whether the user exists (account enumeration check). **[Blocked by OQ-7]**

### TC-LOCK-022: Wrong password during an active lock does not unlock the account
- **Preconditions:** Account locked.
- **Steps:** 1. Submit wrong passwords repeatedly within 15 minutes.
- **Expected:** Account remains locked. Never auto-unlocks early.

### TC-LOCK-023: Locked state cannot be bypassed through other entry points
- **Preconditions:** Account locked.
- **Steps:** 1. Attempt login through each alternate channel (API, mobile app, SSO, "remember me"/existing token refresh) with correct credentials. **[Blocked by OQ-8]**
- **Expected:** All channels deny access while locked.

### TC-LOCK-024: Empty or malformed credentials
- **Preconditions:** Counter = 0.
- **Steps:** 1. Submit empty password. 2. Submit an over-long or special-character password. 3. Check the counter.
- **Expected:** Behaviour per OQ-9 (whether these count as failed attempts). **[Blocked by OQ-9]**

### TC-LOCK-025: Case and whitespace in username
- **Preconditions:** Account "User1" exists.
- **Steps:** 1. Fail 5 times using variants ("user1", "USER1", " User1 ").
- **Expected:** Failures are attributed to the same account if the system treats usernames as equivalent, so the lock triggers. **[Blocked by OQ-10]**

---

## D. Concurrency, persistence and robustness

### TC-LOCK-030: Concurrent failed attempts
- **Preconditions:** Counter = 0.
- **Steps:** 1. Send 10 simultaneous wrong-password requests for U.
- **Expected:** Account is locked. Counter is accurate and no attempt slips through unchecked (no race allowing more failures than the limit). A subsequent valid login is denied.

### TC-LOCK-031: Concurrent valid login and failure at counter = 4
- **Preconditions:** Counter = 4.
- **Steps:** 1. Simultaneously submit one wrong and one correct password.
- **Expected:** The final state is consistent (either locked with counter 5 or unlocked with counter 0, per order of processing); no corrupted state. Document the observed outcome.

### TC-LOCK-032: Lock survives logout, browser change, and server restart
- **Preconditions:** Account locked.
- **Steps:** 1. Try from a different browser/device/IP. 2. Restart the application service. 3. Try again within 15 minutes.
- **Expected:** Account remains locked in all cases and unlock timing is not reset by the restart.

### TC-LOCK-033: Counter persists across sessions before lock
- **Preconditions:** Counter = 0.
- **Steps:** 1. Fail 3 times in browser A. 2. Fail 2 times in browser B.
- **Expected:** Account locked (5 consecutive failures regardless of session).

### TC-LOCK-034: Unlock timer and time zones / DST / clock changes
- **Preconditions:** Account locked.
- **Steps:** 1. Change the server clock across a DST boundary or adjust time zone. 2. Verify the unlock time.
- **Expected:** Unlock happens 15 real minutes after locking, independent of time zone or DST. (Lock timestamp should use UTC/monotonic time.)

### TC-LOCK-035: Re-lock after auto-unlock
- **Preconditions:** Account auto-unlocked after TC-LOCK-005.
- **Steps:** 1. Fail 5 times again.
- **Expected:** Account locks again for another 15 minutes (assuming counter was reset; see OQ-4).

### TC-LOCK-036: Failures after unlock before success
- **Preconditions:** Account just auto-unlocked.
- **Steps:** 1. Fail once. 2. Check the counter.
- **Expected:** Counter = 1 if reset on unlock; otherwise a single failure could re-lock. **[Blocked by OQ-4]**

---

## E. Open questions (not assumed in the tests above)

| ID | Question | Why it matters |
|----|----------|----------------|
| OQ-1 | What is the scope of "failed login attempt"? Only wrong password, or also unknown username, expired password, failed MFA/second factor, disabled account? | Defines what increments the counter. |
| OQ-2 | Is the counter per account, per IP, per device, or a combination? | Affects TC-LOCK-016, TC-LOCK-032, TC-LOCK-033. |
| OQ-3 | What message is shown on the 5th failure and when locked? Does it reveal the lock (UX) or stay generic (security)? | Expected results of TC-LOCK-003, 004. |
| OQ-4 | When the account auto-unlocks, is the failed-attempt counter reset to 0? | Determines whether 1 failure after unlock re-locks the account. |
| OQ-5 | Do login attempts made during the lock (correct or incorrect) extend the lock, restart the 15-minute timer, or increment the counter? | TC-LOCK-012, 022. |
| OQ-6 | Is the boundary inclusive: is the account unlocked at exactly 15:00 or only after? | TC-LOCK-014. |
| OQ-7 | How are non-existent usernames handled (no lock, same message as real accounts)? | Account-enumeration security. |
| OQ-8 | Which entry points are covered (web, mobile, API, SSO, token refresh)? Does an already active session survive the lock? | TC-LOCK-023. |
| OQ-9 | Do empty or malformed inputs, or requests rejected by validation/CAPTCHA/rate limiting, count as failures? | TC-LOCK-024. |
| OQ-10 | Are usernames case-insensitive and trimmed for the purpose of counting? | TC-LOCK-025. |
| OQ-11 | Can the user or an admin unlock earlier (password reset, support unlock, email link)? Does a password reset clear the lock and counter? | Missing requirement; unlock path beyond the timer. |
| OQ-12 | Is the lock duration fixed, or does it escalate on repeated lockouts? Is 15 minutes configurable? | Affects test data and environment setup. |
| OQ-13 | Is the user notified (email/alert) about the lock? Are lock events logged/audited? | Possible additional test cases for notification and audit. |
| OQ-14 | Does the counter expire if failures are spread over a long time (e.g., 2 failures today, 3 tomorrow)? "Consecutive" is only defined relative to a success. | Could affect TC-LOCK-033 and long-running scenarios. |
| OQ-15 | How is the countdown measured (server time, UTC) and how is it expected to behave with clock changes? Is remaining lock time shown to the user? | TC-LOCK-034. |

## Coverage summary
- **Positive:** 001-007 (login, counting, lock, unlock, reset).
- **Boundary:** 010-016 (4/5/6 attempts; 14:59/15:00/15:01; per-account isolation).
- **Negative:** 020-025 (non-consecutive failures, unknown user, bypass, malformed input, username variants).
- **Non-functional/robustness:** 030-036 (concurrency, persistence, time handling, re-lock).
