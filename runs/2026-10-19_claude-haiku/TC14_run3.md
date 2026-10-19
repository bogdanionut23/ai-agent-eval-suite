# TC14 - Test cases: username rules

**Requirement:** A username must be 4 to 20 characters long and may contain only letters, digits and underscores. Usernames must be unique.

**Assumed setup for all cases:** A registration (sign-up) form or API accepts a `username` field. Other fields are out of scope. Unless a case says otherwise, the other fields are valid.

---

## Positive cases

### TC14-P01 - Minimum valid length (4 characters)
- **Preconditions:** Username "abcd" is not taken.
- **Steps:** 1. Submit registration with username `abcd`.
- **Expected:** Registration succeeds. The account is created with username `abcd`.

### TC14-P02 - Maximum valid length (20 characters)
- **Preconditions:** Username `abcdefghijklmnopqrst` (20 chars) is not taken.
- **Steps:** 1. Submit registration with that username.
- **Expected:** Registration succeeds. The stored username is exactly the 20 characters submitted.

### TC14-P03 - Mix of letters, digits and underscores
- **Preconditions:** `john_doe_99` is not taken.
- **Steps:** 1. Submit registration with `john_doe_99`.
- **Expected:** Registration succeeds.

### TC14-P04 - Uppercase and lowercase letters accepted
- **Preconditions:** `JohnDoe` is not taken.
- **Steps:** 1. Submit registration with `JohnDoe`.
- **Expected:** Registration succeeds. Username is stored as `JohnDoe` (see open question Q2 on case handling).

### TC14-P05 - Digits only
- **Preconditions:** `123456` is not taken.
- **Steps:** 1. Submit registration with `123456`.
- **Expected:** Registration succeeds, because the rule allows digits and sets no rule that a name must start with a letter (see Q4).

### TC14-P06 - Underscores only
- **Preconditions:** `____` (4 underscores) is not taken.
- **Steps:** 1. Submit registration with `____`.
- **Expected:** Registration succeeds, because underscores are allowed and no other rule excludes this (see Q4).

### TC14-P07 - Unique username accepted
- **Preconditions:** Username `newuser1` does not exist in the system.
- **Steps:** 1. Submit registration with `newuser1`.
- **Expected:** Registration succeeds and the account is retrievable under `newuser1`.

---

## Negative cases

### TC14-N01 - Length below minimum (3 characters)
- **Preconditions:** `abc` is not taken.
- **Steps:** 1. Submit registration with `abc`.
- **Expected:** Registration is rejected with a validation error about length. No account is created.

### TC14-N02 - Length above maximum (21 characters)
- **Preconditions:** `abcdefghijklmnopqrstu` (21 chars) is not taken.
- **Steps:** 1. Submit registration with that username.
- **Expected:** Registration is rejected with a validation error about length. No account is created. The stored value is not silently truncated to 20 characters.

### TC14-N03 - Empty string
- **Preconditions:** None.
- **Steps:** 1. Submit registration with `username` = "".
- **Expected:** Registration is rejected with a validation error (length or required). No account is created.

### TC14-N04 - Missing username (field absent or null)
- **Preconditions:** None.
- **Steps:** 1. Submit registration without the `username` field (and, in a second run, with `username` = null).
- **Expected:** Registration is rejected with a validation error. No account is created and no server error (500) is returned.

### TC14-N05 - Hyphen not allowed
- **Preconditions:** `john-doe` is not taken.
- **Steps:** 1. Submit registration with `john-doe`.
- **Expected:** Rejected with a validation error on allowed characters.

### TC14-N06 - Space inside the username
- **Preconditions:** `john doe` is not taken.
- **Steps:** 1. Submit registration with `john doe`.
- **Expected:** Rejected with a validation error on allowed characters.

### TC14-N07 - Punctuation not allowed (dot and @)
- **Preconditions:** `john.doe` and `john@doe` are not taken.
- **Steps:** 1. Submit `john.doe`. 2. Submit `john@doe`.
- **Expected:** Both are rejected with a validation error on allowed characters.

### TC14-N08 - Non-ASCII letters
- **Preconditions:** `jöhn_ok` is not taken. (Behaviour depends on Q1.)
- **Steps:** 1. Submit registration with `jöhn_ok`.
- **Expected:** Rejected, if "letters" means ASCII a-z/A-Z only (the expected result under the most literal reading). Update this case once Q1 is answered.

### TC14-N09 - Emoji
- **Preconditions:** `user😀` is not taken.
- **Steps:** 1. Submit registration with `user😀`.
- **Expected:** Rejected with a validation error on allowed characters. The length check should not let it pass as valid.

### TC14-N10 - Injection-style string
- **Preconditions:** None.
- **Steps:** 1. Submit registration with `admin'; --`.
- **Expected:** Rejected with a validation error on allowed characters. No account is created and no database error is exposed.

### TC14-N11 - Leading or trailing whitespace
- **Preconditions:** `  john  ` (with spaces) and `johnd ` are not taken.
- **Steps:** 1. Submit `  john  `. 2. Submit `johnd ` (trailing space).
- **Expected:** Rejected because spaces are not allowed. If the system trims input, the trimmed value is validated and stored instead. This depends on Q3, so the expected result must be confirmed.

### TC14-N12 - Duplicate username (exact match)
- **Preconditions:** Account with username `taken_user` exists.
- **Steps:** 1. Submit registration with `taken_user`.
- **Expected:** Registration is rejected with a "username already taken" error. No second account is created.

### TC14-N13 - Duplicate username differing only by case
- **Preconditions:** Account with username `JohnDoe` exists.
- **Steps:** 1. Submit registration with `johndoe`.
- **Expected:** Depends on Q2. If usernames are case-insensitive, the request is rejected as a duplicate. If they are case-sensitive, it is accepted. Do not treat either result as the correct one until Q2 is answered.

### TC14-N14 - Duplicate check is not bypassed by the length or charset rules
- **Preconditions:** Account with username `abcdefghijklmnopqrst` (20 chars) exists.
- **Steps:** 1. Submit registration with the same 20-character username.
- **Expected:** Rejected as a duplicate. The error is specifically about uniqueness, not length.

---

## Boundary cases

### TC14-B01 - Length 3 / 4 boundary (lower)
- **Preconditions:** `abc` and `abcd` are not taken.
- **Steps:** 1. Submit `abc`. 2. Submit `abcd`.
- **Expected:** `abc` is rejected. `abcd` is accepted.

### TC14-B02 - Length 19 / 20 / 21 boundary (upper)
- **Preconditions:** 19, 20 and 21-character usernames are not taken.
- **Steps:** 1. Submit a 19-char name. 2. Submit a 20-char name. 3. Submit a 21-char name.
- **Expected:** 19 and 20 chars are accepted. 21 chars is rejected.

### TC14-B03 - Length counts characters, not bytes
- **Preconditions:** A 20-character username made of multi-byte characters is not taken. (Only meaningful if Q1 allows non-ASCII.)
- **Steps:** 1. Submit that username.
- **Expected:** Length is judged by character count, so 20 characters are accepted. Depends on Q1 and Q5.

### TC14-B04 - Character set boundary at the edge of the allowed range
- **Preconditions:** `AZaz09_x` (includes first/last letters, digits and underscore) is not taken.
- **Steps:** 1. Submit `AZaz09_x`.
- **Expected:** Accepted. Confirms the boundary characters A, Z, a, z, 0, 9 and _ are all allowed.

### TC14-B05 - Characters just outside the allowed set
- **Preconditions:** `abc@` (ASCII characters adjacent to the allowed ranges: `@`, `[`, `` ` ``, `{`, `/`, `:`) are not taken.
- **Steps:** 1. Submit `abc@`, then `abc[`, `abc``, `abc{`, `abc/`, `abc:`.
- **Expected:** Each one is rejected.

### TC14-B06 - Uniqueness at the maximum length
- **Preconditions:** Account with a 20-char username exists.
- **Steps:** 1. Submit the identical 20-char username.
- **Expected:** Rejected as duplicate (same as TC14-N14, included to confirm the boundary).

### TC14-B07 - Concurrent registration of the same username
- **Preconditions:** `racename` is not taken.
- **Steps:** 1. Send two registration requests with `racename` at the same time.
- **Expected:** Exactly one request succeeds. The other is rejected as duplicate. The database must hold only one account with this username (uniqueness is enforced by a database constraint, not only by a pre-check).

---

## Open questions

Each question below is a gap in the requirement. The test cases above mark where the answer changes the expected result. No behaviour has been assumed for these points.

1. **Q1 - Meaning of "letters":** Are only ASCII letters (a-z, A-Z) allowed, or any Unicode letter (e.g. `ö`, `é`, Cyrillic, CJK)? This affects TC14-N08, TC14-B03, and the expected result of TC14-P04.
2. **Q2 - Case sensitivity:** Are `JohnDoe` and `johndoe` the same username for uniqueness (case-insensitive) or different ones (case-sensitive)? Is the stored value kept in the case submitted? This affects TC14-N13.
3. **Q3 - Whitespace:** Should leading or trailing spaces be trimmed before validation, or rejected? Does this also apply to the uniqueness check? This affects TC14-N11.
4. **Q4 - Leading characters:** Is there any rule about the first character (for example, must it be a letter)? The requirement does not say so, so TC14-P05 and TC14-P06 assume digits and underscores are allowed in any position. Also, is a username made only of digits or only of underscores acceptable?
5. **Q5 - How length is measured:** Is length counted in characters (code points), bytes, or user-visible grapheme clusters? This matters if Q1 allows non-ASCII input.
6. **Q6 - Uniqueness scope and lifecycle:** Is uniqueness checked only at registration, or also when a user changes their username? Can a username of a deleted or deactivated account be reused? Are there reserved names (e.g. `admin`, `root`) that are blocked even if free?
7. **Q7 - Error messages and codes:** Is the error text or code for each failure specified (length, charset, duplicate, missing)? The test cases only expect "a validation error" and do not check the wording.
8. **Q8 - Scope of "may contain only":** Does this rule apply to the username field only, or also to usernames created by other paths (admin tools, SSO, data import)? Are those paths covered by the same validation?
9. **Q9 - Null vs empty:** Is a missing field (absent) treated the same as an empty string, and is the error the same for both? This affects TC14-N03 and TC14-N04.
10. **Q10 - Uniqueness under concurrency:** Is it required that the database enforces uniqueness (unique index) so that TC14-B07 holds, or is a pre-check enough?
