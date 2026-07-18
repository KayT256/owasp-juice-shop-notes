- Since I already have access to [[admin-section]], I know that Bjoern's email is `bjoern.kimminich@gmail.com` and `bjoern@owasp.org`
- The target is Bjoern's OWASP account `bjoern@owasp.org`
- Check to see if I can still stalk for sensitive data exposure like I did earlier with [[meta-geo-stalking]] and [[visual-geo-stalking]] but no luck
- Check the `Users` schema I got earlier from [[password-strength]] but do not see any security question field
- Look at the table names I got earlier from [[password-strength]]. Bingo, found a table named `SecurityAnswers`
- Do sql injection to see its schema `')) UNION SELECT 1, 2, 3, sql, 5, 6, 7, 8, 9 FROM sqlite_master WHERE tbl_name='SecurityAnswers'--`
- See the schema
```
CREATE TABLE `SecurityAnswers` (`UserId` INTEGER UNIQUE REFERENCES `Users` (`id`) ON DELETE NO ACTION ON UPDATE CASCADE, `SecurityQuestionId` INTEGER REFERENCES `SecurityQuestions` (`id`) ON DELETE NO ACTION ON UPDATE CASCADE, `id` INTEGER PRIMARY KEY AUTOINCREMENT, `answer` VARCHAR(255), `createdAt` DATETIME NOT NULL, `updatedAt` DATETIME NOT NULL)
```
- Now, do `')) UNION SELECT 1, 2, answer, 4, 5, 6, 7, 8, 9 FROM SecurityAnswers WHERE UserId='13'--` (I know his id is 13 because I have access to [[admin-section]])
- See
```
{"id":1,"name":"2","description":"20a257391db163bf0149fe64d04236548fc30cddae8304417d9dae791e78c910","price":4,"deluxePrice":5,"image":6,"createdAt":7,"updatedAt":8,"deletedAt":9},
```
- Use https://md5.gromweb.com/ to reverse the MD5 hash with rainbow table
- But this time, I am not as lucky as when I did with [[password-strength]]
- This hash does not exist in the rainbow table (or it may even be salted)
- It seems I should go back to social stalking
- Search for Bjoern Kimminich
- Try the first video
- [BeNeLux Day 2018: Juice Shop: OWASP's Most Broken Flagship - Björn Kimminich](https://youtu.be/Lu0-kDdtVf4?si=wtaU0rC9lWjJdO7X)
- Copy the link and paste it to `notebooklm.google.com` and see where he talks about his pet
- Bingo, he mentioned the pet's name at 4:30, which is `Zaya`
- Enter the pet's name and change the password to `Bjoern'sPassword123`
- Complete the challenge