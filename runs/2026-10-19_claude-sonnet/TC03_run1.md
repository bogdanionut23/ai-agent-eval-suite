# Test Cases: Profile Picture Upload

Requirement: Users can upload a profile picture. Allowed formats are JPG and PNG. The maximum file size is 5 MB. After upload the picture is shown on the profile page.

Assumptions used in the cases below (each is also listed as an open question):
- 5 MB is treated as 5 x 1024 x 1024 = 5,242,880 bytes. The boundary cases at that limit depend on the answer to OQ-1.
- "Maximum 5 MB" is read as "a file of exactly 5 MB is accepted". This depends on the answer to OQ-2.
- No expected result invents UI wording. Where the exact message is unspecified, the expected result says "an error message is shown", and the text is a question.

Common preconditions (P0): the user has an account, is logged in, and is on the profile page or the profile edit page, where the upload control is available. Test files are available locally.

## 1. Positive scenarios

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-PP-001 | Upload a valid JPG | P0. File `photo.jpg`, about 1 MB, valid JPEG. | 1. Open the upload control. 2. Select `photo.jpg`. 3. Confirm or submit the upload. 4. Go to the profile page. | The upload succeeds with no error. The picture is shown on the profile page. |
| TC-PP-002 | Upload a valid PNG | P0. File `photo.png`, about 1 MB, valid PNG. | Same as TC-PP-001 with `photo.png`. | The upload succeeds. The picture is shown on the profile page. |
| TC-PP-003 | Upload a `.jpeg` extension file | P0. Valid JPEG file named `photo.jpeg`. | Upload `photo.jpeg`. | Accepted, because JPEG and JPG are the same format (see OQ-3). The picture is shown. |
| TC-PP-004 | Picture persists after reload and re-login | P0. TC-PP-001 passed. | 1. Reload the profile page. 2. Log out and log in again. 3. Open the profile page. | The uploaded picture is still shown. |
| TC-PP-005 | Upload from a profile with no existing picture | P0. The user has no profile picture yet. | Upload a valid JPG. | The picture is shown in place of the default or placeholder avatar. |
| TC-PP-006 | Replace an existing picture | P0. The user already has picture A. | Upload a different valid picture B. | Picture B is shown on the profile page. Picture A is no longer shown (see OQ-9 for what happens to A in storage). |
| TC-PP-007 | Upload a small file | P0. Valid JPG of 10 KB. | Upload the file. | Accepted and shown. |
| TC-PP-008 | Uppercase extension | P0. Valid JPEG named `PHOTO.JPG`, and a valid PNG named `PHOTO.PNG`. | Upload each in turn. | Both are accepted (extension check is case-insensitive) and shown. |

## 2. Boundary scenarios (file size)

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-PP-010 | File exactly at the limit (5,242,880 bytes), JPG | P0. Valid JPG of exactly 5,242,880 bytes. | Upload the file. | Accepted and shown (assumes the limit is inclusive, OQ-2). |
| TC-PP-011 | File exactly at the limit, PNG | P0. Valid PNG of exactly 5,242,880 bytes. | Upload the file. | Accepted and shown (same assumption). |
| TC-PP-012 | File one byte under the limit | P0. Valid JPG of 5,242,879 bytes. | Upload the file. | Accepted and shown. |
| TC-PP-013 | File one byte over the limit | P0. Valid JPG of 5,242,881 bytes. | Upload the file. | Rejected with an error message about size. No picture is changed on the profile. |
| TC-PP-014 | File clearly over the limit | P0. Valid PNG of 6 MB. | Upload the file. | Rejected with an error message. Existing picture (if any) is unchanged. |
| TC-PP-015 | Decimal 5 MB boundary (5,000,000 bytes) | P0. Valid JPG of 5,000,000 bytes. | Upload the file. | Accepted, since it is below 5 MiB. If the product defines 5 MB as 5,000,000 bytes, it is accepted at exactly the limit as well (OQ-1). |
| TC-PP-016 | Very large file | P0. Valid JPG of about 100 MB. | Upload the file. | Rejected with an error message. The system stays responsive, with no crash or timeout page. |
| TC-PP-017 | Minimum file size, smallest valid image | P0. Valid 1x1 pixel PNG (under 1 KB). | Upload the file. | Accepted, or rejected with a clear message if a minimum dimension rule exists (OQ-6). Record the actual behaviour. |

## 3. Negative scenarios

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-PP-020 | Unsupported format: GIF | P0. Valid `anim.gif`. | Upload the file. | Rejected with an error message about the file type. No change to the profile. |
| TC-PP-021 | Unsupported format: BMP | P0. Valid `image.bmp`. | Upload the file. | Rejected with an error message. |
| TC-PP-022 | Unsupported format: WEBP | P0. Valid `image.webp`. | Upload the file. | Rejected with an error message. |
| TC-PP-023 | Unsupported format: SVG | P0. Valid `image.svg`. | Upload the file. | Rejected with an error message. |
| TC-PP-024 | Unsupported format: HEIC | P0. Valid `image.heic` (typical phone photo). | Upload the file. | Rejected with an error message (see OQ-3 about HEIC). |
| TC-PP-025 | Non-image file: PDF | P0. `doc.pdf`. | Upload the file. | Rejected with an error message. |
| TC-PP-026 | Executable or script renamed to .jpg | P0. A text or script file renamed `evil.jpg`. | Upload the file. | Rejected, because the content is not a real JPEG. The file is not stored or served. |
| TC-PP-027 | GIF renamed to .png | P0. A real GIF file renamed `fake.png`. | Upload the file. | Rejected on content validation (see OQ-5), or if accepted, recorded as a defect or an open question. |
| TC-PP-028 | Valid JPG renamed to .png (mismatch) | P0. A real JPEG renamed `photo.png`. | Upload the file. | Behaviour recorded. Expected per OQ-5; at a minimum the file must not break the profile page. |
| TC-PP-029 | Corrupt JPG | P0. A JPG with a valid header but truncated or damaged data. | Upload the file. | Rejected with an error message, or handled safely. The profile page does not show a broken image. |
| TC-PP-030 | Empty file (0 bytes) | P0. `empty.jpg` with 0 bytes. | Upload the file. | Rejected with an error message. |
| TC-PP-031 | File with no extension | P0. A valid JPEG named `photo` with no extension. | Upload the file. | Behaviour recorded per OQ-5. Must not break the page. |
| TC-PP-032 | Double extension | P0. File named `photo.jpg.exe` and another named `photo.exe.jpg` containing non-image data. | Upload each. | Both rejected. |
| TC-PP-033 | Cancel the file chooser | P0. | 1. Open the upload control. 2. Cancel the file dialog. | Nothing changes. No error is shown. The existing picture, if any, is unchanged. |
| TC-PP-034 | Failed upload does not remove existing picture | P0. The user has picture A. | Upload an invalid file (for example a 6 MB JPG). | The error is shown and picture A is still shown on the profile page. |
| TC-PP-035 | Upload while not authenticated | The session has expired, or the user is logged out. | Attempt the upload (for example from a stale open page, or by calling the upload endpoint directly without a session). | The upload is refused (login required or 401/403). No picture is stored. |
| TC-PP-036 | Network interruption during upload | P0. Valid 4 MB JPG. Network can be cut. | 1. Start the upload. 2. Disconnect the network mid-upload. | An error or retry option is shown. The profile is not left in a half-updated state. The previous picture remains. |
| TC-PP-037 | Server-side size enforcement | P0. A 6 MB JPG and a way to bypass the client (for example direct API call). | Send the file directly to the upload endpoint. | The server rejects it. The size limit is not enforced only in the browser. |
| TC-PP-038 | Server-side type enforcement | P0. A PDF, and a way to bypass the client check. | Send the file directly to the upload endpoint with an image content type. | The server rejects it. |
| TC-PP-039 | Filename with special characters | P0. Valid JPG named `ph&ot<o> ' é 照片.jpg`. | Upload the file. | Accepted (or rejected with a clear message). No script injection, and the picture displays correctly. |

## 4. Display scenarios

| ID | Title | Preconditions | Steps | Expected result |
|---|---|---|---|---|
| TC-PP-040 | Picture shown immediately after upload | P0. | Upload a valid picture. | The picture is shown on the profile page without needing a manual refresh (timing per OQ-7). |
| TC-PP-041 | Large-dimension image displays correctly | P0. Valid JPG of 8000x6000 pixels and under 5 MB. | Upload the file. | The picture is shown without breaking the profile page layout. Scaling or cropping follows OQ-6. |
| TC-PP-042 | Tall, wide and square images | P0. Valid images with ratios 1:1, 4:1 and 1:4. | Upload each. | Each is shown without distortion or layout breakage (cropping behaviour per OQ-6). |
| TC-PP-043 | PNG with transparency | P0. PNG with a transparent background. | Upload the file. | Shown correctly. Transparent areas look as specified (OQ-6). |
| TC-PP-044 | JPG with EXIF rotation | P0. Phone photo with an EXIF orientation tag. | Upload the file. | The picture is shown in the correct orientation. |
| TC-PP-045 | Picture visible in supported browsers and on mobile | P0. | Repeat TC-PP-001 and TC-PP-002 on the supported browsers and on a mobile viewport. | Upload and display work in all supported environments. |
| TC-PP-046 | Picture shown only to permitted viewers | P0. A second user account. | View the first user's profile as the second user and as a logged-out visitor. | Visibility follows the profile's access rules (OQ-8). |

## 5. Open questions

| # | Question | Why it matters / affected cases |
|---|---|---|
| OQ-1 | Is "5 MB" 5 x 1024 x 1024 bytes (5 MiB) or 5,000,000 bytes? | Changes the exact boundary bytes in TC-PP-010 to TC-PP-015. |
| OQ-2 | Is a file of exactly 5 MB accepted (limit inclusive) or rejected? | TC-PP-010, TC-PP-011, TC-PP-013. |
| OQ-3 | Are `.jpeg`, `.jpe`, and `.jfif` extensions counted as JPG? Is HEIC (the default phone format) intentionally excluded? Is the extension check case-insensitive? | TC-PP-003, TC-PP-008, TC-PP-024. |
| OQ-4 | What exactly is the error message or behaviour for a wrong format or an oversized file (wording, location, and whether it appears before or after the upload starts)? | Expected results in section 3 only say "an error message is shown". |
| OQ-5 | Is the format validated by extension, by MIME type, or by file content (magic bytes)? What happens when they disagree? | TC-PP-026 to TC-PP-031. Security-relevant. |
| OQ-6 | Are there minimum or maximum pixel dimensions or aspect-ratio rules? Is the picture cropped, resized, or compressed on upload? Is the original kept? How is it displayed (shape, size)? | TC-PP-017, TC-PP-041 to TC-PP-043. |
| OQ-7 | When must the picture appear: instantly after upload, or after processing? Is there a progress indicator or a success message? | TC-PP-040. |
| OQ-8 | Who can see the picture (only the owner, other users, public)? | TC-PP-046. |
| OQ-9 | What happens to the previous picture when replaced (deleted or kept)? Can the user remove their picture and return to the default avatar? The requirement mentions neither removal nor replacement. | TC-PP-006. Missing requirement. |
| OQ-10 | Are animated PNG (APNG) and progressive or CMYK JPEG accepted? | Not covered by cases until answered. |
| OQ-11 | Are there upload rate limits or a limit on the number of uploads per time period? Can the user upload more than one file at a time (multi-select or drag and drop of several files)? | Not covered until answered. |
| OQ-12 | Is EXIF metadata (for example GPS location) stripped from the stored picture? | Privacy; not covered until answered. |
| OQ-13 | Are there performance expectations (upload time for a 5 MB file) and accessibility requirements (keyboard operation, alt text for the picture)? | Not covered until answered. |
| OQ-14 | Where in the product is the upload done (profile page itself, a settings page, at registration)? Is it available on all platforms (web, iOS, Android)? | Affects the preconditions and platform coverage in TC-PP-045. |
