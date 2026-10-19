# TC03 - Test cases: Profile picture upload

## Requirement (as given)

Users can upload a profile picture. Allowed formats are JPG and PNG. The maximum file size is 5 MB. After upload the picture is shown on the profile page.

## Assumptions used for the test cases

These are the only interpretations made. Each one is also listed under Open questions.

- "5 MB" is read as 5 x 1,048,576 bytes (5 MiB). A file of exactly that size is accepted, and one byte more is rejected.
- A file is "JPG" if it is a real JPEG, and "PNG" if it is a real PNG. The extension alone is not treated as proof of format.
- Only a logged-in user can upload a picture for their own profile.

## Test cases

### Positive

**TC03-01 - Upload a valid JPG and see it on the profile page**
- Preconditions: User is logged in. User has no profile picture set. A valid JPG under 5 MB (e.g. 500 KB, 1000x1000 px) is available locally.
- Steps:
  1. Open the profile page.
  2. Choose the upload control for the profile picture.
  3. Select the JPG file and confirm the upload.
  4. Wait for the upload to finish.
  5. Look at the profile page.
- Expected result: The upload succeeds with a success indication and no error. The uploaded image is shown on the profile page. The displayed image matches the file that was uploaded.

**TC03-02 - Upload a valid PNG and see it on the profile page**
- Preconditions: User is logged in. A valid PNG under 5 MB (e.g. 300 KB, with transparency) is available locally.
- Steps:
  1. Open the profile page.
  2. Choose the upload control and select the PNG file.
  3. Confirm the upload and wait for it to finish.
  4. Look at the profile page.
- Expected result: The upload succeeds. The PNG is shown on the profile page. The transparent areas are rendered, not replaced by a different background from the upload itself (see open questions on processing).

**TC03-03 - Uploaded picture persists after page reload**
- Preconditions: TC03-01 or TC03-02 has completed successfully.
- Steps:
  1. Reload the profile page (full reload).
  2. Log out, then log in again as the same user and open the profile page.
- Expected result: The uploaded picture is still shown after the reload and after the new login.

**TC03-04 - Replace an existing profile picture**
- Preconditions: User is logged in and already has a profile picture (picture A).
- Steps:
  1. Open the profile page.
  2. Upload a valid JPG or PNG (picture B) under 5 MB.
  3. Wait for the upload to finish.
  4. Look at the profile page.
- Expected result: Picture B is shown. Picture A is no longer shown anywhere on the profile page after the reload.

**TC03-05 - Upload of an extension in upper case is accepted**
- Preconditions: User is logged in. A valid JPEG file named `photo.JPG` (upper-case extension) under 5 MB is available.
- Steps:
  1. Upload `photo.JPG` as the profile picture.
  2. Look at the profile page.
- Expected result: The upload is accepted and the picture is shown. (Assumes extension matching is case-insensitive. See open questions.)

**TC03-06 - JPEG file with the .jpeg extension**
- Preconditions: User is logged in. A valid JPEG file named `photo.jpeg` under 5 MB is available.
- Steps:
  1. Upload `photo.jpeg` as the profile picture.
  2. Look at the profile page.
- Expected result: The upload is accepted and the picture is shown. (Depends on whether `.jpeg` is in the allowed list. See open questions.)

### Boundary (file size)

**TC03-07 - File exactly at 5 MB (5,242,880 bytes) is accepted**
- Preconditions: User is logged in. A valid JPG of exactly 5,242,880 bytes is available. Check the size with `ls -l` or the OS file properties.
- Steps:
  1. Upload the file as the profile picture.
  2. Look at the profile page.
- Expected result: The upload is accepted and the picture is shown.

**TC03-08 - File of 5,242,881 bytes (5 MiB + 1 byte) is rejected**
- Preconditions: User is logged in. A valid JPG of exactly 5,242,881 bytes is available. The current profile picture is known (or none is set).
- Steps:
  1. Upload the file as the profile picture.
  2. Observe the result.
  3. Look at the profile page.
- Expected result: The upload is rejected. An error message tells the user that the file is too large. The profile picture does not change.

**TC03-09 - File just under the limit (5,242,879 bytes) is accepted**
- Preconditions: User is logged in. A valid JPG of 5,242,879 bytes is available.
- Steps:
  1. Upload the file as the profile picture.
  2. Look at the profile page.
- Expected result: The upload is accepted and the picture is shown.

**TC03-10 - Oversized file (e.g. 20 MB) is rejected**
- Preconditions: User is logged in. A valid JPG of about 20 MB is available.
- Steps:
  1. Upload the file as the profile picture.
  2. Observe the result.
- Expected result: The upload is rejected with an error about the file size. No picture is stored. The profile page is unchanged.

### Negative (format)

**TC03-11 - GIF is rejected**
- Preconditions: User is logged in. A valid GIF under 5 MB is available.
- Steps:
  1. Upload the GIF as the profile picture.
  2. Observe the result.
  3. Look at the profile page.
- Expected result: The upload is rejected with an error that the format is not allowed. The profile picture does not change.

**TC03-12 - WebP is rejected**
- Preconditions: User is logged in. A valid WebP under 5 MB is available.
- Steps:
  1. Upload the WebP file as the profile picture.
  2. Observe the result.
- Expected result: The upload is rejected with a format error. The profile picture does not change.

**TC03-13 - PDF is rejected**
- Preconditions: User is logged in. A valid PDF under 5 MB is available.
- Steps:
  1. Upload the PDF as the profile picture.
  2. Observe the result.
- Expected result: The upload is rejected with a format error. The profile picture does not change.

**TC03-14 - Non-image file with a .jpg extension is rejected**
- Preconditions: User is logged in. A plain text file renamed to `notes.jpg` (under 5 MB) is available.
- Steps:
  1. Upload `notes.jpg` as the profile picture.
  2. Observe the result.
- Expected result: The upload is rejected because the content is not a JPEG. The profile picture does not change. (Expected result is proposed. See open questions.)

**TC03-15 - PNG content with a .jpg extension**
- Preconditions: User is logged in. A valid PNG file renamed to `photo.jpg` is available.
- Steps:
  1. Upload `photo.jpg` as the profile picture.
  2. Observe the result.
- Expected result: Proposed: the upload is rejected because the content does not match the JPG format. Alternatively, it is accepted if the format is decided by extension only. See open questions.

**TC03-16 - Corrupt JPG (truncated file) is rejected**
- Preconditions: User is logged in. A valid JPG truncated to half its size (under 5 MB) is available.
- Steps:
  1. Upload the truncated file as the profile picture.
  2. Observe the result.
  3. Look at the profile page.
- Expected result: The upload is rejected with an error about an invalid or unreadable image. The page does not show a broken image.

**TC03-17 - File without an extension is rejected**
- Preconditions: User is logged in. A valid JPG file with no file extension is available.
- Steps:
  1. Upload the file as the profile picture.
  2. Observe the result.
- Expected result: Proposed: the upload is rejected with a format error. (See open questions: content-based validation.)

**TC03-18 - Executable or script file renamed to .png is rejected**
- Preconditions: User is logged in. A small shell script or `.exe` file renamed to `evil.png` is available.
- Steps:
  1. Upload `evil.png` as the profile picture.
  2. Observe the result.
  3. Look at the profile page.
- Expected result: The upload is rejected. The file is not stored as the picture and is not served or executed.

### Negative (no file, failure during upload)

**TC03-19 - Submit with no file selected**
- Preconditions: User is logged in. No file is selected in the upload control.
- Steps:
  1. Open the profile page.
  2. Press the upload/confirm button without choosing a file.
- Expected result: No request is sent, or the request is refused. An error or validation message tells the user to select a file. The profile picture does not change.

**TC03-20 - Network failure during upload keeps the old picture**
- Preconditions: User is logged in and has picture A. A valid JPG under 5 MB is available. Network can be cut during the upload (e.g. browser offline mode or throttling and cut at 50%).
- Steps:
  1. Start uploading the JPG.
  2. Disconnect the network while the upload is in progress.
  3. Reconnect and reload the profile page.
- Expected result: The user is told the upload failed. Picture A is still shown. No partial or broken picture is saved.

**TC03-21 - Rejected upload does not remove the existing picture**
- Preconditions: User is logged in and has picture A. A GIF (or oversized file) is available.
- Steps:
  1. Upload the GIF (or oversized file) as the profile picture.
  2. Observe the error.
  3. Reload the profile page.
- Expected result: The error is shown. Picture A is still shown after the reload.

### Negative (authorisation)

**TC03-22 - Anonymous user cannot upload**
- Preconditions: User is logged out. The URL of the upload endpoint or the profile page is known.
- Steps:
  1. Try to open the profile page and use the upload control.
  2. If possible, send an upload request to the endpoint directly without a session.
- Expected result: The user is redirected to log in, or the request is refused with an authentication error. No picture is stored.

**TC03-23 - User cannot change another user's picture**
- Preconditions: User A and User B exist. User A is logged in. The profile ID or endpoint for User B is known.
- Steps:
  1. As User A, send an upload request that targets User B's profile.
- Expected result: The request is refused (authorisation error). User B's picture does not change.

### Display

**TC03-24 - Picture is visible on the profile page to other users**
- Preconditions: User A has uploaded a valid picture (TC03-01). User B is logged in on a different browser or session.
- Steps:
  1. As User B, open User A's profile page.
- Expected result: User A's picture is shown. (Visibility rules are an assumption. See open questions.)

**TC03-25 - Picture is shown at phone width without breaking the layout**
- Preconditions: User has a valid uploaded picture. Browser is set to a phone viewport (e.g. 375 px wide).
- Steps:
  1. Open the profile page.
- Expected result: The picture is fully visible and fits the screen. The page has no horizontal scrolling caused by the picture.

**TC03-26 - Large image dimensions under 5 MB**
- Preconditions: User is logged in. A valid PNG under 5 MB but with very large pixel dimensions (e.g. 8000x8000 px) is available.
- Steps:
  1. Upload the PNG as the profile picture.
  2. Observe the result and look at the profile page.
- Expected result: Proposed: the upload is accepted and the picture is shown without freezing the page. (The requirement does not limit pixel dimensions. See open questions.)

## Open questions

These points are not defined in the requirement. No behaviour has been assumed for them beyond the proposed expectations marked above.

1. **Size unit and limit:** Is "5 MB" 5,000,000 bytes or 5,242,880 bytes (5 MiB)? This changes the expected results of TC03-07 to TC03-09.
2. **Inclusive limit:** Is a file of exactly 5 MB accepted (the "maximum" is inclusive)? TC03-07 assumes yes.
3. **Format check method:** Is the format checked by file extension, by MIME type sent by the browser, or by file content (magic bytes)? This affects TC03-14, TC03-15 and TC03-18.
4. **Extension variants:** Are `.jpeg`, `.jpe` and upper-case extensions (`.JPG`, `.PNG`) allowed? TC03-05 and TC03-06 assume yes.
5. **Files without extension:** Is a file without an extension rejected, or accepted if its content is a valid JPG or PNG? (TC03-17.)
6. **Error messages:** What exact text or message type is shown for each rejection (size, format, corrupt file, network failure)? Only the presence of an error is tested here.
7. **Image processing:** Is the uploaded image stored as-is, or is it resized, cropped, re-encoded or compressed? Does the 5 MB limit apply to the original file or to the stored result? Does a PNG keep its transparency?
8. **Dimension limits:** Are there minimum or maximum pixel dimensions, or an aspect-ratio rule? (TC03-26.)
9. **Animated PNG (APNG):** Is an animated PNG a valid PNG for this feature?
10. **Empty file:** Is a 0-byte file rejected? No test case is included because the expected result is not defined.
11. **Replacement behaviour:** When a new picture is uploaded, is the old one deleted from storage or kept? Is there a history?
12. **Removing a picture:** Is there a way to delete the profile picture and go back to a default image? Not in the requirement, so no test case is included. What is the default image?
13. **Visibility:** Who can see a user's profile picture: only the owner, logged-in users, or everyone including anonymous visitors? TC03-24 assumes other logged-in users can see it.
14. **Caching:** After a replacement, how soon must the new picture appear to other viewers (instant, or after cache expiry)? Is a browser cache-busting rule required?
15. **Where the picture appears:** The requirement says "shown on the profile page". Must it also appear elsewhere (header, comments, lists)? Only the profile page is tested.
16. **Upload method:** Is drag-and-drop required, or only a file picker? Is camera capture on mobile in scope? Is multiple file selection blocked?
17. **Upload progress:** Is a progress indicator or a cancel option required during upload?
18. **Accessibility:** Are there accessibility requirements for the upload control and the picture (e.g. alt text, keyboard use)?
19. **Supported browsers and devices:** Which browsers and OS versions must be covered?
20. **Malicious content:** Beyond rejecting non-image content (TC03-18), is there a requirement for scanning uploaded images (e.g. malware or EXIF metadata stripping, e.g. GPS data)?
21. **Rate limiting:** Is there a limit on how often a user can upload a picture?
