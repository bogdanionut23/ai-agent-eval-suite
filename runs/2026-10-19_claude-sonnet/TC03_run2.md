# Test Cases: Profile Picture Upload

Requirement: Users can upload a profile picture. Allowed formats are JPG and PNG. The maximum file size is 5 MB. After upload the picture is shown on the profile page.

Assumption used for boundary tests: 5 MB is treated as 5 * 1024 * 1024 = 5,242,880 bytes. This is an open question (see OQ-1), so boundary results must be re-checked once it is answered.

## Positive

### TC-01: Upload a valid JPG
- Preconditions: User is logged in and on the profile page. A JPG file of about 1 MB (`photo.jpg`) is available.
- Steps:
  1. Open the profile picture upload control.
  2. Select `photo.jpg`.
  3. Confirm or submit the upload.
  4. Open or refresh the profile page.
- Expected result: Upload succeeds with a success indication. The uploaded picture is shown on the profile page and is still shown after a refresh.

### TC-02: Upload a valid PNG
- Preconditions: As TC-01, with a PNG file of about 1 MB (`photo.png`).
- Steps: As TC-01 using `photo.png`.
- Expected result: Upload succeeds. The PNG is shown on the profile page and persists after a refresh.

### TC-03: Upload a file within the size limit (mid-range)
- Preconditions: Logged-in user. A valid JPG of about 3 MB.
- Steps: Upload the file.
- Expected result: Upload succeeds and the picture is shown on the profile page.

### TC-04: Upload with uppercase or mixed-case extension
- Preconditions: Logged-in user. Valid image files named `PHOTO.JPG` and `photo.PnG`.
- Steps: Upload each file in turn.
- Expected result: Both are accepted as JPG and PNG (extension matching is case-insensitive) and are shown on the profile page. (See OQ-3 for `.jpeg`.)

## Boundary

### TC-05: File size exactly at the limit
- Preconditions: Logged-in user. A valid JPG of exactly 5,242,880 bytes.
- Steps: Upload the file.
- Expected result: Upload succeeds, because "maximum 5 MB" implies 5 MB is allowed. The picture is shown on the profile page.

### TC-06: File size one byte below the limit
- Preconditions: Logged-in user. A valid PNG of 5,242,879 bytes.
- Steps: Upload the file.
- Expected result: Upload succeeds and the picture is shown.

### TC-07: File size one byte above the limit
- Preconditions: Logged-in user. A valid JPG of 5,242,881 bytes.
- Steps: Upload the file.
- Expected result: Upload is rejected with a clear error message about the size limit. The current profile picture is unchanged.

### TC-08: Very small valid file
- Preconditions: Logged-in user. A valid 1x1 pixel PNG (under 1 KB).
- Steps: Upload the file.
- Expected result: Upload succeeds (no minimum size is specified; see OQ-5) and the picture is shown.

### TC-09: Limit expressed in decimal MB
- Preconditions: Logged-in user. A valid JPG of 5,000,001 bytes, and another of 5,200,000 bytes.
- Steps: Upload each file.
- Expected result: Behavior matches the answer to OQ-1. With 1 MB = 1,048,576 bytes both files are accepted. With 1 MB = 1,000,000 bytes both are rejected. Record the observed behavior.

## Negative

### TC-10: Unsupported format (GIF)
- Preconditions: Logged-in user. A valid `anim.gif` of 100 KB.
- Steps: Attempt to upload it.
- Expected result: Upload is rejected with an error stating the allowed formats (JPG, PNG). The profile picture is unchanged.

### TC-11: Other unsupported formats (WEBP, BMP, SVG, HEIC, TIFF, PDF)
- Preconditions: Logged-in user. One small valid file of each type.
- Steps: Attempt to upload each file.
- Expected result: Each is rejected with a format error. Nothing is stored and the profile picture is unchanged.

### TC-12: Non-image file renamed with an image extension
- Preconditions: Logged-in user. A text or executable file renamed to `fake.jpg`.
- Steps: Upload it.
- Expected result: Upload is rejected (content or MIME type is validated, not only the extension). No file is stored or displayed. The behavior is reported as a failure if the file is accepted.

### TC-13: Valid image with the wrong extension
- Preconditions: Logged-in user. A real PNG file renamed to `image.jpg`.
- Steps: Upload it.
- Expected result: Behavior follows the answer to OQ-4 (accept, because content is a valid allowed format, or reject, because of the mismatch). It must not cause an error page or a broken image.

### TC-14: File with no extension
- Preconditions: Logged-in user. A valid PNG named `photo` with no extension.
- Steps: Upload it.
- Expected result: Behavior follows OQ-4. Either it is accepted based on content, or it is rejected with a clear format error. No crash.

### TC-15: Empty file (0 bytes)
- Preconditions: Logged-in user. An empty file named `empty.jpg`.
- Steps: Upload it.
- Expected result: Upload is rejected with an error. No broken image is shown. (Exact message: see OQ-5.)

### TC-16: Corrupted image
- Preconditions: Logged-in user. A truncated or corrupted JPG with a valid header.
- Steps: Upload it.
- Expected result: Upload is rejected with a clear error, or the system handles it gracefully. A broken image icon must not be shown on the profile page.

### TC-17: Much larger file (for example 50 MB)
- Preconditions: Logged-in user. A valid JPG of about 50 MB.
- Steps: Upload it.
- Expected result: Upload is rejected with a size error. The server and client stay responsive. Ideally the client rejects it before transferring the whole file.

### TC-18: Cancel the file picker
- Preconditions: Logged-in user on the profile page.
- Steps:
  1. Open the upload control.
  2. Cancel the file dialog without choosing a file.
- Expected result: No upload occurs and no error is shown. The existing picture is unchanged.

### TC-19: Unauthenticated upload attempt
- Preconditions: User is logged out, or the session has expired. The upload endpoint is known.
- Steps: Send an upload request directly to the endpoint without a valid session.
- Expected result: Request is rejected (401 or 403, or a redirect to login). No file is stored.

### TC-20: Network interruption during upload
- Preconditions: Logged-in user. A valid 4 MB file.
- Steps:
  1. Start the upload.
  2. Disconnect the network mid-transfer.
  3. Reconnect and reload the profile page.
- Expected result: An error or retry option is shown. The previous picture is intact and there is no partially uploaded or corrupted picture.

### TC-21: Server-side validation bypass
- Preconditions: Logged-in user. A 6 MB file and a `.gif` file. A tool such as curl or Postman is available.
- Steps: Send each file directly to the upload API, bypassing client-side checks.
- Expected result: The server rejects both with appropriate 4xx errors. Limits are enforced server-side, not only in the UI.

## Display and Behavior After Upload

### TC-22: Picture displayed on profile page after upload
- Preconditions: Logged-in user with no picture set (default avatar shown).
- Steps:
  1. Upload a valid PNG.
  2. Navigate to the profile page.
- Expected result: The uploaded picture replaces the default avatar. It is not stretched or distorted and it loads without error.

### TC-23: Replace an existing picture
- Preconditions: User already has picture A set.
- Steps: Upload a different valid picture B.
- Expected result: Picture B is shown. Picture A is no longer shown, even after a hard refresh (no stale cache). (See OQ-6.)

### TC-24: Failed upload does not change the existing picture
- Preconditions: User already has a picture set.
- Steps: Attempt to upload an invalid file (for example a GIF or a 6 MB file).
- Expected result: Error shown. The previous picture remains displayed.

### TC-25: Image with a very large pixel size but under 5 MB
- Preconditions: Logged-in user. A highly compressed JPG of 10000x10000 pixels and under 5 MB.
- Steps: Upload it.
- Expected result: Behavior follows OQ-7. At minimum, the system must not hang or crash, and the page layout is not broken.

### TC-26: Image with unusual aspect ratio
- Preconditions: Logged-in user. A valid 2000x100 PNG.
- Steps: Upload it and view the profile page.
- Expected result: The picture is displayed without breaking the layout (crop or fit per design; see OQ-7).

### TC-27: Filename with special characters
- Preconditions: Logged-in user. Valid files named `fotó (1) ü#&%.jpg`, a name with spaces, and a very long name.
- Steps: Upload each.
- Expected result: Upload succeeds and the picture is displayed. No encoding or path errors occur, and no path traversal is possible through the filename.

### TC-28: Picture visible after logout and login
- Preconditions: User has uploaded a valid picture.
- Steps: Log out, log in again, open the profile page.
- Expected result: The uploaded picture is still shown (persistence).

### TC-29: Image with EXIF orientation
- Preconditions: Logged-in user. A JPG with EXIF orientation set to rotated.
- Steps: Upload it and view the profile page.
- Expected result: The picture is displayed in the intended orientation (see OQ-7 for the expected handling).

### TC-30: Double submission
- Preconditions: Logged-in user. A valid file.
- Steps: Click the upload or confirm button twice quickly.
- Expected result: Only one upload is processed, and no duplicate or error state results.

### TC-31: Basic security check on content
- Preconditions: Logged-in user. A valid JPG containing embedded script or metadata payloads, and an SVG with script renamed to `.png`.
- Steps: Upload each and view the profile page.
- Expected result: Uploads are rejected or sanitized. No script executes when the profile page is viewed.

## Open Questions

- OQ-1: Is "5 MB" 5 x 1024 x 1024 bytes (MiB) or 5,000,000 bytes? Is exactly 5 MB accepted? TC-05, TC-07 and TC-09 depend on this.
- OQ-2: What exact error messages and UI feedback should be shown for a wrong format, an oversized file and a failed upload? Is there a progress indicator?
- OQ-3: Are `.jpeg` (and `.jpe`/`.jfif`) accepted as JPG? Is extension matching case-insensitive? Is validation by extension, MIME type, file content (magic bytes) or all of these?
- OQ-4: What happens when the extension and the real content disagree (PNG renamed to `.jpg`), or the file has no extension?
- OQ-5: Is there a minimum file size or minimum pixel dimension? How should an empty or 0-byte file be handled?
- OQ-6: Can a user replace or remove the picture? Is the old file deleted? Is there a default avatar when no picture is set?
- OQ-7: Are there maximum pixel dimensions, automatic resizing, cropping or thumbnails? How are the picture and its aspect ratio displayed on the profile page? Is EXIF orientation applied and is EXIF metadata (such as GPS) stripped?
- OQ-8: Is the picture shown immediately after upload (without a page reload), or only on the next page load? Where else is it shown (header, comments, other users' views)?
- OQ-9: Who can see the profile picture (privacy and visibility rules), and is there moderation or content review?
- OQ-10: Which platforms and browsers are in scope (web, mobile apps)? Is camera capture or drag and drop supported? Is multi-file selection allowed (and what happens if several files are chosen)?
- OQ-11: Are animated PNG (APNG), progressive JPEG, CMYK JPEG, or 16-bit PNG supported?
- OQ-12: Are there any rate limits or quotas on uploads per user? What are the accessibility requirements (alt text, keyboard operation of the upload control)?
