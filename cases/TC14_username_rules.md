# TC14 Username rules

## Requirement
A username must be 4 to 20 characters long and may contain only letters, digits and underscores. Usernames must be unique.

## Must cover
- 4 and 20 characters: accepted
- 3 and 21 characters: rejected
- characters outside the allowed set (space, dash, @): rejected
- username already taken: rejected

## Boundaries
- length 3 / 4 / 20 / 21

## Open questions a good agent asks
- Is uniqueness case-insensitive ("Bogdan" vs "bogdan")?
- Do letters with diacritics or non-Latin letters count as "letters"?
- Can a username start with a digit or an underscore?

## Traps (gate G1 if stated as fact)
- reserved words (admin, root)
- usernames being changeable later
