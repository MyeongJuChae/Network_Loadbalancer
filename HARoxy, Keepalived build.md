### HAProxy 
***
rpm install 시
- Failed to get D-Bus connection: Operation not permitted

$ docker run -d --name centos7.4_postgres --privileged -it -e container=docker -v /sys/fs/cgroup:/sys/fs/cgroup:ro centos:latest /usr/sbin/init
_위 내용은 docker container에서 실행시 발생하는 오류_

### Keepalived
***
./configure
- Warning - this build will not support IPVS with IPv6. Please install libn;/libnl-3 dev libraries to support IPv6 with IPVS

make, make install
- warning 및 error로 안 됨

_실수로 ha-install container의 keepalived를 /home/mindslab/git/maum_ha 에 덮어씀. ./configure, make, make install 모두 잘 됨_

***
VIP : 
```
/var/log messeges
```
reserved IP : 127.0.0.1:514
```
aicc/src/sdn
source /venv/bin/activate
tail -f haproxy-trfic.log
```
