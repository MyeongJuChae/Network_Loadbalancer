## 정상 상태 (keeapliaved, haproxy 모두 running)
| | Master(191) | Backup(95) |
| ---: | ---: | ---: |
| keepalived | running | running |
| HAProxy | running | running |
- HAProxy log 191번
***

1. Master(191) keepalived stop

| | Master(191) | Backup(95) |
| ---: | ---: | ---: |
| keepalived | running -> **stop** | running |
| HAProxy | running | running |
- 95번이 Master로 바뀜. 191번은 Backup 으로
- HAProxy log 95번 으로
***
2. Backup이 된 191번 keepalived start

| | Backup(191) | Master(95) |
| ---: | ---: | ---: |
| keepalived | stop -> **start** | running |
| HAProxy | running | running |
- 191번 Backup state 유지
- HAproxy log 95번 유지
***
3. Master가 된 95번 keepalived stop

| | Backup(191) | Master(95) |
| ---: | ---: | ---: |
| keepalived | running | running -> **stop** |
| HAProxy | running | running |
- 191번 Master로 바뀜. 
- HAproxy log 191번으로.

**keepalived 는 Master 와 Backup 역할 수행이 잘 됨.**
***
4. Master(191번) HAProxy stop

| | Master(191) | Backup(95) |
| ---: | ---: | ---: |
| keepalived | running | running |
| HAProxy | running -> **stop** | running |
- Master(191)의 HAProxy log 정지
- 다시 Master(191)의 HAProxy start시 191번에 HAProxy log

**HAProxy 만으로는 Backup 역할이 안 되는 듯**
***
5. Master(191번) keepalived stop, Backup(95번) HAProxy stop

| | Master(191) | Backup(95) |
| ---: | ---: | ---: |
| keepalived | running -> **stop** | running |
| HAProxy | running | running -> **stop** |
- Master가 된 95의 Haproxy log  정지
- 다시 Master(95)의 HAProxy **start**시 95번에 HAProxy log
