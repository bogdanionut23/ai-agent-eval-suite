# Test Cases: Customer List CSV Export

Requirement: A user can export the current list of customers to a CSV file. The export contains only the rows that match the active filters. The maximum export size is 10,000 rows.

## Assumptions

Where the requirement does not say, I do not assume behaviour. Expected results that depend on an open question are marked **[OQ-n]**. The tester or product owner must confirm them before execution.

## Test cases

### Positive

**TC-EXP-001: Export the full list with no filters**
- Preconditions: User is logged in and has export permission. 50 customers exist. No filters are active.
- Steps: 1. Open the customer list. 2. Confirm no filters are applied. 3. Click Export to CSV.
- Expected: A CSV file downloads. It has 50 data rows, one per customer. The file opens in a spreadsheet tool without errors.

**TC-EXP-002: Export with a single active filter**
- Preconditions: 100 customers exist. 30 have Status = Active. Filter Status = Active is applied, and the list shows 30 rows.
- Steps: 1. Click Export to CSV. 2. Open the file.
- Expected: The CSV has exactly 30 data rows. All have Status = Active. No row from the other 70 customers is present.

**TC-EXP-003: Export with multiple active filters combined**
- Preconditions: Filters Status = Active and Country = Germany are applied (combined as the product defines, see OQ-6). The list shows N rows.
- Steps: 1. Click Export to CSV. 2. Compare the file to the on-screen list.
- Expected: The CSV rows equal the on-screen rows, with the same count and the same customers.

**TC-EXP-004: Exported data matches the list values**
- Preconditions: A filtered list of at least 5 customers is displayed.
- Steps: 1. Export. 2. Compare each CSV row with the UI row for every column.
- Expected: Values match the UI. Columns match what OQ-2 confirms. The first row is a header row if confirmed by OQ-3.

**TC-EXP-005: Export respects the current sort order**
- Preconditions: List is sorted by Name descending, with a filter applied.
- Steps: 1. Export. 2. Check row order in the file.
- Expected: Row order matches the on-screen order **[OQ-4]**.

**TC-EXP-006: Export after changing filters reflects the new filters**
- Preconditions: Filter A applied, then changed to Filter B.
- Steps: 1. Apply filter A. 2. Change to filter B. 3. Export.
- Expected: The CSV contains only rows matching filter B. No rows from filter A remain.

**TC-EXP-007: Export after clearing filters**
- Preconditions: A filter is applied, then cleared.
- Steps: 1. Apply filter. 2. Clear the filter. 3. Export.
- Expected: The CSV contains all customers (up to the limit).

### Boundary

**TC-EXP-008: Exactly 10,000 rows**
- Preconditions: Filter result set is exactly 10,000 customers.
- Steps: 1. Click Export. 2. Count data rows in the file.
- Expected: Export succeeds with 10,000 data rows, none omitted.

**TC-EXP-009: 9,999 rows**
- Preconditions: Filter result set is 9,999 customers.
- Steps: Export and count rows.
- Expected: Export succeeds with 9,999 rows.

**TC-EXP-010: 10,001 rows**
- Preconditions: Filter result set is 10,001 customers.
- Steps: 1. Click Export. 2. Observe the outcome.
- Expected: The system enforces the limit as defined in OQ-1 (export blocked with a message, or truncated to 10,000 with a visible warning). It must not silently produce a file that looks complete but is not.

**TC-EXP-011: Zero matching rows**
- Preconditions: Filters are applied that match no customers.
- Steps: Click Export.
- Expected: Behaviour per OQ-5 (empty file with header only, or a message that there is nothing to export). No error or crash.

**TC-EXP-012: Exactly one matching row**
- Preconditions: Filter matches one customer.
- Steps: Export.
- Expected: CSV contains one data row (plus header if applicable).

**TC-EXP-013: Large total list, small filtered result**
- Preconditions: 50,000 customers exist. Filter returns 200.
- Steps: Export.
- Expected: Export succeeds with 200 rows. The limit applies to the filtered rows, not to the total list **[OQ-1]**.

**TC-EXP-014: Large total list, filter result over the limit**
- Preconditions: 50,000 customers exist, no filters.
- Steps: Click Export.
- Expected: Limit behaviour per OQ-1. The user is clearly informed.

### Negative

**TC-EXP-015: User without export permission**
- Preconditions: Logged-in user lacks export permission **[OQ-7]**.
- Steps: Open the customer list and look for the Export action. Attempt a direct request to the export endpoint.
- Expected: The action is hidden or disabled in the UI. The direct request is rejected (403) and no data is returned.

**TC-EXP-016: Unauthenticated request**
- Preconditions: Session expired or not logged in.
- Steps: Call the export endpoint or click Export after the session expires.
- Expected: The user is redirected to login or receives 401. No data is returned.

**TC-EXP-017: Server or network failure during export**
- Preconditions: Simulate a server error or a dropped connection mid-export.
- Steps: Click Export.
- Expected: A clear error message is shown. No partial or corrupt file is presented as successful. The user can retry.

**TC-EXP-018: Double click on Export**
- Preconditions: Filtered list displayed.
- Steps: Double-click Export quickly.
- Expected: No duplicate or corrupt downloads, or the second click is ignored while the first export is in progress **[OQ-8]**.

**TC-EXP-019: Data changes between viewing and exporting**
- Preconditions: Another user adds or deletes a matching customer after the list was loaded.
- Steps: 1. Load the list. 2. Other user modifies data. 3. Export without refreshing.
- Expected: Behaviour per OQ-9 (snapshot of what was displayed, or current data at export time). The file must be consistent with one of those two, not a mix.

### Data integrity and format

**TC-EXP-020: Special characters are escaped correctly**
- Preconditions: Customers with names containing commas, double quotes, and line breaks (for example `Smith, John`, `Acme "Best" Ltd`, a multi-line address).
- Steps: Export. Open in a spreadsheet tool and in a text editor.
- Expected: Fields are quoted and escaped per RFC 4180. Each customer stays on its own logical row. Columns do not shift.

**TC-EXP-021: Unicode and non-Latin characters**
- Preconditions: Customers with names such as `Müller`, `Žofia`, `李雷`, `Ōtani`.
- Steps: Export and open the file.
- Expected: Characters display correctly. Encoding is as defined in OQ-10.

**TC-EXP-022: Formula injection protection**
- Preconditions: A customer field contains a value starting with `=`, `+`, `-`, or `@` (for example `=HYPERLINK("http://x")`).
- Steps: Export and open in a spreadsheet application.
- Expected: The value is not executed as a formula (neutralised, for example with a leading apostrophe or quoting) **[OQ-11, security expectation to confirm]**.

**TC-EXP-023: Empty and null fields**
- Preconditions: Customers with blank optional fields.
- Steps: Export.
- Expected: Blank fields appear as empty values. The column count stays consistent in every row. No literal "null" or "undefined" text.

**TC-EXP-024: Dates, numbers and time zones**
- Preconditions: Customers with date and numeric fields.
- Steps: Export and compare with the UI.
- Expected: Format and time zone follow OQ-12. Values are not altered or rounded.

**TC-EXP-025: File name and type**
- Preconditions: Any export.
- Steps: Download the file and check name, extension, and MIME type.
- Expected: Extension `.csv`, MIME `text/csv`, name per OQ-13.

### Filters and scope

**TC-EXP-026: Text search or quick-search filter**
- Preconditions: A free-text search is applied and is treated as an active filter.
- Steps: Export.
- Expected: Only matching rows are exported **[OQ-6: whether search counts as a filter]**.

**TC-EXP-027: Pagination does not limit the export**
- Preconditions: Filtered result is 250 rows, page size is 25, user is on page 3.
- Steps: Export.
- Expected: The CSV contains all 250 rows, not just page 3 **[OQ-14]**.

**TC-EXP-028: Hidden or selected columns**
- Preconditions: User hid some columns in the list view.
- Steps: Export.
- Expected: Columns per OQ-2.

**TC-EXP-029: Filter values with special characters**
- Preconditions: Filter value includes an apostrophe, `%`, or Unicode characters.
- Steps: Apply the filter, then export.
- Expected: Filter is applied correctly and the CSV matches the on-screen result. No injection or error.

### Non-functional

**TC-EXP-030: Performance at maximum size**
- Preconditions: Result set of 10,000 rows.
- Steps: Export and measure time and memory.
- Expected: Completes within the time defined in OQ-15, with no timeout or UI freeze.

**TC-EXP-031: Concurrent exports by multiple users**
- Preconditions: Several users export at the same time.
- Steps: Each user exports their own filtered list.
- Expected: Each user receives only their own filtered data. No cross-contamination.

**TC-EXP-032: Export is audited**
- Preconditions: Audit logging exists **[OQ-16]**.
- Steps: Export, then check the audit log.
- Expected: An entry exists with the user, time, filters, and row count, if auditing is required.

**TC-EXP-033: Browser and platform compatibility**
- Preconditions: Supported browsers.
- Steps: Export on each supported browser.
- Expected: The file downloads and opens correctly on each.

## Open questions

| # | Question |
|---|----------|
| OQ-1 | What happens when results exceed 10,000 rows: block the export with a message, or truncate to the first 10,000 with a warning? Is the limit applied to filtered rows or to the total list? |
| OQ-2 | Which columns are exported: all customer fields, only visible columns, or a fixed set? Are sensitive or personal fields (such as email, phone) included? |
| OQ-3 | Is there a header row? Are column names the UI labels or technical names? |
| OQ-4 | Must the row order match the current sort order, or is order undefined? |
| OQ-5 | What should happen when zero rows match: an empty file, a header-only file, or a message and no download? |
| OQ-6 | Does the free-text search count as a "filter"? How are multiple filters combined (AND/OR)? |
| OQ-7 | Which roles may export? Is there a separate permission? |
| OQ-8 | Is the export synchronous or asynchronous (for example emailed or a background job)? Can several exports run at once for one user? |
| OQ-9 | Is the export a snapshot of the displayed list or a fresh query at click time? |
| OQ-10 | Required file encoding (UTF-8, with or without BOM), delimiter (comma or locale-dependent semicolon), and line ending? |
| OQ-11 | Is protection against CSV/formula injection required? |
| OQ-12 | Required formats for dates, numbers, currency, and time zone? |
| OQ-13 | File naming convention (for example timestamp included)? |
| OQ-14 | Does "current list" mean all pages of the filtered result, or only the page displayed? |
| OQ-15 | Performance expectations (maximum export time)? |
| OQ-16 | Are audit logging, rate limiting or notifications required? |
| OQ-17 | Does the export include soft-deleted, archived or inactive customers if the list hides them by default? |
| OQ-18 | Is the limit configurable, and is it the same for all roles? |
