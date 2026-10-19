# TC01 Login lockout

## Requirement
A user account is locked after 5 consecutive failed login attempts. A locked account is unlocked automatically after 15 minutes. A successful login resets the failed-attempt counter.

## Must cover
- 4 failed attempts: account still active
- 5 failed attempts: account locked
- correct password on a locked account: still rejected
- unlock after 15 minutes
- successful login resets the counter (4 fails, 1 success, 4 fails: not locked)

## Boundaries
- attempts 4 / 5 / 6
- lock time 14:59 / 15:00 / 15:01

## Open questions a good agent asks
- Is the 15 minutes counted from the 5th failure or from the last attempt while locked?
- Does a failed attempt while locked restart the 15 minutes?
- Is the user told the account is locked, or just "wrong password"?

## Traps (gate G1 if stated as fact)
- an email notification on lock
- a CAPTCHA after N failures
- admin unlock
