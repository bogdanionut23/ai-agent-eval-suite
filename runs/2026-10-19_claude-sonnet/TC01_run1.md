# Test Cases: Account Lockout (5 failed attempts, 15-minute auto-unlock, counter reset)

## Assumptions (stated, not invented behaviour)
- N = 5 failed attempts; lock duration = 15 minutes; "consecutive" means no successful login between the failures.
- Where the requirement is silent, the test case says "see OQ-x" and the expected result is marked TBC until the question is answered.
- Test user: `user_a`, valid password `Valid#123`, wrong password `Wrong#000`. A second user `user_b` is used for isolation tests.
- Time-based tests need a controllable clock or a configurable lock duration in a test environment.

## Test cases

### Positive

**TC-LOCK-001 - Successful login with no prior failures**
- Preconditions: `user_a` exists, is active, counter = 0.
- Steps: 1. Log in with valid credentials.
- Expected: Login succeeds. Counter remains 0.

**TC-LOCK-002 - Account locks on the 5th consecutive failed attempt**
- Preconditions: `user_a` active, counter = 0.
- Steps: 1. Submit wrong password 5 times in a row.
- Expected: Attempts 1-4 are rejected with the normal "invalid credentials" response and the account is not locked. After attempt 5 the account is locked. (Message on attempt 5: see OQ-4.)

**TC-LOCK-003 - Locked account rejects login with correct password**
- Preconditions: `user_a` is locked (from TC-LOCK-002), less than 15 minutes ago.
- Steps: 1. Log in with the valid password.
- Expected: Login is rejected, no session is created. Lock is not shortened or removed.

**TC-LOCK-004 - Automatic unlock after 15 minutes**
- Preconditions: `user_a` locked at time T.
- Steps: 1. Advance the clock to T + 15:00 (or wait). 2. Log in with the valid password.
- Expected: Login succeeds without admin action. Counter is 0 afterwards.

**TC-LOCK-005 - Successful login resets the counter**
- Preconditions: `user_a` active, counter = 0.
- Steps: 1. Fail login 4 times. 2. Log in successfully. 3. Fail login 4 more times. 4. Log in successfully.
- Expected: Account is never locked, because the counter reset after each success. (4 + 4 failures are not 8 consecutive.)

**TC-LOCK-006 - Counter reset allows a full new set of 5 attempts**
- Preconditions: as TC-LOCK-005 after step 2 (counter reset).
- Steps: 1. Fail 4 times (not locked). 2. Fail a 5th time.
- Expected: Account locks only on the 5th failure after the reset.

**TC-LOCK-007 - Locking one user does not affect another**
- Preconditions: `user_a` and `user_b` active.
- Steps: 1. Fail 5 times for `user_a`. 2. Log in as `user_b` with valid credentials.
- Expected: `user_b` logs in successfully; only `user_a` is locked.

### Negative

**TC-LOCK-008 - Non-consecutive failures do not lock the account**
- Preconditions: counter = 0.
- Steps: 1. Fail 3 times. 2. Log in successfully. 3. Fail 2 times.
- Expected: Not locked (total failures = 5, but not consecutive).

**TC-LOCK-009 - Wrong username does not count against any account**
- Preconditions: no account named `ghost`; `user_a` counter = 0.
- Steps: 1. Attempt login as `ghost` 5 times. 2. Log in as `user_a` with valid credentials.
- Expected: `user_a` is not locked and logs in. Response for `ghost` does not reveal that the account does not exist (see OQ-8).

**TC-LOCK-010 - Failed attempts while locked do not extend the lock (TBC)**
- Preconditions: `user_a` locked at T.
- Steps: 1. At T + 10 min, try a wrong password. 2. At T + 15 min, log in with the valid password.
- Expected: TBC, see OQ-3. If attempts during lock do not extend it, login at T + 15 succeeds.

**TC-LOCK-011 - Valid password during lock does not reset counter or unlock**
- Preconditions: `user_a` locked at T.
- Steps: 1. At T + 5 min, log in with the valid password (rejected). 2. At T + 14:59, log in with the valid password.
- Expected: Both rejected. The lock is still active.

**TC-LOCK-012 - Failure after auto-unlock starts a fresh count**
- Preconditions: `user_a` locked at T, auto-unlocked at T + 15.
- Steps: 1. After T + 15, fail login once. 2. Fail 3 more times.
- Expected: Not locked after 4 failures (counter was reset by unlock, see OQ-5). Locks on the 5th.

**TC-LOCK-013 - Locked state survives page reload, new session, other browser/device**
- Preconditions: `user_a` locked.
- Steps: 1. Try to log in from a different browser, device and IP.
- Expected: Login rejected on all; lock is account-based, not session- or client-based.

**TC-LOCK-014 - Parallel failed attempts**
- Preconditions: counter = 0.
- Steps: 1. Send 10 wrong-password requests concurrently.
- Expected: Account is locked; counter is not corrupted (no lost updates, no more than the defined behaviour for requests beyond the 5th); a following valid login is rejected.

### Boundary

**TC-LOCK-015 - 4 failures: not locked (N-1)**
- Preconditions: counter = 0.
- Steps: 1. Fail 4 times. 2. Log in with the valid password.
- Expected: Login succeeds, account not locked, counter reset to 0.

**TC-LOCK-016 - 5 failures: locked (N)**
- Preconditions: counter = 0.
- Steps: 1. Fail 5 times. 2. Log in with the valid password.
- Expected: Login rejected, account locked.

**TC-LOCK-017 - 6th attempt while locked (N+1)**
- Preconditions: 5 failures done.
- Steps: 1. Make a 6th attempt with wrong password.
- Expected: Rejected as locked. Account remains locked. Lock expiry unchanged (TBC, OQ-3).

**TC-LOCK-018 - Login at 14:59 after lock is rejected**
- Preconditions: locked at T.
- Steps: 1. At T + 14:59, log in with the valid password.
- Expected: Rejected, account still locked.

**TC-LOCK-019 - Login at exactly 15:00 after lock**
- Preconditions: locked at T.
- Steps: 1. At T + 15:00.000, log in with the valid password.
- Expected: TBC, see OQ-2 (is the boundary inclusive?). Recommended to be confirmed with the product owner before execution.

**TC-LOCK-020 - Login at 15:01 after lock succeeds**
- Preconditions: locked at T.
- Steps: 1. At T + 15:01, log in with the valid password.
- Expected: Login succeeds.

**TC-LOCK-021 - Fifth attempt is the successful one**
- Preconditions: counter = 0.
- Steps: 1. Fail 4 times. 2. 5th attempt with valid password.
- Expected: Login succeeds, no lock, counter = 0.

**TC-LOCK-022 - Lock timer starts at the 5th failure, not the first**
- Preconditions: counter = 0.
- Steps: 1. Fail attempt 1 at T0. 2. Fail attempts 2-5 at T0 + 10 min. 3. At T0 + 10 + 15 min, log in with valid password. 4. In a separate run, log in at T0 + 15 min.
- Expected: Step 3 succeeds. Step 4 is rejected (lock began at the 5th failure, TBC OQ-2 on the exact reference time).

**TC-LOCK-023 - Failures spread over a long period (time window)**
- Preconditions: counter = 0.
- Steps: 1. Fail once per day for 5 days with no success in between.
- Expected: TBC, see OQ-1 (is there a time window for "consecutive"?). Literal reading: account is locked on day 5.

**TC-LOCK-024 - Lock after the unlock cycle repeats**
- Preconditions: locked and auto-unlocked once.
- Steps: 1. Fail 5 more times.
- Expected: Locked again for 15 minutes (not longer, unless progressive lockout is specified, see OQ-7).

## Open questions (requirement ambiguous or incomplete)

1. **Window for "consecutive":** Is there a time window (e.g. 5 failures within 10 minutes), or do failures count indefinitely until a success? Does the counter expire?
2. **15-minute reference and boundary:** Is the 15 minutes measured from the 5th failed attempt or from the time of the lock being applied? Is unlock at exactly 15:00 inclusive (>=) or exclusive? Which clock/time zone is authoritative (server)?
3. **Attempts during the lock:** Do login attempts (wrong or correct) while locked extend the lock, restart the 15 minutes, or increment the counter?
4. **User-facing behaviour:** What message is shown on the 5th failure and while locked? May it reveal that the account is locked (usability) or must it stay generic (account enumeration risk)? Is the remaining time shown?
5. **Counter after auto-unlock:** Is the counter reset to 0 on auto-unlock? The requirement only says a successful login resets it. If not reset, would the first failure after unlock re-lock the account immediately?
6. **Other unlock paths:** Can an admin unlock or can the user unlock via password reset / email link before 15 minutes? Does a password reset reset the counter?
7. **Repeat lockouts:** Is the duration always 15 minutes, or progressive for repeated lockouts?
8. **Non-existent accounts:** Are failures for unknown usernames tracked (e.g. per IP) and is the response the same as for real accounts?
9. **Scope of "login":** Does this apply to all channels (web, mobile app, API, SSO, MFA step)? Does a failed MFA code count as a failed login attempt? Are multiple password-related endpoints (e.g. password change, "confirm password") counted?
10. **What is a "failed attempt":** Only wrong password, or also wrong username, expired password, disabled account, malformed request, captcha failure?
11. **Active sessions:** If the user has an existing session when the account is locked (e.g. failures from another device), is that session terminated?
12. **Concurrency:** Expected behaviour for simultaneous attempts (race at the 5th failure)?
13. **Audit/notification:** Must lock and unlock events be logged and the user notified (email)? 
14. **Persistence:** Does the lock survive restarts/deployments and work across multiple servers (shared state)?
15. **Rate of attempts / non-functional:** Any expected response-time behaviour or other throttling that interacts with this rule?
