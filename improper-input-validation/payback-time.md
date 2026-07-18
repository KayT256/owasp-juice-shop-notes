- I see that there are 2 payment methods: `Card` and `Digital wallet`
- Sending money back to the `Card` may not (?) be feasible at the moment, so increasing the `Digital wallet` balance is a much better choice
- Pay for an order from start to finish with the `Digital wallet` to see the traffic
- See that there are mostly GET requests at first, but when clicking "Place your order and Pay", there is a POST request
```
POST /rest/basket/2/checkout HTTP/1.1
[...]
{"couponData":"bnVsbA==","orderDetails":{"paymentId":"wallet","addressId":"4","deliveryMethodId":"1"}}
```
- Send it to Repeater
- Try removing the `paymentId` field
- The order is still completed
- And the wallet balance is not reduced! But the challenge is not completed.
- Earlier, I was trying to find a field like `totalPrice` or `quantity` to put a negative value, so when the math do the subtraction, it will add money to my wallet balance instead of reducing it.
- Let's try to manipulate the `BasketItems` endpoint (which I know about through [[manipulate-basket]]) earlier so the quantity is negative
```
POST /api/BasketItems/ HTTP/1.1
[...]
{"ProductId":1,"BasketId":"2","quantity":-10}
```
- Oh, it says `ProductId` must be unique, so I will change it too.
```
POST /api/BasketItems/ HTTP/1.1
[...]
{"ProductId":10,"BasketId":"2","quantity":-10}
```
- Succeed. 
- Go to the basket. See `Total Price: -297.90999999999997`
- Now, do the checkout again.
- Complete the challenge.