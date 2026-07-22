- See eastere.gg in http://localhost:3000/ftp
- Just like [[forgotten-developer-backup]], use Poison Null Byte injection
- Go to http://localhost:3000/ftp/eastere.gg%2500.md
- See a file downloaded
```
"Congratulations, you found the easter egg!"
- The incredibly funny developers

...

...

...

Oh' wait, this isn't an easter egg at all! It's just a boring text file! The real easter egg can be found here:

L2d1ci9xcmlmL25lci9mYi9zaGFhbC9ndXJsL3V2cS9uYS9ybmZncmUvcnR0L2p2Z3V2YS9ndXIvcm5mZ3JlL3J0dA==

Good luck, egg hunter!
```
- Complete the challenge
- But I would like to see what the real easter egg is.
- This for mat is base64 so decode with https://www.offlineutils.com/base64
- See `/gur/qrif/ner/fb/shaal/gurl/uvq/na/rnfgre/rtt/jvguva/gur/rnfgre/rtt`
- It looks like ROT13 (because `gur` is a major flag, which is the `the` in English. `the` is the most common 3 letters word in English - glad I did an exercise on this at school before!)
- Decode it and get the `/the/devs/are/so/funny/they/hid/an/easter/egg/within/the/easter/egg`
- Go to http://localhost:3000/the/devs/are/so/funny/they/hid/an/easter/egg/within/the/easter/egg
- Complete the [[nested-easter-egg]] challenge.