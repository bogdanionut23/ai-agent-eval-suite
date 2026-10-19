# Test Cases: Username Validation

Requirement: A username must be 4 to 20 characters long and may contain only letters, digits and underscores. Usernames must be unique.

## Assumptions (taken literally from the requirement, nothing added)
- "Letters" is tested with ASCII letters A-Z / a-z. Non-ASCII letters are an open question (Q1).
- Uniqueness is tested as exact match. Case sensitivity is an open question (Q2).
- Validation is exercised through the registration (create username) function or form. Error message wording is not specified, so the expected result says "rejected with a validation error" and does not assert specific text.

## Preconditions common to all cases
- The registration function/form is available.
- Unless stated otherwise, no user with the tested username exists yet.
- Existing users used as preconditions are named in the case.

---

## A. Positive cases

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-U-001 | Valid username, letters only, mid-length | Username "johnsmith" not registered | 1. Enter "johnsmith". 2. Submit. | Accepted; account created with username "johnsmith". |
| TC-U-002 | Valid username, letters and digits | "user123" not registered | 1. Enter "user123". 2. Submit. | Accepted. |
| TC-U-003 | Valid username with underscore in middle | "john_smith" not registered | 1. Enter "john_smith". 2. Submit. | Accepted. |
| TC-U-004 | Valid username, digits only (length 6) | "123456" not registered | 1. Enter "123456". 2. Submit. | Accepted (requirement allows digits; no rule requires a letter). See Q4. |
| TC-U-005 | Valid username, underscores only (length 4) | "____" not registered | 1. Enter "____". 2. Submit. | Accepted per literal reading. See Q4. |
| TC-U-006 | Valid username, mixed upper and lower case | "JohnSmith" not registered | 1. Enter "JohnSmith". 2. Submit. | Accepted; stored as entered (or normalised, see Q2). |
| TC-U-007 | Username starting with a digit | "1user" not registered | 1. Enter "1user". 2. Submit. | Accepted (no rule restricts first character). See Q4. |
| TC-U-008 | Username starting and ending with underscore | "_user_" not registered | 1. Enter "_user_". 2. Submit. | Accepted. See Q4. |
| TC-U-009 | Two different valid usernames can both be registered | Neither "alice_1" nor "alice_2" registered | 1. Register "alice_1". 2. Register "alice_2". | Both accepted. |

## B. Boundary cases (length)

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-U-010 | Minimum length: 4 characters | "abcd" not registered | 1. Enter "abcd". 2. Submit. | Accepted. |
| TC-U-011 | Below minimum: 3 characters | None | 1. Enter "abc". 2. Submit. | Rejected with a validation error (too short). No account created. |
| TC-U-012 | Maximum length: 20 characters | "abcdefghij0123456789" (20 chars) not registered | 1. Enter "abcdefghij0123456789". 2. Submit. | Accepted. |
| TC-U-013 | Above maximum: 21 characters | None | 1. Enter "abcdefghij01234567890" (21 chars). 2. Submit. | Rejected with a validation error (too long). No account created. |
| TC-U-014 | Minimum minus many: 1 character | None | 1. Enter "a". 2. Submit. | Rejected (too short). |
| TC-U-015 | Empty username | None | 1. Leave the field empty. 2. Submit. | Rejected (too short / required). |
| TC-U-016 | Far above maximum: 100 characters | None | 1. Enter 100 valid characters. 2. Submit. | Rejected (too long); no crash, truncation or server error. |
| TC-U-017 | Maximum length made of underscores | "____________________" (20) not registered | 1. Enter 20 underscores. 2. Submit. | Accepted per literal reading. See Q4. |
| TC-U-018 | Length counted in characters, not bytes (20 characters, ASCII) | None | 1. Enter a 20-char ASCII name. 2. Submit. | Accepted. (Multi-byte handling is Q1.) |

## C. Negative cases (character set)

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-U-020 | Hyphen not allowed | None | 1. Enter "john-smith". 2. Submit. | Rejected (invalid character). |
| TC-U-021 | Space inside username | None | 1. Enter "john smith". 2. Submit. | Rejected. |
| TC-U-022 | Leading space | None | 1. Enter " johnsmith". 2. Submit. | Rejected, or see Q5 if input is trimmed before validation. |
| TC-U-023 | Trailing space | None | 1. Enter "johnsmith ". 2. Submit. | Rejected, or see Q5. |
| TC-U-024 | Only spaces, 4 characters | None | 1. Enter four spaces. 2. Submit. | Rejected. |
| TC-U-025 | Dot not allowed | None | 1. Enter "john.smith". 2. Submit. | Rejected. |
| TC-U-026 | At sign not allowed | None | 1. Enter "john@smith". 2. Submit. | Rejected. |
| TC-U-027 | Other special characters, each tried separately | None | 1. Enter each of: "user!name", "user#name", "user$name", "user%name", "user&name", "user*name", "user+name", "user=name", "user/name", "user\\name". 2. Submit each. | Every one rejected. |
| TC-U-028 | Accented letter | None | 1. Enter "joséabc" style name, e.g. "josé123". 2. Submit. | Outcome depends on Q1. Record actual behaviour; do not mark pass/fail until answered. |
| TC-U-029 | Non-Latin letters | None | 1. Enter "пользователь" (Cyrillic) and "用户名称测试" (CJK). 2. Submit each. | Outcome depends on Q1. Record actual behaviour. |
| TC-U-030 | Emoji | None | 1. Enter "user😀name". 2. Submit. | Rejected (not a letter, digit or underscore). |
| TC-U-031 | Tab / newline / control character | None | 1. Enter "user\tname" and "user\nname". 2. Submit. | Rejected. |
| TC-U-032 | Zero-width / invisible character | None | 1. Enter "user" + U+200B + "name". 2. Submit. | Rejected. |
| TC-U-033 | Valid length but one invalid char at boundary (20 chars incl. "-") | None | 1. Enter 19 valid chars plus "-". 2. Submit. | Rejected for character, not length. |
| TC-U-034 | Both rules violated: 3 chars with invalid char | None | 1. Enter "a-b". 2. Submit. | Rejected; at least one error shown. Whether both errors are listed is unspecified (Q6). |

## D. Injection / robustness

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-U-040 | SQL injection string | None | 1. Enter "admin' OR '1'='1". 2. Submit. | Rejected (invalid characters). No DB error, no data exposure. |
| TC-U-041 | HTML/script string | None | 1. Enter "<script>alert(1)</script>". 2. Submit. | Rejected; nothing executed or rendered. |
| TC-U-042 | Null / missing value via API | API accessible | 1. Send request with username null, then with the field omitted. | Rejected with a validation error; no 500 error. |
| TC-U-043 | Non-string type via API | API accessible | 1. Send username as number 12345, then as array. | Rejected (or consistently handled); no 500 error. Exact behaviour unspecified. |

## E. Uniqueness

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-U-050 | Duplicate of an existing username (exact match) | User "john_smith" exists | 1. Enter "john_smith". 2. Submit. | Rejected with a "already taken" type error. Existing account unchanged. |
| TC-U-051 | Duplicate differing only in case | User "JohnSmith" exists | 1. Enter "johnsmith". 2. Submit. | Depends on Q2. Record actual behaviour. |
| TC-U-052 | Near-duplicate (one character different) is allowed | User "john_smith" exists | 1. Enter "john_smith1". 2. Submit. | Accepted. |
| TC-U-053 | Duplicate at minimum length | User "abcd" exists | 1. Enter "abcd". 2. Submit. | Rejected (duplicate). |
| TC-U-054 | Duplicate at maximum length | A 20-char user exists | 1. Enter the same 20-char name. 2. Submit. | Rejected (duplicate). |
| TC-U-055 | Concurrent registration of the same username | "race_user" not registered; two clients ready | 1. Submit "race_user" from two clients at the same moment. | Exactly one succeeds; the other is rejected. Only one account exists afterwards. |
| TC-U-056 | Rapid double submit from one client | "dbl_user" not registered | 1. Double-click Submit / resend the request twice. | One account created; second request rejected or ignored. No duplicates in storage. |
| TC-U-057 | Invalid and duplicate at once | User "abc" cannot exist; user "john_smith" exists | 1. Enter "john-smith" (invalid char, similar to an existing one). 2. Submit. | Rejected for format. No account created. |
| TC-U-058 | Username freed after account deletion | User "temp_user" exists and is then deleted | 1. Register "temp_user" again. | Depends on Q7. Record actual behaviour. |
| TC-U-059 | Uniqueness check on the stored value across username change | Users "user_a", "user_b" exist (only if rename is supported, Q8) | 1. Rename "user_a" to "user_b". | Rejected (duplicate). Skip if rename is not in scope. |

---

## Open questions (not assumed in the tests above)

1. **Definition of "letters":** ASCII A-Z/a-z only, or any Unicode letter (accented, Cyrillic, CJK)? Affects TC-U-028, TC-U-029 and how length is counted for multi-byte characters.
2. **Case sensitivity of uniqueness:** Are "JohnSmith" and "johnsmith" the same username? Is the stored value normalised (lowercased) or kept as entered? Affects TC-U-006 and TC-U-051.
3. **Where is validation enforced:** client-side only, server-side, or both? Is there an API in scope as well as a UI? Affects TC-U-042 and TC-U-043.
4. **Further restrictions on structure:** Is a username made only of digits or only of underscores acceptable? May it start with a digit or underscore, or end with an underscore? The requirement allows these, but they are unusual and worth confirming (TC-U-004, 005, 007, 008, 017).
5. **Whitespace handling:** Is leading/trailing whitespace trimmed before validation, or rejected as invalid?
6. **Error messages:** Required wording, and whether multiple violations are reported together or one at a time (TC-U-034). Does the message for a duplicate reveal that the name exists?
7. **Reuse of deleted usernames:** May a username be re-registered after the account is deleted or deactivated? Is a reserved list (e.g. "admin", "root") in scope?
8. **Can usernames be changed after creation?** If yes, the same rules and uniqueness must apply (TC-U-059).
9. **Normalisation:** Is Unicode normalisation applied (relevant only if Q1 allows non-ASCII)?
10. **Performance / scale of the uniqueness check:** Any expected response time or behaviour under a very large user base?
11. **Rate limiting / enumeration:** Is there any limit on repeated availability checks? (Out of scope unless confirmed.)

## Coverage summary
- Positive: TC-U-001 to 009
- Boundary: TC-U-010 to 018 (lengths 0, 1, 3, 4, 20, 21, 100)
- Negative: TC-U-020 to 034, 040 to 043
- Uniqueness: TC-U-050 to 059
- Cases marked "depends on Qn" are intentionally left without a pass/fail verdict until the open question is answered.
