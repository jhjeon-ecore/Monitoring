# Zabbix 구축
1. zabbix-server 설치방법
2. snmp 설치 및 설정, 수집, 비주얼화
3. zabbix-agent 설치 및 설정, 수집, 비주얼화
4. IPMI 설정, 수집, 비주얼화
---
## Zabbix-server 설치(3.x-LTS 버전)
### 패키지 설치
- epel 설치
```linux
# yum install epel-release
```
- 자빅스 저장소 추가
```linux
# rpm -ivh http://repo.zabbix.com/zabbix/3.0/rhel/7/x86_64/zabbix-release-3.0-1.el7.noarch.rpm
```
- 자빅스 패키지 설치
```linux
# yum install zabbix-server-mysql zabbix-web-mysql zabbix-agent

=====================================================================================================================================
 Package                              Arch                Version                            Repository                         Size
=====================================================================================================================================
Installing:
 zabbix-agent                         x86_64              3.0.31-1.el7                       zabbix                            348 k
 zabbix-server-mysql                  x86_64              3.0.31-1.el7                       zabbix                            1.8 M
 zabbix-web-mysql                     noarch              3.0.31-1.el7                       zabbix                            6.8 k
Installing for dependencies:
 OpenIPMI                             x86_64              2.0.27-1.el7                       base                              243 k
 OpenIPMI-libs                        x86_64              2.0.27-1.el7                       base                              523 k
 OpenIPMI-modalias                    x86_64              2.0.27-1.el7                       base                               16 k
 apr                                  x86_64              1.4.8-7.el7                        base                              104 k
 apr-util                             x86_64              1.5.2-6.el7                        base                               92 k
 dejavu-fonts-common                  noarch              2.33-6.el7                         base                               64 k
 dejavu-sans-fonts                    noarch              2.33-6.el7                         base                              1.4 M
 fontpackages-filesystem              noarch              1.44-8.el7                         base                              9.9 k
 fping                                x86_64              3.10-4.el7                         epel                               46 k
 gnutls                               x86_64              3.3.29-9.el7_6                     base                              680 k
 httpd                                x86_64              2.4.6-97.el7.centos                updates                           2.7 M
 httpd-tools                          x86_64              2.4.6-97.el7.centos                updates                            93 k
 iksemel                              x86_64              1.4-2.el7.centos                   zabbix-non-supported               49 k
 libX11                               x86_64              1.6.7-3.el7_9                      updates                           607 k
 libX11-common                        noarch              1.6.7-3.el7_9                      updates                           164 k
 libXau                               x86_64              1.0.8-2.1.el7                      base                               29 k
 libXpm                               x86_64              3.5.12-1.el7                       base                               55 k
 libjpeg-turbo                        x86_64              1.2.90-8.el7                       base                              135 k
 libxcb                               x86_64              1.13-1.el7                         base                              214 k
 libzip                               x86_64              0.10.1-8.el7                       base                               48 k
 mailcap                              noarch              2.1.41-2.el7                       base                               31 k
 net-snmp-libs                        x86_64              1:5.7.2-49.el7_9.1                 updates                           751 k
 nettle                               x86_64              2.7.1-8.el7                        base                              327 k
 php                                  x86_64              5.4.16-48.el7                      base                              1.4 M
 php-bcmath                           x86_64              5.4.16-48.el7                      base                               58 k
 php-cli                              x86_64              5.4.16-48.el7                      base                              2.7 M
 php-common                           x86_64              5.4.16-48.el7                      base                              565 k
 php-gd                               x86_64              5.4.16-48.el7                      base                              128 k
 php-ldap                             x86_64              5.4.16-48.el7                      base                               53 k
 php-mbstring                         x86_64              5.4.16-48.el7                      base                              506 k
 php-mysql                            x86_64              5.4.16-48.el7                      base                              102 k
 php-pdo                              x86_64              5.4.16-48.el7                      base                               99 k
 php-xml                              x86_64              5.4.16-48.el7                      base                              126 k
 t1lib                                x86_64              5.1.2-14.el7                       base                              166 k
 trousers                             x86_64              0.3.14-2.el7                       base                              289 k
 unixODBC                             x86_64              2.3.1-14.el7                       base                              413 k
 zabbix-web                           noarch              3.0.31-1.el7                       zabbix                            2.3 M

Transaction Summary
=====================================================================================================================================
Install  3 Packages (+37 Dependent packages)
```
- MariaDB 설치  
```linux
# yum install mariadb-server
# systemctl start mariadb
```
### 초기 설정
- zabbix_server.conf 파일변경  
    DBName -> zabbix   
    DBUser -> zabbix  
    DBPassword -> DB에서 사용할 암호 입력  (1234로 설정함)
```linux
# vi /etc/zabbix/zabbix_server.conf
### Option: DBName
#       Database name.
#
# Mandatory: yes
# Default:
# DBName=

DBName=zabbix  

### Option: DBUser
#       Database user.
#
# Mandatory: no
# Default:
# DBUser=

DBUser=zabbix  

### Option: DBPassword
#       Database password.
#       Comment this line if no password is used.
#
# Mandatory: no
# Default:
DBPassword=1234
```
### DB 생성 및 import  
자빅스 서버가 데이터를 저장할 db생성
```linux
[root@ztest1 ~]# mysql -u root
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 6
Server version: 5.5.68-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database zabbix character set utf8 collate utf8_bin;
Query OK, 1 row affected (0.01 sec)

MariaDB [(none)]> grant all privileges on zabbix.* to 'zabbix'@'localhost' identified by '1234';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> quit
Bye
```
> 1234는 DBPassword에 설정한 비밀번호와 동일하게 입력  
> 생성된 db에 자빅스 스키마와 초기 데이터를 import
```linux
# zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
```
### 실행
자빅스 실행할 계정 생성 및 패키지 실행
```linux
# useradd -m -s /bin/bash zabbix
# systemctl restart zabbix-server zabbix-agent httpd
# systemctl enble zabbix-server zabbix-agent httpd
```
서비스 상태 확인
```linux
# netstat -ntpl
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp6       0      0 :::10050                :::*                    LISTEN      26018/zabbix_agentd 
tcp6       0      0 :::10051                :::*                    LISTEN      26025/zabbix_server
```
### 웹 프론트엔드 설정
> PHP로 작성된 자빅스 웹 프론트엔드  
> (PHP 버전 5.4.0 이상이어야 함)  
#### 브라우저 실행 후 http://<server_ip>/zabbix 로 이동  
1. welcome  
'Welcome to Zabbix 3.0'창이 뜨는 것을 확인할 수 있다. Next step 클릭하여 다음으로 넘어간다.
2. PHP prerequisites
PHP 관련 필수 요구 패키지 확인. Fail 을 OK로 수정 후 httpd 재실행한 다음 Next step 클릭한다.  
PHP option "memory_limit"  
PHP option "post_max_size"  
PHP option "upload_max_filesize"  
PHP option "max_execution_time"  
PHP option "max_input_time"  
PHP option "date.timezone"  
위 사항은 모두 php.ini 설정값을 변경하면 된다. (Fail인 항목만 다음 설정값을 참고하여 수정, 모두 수정할 필요 X)
    ```linux
    # vi /etc/php.ini
    memory_limit = 128M
    post_max_size = 10M
    max_execution_time = 300
    max_input_time = 300
    upload_max_filesize = 2M
    date.timezone = Asia/Seoul
    ```
    수정 후 httpd 재시작 후 새로고침. 모두 OK 확인하고 Next step 클릭
3. database access  
Datebase name과 User는 모두 zabbix로, Password는 설정한 값(1234)로 입력 후 Next step 클릭  
4. Zabbix server details  
Name 값은 자빅스 화면 우측 상단의 페이지 제목과 레이블에 사용되므로 설정해준다. (Zabbix one이라고 임의로 설정하였음)
5. summary  
설정된 값이 제대로 구성됐는지 확인 후 Next step 클릭  
6. writing the configuration file  
'Congratulations! You have successfully installed Zabbix frontend.'가 화면에 뜬다면 설치 완료. Finish 클릭  
7. logging in  
기본 계정 정보(Username : Admin / Password : zabbix)로 로그인  
대시보드 창이 뜨면 웹 프론트엔드 설정 완료 & 로그인 성공

