# Test Cases: Username Validation

Requirement: A username must be 4 to 20 characters long and may contain only letters, digits and underscores. Usernames must be unique.

## Assumptions (stated, not invented behaviour)

- "Letters" is tested here with ASCII letters (A-Z, a-z). Non-ASCII letters are in the open questions, not assumed either way.
- Length is counted in characters, as the requirement says. How characters are counted is in the open questions.
- Test cases that depend on an open question are marked **[Blocked by Q#]**. Their expected result is the one most likely, and it must be confirmed before the test is run.

## Positive Test Cases

### TC-P01 Valid username, lowercase letters only
- Preconditions: Username "tester" does not exist.
- Steps: 1. Submit registration with username "tester".
- Expected: Accepted. Account is created with username "tester".

### TC-P02 Valid username with mixed case
- Preconditions: Username "JohnDoe" does not exist.
- Steps: 1. Submit "JohnDoe".
- Expected: Accepted.

### TC-P03 Valid username with digits and letters
- Preconditions: "user123" does not exist.
- Steps: 1. Submit "user123".
- Expected: Accepted.

### TC-P04 Valid username with underscores
- Preconditions: "john_doe_1" does not exist.
- Steps: 1. Submit "john_doe_1".
- Expected: Accepted.

### TC-P05 Username made only of digits
- Preconditions: "123456" does not exist.
- Steps: 1. Submit "123456".
- Expected: Accepted (digits are allowed and no rule requires a letter). **[Blocked by Q5]**

### TC-P06 Username made only of underscores
- Preconditions: "____" does not exist.
- Steps: 1. Submit "____" (4 underscores).
- Expected: Accepted per the literal rule. **[Blocked by Q5]**

### TC-P07 Username starting with a digit or underscore
- Preconditions: "1user" and "_user" do not exist.
- Steps: 1. Submit "1user". 2. Submit "_user".
- Expected: Both accepted per the literal rule. **[Blocked by Q5]**

### TC-P08 Two different usernames can both be registered
- Preconditions: "alice_01" and "bob_02" do not exist.
- Steps: 1. Register "alice_01". 2. Register "bob_02".
- Expected: Both accepted.

## Boundary Test Cases (Length)

### TC-B01 Minimum length, 4 characters
- Preconditions: "abcd" does not exist.
- Steps: 1. Submit "abcd".
- Expected: Accepted.

### TC-B02 One below minimum, 3 characters
- Preconditions: None.
- Steps: 1. Submit "abc".
- Expected: Rejected with a length error. No account is created.

### TC-B03 One above minimum, 5 characters
- Preconditions: "abcde" does not exist.
- Steps: 1. Submit "abcde".
- Expected: Accepted.

### TC-B04 One below maximum, 19 characters
- Preconditions: "aaaaaaaaaaaaaaaaaaa" (19 a's) does not exist.
- Steps: 1. Submit a 19-character valid username.
- Expected: Accepted.

### TC-B05 Maximum length, 20 characters
- Preconditions: "aaaaaaaaaaaaaaaaaaaa" (20 a's) does not exist.
- Steps: 1. Submit a 20-character valid username.
- Expected: Accepted.

### TC-B06 One above maximum, 21 characters
- Preconditions: None.
- Steps: 1. Submit a 21-character username of valid characters.
- Expected: Rejected with a length error. No account is created.

### TC-B07 Boundary lengths with mixed allowed characters
- Preconditions: None of the values exist.
- Steps: 1. Submit "a_1b" (4). 2. Submit "a_1b_c2d_e3f_g4h_i5j" (20). 3. Submit "a_1b_c2d_e3f_g4h_i5j_" (21).
- Expected: First two accepted; the third rejected for length.

### TC-B08 Very long input
- Preconditions: None.
- Steps: 1. Submit a username of 1,000 valid characters (and, separately, 100,000).
- Expected: Rejected with a length error. No crash, timeout, truncation to 20 characters, or server error.

## Negative Test Cases (Content)

### TC-N01 Empty username
- Steps: 1. Submit "".
- Expected: Rejected (below minimum length / required). No account is created.

### TC-N02 Whitespace-only username
- Steps: 1. Submit "    " (4 spaces).
- Expected: Rejected (spaces are not allowed characters).

### TC-N03 Space inside the username
- Steps: 1. Submit "john doe".
- Expected: Rejected for an invalid character.

### TC-N04 Leading and trailing space
- Steps: 1. Submit " johndoe" and "johndoe ".
- Expected: Rejected for an invalid character, or trimmed first. **[Blocked by Q4]**

### TC-N05 Hyphen
- Steps: 1. Submit "john-doe".
- Expected: Rejected for an invalid character.

### TC-N06 Dot
- Steps: 1. Submit "john.doe".
- Expected: Rejected.

### TC-N07 Common special characters
- Steps: Submit each separately: "user@name", "user!", "user#1", "user$", "us%er", "user&1", "user*1", "user+1", "user=1", "user/1", "user\1", "user'1", "user\"1", "(user)", "user;1", "user:1", "user,1", "user?1".
- Expected: Each rejected for an invalid character.

### TC-N08 Non-ASCII letters
- Steps: Submit "josé", "müller", "名前名前", "иван_1".
- Expected: Behaviour depends on the definition of "letters". **[Blocked by Q1]**

### TC-N09 Emoji
- Steps: Submit "user😀" and "😀😀😀😀".
- Expected: Rejected for an invalid character. **[Blocked by Q2]** for length counting of such input.

### TC-N10 Unicode look-alikes and full-width characters
- Steps: Submit "ａｂｃｄ" (full-width) and a username using Cyrillic "а" mixed with Latin letters.
- Expected: Rejected, since they are not the allowed ASCII set. **[Blocked by Q1]**

### TC-N11 Control and invisible characters
- Steps: Submit usernames containing a tab, a newline, a null byte (%00), and a zero-width space (U+200B), for example "user\tname".
- Expected: Rejected for an invalid character. No truncation at the null byte.

### TC-N12 Valid length but one invalid character
- Steps: Submit "valid_user!" (11 characters).
- Expected: Rejected for an invalid character, even though the length is valid.

### TC-N13 Invalid length and invalid characters together
- Steps: Submit "a!" (2 characters, one invalid).
- Expected: Rejected. At least one error is shown. Which error(s) are shown is not specified. **[Blocked by Q8]**

### TC-N14 Injection strings
- Steps: Submit "' OR 1=1 --", "<script>alert(1)</script>", "admin'--", "../../etc".
- Expected: Rejected for invalid characters. No script runs, no SQL error, no server error.

### TC-N15 Missing field and wrong type
- Steps: 1. Send a request with no username field. 2. Send null. 3. Send a number (12345) or an array in place of a string.
- Expected: Rejected with a validation error. No server error (HTTP 5xx). **[Blocked by Q9]** for the exact error format.

## Uniqueness Test Cases

### TC-U01 Duplicate username, exact match
- Preconditions: Account "taken_user" exists.
- Steps: 1. Submit registration with "taken_user".
- Expected: Rejected with a "username already taken" error. The existing account is unchanged and no second account is created.

### TC-U02 Duplicate username, different case
- Preconditions: Account "JohnDoe" exists.
- Steps: 1. Submit "johndoe". 2. Submit "JOHNDOE".
- Expected: Depends on whether uniqueness is case-sensitive. **[Blocked by Q3]**

### TC-U03 Duplicate differing only by surrounding whitespace
- Preconditions: Account "johndoe" exists.
- Steps: 1. Submit "johndoe " (trailing space).
- Expected: Rejected (invalid character or duplicate after trimming). **[Blocked by Q4]**

### TC-U04 Concurrent registration of the same username
- Preconditions: "race_user" does not exist.
- Steps: 1. Send two registration requests for "race_user" at the same moment, from two clients.
- Expected: Exactly one succeeds. The other gets a "username already taken" error. Only one account exists.

### TC-U05 Rapid repeated submission (double click)
- Preconditions: "dbl_user" does not exist.
- Steps: 1. Submit the form twice within milliseconds with the same username.
- Expected: One account is created. No duplicate record and no confusing error for the user who submitted legitimately.

### TC-U06 Uniqueness at the data layer
- Preconditions: Account "dbunique" exists.
- Steps: 1. Bypass the UI/API validation (for example a direct database insert or a second service) and try to insert "dbunique".
- Expected: The database rejects it through a unique constraint or index.

### TC-U07 Reuse after account deletion
- Preconditions: Account "gone_user" was created, then deleted.
- Steps: 1. Register "gone_user" again.
- Expected: Depends on policy. **[Blocked by Q6]**

### TC-U08 Uniqueness check for a username that is also invalid
- Preconditions: Account "abc" cannot exist (invalid length).
- Steps: 1. Submit "abc".
- Expected: Rejected for length. It is not reported as a duplicate. The message does not reveal anything about other accounts.

### TC-U09 Reserved or system names
- Preconditions: None.
- Steps: 1. Submit "admin", "root", "support", "system".
- Expected: Not covered by the requirement. **[Blocked by Q7]**

### TC-U10 Changing a username to an existing one
- Preconditions: Accounts "user_a" and "user_b" exist, and username change is a supported feature.
- Steps: 1. Log in as user_a. 2. Change the username to "user_b".
- Expected: Rejected with "already taken". **[Blocked by Q10]** (confirm that renaming is in scope)

### TC-U11 Changing a username to its own current value
- Preconditions: Account "user_a" exists and renaming is supported.
- Steps: 1. Change the username to "user_a".
- Expected: Not reported as a duplicate of itself. **[Blocked by Q10]**

## Cross-Cutting Checks

### TC-X01 Same rules in UI and API
- Steps: Run TC-B02, TC-B06 and TC-N05 through the UI form and again through the API directly.
- Expected: Same accept/reject outcome in both. Client-side validation is not the only line of defence.

### TC-X02 Error message quality
- Steps: Trigger each failure type (too short, too long, invalid character, duplicate).
- Expected: Each shows a distinct, clear message that states the rule broken. Messages do not expose internals.

### TC-X03 Failed attempts leave no trace
- Steps: After any rejection above, check the user store.
- Expected: No partial or orphan account.

### TC-X04 Stored value equals submitted value
- Preconditions: "MiXed_Case9" does not exist.
- Steps: 1. Register it. 2. Read the account back.
- Expected: The stored and displayed username is exactly "MiXed_Case9", unless a normalisation rule is defined. **[Blocked by Q3]**

## Open Questions (requirement is ambiguous or incomplete)

1. **Definition of "letters".** Only ASCII A-Z / a-z, or any Unicode letter (é, ü, 名, и)? This changes TC-N08 and TC-N10.
2. **How is length counted?** By characters (code points), UTF-16 units, grapheme clusters, or bytes? Matters for emoji and combining characters, if they are allowed at all.
3. **Is uniqueness case-sensitive?** Are "JohnDoe" and "johndoe" the same username? Is the original case kept for display, or is the name lower-cased when stored?
4. **Leading and trailing whitespace.** Is it rejected, or trimmed before validation and the uniqueness check? Does length count before or after trimming?
5. **Extra restrictions on the first character or composition?** May a name start with a digit or underscore? May it be only digits or only underscores? The text allows it, but this is often unintended.
6. **Reuse of deleted or deactivated usernames.** Can a username be registered again after account deletion or deactivation? Is there a cooling-off period?
7. **Reserved words.** Is there a blocklist ("admin", "root", offensive terms)? The requirement does not mention one.
8. **Error behaviour when several rules fail at once.** Show all errors or only the first? Is the order of checks defined?
9. **Required error messages, codes and API contract.** What are the exact texts, HTTP status codes, and response format? How should null, a missing field, or a non-string be handled?
10. **Scope of the rule.** Does it apply only to registration, or also to username change, admin-created accounts, imports and SSO-provisioned accounts? Are existing usernames that violate the rule (legacy data) grandfathered?
11. **Unicode normalisation.** If non-ASCII input is allowed, is it normalised (NFC/NFKC) before the uniqueness check?
12. **Real-time availability check.** Is a live "username available" check expected? If so, are there rate limits, and does it risk revealing which usernames exist (enumeration)?
13. **Performance and scale.** Are there response-time or volume targets for the uniqueness check?
14. **Localisation.** Are error messages needed in several languages?

## Coverage Summary

| Area | Test cases |
|---|---|
| Positive | TC-P01 to TC-P08 |
| Length boundaries (3/4/5, 19/20/21) | TC-B01 to TC-B08 |
| Invalid content and robustness | TC-N01 to TC-N15 |
| Uniqueness | TC-U01 to TC-U11 |
| Cross-cutting | TC-X01 to TC-X04 |

Total: 46 test cases. Of these, 15 depend on an open question (marked **[Blocked by Q#]**) and should not be run until the question is answered.
