# HA & LB 설정

## 1. keeplived 설정
---
### 1.1 VIP (Virtual IP)
---
만약 LB+HA 구성을 물리적으로 분리된 서버에 하지 않고 LB의 대상이 되는 서버에 구성하는 경우 VIP(Virtual IP)를 설정할 필요가 있다. 
일단 외부 IP와 바인딩 가능하도록 OS설정 수정이 필요합니다. `/etc/sysctl.conf` 설정 수정이 필요하다.

```
$ echo 'net.ipv4.ip_nonlocal_bind=1' >> /etc/sysctl.conf
$ sysctl -p
```

### 1.2 keepalived 실행 환경
---
keepalived를 설치하면 기본적(설치할 때 path를 지정하지 않으면)으로 `/usr/local/sbin`에 binary 파일을 복사하고 아래의 두 경로에 설정 파일을 복사해 놓는다. 
+ `/usr/local/etc/keepalived/keepalived.conf`: keepalived 기본설정 파일
+ `/usr/local/etc/sysconfig/keepalived`: syslog 관련 설정 파일
+ `/usr/local/etc/sysconfig/keepalived` 내용
    - 변경 전 (설치되면 기본값)
        ```
        KEEPALIVED_OPTION="-D"
        ```
    - 변경 후: 옵션 `-P`는 VRRP(Virtual Router Redundancy Protocol) 서브시스템만 사용하겠다는 의미. 보통 keepalived와 리눅스 LVS(Linux Virtual Server)를 이용해서 LB를 구현하는 것도 가능한데 여기서는 LB는 HAProxy로 구현하고 keepalived의 VRRP 기능만 이용해서 HA 구성을 하기 때문에 필요한 옵션. 
        ```
        KEEPALIVED_OPTION="-D -P -f /usr/local/etc/keepalived/keepalived.conf"
        ```
    
### 1.3 keepalived 1 설정 (Master)
---
아래는 `IP`서버를 **MASTER**로 설정한 keepalived 설정 예제인데 interface(NIC)는 `eno3`이며 VIP는 `IP`로 지정하였다 
```
! Configuration File for keepalived

global_defs {
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_scrpit chk_haproxy {
    script "/usr/bin/killall -0 haproxy"
    interval 2
    weight -4
}

vrrp_instance VI_1 {
    state MASTER
    interface eno1
    virtual_router_id 51
    priority 100
    nopreempt
    unicast_src_ip IP
    unicast_peer {
        IP
    }
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass passAI
    }
    virtual_ipaddress {
        IP
    }
    track_script {
        chk_haproxy
    }

}
```

### 1.4 keeplived 2 설정 (Backup)
---
아래는 `IP`서버를 **BACKUP**로 설정한 keepalived 설정 예제인데 interface(NIC)는 `enp5s0`이며 VIP는 `IP`로 지정하였다 
```
! Configuration File for keepalived

global_defs {
   router_id LVS_DEVEL
   vrrp_skip_check_adv_addr
   vrrp_garp_interval 0
   vrrp_gna_interval 0
}

vrrp_scrpit chk_haproxy {
    scrpit "/usr/bin/killall -0 haproxy"
    interval 2
    weight -4
}

vrrp_instance VI_1 {
    state BACKUP
    interface enp5s0
    virtual_router_id 51
    priority 100
    unicast_src_ip IP
    unicast_peer {
     IP
    }
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass passAI
    }
    virtual_ipaddress {
        IP
    }
    track_script {
        chk_haproxy
    }

}

```
### 1.5 keepalived 실행
---

그리고 시스템 서비스에 필요한 파일 `/usr/lib/systemd/system/keepalived.service`도 설치된다. 파일의 내용은
```
[Unit]
Description=LVS and VRRP High Availability Monitor
After=network-online.target syslog.target 
Wants=network-online.target 

[Service]
Type=forking
PIDFile=/run/keepalived.pid
KillMode=process
EnvironmentFile=-/usr/local/etc/sysconfig/keepalived
ExecStart=/usr/local/sbin/keepalived $KEEPALIVED_OPTIONS
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```

아래를 실행하여 `keepalived` 서비스를 시작한다.
```
systemctl daemon-reload
systemctl start keepalived
```

## 2. HA proxy 실행 및 설정
---
### 2.1 HAProxy 설정
---
+ 설정파일은 아래와 위치하는데 없으면 생성한다. `/etc/haproxy/haproxy.cfg`
+ HAProxy 설정으로 두 서버에 동일하게 설정한다.
+ VIP address를 HA proxy LB 설정의 front로 설정하고 IP의 50051 포트를 bind한다. 여기서 외부 IP와 바인딩 가능하도록 OS설정이 되어 있지 않으면 bind 에러가 발생하면서 정상적으로 기동되지 않는다.

```
#---------------------------------------------------------------------
# Example configuration for a possible web application.  See the
# full configuration options online.
#
#   http://haproxy.1wt.eu/download/1.4/doc/configuration.txt
#
#---------------------------------------------------------------------

#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    log         127.0.0.1:514 local0
    maxconn     50000
    user        
    group       

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    log                     global
    maxconn                 3000
    mode                    http
    option                  http-use-htx
    option                  httplog
    option                  logasap
    timeout connect         10s
    timeout client          30s
    timeout server          30s

#---------------------------------------------------------------------
# main frontend which proxys to the backends
#---------------------------------------------------------------------
frontend tts_front
    bind IP:50051 proto h2
    capture request header in.sessionid len 100
    default_backend tts_servers
 
#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
backend tts_servers
    balance roundrobin
    server  tts1 IP:50051 proto h2 maxconn 100
    server  tts2 IP:50051 proto h2 maxconn 100
```

### 2.2 HAProxy의 실행 
---

`keepalived`와 달리 HAProxy는 실행 관련 설정이나 systemd 서비스 관련 파일 등을 `make install`할 때 생성되지 않으므로 직접 아래와 같이 작업해 주어야 한다.

+ `haproxy.service` 등록: git clone한 경로에서 `contrib/systemd` 아래 `Makefile`을 실행하면 haproxy.service 파일이 생성되는데 이 파일을 `/usr/lib/systemd/system` 경로에 복사한다.
    
    ```
    cd contrib/systemd
    make
    sudo cp haproxy.service /usr/lib/systemd/system
    ```
+ `haproxy.service` 내용
    
    ```
    [Unit]
    Description=HAProxy Load Balancer
    After=network-online.target
Wants=network-online.target
    
    [Service]
    EnvironmentFile=-/etc/default/haproxy
    EnvironmentFile=-/etc/sysconfig/haproxy
    Environment="CONFIG=/etc/haproxy/haproxy.cfg" "PIDFILE=/run/haproxy.pid" "EXTRAOPTS=-S /run/haproxy-master.sock"
    ExecStartPre=/usr/local/sbin/haproxy -f $CONFIG -c -q $EXTRAOPTS
    ExecStart=/usr/local/sbin/haproxy -Ws -f $CONFIG -p $PIDFILE $EXTRAOPTS
    ExecReload=/usr/local/sbin/haproxy -f $CONFIG -c -q $EXTRAOPTS
    ExecReload=/bin/kill -USR2 $MAINPID
    KillMode=mixed
    Restart=always
    SuccessExitStatus=143
Type=notify
    
    [Install]
    WantedBy=multi-user.target
    ```
+ `haproxy` 로그 설정 
    HAProxy 로그는 `rsyslog`를 사용한다.
    
    - `/etc/rsyslog.d` 경로에 `haproxy.conf`를 생성하고 아래를 입력한다.
        
        ```
        # Collect log with UDP
        $ModLoad imudp
    $UDPServerAddress 127.0.0.1
    $UDPServerRun 514
        
        # Creating separate log files based on the severity
        local0.* /var/log/haproxy-traffic.log
        local0.notice /var/log/haproxy-admin.log
        ```
    - `/etc/haproxy/haproxy.cfg` 설정 파일에 Syslog 서버 설정을 지정한다.
        
        ```
        global
            log 127.0.0.1:514  local0 
        ```
+ `haproxy` 실행
    
    ```
    systemctl daemon-reload
    systemctl stop rsyslog
    systemctl stop haproxy
    systemctl start rsyslog
    systemctl start haproxy
    ```
### 2.3 HAProxy 테스트 결과

```log
<134>Nov 13 23:14:09 localhost.localdomain haproxy[24843]: 10.122.65.224:50797 [13/Nov/2020:23:13:52.024] tts_front tts_servers/tts1 0/0/8/17506/+17514 200 +1141 - - ---- 4/4/4/2/0 0/0 {pcm} "POST http://IP/server/Service HTTP/2.0"
```
<img src="https://cdn.haproxy.com/wp-content/uploads/2019/02/image2.png" width="80%">

위의 로그 포맷이 `HTTP log`의 기본 포맷이며 ` 0/0/8/17506/+17514`(`TR/Tw/Tc/Tr/Ta`) 의미는 아래와 같다:

| Timer	| Meaning |
|:------|:--------|
| `TR`	| `The total time to get the client request (HTTP mode only).` |
| `Tw`	| `The total time spent in the queues waiting for a connection slot.` |
| `Tc`	| `The total time to establish the TCP connection to the server.` |
| `Tr`	| `The server response time (HTTP mode only).` |
| `Ta`	| `The total active time for the HTTP request (HTTP mode only).` |
| `Tt`	| `The total TCP session duration time, between the moment the proxy accepted it and the moment both ends were closed.` |

<img src="https://cdn.haproxy.com/wp-content/uploads/2019/02/image3.png" width="80%">
<br>
<br>
