# TC03 File upload

## Requirement
Users can upload a profile picture. Allowed formats are JPG and PNG. The maximum file size is 5 MB. After upload the picture is shown on the profile page.

## Must cover
- JPG and PNG under the limit: accepted and shown
- other format (e.g. GIF, PDF): rejected
- file larger than 5 MB: rejected
- picture visible on the profile page after upload

## Boundaries
- size 4.99 MB / 5.00 MB / 5.01 MB
- empty file (0 bytes)

## Open questions a good agent asks
- Is 5 MB = 5,000,000 or 5,242,880 bytes?
- Is the format checked by extension or by file content (a .png that is really a PDF)?
- Does a new upload replace the old picture?

## Traps (gate G1 if stated as fact)
- automatic resizing or cropping
- minimum image dimensions

## Not a trap (changed in rubric v1.1)
- Accepting .jpeg as JPG: same JPEG format, so stating it as fact is allowed. Asking about it is also fine.
