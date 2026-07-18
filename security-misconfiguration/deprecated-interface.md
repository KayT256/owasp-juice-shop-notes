- Search for "B2B" in `main.js`
- See `Input area for uploading a single invoice PDF or XML B2B order file or a ZIP archive containing multiple invoices or orders` in the `complaintMessage` section
- Create a `b2b.pdf`
- Upload it on http://localhost:3000/#/complain
- Send it, nothing happens
- Try with XML
- It does not allow uploading XML
- Upload a normal `b2b.pdf`
- Turn intercept on in burpsuite
- Click Submit.
- See
```
Content-Disposition: form-data; name="file"; filename="b2b.pdf"
Content-Type: application/pdf
```
- Change it to
```
Content-Disposition: form-data; name="file"; filename="b2b.xml"
Content-Type: application/xml
```
(because I see in `main.js` that pdf and zip files are handled by other current logic, while xml hits the old deprecated B2B code path)
- Click forward
- Complete challenge