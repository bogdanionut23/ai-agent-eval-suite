# TC12: Test cases for "Export customers list to CSV"

**Requirement:** A user can export the current list of customers to a CSV file. The export contains only the rows that match the active filters. The maximum export size is 10,000 rows.

Where the requirement is silent, the expected result is marked **[OQ-n]** and refers to the open questions at the end. Those expectations are provisional until the question is answered.

---

## Positive scenarios

### TC12-01: Export with no filters active exports the full customer list
- **Preconditions:** User is logged in with export permission. The system holds 50 customers. No filters are active.
- **Steps:**
  1. Open the customer list.
  2. Confirm no filter is applied.
  3. Click "Export to CSV".
  4. Open the downloaded file.
- **Expected result:** A `.csv` file downloads. It contains a header row and 50 data rows, one per customer. No row is missing or duplicated.

### TC12-02: Export contains only rows matching a single active filter
- **Preconditions:** User is logged in with export permission. The system holds 50 customers, 12 of them with Country = "DE".
- **Steps:**
  1. Open the customer list.
  2. Apply filter Country = "DE". Confirm the list shows 12 customers.
  3. Click "Export to CSV".
  4. Open the file.
- **Expected result:** The file has a header and exactly 12 data rows. Every data row has Country = "DE". No other customer appears.

### TC12-03: Export contains only rows matching all active filters combined
- **Preconditions:** Customers exist with mixed attributes, e.g. Country = "DE" and Status = "Active" for 5 customers, Country = "DE" and Status = "Inactive" for 4, Country = "FR" and Status = "Active" for 6.
- **Steps:**
  1. Apply filter Country = "DE" and filter Status = "Active".
  2. Confirm the list shows 5 customers.
  3. Export to CSV and open the file.
- **Expected result:** The file has exactly 5 data rows, each with Country = "DE" and Status = "Active". Customers that match only one of the two filters are excluded. [OQ-4]

### TC12-04: Row count in the export equals the row count shown in the list
- **Preconditions:** A filter is applied that yields 37 customers.
- **Steps:**
  1. Note the result count shown in the UI (37).
  2. Export to CSV.
  3. Count the data rows in the file (excluding the header).
- **Expected result:** The data row count is 37, matching the UI count.

### TC12-05: Export after the filter is cleared returns the full list again
- **Preconditions:** A filter was applied earlier in the session and then removed.
- **Steps:**
  1. Apply a filter, then remove it.
  2. Confirm the list shows all customers.
  3. Export to CSV.
- **Expected result:** The export contains all customers, not only the previously filtered subset.

### TC12-06: Export of all matching rows covers more than one page of the list
- **Preconditions:** 250 customers match the active filter. The list view is paginated at 50 rows per page.
- **Steps:**
  1. Apply the filter. Confirm the list shows 250 results across 5 pages.
  2. Export to CSV while on page 1.
  3. Count the data rows in the file.
- **Expected result:** The file contains all 250 matching rows, not only the 50 on page 1. [OQ-16]

### TC12-07: Export with exactly 10,000 matching rows succeeds (upper boundary, allowed)
- **Preconditions:** Exactly 10,000 customers match the active filter (or no filter, if the total is 10,000).
- **Steps:**
  1. Apply the filter (or leave none active).
  2. Click "Export to CSV".
  3. Wait for the download to complete. Count the data rows.
- **Expected result:** The export succeeds with no error or truncation message. The file has 10,000 data rows plus a header. [OQ-2]

### TC12-08: Export with 9,999 matching rows succeeds (just below the limit)
- **Preconditions:** Exactly 9,999 customers match the active filter.
- **Steps:**
  1. Apply the filter.
  2. Export to CSV. Count the data rows.
- **Expected result:** The export succeeds. The file has 9,999 data rows plus a header.

### TC12-09: Export with exactly 1 matching row (lower boundary)
- **Preconditions:** Exactly 1 customer matches the active filter.
- **Steps:**
  1. Apply a filter that matches one customer.
  2. Export to CSV.
- **Expected result:** The file has a header and exactly 1 data row, which is the matching customer.

### TC12-10: Export with zero matching rows
- **Preconditions:** A filter is applied that matches no customers.
- **Steps:**
  1. Apply the filter. Confirm the list is empty.
  2. Click "Export to CSV".
- **Expected result:** Either a file with only a header row is downloaded, or the user sees a clear "no rows to export" message and no file is created. Behaviour must be consistent and not produce an error. [OQ-7]

---

## Negative scenarios

### TC12-11: Export with 10,001 matching rows is rejected or truncated to the limit
- **Preconditions:** 10,001 customers match the active filter (or no filter, if the total is 10,001).
- **Steps:**
  1. Apply the filter.
  2. Click "Export to CSV".
  3. Observe the result. If a file downloads, open it and count the data rows.
- **Expected result:** The system does not produce a file with more than 10,000 data rows. Either the export is refused with a message that states the 10,000-row limit, or the export is limited to 10,000 rows and the user is clearly told that the file is incomplete. [OQ-1]

### TC12-12: Export with a very large result (for example 50,000 rows) is capped at the limit
- **Preconditions:** 50,000 customers match the filter.
- **Steps:**
  1. Apply the filter.
  2. Click "Export to CSV".
  3. Observe the result and count any rows in the file.
- **Expected result:** The same as TC12-11. No file exceeds 10,000 data rows, and the limit is communicated to the user. [OQ-1]

### TC12-13: User without export permission cannot export
- **Preconditions:** A user who lacks the export permission (e.g. read-only role) is logged in. Customers exist and are visible to the user.
- **Steps:**
  1. Open the customer list.
  2. Check whether the "Export to CSV" control is shown or enabled.
  3. If it is shown, click it.
- **Expected result:** The export control is hidden or disabled. If the user reaches the export action directly (for example through a URL or API call), the request is denied with an authorization error, and no file is produced. [OQ-10]

### TC12-14: Export request with an expired session is refused
- **Preconditions:** The user's session has expired. The customer list page is still open in the browser.
- **Steps:**
  1. Click "Export to CSV".
- **Expected result:** The user is redirected to the login page or prompted to sign in again. No file is downloaded.

### TC12-15: Export fails on a server error and no partial file is delivered
- **Preconditions:** The export service fails partway through (for example a database timeout or a simulated 500 error). Use a test hook or fault injection.
- **Steps:**
  1. Apply a filter that matches about 5,000 rows.
  2. Click "Export to CSV".
  3. Observe the outcome.
- **Expected result:** The user sees an error message that says the export failed and suggests retrying. No corrupt or partially written CSV is saved as a valid file.

### TC12-16: Export is not affected by filters that were changed after the export started
- **Preconditions:** A large export is in progress. Filters are applied through the UI.
- **Steps:**
  1. Apply filter A (e.g. Country = "DE"). Click "Export to CSV".
  2. Before the file is ready, change the filter to B (e.g. Country = "FR").
  3. Open the downloaded file once it completes.
- **Expected result:** The file reflects filter A, the filter active at the moment of the click. It does not reflect filter B. [OQ-12]

### TC12-17: Repeated clicks on "Export to CSV" do not create corrupt or duplicate output
- **Preconditions:** User has export permission. A filter matches about 1,000 rows.
- **Steps:**
  1. Double-click or rapidly click "Export to CSV" several times.
  2. Count the downloaded files and inspect each.
- **Expected result:** Each file is complete and valid, with no duplicate rows inside any single file. Whether duplicate downloads are created is defined by the UI behaviour. The button should be disabled or debounced while an export is running. [OQ-13]

### TC12-18: Export fails for a filter that yields more than the limit, and the error does not expose internal details
- **Preconditions:** 10,001 rows match the filter (see TC12-11).
- **Steps:**
  1. Trigger the export.
  2. Read the error message, if one is shown.
- **Expected result:** The message states the limit in user-facing terms. It does not show stack traces, SQL, or internal identifiers.

---

## CSV format and data integrity

### TC12-19: Values containing commas, double quotes and line breaks are escaped correctly
- **Preconditions:** A customer exists with name `Smith, "Bob" Ltd` and address containing a line break.
- **Steps:**
  1. Filter the list so that this customer is included.
  2. Export to CSV.
  3. Open the file in a CSV-aware parser (for example Python `csv` module or a spreadsheet application).
- **Expected result:** The parsed values match the stored values exactly. The comma, quotes and line break are inside a properly quoted field. Column count is the same on every row.

### TC12-20: Non-ASCII characters are preserved
- **Preconditions:** Customers exist with names such as "Müller", "Øster", "Łukasz" and "李明".
- **Steps:**
  1. Export the full list to CSV.
  2. Open the file in a UTF-8-aware editor and in a spreadsheet application.
- **Expected result:** All characters display correctly with no mojibake. The encoding is as specified. [OQ-6]

### TC12-21: Cells starting with formula characters are neutralised (CSV injection)
- **Preconditions:** A customer exists with a name value `=HYPERLINK("http://example.test","x")` or a value starting with `+`, `-` or `@`.
- **Steps:**
  1. Export the customer list to CSV.
  2. Open the file in a spreadsheet application.
- **Expected result:** The value is not evaluated as a formula. It is stored as text, for example prefixed with an apostrophe or otherwise escaped. [OQ-11]

### TC12-22: Column set in the export matches the defined specification
- **Preconditions:** The customer list shows some columns. The full customer record has more fields.
- **Steps:**
  1. Export to CSV.
  2. Read the header row and compare it with the specification.
- **Expected result:** The header row lists exactly the columns defined for the export, in the defined order. [OQ-5]

### TC12-23: Fields the user is not allowed to see are not exported
- **Preconditions:** Some customer fields (for example a national ID or payment details) are restricted for this user's role.
- **Steps:**
  1. Log in as the restricted role.
  2. Export to CSV.
  3. Read the header row and sample rows.
- **Expected result:** The restricted fields are absent from the file. [OQ-5]

### TC12-24: Date and number formats in the export are consistent and documented
- **Preconditions:** Customers exist with dates and numeric values such as revenue of 1234.5.
- **Steps:**
  1. Export to CSV.
  2. Inspect the date and numeric columns.
- **Expected result:** Dates and numbers use one consistent format on every row, matching the specification (for example ISO 8601 for dates, `.` as decimal separator). [OQ-6]

---

## Boundary and edge scenarios

### TC12-25: Filter that matches all customers with exactly the limit reached by filter combination
- **Preconditions:** Two filters are applied. Their combined result is exactly 10,000 rows, while each filter alone matches more than 10,000.
- **Steps:**
  1. Apply both filters.
  2. Export to CSV.
- **Expected result:** The limit is applied to the combined (AND) result, not to each filter separately. The export succeeds with 10,000 rows. [OQ-3]

### TC12-26: Search text and sort order are reflected in the export, if they are part of the "current list"
- **Preconditions:** A free-text search is active and the list is sorted by Name descending.
- **Steps:**
  1. Apply a search term and sort by Name descending.
  2. Export to CSV.
  3. Read the row order and match.
- **Expected result:** Rows match the search term. Row order matches the on-screen sort. If the spec excludes search or sort, the behaviour is as documented. [OQ-4]

### TC12-27: Export of 10,000 rows completes within the expected time
- **Preconditions:** Exactly 10,000 rows match. Test runs on a staging environment with production-like data volume.
- **Steps:**
  1. Start the export. Measure the time to download.
- **Expected result:** The export completes within the agreed time budget, with no request timeout in the browser or the proxy. [OQ-9, OQ-14]

### TC12-28: Export file name follows the defined convention
- **Preconditions:** None.
- **Steps:**
  1. Export to CSV on two different days.
  2. Read the file names.
- **Expected result:** File names follow the defined pattern (for example `customers_YYYY-MM-DD.csv`). The name is not the same across days if the convention includes a date. [OQ-8]

### TC12-29: Export is recorded in the audit log, if the requirement requires it
- **Preconditions:** Audit logging is enabled.
- **Steps:**
  1. Export to CSV with a filter applied.
  2. Check the audit log.
- **Expected result:** An entry records the user, the time, the filter applied and the row count exported. [OQ-15]

---

## Open questions

These are not answered by the requirement. Each one blocks or changes the expected result of the test cases listed.

- **OQ-1: Behaviour above 10,000 rows.** Should the export be refused with a message, or should it produce a file capped at 10,000 rows with a warning? Affects TC12-11, TC12-12, TC12-18.
- **OQ-2: Is the limit inclusive?** Is exactly 10,000 rows allowed, or must the result be below 10,000? The tests assume 10,000 is allowed. Affects TC12-07.
- **OQ-3: How is the limit counted?** Is it the number of rows after all filters are combined? Assumed yes. Affects TC12-25.
- **OQ-4: What is an "active filter"?** Are filters combined with AND? Do free-text search and sort order count as part of the "current list"? Affects TC12-03, TC12-26.
- **OQ-5: Which columns are exported?** All customer fields, the visible columns only, or a defined export set? Are restricted fields excluded? Affects TC12-22, TC12-23.
- **OQ-6: CSV format details.** Delimiter, quoting rules, header row, line endings, character encoding (UTF-8 with or without BOM), and date and number formats. Affects TC12-19, TC12-20, TC12-24.
- **OQ-7: Empty result.** Should an export with zero rows produce a header-only file, or should the user see a message and no file? Affects TC12-10.
- **OQ-8: File name.** Is there a required naming convention, such as a date or timestamp? Affects TC12-28.
- **OQ-9: Delivery method.** Is the file downloaded immediately, or is an asynchronous path (email, notification, later download) used for large exports? Affects TC12-27.
- **OQ-10: Permissions.** Which roles may export? Does the export respect the same row-level access rules as the list view? Affects TC12-13.
- **OQ-11: CSV injection.** Is sanitising cells that start with `=`, `+`, `-` or `@` required? Affects TC12-21.
- **OQ-12: Data consistency.** Is the export a snapshot at the time of the click? What happens if customer data changes while the export runs? Affects TC12-16.
- **OQ-13: Concurrency and rate limits.** Can a user run several exports at once? Is there a rate limit? Should the button be disabled during an export? Affects TC12-17.
- **OQ-14: Performance target.** What is the maximum acceptable time for a 10,000-row export? Affects TC12-27.
- **OQ-15: Audit logging.** Must each export be logged? Affects TC12-29.
- **OQ-16: Scope of "current list".** Does the export cover all pages of the filtered result, or only the page the user is viewing? The requirement reads as all matching rows. Affects TC12-06.
