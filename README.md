# TTS HA와 Load-balacning 구성
TTS HA(High Availability) load balancing 관련 검토 결과 HAProxy + keepalived가 가장 적당한 solution. 

Nginx와 keepalived도 많이 사용하는 구성이나 keepalived와 같이 구성하려면 Nginx Plus를 사용해야 함. Nginx Plus는 상용 버전으로 instance당 라이선스가 4백만원 정도 되는 듯.

## 1. HAProxy 설치
www.haproxy.org 홈페이지 참고.
Centos 7.5에서 yum install -y haproxy 로 설치된 haproxy의 버전은 1.5.18임 -> grpc를 지원하기 위해서는 버전 2.0 이상이어야 함.

결국 소스를 다운로드 받아서 build해야 함. 현재 stable 버전은 2.3, 2.3 git repo는 아래와 같음
http://git.haproxy.org/git/haproxy-2.3.git/

git clone후 haproxy-2.3 디렉토리의 INSTALL 문서 참고

```
cd HAProxy
git clone http://git.haproxy.org/git/haproxy-2.3.git
cd haproxy-2.3 
sudo yum install -y systemd-devel --> AICC 도커 환경에서 dependency 설치 필요
sudo yum install -y openssl-devel --> AICC 도커 환경에서 dependency 설치 필요
make -j $(nproc) TARGET=linux-glibc USE_OPENSSL=1 USE_ZLIB=1 USE_PCRE=1 USE_SYSTEMD=1
--> USE_LUA=1은 make 옵션에서 삭제하고 빌드함. 빌드 성공
sudo make install
```
+ make install의 결과
```
[mindslab@localhost haproxy-2.3]$ sudo make install
`haproxy' -> `/usr/local/sbin/haproxy'
`doc/haproxy.1' -> `/usr/local/share/man/man1/haproxy.1'
install: creating directory `/usr/local/doc/haproxy'
`doc/configuration.txt' -> `/usr/local/doc/haproxy/configuration.txt'
`doc/management.txt' -> `/usr/local/doc/haproxy/management.txt'
`doc/seamless_reload.txt' -> `/usr/local/doc/haproxy/seamless_reload.txt'
`doc/architecture.txt' -> `/usr/local/doc/haproxy/architecture.txt'
`doc/peers-v2.0.txt' -> `/usr/local/doc/haproxy/peers-v2.0.txt'
`doc/regression-testing.txt' -> `/usr/local/doc/haproxy/regression-testing.txt'
`doc/cookie-options.txt' -> `/usr/local/doc/haproxy/cookie-options.txt'
`doc/lua.txt' -> `/usr/local/doc/haproxy/lua.txt'
`doc/WURFL-device-detection.txt' -> `/usr/local/doc/haproxy/WURFL-device-detection.txt'
`doc/proxy-protocol.txt' -> `/usr/local/doc/haproxy/proxy-protocol.txt'
`doc/linux-syn-cookies.txt' -> `/usr/local/doc/haproxy/linux-syn-cookies.txt'
`doc/SOCKS4.protocol.txt' -> `/usr/local/doc/haproxy/SOCKS4.protocol.txt'
`doc/network-namespaces.txt' -> `/usr/local/doc/haproxy/network-namespaces.txt'
`doc/DeviceAtlas-device-detection.txt' -> `/usr/local/doc/haproxy/DeviceAtlas-device-detection.txt'
`doc/51Degrees-device-detection.txt' -> `/usr/local/doc/haproxy/51Degrees-device-detection.txt'
`doc/netscaler-client-ip-insertion-protocol.txt' -> `/usr/local/doc/haproxy/netscaler-client-ip-insertion-protocol.txt'
`doc/peers.txt' -> `/usr/local/doc/haproxy/peers.txt'
`doc/close-options.txt' -> `/usr/local/doc/haproxy/close-options.txt'
`doc/SPOE.txt' -> `/usr/local/doc/haproxy/SPOE.txt'
`doc/intro.txt' -> `/usr/local/doc/haproxy/intro.txt'
```

참고로 인터넷 환경이 없을 경우 아래와 같이 package를 미리 받아 둘 수 있다. 
```
sudo yum install --downloadonly --downloaddir=. openssl-devel
sudo yum install --downloadonly --downloaddir=. systemd-devel
```
설치 후 현재경로에서 아래를 확인

```
[minds@localhost haproxy-2.3]$ ./haproxy -v
HA-Proxy version 2.3.0-3a1071-2 2020/11/06 - https://haproxy.org/
Status: stable branch - will stop receiving fixes around Q1 2022.
Known bugs: http://www.haproxy.org/bugs/bugs-2.3.0.html
Running on: Linux 3.10.0-1127.19.1.el7.x86_64 #1 SMP Tue Aug 25 17:23:54 UTC 2020 x86_64
```
haproxy는 /usr/local/sbin에 설치됨.


## 2. Keepalived 설치
stable 2.1.5 버전 설치

```
cd Keepalived
git clone https://github.com/acassen/keepalived.git
```

아래 dependency 패키지를 먼저 설치
```
sudo yum install -y openssl-devel libnl3-devel ipset-devel
sudo yum install -y iptables-devel
sudo yum install -y libnftnl-devel libmnl-devel
```

git local 디렉토리로 이동해서 빌드와 설치

```
cd keepalived
./build_setup --> 이걸 먼저 실행해야 configure가 생김
./configure
make
sudo make install
```

/usr/local/sbin에 keepalived가 설치됨

/usr/local/etc 아래에 keepalived.conf가 위치함


### 3. HAProxy와 Keepalived offline 설치- 인터넷 환경이 없을 때
금융권 프로젝트의 경우 패쇄망을 사용하기 때문에 yum install로 필요한 패키지를 설치할 수 없기 떄문에 rpm 패키지를 미리 받아놓고 설치할 필요가 있다.
HAProxy와 Keepalived 경로에 필요한 RPM 패키지가 포함되어 있다. 각각의 경로로 이동하여 아래의 명령어로 필요 소프트웨어를 설치한다.
```
rpm -Uvh --force --nodeps *.rpm
(or)
rpm -ivh --force --nodeps *.rpm
```
`-U`는 업데이트된 패키지가 있으면 받아와서 설치하는 옵션인데 인터넷 환경이 없으면 `-i` 옵션과 동일하다.
+ HAProxy 필수 소프트웨어 설치 및 haproxy 빌드
	```
	cd HAProxy
	ls -al
	rpm -ivh --force --nodeps *.rpm
	cd haproxy-2.3
	make -j $(nproc) TARGET=linux-glibc USE_OPENSSL=1 USE_ZLIB=1 USE_PCRE=1 USE_SYSTEMD=1
	sudo make install
	```
+ Keepalived 필수 소프트웨어 설치 및 keepalived 빌드
```
cd Keepalived
ls -al
rpm -ivh --force --nodeps *.rpm
cd keepalived
./build_setup 
./configure
make
sudo make install
```


