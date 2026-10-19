# TC12 - Test cases: CSV export of filtered customer list

**Requirement:** A user can export the current list of customers to a CSV file. The export contains only the rows that match the active filters. The maximum export size is 10,000 rows.

---

## Assumptions (used only to make the test cases runnable)

These are not stated in the requirement. Each one is listed again in the open questions and should be confirmed before the cases are final.

- The export is started from the customer list screen, using a button or menu item labelled "Export" (or similar).
- The file is downloaded to the user's device, and the browser or app then saves it.
- The CSV has a header row, comma delimiter, UTF-8 encoding, and one row per customer.
- The "active filters" are the filters applied to the list at the moment the export is started.

---

## Positive scenarios

### TC-EXP-001 - Export with no filters active exports the full customer list
**Preconditions:** User is logged in and has export permission. The system holds 250 customers. No filters are active on the customer list.
**Steps:**
1. Open the customer list.
2. Confirm the list shows all 250 customers (or the first page with a total count of 250).
3. Click "Export".
4. Open the downloaded CSV file.

**Expected result:** A CSV file is downloaded. It contains a header row and exactly 250 data rows, one for each customer. No customer is missing and none is duplicated.

### TC-EXP-002 - Export with one filter active contains only matching rows
**Preconditions:** 250 customers exist, of which 40 have status "Active".
**Steps:**
1. Open the customer list.
2. Apply the filter Status = "Active". Confirm the list shows 40 rows.
3. Click "Export".
4. Open the CSV file.

**Expected result:** The CSV contains a header and exactly 40 data rows. Every row has Status = "Active". No inactive customers appear.

### TC-EXP-003 - Export with several filters combined (AND) contains only rows matching all of them
**Preconditions:** Customers exist with varied Country and Status values. Filter A: Country = "DE". Filter B: Status = "Active". Assume 12 customers match both.
**Steps:**
1. Apply Country = "DE" and Status = "Active" to the list. Confirm 12 rows are shown.
2. Click "Export".
3. Open the CSV file.

**Expected result:** The CSV contains exactly 12 data rows. Each row has Country = "DE" and Status = "Active". Customers that match only one of the two filters are not present.

### TC-EXP-004 - Export order matches the order of the on-screen list
**Preconditions:** Filter applied. The list is sorted by "Last name" descending.
**Steps:**
1. Apply a filter and sort the list by Last name descending.
2. Click "Export".
3. Compare the order of rows in the CSV with the order on screen.

**Expected result:** The rows in the CSV appear in the same order as on screen. (This case depends on open question Q4. Mark it as skipped if sort order is not part of the requirement.)

### TC-EXP-005 - Exported file name and extension
**Preconditions:** Any filter state.
**Steps:**
1. Click "Export".
2. Check the file name and extension of the downloaded file.

**Expected result:** The file has a .csv extension. The name follows the agreed convention (for example customers_YYYY-MM-DD.csv). (Naming convention is open question Q6.)

### TC-EXP-006 - Export after a filter change reflects the new filter, not the previous one
**Preconditions:** 100 customers exist. 30 match filter X and 70 match filter Y.
**Steps:**
1. Apply filter X. Export. Save the file as export1.csv.
2. Change the filter to Y. Export again. Save the file as export2.csv.

**Expected result:** export1.csv has 30 rows, all matching X. export2.csv has 70 rows, all matching Y. Neither file contains rows from the other filter set.

### TC-EXP-007 - Exported columns match the columns in the list view (or the agreed export column set)
**Preconditions:** Customer list displays a known set of columns.
**Steps:**
1. Export the list with any filter.
2. Open the CSV and compare the header row with the agreed column set.

**Expected result:** The header row contains the agreed columns in the agreed order. Values are present for each customer in every column that is not empty in the system. (Column set is open question Q3.)

### TC-EXP-008 - Special characters are exported correctly
**Preconditions:** A customer exists with name "Müller, Jörg", a company "Smith & Sons \"Trading\"", and a note containing a line break.
**Steps:**
1. Filter the list so that this customer is shown.
2. Export.
3. Open the CSV in a tool that reads UTF-8 and in a CSV parser.

**Expected result:** The name keeps its umlaut. The comma inside the name, the quotes, and the line break are wrapped and escaped per RFC 4180. The file parses into the same number of columns on every row, and the values match the stored values.

---

## Negative scenarios

### TC-EXP-009 - Export with a filter that matches no customers
**Preconditions:** A filter is applied that matches 0 customers.
**Steps:**
1. Apply a filter that returns an empty list.
2. Click "Export".

**Expected result:** The system does one of two defined things: (a) it downloads a CSV with the header row only and no data rows, or (b) it shows a message such as "Nothing to export" and does not download a file. The behaviour must match the decision from open question Q5. The system must not crash or produce a corrupt file.

### TC-EXP-010 - Export is blocked for a user without export permission
**Preconditions:** User is logged in with a role that does not have export permission. The customer list is visible to this user.
**Steps:**
1. Open the customer list.
2. Check whether an "Export" control is visible.
3. If it is visible, click it.
4. If the control is hidden, call the export endpoint directly with the same filters (if the API is reachable).

**Expected result:** The control is hidden. If it is reached through the API, the request is rejected with an authorization error (for example HTTP 403). No file is produced and no customer data is returned.

### TC-EXP-011 - Export is not possible without a logged-in session
**Preconditions:** User session has expired.
**Steps:**
1. Attempt to trigger the export (through the UI or through the export URL copied from an earlier session).

**Expected result:** The user is redirected to login or receives an authentication error. No customer data is included in any response.

### TC-EXP-012 - Export is rejected when the matching row count exceeds 10,000
**Preconditions:** 12,000 customers exist. No filters are applied, or a filter matches 12,000 customers.
**Steps:**
1. Confirm the list shows a total of 12,000 matching rows.
2. Click "Export".

**Expected result:** The export does not produce a file with 12,000 rows. The user is shown a clear message that the export is limited to 10,000 rows and asked to narrow the filters. (The exact handling is open question Q2. The test asserts that the output is never more than 10,000 rows.)

### TC-EXP-013 - Export does not include customers hidden by a permission or scope rule
**Preconditions:** User can see only customers of region "EU". The system has customers in "US" and "EU". No filter is applied.
**Steps:**
1. Open the list as this user and count the visible rows.
2. Export.
3. Open the CSV.

**Expected result:** The CSV contains only EU customers, and the number of rows equals the number visible on screen. Customers outside the user's scope do not appear, even though no filter excludes them in the UI.

### TC-EXP-014 - Filter value tampered in the request is handled safely
**Preconditions:** The export request is sent from the UI with filters encoded in the request.
**Steps:**
1. Intercept the export request.
2. Replace a filter value with an invalid value (for example status = "<script>" or a value of the wrong type).
3. Send the request.

**Expected result:** The request is rejected with a validation error (HTTP 400 or equivalent), or the invalid filter is ignored in a defined way. The response does not include unfiltered data, and the server does not error with a stack trace.

### TC-EXP-015 - CSV formula injection is neutralised
**Preconditions:** A customer exists whose name or note starts with "=", "+", "-", or "@" (for example "=HYPERLINK(...)" ).
**Steps:**
1. Filter the list so that this customer is included.
2. Export.
3. Open the CSV in a spreadsheet application.

**Expected result:** The cell is not evaluated as a formula. The value is escaped (for example prefixed with an apostrophe or a tab), in line with the agreed security rule. (Whether this is required is open question Q9.)

### TC-EXP-016 - Export fails with a clear message when the download or generation fails
**Preconditions:** The export service is made to fail (for example a simulated timeout or a 500 error from the backend).
**Steps:**
1. Trigger an export with a valid filter.
2. Observe the UI.

**Expected result:** The user sees an error message that says the export failed and can be retried. No partial or empty CSV file is saved as if it were complete. The UI does not stay in a loading state indefinitely.

---

## Boundary scenarios

### TC-EXP-017 - Export at exactly 10,000 rows is accepted
**Preconditions:** Exactly 10,000 customers match the active filter.
**Steps:**
1. Confirm the list shows 10,000 matching rows.
2. Click "Export".
3. Open the CSV and count the data rows.

**Expected result:** The export succeeds. The CSV contains exactly 10,000 data rows plus one header row. No row is dropped.

### TC-EXP-018 - Export at 10,001 rows is rejected
**Preconditions:** Exactly 10,001 customers match the active filter.
**Steps:**
1. Click "Export".

**Expected result:** The export is not produced as a complete file of 10,001 rows. The user is told the limit is 10,000 rows. (Same handling as TC-EXP-012; the one-row difference checks the off-by-one at the limit.)

### TC-EXP-019 - Export at 9,999 rows contains all rows
**Preconditions:** Exactly 9,999 customers match the active filter.
**Steps:**
1. Click "Export".
2. Count the data rows in the CSV.

**Expected result:** The CSV contains 9,999 data rows. The export is complete.

### TC-EXP-020 - Export of one row (minimum non-empty result)
**Preconditions:** Exactly one customer matches the active filter.
**Steps:**
1. Click "Export".
2. Open the CSV.

**Expected result:** The CSV contains the header and exactly one data row, which is the matching customer.

### TC-EXP-021 - Export of an empty customer database with no filters
**Preconditions:** The system holds 0 customers. No filters applied.
**Steps:**
1. Click "Export".

**Expected result:** Same result as TC-EXP-009 (the empty-result behaviour defined by Q5 applies).

### TC-EXP-022 - Filter applied to a field with a value at the boundary of a range filter
**Preconditions:** Customers exist with created_date values of 2026-01-01, 2026-10-18 and 2026-10-19. Filter: created_date between 2026-01-01 and 2026-10-19 (inclusive, as shown in the UI).
**Steps:**
1. Apply the date filter.
2. Export.

**Expected result:** The CSV includes the customers dated on both boundary dates (2026-01-01 and 2026-10-19). Whether the end date is inclusive must match the UI. (Date boundary semantics are open question Q7.)

### TC-EXP-023 - Maximum file size or long text fields
**Preconditions:** 10,000 customers exist, and each has a note field of 2,000 characters.
**Steps:**
1. Export with a filter that returns all 10,000 rows.
2. Check that the download completes and that the file opens.

**Expected result:** The download completes within the agreed time limit. The file is complete, and the row count is correct. No field is truncated unless truncation is part of the documented behaviour.

### TC-EXP-024 - Export is not allowed to run twice at the same time from one screen (double click)
**Preconditions:** Filter applied, 100 matching rows.
**Steps:**
1. Double-click "Export" quickly.

**Expected result:** Either one file is downloaded, or two identical files are downloaded. No file is corrupt, and the UI does not show a duplicate or inconsistent state.

### TC-EXP-025 - Export reflects the data at the time of the click
**Preconditions:** A filter is applied. One customer matches the filter.
**Steps:**
1. Apply the filter.
2. In a second session, add a second customer that matches the filter, but do not refresh the first session.
3. In the first session, click "Export".

**Expected result:** The export includes the rows that match the filter at the time of the export request (defined behaviour per Q8). The file is consistent with the request time and is not a mix of the two states.

---

## Open questions

These points are not defined in the requirement. The test cases above make the assumptions listed at the top. Nothing below has been decided by me.

- **Q1. What is the export format in full?** Is it RFC 4180 CSV with a comma delimiter? Is the encoding UTF-8, and is a byte-order mark (BOM) required for Excel? Are dates and numbers formatted in a particular locale?
- **Q2. What happens when the matching rows exceed 10,000?** Is the export blocked with a message, or is it truncated to the first 10,000 rows with a warning? In the truncated case, which rows are kept (sort order)?
- **Q3. Which columns are exported?** Are these the columns shown in the list, or a fixed export set? Are hidden or sensitive fields (for example personal data, internal IDs) included? Is column order defined?
- **Q4. Does the export follow the current sort order of the list?** Or is it in a fixed order?
- **Q5. What is the output when no rows match?** A header-only CSV, a "nothing to export" message, or no file at all?
- **Q6. What is the file name convention?** Does it include the date, the user, or the filter?
- **Q7. How are range filters treated at the boundaries?** Are date and number ranges inclusive or exclusive? Are time zones used for date filters?
- **Q8. Is the export a snapshot at the moment of the request, or is it generated later?** If it is generated asynchronously, how does the user receive it (email, download link, expiry time)?
- **Q9. Is CSV formula injection a requirement?** Should values starting with =, +, -, or @ be escaped?
- **Q10. Who may export?** Is export restricted by role? Does the user's data scope (for example region or team) apply to the export?
- **Q11. Is the export audited?** Should each export be logged (user, time, filters, row count)?
- **Q12. What are the performance and timeout requirements?** Is there a maximum time for an export of 10,000 rows?
- **Q13. Does "current list" include archived, deleted or soft-deleted customers?** Is there a filter that shows them?
- **Q14. What is the behaviour for an export that is still running when the user changes the filter or logs out?**
- **Q15. Is there a limit on how many exports a user may run in a given time period?**
