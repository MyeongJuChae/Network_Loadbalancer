## keepalived
1. -n 1200 --rps 20

**stop Backup** -> sdn, halog에서는 문제 없음.
95번 서버에서는 아래와 같은 에러 발생
```
1185 ok
15 Unavailable
error distribution:
rpc error : code Unvailable desc = transport is closing
```
**stop Master** -> sdn, halog에서는 문제 없음.
95번 서버에서는 아래와 같은 에러 발생
```
1184 ok
16 Unavailable
error distribution:
rpc error : code Unavailable desc = transport is closing
```
