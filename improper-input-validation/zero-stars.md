- Go to http://localhost:3000/#/contact
- Enter details and any rating
- Open burpsuite and turn intercept on
- Click submit. See the POST request
```
[...]

{"captchaId":1,"captcha":"12","comment":"Super bad! (anonymous)","rating":1}
```
- Change `{"captchaId":1,"captcha":"12","comment":"Super bad! (anonymous)","rating":1}` to `{...,"rating":0}`
- Click forward
- Complete the challenge
