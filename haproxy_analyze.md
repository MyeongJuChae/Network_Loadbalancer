## HAProxy 로그 분석 참고 링크
https://www.haproxy.com/blog/introduction-to-haproxy-logging/
***
## ghz load test
95번 서버
/home/mindslab/tts-load
```
ghz --insecure --async --proto ng_tts.proto --call maum.brain.tts.NgTtsService/SpeakWav -B ./TtsRequest.bin -n 100 --rps 2 10.122.64.191:50051
`--insecure`: Use plaintext and insecure connection.
`--async`: Make requests asynchronous as soon as possible. Does not wait for request to finish before sending next one.
`--proto`: The path to The Protocol Buffer .proto file for input
`--call`: A fully-qualified method name in 'package.Service/Method' or 'package.Service.Method' format. For example, `maum.brain.tts.NgTtsService/SpeakWav` 
`-B`: Path for the call data as serialized binary message. The format is the same as for -b switch.
`-n`: The total number of requests to run. Default is 200. The combination of -c and -n are critical in how the benchmarking is done. ghz takes the -c argument and spawns that many worker goroutines. In parallel these goroutines each do their share (n / c) requests. So for example with the default -c 50 -n 200 options we would spawn 50 goroutines which in parallel each do 4 requests.
`--rps`: Rate limit in how many requsts per second (RPS) we perform in total. Default is no rate limit. The total RPS will be distributed among all the workers as specified by concurrency options.

```
## Log 위치

```
/sdn/maum/logs/supervisor (tail -f sdn.out)
/var/log/haproxy-traffic.log (tail -f haproxy-traffic.log)
```

***
## haproxy-log-analysis python package
- https://pypi.org/project/haproxy-log-analysis/
