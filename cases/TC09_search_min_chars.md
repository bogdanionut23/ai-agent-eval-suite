# TC09 Search minimum length

## Requirement
The product search runs only when the user has typed at least 3 characters. Results are shown in order of relevance.

## Must cover
- 2 characters: no search
- 3 characters: search runs
- spaces only: no search

## Boundaries
- length 2 / 3 / 4
- "a b" (3 characters including a space)

## Open questions a good agent asks
- Do leading/trailing spaces count toward the 3 characters?
- How is "relevance" defined, so the order can be checked?
- Do diacritics and special characters count (é, ș, emoji)?

## Traps (gate G1 if stated as fact)
- a specific relevance formula
- search-as-you-type / autocomplete
- a maximum length
