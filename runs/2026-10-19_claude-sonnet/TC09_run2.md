# Test Cases: Product Search Minimum Length and Relevance Ordering

Requirement: "The product search runs only when the user has typed at least 3 characters. Results are shown in order of relevance."

## Assumptions (stated, not invented behaviour)
- "Runs" means the search request is executed and results are displayed. Below the threshold, no search executes.
- Relevance ranking is defined elsewhere. Tests below check only that ordering is consistent with the defined relevance score, using a prepared data set where the expected order is known.
- Anything the requirement does not define is listed under Open Questions, and the related tests are marked BLOCKED or "verify against answer".

## Test data (prepared catalogue)
- P1 "Blue Cotton Shirt" (term "shirt" in the name, high relevance)
- P2 "Shirt Washing Detergent" (term in the name, different category)
- P3 "Garment Care Kit" (description mentions "shirt" once, low relevance)
- P4 "Running Shoes" (no match for "shirt")
- P5 "T-Shirt Classic" (name match, expected rank to be confirmed with the relevance owner)
- A product named "Éclair Mug" and a product named "C++ Handbook"
- A product with a 2-character name "TV" and a product with a numeric name "007"

## Positive

**TC09-P01**
- Title: Search runs at exactly 3 characters
- Preconditions: Search box is visible and empty; catalogue contains P1 to P5.
- Steps: 1. Type "shi" in the search box.
- Expected: Search executes (one request is made), and results containing "shi" are displayed.

**TC09-P02**
- Title: Search runs for more than 3 characters
- Preconditions: As above.
- Steps: 1. Type "shirt".
- Expected: Search executes and matching products (P1, P2, P3, P5) are displayed; P4 is not displayed.

**TC09-P03**
- Title: Results are ordered by relevance (best match first)
- Preconditions: Catalogue prepared so relevance order for "shirt" is known (for example P1 before P3).
- Steps: 1. Type "shirt". 2. Read the result list top to bottom.
- Expected: Products appear in descending relevance. A product with the term in its name ranks above a product with the term only in its description.

**TC09-P04**
- Title: Order is stable and repeatable
- Preconditions: As above.
- Steps: 1. Search "shirt", note the order. 2. Clear and search "shirt" again. 3. Reload the page and search again.
- Expected: The same order is shown each time (no data changes in between).

**TC09-P05**
- Title: Search triggers when the third character is typed after two
- Preconditions: Search box is empty.
- Steps: 1. Type "sh" (no search). 2. Type "i".
- Expected: No search at 2 characters; search executes upon the third character.

**TC09-P06**
- Title: Query refined by more characters re-ranks results
- Preconditions: As above.
- Steps: 1. Type "shi". 2. Continue typing to "shirt cotton".
- Expected: Result list updates and is again ordered by relevance for the new query; the most relevant item for "shirt cotton" (P1) is first.

## Negative

**TC09-N01**
- Title: No search with 0 characters
- Preconditions: Search box empty.
- Steps: 1. Focus the box and press Enter / click the search button without typing.
- Expected: No search request is made; no results shown. (Message or hint: see Open Question 3.)

**TC09-N02**
- Title: No search with 1 character
- Steps: 1. Type "s".
- Expected: No search request is made; no results shown.

**TC09-N03**
- Title: No search with 2 characters
- Steps: 1. Type "sh".
- Expected: No search request is made; no results shown.

**TC09-N04**
- Title: Submit by Enter key with 2 characters
- Steps: 1. Type "sh". 2. Press Enter.
- Expected: Search does not run.

**TC09-N05**
- Title: Submit by search button with 2 characters
- Steps: 1. Type "sh". 2. Click the search button (if present).
- Expected: Search does not run.

**TC09-N06**
- Title: Deleting characters below the threshold stops searching
- Steps: 1. Type "shirt" (results shown). 2. Backspace to "sh".
- Expected: No new search is executed for "sh". What happens to the previously shown results (cleared or kept) is undefined: see Open Question 4.

**TC09-N07**
- Title: Paste of a 2-character string
- Steps: 1. Paste "ab" into the box.
- Expected: No search runs.

**TC09-N08**
- Title: Query with no matching products (3+ characters)
- Steps: 1. Type "zzzzzz".
- Expected: Search runs and returns an empty result; the application handles it without error. (Empty-state wording: see Open Question 9.)

**TC09-N09**
- Title: Search not triggered through direct URL or API with a short query
- Steps: 1. Request the search URL/endpoint with q="ab" directly.
- Expected: Behaviour should be consistent with the rule (no results or a validation response). Whether the rule is enforced server-side is undefined: see Open Question 6.

## Boundary

**TC09-B01**
- Title: 2 characters (just below limit)
- Steps: 1. Type "tv" (a real product "TV" exists).
- Expected: No search runs, so "TV" is not found. Confirms the threshold applies even when a matching product exists.

**TC09-B02**
- Title: 3 characters (at limit)
- Steps: 1. Type "007".
- Expected: Search runs and returns the product "007".

**TC09-B03**
- Title: 4 characters (just above limit)
- Steps: 1. Type "shir".
- Expected: Search runs; results ordered by relevance.

**TC09-B04**
- Title: Leading/trailing spaces counted?
- Steps: 1. Type "  a" (2 spaces + 1 letter). 2. Type "ab " (2 letters + space).
- Expected: Depends on Open Question 1. Mark as BLOCKED until answered; record the actual behaviour.

**TC09-B05**
- Title: 3 spaces only
- Steps: 1. Type "   " (three spaces).
- Expected: Depends on Open Question 1 (likely no search). BLOCKED until answered.

**TC09-B06**
- Title: Multi-byte and accented characters
- Steps: 1. Type "Écl" (3 characters). 2. Type a 3-character string with an emoji or a character outside the BMP.
- Expected: Each user-visible character counts as one; "Écl" runs the search and finds "Éclair Mug". Emoji handling: see Open Question 2.

**TC09-B07**
- Title: Special characters count towards length
- Steps: 1. Type "C++".
- Expected: Search runs (3 characters) and finds "C++ Handbook" without an error or injection issue.

**TC09-B08**
- Title: Very long query
- Steps: 1. Paste a query of 500 and then 5,000 characters.
- Expected: No crash; search runs or is rejected in a controlled manner. Maximum length is undefined: see Open Question 7.

**TC09-B09**
- Title: Result set of exactly 1 and of a large number of products
- Steps: 1. Search a term matching one product. 2. Search a term matching many products (more than one page).
- Expected: Single result shown correctly; large set ordered by relevance across all pages (page 2 items are less relevant than page 1 items). Pagination rules: see Open Question 8.

**TC09-B10**
- Title: Typing speed around the threshold
- Steps: 1. Type "shi" very fast, then delete and retype repeatedly.
- Expected: No search for 1 to 2 characters; no duplicate or out-of-order responses; the displayed results always match the current query.

## Ties and ordering edge cases

**TC09-R01**
- Title: Equal relevance
- Steps: 1. Search a term that yields products with identical relevance.
- Expected: All are shown; their relative order is deterministic. Tie-break rule: see Open Question 5.

**TC09-R02**
- Title: Case insensitivity affects relevance equally
- Steps: 1. Search "SHIRT", "shirt" and "Shirt".
- Expected: Same results in the same order (if the search is case-insensitive: see Open Question 2).

## Open Questions (not assumed in the tests above)
1. Are leading/trailing spaces, or whitespace-only input, counted towards the 3 characters? Is the query trimmed before counting?
2. How are characters counted: user-visible characters, UTF-16 units, or bytes (emoji, combining accents)? Is search case-sensitive? Does the threshold apply to every language, including those where 1 or 2 characters are meaningful (for example CJK)?
3. What should the user see when fewer than 3 characters are typed: nothing, a hint such as "Type at least 3 characters", or a disabled search button?
4. When the user deletes characters back below 3, are previous results cleared or kept?
5. What defines "relevance" (name vs description vs category, exact vs partial match, popularity, stock)? What is the tie-break for equal relevance? Does the order depend on the user (personalisation) or change over time?
6. Is the minimum enforced only in the UI or also in the API/server? What should the API return for short queries (error code, empty list)?
7. Is there a maximum query length?
8. Pagination, page size, and a maximum number of results: does relevance order hold across pages?
9. What is shown when there are no results?
10. Is the search triggered as the user types (with debounce), or only on submit? What is the debounce time? This affects how the threshold is tested.
11. Do filters or sorting options exist that could override relevance order? If the user chooses another sort, is the requirement still valid?
12. Does the requirement apply to all entry points (header search, mobile app, voice input, search suggestions/autocomplete)?
