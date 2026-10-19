# TC12 - Test cases: Export customer list to CSV (max 10,000 rows)

## Requirement

A user can export the current list of customers to a CSV file. The export contains only the rows that match the active filters. The maximum export size is 10,000 rows.

## Assumptions and general preconditions

These apply to every test case unless the case says otherwise.

- The user is logged in and has permission to view the customer list (permission rules are an open question, see Q6).
- The customer list contains a known data set that is seeded for the test.
- The user has a browser that can download files.
- Expected results for behaviour the requirement does not define are marked **[OPEN]** and point to the open question. They are not treated as pass/fail criteria until the question is answered.

---

## Positive scenarios

### TC12-01 Export with no filters active exports the full list

- **Preconditions:** Customer list has 250 customers. No filter is active.
- **Steps:**
  1. Open the customer list.
  2. Confirm that no filter is applied.
  3. Click "Export to CSV".
  4. Open the downloaded file.
- **Expected result:** A CSV file downloads. It contains one header row and 250 data rows, one per customer. No customer is missing or duplicated.
- **Note:** Whether "current list" means all pages or only the visible page is **[OPEN]** (Q1).

### TC12-02 Export with a single active filter contains only matching rows

- **Preconditions:** 250 customers, of which 40 have `Country = Germany`.
- **Steps:**
  1. Apply the filter `Country = Germany`.
  2. Confirm the list shows 40 customers.
  3. Click "Export to CSV".
  4. Open the file.
- **Expected result:** The file has 40 data rows, all with `Country = Germany`. No other customer appears.

### TC12-03 Export with several active filters contains only rows matching all of them

- **Preconditions:** Filter A (`Country = Germany`) and filter B (`Status = Active`) are both set. Exactly 12 customers match both.
- **Steps:**
  1. Apply filter A and filter B.
  2. Confirm the on-screen list shows 12 customers.
  3. Export to CSV and open the file.
- **Expected result:** The file has exactly 12 data rows. Each row matches both filters. The row count equals the on-screen count.
- **Note:** The combination logic (AND or OR) is **[OPEN]** (Q4). This case assumes AND, which is the usual reading of "active filters".

### TC12-04 Exported row count equals the on-screen filtered count

- **Preconditions:** Any filter combination that yields N customers, where 1 <= N <= 10,000.
- **Steps:**
  1. Apply the filters and note the count shown in the UI.
  2. Export to CSV.
  3. Count the data rows in the file, excluding the header.
- **Expected result:** Data row count equals N.

### TC12-05 CSV structure is valid

- **Preconditions:** Export of at least 5 customers with the standard fields.
- **Steps:**
  1. Export to CSV.
  2. Open the file in a plain-text editor.
  3. Check the first line, the delimiter, and the line endings.
  4. Open the file in a spreadsheet application.
- **Expected result:** The first line is a header row. Every row has the same number of columns as the header. The file opens in a spreadsheet with one column per field and no shifted values.
- **Note:** Delimiter, encoding, line endings, and the column set and order are **[OPEN]** (Q3, Q5).

### TC12-06 Export respects the active sort order

- **Preconditions:** Customers sorted by `Name` descending.
- **Steps:**
  1. Apply a sort and apply a filter.
  2. Export to CSV.
  3. Read the row order in the file.
- **Expected result:** Row order in the file matches the on-screen order.
- **Note:** Whether sort is carried into the export is **[OPEN]** (Q4).

### TC12-07 Export of exactly 10,000 matching rows succeeds

- **Preconditions:** Filter matches exactly 10,000 customers.
- **Steps:**
  1. Apply the filter.
  2. Confirm the count is 10,000.
  3. Click "Export to CSV".
  4. Count the data rows in the file.
- **Expected result:** Export completes. The file has exactly 10,000 data rows and no error is shown.

---

## Boundary scenarios

### TC12-08 Export of 9,999 matching rows

- **Preconditions:** Filter matches 9,999 customers.
- **Steps:** Apply the filter, export, count the data rows.
- **Expected result:** File has 9,999 data rows. No warning or error.

### TC12-09 Export of 10,001 matching rows

- **Preconditions:** Filter matches 10,001 customers.
- **Steps:** Apply the filter, click "Export to CSV".
- **Expected result:** **[OPEN]** (Q2). The requirement sets a maximum of 10,000 but does not say what happens above it. The test must not pass on silent truncation. At minimum:
  - The user is told that the limit was reached, or the export is blocked with a message. No file is silently cut to 10,000 rows without notice.
  - The message tells the user how to reduce the result (for example, narrow the filters).

### TC12-10 Export of a single matching row

- **Preconditions:** Filter matches exactly 1 customer.
- **Steps:** Apply the filter, export, open the file.
- **Expected result:** File has the header and exactly 1 data row.

### TC12-11 Export when no rows match the filters

- **Preconditions:** Filter matches 0 customers.
- **Steps:** Apply the filter, click "Export to CSV".
- **Expected result:** **[OPEN]** (Q5). Either a file with only the header row is downloaded, or the export is disabled or shows a message. A file with no header, or a corrupted file, is a failure.

### TC12-12 Maximum field length and multi-line values

- **Preconditions:** One customer has a 2,000-character note field, and another has a note containing a line break.
- **Steps:** Export to CSV. Open the file in a spreadsheet and in a text editor.
- **Expected result:** The long value is exported in full. The multi-line value stays inside one CSV cell (quoted), and the row count is not inflated by the line break.

### TC12-13 Field values with delimiter, quote and newline characters

- **Preconditions:** Customer name `Smith, "Jo" Ltd`. Address contains a comma.
- **Steps:** Export, open the file in a spreadsheet.
- **Expected result:** Each value is in its own column. Commas and double quotes are escaped per the CSV standard (RFC 4180 or the agreed format). No values shift to the next column.

### TC12-14 Empty and null fields

- **Preconditions:** Some customers have no phone number or no email.
- **Steps:** Export and open the file.
- **Expected result:** Empty fields appear as empty CSV cells. The text "null", "undefined" or "NaN" does not appear unless it is stored data.

### TC12-15 Non-ASCII characters

- **Preconditions:** Customers with names containing "Müller", "Łukasz", "Søren" and CJK characters.
- **Steps:** Export, open the file in a spreadsheet application.
- **Expected result:** Characters display correctly with no mojibake. Encoding is as specified (Q5).

### TC12-16 Filtered export does not include rows that are excluded by the filter, even at the boundary

- **Preconditions:** Filter is `Created date >= 2026-01-01`. One customer was created exactly on 2026-01-01 and one on 2025-12-31 23:59.
- **Steps:** Apply the filter, export.
- **Expected result:** The 2026-01-01 customer is included. The 2025-12-31 customer is not included. The boundary matches the on-screen behaviour.

---

## Negative scenarios

### TC12-17 Export button with a filter that causes an empty list while data exists

- **Preconditions:** Filter set to a value that matches no customers.
- **Steps:** Click "Export to CSV".
- **Expected result:** Same as TC12-11. No row from outside the filter appears in the file.

### TC12-18 Export by a user without permission

- **Preconditions:** User account without the customer export permission.
- **Steps:** Open the customer list. Try to export.
- **Expected result:** **[OPEN]** (Q6). If the permission exists, the export is hidden or disabled, and a direct request is denied. No customer data is downloaded.

### TC12-19 Server error during export

- **Preconditions:** Simulate a 500 error from the export endpoint, for example with a network mock.
- **Steps:** Click "Export to CSV".
- **Expected result:** The user sees a clear error message. No partial or zero-byte file is saved as if it were complete. The UI stays usable and the user can retry.

### TC12-20 Session expires during export

- **Preconditions:** Session token expires while the export is being prepared.
- **Steps:** Start the export. Let the session expire before the file is ready.
- **Expected result:** The export does not return data after the session is invalid. The user is asked to log in again. No data is written to the file.

### TC12-21 Network interruption during download

- **Preconditions:** Large export (around 10,000 rows).
- **Steps:** Start the export. Drop the network connection mid-download.
- **Expected result:** The browser reports a failed download. The user can retry. A truncated file is not presented as complete.

### TC12-22 Formula injection in exported cells

- **Preconditions:** A customer name starts with `=HYPERLINK("http://example.test","x")` or `=1+1`. Another starts with `+`, `-` or `@`.
- **Steps:** Export, open the file in a spreadsheet application.
- **Expected result:** **[OPEN]** (Q9). If sanitisation is required, the cell is shown as text and the formula does not run. The cell value is not executed when the file is opened.

### TC12-23 Filter applied during export but before the file is ready

- **Preconditions:** The export is requested with filter A. The user changes the filter to B before the file is prepared.
- **Steps:** Click export with filter A, then change the filter to B while the export is still running.
- **Expected result:** **[OPEN]** (Q4, Q10). The file contains only the rows for the filter that was active when the export was clicked. Rows for filter B do not appear.

### TC12-24 Customer data changes while the export runs

- **Preconditions:** Export is in progress. Another user edits or deletes a customer in the filtered set.
- **Steps:** Start the export. Edit one customer in another session during the export.
- **Expected result:** **[OPEN]** (Q12). The file is internally consistent: each row is either the old or the new version, and no row appears twice or is lost. A snapshot is expected.

---

## Performance and load (boundary)

### TC12-25 Export of 10,000 rows completes within the agreed time

- **Preconditions:** Data set with at least 10,000 matching customers.
- **Steps:** Export. Measure the time from click to download start.
- **Expected result:** Completes within the time limit. **[OPEN]** (Q10): no time limit is stated in the requirement. Use the agreed limit when it is known.

### TC12-26 Repeated exports are independent

- **Preconditions:** Export has run once with filter A.
- **Steps:** Change the filter to B. Export again. Compare both files.
- **Expected result:** Each file contains only the rows for its own filter. The first file is unchanged on disk.

---

## Open questions

These need an answer before the tests above can be finalised. Behaviour is not invented for them.

1. **Scope of "current list":** Does it mean all rows that match the filters across all pages, or only the rows on the visible page?
2. **Limit behaviour:** When the matches exceed 10,000, should the export be blocked, truncated with a warning, or should the user be asked to narrow the filters? Does the 10,000 limit count data rows only, excluding the header?
3. **Columns:** Which fields are included? All fields, or only the visible columns? In what order, and with what header labels?
4. **Filter and sort semantics:** Are multiple filters combined with AND or OR? Is the current sort order applied to the export? If a filter changes during an export, which filter applies?
5. **Empty results and file format:** What is downloaded when zero rows match? What are the delimiter, encoding (UTF-8 with or without BOM), line endings, date and number formats, and quoting rules?
6. **Permissions and audit:** Which roles can export? Is the export logged for audit? Are any fields (for example personal data) excluded for some roles?
7. **File name:** What is the file name pattern, and does it include a timestamp or filter description?
8. **Delivery mode:** Is the export a direct download, or is it asynchronous (for example, an emailed link or a notification) for large results?
9. **Security:** Is CSV formula injection (cells starting with `=`, `+`, `-`, `@`) to be neutralised?
10. **Performance:** What is the maximum acceptable time to produce a 10,000-row export? Is there a timeout?
11. **Deleted and inactive customers:** Are soft-deleted or inactive customers included when the filters match them?
12. **Consistency:** Is the export a snapshot at the moment of the click, or can it reflect edits made during generation?
13. **Localisation:** Are dates, numbers and currency formatted by locale, or in a fixed format?
14. **Error messages:** What exact text should be shown for the limit, the empty result, and server failure?
