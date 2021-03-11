#증설 관련 정보
----
## 장비 IP address
---
+ 기존장비 IP: 
+ 증설장비 IP: 
+ VIP: 

## 서비스 포트
- waveglow

- tacotron2

- core
 
- custom

- sdn


## root 권한으로 작업 실행
지금부터 진행하는 작업은 모두 root 권한으로 작업해야 함.

## haproxy, keepalived 설정 복사
ha_config.tar를 반입하고 아래의 명령으로 지정 경로에 복사되도록 한다.
```
tar xvf ha_config.tar -C /
```
tar에 압축된 파일의 정보
```
/usr/local/etc/keepalived/keepalived.conf.master
/usr/local/etc/keepalived/keepalived.conf.backup
/usr/local/etc/sysconfig/keepalived
/usr/lib/systemd/system/keepalived.service
/etc/haproxy/haproxy.cfg
/etc/rsyslog.d/haproxy.conf
/usr/lib/systemd/system/haproxy.service
```

## 설정 변경
+ 장비를 MASTER로 하고 장비를 BACKUP으로 한다.
+ 각 장비의 keepalived 설정 경로로 이동하여 
```
cd /usr/local/etc/keepalived
cp keepalived.conf.master keepalived.conf
(or)
cp keepalived.conf.backup keepalived.conf
```

## 커널 설정 변경
외부 IP와 바인딩할 수 있도록 /etc/sysctl.conf 설정 변경
```
echo 'net.ipv4.ip_nonlocal_bind=1' >> /etc/sysctl.conf
sysctl -p
```

## 서비스 등록 및 서비스 시작
keepalived와 haproxy 서비스 스크립트를 등록하고 서비스를 시작한다.
```
systemctl list-units --type service
systemctl enable keepalived.service
systemctl enable haproxy.service
systemctl list-units --type service

systemctl start keepalived.service (.service 생략 가능)
systemctl status keepalived
systemctl start haproxy
systemctl status haproxy
```

## 서비스 설정 변경 후 reload
keepalived 또는 haproxy 관련 설정이 변경된 경우 서버를 재시작하지 않고 설정변경만 하고 싶은 경우 아래의 명령을 사용
```
systemctl reload haproxy
systemctl reload keepalived
```
예를 들어,
+ haproxy 설정 관련해서 haproxy stats 페이지 확인을 위해 /etc/haproxy/haproxy.cfg에 아래 라인을 추가했을 경우
```
(추가된 설정)
#---------------------------------------------------------------------
# frontend for stats
#---------------------------------------------------------------------
frontend stats
    bind *:8404
    stats enable
    stats uri /stats
    stats refresh 10s
    stats admin if LOCALHOST

(기존 설정)
#---------------------------------------------------------------------
# main frontend which proxys to the backends
#---------------------------------------------------------------------
frontend tts_front
    bind IP proto h2
    capture request header in.sessionid len 100
    default_backend tts_servers
```

+ haproxy 서비스를 리로드한다.
```
systemctl reload haproxy
```
+ haproxy 통계 페이지에 접속한다. 웹브라우즈 주소창에 아래를 입력
```
http:/IP/stats
```

## HAProxy의 Health check
backend 서버의 liveness를 체크하기 위해 health check을 사용해야 한다.
HARroxy는 HTTP, TCP 프로토콜 레벨의 http-check, tcp-check 기능을 지원하는데 grpc는 따로 지원하지 않아 external-check을 사용해서 사용자 script를 실행하는 것으로 제공해야 한다. 아래 링크 참고.
[링크] https://www.loadbalancer.org/blog/how-to-write-an-external-custom-healthcheck-for-haproxy/
[링크] https://www.haproxy.com/blog/announcing-haproxy-2-2/
insecure-fork-wanted 내용 참고

external-check haproxy.cfg 관련 설정
```
global
    log         127.0.0.1:514 local0
    maxconn     50000
    insecure-fork-wanted   #추가
    external-check              #추가
    user        
    group       

(...생략...)
backend tts_servers
    option  external-check   #추가
    external-check command /srv/check/check-tts.sh   #추가
    balance roundrobin
    #server  tts1  proto h2 maxconn 100 
    #server  tts2  proto h2 maxconn 100 
    server  tts1  proto h2 maxconn 100 check inter 10000   #변경
    server  tts2  proto h2 maxconn 100 check inter 10000   #변경

```
## Health check 스크립트 내용
헬쓰 체크 스크립트는 /src/check에 위치하며 처음 설치하는 경우 /srv 경로에서 아래의 tar파일을 풀어서 설치한다.
```
cd /srv
tar xvf tts-check.tar
```
스크립트는 haproxy 설정에 의해 check-tts.sh가 호출되면 tts-check.py(간단한 TTS grpc client)를 실행해서 TTS 서비스의 정상운영 여부(Up or Down)을 판단한다.
```bash
-rwxrwxr-x. 1    626  1월 20 10:48 check-tts.sh
-rw-rw-r--. 1   1117  1월 19 19:51 ng_tts.proto
-rw-rw-r--. 1  13852  1월 19 19:51 ng_tts_pb2.py
-rw-rw-r--. 1   7640  1월 19 19:52 ng_tts_pb2.pyc
-rw-rw-r--. 1   6743  1월 19 19:51 ng_tts_pb2_grpc.py
-rw-rw-r--. 1   5496  1월 19 19:52 ng_tts_pb2_grpc.pyc
-rw-rw-r--. 1  30856  1월 20 10:51 test00.wav
-rwxrwxr-x. 1   1640  1월 20 09:33 tts-check.py

```

## Firewall 관련 명령어(참고용)
keepalived 동작 관련 문제가 있을 경우 firewalld 서비스를 확인할 필요가 있음
아래 명령어들을 참고해서 troubleshooting할 것

```
systemctl start firewalld
systemctl enable firewalld
firewall-cmd state
firewall-cmd --state
firewall-cmd --list-all
firewall-cmd --permanent --zone=public --add-port=80/tcp
firewall-cmd --list-all
firewall-cmd --reload
firewall-cmd --list-all
firewall-cmd --permanent --zone=public --add-port=5060/tcp
firewall-cmd --permanent --zone=public --add-port=5080/udp
firewall-cmd --reload
firewall-cmd --list-all
firewall-cmd --permanent --zone=public --add-port=5060/udp
firewall-cmd --permanent --zone=public --add-port=5080/tcp
firewall-cmd --reload
firewall-cmd --list-all
```

