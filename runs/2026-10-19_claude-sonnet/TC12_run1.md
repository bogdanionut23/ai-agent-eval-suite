# Test Cases: Customer List CSV Export

Requirement: A user can export the current list of customers to a CSV file. The export contains only the rows that match the active filters. The maximum export size is 10,000 rows.

## Assumptions

None of the behaviour below is invented. Where the requirement is silent, the expected result is marked "see OQ-n" and the open question is listed in the last section.

## Test cases

### Positive

**TC-01: Export with no filters, small list**
- Preconditions: User is logged in with export permission. The customer list has 50 customers. No filters are active.
- Steps: 1. Open the customer list. 2. Click Export to CSV.
- Expected: A CSV file is downloaded containing 50 data rows, one per customer in the list. The columns are consistent with the list (see OQ-3 for the exact column set).

**TC-02: Export with one active filter**
- Preconditions: 200 customers; 40 match the filter (for example Country = Germany).
- Steps: 1. Apply the filter. 2. Confirm the list shows 40 rows. 3. Click Export.
- Expected: The CSV contains exactly 40 data rows. Every row matches the filter, and no non-matching customer appears.

**TC-03: Export with multiple combined filters**
- Preconditions: Customers with varied country, status and creation date. Two or more filters are applied together.
- Steps: 1. Apply the filters. 2. Note the row count shown. 3. Export.
- Expected: The CSV contains exactly the rows that satisfy all active filters, matching the on-screen count.

**TC-04: Export reflects a changed filter**
- Preconditions: A filter is applied and exported once.
- Steps: 1. Export. 2. Change the filter value. 3. Export again.
- Expected: The second file reflects the new filter only. It has no leftover rows from the first filter.

**TC-05: Export after clearing filters**
- Preconditions: A filter is applied, then cleared.
- Steps: 1. Apply the filter. 2. Clear all filters. 3. Export.
- Expected: The CSV contains all customers (up to the limit), not the previously filtered subset.

**TC-06: File is valid CSV**
- Preconditions: The list contains customers whose values include a comma, a double quote, a line break and non-ASCII characters (for example "Müller, Hans" and a name with an accent or CJK characters).
- Steps: 1. Export. 2. Open the file in a CSV parser and in a spreadsheet application.
- Expected: The file parses without error. Special characters are escaped correctly, the header row is present (see OQ-3), and the number of fields is the same on every row. Non-ASCII characters display correctly (see OQ-5 for encoding).

**TC-07: Exported values match the list**
- Preconditions: A list with a known set of customers.
- Steps: 1. Export. 2. Compare each CSV row to the on-screen record.
- Expected: Values are identical for every exported row, with no truncation or reformatting that loses data.

### Boundary

**TC-08: Exactly 10,000 matching rows**
- Preconditions: Exactly 10,000 customers match the active filters.
- Steps: 1. Click Export.
- Expected: Export succeeds. The CSV has exactly 10,000 data rows.

**TC-09: 9,999 matching rows**
- Preconditions: Exactly 9,999 customers match.
- Steps: 1. Export.
- Expected: Export succeeds with 9,999 data rows.

**TC-10: 10,001 matching rows**
- Preconditions: Exactly 10,001 customers match.
- Steps: 1. Export.
- Expected: Behaviour is defined by the product owner (see OQ-1). The export must not silently produce a file that looks complete while omitting a row. The user must be told either that the export was blocked or that it was truncated.

**TC-11: Far above the limit**
- Preconditions: 50,000 customers match (no filters).
- Steps: 1. Export.
- Expected: Same handling as TC-10. The system stays responsive, does not time out or crash, and never writes more than 10,000 rows.

**TC-12: Exactly 1 matching row**
- Preconditions: One customer matches the filters.
- Steps: 1. Export.
- Expected: CSV with a header (see OQ-3) and one data row.

**TC-13: Filter takes the result from above the limit to within it**
- Preconditions: 12,000 customers in total. A filter reduces the result to 9,500.
- Steps: 1. Without a filter, attempt export (limit handling applies). 2. Apply the filter. 3. Export.
- Expected: The filtered export succeeds with 9,500 rows. The limit applies to the filtered count and not to the total number of customers.

### Negative

**TC-14: No rows match the filters**
- Preconditions: The active filters return 0 customers.
- Steps: 1. Click Export (or observe whether the control is available).
- Expected: Behaviour as decided in OQ-2. The system must not crash or download a corrupt file. Either the action is disabled or a clear message is shown, or a header-only file is produced.

**TC-15: Empty customer list (no customers at all)**
- Preconditions: The system has no customers.
- Steps: 1. Open the list. 2. Try to export.
- Expected: Same as TC-14, with no error and no malformed file.

**TC-16: User without export permission**
- Preconditions: A user whose role does not allow export (see OQ-6).
- Steps: 1. Open the customer list. 2. Try to export, including by calling the export endpoint directly.
- Expected: The export control is hidden or disabled, and a direct request is rejected with an authorization error (403 or equivalent). No data is returned.

**TC-17: Unauthenticated or expired session**
- Preconditions: The session has expired.
- Steps: 1. Click Export.
- Expected: The user is redirected to login or shown a session-expired message. No file is produced and no data is leaked.

**TC-18: Server or network failure during export**
- Preconditions: Simulate a failure (server error or dropped connection) mid-export.
- Steps: 1. Click Export.
- Expected: A clear error message is shown. No partial or corrupt file is presented as a successful export, and the user can retry.

**TC-19: Repeated clicks on Export**
- Preconditions: A large result set (for example 10,000 rows) so export takes noticeable time.
- Steps: 1. Click Export several times quickly.
- Expected: No duplicate or conflicting downloads, or the duplicates are well formed. The UI shows progress or disables the button.

**TC-20: Filter input that could break the query or CSV (injection)**
- Preconditions: A filter on a free-text field.
- Steps: 1. Enter filter text such as `' OR 1=1 --` and, separately, text starting with `=`, `+`, `-` or `@` (for example `=HYPERLINK("http://x")`). 2. Export. 3. Open in a spreadsheet application.
- Expected: The filter does not return unfiltered data. Customer values that begin with formula characters do not execute as formulas when the file is opened (see OQ-7).

### Consistency and security

**TC-21: Data changes between viewing and exporting**
- Preconditions: Two sessions. Session A has a filter applied. Session B adds or edits a matching customer.
- Steps: 1. In session B, add a customer matching A's filter. 2. In session A, export without refreshing.
- Expected: Behaviour per OQ-4 (snapshot of the displayed list or live data). The result must be consistent and documented.

**TC-22: Sort order**
- Preconditions: The list is sorted by a column (for example name descending).
- Steps: 1. Export.
- Expected: Row order in the CSV follows the on-screen sort (see OQ-8). If not, the order is documented.

**TC-23: Pagination does not limit the export**
- Preconditions: 500 matching customers, page size 25.
- Steps: 1. View page 1 (or page 3). 2. Export.
- Expected: The CSV contains all 500 matching rows, not only the current page.

**TC-24: Data scope and multi-tenancy**
- Preconditions: User belongs to account or tenant A. Tenant B has customers.
- Steps: 1. Export.
- Expected: Only data the user is permitted to see appears. No rows from other tenants.

## Open questions

1. **Over-limit behaviour (OQ-1).** What should happen when more than 10,000 rows match? Options: block the export with a message, truncate to the first 10,000 with a warning, or offer to refine the filters. Which rows are kept when truncating, and in what order?
2. **Zero results (OQ-2).** Should Export be disabled, show a message, or produce a header-only file when no rows match?
3. **Columns and header (OQ-3).** Which columns are exported? Does that include columns hidden in the UI? Is there a header row? Are column names the UI labels or technical names?
4. **Data consistency (OQ-4).** Is the export a snapshot of what is on screen, or a fresh query when the button is pressed?
5. **File format details (OQ-5).** Delimiter (comma or locale-dependent semicolon), encoding (UTF-8 with or without BOM), line endings, date, number and currency formats, and the file naming convention.
6. **Permissions (OQ-6).** Which roles may export? Is the export audited or logged?
7. **Sensitive data and CSV injection (OQ-7).** Is personal data (PII) masked or excluded? Must values starting with `=`, `+`, `-` or `@` be neutralised?
8. **Sort order (OQ-8).** Does the export follow the current sort? Is "current list" meant to include the sort?
9. **Delivery mechanism.** Is the export synchronous (browser download) or asynchronous (email or notification link)? Is there a timeout, and what is the expected performance for 10,000 rows?
10. **Limit scope.** Is the 10,000 limit a system-wide constant or configurable? Is it per export or per user per day? Does it count the header row?
11. **Search versus filters.** Does a free-text search box count as an "active filter"? What about a default filter applied automatically (for example "Active customers only")?
12. **Concurrency and rate limiting.** Can a user run several exports at once? Are there rate limits?
