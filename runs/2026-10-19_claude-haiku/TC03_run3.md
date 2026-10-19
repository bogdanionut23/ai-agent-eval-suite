# TC03: Profile picture upload - test cases

## Requirement

Users can upload a profile picture. Allowed formats are JPG and PNG. The maximum file size is 5 MB. After upload the picture is shown on the profile page.

## Assumptions (used only to make the cases executable)

- "5 MB" is treated as 5 * 1024 * 1024 = 5,242,880 bytes. Open question 2 covers this.
- "Shown on the profile page" means the picture is rendered for the logged-in user on their own profile page.
- The user is logged in with an account that has a profile.

## Positive scenarios

### TC03-P01: Upload a valid JPG within the size limit
- **Preconditions:** User is logged in. Profile page is open. A JPG file of about 1 MB exists locally.
- **Steps:**
  1. Open the profile picture upload control.
  2. Select the JPG file.
  3. Confirm the upload.
- **Expected result:** Upload succeeds with no error message. The uploaded picture is shown on the profile page.

### TC03-P02: Upload a valid PNG within the size limit
- **Preconditions:** User is logged in. A PNG file of about 1 MB exists locally.
- **Steps:**
  1. Open the profile picture upload control.
  2. Select the PNG file.
  3. Confirm the upload.
- **Expected result:** Upload succeeds. The PNG is shown on the profile page.

### TC03-P03: Picture is still shown after page reload and new session
- **Preconditions:** TC03-P01 has completed successfully.
- **Steps:**
  1. Reload the profile page.
  2. Log out, log in again, and open the profile page.
- **Expected result:** The uploaded picture is shown in both cases.

### TC03-P04: Replacing an existing profile picture
- **Preconditions:** User already has a profile picture uploaded.
- **Steps:**
  1. Upload a new valid JPG or PNG.
  2. Open the profile page.
- **Expected result:** The new picture is shown. The old picture is no longer shown.

### TC03-P05: Upload a file of exactly the maximum size
- **Preconditions:** User is logged in. A valid JPG of exactly 5 MB (5,242,880 bytes) exists locally.
- **Steps:**
  1. Upload the file.
- **Expected result:** Upload is accepted and the picture is shown on the profile page. (The 5 MB limit is inclusive. See open question 2.)

### TC03-P06: Upload of uppercase or mixed-case extension
- **Preconditions:** User is logged in. Valid image files named `photo.JPG` and `photo.Png` exist.
- **Steps:**
  1. Upload `photo.JPG`. Confirm it is shown.
  2. Upload `photo.Png`. Confirm it is shown.
- **Expected result:** Both are accepted, since the format rule is about JPG/PNG and not letter case. (See open question 4.)

## Negative scenarios

### TC03-N01: Reject a file in an unsupported format
- **Preconditions:** User is logged in. A GIF, WEBP, BMP, HEIC or SVG file of under 5 MB exists locally.
- **Steps:**
  1. Attempt to upload the file.
- **Expected result:** Upload is rejected. An error message says the format is not allowed. The current profile picture is unchanged.

### TC03-N02: Reject a file over the size limit
- **Preconditions:** User is logged in. A valid JPG of 5 MB + 1 byte exists locally.
- **Steps:**
  1. Attempt to upload the file.
- **Expected result:** Upload is rejected. An error message says the file is too large and states the 5 MB limit. The current profile picture is unchanged.

### TC03-N03: Reject a large file of a supported format
- **Preconditions:** User is logged in. A valid PNG of about 20 MB exists locally.
- **Steps:**
  1. Attempt to upload the file.
- **Expected result:** Upload is rejected with a size error, not a format error.

### TC03-N04: Reject a non-image file renamed to .jpg
- **Preconditions:** User is logged in. A text file (or PDF) renamed to `fake.jpg` and under 5 MB exists locally.
- **Steps:**
  1. Attempt to upload `fake.jpg`.
- **Expected result:** Upload is rejected because the content is not a valid JPG. The system should not show a broken image on the profile page. (The requirement does not say whether content is checked, so this is an expectation to confirm. See open question 3.)

### TC03-N05: Reject a PNG file renamed to .jpg, or the reverse
- **Preconditions:** User is logged in. A valid PNG file renamed to `photo.jpg` exists locally.
- **Steps:**
  1. Attempt to upload `photo.jpg`.
- **Expected result:** Behaviour depends on open question 3. If content is validated, the file is accepted as PNG data or rejected. In either case, no broken image is shown.

### TC03-N06: Reject an empty (0-byte) file
- **Preconditions:** User is logged in. A 0-byte file named `empty.png` exists locally.
- **Steps:**
  1. Attempt to upload the file.
- **Expected result:** Upload is rejected with an error. No broken image is shown on the profile page.

### TC03-N07: Upload attempted without being logged in
- **Preconditions:** User is logged out. The upload endpoint or page is known.
- **Steps:**
  1. Attempt to upload a valid JPG.
- **Expected result:** Upload is refused and the user is asked to log in. No picture is stored.

### TC03-N08: Upload is interrupted (network loss during transfer)
- **Preconditions:** User is logged in. A valid 4 MB JPG exists locally. Network can be disabled during upload.
- **Steps:**
  1. Start the upload.
  2. Disconnect the network before the upload completes.
  3. Restore the network and reload the profile page.
- **Expected result:** An error is shown during the upload. The previous profile picture is still shown. No partial or corrupt file is shown.

### TC03-N09: Cancel the file selection dialog
- **Preconditions:** User is logged in. Profile page is open.
- **Steps:**
  1. Open the upload control.
  2. Cancel the file selection dialog without choosing a file.
- **Expected result:** No upload happens. No error is shown. The current picture is unchanged.

### TC03-N10: Upload of a corrupt JPG or PNG
- **Preconditions:** User is logged in. A JPG with a valid extension but truncated content exists locally.
- **Steps:**
  1. Attempt to upload the file.
- **Expected result:** Either the upload is rejected with an error, or it is accepted and the picture shows a clear placeholder. The page must not crash. (See open question 3.)

## Boundary scenarios

### TC03-B01: File size one byte below the limit
- **Preconditions:** User is logged in. A valid JPG of 5,242,879 bytes exists locally.
- **Steps:**
  1. Upload the file.
- **Expected result:** Upload is accepted and the picture is shown.

### TC03-B02: File size exactly at the limit
- **Preconditions:** User is logged in. A valid PNG of 5,242,880 bytes exists locally.
- **Steps:**
  1. Upload the file.
- **Expected result:** Upload is accepted (limit is inclusive). See open question 2.

### TC03-B03: File size one byte above the limit
- **Preconditions:** User is logged in. A valid PNG of 5,242,881 bytes exists locally.
- **Steps:**
  1. Upload the file.
- **Expected result:** Upload is rejected with a size error.

### TC03-B04: Very small valid image
- **Preconditions:** User is logged in. A valid 1x1 pixel PNG (about 70 bytes) exists locally.
- **Steps:**
  1. Upload the file.
- **Expected result:** Upload is accepted and the picture is shown. (The requirement sets no minimum dimensions. See open question 5.)

### TC03-B05: Image with very large dimensions but size under 5 MB
- **Preconditions:** User is logged in. A highly compressed JPG of 10000 x 10000 pixels and under 5 MB exists locally.
- **Steps:**
  1. Upload the file.
  2. Open the profile page.
- **Expected result:** Behaviour is defined by open question 5. Expected: accepted and shown without breaking the page layout, or rejected with a clear message. It must not hang the page.

### TC03-B06: Upload of a file with a double extension
- **Preconditions:** User is logged in. A valid JPG named `photo.png.jpg` exists locally.
- **Steps:**
  1. Upload the file.
- **Expected result:** Accepted, because the final extension is JPG and the content is valid JPG. Or rejected if the rule uses the first extension. (See open question 4.)

### TC03-B07: Upload of the same file twice in a row
- **Preconditions:** User is logged in. A valid JPG exists locally.
- **Steps:**
  1. Upload the file.
  2. Upload the same file again.
- **Expected result:** Both uploads succeed (or the second is a no-op). The picture is shown and no duplicate or broken state appears.

## Open questions

1. **Format detection:** Is the format check based on the file extension, the MIME type sent by the browser, or the actual file content (magic bytes)? Expected behaviour for a renamed file (TC03-N04, TC03-N05) depends on this.
2. **Size limit definition:** Is 5 MB 5,000,000 bytes or 5,242,880 bytes (MiB)? Is the limit inclusive? Does it apply to the file or to the whole request body?
3. **Content validation:** Must a corrupt or mislabelled image be rejected at upload, or may it be accepted? What is shown if it is accepted and cannot be decoded?
4. **Extension case and variants:** Are `.JPG`, `.jpeg`, `.JPEG` and `.PNG` accepted? Is `.jpeg` a JPG? How are double extensions handled?
5. **Image dimensions:** Are there minimum or maximum pixel dimensions, or an aspect ratio requirement? Is the picture cropped, resized or shown as uploaded on the profile page?
6. **Animated PNG (APNG):** Is an animated PNG a valid PNG? Is it shown animated or as a still frame?
7. **Replacement and deletion:** Does a new upload replace the old picture immediately and permanently? Is there a way to remove a profile picture and go back to a default? (Not stated in the requirement.)
8. **Error messages:** What exact text should be shown for each error (format, size, network failure)? Is the message translated?
9. **Where else is the picture shown:** The requirement only says "the profile page". Is it also shown in comments, headers or to other users viewing this user's profile? Are other users allowed to see it?
10. **Access and permissions:** Can an admin or another user upload a picture for someone else? Is there a rate limit on uploads?
11. **Caching:** After replacing the picture, how soon must the new one appear? Is a browser or CDN cache allowed to show the old picture?
12. **Storage and privacy:** Is EXIF metadata (such as GPS location) removed on upload? Not mentioned in the requirement.
13. **Upload method:** Is upload done from a file picker only, or also from a camera on mobile devices? Are drag-and-drop uploads supported?
14. **Mobile and accessibility:** Is the upload control usable on phone screens and with a keyboard or screen reader? Not stated.
15. **Expected behaviour on the profile page when no picture exists:** Is there a default avatar? What is shown while the upload is still processing?
