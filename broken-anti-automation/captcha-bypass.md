- Open burp
- Go to http://localhost:3000/#/contact
- Fill all information (with correct answer for CAPTCHA)
- Click submit
- Go to HTTP history
- Send this to intruder
```
POST /api/Feedbacks/ HTTP/1.1
[...]
{"captchaId":8,"captcha":"-7","comment":"this is bad (anonymous)","rating":1}
```
- See that it uses `captchaId` to know what captcha and the answer `captcha` will correspond to that `captchaId`. In other words, the same `captchaId` will always have the same `captcha` answer - meaning I can send the same request again and again.
- Send the request to Intruder
- Add a random `§` anywhere
```
{"captchaId":8,"captcha":"-7","comment":"this is bad (anonymous)","rating":1}§§
```
- Choose null payloads
- Choose Generate 20 payloads
- Complete challenge