## HAProxy 로그 분석 참고 링크
https://www.haproxy.com/blog/introduction-to-haproxy-logging/
***
## ghz load test
95번 서버
/home/mindslab/tts-load
```
ghz --insecure --async --proto ng_tts.proto --call maum.brain.tts.NgTtsService/SpeakWav -B ./TtsRequest.bin -n 100 --rps 2 10.122.64.119:50051
`--insecure`: Use plaintext and insecure connection.
`--async`: Make requests asynchronous as soon as possible. Does not wait for request to finish before sending next one.
`--proto`: The path to The Protocol Buffer .proto file for input
`--call`: A fully-qualified method name in 'package.Service/Method' or 'package.Service.Method' format. For example, `maum.brain.tts.NgTtsService/SpeakWav` 
`-B`: Path for the call data as serialized binary message. The format is the same as for -b switch.
`-n`: The total number of requests to run. Default is 200. The combination of -c and -n are critical in how the benchmarking is done. ghz takes the -c argument and spawns that many worker goroutines. In parallel these goroutines each do their share (n / c) requests. So for example with the default -c 50 -n 200 options we would spawn 50 goroutines which in parallel each do 4 requests.
`--rps`: Rate limit in how many requsts per second (RPS) we perform in total. Default is no rate limit. The total RPS will be distributed among all the workers as specified by concurrency options.

```
***
## Log 위치

```
/var/log/haproxy-traffic.log (tail -f haproxy-traffic.log)
/srv/maum/logs/supervisor (tail -f sdn.out)
```

***
## halog
명령어
```
cd ~/HAProxy/haproxy-2.3/contrib/halog 
halog 파일이 없을 경우 make

./halog -srv -H < /var/log/haproxy-traffic.log | column -t
./halog -ut -H < /var/log/haproxy-traffic.log | column -t

termination code
-- : normal
cD : 클라이언트에서 
SC : 서버에서 TCP 거부 (연결 끊김)
PC : process socket의 limit이 다 참.

-H HTTP로그 (TCP 제외)
-E 에러 빼고 보기(5xx status 제외)
-e 에러만 보기(5xx or negative status)
-rt|-RT <time> <time> 값보다 큰 | <time> 값보다 작은
-Q|-QS 큐 리퀘스트 (모든 큐 | 서버 큐)
-tcn|-TCN <code> termnation code (포함 | 미포함)
-hs|-HS <[min][:][max]> 추후 정리
-time <[min][:max]> 시간 범위

halog [-h|--help] for long help
       halog [-q] [-c] [-m <lines>]
       {-cc|-gt|-pct|-st|-tc|-srv|-u|-uc|-ue|-ua|-ut|-uao|-uto|-uba|-ubt|-ic}
       [-s <skip>] [-e|-E] [-H] [-rt|-RT <time>] [-ad <delay>] [-ac <count>]
       [-v] [-Q|-QS] [-tcn|-TCN <termcode>] [ -hs|-HS [min][:[max]] ] [ -time [min][:[max]] ] < log
      
Input filters (several filters may be combined) :
 -H                      only match lines containing HTTP logs (ignore TCP)
 -E                      only match lines without any error (no 5xx status)
 -e                      only match lines with errors (status 5xx or negative)
 -rt|-RT <time>          only match response times larger|smaller than <time>
 -Q|-QS                  only match queued requests (any queue|server queue)
 -tcn|-TCN <code>        only match requests with/without termination code <code>
 -hs|-HS <[min][:][max]> only match requests with HTTP status codes within/not
                         within min..max. Any of them may be omitted. Exact
                         code is checked for if no ':' is specified.
 -time <[min][:max]>     only match requests recorded between timestamps.
                         Any of them may be omitted.
Modifiers
 -v                      invert the input filtering condition
 -q                      don't report errors/warnings
 -m <lines>              limit output to the first <lines> lines
 -s <skip_n_fields>      skip n fields from the beginning of a line (default 5)
                         you can also use -n to start from earlier then field 5

Output filters - only one may be used at a time
 -c    only report the number of lines that would have been printed
 -pct  output connect and response times percentiles
 -st   output number of requests per HTTP status code
 -cc   output number of requests per cookie code (2 chars)
 -tc   output number of requests per termination code (2 chars)
 -srv  output statistics per server (time, requests, errors)
 -ic   output statistics per ip count (time, requests, errors)
 -u*   output statistics per URL (time, requests, errors)
       Additional characters indicate the output sorting key :
       -u : by URL, -uc : request count, -ue : error count
       -ua : average response time, -ut : average total time
       -uao, -uto: average times computed on valid ('OK') requests
       -uba, -ubt: average bytes returned, total bytes returned


```
## haproxy-log-analysis python package
- https://pypi.org/project/haproxy-log-analysis/
