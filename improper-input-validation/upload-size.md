- Go to http://localhost:3000/#/complain
- If uploading a file > 100KB, it immediately displays `File too large. Maximum 100 KB allowed.` This is client side check.
- Create a `b2b.pdf` file with any content (`Test123`)
- Input any message
- Click Submit
- See the Request
```
POST /file-upload HTTP/1.1
[...]
------WebKitFormBoundary4HhLnNfRCgWjvjQD
Content-Disposition: form-data; name="file"; filename="b2b.pdf"
Content-Type: application/pdf

Test123

------WebKitFormBoundary4HhLnNfRCgWjvjQD--
```
- Perfect. Now, I can just edit that field to anything I want.
- Send the request to Repeater
- Create a pdf of 1MB `head -c 1048576 /dev/urandom > 1MB.pdf`
- Use burp `Paste text from file` to insert the file binart (replacing the existing `Test123`) to the request
- Click Send
- See
```
HTTP/1.1 500 Internal Server Error
[...]
MulterError: File too large
[...]
```
- So not only does it validate the size on client size, it also does so server side because `Multer` is Node.js middleware Express apps commonly use to handle `multipart/form-data` file uploads.
- But, let's try a file with 100,001 bytes because the Multer limits may be different from that of client-side's limit `head -c 100001 /dev/urandom > 100KB-and-1B.pdf`
- Insert that into burp request. 
- Click send
- See
```
HTTP/1.1 204 No Content
[...]
```
- Complete the challenge. This means that the `Multer` limit is higher than that of client side's.