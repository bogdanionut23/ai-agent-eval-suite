# TC09 - Test cases: product search (min 3 characters, relevance order)

Requirement: The product search runs only when the user has typed at least 3 characters. Results are shown in order of relevance.

Assumptions used (only where the requirement is silent, flagged so they can be confirmed): the catalogue has products named e.g. "Laptop", "Lamp", "Lap desk"; "search runs" means a search request is sent and results are displayed. Where behaviour is not defined by the requirement, the expected result says "see OQ-n" rather than inventing behaviour.

## Positive

**TC-01 - Search runs at exactly 3 characters (boundary)**
- Preconditions: User is on the product search page; catalogue contains products matching "lap".
- Steps: 1. Click the search field. 2. Type "lap".
- Expected: Search runs; a result list for "lap" is displayed.

**TC-02 - Search runs with more than 3 characters**
- Preconditions: As TC-01; products matching "laptop" exist.
- Steps: 1. Type "laptop".
- Expected: Search runs; results matching "laptop" are displayed.

**TC-03 - Results ordered by relevance**
- Preconditions: Known test data: product A is an exact name match for "laptop", product B mentions "laptop" only in its description, product C is a weaker match. Relevance ranking as defined by the product owner (see OQ-1).
- Steps: 1. Type "laptop". 2. Inspect the order of results.
- Expected: Results are listed from most to least relevant (A before B before C). Order is not alphabetical, by price or by date unless that equals relevance.

**TC-04 - Relevance order is stable for the same query**
- Preconditions: As TC-03.
- Steps: 1. Search "laptop" twice (reload the page between runs).
- Expected: Same results in the same order both times, given unchanged data.

**TC-05 - Search triggered when the 3rd character is reached by continued typing**
- Preconditions: Empty search field.
- Steps: 1. Type "l", "a", then "p", one character at a time.
- Expected: No search after 1st and 2nd character; search runs once the 3rd character is typed (subject to OQ-2 on how the search is triggered).

**TC-06 - Search again after editing a valid query**
- Preconditions: Results shown for "laptop".
- Steps: 1. Change the text to "lamp".
- Expected: Search runs for "lamp"; the result list is replaced with results for "lamp", in relevance order.

## Negative

**TC-07 - No search with 0 characters**
- Preconditions: Empty search field.
- Steps: 1. Focus the field. 2. Submit/press Enter without typing.
- Expected: No search is run. No results list is produced from a search. Message/feedback behaviour: see OQ-3.

**TC-08 - No search with 1 character**
- Steps: 1. Type "l".
- Expected: No search runs; no request is sent.

**TC-09 - No search with 2 characters (boundary, one below minimum)**
- Steps: 1. Type "la".
- Expected: No search runs; no request is sent.

**TC-10 - Deleting back below 3 characters**
- Preconditions: Results shown for "lap".
- Steps: 1. Press Backspace once to give "la".
- Expected: No new search runs for "la". What happens to the previously shown results: see OQ-4.

**TC-11 - Explicit submit with 2 characters**
- Steps: 1. Type "la". 2. Press Enter or click the search button (if present).
- Expected: Search does not run (requirement says "only when").

**TC-12 - Whitespace only, 3 spaces**
- Steps: 1. Type "   " (3 spaces).
- Expected: Behaviour depends on whether spaces count as characters; see OQ-5. Must not produce an error or crash.

**TC-13 - Padded short term, " a " (space, letter, space)**
- Steps: 1. Type " a ".
- Expected: Depends on trimming rule; see OQ-5.

**TC-14 - Query with no matching products**
- Steps: 1. Type "zzzzqq".
- Expected: Search runs (>= 3 chars); empty-result state shown without error. Wording of the empty state: see OQ-6.

**TC-15 - Pasted text shorter than 3 characters**
- Steps: 1. Paste "ab" into the field.
- Expected: No search runs.

## Boundary and special input

**TC-16 - Pasted text of exactly 3 characters**
- Steps: 1. Paste "lap".
- Expected: Search runs.

**TC-17 - Special characters, 3 chars: "@#%"**
- Steps: 1. Type "@#%".
- Expected: Counts as 3 characters (see OQ-5); search runs or returns empty results without error or injection effects.

**TC-18 - Unicode / multi-byte, 3 characters: "日本語"**
- Steps: 1. Type "日本語".
- Expected: Counted as 3 characters (not bytes); search runs.

**TC-19 - Emoji / combining characters**
- Steps: 1. Type "e" + combining accent + "ab" ("éab" via combining mark).
- Expected: Character counting rule undefined; see OQ-5. No crash.

**TC-20 - Very long query (e.g. 1000+ characters)**
- Steps: 1. Paste a 1000-character string.
- Expected: No crash or UI break; behaviour per maximum length rule, see OQ-7.

**TC-21 - Mixed case gives same results**
- Steps: 1. Search "LAP". 2. Search "lap".
- Expected: Both run and, unless case sensitivity is specified otherwise (OQ-8), return the same results in the same order.

**TC-22 - SQL/script injection string**
- Steps: 1. Type "' OR 1=1 --" and "<script>alert(1)</script>".
- Expected: Search runs (>= 3 chars) but input is handled safely: no SQL error, no script execution, no data leak; results are plain matches or empty.

**TC-23 - Rapid typing past the threshold**
- Steps: 1. Type "laptop" quickly.
- Expected: Final result list corresponds to "laptop", not to an earlier partial query (no out-of-order response overwriting). Number of searches issued: see OQ-2.

**TC-24 - Relevance ties**
- Preconditions: Two products with identical relevance for the query.
- Steps: 1. Search the term.
- Expected: Both shown; their relative order is deterministic per the tie-break rule (OQ-9).

**TC-25 - Large result set ordering across pages**
- Preconditions: More matches than fit on one page.
- Steps: 1. Search a broad term. 2. Go to page 2.
- Expected: Page 2 continues the relevance order from page 1 with no duplicates or reordering (pagination: OQ-10).

## Open questions (requirement is ambiguous or incomplete)

- OQ-1: How is "relevance" defined (name match vs description, exact vs partial, popularity, stock, boosting)? Without it, TC-03 cannot have a precise expected order.
- OQ-2: What triggers the search: on every keystroke (as-you-type), after a debounce, or only on submit (Enter/button)? "Typed at least 3 characters" suggests as-you-type but this is not stated.
- OQ-3: What does the user see with fewer than 3 characters: nothing, a hint such as "Enter at least 3 characters", a disabled button?
- OQ-4: When the user deletes below 3 characters after results were shown, are the old results kept, cleared, or replaced by a hint?
- OQ-5: How are characters counted: are leading/trailing/inner spaces counted or trimmed? Are emoji, combining marks, and special characters counted as one character each?
- OQ-6: What is the expected empty-result message and behaviour (suggestions, spell correction)?
- OQ-7: Is there a maximum query length, and what happens above it?
- OQ-8: Is the search case-insensitive and accent-insensitive?
- OQ-9: What is the tie-break order for equal relevance?
- OQ-10: Is there pagination, a result limit, or infinite scroll; and does the relevance order hold across pages?
- OQ-11: Does the 3-character rule apply equally to all entry points (header search, mobile, voice, URL query parameter such as /search?q=ab, API)? Server-side enforcement is not specified.
- OQ-12: Can users sort or filter results afterwards, overriding relevance order? Is relevance the default only?
- OQ-13: Performance and accessibility expectations (response time, screen-reader announcement of results or of the minimum-length rule) are not stated.
