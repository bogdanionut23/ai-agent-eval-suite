# Test Cases: Profile Picture Upload

Requirement: Users can upload a profile picture. Allowed formats are JPG and PNG. The maximum file size is 5 MB. After upload the picture is shown on the profile page.

Assumptions (minimal, to make tests executable): the user is logged in and has access to a profile page with an upload control. "5 MB" is treated as a boundary to be confirmed (see Open Questions 1). Where the expected result depends on an unresolved question, the test is marked **[Blocked by OQ#]** and the expected result is what must be confirmed, not invented.

## Positive

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-PP-001 | Upload valid JPG | Logged-in user; a valid .jpg of about 1 MB | 1. Open profile/upload control. 2. Select the .jpg. 3. Confirm/submit upload. 4. Open the profile page. | Upload succeeds; success feedback shown; the picture is displayed on the profile page. |
| TC-PP-002 | Upload valid PNG | Logged-in user; a valid .png of about 1 MB | Same as TC-PP-001 with the .png | Upload succeeds; the picture is displayed on the profile page. |
| TC-PP-003 | Uppercase extension accepted | Valid image named photo.JPG and another named photo.PNG | Upload each in turn | Both accepted and displayed (extension check is case-insensitive). **[OQ 3]** |
| TC-PP-004 | .jpeg extension | Valid JPEG file named photo.jpeg | Upload it | Accepted and displayed, if .jpeg counts as JPG. **[Blocked by OQ 3]** |
| TC-PP-005 | Picture persists | TC-PP-001 completed | 1. Log out. 2. Log in again. 3. Open profile page. | The uploaded picture is still displayed. |
| TC-PP-006 | Replace existing picture | User already has a picture | 1. Upload a different valid image. 2. Open profile page. | New picture displayed; old one no longer shown. **[OQ 6]** |
| TC-PP-007 | Picture visible after reload/other browser | TC-PP-001 completed | Reload page; open in another supported browser/session | Picture displayed consistently. |

## Boundary (file size)

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-BD-001 | File just under limit | Valid JPG of 4.99 MB (5,242,000 bytes if MB = 1024 KB) | Upload | Accepted and displayed. |
| TC-BD-002 | File exactly at limit | Valid PNG of exactly 5 MB (5,242,880 bytes; also test 5,000,000 bytes) | Upload | Accepted and displayed, if "maximum 5 MB" is inclusive. **[Blocked by OQ 1]** |
| TC-BD-003 | File 1 byte over limit | Valid JPG of 5,242,881 bytes | Upload | Rejected with a clear size error; picture not changed. |
| TC-BD-004 | File clearly over limit | Valid PNG of 6 MB and one of 50 MB | Upload each | Rejected with a size error; no timeout or crash; existing picture unchanged. |
| TC-BD-005 | Smallest valid file | Valid 1x1 px PNG (well under 1 KB) | Upload | Accepted, or rejected with a documented minimum rule. **[OQ 4, 5]** |
| TC-BD-006 | Zero-byte file | Empty file named empty.jpg | Upload | Rejected with a clear error; nothing stored. |
| TC-BD-007 | Large dimensions, small size | Valid PNG 10000x10000 px under 5 MB | Upload | Behaviour per dimension rules. **[Blocked by OQ 4]** |

## Negative

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-NG-001 | Unsupported format GIF | A valid .gif | Upload | Rejected with a message naming allowed formats (JPG, PNG); picture unchanged. |
| TC-NG-002 | Unsupported format others | Valid .bmp, .webp, .svg, .tiff, .heic, .pdf files | Upload each | Each rejected with a format error. (WebP/HEIC are common; confirm they must be rejected, OQ 3.) |
| TC-NG-003 | Renamed non-image | A .exe or .txt renamed to photo.jpg | Upload | Rejected: content validated, not only extension. No file stored or executed. |
| TC-NG-004 | Wrong extension, valid content | A real PNG renamed photo.jpg (and vice versa) | Upload | Defined behaviour (accept based on content, or reject). **[Blocked by OQ 3]** |
| TC-NG-005 | No extension | Valid PNG named "photo" with no extension | Upload | Behaviour per rules (reject or accept by content). **[OQ 3]** |
| TC-NG-006 | Double extension | File named photo.jpg.exe and photo.php.png (with non-image content) | Upload | Rejected; not stored or served as executable. |
| TC-NG-007 | Corrupt image | Truncated .jpg / .png with valid header | Upload | Rejected with an error, or handled gracefully; no broken image shown on profile page. |
| TC-NG-008 | Cancel file selection | Upload dialog open | Cancel the file chooser | No change; no error; existing picture unchanged. |
| TC-NG-009 | Submit with no file | Upload control available | Click upload without choosing a file | Validation message; no request creating an empty picture. |
| TC-NG-010 | Unauthenticated upload | User not logged in / session expired | Attempt upload (UI or direct API call) | Rejected (redirect to login or 401/403); nothing stored. |
| TC-NG-011 | Upload to another user's profile | Logged in as user A | Call upload endpoint targeting user B's ID | Rejected (403); B's picture unchanged. |
| TC-NG-012 | Network failure mid-upload | Valid file; throttle/disconnect network during upload | Start upload, cut connection | Clear error; no partial/corrupt picture; previous picture unchanged; retry possible. |
| TC-NG-013 | Server-side size enforcement | Valid 6 MB file | Send directly to API bypassing client checks | Rejected server-side (e.g., 413/400). |
| TC-NG-014 | Server-side type enforcement | GIF content with image/jpeg header | Send directly to API | Rejected server-side. |
| TC-NG-015 | Special characters in filename | Valid image named `ü ñ 日本 \ ' " ; <script>.png` | Upload | Accepted or cleanly rejected; no XSS or path traversal; name handled safely. |
| TC-NG-016 | Malicious content | Image with embedded script/metadata payload | Upload, view profile page | Not executed; content sanitised or rejected. |
| TC-NG-017 | Double submit / concurrent uploads | Valid file | Click upload twice quickly; or upload from two tabs | One consistent final picture; no duplicates or errors. |

## Display / UX

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-DP-001 | Display on profile page | TC-PP-001 done | Open profile page | Uploaded picture shown in the profile picture area. |
| TC-DP-002 | Non-square / extreme aspect ratio | Valid wide (4000x200) and tall (200x4000) images | Upload each | Displayed per the defined crop/scale rule; not distorted unexpectedly. **[OQ 4]** |
| TC-DP-003 | Transparent PNG | PNG with alpha channel | Upload | Transparency shown correctly (or background defined). |
| TC-DP-004 | EXIF orientation | JPG with EXIF rotation tag | Upload | Displayed in the correct orientation. |
| TC-DP-005 | No picture uploaded yet | New user | Open profile page | Defined placeholder shown (no broken image). **[OQ 7]** |
| TC-DP-006 | Upload feedback | Any valid upload | Observe UI | Progress/success indicator shown; picture updates without stale cache. |
| TC-DP-007 | Error message quality | Trigger format and size errors | Observe messages | Messages state which rule failed and the allowed values. |
| TC-DP-008 | Responsive and accessibility | Profile page with picture | View on mobile/tablet/desktop; use screen reader and keyboard | Picture scales correctly; upload control keyboard-operable; image has alt text. |
| TC-DP-009 | Browser compatibility | Valid file | Upload in supported browsers (Chrome, Firefox, Safari, Edge; mobile) | Same behaviour in all. |

## Open Questions (requirement ambiguous or incomplete)

1. **Size limit definition**: Is 5 MB 5,000,000 bytes or 5,242,880 bytes (MiB)? Is the limit inclusive (exactly 5 MB allowed)?
2. **Where is the picture shown?** "The profile page" only: or also in headers, comments, lists, other users' views? Who can see it (public, friends, private)?
3. **Format rules**: Is .jpeg (and .jpe) accepted as JPG? Is validation by extension, MIME type or file content? What happens when extension and content disagree? Case sensitivity of extensions? Are WebP, HEIC, GIF explicitly rejected?
4. **Dimensions**: Minimum/maximum pixel dimensions? Required aspect ratio? Is the image cropped, resized or scaled on display? Is the original kept?
5. **Minimum file size** and handling of empty or tiny files.
6. **Replace / remove**: Can the user replace or delete the picture? Is the old file deleted? Is there a default avatar after removal?
7. **No-picture state**: What is displayed before any upload (placeholder/initials)?
8. **Error handling**: Exact messages and behaviour on rejection; is the previous picture retained?
9. **Upload mechanics**: Is there a preview or crop step before confirming? Any progress indicator? Timeout limits? Drag and drop support? Multiple files selected at once?
10. **Timing**: "After upload" means immediately visible, or after processing/moderation? Acceptable latency?
11. **Content moderation / security**: Is virus scanning, EXIF stripping (privacy, GPS data) or inappropriate-content review required?
12. **Permissions**: Can admins upload on behalf of users? Any rate limit on uploads?
13. **Platforms**: Web only, or also mobile apps/API? Supported browsers and devices?
14. **Animated or multi-frame images**, progressive JPEG, CMYK JPEG, 16-bit PNG: supported?
15. **Non-functional**: Storage location, retention, CDN caching and cache invalidation after replacement; localisation of messages.
