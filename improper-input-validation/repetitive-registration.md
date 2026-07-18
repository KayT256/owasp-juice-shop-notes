- Follow the DRY principle while registering a user.
- It seems that this challenge need me to tamper with the Password and Repeat Password field.
- Open Burp suite -> Proxy -> Intercept on
- Edit the POST Request
```
[...]
{"email":"test@gmail.com","password":"Test123#","passwordRepeat":"Test123#","securityQuestion":{"id":1,"question":"Your eldest siblings middle name?","createdAt":"2026-07-01T13:49:51.076Z","updatedAt":"2026-07-01T13:49:51.076Z"},"securityAnswer":"Test"}
```
- Change `passwordRepeat":"Test123#"` to `passwordRepeat":"Test1234#"`
- Complete challenge.