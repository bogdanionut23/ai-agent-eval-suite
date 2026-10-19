# Test cases: account lockout after failed logins

## Requirement

A user account is locked after 5 consecutive failed login attempts. A locked account is unlocked automatically after 15 minutes. A successful login resets the failed-attempt counter.

## Assumptions used for the test cases

Only behaviour stated in the requirement is treated as specified. Where a test depends on a detail the requirement does not give, the test is marked with the open question it depends on (OQ-n, see the last section). Common preconditions below apply unless a test says otherwise:

- An active account `user_A` exists with known password `Correct#1`.
- The failed-attempt counter for `user_A` is 0 and the account is unlocked.
- Login is attempted through the standard login form or endpoint with username and password.
- Failed login = wrong password for an existing active account.

## Positive scenarios

### TC-01: Four consecutive failures do not lock the account

- **Preconditions:** `user_A` is active, unlocked, counter = 0.
- **Steps:**
  1. Submit `user_A` with a wrong password. Record the response.
  2. Repeat step 1 three more times (4 failures in total).
  3. Submit `user_A` with the correct password `Correct#1`.
- **Expected result:**
  - Attempts 1 to 4 are rejected with the standard invalid-credentials response. The response does not state that the account is locked.
  - Attempt 5 (correct password) succeeds and a session is created.
  - The counter is reset to 0 (checked through TC-05).

### TC-02: The fifth consecutive failure locks the account

- **Preconditions:** `user_A` is active, unlocked, counter = 0.
- **Steps:**
  1. Submit `user_A` with a wrong password 5 times in a row.
  2. Check the account state (admin view or API, if available).
  3. Submit `user_A` with the correct password.
- **Expected result:**
  - Attempts 1 to 4 return the invalid-credentials response.
  - Attempt 5 is rejected and the account is reported as locked (or the lock is otherwise visible in the account state). Exact message text is not specified; see OQ-3.
  - Account state is locked.
  - Step 3 is rejected, with no session created.

### TC-03: A locked account rejects the correct password

- **Preconditions:** `user_A` is locked by TC-02 (5 consecutive failures), less than 15 minutes ago.
- **Steps:**
  1. Submit `user_A` with the correct password `Correct#1`.
- **Expected result:** Login is rejected with the locked-account response. No session is created.

### TC-04: A locked account is unlocked automatically after 15 minutes

- **Preconditions:** `user_A` locked by 5 consecutive failures at time T0. No login attempts made in between (see TC-08 for attempts during the lock).
- **Steps:**
  1. Advance time to T0 + 15 min + 1 s (or wait 15 min in a real environment).
  2. Submit `user_A` with the correct password `Correct#1`.
- **Expected result:**
  - Login succeeds and a session is created.
  - Account state is unlocked.
  - Counter is 0 after the successful login.

### TC-05: A successful login resets the failed-attempt counter

- **Preconditions:** `user_A` is active, unlocked, counter = 0.
- **Steps:**
  1. Submit wrong password 3 times.
  2. Submit the correct password (success). Log out.
  3. Submit wrong password 4 times.
  4. Check the account state.
  5. Submit one more wrong password (5th failure after the reset).
- **Expected result:**
  - Step 2 succeeds and the counter is reset to 0.
  - After step 3 (4 failures since the reset) the account is not locked. Step 4 shows unlocked.
  - Step 5 locks the account. The 3 failures before the success did not count toward the threshold.

### TC-06: Counter starts from zero after an automatic unlock

- **Preconditions:** `user_A` was locked and has automatically unlocked after 15 minutes (TC-04 path, but the user does not log in during the unlock step).
- **Steps:**
  1. Submit wrong password 4 times.
  2. Check the account state.
  3. Submit wrong password once more.
  4. Check the account state.
- **Expected result:**
  - After step 1 the account is not locked.
  - After step 3 (5 failures since the unlock) the account is locked.
  - The failures from before the lock do not carry over. Exact counter state after unlock depends on OQ-7, but a locked-then-unlocked account must get the full 5-failure allowance again.

### TC-07: Counter is per account, not global

- **Preconditions:** `user_A` and `user_B` are both active, unlocked, counter = 0.
- **Steps:**
  1. Submit wrong password 5 times for `user_A`. Confirm `user_A` is locked.
  2. Submit the correct password for `user_B`.
- **Expected result:** `user_B` login succeeds. Locking `user_A` does not affect `user_B`.

## Negative scenarios

### TC-08: Attempts made during the lock do not unlock the account early

- **Preconditions:** `user_A` locked at T0 by TC-02.
- **Steps:**
  1. At T0 + 5 min, submit the correct password. Expected: rejected (see TC-03).
  2. At T0 + 10 min, submit a wrong password.
  3. At T0 + 15 min + 1 s, submit the correct password.
- **Expected result:**
  - Steps 1 and 2 are rejected. The account stays locked.
  - Step 3 succeeds, so the lock ended at the 15-minute mark and was not extended by the attempts in step 2.
  - Whether the lock timer is extended by attempts during the lock is OQ-4. Step 3 is valid only if OQ-4 is answered "no extension".

### TC-09: Login is still rejected one second before the 15-minute mark

- **Preconditions:** `user_A` locked at T0.
- **Steps:**
  1. Advance time to T0 + 14 min 59 s.
  2. Submit the correct password.
- **Expected result:** Login is rejected with the locked-account response. The account stays locked.

### TC-10: Failures on one account do not affect a different account

- **Preconditions:** `user_A` and `user_B` active, counter = 0.
- **Steps:**
  1. Submit 4 wrong passwords for `user_A`.
  2. Submit 4 wrong passwords for `user_B`.
  3. Submit 1 wrong password for `user_A`.
  4. Submit the correct password for `user_B`.
- **Expected result:**
  - Step 3 locks `user_A` (5 consecutive failures on `user_A`).
  - `user_B` has 4 failures and is not locked, so step 4 succeeds.

### TC-11: A successful login on another account does not reset this account's counter

- **Preconditions:** `user_A` and `user_B` active, counter = 0.
- **Steps:**
  1. Submit 4 wrong passwords for `user_A`.
  2. Log in successfully as `user_B`.
  3. Submit 1 wrong password for `user_A`.
- **Expected result:** `user_A` is locked after step 3. The success on `user_B` did not reset `user_A`'s counter.

### TC-12: Four failures followed by a success does not produce a lock on a later single failure

- **Preconditions:** `user_A` active, counter = 0.
- **Steps:**
  1. Submit 4 wrong passwords.
  2. Submit the correct password (success). Log out.
  3. Submit 1 wrong password.
  4. Check the account state.
- **Expected result:** Account is not locked after step 4. Attempts before the success are not carried forward.

### TC-13: Non-existent username does not create or lock an account

- **Preconditions:** Username `ghost_user` does not exist.
- **Steps:**
  1. Submit `ghost_user` with any password 5 times.
- **Expected result:**
  - Each attempt returns the standard invalid-credentials response.
  - No account is created or locked.
  - The response is the same as for a real account with a wrong password, so attackers cannot tell which usernames exist. Exact requirement is in OQ-5.

### TC-14: Failed attempts on an inactive or disabled account (not part of lockout)

- **Preconditions:** `user_D` exists but is disabled by an admin.
- **Steps:**
  1. Submit `user_D` with the correct password.
- **Expected result:** Login is rejected because the account is disabled. The lock logic does not apply, and the rejection reason is the disabled-account reason, not a lockout reason. Behaviour for failed attempts on disabled accounts is OQ-6.

## Boundary scenarios

### TC-15: Four failures is below the threshold (boundary below)

Covered by TC-01 (steps 1 to 4 and step 5 success). Kept as a separate case for the exact threshold: count 4 is not locked.

- **Preconditions:** `user_A` active, counter = 0.
- **Steps:** Submit 4 wrong passwords. Check the account state.
- **Expected result:** Account is unlocked.

### TC-16: Five failures is exactly the threshold (boundary at)

Covered by TC-02. Kept as a separate case for the exact threshold: count 5 is locked.

- **Preconditions:** `user_A` active, counter = 0.
- **Steps:** Submit 5 wrong passwords. Check the account state.
- **Expected result:** Account is locked.

### TC-17: Correct password on the fifth attempt succeeds (boundary just under lock)

- **Preconditions:** `user_A` active, counter = 0.
- **Steps:**
  1. Submit 4 wrong passwords.
  2. Submit the correct password.
- **Expected result:** Login succeeds. The account is not locked at any point. Counter is 0 afterwards.

### TC-18: Lock expiry at exactly 15 minutes

- **Preconditions:** `user_A` locked at T0.
- **Steps:**
  1. Advance time to T0 + 15 min 00 s exactly.
  2. Submit the correct password.
- **Expected result:** Behaviour at exactly 15 min 00 s (locked or unlocked) is OQ-2. The test records the result, and the expected result is set once OQ-2 is answered. The result at T0 + 15 min 01 s is covered by TC-04.

### TC-19: Five failures in quick succession under concurrency

- **Preconditions:** `user_A` active, counter = 0.
- **Steps:**
  1. Send 5 wrong-password login requests for `user_A` concurrently (within the same second).
  2. Check the account state and counter.
- **Expected result:**
  - Account is locked.
  - The counter is exactly 5, with no lost updates. Each request is recorded as a failure.
  - No more than 5 failures are needed to lock.

### TC-20: Sixth attempt after the lock is rejected

- **Preconditions:** `user_A` locked by 5 failures at T0.
- **Steps:**
  1. Submit a wrong password (6th failure).
  2. Submit the correct password within the lock window.
- **Expected result:** Both are rejected. The 6th failure does not change the lock expiry time (see OQ-4) and does not change the lock reason.

## Open questions

These are not answered by the requirement. Tests that depend on them are marked above.

- **OQ-1: Scope of "consecutive".** Are failures counted per account across all clients, IPs and devices? Is there a time window after which the counter resets even without a success (for example 30 minutes of inactivity)?
- **OQ-2: Exact expiry boundary.** Is the account unlocked at exactly 15 min 00 s, or only after it? Is the lock time measured from the 5th failure or from the moment the lock is written?
- **OQ-3: Response to the user.** What is the message and HTTP status for a locked account? Is the lock disclosed to the user at all, or does the response look like a normal wrong-password response (which would hide the lock from an attacker)?
- **OQ-4: Attempts during the lock.** Do wrong-password attempts during the lock extend the lock timer, or increment the counter beyond 5? Is a correct password during the lock rejected without any effect on the timer?
- **OQ-5: Non-existent usernames.** Do failed attempts on unknown usernames count anywhere? Must the response be identical to a wrong password on a real account?
- **OQ-6: Disabled or inactive accounts.** Do failed attempts on a disabled account count toward the lock? What is the response?
- **OQ-7: Counter after unlock.** Is the counter reset to 0 when the lock expires, or when the next successful login happens? If it is reset only on success, is the counter still 5 right after the automatic unlock?
- **OQ-8: Unlock mechanism.** Is the unlock time-based only, or is it lazy (checked at the next login attempt) or done by a background job? Does an unlock send a notification?
- **OQ-9: Other reset events.** Does a password reset, password change, or admin unlock also reset the counter or lift the lock? Does a successful login through SSO, MFA, or API token count as a "successful login" that resets the counter?
- **OQ-10: Persistence and clock.** Is the lock time stored durably, so it survives a server restart or failover? Which clock is used (server time, UTC)?
- **OQ-11: Concurrency.** Is the counter increment atomic across parallel requests (see TC-19)? What is the expected result when a correct and a wrong password arrive at the same time during the 5th failure?
- **OQ-12: Scope of the lock across channels.** Does the lock apply to every login channel (web, mobile, API, SSO), or only to the password form?
- **OQ-13: Notifications.** Should the user be notified by email or otherwise when the account is locked?
