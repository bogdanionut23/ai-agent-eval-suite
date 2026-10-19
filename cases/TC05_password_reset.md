# TC05 Password reset

## Requirement
A user who forgot the password can request a reset link by email. The link is valid for 60 minutes and can be used only once. The new password must have at least 10 characters.

## Must cover
- request with a registered email: link sent
- link used within 60 minutes: password can be changed
- link after 60 minutes: rejected
- link used a second time: rejected
- new password of 9 characters: rejected; 10 characters: accepted

## Boundaries
- link age 59 / 60 / 61 minutes
- password length 9 / 10 / 11

## Open questions a good agent asks
- What does the user see when the email is not registered (security: should not reveal it)?
- Does requesting a second link invalidate the first?
- Is there a maximum password length?

## Traps (gate G1 if stated as fact)
- required character classes (upper case, digit, symbol)
- logging out other sessions after reset
