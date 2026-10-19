# TC14 Run 1: Test cases for username rules

**Requirement:** A username must be 4 to 20 characters long and may contain only letters, digits and underscores. Usernames must be unique.

**Working assumptions (used only where the requirement states a rule directly):**
- "Length" means number of characters, and the limits are inclusive (4 and 20 are valid).
- "Letters" and "digits" are read literally as the ASCII sets a-z, A-Z, 0-9. The non-ASCII case is covered by an open question, not a test expectation.
- Any character outside letters, digits and underscore makes the username invalid, including spaces.
- The username is validated as submitted. No trimming is assumed, so leading or trailing spaces count as characters that break the rule.

---

## 1. Positive scenarios

### TC14-P01: Accept a username at the minimum length (4 characters)
- **Preconditions:** No account exists with username `abcd`. The registration form (or API endpoint) is available.
- **Steps:**
  1. Enter `abcd` as the username.
  2. Enter valid values for all other required fields.
  3. Submit the registration.
- **Expected result:** The account is created. The username is stored as `abcd`.

### TC14-P02: Accept a username at the maximum length (20 characters)
- **Preconditions:** No account exists with username `abcdefghijklmnopqrst`.
- **Steps:**
  1. Enter `abcdefghijklmnopqrst` (exactly 20 characters).
  2. Submit the registration.
- **Expected result:** The account is created. The username is stored in full, with no truncation.

### TC14-P03: Accept letters, digits and underscores mixed together
- **Preconditions:** No account exists with username `John_99`.
- **Steps:**
  1. Enter `John_99`.
  2. Submit the registration.
- **Expected result:** The account is created with username `John_99`.

### TC14-P04: Accept an uppercase-only username
- **Preconditions:** No account exists with username `ABCDEF`.
- **Steps:**
  1. Enter `ABCDEF`.
  2. Submit the registration.
- **Expected result:** The account is created. Case is kept as entered.

### TC14-P05: Accept a digits-only username
- **Preconditions:** No account exists with username `2024`.
- **Steps:**
  1. Enter `2024`.
  2. Submit the registration.
- **Expected result:** The account is created. The requirement does not forbid a username made only of digits. See OQ-05.

### TC14-P06: Accept a username that starts or ends with an underscore
- **Preconditions:** No account exists with username `_user_`.
- **Steps:**
  1. Enter `_user_` (6 characters).
  2. Submit the registration.
- **Expected result:** The account is created. The requirement allows underscores anywhere. See OQ-06 about leading or trailing underscores.

### TC14-P07: Accept a username made only of underscores at the minimum length
- **Preconditions:** No account exists with username `____`.
- **Steps:**
  1. Enter `____` (4 underscores).
  2. Submit the registration.
- **Expected result:** The account is created. The requirement counts underscores as allowed characters. See OQ-06.

### TC14-P08: Unique username is accepted on availability check
- **Preconditions:** No account exists with username `freshname`. The availability check is available, if the product has one.
- **Steps:**
  1. Enter `freshname` in the username field.
  2. Trigger the availability check, or move focus out of the field.
- **Expected result:** The username is reported as available. No duplicate error is shown.

---

## 2. Negative scenarios

### TC14-N01: Reject a username one character below the minimum (3 characters)
- **Preconditions:** No account exists with username `abc`.
- **Steps:**
  1. Enter `abc`.
  2. Submit the registration.
- **Expected result:** The submission is rejected. A validation message states the length rule. No account is created.

### TC14-N02: Reject a username one character above the maximum (21 characters)
- **Preconditions:** No account exists with username `abcdefghijklmnopqrstu`.
- **Steps:**
  1. Enter `abcdefghijklmnopqrstu` (21 characters).
  2. Submit the registration.
- **Expected result:** The submission is rejected. A validation message states the length rule. No account is created.

### TC14-N03: Reject an empty username
- **Preconditions:** The username field is empty.
- **Steps:**
  1. Leave the username field empty.
  2. Fill in the other fields with valid values.
  3. Submit the registration.
- **Expected result:** The submission is rejected. The message indicates the username is required. No account is created.

### TC14-N04: Reject a whitespace-only username
- **Preconditions:** No account exists with username `"    "` (four spaces).
- **Steps:**
  1. Enter four space characters.
  2. Submit the registration.
- **Expected result:** The submission is rejected. Spaces are not allowed characters, and the value must not be stored as a valid username.

### TC14-N05: Reject a username containing a space in the middle
- **Preconditions:** No account exists with username `john doe`.
- **Steps:**
  1. Enter `john doe`.
  2. Submit the registration.
- **Expected result:** The submission is rejected with a message about invalid characters. No account is created.

### TC14-N06: Reject a leading or trailing space
- **Preconditions:** No account exists with username `john` (the stored value must not be `" john"` or `"john "`).
- **Steps:**
  1. Enter ` john` (leading space).
  2. Submit the registration.
  3. Repeat with `john ` (trailing space).
- **Expected result:** Both submissions are rejected with an invalid-characters message. The system does not silently trim the value. See OQ-03.

### TC14-N07: Reject a hyphen
- **Preconditions:** No account exists with username `john-doe`.
- **Steps:**
  1. Enter `john-doe`.
  2. Submit the registration.
- **Expected result:** The submission is rejected with an invalid-characters message. No account is created.

### TC14-N08: Reject a period
- **Preconditions:** No account exists with username `john.doe`.
- **Steps:**
  1. Enter `john.doe`.
  2. Submit the registration.
- **Expected result:** The submission is rejected with an invalid-characters message.

### TC14-N09: Reject common special characters
- **Preconditions:** No account exists with any of the test values.
- **Steps:**
  1. Submit each of these values, one at a time: `john@doe`, `john!x`, `john#1`, `john$`, `john%`, `john&x`, `john*x`, `john+x`, `john/x`, `john\x`.
- **Expected result:** Each submission is rejected with an invalid-characters message. No account is created for any of them.

### TC14-N10: Reject an emoji
- **Preconditions:** No account exists with username `john😀`.
- **Steps:**
  1. Enter `john😀`.
  2. Submit the registration.
- **Expected result:** The submission is rejected with an invalid-characters message.

### TC14-N11: Reject an injection-style string
- **Preconditions:** No account exists with username `admin'--`.
- **Steps:**
  1. Enter `admin'--`.
  2. Submit the registration.
- **Expected result:** The submission is rejected with an invalid-characters message. No account is created, and the input is not passed into a query or stored as a valid username.

### TC14-N12: Reject a duplicate username (exact match)
- **Preconditions:** An account exists with username `taken_user`.
- **Steps:**
  1. Enter `taken_user`.
  2. Fill in the other fields with valid values.
  3. Submit the registration.
- **Expected result:** The submission is rejected with a message stating the username is already taken. No second account is created. The existing account is unchanged.

### TC14-N13: Reject a duplicate username on the availability check
- **Preconditions:** An account exists with username `taken_user`.
- **Steps:**
  1. Enter `taken_user` in the username field.
  2. Trigger the availability check.
- **Expected result:** The username is reported as unavailable.

### TC14-N14: Reject a missing username field in an API request
- **Preconditions:** The registration API endpoint is available.
- **Steps:**
  1. Send a registration request with no `username` key.
  2. Send the same request with `"username": null`.
- **Expected result:** Both requests return a client error (for example HTTP 400). The response names the username field. No account is created.

### TC14-N15: Reject a non-string username in an API request
- **Preconditions:** The registration API endpoint is available.
- **Steps:**
  1. Send a registration request with `"username": 12345` (number, not string).
- **Expected result:** The request is rejected with a client error, or the value is coerced and then validated under the same rules. Either way, no account is created with a non-string username. See OQ-08.

---

## 3. Boundary scenarios

### TC14-B01: Length boundary table (run as a single data-driven test)

| Input | Length | Expected |
|---|---|---|
| `abc` | 3 | Rejected |
| `abcd` | 4 | Accepted |
| `abcdefghijklmnopqrst` | 20 | Accepted |
| `abcdefghijklmnopqrstu` | 21 | Rejected |

- **Preconditions:** None of the accepted values exist as accounts yet.
- **Steps:** Submit each row as the username, one at a time, and record the outcome.
- **Expected result:** Each row matches the Expected column.

### TC14-B02: Character-set boundary at the edges of the allowed ranges
- **Preconditions:** No account exists with username `Az09_`.
- **Steps:**
  1. Enter `Az09_` (includes the first and last letters, the first and last digit, and an underscore).
  2. Submit the registration.
- **Expected result:** The account is created.

### TC14-B03: Characters just outside the allowed ranges
- **Preconditions:** None of the values exist as accounts.
- **Steps:**
  1. Submit `abc` + the backtick character (`` ` ``, ASCII code point just below `a`, not a letter).
  2. Submit `abc` + `{` (just above `z`).
  3. Submit `abc` + `@` (just below `A`).
  4. Submit `abc` + `[` (just above `Z`).
  5. Submit `abc` + `/` (just below `0`).
  6. Submit `abc` + `:` (just above `9`).
- **Expected result:** Each value is rejected. These are the ASCII neighbours of the allowed ranges, so they show the character-class check is exact.

### TC14-B04: Length counted in characters, not bytes
- **Preconditions:** None.
- **Steps:**
  1. This test is only meaningful if non-ASCII letters are allowed. Defer until OQ-01 is answered.
- **Expected result:** To be defined after OQ-01. Do not run this test until then.

### TC14-B05: Uniqueness at the maximum length
- **Preconditions:** An account exists with username `abcdefghijklmnopqrst` (20 characters).
- **Steps:**
  1. Attempt to register a second account with the same 20-character username.
- **Expected result:** The second registration is rejected as a duplicate. No truncation makes the two values collide or differ incorrectly.

### TC14-B06: Uniqueness at the minimum length
- **Preconditions:** An account exists with username `abcd`.
- **Steps:**
  1. Attempt to register a second account with `abcd`.
- **Expected result:** The second registration is rejected as a duplicate.

### TC14-B07: Near-duplicate usernames are treated as distinct
- **Preconditions:** An account exists with username `user_1`.
- **Steps:**
  1. Register `user_2`.
  2. Register `user_10`.
  3. Register `User_1` (case change only). See OQ-02 for the expected result of this step.
- **Expected result:** `user_2` and `user_10` are accepted. Steps for `User_1` depend on OQ-02.

### TC14-B08: Concurrent registration with the same username
- **Preconditions:** No account exists with username `race_user`. Two registration requests can be sent at the same time.
- **Steps:**
  1. Send two registration requests with username `race_user` at the same time, with different emails.
  2. Wait for both responses.
- **Expected result:** Exactly one account is created. The other request is rejected as a duplicate. The database has no two rows with `race_user`. This checks that uniqueness is enforced at the data layer, not only by a pre-check.

---

## 4. Open questions

These points are not stated in the requirement. The tests above that depend on them are marked, and no behaviour has been assumed for them.

- **OQ-01: Allowed letters.** Does "letters" include non-ASCII letters (for example `jöhn`, `José`, `Ολγα`)? If yes, which scripts? Does this affect the length count, which would then be characters, not bytes? Affects TC14-B04 and any test using accented letters.
- **OQ-02: Case and uniqueness.** Is uniqueness case-sensitive? Are `John` and `john` different usernames? If the answer is case-insensitive, is the stored value kept as entered, or normalized to lower case?
- **OQ-03: Whitespace.** Should the system trim leading and trailing spaces before validation, or reject them? The requirement does not say. TC14-N04 and TC14-N06 assume rejection.
- **OQ-04: Unicode normalization.** Are visually identical strings with different Unicode forms (for example composed and decomposed `é`) treated as the same username?
- **OQ-05: Digits-only usernames.** Is a username made only of digits allowed? The requirement says it may contain digits, and does not forbid all-digit values. TC14-P05 assumes allowed.
- **OQ-06: Underscore position.** Are leading, trailing and repeated underscores allowed? An underscore-only username (TC14-P07) is also unclear.
- **OQ-07: Reserved and offensive names.** Are names such as `admin`, `root`, `support` or profanity reserved or blocked? Not in the requirement.
- **OQ-08: Input types.** For the API, should a non-string value be rejected outright, or coerced to a string and then validated? TC14-N15 accepts either outcome as long as no account is created.
- **OQ-09: Uniqueness scope and lifecycle.** Does a username stay reserved after its account is deleted or deactivated? Can a username be released and reused? Does uniqueness also apply when a username is changed after registration?
- **OQ-10: Where validation happens.** Must the length and character rules be enforced in the UI, the API and the database? Should the same rules apply to a username change, not only to registration?
- **OQ-11: Error messages.** Are the exact validation messages specified? The tests check that a message is shown, not its wording.
- **OQ-12: Availability check.** Is there a real-time availability check? TC14-P08 and TC14-N13 assume one exists. If not, those tests are not needed.
- **OQ-13: Length unit for emoji and combined characters.** If non-ASCII is allowed, how are multi-code-point characters (emoji, combining marks) counted?
