# TC09 - Test cases: product search minimum input length and relevance ordering

**Requirement:** The product search runs only when the user has typed at least 3 characters. Results are shown in order of relevance.

## Test cases

### Positive

**TC09-01 - Search runs at exactly 3 characters**
- Preconditions: User is on the product search page or component. Search is enabled. The catalogue contains at least one product whose name starts with "abc" (or any product matching the test term).
- Steps:
  1. Click into the search input.
  2. Type `abc`.
  3. Wait for the search to complete (or for the debounce, if any, to elapse).
- Expected result: The search executes once the third character is entered. Matching products are displayed.

**TC09-02 - Search runs for a longer query**
- Preconditions: As TC09-01. Catalogue contains products matching `headphones`.
- Steps:
  1. Type `headphones` into the search input.
  2. Wait for results.
- Expected result: The search executes and returns products matching the query.

**TC09-03 - Results are ordered by relevance**
- Preconditions: Catalogue contains products with differing degrees of match to the term, e.g. one whose name equals the term exactly, one whose name starts with it, and one where it appears only in the description. The expected relevance order is defined by the open questions below; for this test, the ordering rule must be confirmed first.
- Steps:
  1. Type the search term.
  2. Read the order of the returned result list.
- Expected result: Results are listed from most to least relevant, consistent with the relevance rule confirmed by the product owner.

**TC09-04 - Relevance order is stable across repeated searches**
- Preconditions: As TC09-03.
- Steps:
  1. Search for the term and record the result order.
  2. Clear the input and search for the same term again.
- Expected result: The same term produces the same result order each time.

**TC09-05 - Search is case-insensitive (if confirmed)**
- Preconditions: Catalogue contains a product named with uppercase letters, e.g. `Headphones`.
- Steps:
  1. Type `HEADPHONES` (or `headphones`) and search.
- Expected result: The product is returned. (Applies only if case-insensitive matching is confirmed; see open questions.)

### Negative

**TC09-06 - No search runs with 0 characters**
- Preconditions: Search input is empty.
- Steps:
  1. Focus the empty search input.
  2. Do not type anything.
- Expected result: No search request is made. No results are shown, or the page shows its default state (to be confirmed).

**TC09-07 - No search runs with 1 character**
- Preconditions: Search input is empty.
- Steps:
  1. Type `a`.
- Expected result: No search request is made. No results are displayed.

**TC09-08 - No search runs with 2 characters**
- Preconditions: Search input is empty.
- Steps:
  1. Type `ab`.
- Expected result: No search request is made. No results are displayed.

**TC09-09 - Search does not run when the input is only spaces**
- Preconditions: Search input is empty.
- Steps:
  1. Type three space characters (`   `).
- Expected result: No search request is made, because the input is not 3 meaningful characters. (Whether spaces count toward the 3 characters is an open question; this test assumes they do not, and must be confirmed.)

**TC09-10 - Search with no matching products**
- Preconditions: Search is enabled. No product name or description contains `zzzzzz`.
- Steps:
  1. Type `zzzzzz` and search.
- Expected result: The search runs. An empty-results message is shown and no product cards appear. The message text is to be confirmed.

**TC09-11 - Search is not triggered by a 2-character query on a previously long query being shortened**
- Preconditions: Search for `headphones` has been run and results are shown.
- Steps:
  1. Delete characters until the input reads `he` (2 characters).
- Expected result: No new search is run for `he`. The previous results are either cleared or kept; the behaviour must match the open question on this case.

**TC09-12 - Special characters and symbols**
- Preconditions: Search is enabled.
- Steps:
  1. Type `!!!` (3 non-alphanumeric characters) and search.
- Expected result: Behaviour is as defined by the open question on symbol-only queries. The page must not crash or show an error page.

### Boundary

**TC09-13 - Boundary: 2 to 3 characters**
- Preconditions: Search input is empty.
- Steps:
  1. Type `ab` and check whether a search request is made.
  2. Type `c` so the input reads `abc`, and check again.
- Expected result: No search at 2 characters. A search runs at exactly 3 characters.

**TC09-14 - Boundary: removing a character drops below 3**
- Preconditions: Input reads `abc` and a search has run.
- Steps:
  1. Press Backspace so the input reads `ab`.
- Expected result: No new search runs for `ab`. The effect on the currently displayed results is an open question.

**TC09-15 - Boundary: very long query**
- Preconditions: Search is enabled.
- Steps:
  1. Paste a string of 500 characters.
- Expected result: The search runs once. The page stays responsive. Any maximum length is an open question.

**TC09-16 - Boundary: multi-byte and non-Latin characters**
- Preconditions: Search is enabled. Catalogue contains a product with a non-Latin name.
- Steps:
  1. Type 2 non-Latin characters (e.g. `ко`), then a third (`ко ш`or a 3-character word).
- Expected result: Length is counted as the number of characters the user sees, not bytes. The search runs at 3 characters. (Counting rule to be confirmed.)

**TC09-17 - Boundary: leading and trailing spaces**
- Preconditions: Search is enabled.
- Steps:
  1. Type ` ab ` (space, a, b, space) and check whether a search runs.
  2. Type ` abc` (leading space, then three characters).
- Expected result: Depends on whether the input is trimmed before counting. Under the trimmed rule, `ab` plus spaces does not search. Confirmation is required (see open questions).

**TC09-18 - Boundary: paste of a 3-character string**
- Preconditions: Search input is empty.
- Steps:
  1. Paste `abc` in one action.
- Expected result: The search runs once for `abc`. No search runs for intermediate lengths.

**TC09-19 - Boundary: rapid typing**
- Preconditions: Search is enabled.
- Steps:
  1. Type `headphones` quickly, character by character.
- Expected result: At most one request is sent per debounce window, or one request per character from the third onward (to be confirmed). The final results match `headphones`.

## Open questions

1. **Minimum length counting:** Are spaces counted toward the 3 characters? Are leading and trailing spaces trimmed before counting?
2. **Character unit:** Is length counted in user-perceived characters, Unicode code points or bytes?
3. **Maximum length:** Is there an upper limit on the query length?
4. **Below-minimum behaviour:** When the input falls below 3 characters after a search, are the previous results cleared, kept, or replaced by a hint message? What is the hint text, if any?
5. **Empty-state message:** What is shown when the input is under 3 characters, and what is shown when no products match?
6. **Trigger:** Does the search run on each keystroke, after a debounce delay (and if so, how long), or only on Enter or a button click? Does the 3-character rule apply to the trigger or to the request?
7. **Relevance definition:** What defines relevance? Name match vs. description match, exact vs. partial, weighting of fields, popularity, stock status, recency, or a backend score? What is the tie-break rule?
8. **Relevance scope:** Is ordering applied by the backend or by the client? Does it apply across pages (pagination)?
9. **Case sensitivity:** Is matching case-insensitive? Are accents and diacritics ignored?
10. **Matching mode:** Is matching prefix, substring, word-based, or fuzzy (typo tolerant)? Does a multi-word query require all words to match?
11. **Symbol-only queries:** Should `!!!` or similar queries run a search, be blocked, or show a message?
12. **Scope of "product search":** Does the requirement cover only the search box, or also filtered/faceted search and category pages? Which fields are searched?
13. **Error handling:** What is shown if the search request fails or times out?
14. **Accessibility:** Is the minimum-length rule communicated to the user (for example through helper text) and are results announced to screen readers?
15. **Offline or slow network:** What happens to a search that is in progress when the user types more characters?
