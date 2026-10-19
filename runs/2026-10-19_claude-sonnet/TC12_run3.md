# Test Cases: Customer List CSV Export

Requirement: A user can export the current list of customers to a CSV file. The export contains only the rows that match the active filters. The maximum export size is 10,000 rows.

## Assumptions
Only what the requirement states is assumed. Anything beyond that is listed under Open Questions, and expected results that depend on an answer are marked **[OQ-n]**.

## Test Cases

### Positive

**TC-01: Export with no filters, small list**
- Preconditions: User can open the customer list. 50 customers exist. No filters active.
- Steps: 1. Open the customer list. 2. Click Export to CSV.
- Expected: A CSV file is produced containing all 50 customers, one row per customer. Row count matches the list.

**TC-02: Export with a single active filter**
- Preconditions: 200 customers exist, 40 match filter "Country = Germany".
- Steps: 1. Apply the filter. 2. Confirm the list shows 40 rows. 3. Click Export.
- Expected: CSV contains exactly those 40 customers. No non-matching customers are present.

**TC-03: Export with multiple combined filters**
- Preconditions: Customers with varied status and country. A known number (N) match "Status = Active" AND "Country = France".
- Steps: 1. Apply both filters. 2. Export.
- Expected: CSV contains exactly the N customers matching both filters, per the list's combination logic.

**TC-04: Export reflects the current list state after filter change**
- Preconditions: Filter A applied, then changed to filter B.
- Steps: 1. Apply filter A. 2. Change to filter B. 3. Export.
- Expected: CSV matches filter B only, with no residue from A.

**TC-05: Export after clearing filters**
- Preconditions: A filter was applied, then removed.
- Steps: 1. Apply filter. 2. Clear filters. 3. Export.
- Expected: CSV contains the full unfiltered list (up to the limit).

**TC-06: CSV validity and content integrity**
- Preconditions: Customers whose data contains commas, double quotes, line breaks, and non-ASCII characters (e.g. "Müller, Anna", `Acme "Pro" Ltd`, Chinese or accented names).
- Steps: 1. Export. 2. Open the file in a text editor and in a spreadsheet application.
- Expected: File parses as valid CSV. Special characters are correctly quoted or escaped. Values are unaltered. Characters display correctly. Column count is consistent in every row. [OQ-3, OQ-4]

**TC-07: Row data matches the UI**
- Preconditions: Filtered list of 10 customers.
- Steps: 1. Export. 2. Compare each CSV row with the on-screen row.
- Expected: Values of every exported field match the source data for each customer. [OQ-2]

**TC-08: Export respects current sort order**
- Preconditions: List sorted by Name descending.
- Steps: 1. Export.
- Expected: Row order in the CSV equals the on-screen order. [OQ-6]

### Boundary

**TC-09: Exactly 10,000 rows**
- Preconditions: Filter result set is exactly 10,000 customers.
- Steps: 1. Apply filter. 2. Export.
- Expected: Export succeeds. CSV contains 10,000 data rows with no truncation.

**TC-10: 9,999 rows**
- Preconditions: Filter result set is 9,999 customers.
- Steps: Export.
- Expected: Export succeeds with 9,999 data rows.

**TC-11: 10,001 rows**
- Preconditions: Filter result set is 10,001 customers.
- Steps: Export.
- Expected: Behavior follows the limit rule: either the export is blocked with a clear message, or it is truncated to 10,000 with a clear notice. Customers are never silently dropped. [OQ-1]

**TC-12: Well above the limit**
- Preconditions: Filter result set is 50,000 customers.
- Steps: Export.
- Expected: Same limit handling as TC-11. No more than 10,000 rows are ever delivered. The system remains responsive (no timeout or crash). [OQ-1]

**TC-13: Filter narrows an over-limit list to under the limit**
- Preconditions: 15,000 customers total. A filter reduces the result to 8,000.
- Steps: 1. Observe the unfiltered state (over limit). 2. Apply the filter. 3. Export.
- Expected: The 8,000 matching rows are exported. The limit applies to filtered rows, not the total customers.

**TC-14: Exactly one row**
- Preconditions: Filter matches exactly 1 customer.
- Steps: Export.
- Expected: CSV contains the header (if applicable) and 1 data row. [OQ-3]

**TC-15: Empty result set (zero rows)**
- Preconditions: Filter matches no customers.
- Steps: Click Export (if available).
- Expected: Defined behavior: either Export is disabled or a "nothing to export" message is shown, or a header-only CSV is produced. No error or crash. [OQ-5]

### Negative

**TC-16: Filters that match nothing are not ignored**
- Preconditions: Filter yielding zero results, while customers exist in total.
- Steps: Export.
- Expected: The export never falls back to the full unfiltered list. [OQ-5]

**TC-17: Unauthorized user**
- Preconditions: User without export permission (if roles exist).
- Steps: Open the list. Try to export, including by calling the export endpoint directly.
- Expected: Export is hidden, disabled, or rejected with an authorization error. No data is returned. [OQ-7]

**TC-18: Unauthenticated or expired session**
- Preconditions: Session expires after the list is loaded.
- Steps: Click Export.
- Expected: User is prompted to re-authenticate. No file is delivered. No data leakage.

**TC-19: Server or network failure during export**
- Preconditions: Simulate a network drop or a 500 error during generation.
- Steps: Click Export.
- Expected: A clear error message is shown. No partial or corrupt file is presented as successful. The user can retry.

**TC-20: Repeated clicks on Export**
- Preconditions: Large result set (near 10,000).
- Steps: Click Export multiple times quickly.
- Expected: Either one export is generated or each is valid and consistent. No duplicated or corrupted files. [OQ-8]

**TC-21: Data changes between viewing and exporting**
- Preconditions: Two users. User A views a filtered list. User B adds or deletes a matching customer.
- Steps: User A clicks Export.
- Expected: Behavior is defined: export reflects either the displayed state or the current data. The row count is within the limit and consistent with the stated rule. [OQ-9]

**TC-22: Injection through data (formula injection)**
- Preconditions: Customer name such as `=HYPERLINK("http://example.com")` or `+cmd|' /C calc'!A0`.
- Steps: Export and open in a spreadsheet application.
- Expected: Formulas are not executed (cells neutralized per the security policy). [OQ-10]

**TC-23: Tampered filter parameters**
- Preconditions: Export triggered via a request that includes filter parameters.
- Steps: Modify the request to alter or remove filters, or to request a higher limit (e.g. 50,000).
- Expected: The server enforces the 10,000 cap and valid filter semantics. Invalid input is rejected.

### Non-functional (basic)

**TC-24: Performance at maximum size**
- Preconditions: 10,000-row result set.
- Steps: Export and measure the time.
- Expected: Completes within the agreed threshold without UI freeze. [OQ-11]

**TC-25: Filename and download behavior**
- Preconditions: Any non-empty result.
- Steps: Export in the supported browsers.
- Expected: The file downloads with a .csv extension and a sensible name. [OQ-12]

## Open Questions

1. **Over-limit behavior:** What happens when the matching rows exceed 10,000? Is the export blocked with an error, truncated to the first 10,000, or split into several files? Should the user be warned before or after the click?
2. **Columns:** Which fields are exported? Are they the visible columns only, or all customer attributes? Are sensitive fields (e.g. personal data) included?
3. **Header row, delimiter, encoding:** Is there a header row? Which delimiter (comma, semicolon, locale-dependent)? Which encoding (UTF-8, with or without BOM)?
4. **Formatting of values:** How are dates, numbers, currency, and null or empty values formatted? Do they follow the UI locale?
5. **Empty result:** Is Export disabled, does it show a message, or does it produce a header-only file?
6. **Sort order:** Does the export follow the on-screen sort, or is the order unspecified?
7. **Permissions:** Which roles may export? Is export audited or logged?
8. **Concurrency:** Is the export synchronous or asynchronous (e.g. emailed link)? Are duplicate requests prevented?
9. **Data consistency:** Does the export use a snapshot from the moment the list was loaded, or live data at click time?
10. **Security:** Is CSV formula injection mitigation required?
11. **Performance:** What is the acceptable export time at 10,000 rows?
12. **File naming and supported browsers or platforms:** Is there a naming convention (e.g. including a timestamp)? Which clients are in scope?
13. **"Current list":** Does it include search text, sorting, and pagination state? Does the export cover all pages of the filtered results or only the visible page?
14. **Definition of "active filters":** Are filters persisted across sessions? Are default or hidden filters (e.g. "exclude archived") included?
15. **Is the 10,000 limit configurable**, and is it counted in rows including or excluding the header?
