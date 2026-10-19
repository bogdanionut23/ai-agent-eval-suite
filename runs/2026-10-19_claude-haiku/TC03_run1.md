# TC03 - Test cases: Profile picture upload

Requirement: Users can upload a profile picture. Allowed formats are JPG and PNG. The maximum file size is 5 MB. After upload the picture is shown on the profile page.

Assumptions used for expected results (each is also listed under Open questions):
- "5 MB" is treated as 5 x 1024 x 1024 = 5,242,880 bytes. Boundary tests state the byte values explicitly so they can be re-pointed if the decimal definition (5,000,000 bytes) is chosen.
- The user is logged in and is uploading to their own profile.
- A "format" is judged by the actual file content, not only by the extension. Where the two disagree, the case is marked as blocked by an open question.

---

## Positive scenarios

### TC03-01 - Upload a JPG under the size limit
- Preconditions: User is logged in. User has a JPG file of about 500 KB on the device. User has no profile picture yet.
- Steps:
  1. Open the profile page and choose "Upload profile picture".
  2. Select the JPG file and confirm the upload.
  3. Wait for the upload to finish.
- Expected result: The upload succeeds with no error message. The uploaded image is shown on the profile page. The image is not distorted or cropped unexpectedly.

### TC03-02 - Upload a PNG under the size limit
- Preconditions: User is logged in. User has a PNG file of about 500 KB. User has no profile picture yet.
- Steps:
  1. Open the profile page and choose "Upload profile picture".
  2. Select the PNG file and confirm the upload.
  3. Wait for the upload to finish.
- Expected result: The upload succeeds. The PNG is shown on the profile page. Transparency in the PNG is rendered without a broken background.

### TC03-03 - Uploaded picture persists after reload
- Preconditions: TC03-01 or TC03-02 has completed successfully.
- Steps:
  1. Reload the profile page (or log out and log back in).
  2. Check the profile picture.
- Expected result: The same picture is still shown. It was saved, not only shown in the current session.

### TC03-04 - Replace an existing profile picture
- Preconditions: User is logged in and already has a profile picture (picture A).
- Steps:
  1. Upload a new valid JPG or PNG (picture B).
  2. Wait for the upload to finish and check the profile page.
- Expected result: Picture B is shown. Picture A is no longer shown anywhere on the profile page.

### TC03-05 - Upload a JPG with an uppercase extension
- Preconditions: User is logged in. The file is a valid JPG named "photo.JPG".
- Steps:
  1. Upload "photo.JPG" as the profile picture.
- Expected result: Accepted, the same as "photo.jpg". (Assumption; see Open question Q4.)

### TC03-06 - Upload a JPG with the .jpeg extension
- Preconditions: User is logged in. The file is a valid JPEG named "photo.jpeg".
- Steps:
  1. Upload "photo.jpeg" as the profile picture.
- Expected result: Accepted, because .jpeg is the same JPG format. (Assumption; see Open question Q4.)

---

## Size boundary scenarios

### TC03-07 - File size exactly at the limit (5,242,880 bytes)
- Preconditions: User is logged in. A valid JPG of exactly 5,242,880 bytes exists.
- Steps:
  1. Upload the file as the profile picture.
- Expected result: Accepted. The limit is inclusive ("maximum" means up to and including the limit). The picture is shown on the profile page.

### TC03-08 - File size one byte over the limit (5,242,881 bytes)
- Preconditions: User is logged in. A valid JPG of exactly 5,242,881 bytes exists.
- Steps:
  1. Upload the file as the profile picture.
- Expected result: Rejected before or during upload. A clear error message says the file is too large and gives the 5 MB limit. The previous profile picture (or the default one) remains unchanged.

### TC03-09 - File size one byte under the limit (5,242,879 bytes)
- Preconditions: User is logged in. A valid PNG of exactly 5,242,879 bytes exists.
- Steps:
  1. Upload the file as the profile picture.
- Expected result: Accepted and shown on the profile page.

### TC03-10 - Oversized file is rejected on the server, not only in the browser
- Preconditions: User is logged in. A valid JPG of 6 MB exists. A tool such as curl or a proxy is available to send the upload request directly to the API, skipping the client-side check.
- Steps:
  1. Send the upload request for the 6 MB file directly to the upload endpoint with a valid session.
  2. Read the response and then check the stored profile picture.
- Expected result: The server returns an error (for example 413 or 400) and does not store the file. The stored profile picture does not change.

---

## Format negative scenarios

### TC03-11 - Reject GIF
- Preconditions: User is logged in. A valid GIF file of 100 KB exists.
- Steps:
  1. Upload the GIF as the profile picture.
- Expected result: Rejected. A clear error message says the format is not allowed and lists JPG and PNG. The profile picture does not change.

### TC03-12 - Reject WEBP
- Preconditions: User is logged in. A valid WEBP file of 100 KB exists.
- Steps:
  1. Upload the WEBP file as the profile picture.
- Expected result: Rejected with a clear format error. The profile picture does not change.

### TC03-13 - Reject other image and document types (BMP, HEIC, SVG, PDF)
- Preconditions: User is logged in. Valid sample files exist for BMP, HEIC, SVG and PDF, each under 5 MB.
- Steps:
  1. For each file, upload it as the profile picture.
- Expected result: Each upload is rejected with a format error. None of them is stored or shown.

### TC03-14 - Reject a non-image file with a .jpg extension
- Preconditions: User is logged in. A text file (or a PDF) is renamed to "fake.jpg".
- Steps:
  1. Upload "fake.jpg" as the profile picture.
- Expected result: Rejected. The file is not stored and is not shown as a picture. A broken-image placeholder must not appear on the profile page. (Assumption that content is checked; see Open question Q5.)

### TC03-15 - Reject an extension that is not allowed for a valid image
- Preconditions: User is logged in. A valid PNG file is renamed to "photo.gif".
- Steps:
  1. Upload "photo.gif" as the profile picture.
- Expected result: Rejected by the extension check. (Assumption; see Open question Q5.)

### TC03-16 - Corrupt JPG with a valid extension
- Preconditions: User is logged in. A file named "broken.jpg" is truncated halfway through, so it is not a valid image. Its size is under 5 MB.
- Steps:
  1. Upload "broken.jpg" as the profile picture.
- Expected result: Rejected with an error such as "file could not be read as an image". No broken image is shown on the profile page. The previous picture stays.

### TC03-17 - Zero-byte file
- Preconditions: User is logged in. An empty file named "empty.png" (0 bytes) exists.
- Steps:
  1. Upload "empty.png" as the profile picture.
- Expected result: Rejected with an error. Nothing is stored or shown.

---

## Error handling and state scenarios

### TC03-18 - Failed upload keeps the existing picture
- Preconditions: User is logged in and has picture A set. A GIF file exists.
- Steps:
  1. Try to upload the GIF (see TC03-11).
  2. Reload the profile page.
- Expected result: Picture A is still shown after the failed attempt.

### TC03-19 - Cancelling the file picker changes nothing
- Preconditions: User is logged in and has picture A set.
- Steps:
  1. Choose "Upload profile picture".
  2. Close the file picker without selecting a file.
- Expected result: No upload starts and no error appears. Picture A is still shown.

### TC03-20 - Confirm upload without selecting a file
- Preconditions: User is logged in. The upload dialog is open and no file is selected.
- Steps:
  1. Press the confirm or upload button.
- Expected result: The action does nothing harmful. A message asks the user to select a file. No request is stored or sent with an empty file.

### TC03-21 - Network interruption during upload
- Preconditions: User is logged in and has picture A set. A valid 3 MB JPG exists. The network can be disconnected during the upload.
- Steps:
  1. Start the upload of the JPG.
  2. Disconnect the network before the upload completes.
  3. Restore the network and reload the profile page.
- Expected result: An error is shown during the upload. No partial or corrupt picture is stored. Picture A is still shown.

### TC03-22 - Double submit of the same upload
- Preconditions: User is logged in. A valid 1 MB PNG exists.
- Steps:
  1. Select the PNG.
  2. Click the upload button twice quickly.
- Expected result: Only one picture is stored. The profile page shows the picture once with no duplicate entries and no error.

### TC03-23 - Upload by an unauthenticated visitor
- Preconditions: User is logged out.
- Steps:
  1. Try to open the upload function on a profile page, or call the upload endpoint without a session.
- Expected result: The upload is refused, and the user is asked to log in. No file is stored. (Assumption; see Open question Q9.)

---

## Display scenarios

### TC03-24 - Picture is displayed at a usable size on a phone-width screen
- Preconditions: User is logged in. A valid 4000 x 3000 px JPG of under 5 MB exists. The browser is set to phone width (about 375 px).
- Steps:
  1. Upload the JPG.
  2. View the profile page at phone width.
- Expected result: The picture is shown inside the profile layout. It does not overflow the screen and causes no horizontal scrolling.

### TC03-25 - Picture is shown in the correct orientation
- Preconditions: User is logged in. A JPG taken on a phone has an EXIF orientation tag that requires a 90 degree rotation.
- Steps:
  1. Upload the JPG.
  2. View the profile page.
- Expected result: The picture is shown upright, as it was taken. (Expected behaviour is not in the requirement; see Open question Q7.)

---

## Open questions

1. Size limit definition. Is 5 MB 5,242,880 bytes (MiB) or 5,000,000 bytes? The boundary cases TC03-07 to TC03-09 depend on this.
2. Is the limit inclusive? Is a file of exactly 5 MB accepted? (The requirement says "maximum", which suggests inclusive, but this is not stated.)
3. Which formats count as JPG and PNG? Are .jpeg, .JPG and .PNG accepted? Are other encodings of the same formats (progressive JPG, 16-bit PNG, animated PNG/APNG) accepted?
4. Are uppercase and alternate extensions (.JPG, .jpeg) allowed? Expected results in TC03-05 and TC03-06 are assumptions.
5. Is the format checked by file content (magic bytes / MIME type) or only by file extension? What happens when they disagree (a PNG named .gif, or a text file named .jpg)? TC03-14 and TC03-15 depend on this.
6. What is the maximum image dimension in pixels? Are there limits on width and height, or on total pixels? Are very large images resized, cropped, or stored as they are? The requirement says nothing about this.
7. Should the picture be auto-rotated using EXIF orientation? Is cropping or a crop tool required? TC03-25 depends on this.
8. Whose profile page is the picture shown on? Only the owner's own page, or also on other users' pages, search results, comments or the header? "Shown on the profile page" is not specific enough.
9. Who can upload? Is any logged-in user allowed to upload only to their own profile? Can an admin change someone else's picture? What exactly happens when a logged-out user tries (TC03-23)?
10. What is shown while no picture is set: a default avatar, initials, or nothing?
11. Is removing a picture (delete) a requirement? It is not mentioned.
12. Is the picture shown immediately after the upload, or only after saving and reloading? What is the expected delay?
13. What is the exact error text and how is it shown (inline, toast, or modal)? Is there a specific error code requirement for the API?
14. Can more than one file be selected at once? If so, is only the first file used, or is the upload rejected?
15. Is there a limit on how often a picture can be changed (rate limit)?
16. Is the stored image re-encoded or stored byte-for-byte as uploaded? This affects metadata (EXIF location) handling and privacy.
17. Are there requirements for the upload progress indicator, retries after network failure, or a maximum upload time?
18. Accessibility: is alt text required for the picture, and does the upload control work with keyboard and screen readers? Not specified.
