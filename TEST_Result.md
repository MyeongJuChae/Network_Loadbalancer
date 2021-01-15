## keepalived
1. -n 1200 --rps 20

**stop Backup(95번)** -> sdn, halog에서는 문제 없음.
95번 서버에서는 아래와 같은 에러 발생
```
1185 ok
15 Unavailable
error distribution:
rpc error : code Unvailable desc = transport is closing
```
**stop Master(191번)** -> sdn, halog에서는 문제 없음.
95번 서버에서는 아래와 같은 에러 발생
```
1184 ok
16 Unavailable
error distribution:
rpc error : code Unavailable desc = transport is closing
```

## HAProxy
1. -n 1200 --rps 20

**stop haproxy(191번 tts1)**
```
request_ok 는 분할이 잘 되나 tts1의 avg_rt이 폭발적으로 커짐

```
2. -n 1600 --rps 40
**stop haproxy(191번 tts1)-> start haproxy-> stop haproxy**
**haproxy 멈춘 부분에서 tts2(166)으로 넘어감**
```
1460 ok
140 DeadlineExceeded
error distribution:
rpc error: code = DeadlineExceeded desc = context deadline exceeded
```
