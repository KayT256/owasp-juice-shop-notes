- Go to http://localhost:3000/#/register
- Enter all details like normal
- Turn Intercept on in burp suite
- Click submit
- See
```
[...]
{"email":"test@gmail.com","password":"Test123#","passwordRepeat":"Test123#","securityQuestion":{"id":1,"question":"Your eldest siblings middle name?","createdAt":"2026-07-03T14:21:50.548Z","updatedAt":"2026-07-03T14:21:50.548Z"},"securityAnswer":"a"}
```
- Remove all details
```
[...]
{"email":"","password":"","passwordRepeat":"","securityQuestion":{"id":1,"question":"","createdAt":"","updatedAt":""},"securityAnswer":""}
```
- Click forward
- Complete challenge.