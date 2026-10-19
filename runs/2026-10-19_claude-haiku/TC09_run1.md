# TC09 - Test cases: product search threshold and relevance ordering

## Requirement

"The product search runs only when the user has typed at least 3 characters. Results are shown in order of relevance."

## Test cases

### Positive scenarios

**TC09-01 - Search runs at exactly 3 characters**
- Preconditions: User is on a page with the product search box. The catalogue contains at least one product whose name starts with "abc".
- Steps:
  1. Click into the product search box.
  2. Type "abc".
- Expected result: The search runs and results are displayed for "abc".

**TC09-02 - Search runs for input longer than 3 characters**
- Preconditions: As TC09-01.
- Steps:
  1. Type "abcd" into the product search box.
- Expected result: The search runs and results for "abcd" are displayed, ordered by relevance.

**TC09-03 - Results are ordered by relevance (exact match first)**
- Preconditions: The catalogue contains product "Wireless Mouse" and product "Mouse Pad Wireless XL", and both match "mouse".
- Steps:
  1. Type "mouse" into the product search box.
  2. Observe the order of the results.
- Expected result: "Wireless Mouse" (the product whose name best matches the query) is listed before "Mouse Pad Wireless XL". Results are not in alphabetical or price order.

**TC09-04 - Relevance order is stable for the same query**
- Preconditions: As TC09-03.
- Steps:
  1. Type "mouse" and record the result order.
  2. Clear the box, then type "mouse" again.
  3. Record the result order.
- Expected result: Both result orders are identical.

**TC09-05 - Search runs for a multi-word query of 3 or more characters**
- Preconditions: The catalogue contains "Red Running Shoes".
- Steps:
  1. Type "red run" into the product search box.
- Expected result: The search runs and "Red Running Shoes" is among the results.

### Negative scenarios

**TC09-06 - Search does not run with an empty box**
- Preconditions: User is on a page with the product search box. No text has been entered.
- Steps:
  1. Click into the product search box and leave it empty.
  2. Click outside the box.
- Expected result: No search request is sent and no results are displayed.

**TC09-07 - Search does not run with 1 character**
- Preconditions: As TC09-06.
- Steps:
  1. Type "a" into the product search box.
- Expected result: No search request is sent and no results are displayed.

**TC09-08 - Search does not run with 2 characters**
- Preconditions: As TC09-06.
- Steps:
  1. Type "ab" into the product search box.
- Expected result: No search request is sent and no results are displayed.

**TC09-09 - Search does not run for whitespace-only input**
- Preconditions: As TC09-06.
- Steps:
  1. Type three spaces "   " into the product search box.
- Expected result: No search request is sent and no results are displayed. (Subject to open question Q2.)

**TC09-10 - Valid query with no matching products**
- Preconditions: The catalogue contains no product matching "zzz".
- Steps:
  1. Type "zzz" into the product search box.
- Expected result: The search runs, and an empty-result message is displayed. No error is shown.

**TC09-11 - Backend error during search**
- Preconditions: The search service returns an error (simulated with a stub or by disabling the service).
- Steps:
  1. Type "abc" into the product search box.
- Expected result: An error message is shown to the user. The page does not crash and the search box remains usable.

### Boundary scenarios

**TC09-12 - Threshold is reached exactly on the third character**
- Preconditions: As TC09-01.
- Steps:
  1. Type "a", then "b".
  2. Observe that no search has run.
  3. Type "c" so the box contains "abc".
- Expected result: No search runs after "a" or "ab". The search runs once the box reaches "abc".

**TC09-13 - Results are cleared when the input drops below 3 characters**
- Preconditions: Results for "abcd" are displayed.
- Steps:
  1. Delete one character so the box contains "abc".
  2. Delete one more so the box contains "ab".
- Expected result: After "abc" the search runs and results are shown. After "ab" the behaviour matches the answer to open question Q3. The expected behaviour is not defined by the requirement and must be confirmed before this test is executed.

**TC09-14 - Paste of a string longer than 3 characters**
- Preconditions: Clipboard contains "keyboard".
- Steps:
  1. Paste "keyboard" into the empty product search box.
- Expected result: The search runs once for "keyboard" and results are displayed. No search runs for partial strings.

**TC09-15 - Paste of a string shorter than 3 characters**
- Preconditions: Clipboard contains "ab".
- Steps:
  1. Paste "ab" into the empty product search box.
- Expected result: No search runs.

**TC09-16 - Multi-byte characters are counted as characters**
- Preconditions: The catalogue contains a product whose name starts with "żół" or another non-ASCII three-character prefix.
- Steps:
  1. Type three non-ASCII characters (for example "żół").
- Expected result: The search runs after the third character, the same as for ASCII input. (Subject to open question Q1.)

**TC09-17 - Very long query**
- Preconditions: As TC09-01.
- Steps:
  1. Paste a 500-character string into the product search box.
- Expected result: The search runs and no crash or hang occurs. Any length limit behaviour is subject to open question Q9.

**TC09-18 - Case-insensitive match**
- Preconditions: The catalogue contains "Mouse".
- Steps:
  1. Type "MOU" into the product search box.
- Expected result: The search runs (the threshold is met) and "Mouse" is returned. (Subject to open question Q6.)

## Open questions

The requirement is incomplete on the points below. No behaviour has been assumed for them; the related tests are marked as subject to these answers.

- **Q1.** How are "characters" counted: Unicode code points, grapheme clusters, or bytes? (Affects TC09-16.)
- **Q2.** Are spaces counted toward the 3 characters? Is the input trimmed before counting, so that "   " or " a " do not count? (Affects TC09-09.)
- **Q3.** When the user deletes characters so that the input falls below 3, what happens to the displayed results: are they cleared, kept, or greyed out? Is a pending search cancelled? (Affects TC09-13.)
- **Q4.** Does the search run on every keystroke once the threshold is met, or is there a debounce delay, a submit button, or an Enter key requirement? If there is a debounce, what is the delay?
- **Q5.** What defines "relevance"? Which fields are weighted (name, description, category, tags), and is there a tie-breaking rule when scores are equal? Is the order fixed, or can the user change the sort? (Affects TC09-03 and TC09-04.)
- **Q6.** Is matching case-insensitive and diacritic-insensitive? Does it match only word prefixes, substrings, or fuzzy matches (typos)? (Affects TC09-18.)
- **Q7.** Does the 3-character rule apply equally to all search entry points, such as the header search, category pages, and the public API?
- **Q8.** Is the 3-character rule enforced only in the UI, or also in the backend or API? What does the API return for a 2-character query: an error, an empty list, or a full result set?
- **Q9.** Is there a maximum query length? If so, what is the behaviour above it: truncation, rejection, or an error message? (Affects TC09-17.)
- **Q10.** What is displayed when a query returns no results, and what is displayed on a timeout or server error? Is the exact message text specified? (Affects TC09-10 and TC09-11.)
- **Q11.** Is pagination or infinite scrolling used? If so, does the relevance order hold across pages?
- **Q12.** Does the search trigger for input entered by voice, autocomplete, or browser suggestions in the same way as for typed input? (Affects TC09-14.)
- **Q13.** Is the search also triggered by a suggestion dropdown (type-ahead) that has its own minimum length, and is that minimum the same 3 characters?
