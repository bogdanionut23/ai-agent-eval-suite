# Test Cases: Username Validation

Requirement: A username must be 4 to 20 characters long and may contain only letters, digits and underscores. Usernames must be unique.

## Assumptions (stated, not invented behaviour)
- "Letters" is tested with ASCII A-Z / a-z only. Non-ASCII letters are an open question (Q1).
- Tests are written against a generic "create account / set username" action. Expected rejections say "rejected with a validation error". The exact message text is an open question (Q8).
- Test data below uses the placeholder user `taken_user`, which must already exist.

## Preconditions common to all tests
- The system is running and the registration (or change-username) form or API is reachable.
- Unless stated, no user with the tested username exists.

## Positive

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC14-P01 | Typical valid username (letters only) | `alice` not registered | Submit username `alice` | Accepted; account created with username `alice` |
| TC14-P02 | Letters and digits | `user123` not registered | Submit `user123` | Accepted |
| TC14-P03 | Letters, digits and underscores | `john_doe_99` not registered | Submit `john_doe_99` | Accepted |
| TC14-P04 | Uppercase letters | `ALICE` not registered | Submit `ALICE` | Accepted (see Q2 for case-uniqueness) |
| TC14-P05 | Digits only | `12345` not registered | Submit `12345` | Accepted, since digits are allowed and no rule requires a letter (see Q4) |
| TC14-P06 | Underscore at start and end | `_ab_` not registered | Submit `_ab_` | Accepted (see Q5) |
| TC14-P07 | Underscores only, minimum length | `____` not registered | Submit `____` | Accepted per the literal rule (see Q4, Q5) |

## Boundary (length)

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC14-B01 | Length 3 (one below minimum) | `abc` not registered | Submit `abc` | Rejected with a length validation error |
| TC14-B02 | Length 4 (minimum) | `abcd` not registered | Submit `abcd` | Accepted |
| TC14-B03 | Length 5 (just above minimum) | `abcde` not registered | Submit `abcde` | Accepted |
| TC14-B04 | Length 19 (just below maximum) | 19-char name `a` x 19 not registered | Submit 19 x `a` | Accepted |
| TC14-B05 | Length 20 (maximum) | 20-char name `a` x 20 not registered | Submit 20 x `a` | Accepted |
| TC14-B06 | Length 21 (one above maximum) | 21-char name not registered | Submit 21 x `a` | Rejected with a length validation error |
| TC14-B07 | Length 20 mixing all allowed character classes | Name `Ab1_Ab1_Ab1_Ab1_Ab1_` (20 chars) not registered | Submit it | Accepted |
| TC14-B08 | Very long input (e.g. 1000 chars) | none | Submit 1000 x `a` | Rejected with a length error; no crash, truncation or timeout. Nothing is stored |

## Negative (format)

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC14-N01 | Empty username | none | Submit empty string | Rejected (length error or required-field error) |
| TC14-N02 | Missing username field / null (API) | API access | Send the request without the field, then with `null` | Rejected; no account created; no server error (5xx) |
| TC14-N03 | Whitespace only | none | Submit 5 spaces | Rejected |
| TC14-N04 | Space inside | none | Submit `john doe` | Rejected (invalid character) |
| TC14-N05 | Leading space | none | Submit ` alice` | Rejected, or see Q6 if trimming is applied |
| TC14-N06 | Trailing space | none | Submit `alice ` | Rejected, or see Q6 if trimming is applied |
| TC14-N07 | Hyphen | none | Submit `john-doe` | Rejected |
| TC14-N08 | Dot | none | Submit `john.doe` | Rejected |
| TC14-N09 | At sign | none | Submit `john@doe` | Rejected |
| TC14-N10 | Common special characters, one per run | none | Submit each of `ab!c`, `ab#c`, `ab$c`, `ab%c`, `ab&c`, `ab*c`, `ab/c`, `ab\c`, `ab'c`, `ab"c` | Each is rejected (invalid character) |
| TC14-N11 | Non-ASCII letter | none | Submit `josé1` | Behaviour per Q1. Until answered, record the actual result |
| TC14-N12 | Non-Latin script | none | Submit `用户名称abc` | Behaviour per Q1. Until answered, record the actual result |
| TC14-N13 | Emoji | none | Submit `user😀name` | Rejected |
| TC14-N14 | Valid character count but invalid character | none | Submit `abc-` (4 chars) | Rejected for the character, not accepted on length alone |
| TC14-N15 | Invalid character and wrong length together | none | Submit `a!` | Rejected; at least one error shown (see Q8 for whether both are shown) |
| TC14-N16 | Tab / newline / control characters | none | Submit `ab\tcd` and `ab\ncd` | Rejected |
| TC14-N17 | Injection strings | none | Submit `admin' OR '1'='1`, `<script>x</script>` | Rejected as invalid characters; no SQL error, no script execution |
| TC14-N18 | Unicode look-alike of a digit or underscore | none | Submit `user＿name` (full-width underscore) and `user١٢٣` (Arabic-Indic digits) | Behaviour per Q1. Until answered, record the actual result |

## Uniqueness

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC14-U01 | Duplicate of an existing username | User `taken_user` exists | Submit `taken_user` | Rejected with a "username already taken" type error; the existing account is unchanged |
| TC14-U02 | Different username accepted | User `taken_user` exists | Submit `taken_user2` | Accepted |
| TC14-U03 | Case variant of existing username | User `taken_user` exists | Submit `Taken_User` | Behaviour per Q2. Until answered, record the actual result |
| TC14-U04 | Duplicate attempted after the first is created | `newname1` not registered | Register `newname1`, then register `newname1` again | First accepted, second rejected |
| TC14-U05 | Concurrent registration of the same name | `racer_01` not registered | Send two simultaneous requests for `racer_01` | Exactly one succeeds; the other is rejected; only one account exists |
| TC14-U06 | Duplicate with surrounding whitespace | User `taken_user` exists | Submit ` taken_user` | Rejected (invalid character or duplicate), and no second account is created |
| TC14-U07 | Invalid name is not reserved or stored | `bad-name` not registered | Submit `bad-name` (rejected), then submit `bad_name` | First rejected, second accepted; no partial record from the rejected attempt |
| TC14-U08 | Name of a deleted or deactivated account | User `gone_user` was deleted/deactivated | Submit `gone_user` | Behaviour per Q3. Until answered, record the actual result |
| TC14-U09 | Uniqueness on username change (if the feature exists) | Users `taken_user` and `other_user` exist | Log in as `other_user`, change the username to `taken_user` | Rejected; `other_user` keeps its name. Applies only if Q7 confirms changes are supported |
| TC14-U10 | Re-submitting one's own current username (if changes are supported) | User `taken_user` logged in | Change the username to `taken_user` | Behaviour per Q7. Until answered, record the actual result |

## Open questions (requirement is ambiguous or incomplete)

1. **Definition of "letters":** ASCII A-Z/a-z only, or any Unicode letters (accented, Cyrillic, CJK)? The same applies to "digits" (Arabic-Indic and other Unicode digits).
2. **Case sensitivity of uniqueness:** Are `Alice` and `alice` the same username? Is the stored username case-preserving?
3. **Deleted or deactivated accounts:** Can their usernames be reused?
4. **Content restrictions:** Must a username contain at least one letter? Are digits-only (`12345`) or underscore-only (`____`) names allowed?
5. **Position of underscores:** Are leading or trailing underscores, or consecutive underscores, allowed?
6. **Whitespace handling:** Is leading or trailing whitespace trimmed before validation, or rejected? At which point is length measured, before or after trimming and normalisation?
7. **Scope of the rule:** Does it apply to registration only, or also to changing a username, to admin-created accounts, to imports and to API clients? Can a username be changed at all?
8. **Error behaviour:** What are the exact error messages? When several rules fail at once (wrong length and bad character), are all errors shown or only the first? Should the form validate live or on submit?
9. **Reserved names:** Are any names blocked (`admin`, `root`, `system`, offensive words)? The requirement is silent.
10. **Length counting:** For inputs outside the allowed set, such as characters outside the Basic Multilingual Plane, is length counted in characters, code points or bytes? Is Unicode normalisation (NFC/NFKC) applied?
11. **Uniqueness scope:** Is it unique globally or per tenant/organisation?
12. **Existing data:** Do usernames that already exist and violate the new rules need migration or grandfathering?
13. **Enforcement layers:** Is validation expected both client-side and server-side, and must the database enforce a unique constraint?

## Notes
- Where a test says "behaviour per Qn", the expected result is intentionally not invented. The test should be finalised once the question is answered.
- Highest risk areas: length boundaries (B01, B02, B05, B06), case-sensitive uniqueness (U03) and concurrent duplicates (U05).
