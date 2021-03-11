## TTS 증설 trouble shooting
***
1. 증설 서버에서 keepalived 작동 안 됨. 

/usr/lib/systemd/system/keepalived.service 변경

[SERVICE]

Type=~~notify~~ -> Type=**forking**
***

2. HAProxy VIP port trouble(haproxy.cfg)
- 기존
```
frontend tts_front
    bind 10.211.70.247:50052 proto h2
    capture request header in.sessionid len 100
    default_backend tts_servers
```
50052 상태에서 haproxy restart에서 문제 발생. binding 실패

port를 **50053**으로 바꿈

- 변경 후
```
frontend tts_front
    bind IP:50053 proto h2
    capture request header in.sessionid len 100
    default_backend tts_servers
```
***
3. HAProxy health check
tts-check.py 파일의 line 33 `f.write` 명령어 권한 관련 문제 발생.
```
    for tts in resp:
      f.write(tts.mediaData)
    f.close()
```
증설 서버의 디렉토리 권한이 redhat/redhat 으로 되어 있어 health check 실패.

**-> chown 으로 해결(...)**