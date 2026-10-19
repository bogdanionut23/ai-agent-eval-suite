# TC09 – Test cases: product search minimum length and relevance ordering

Requirement: "The product search runs only when the user has typed at least 3 characters. Results are shown in order of relevance."

Assumptions are kept minimal. Where the requirement is silent, the behaviour is listed under Open Questions and the related test cases mark the expected result as "to be confirmed" (TBC) rather than inventing it.

## Test cases

### A. Minimum length (positive)

**TC09-01 – Search runs at exactly 3 characters (boundary)**
- Preconditions: User is on the product search page; catalogue contains products matching "lap" (e.g. "Laptop").
- Steps: 1. Click the search box. 2. Type "lap".
- Expected: A search request is executed once the 3rd character is entered; results matching "lap" are displayed.

**TC09-02 – Search runs with more than 3 characters**
- Preconditions: As TC09-01.
- Steps: 1. Type "laptop".
- Expected: Search runs and shows results matching "laptop".

**TC09-03 – Search triggers when the 3rd character is typed after 2 characters**
- Preconditions: As TC09-01; search box empty.
- Steps: 1. Type "la" and observe. 2. Type "p".
- Expected: After step 1 no search is executed (no request, no results). After step 2 the search is executed.

**TC09-04 – Search by submitting (Enter / search button) with 3 or more characters**
- Preconditions: As TC09-01.
- Steps: 1. Type "lap". 2. Press Enter (or click the search button).
- Expected: Search is executed and results are shown; no duplicate or error behaviour.

### B. Minimum length (negative and boundary)

**TC09-05 – No search with 0 characters**
- Preconditions: Search box empty.
- Steps: 1. Focus the search box. 2. Press Enter / click the search button.
- Expected: No search is executed. Resulting UI message/state is TBC (see OQ-3).

**TC09-06 – No search with 1 character**
- Steps: 1. Type "l". 2. Press Enter / click search.
- Expected: No search request is sent; no results are shown. UI feedback TBC.

**TC09-07 – No search with 2 characters (boundary, just below limit)**
- Steps: 1. Type "la". 2. Wait longer than any debounce delay. 3. Press Enter / click search.
- Expected: No search request is sent at any point.

**TC09-08 – Deleting back below 3 characters**
- Preconditions: User has typed "lap" and results are displayed.
- Steps: 1. Press Backspace once so the input is "la".
- Expected: No new search is executed for "la". Whether the previous results stay, clear, or are replaced by a hint is TBC (OQ-4).

**TC09-09 – Whitespace-only input of 3 characters**
- Steps: 1. Type three spaces "   ".
- Expected: TBC (OQ-1). Test verifies and records actual behaviour; a search for blank terms returning the whole catalogue would be a defect candidate if whitespace is not meant to count.

**TC09-10 – Leading/trailing spaces: "a  " and " ab"**
- Steps: 1. Type "a" followed by two spaces. 2. Clear, type a space followed by "ab".
- Expected: TBC (OQ-1). Verify whether the count is based on raw or trimmed input and report.

**TC09-11 – Three characters that are special symbols**
- Steps: 1. Type "%%%", then "<<<", then "'--".
- Expected: Counts as 3 characters (per OQ-2 TBC). The search runs without error, no SQL/HTML injection effect, no crash; results are empty or valid.

**TC09-12 – Multi-byte and accented characters**
- Steps: 1. Type "éclair" prefix "écl" (3 characters). 2. Type a 2-emoji string and a 3-emoji string. 3. Type 3 CJK characters.
- Expected: "écl" and 3 CJK characters count as 3 characters and trigger search. Emoji/combined-character counting is TBC (OQ-2); verify consistency between UI counting and server validation.

**TC09-13 – Paste of text with 3 or more characters**
- Steps: 1. Paste "laptop" into the empty search box.
- Expected: Search runs once without needing a further keystroke.

**TC09-14 – Paste of text with 2 characters**
- Steps: 1. Paste "la".
- Expected: No search is executed.

**TC09-15 – Server-side enforcement (bypass the UI)**
- Preconditions: Access to the search API/endpoint.
- Steps: 1. Send a search request directly with query "ab". 2. Send with query "abc". 3. Send with an empty query.
- Expected: "abc" returns results. "ab" and empty are rejected or return no results per TBC (OQ-5), but are not processed as a normal search.

**TC09-16 – Very long input (upper boundary)**
- Steps: 1. Type/paste a string of 255, then 1,000+ characters.
- Expected: No crash or timeout; behaviour consistent with the system's max length (OQ-6 TBC).

### C. Relevance ordering

**TC09-17 – Exact match ranks above partial match**
- Preconditions: Catalogue has "Laptop" and "Laptop sleeve" and "Gaming laptop bag" (data agreed with the relevance definition, see OQ-7).
- Steps: 1. Search "laptop".
- Expected: Results are ordered by descending relevance as defined by the relevance rule; the most relevant item is first. Exact specifics TBC.

**TC09-18 – Title match ranks above description-only match**
- Preconditions: Product A has the term in its name; Product B has the term only in its description.
- Steps: 1. Search for the term.
- Expected: A appears before B (if name is intended to outrank description; TBC OQ-7).

**TC09-19 – Ordering is stable and deterministic**
- Steps: 1. Run the same search 5 times.
- Expected: Same order every time (assuming data unchanged).

**TC09-20 – Ordering is consistent across pages**
- Preconditions: Search returns more results than one page.
- Steps: 1. Search a broad term. 2. Compare the last item on page 1 with the first on page 2 and the order across pages.
- Expected: Relevance order continues across pages with no duplicates or omissions.

**TC09-21 – Ties in relevance**
- Preconditions: Two products with identical relevance score.
- Steps: 1. Search a term that matches both equally.
- Expected: Order is stable; tie-break rule is TBC (OQ-8).

**TC09-22 – Ordering updates when query changes**
- Steps: 1. Search "lap". 2. Extend to "lapt", then "laptop".
- Expected: Each result set is re-ranked for the current query; results correspond to the latest input (no stale results from an earlier, slower request displayed after a later one).

**TC09-23 – Relevance ordering with case and accent variations**
- Steps: 1. Search "LAP", "Lap", "lap".
- Expected: Same result set and order for all case variants (case-insensitivity itself is TBC, OQ-9).

**TC09-24 – No results**
- Steps: 1. Search "zzzzzz".
- Expected: Search runs; an empty-results state is shown without errors.

**TC09-25 – Single result**
- Steps: 1. Search a term matching exactly one product.
- Expected: One result displayed correctly.

**TC09-26 – User-selected sort overrides relevance (if available)**
- Preconditions: UI offers other sorts (price, newest).
- Steps: 1. Search a term. 2. Choose "Price low to high". 3. Re-run the search.
- Expected: TBC (OQ-10): sort choice applied, default returns to relevance on new search or persists.

### D. Behaviour/non-functional

**TC09-27 – Rapid typing (debounce and request count)**
- Steps: 1. Type "laptop" quickly (under 200 ms per key).
- Expected: No request for 1- or 2-character prefixes; the number of requests for 3+ characters follows the debounce design (TBC OQ-11); final results match "laptop".

**TC09-28 – Accessibility of the minimum-length hint**
- Steps: 1. Use keyboard and a screen reader; type 1-2 characters.
- Expected: Any hint/message about the minimum length is announced and is not colour-only. (Existence of a hint is TBC OQ-3.)

**TC09-29 – Localisation**
- Steps: 1. Switch language and repeat TC09-01 and TC09-06.
- Expected: Same 3-character rule applies in all locales; any message is translated.

**TC09-30 – Response time for ranked results**
- Steps: 1. Search a common term on a full catalogue.
- Expected: Results within the agreed performance target (TBC OQ-12).

## Open questions (requirement is ambiguous or incomplete)

1. **OQ-1 Whitespace:** Do spaces count towards the 3 characters? Is the input trimmed? Is "a b" (3 characters, 2 meaningful) valid?
2. **OQ-2 What is a "character"?** Code points, UTF-16 units, or grapheme clusters (emoji, accents with combining marks)? Are digits and symbols counted?
3. **OQ-3 Feedback below the minimum:** Should the user see a hint ("Type at least 3 characters"), a disabled button, or nothing?
4. **OQ-4 Results when input drops below 3:** Keep previous results, clear them, or show a default/landing state?
5. **OQ-5 Server-side rule:** Must the API also reject queries under 3 characters, and with what response/status?
6. **OQ-6 Maximum length:** Is there an upper limit on query length, and what happens beyond it?
7. **OQ-7 Definition of relevance:** Which fields (name, description, SKU, category, brand) are searched and how are they weighted? Does it involve text scoring, popularity, stock, or personalisation? Without this, expected order cannot be stated precisely.
8. **OQ-8 Tie-breaking:** Secondary ordering when relevance is equal (alphabetical, price, newest)?
9. **OQ-9 Matching rules:** Case sensitivity, accent insensitivity, partial/prefix versus whole word, typo tolerance, stemming and synonyms.
10. **OQ-10 Other sorting/filtering:** Can the user change the sort order? Does relevance remain the default after filters?
11. **OQ-11 Trigger mechanism:** Search-as-you-type (with what debounce) or only on submit? Does "runs" mean on every keystroke past 3?
12. **OQ-12 Non-functional targets:** Expected response time, page size, maximum results returned.
13. **OQ-13 Scope:** Does the rule apply to all channels (web, mobile app, API, voice) and to autocomplete/suggestions as well as the full results page?
14. **OQ-14 Out-of-stock/hidden products:** Are they included in results and ranked?

## Coverage summary
- Positive: TC09-01 to 04, 13, 17 to 20, 22 to 25
- Negative: TC09-05 to 09, 14, 15
- Boundary: 0, 1, 2, 3 and 4+ characters (TC09-05 to 07, 01, 02), long input (TC09-16), ties (TC09-21)
- Non-functional: TC09-27 to 30
