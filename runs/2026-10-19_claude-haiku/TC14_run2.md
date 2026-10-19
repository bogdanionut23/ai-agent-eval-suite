# TC14: Username validation test cases

Requirement: A username must be 4 to 20 characters long and may contain only letters, digits and underscores. Usernames must be unique.

Test cases that depend on an unclear point are marked **[Blocked by open question Qn]**. Their expected result is the behaviour the requirement implies, but it must be confirmed before the test is run.

## Positive scenarios

### TC14-01: Minimum valid length (4 characters)
- **Preconditions:** Registration form is available. No account uses the name "abcd".
- **Steps:** 1. Enter `abcd` as the username. 2. Complete the rest of the registration form with valid data. 3. Submit.
- **Expected result:** The username is accepted and the account is created with username `abcd`.

### TC14-02: Maximum valid length (20 characters)
- **Preconditions:** Registration form is available. No account uses the name `abcdefghijklmnopqrst`.
- **Steps:** 1. Enter `abcdefghijklmnopqrst` (20 characters). 2. Complete the form and submit.
- **Expected result:** The username is accepted and the account is created.

### TC14-03: Letters, digits and underscores together
- **Preconditions:** Registration form is available.
- **Steps:** 1. Enter `user_01`. 2. Complete the form and submit.
- **Expected result:** The username is accepted.

### TC14-04: Uppercase and lowercase letters are both allowed
- **Preconditions:** Registration form is available. No account uses `ABCdef`.
- **Steps:** 1. Enter `ABCdef`. 2. Complete the form and submit.
- **Expected result:** The username is accepted and stored in the case it was entered (`ABCdef`).

### TC14-05: Digits only
- **Preconditions:** Registration form is available. No account uses `1234`.
- **Steps:** 1. Enter `1234`. 2. Complete the form and submit.
- **Expected result:** The username is accepted, because the requirement allows digits and sets no rule about the first character. **[Blocked by open question Q4]**

### TC14-06: Underscores only, at minimum length
- **Preconditions:** Registration form is available.
- **Steps:** 1. Enter `____` (4 underscores). 2. Complete the form and submit.
- **Expected result:** The username is accepted, because underscores are allowed and the length is 4. **[Blocked by open question Q5]**

### TC14-07: Unique username is accepted
- **Preconditions:** Username `fresh_name` does not exist in the system.
- **Steps:** 1. Register with username `fresh_name`. 2. Submit.
- **Expected result:** The account is created and the username is reserved for it.

## Negative scenarios

### TC14-08: Username shorter than 4 characters (3 characters)
- **Preconditions:** Registration form is available.
- **Steps:** 1. Enter `abc`. 2. Submit.
- **Expected result:** Registration is rejected with a validation error that refers to the length rule. No account is created.

### TC14-09: Username longer than 20 characters (21 characters)
- **Preconditions:** Registration form is available.
- **Steps:** 1. Enter `abcdefghijklmnopqrstu` (21 characters). 2. Submit.
- **Expected result:** Registration is rejected with a validation error that refers to the length rule. No account is created.

### TC14-10: Empty username
- **Preconditions:** Registration form is available.
- **Steps:** 1. Leave the username field empty. 2. Submit.
- **Expected result:** Registration is rejected with a validation error. No account is created.

### TC14-11: Username missing from the request (null)
- **Preconditions:** Registration API endpoint is reachable.
- **Steps:** 1. Send a registration request with no username field, or with `username: null`. 2. Observe the response.
- **Expected result:** The request is rejected with a client error (for example HTTP 400). No account is created.

### TC14-12: Space inside the username
- **Preconditions:** Registration form is available.
- **Steps:** 1. Enter `user name`. 2. Submit.
- **Expected result:** Registration is rejected because a space is not a letter, digit or underscore.

### TC14-13: Hyphen in the username
- **Preconditions:** Registration form is available.
- **Steps:** 1. Enter `user-name`. 2. Submit.
- **Expected result:** Registration is rejected because a hyphen is not allowed.

### TC14-14: Special characters
- **Preconditions:** Registration form is available.
- **Steps:** 1. Enter `user@1`. 2. Repeat with `user.1`, `user#1` and `user$1`, one at a time. 3. Submit each.
- **Expected result:** Each attempt is rejected. No account is created.

### TC14-15: Leading or trailing whitespace
- **Preconditions:** Registration form is available. No account uses `user1`.
- **Steps:** 1. Enter ` user1 ` (with spaces at both ends). 2. Submit.
- **Expected result:** Registration is rejected, because spaces are not allowed characters. If the product trims whitespace before validation, the result must instead match the trimmed value. **[Blocked by open question Q3]**

### TC14-16: Emoji in the username
- **Preconditions:** Registration form is available.
- **Steps:** 1. Enter `user😀1`. 2. Submit.
- **Expected result:** Registration is rejected because an emoji is not a letter, digit or underscore.

### TC14-17: Non-ASCII letters
- **Preconditions:** Registration form is available.
- **Steps:** 1. Enter `Jósé`. 2. Submit.
- **Expected result:** Behaviour depends on whether "letters" means ASCII letters only. **[Blocked by open question Q1]**

### TC14-18: Duplicate username (exact match)
- **Preconditions:** An account with username `taken_name` exists.
- **Steps:** 1. Register a new account with username `taken_name`. 2. Submit.
- **Expected result:** Registration is rejected with a message that the username is already taken. The existing account is unchanged and no second account is created.

### TC14-19: Duplicate username differing only in case
- **Preconditions:** An account with username `Alice_1` exists.
- **Steps:** 1. Register a new account with username `alice_1`. 2. Submit.
- **Expected result:** Depends on whether uniqueness is case-sensitive. **[Blocked by open question Q2]**

### TC14-20: Two simultaneous registrations with the same username
- **Preconditions:** No account uses `race_user`. Two clients are ready to submit at the same time.
- **Steps:** 1. Send two registration requests with username `race_user` at the same moment. 2. Compare the responses and check the database.
- **Expected result:** Exactly one account is created with `race_user`. The other request is rejected as a duplicate. The database holds no two rows with that username.

## Boundary scenarios

### TC14-21: Length boundary at the lower edge (3 and 4 characters)
- **Preconditions:** Registration form is available.
- **Steps:** 1. Submit `abc` (3 characters). 2. Submit `abcd` (4 characters).
- **Expected result:** `abc` is rejected. `abcd` is accepted.

### TC14-22: Length boundary at the upper edge (20 and 21 characters)
- **Preconditions:** Registration form is available.
- **Steps:** 1. Submit a 20-character username. 2. Submit a 21-character username.
- **Expected result:** The 20-character username is accepted. The 21-character username is rejected.

### TC14-23: Length counts characters, not bytes
- **Preconditions:** Registration form is available.
- **Steps:** 1. Submit a 20-character username that uses only ASCII. 2. Submit a 20-character username that contains non-ASCII letters (for example `ñññ...`), if non-ASCII letters are allowed.
- **Expected result:** Length is measured in characters. Both are accepted if their character counts are within range. **[Blocked by open questions Q1 and Q8]**

### TC14-24: Uniqueness is checked against the stored value after validation
- **Preconditions:** An account with username `boundary_1` exists.
- **Steps:** 1. Submit `boundary_1`, which is valid in format but already taken. 2. Submit `boundary_2`, which is valid and free.
- **Expected result:** The first submission is rejected as a duplicate. The second is accepted. Format validation and uniqueness do not interfere with each other.

## Open questions

1. **Q1 (character set):** Does "letters" mean only ASCII `a-z` and `A-Z`, or any Unicode letter? Are accented letters such as `é` or `ñ` allowed?
2. **Q2 (uniqueness and case):** Is uniqueness case-sensitive, so that `Alice` and `alice` can both exist? If not, is the comparison case-insensitive only for ASCII, or also for Unicode?
3. **Q3 (whitespace):** Are leading or trailing spaces trimmed before validation, or rejected?
4. **Q4 (first character):** May a username start with a digit or an underscore?
5. **Q5 (all-underscore or all-digit names):** Are names such as `____` or `1234` allowed? Are there reserved names (for example `admin`, `root`, `support`)?
6. **Q6 (length unit):** Is length counted in characters, Unicode code points or grapheme clusters? This matters for emoji and combining marks.
7. **Q7 (change of username):** Can a user change their username after registration? If so, do the same length, character and uniqueness rules apply?
8. **Q8 (reuse after deletion):** When an account is deleted or deactivated, is its username released for reuse?
9. **Q9 (concurrency and enforcement):** Is uniqueness enforced by a database constraint, or only by an application-level check? Which response does a losing concurrent request receive?
10. **Q10 (error messages and layers):** Which error message or error code is shown for each rule? Is validation done on the client, the server, or both? Should the server rule apply to direct API calls?
11. **Q11 (empty versus missing):** Should an empty string and a missing field produce the same error?
12. **Q12 (case preservation):** Is the username stored and displayed in the case it was entered, or normalized to lowercase?
