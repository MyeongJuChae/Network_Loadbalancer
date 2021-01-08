## 95번 /home/mindslab/git/maum_ha의 source를 그대로 사용함

### HAProxy
***
rpm install 시
- Failed to get D-Bus connection: Operation not permitted

$ docker run -d --name centos7.4_postgres --privileged -it -e container=docker -v /sys/fs/cgroup:/sys/fs/cgroup:ro centos:latest /usr/sbin/init

### Keepalived
***
./configure
- Warning - this build will not support IPVS with IPv6. Please install libn;/libnl-3 dev libraries to support IPv6 with IPVS

make, make install
- warning 및 error로 안 됨

_실수로 ha-install container의 keepalived를 /home/mindslab/git/maum_ha 에 덮어씀. ./configure, make, make install 모두 잘 됨_
