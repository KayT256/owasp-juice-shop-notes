- I know the format of redirect is `http://localhost:3000/redirect?to=[target_url]` from [[outdated-allowlist]]
- Try going to http://localhost:3000/redirect?to=http://localhost:3000/
- See the error on browser
```
406 Error: Unrecognized target URL for redirect: http://localhost:3000/
```
- That error confirms the redirect endpoint isn't a plain redirect. It's checking the target URL against a hardcoded allowlist of specific, known external URLs before letting the redirect through.
- My suspicion is that it may not use proper URL parsing but just naive string matching (with `startsWith` or `includes`)
```
https://user:pass@sub.example.com:8443/path/to/resource?query=1&foo=bar#section
\___/   \______/ \_______________/\___/\______________/\______________/\_____/
scheme  userinfo      host         port      path            query      fragment
```
- Try modifying this redirect http://localhost:3000/redirect?to=https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm (which I previously know that it works in [[outdated-allowlist]])
	- Try http://localhost:3000/redirect?to=https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm/something - Successfully being redirected to blockain.info
	- Try http://localhost:3000/redirect?to=https://localhost:3000@blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm - See error `Error: Unrecognized target URL for redirect: https://localhost:3000@blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm`
	- Try http://localhost:3000/redirect?to=http://localhost:3000/redirect?to=https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm - Successfully being redirected to blockain.info
	- Try http://localhost:3000/redirect?to=http://localhost:3000/test?a=https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm - No error!
- That confirms my suspicion. It uses something like `url.includes("https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm")`
	- Try http://localhost:3000/redirect?to=http://localhost:3000/%23/administration/test?a=https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm - Redirected to main page (note: I put `%23` instead of `#` because fragment identifier strips everything after it, as done in [[missing-encoding]])
	- Try http://localhost:3000/redirect?to=https://mail.proton.me/u/5/inbox/test?a=https://blockchain.info/address/1AbKfgvw9psQ41NbLi8kufDQTezwG8DRZm - Successfully redirected to proton mail.
- Wait, when the heck the challenge is mark as completed? I didn't even notice (as I didn't see any confetti).
- Anyway, so now I know that the server sent me somewhere it wasn't supposed to allow (like proton mail). I think that this is the completion's condition.
- And back to when I tried being redirected to `/administration` page, since if put literal `#`, everything behind it is stripped away (the browser strips the `#` when talking to the server, but keeps it available for JavaScript running on the client), but if I put `%23` to survive the outer query-parsing step (the input side), it cannot become a real fragment delimiter. A classic `Catch-22s`.
- There must be a backend component to intercept and convert it back to a real `#` in the HTTP response headers. But based on our testing so far, it seems there is none.
