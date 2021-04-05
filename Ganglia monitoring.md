# Ganglia monitoring 구축
- ganglia master 서버 (web, 수집) : 1대(gmetad, gmond)
- gnaglia client (모니터링 대상) : 그 외(gmond)
- 구성도   
    - OpenAccess Grid   
        - Management cluster(host 3개)   
            **o2-mgt01** : 192.168.102.21   
            o2-big01 ~ o2-big02    
        - OA Computing cluster(host 30개)   
            **o2-com01**  : 192.168.102.101   
            o2-com02 ~ o2-com30    
    - o2-mgt01 : gmetad, gmond   
    그 외 : gmond   
    > 굵게 처리된 host는 각 클러스터 대표 서버
---
## 0. 설치
### 1. yum 패키지 설치
- httpd 및 epel-release 설치 및 확인
```linux
# rpm -qa httpd
httpd-2.2.15-69.el6.centos.x86_64
# yum install epel-release
# rpm -qa epel-release
epel-release-6-8.noarch
```
- ganglia 관련 패키지 설치 (master 서버)
```linux
# yum install ganglia rrdtool ganglia-gmetad ganglia-gmond ganglia-web
```
> client 서버는 yum install ganglia ganglia-gmond
### 2. 오프라인 설치 (rpm 설치 순서)
- rpm -Uvh (rpm파일명)으로 설치   
    cpp, libgcc   
    libgomp 먼저 설치 -> gcc 설치   
    libstdc++, libstdc++-devel 먼저 설치 -> gcc-c++ 설치   
    apr -> apr-util -> pcre 설치 후    
    libconfuse -> libconfuse-devel -> ganglia   
    ganglia-devel, ganglia-gmond 설치   
    yum install rrdtool -> ganglia-gmetad   
    yum install httpd   
    +) php, php-gd, php-ZendFramework 사전 설치돼야 ganglia-web설치 가능   
    php-common -> yum listall libedit -> php-cli -> php   
    yum install libXpm, yum install libjpeg-turbo -> php-gd    (yum install로 다운 XXXXX, rpm -Uvh로 5.3.3-50버전으로 설치)   
    +) php-common-5.3.3-50 버전으로 다운(그 하위버전 다운돼있으면 rpm -e php-common --nodeps 으로 지우고 설치)   
    yum install libxslt->php-xml(yum install로 다운 XXXXX, rpm -Uvh로 5.3.3-50버전으로 설치)   
    php-bcmath , php-process -> php-ZendFramework(얘만 e17버전으로 설치 가능, e16버전 못찾음)    
    -> ganglia-web 설치
## 1. ganglia master 서버 : o2-mgt01
### 1. 설정 파일 수정
- /etc/httpd/conf.d/ganglia.conf 수정
```linux
# vi /etc/httpd/conf.d/ganglia.conf 
Alias /ganglia /usr/share/ganglia

<Location /ganglia>
  Order deny,allow
  Allow from all
  #Deny from all
  #Allow from 127.0.0.1
  #Allow from ::1
  # Allow from .example.com
</Location>
```
- gmetad.conf 수정
```linux
data_source "Management Cluster" o2-mgt01
data_source "OA Computing Cluster" o2-com01

gridname "OpenAccess"

authority "http://192.168.102.21/ganglia/"     //생략가능
```
- gmond.conf 수정
```linux
cluster {
  name = "Management Cluster"
  owner = "unspecified"
  latlong = "192.168.102.21"           //master서버(o2-mgt) ip
  url = "unspecified"
}
host {
  location = "192.168.102.21"           //master서버(o2-mgt) ip
}
udp_send_channel {
  mcast_join = 192.168.102.21           //master서버(o2-mgt) ip
  port = 8649
  ttl = 1
}
udp_recv_channel {
  #mcast_join = 239.2.11.71
  port = 8649
  #bind = 239.2.11.71
}
tcp_accept_channel {
  port = 8649
}
```
### 2. httpd, gmetad, gmond 재시작
```linux
# service httpd stop
# service httpd start
# service gmetad stop
# service gmetad start
# service gmond stop
# service gmond start
```
## 2. ganglia management 클러스터 : o2-big01, o2-big02 (동일)
### 1. gmond.conf 수정
```linux
cluster {
  name = "Management Cluster"
  owner = "unspecified"
  latlong = "192.168.102.21"           //master서버(o2-mgt) ip
  url = "unspecified"
}
host {
  location = "192.168.102.21"           //master서버(o2-mgt) ip
}
udp_send_channel {
  mcast_join = 192.168.102.21           //master서버(o2-mgt) ip
  port = 8649
  ttl = 1
}
udp_recv_channel {
  #mcast_join = 239.2.11.71
  port = 8649
  #bind = 239.2.11.71
}
tcp_accept_channel {
  port = 8649
}
```
### 2. gmond 재시작
```linux
# service gmond stop
# service gmond start
```
## 3. ganglia OA Computing 클러스터 : o2-com01 ~ o2-com30 (동일)
### 1. gmond.conf 수정
```linux
cluster {
  name = "OA Computing Cluster"
  owner = "unspecified"
  latlong = "192.168.102.101"           //cluster 대표 서버(o2-com01) ip
  url = "unspecified"
}
host {
  location = "192.168.102.101"           //cluster 대표 서버(o2-com01) ip
}
udp_send_channel {
  mcast_join = 192.168.102.101"           //cluster 대표 서버(o2-com01) ip
  port = 8649
  ttl = 1
}
udp_recv_channel {
  #mcast_join = 239.2.11.71
  port = 8649
  #bind = 239.2.11.71
}
tcp_accept_channel {
  port = 8649
}
```
### 2. gmond 재시작
```linux
# service gmond stop
# service gmond start
```
