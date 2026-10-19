# TC01 - Test cases: account lockout after failed logins

## Requirement

A user account is locked after 5 consecutive failed login attempts. A locked account is unlocked automatically after 15 minutes. A successful login resets the failed-attempt counter.

## Working assumptions

These are needed to write concrete steps. Each one is also listed under Open questions. Tests that depend on an assumption are marked with it.

- A1: Counting and locking are per user account, not per IP address or per session.
- A2: The 5th consecutive failure locks the account. The 5th attempt itself is a failed attempt that triggers the lock.
- A3: The 15-minute timer starts when the account is locked.
- A4: While locked, a login attempt is rejected, even with the correct password. It is shown to the user as a lock, not as "invalid credentials".
- A5: Unlock after 15 minutes resets the failed-attempt counter to 0.

## Test cases

### Positive scenarios

**TC01-001 - Account locks on the 5th consecutive failed login**
- Type: Positive
- Preconditions: Account `alice` exists, is active, is not locked, and has 0 failed attempts.
- Steps:
  1. Submit a login for `alice` with a wrong password. Repeat 4 times (4 failures in total).
  2. Submit a login for `alice` with a wrong password a 5th time.
  3. Submit a login for `alice` with the correct password.
- Expected result:
  - Attempts 1 to 4 are rejected with the normal invalid-credentials response.
  - Attempt 5 is rejected with the normal invalid-credentials response, and the account becomes locked.
  - Step 3 is rejected because the account is locked. No session is created.

**TC01-002 - Locked account rejects the correct password**
- Type: Positive
- Preconditions: `alice` is locked by 5 consecutive failures (see TC01-001). Less than 15 minutes have passed.
- Steps:
  1. Submit a login for `alice` with the correct password.
- Expected result: The login is rejected with a locked-account response. No session or token is issued.

**TC01-003 - Account unlocks automatically after 15 minutes**
- Type: Positive
- Preconditions: `alice` is locked (see TC01-001). The lock time is recorded.
- Steps:
  1. Wait 15 minutes from the lock time. Do not perform any other action on the account.
  2. Submit a login for `alice` with the correct password.
- Expected result: The login succeeds. A session is created. The account is no longer locked.

**TC01-004 - Successful login resets the failure counter**
- Type: Positive
- Preconditions: `alice` is active, not locked, and has 0 failed attempts.
- Steps:
  1. Submit 4 failed logins with a wrong password.
  2. Submit a login with the correct password.
  3. Submit 4 failed logins with a wrong password.
  4. Submit a login with the correct password.
- Expected result:
  - Step 2 succeeds. The counter is reset.
  - After step 3 the account is not locked (4 consecutive failures, not 5).
  - Step 4 succeeds.

**TC01-005 - Failures after a reset need 5 new consecutive failures to lock**
- Type: Positive
- Preconditions: As TC01-004, after step 2 (counter reset by a successful login).
- Steps:
  1. Submit 4 failed logins with a wrong password.
  2. Submit 1 more failed login with a wrong password (5th consecutive failure since the reset).
- Expected result: The account is locked after the 5th failure following the reset.

**TC01-006 - Lock applies only to the affected account**
- Type: Positive
- Preconditions: Accounts `alice` and `bob` both exist and are active.
- Steps:
  1. Lock `alice` with 5 failed logins.
  2. Submit a login for `bob` with the correct password.
- Expected result: `bob` logs in successfully. `alice` stays locked.

**TC01-007 - Counter is reset after automatic unlock (depends on A5)**
- Type: Positive
- Preconditions: `alice` is locked and then unlocked after 15 minutes (see TC01-003).
- Steps:
  1. Submit 4 failed logins with a wrong password.
  2. Check whether the account is locked.
- Expected result: The account is not locked after 4 failures. Under A5 the counter started from 0 after the unlock.

### Negative scenarios

**TC01-008 - Four failures do not lock the account**
- Type: Negative
- Preconditions: `alice` is active, not locked, and has 0 failed attempts.
- Steps:
  1. Submit 4 failed logins with a wrong password.
  2. Submit a login with the correct password.
- Expected result: The account is not locked. The correct password is accepted.

**TC01-009 - Locked account cannot be unlocked early by a failed attempt**
- Type: Negative
- Preconditions: `alice` is locked (see TC01-001). 5 minutes have passed.
- Steps:
  1. Submit a login with a wrong password.
  2. Submit a login with the correct password.
- Expected result: Both attempts are rejected because the account is locked. The account stays locked.

**TC01-010 - Locked account is still locked at 14 minutes 59 seconds**
- Type: Negative (boundary, just before the unlock time)
- Preconditions: `alice` is locked (see TC01-001). The lock time is recorded as T0.
- Steps:
  1. At T0 + 14 min 59 s, submit a login with the correct password.
- Expected result: The login is rejected with a locked-account response.

**TC01-011 - Successful login is not possible while locked, even after a reset attempt**
- Type: Negative
- Preconditions: `alice` is locked (see TC01-001).
- Steps:
  1. Submit a login with the correct password.
  2. Submit a login with the correct password again.
- Expected result: Both are rejected while the lock is active. No success resets the lock before the 15 minutes end.

**TC01-012 - Unknown username does not change any account's state**
- Type: Negative
- Preconditions: No account `ghost` exists. `alice` is active and not locked.
- Steps:
  1. Submit 5 failed logins for `ghost`.
  2. Submit a login for `alice` with a wrong password once.
  3. Submit a login for `alice` with the correct password.
- Expected result: Step 1 gives the normal invalid-credentials response and does not reveal that the account does not exist. `alice` is not locked. Step 3 succeeds.

**TC01-013 - Login attempts with an empty password are handled as failures (depends on Q4)**
- Type: Negative
- Preconditions: `alice` is active and not locked.
- Steps:
  1. Submit 5 logins for `alice` with an empty password.
- Expected result: Per Q4. Expected under the proposed rule: each attempt is a failure and the account is locked after the 5th.

### Boundary scenarios

**TC01-014 - Lock happens at exactly 5 failures, not 4 and not 6**
- Type: Boundary
- Preconditions: `alice` is active, not locked, and has 0 failed attempts.
- Steps:
  1. Submit 4 failed logins. Check the lock status.
  2. Submit the 5th failed login. Check the lock status.
  3. Submit a login with the correct password.
- Expected result: After step 1 the account is not locked. After step 2 it is locked. Step 3 is rejected.

**TC01-015 - Success on the 4th attempt resets the counter exactly**
- Type: Boundary
- Preconditions: As TC01-004.
- Steps:
  1. Submit 4 failed logins.
  2. Submit a successful login (4th attempt in the sequence is a success).
  3. Submit 4 failed logins.
- Expected result: The account is not locked after step 3.

**TC01-016 - Unlock exactly at the 15-minute mark**
- Type: Boundary
- Preconditions: `alice` is locked at T0 (see TC01-001).
- Steps:
  1. At T0 + 15 min 00 s, submit a login with the correct password.
  2. In a separate run, at T0 + 15 min 01 s, submit a login with the correct password.
- Expected result: Step 2 succeeds. Step 1 follows the rule for the 15-minute boundary (see Q2). Under the stated requirement, both should pass, or the team must decide. Record the actual behaviour.

**TC01-017 - Locked account, lock time is not extended by failed attempts during the lock (depends on Q3)**
- Type: Boundary
- Preconditions: `alice` is locked at T0.
- Steps:
  1. At T0 + 10 min, submit 3 failed logins.
  2. At T0 + 15 min 01 s, submit a login with the correct password.
- Expected result: Under the proposed rule (Q3 is open) the account unlocks at T0 + 15 min. The failed attempts at T0 + 10 min do not extend the lock.

**TC01-018 - Clock boundary: failures spread over a long period (depends on Q1)**
- Type: Boundary
- Preconditions: `alice` is active, not locked, and has 0 failed attempts.
- Steps:
  1. Submit 4 failed logins at T0.
  2. Submit the 5th failed login at T0 + 24 h.
- Expected result: Per Q1. Expected under the proposed rule: the account is locked. The requirement does not define a time window for "consecutive". Record the actual behaviour.

### Concurrency and persistence

**TC01-019 - Parallel failed logins cannot bypass the lock (depends on A2)**
- Type: Negative (concurrency)
- Preconditions: `alice` is active, not locked, and has 0 failed attempts.
- Steps:
  1. Send 10 failed login requests for `alice` at the same time.
  2. After all responses are received, submit a login with the correct password.
- Expected result: The account is locked. The correct-password login is rejected. The number of counted failures is not less than 5. Requests that are processed after the lock is set are rejected as locked.

**TC01-020 - Lock survives a service restart**
- Type: Positive (persistence)
- Preconditions: `alice` is locked at T0 (see TC01-001).
- Steps:
  1. Restart the authentication service at T0 + 5 min.
  2. Submit a login with the correct password at T0 + 6 min.
  3. Submit a login with the correct password at T0 + 15 min 01 s.
- Expected result: Step 2 is rejected. Step 3 succeeds. The lock timer is not reset by the restart.

## Open questions

These points are not defined in the requirement. No behaviour has been invented for them. The tests above that depend on them are marked.

- Q1 - Time window for "consecutive": Is there a time limit between failed attempts? If `alice` fails 4 times, waits a day and then fails once more, is the account locked? (TC01-018)
- Q2 - Boundary of the 15 minutes: Is the account unlocked at exactly 15 minutes, or only after 15 minutes? Which clock is used (server time, UTC)? (TC01-016)
- Q3 - Failed attempts during a lock: Do they extend the lock, or are they ignored? Are they logged or counted? (TC01-017)
- Q4 - What counts as a failed attempt: Does an empty password, a malformed request, a wrong MFA or OTP code, or an expired password count? (TC01-013)
- Q5 - Unknown usernames: Are failed logins for non-existent usernames counted? Is there a lock for them? Does the error text reveal whether the account exists? (TC01-012)
- Q6 - Message shown while locked: What is the exact text or error code for a locked account? Does it say how long is left? Could this reveal that an account exists or is locked (user enumeration)?
- Q7 - Counter after unlock: Is the counter reset to 0 on automatic unlock (A5)? Or does it keep its value? (TC01-007)
- Q8 - Manual unlock: Can an administrator or support agent unlock an account before 15 minutes? Is there an audit trail for this?
- Q9 - Password reset: Does a successful password reset, or a reset request, unlock the account or reset the counter?
- Q10 - Scope of "login": Does the lock apply to all login channels: web, mobile, API tokens, SSO, MFA step, and "remember me" sessions? Do failures on one channel count on the others?
- Q11 - Existing sessions: When the account is locked, are already-open sessions terminated?
- Q12 - Notification: Is the user told by email or other means that the account was locked?
- Q13 - Per-IP or global limits: Is there a separate rate limit per IP address or per client, in addition to the per-account lock? If yes, how do the two interact?
- Q14 - Persistence and storage: Where is the counter stored? Is it kept across restarts and across multiple service instances (TC01-019, TC01-020)?
- Q15 - Consecutive failures and multiple devices: If one device fails 3 times and another device fails 2 times, with no success in between, is the account locked?
