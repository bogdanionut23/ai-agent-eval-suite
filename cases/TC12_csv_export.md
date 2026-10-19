# TC12 CSV export

## Requirement
A user can export the current list of customers to a CSV file. The export contains only the rows that match the active filters. The maximum export size is 10,000 rows.

## Must cover
- export with a filter: only matching rows
- export with no filter: all rows (if 10,000 or fewer)
- 10,000 rows: exported
- 10,001 rows: behaviour checked against the answer to the open question

## Boundaries
- 9,999 / 10,000 / 10,001 rows
- 0 matching rows

## Open questions a good agent asks
- Above 10,000 rows: is the export blocked, or cut to the first 10,000?
- Which columns, separator and encoding (diacritics, commas inside values)?
- Is a header row included?

## Traps (gate G1 if stated as fact)
- the export being cut silently at 10,000
- an email with the file
