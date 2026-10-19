# TC09 - Test cases: product search minimum length and relevance ordering

**Requirement:** The product search runs only when the user has typed at least 3 characters. Results are shown in order of relevance.

**Assumptions used for the test steps (not specified in the requirement):**
- "Search runs" means a search request is sent to the backend and results are fetched and rendered.
- "Typed" means keyboard input in the search box. Where paste or other input methods matter, they are covered by separate cases.
- "Relevance" is whatever the product's ranking defines. The expected results below assume a higher relevance score ranks first. The exact scoring model is an open question.

---

## Positive scenarios

### TC09-01 - Search runs once the third character is typed
- **Preconditions:** User is on the product listing or search page. The search box is empty. The product catalogue contains products whose names start with "sho" (for example "Shoe", "Shorts").
- **Steps:**
  1. Click into the search box.
  2. Type "s", then "h", then "o".
  3. Observe the network traffic and the results area.
- **Expected result:** No search request is sent after "s" or "h". After "o" is typed, exactly one search request for "sho" is sent. Matching products are displayed.

### TC09-02 - Search runs for a 3-character query with no matches and shows an empty state
- **Preconditions:** Search box is empty. No product name or attribute contains "zzq".
- **Steps:**
  1. Type "zzq".
  2. Wait for the response.
- **Expected result:** One search request for "zzq" is sent. The page shows the no-results state (message as defined by the product). No error is shown.

### TC09-03 - Results are ordered by relevance
- **Preconditions:** Catalogue contains, for query "run": "Running Shoes" (name starts with query), "Trail Runner" (query appears as a word in the name), "Brunch Plate" (query appears inside a word), and a product where "run" appears only in the description.
- **Steps:**
  1. Type "run".
  2. Read the order of results from top to bottom.
- **Expected result:** Results appear in descending relevance. Products with the strongest match (name prefix/exact word) appear before weaker matches (substring, description-only). The order is the same on every repeated search for the same query.

### TC09-04 - Continued typing beyond 3 characters keeps searching
- **Preconditions:** Search box contains "sho" and results are shown.
- **Steps:**
  1. Type "e" so the query is "shoe".
  2. Type "s" so the query is "shoes".
- **Expected result:** A new search request is sent for each query state that the product's search trigger accepts ("shoe", "shoes"). Results update to match each query. The order follows relevance for each query.

### TC09-05 - Pasting 3 or more characters runs the search once
- **Preconditions:** Search box is empty.
- **Steps:**
  1. Copy the text "shirt" from another source.
  2. Paste it into the search box.
- **Expected result:** The query has 5 characters, so the search runs. Exactly one search request for "shirt" is sent, not one per character. Results are shown.

### TC09-06 - Search results are rendered in the same relevance order as the backend returns
- **Preconditions:** A mocked backend returns results for "shoe" in a known order (A, B, C, D) with descending relevance scores.
- **Steps:**
  1. Type "shoe".
  2. Compare the displayed order with the backend response order.
- **Expected result:** The displayed order is A, B, C, D. The frontend does not re-sort results in a way that changes this order.

---

## Negative scenarios

### TC09-07 - No search runs with 1 character
- **Preconditions:** Search box is empty. Network monitor is open.
- **Steps:**
  1. Type "s".
- **Expected result:** No search request is sent. No results are displayed. Any results area stays in its initial state.

### TC09-08 - No search runs with 2 characters
- **Preconditions:** Search box is empty. Network monitor is open.
- **Steps:**
  1. Type "sh".
- **Expected result:** No search request is sent. No results are displayed.

### TC09-09 - Empty input does not trigger a search
- **Preconditions:** Search box is empty.
- **Steps:**
  1. Focus the search box and press Enter (or the search button, if one exists).
- **Expected result:** No search request is sent. No results are displayed. No error is shown.

### TC09-10 - Whitespace-only input does not trigger a search
- **Preconditions:** Search box is empty.
- **Steps:**
  1. Type three space characters "   ".
- **Expected result:** Behaviour depends on the trimming rule (see open question OQ-02). Under the expected rule "whitespace does not count as a character", no search request is sent and no results are displayed.

### TC09-11 - Search request failure is handled
- **Preconditions:** Backend search endpoint returns HTTP 500 (or the network is disconnected).
- **Steps:**
  1. Type "shoe".
- **Expected result:** A request is attempted once for "shoe". The user sees a clear error or retry message. The page does not crash, and no stale results from an earlier query are shown as if they were current.

### TC09-12 - Special characters and markup in the query are treated as text
- **Preconditions:** Search box is empty.
- **Steps:**
  1. Type `<b>`.
  2. Observe the request and the page.
- **Expected result:** The 3-character query `<b>` triggers one search. The query is sent and displayed as plain text. No HTML or script is injected or executed.

### TC09-13 - Non-character keys do not count toward the 3-character limit
- **Preconditions:** Search box is empty.
- **Steps:**
  1. Press Shift, Ctrl, Arrow Left, and Tab (no characters are typed).
  2. Type "s", then "h".
- **Expected result:** None of the non-character key presses sends a search request. Only the typed characters count. No search runs after "sh".

---

## Boundary scenarios

### TC09-14 - Exactly 2 characters versus exactly 3 characters
- **Preconditions:** Search box is empty. Network monitor is open.
- **Steps:**
  1. Type "ab" and wait 2 seconds.
  2. Type "c".
- **Expected result:** No request after "ab". Exactly one request for "abc" after "c" is typed.

### TC09-15 - Deleting back below the limit
- **Preconditions:** Search box contains "shoe" and results for "shoe" are shown.
- **Steps:**
  1. Press Backspace once so the query is "sho" (3 characters).
  2. Press Backspace once more so the query is "sh" (2 characters).
- **Expected result:** A search for "sho" runs after the first Backspace. After the second Backspace, no new search request is sent for "sh". What remains on screen is defined by OQ-04. It must not show results for "sh" as if they were a new search.

### TC09-16 - Clearing the field
- **Preconditions:** Search box contains "shoe" and results are shown.
- **Steps:**
  1. Select all text and delete it.
- **Expected result:** No search request is sent for an empty query. The results area is cleared or reset as defined by OQ-04.

### TC09-17 - Multi-byte characters count as one character each
- **Preconditions:** Search box is empty.
- **Steps:**
  1. Type "çàé" (3 accented characters, each a single code point).
- **Expected result:** The query has 3 characters, so one search request is sent for "çàé". Results are shown according to the matching rules.

### TC09-18 - Emoji and combined characters
- **Preconditions:** Search box is empty.
- **Steps:**
  1. Type "😀😀" (2 emoji).
  2. Type a third emoji "😀".
- **Expected result:** No search after two emoji. One search after the third. The count rule (code points vs. grapheme clusters) is defined by OQ-03. Either way, a family-style emoji sequence made of several code points must count according to that rule without crashing.

### TC09-19 - Very long query
- **Preconditions:** Search box is empty.
- **Steps:**
  1. Paste a 500-character string.
- **Expected result:** A single search runs (or the input is cut to the maximum length if one exists, see OQ-07). The page does not freeze and the results area renders correctly.

### TC09-20 - Rapid typing does not show stale results
- **Preconditions:** Backend responds slowly (for example 1-2 seconds per request, with a random delay).
- **Steps:**
  1. Type "sho", then quickly "shoe", then "shoes" within 1 second.
  2. Wait for all responses.
- **Expected result:** Each query sends at most one request, subject to debounce (OQ-05). The results shown belong to the last query "shoes". A slow response for "sho" that arrives late does not overwrite the results for "shoes".

### TC09-21 - Case-insensitive matching and relevance
- **Preconditions:** Catalogue contains "Shoe" and "shoe".
- **Steps:**
  1. Type "SHO" (uppercase).
  2. Type "sho" (lowercase) in a separate session.
- **Expected result:** Both queries trigger a search at exactly 3 characters. Both return the same products in the same relevance order. Case-only differences do not change the order (see OQ-06 for the tie-break rule).

### TC09-22 - Ties in relevance are ordered deterministically
- **Preconditions:** Two products have identical relevance scores for "sho".
- **Steps:**
  1. Type "sho". Note the order.
  2. Reload the page and type "sho" again.
- **Expected result:** The order of the tied products is the same on both runs. The tie-break rule is defined by OQ-06.

---

## Open questions

These points are not defined in the requirement. The test cases above mark where the expected result depends on the answer. No behaviour has been invented for them.

- **OQ-01 - Trigger timing:** Does the search run on every keystroke that reaches 3 characters, or only on Enter, a button click, or after a debounce delay? If there is a debounce, what is its length?
- **OQ-02 - Whitespace:** Are leading and trailing spaces trimmed before the length check? Does "a b" count as 3 characters? Is "   " (three spaces) a search with zero effective characters?
- **OQ-03 - Character counting:** Does "character" mean UTF-16 code unit, Unicode code point, or grapheme cluster? This affects emoji and combined characters.
- **OQ-04 - Behaviour below the limit:** When the query drops from 3 characters to 2 or fewer, do the previous results stay visible, get cleared, or show a hint such as "type at least 3 characters"? Is a hint shown at all?
- **OQ-05 - Debounce and cancellation:** Is there a debounce? Are in-flight requests cancelled when the query changes, or are their responses ignored?
- **OQ-06 - Relevance definition:** What is the relevance model (for example field weights, exact vs. prefix vs. substring matches, popularity, stock, price)? What is the tie-break rule when scores are equal?
- **OQ-07 - Maximum length:** Is there a maximum query length? What happens when it is exceeded?
- **OQ-08 - Case and accents:** Is matching case-insensitive? Is it accent-insensitive (for example "cafe" matching "café")? Does this change the 3-character count?
- **OQ-09 - Matching scope:** Does the 3-character rule apply to all search entry points (main search, category search, mobile search, autocomplete suggestions)? Do autocomplete suggestions have their own minimum?
- **OQ-10 - Result paging and sorting controls:** Is "relevance" the default only, or can the user change the sort order? If the user changes sort, does the 3-character rule still apply to re-sorting?
- **OQ-11 - No-results and error messages:** What text is shown for no results and for a failed request? Is there a retry action?
- **OQ-12 - Input methods:** How does the 3-character rule apply to IME composition (for example Chinese or Japanese input) and to voice input? Does the count happen on committed text only?
- **OQ-13 - Performance targets:** What is the maximum acceptable time from the third character to the first rendered result? Is there a required behaviour for slow responses?
