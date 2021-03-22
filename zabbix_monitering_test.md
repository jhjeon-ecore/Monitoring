# Zabbix 구축
1. zabbix-server 설치방법
2. snmp 설치 및 설정, 수집, 비주얼화
3. zabbix-agent 설치 및 설정, 수집, 비주얼화
4. IPMI 설정, 수집, 비주얼화
---
- 구축 환경  
192.168.50.66 ztest1 -> master 서버 (zabbix-server-mysql zabbix-web-mysq 설치)  
192.168.50.67 ztest2 -> 모니터링 대상 서버 (zabbix-agent 설치)  
192.168.50.68 ztest3 -> 모니터링 대상 서버 (zabbix-agent 설치)  
192.168.50.69 ztest4 -> 모니터링 대상 서버 (zabbix-agent 설치)  
- 필수 사항 : /etc/hosts에 각 서버 정보 저장
---

## 1. Zabbix-server 설치(3.x-LTS 버전)
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
# zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -u zabbix -p zabbix
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
8. 기본 사용자 설정  
우측 상단의 사람 모양 아이콘 선택  
Language를 English(en_US)로 수정, Messaging탭에서 Frontend messaging 옵션 클릭, Messaging timeout (seconds)를 180으로 수정
### 호스트 만들기
1. Configuration > hosts로 이동, Create host 선택
2. Host name : ztest1 입력, Groups에 In groups : Linux servers 하나만 남게 설정
3. Add 클릭
### 아이템 만들기
1. 우측 상단 Group에 Linux servers 선택, ztest1 호스트 옆 items 클릭
2. 우측 상단 Create item 클릭  
Name : CPU load  
Key : system.cpu.load   
Type of information : Numeric(float)
3. Add 클릭
4. Monitering > Latest data에 Hosts를 ztest1로 선택하고 Filter 클릭하면 CPU load 아이템 확인 가능
### 심플 그래프 확인
1. CPU load 아이템이 보이는 상태에서 오른쪽 Graph 클릭하면 확인 가능
### 트리거 생성
1. Configuration > hosts로 이동, ztest1 옆 Triggers 클릭
2. 우측 상단 Create trigger 클릭, 다음과 같이 입력 후 Add 클릭  
Name : CPU load too high on ztest1 for last 3 minutes  
Expression : {ztest1:system.cpu.load.avg(180)}>1
3. Monitering > Triggers에서 확인 가능
---

## 2. SNMP 설치 및 설정, 수집, 비주얼화
> ztest2, ztest3, ztest4에 설치
- snmpd 설치 및 설정
```linux
# yum install net-snmp net-snmp-utils  
# vi /etc/snmp/snmpd.conf

    (38번째줄)
    # First, map the community name "public" into a "security name"

    #       sec.name  source          community
    #com2sec notConfigUser  default       public    -> 주석처리
    com2sec notConfigUser   default      zabbix     -> 추가

    ####
    # Second, map the security name into a group name:

    #       groupName      securityModel securityName
    group   notConfigGroup v1           notConfigUser
    group   notConfigGroup v2c           notConfigUser

    ####
    # Third, create a view for us to let the group have rights to:
    a
    # Make at least  snmpwalk -v 1 localhost -c public system fast again.
    #       name           incl/excl     subtree         mask(optional)
    view    systemview    included   .1.3.6.1.2.1.1
    view    systemview    included   .1.3.6.1.2.1.25.1.1
    view    systemview    included   .1       -> 추가
# systemctl start snmpd
# systemctl enable snmpd
# snmpstatus -v 2c -c zabbix 192.168.50.67 -> 자신의 ip입력(또는 snmp 사용 가능한 다른 호스트 ip)
    (연결상태가 정상이면 다음과 같이 출력됨)
    [UDP: [192.168.50.67]:161->[0.0.0.0]:45251]=>[Linux ztest2 3.10.0-1127.el7.x86_64 #1 SMP Tue Mar 31 23:36:51 UTC 2020 x86_64] Up: 1:01:17.69
    Interfaces: 2, Recv/Trans packets: 18995/17985 | IP: 70501/58786
# snmpwalk -v 2c -c zabbix 192.168.50.67 | head -n 6
    SNMPv2-MIB::sysDescr.0 = STRING: Linux ztest2 3.10.0-1127.el7.x86_64 #1 SMP Tue Mar 31 23:36:51 UTC 2020 x86_64
    SNMPv2-MIB::sysObjectID.0 = OID: NET-SNMP-MIB::netSnmpAgentOIDs.10
    DISMAN-EVENT-MIB::sysUpTimeInstance = Timeticks: (70537) 0:11:45.37
    SNMPv2-MIB::sysContact.0 = STRING: Root <root@localhost> (configure /etc/snmp/snmp.local.conf)
    SNMPv2-MIB::sysName.0 = STRING: ztest2
    SNMPv2-MIB::sysLocation.0 = STRING: Unknown (edit /etc/snmp/snmpd.conf)
```
- 자빅스에서 snmp 호스트 생성  
    - 자빅스 웹페이지 Configuration > Host 에서 Create host 클릭  
        - Host name : ztest2_snmp (snmp와 agent를 구별하기 위해 임의로 설정함)  
        - Groups - New group : SNMP devices (임의)  
        - Agent interfaces는 오른쪽 Remove로 삭제
        - SNMP interfaces에 Add 클릭 후 ztest2의 ip 입력  
        - 상단 Host 옆 Templates 클릭 - Link new templates의 Select 클릭 -> Template SNMP OS Linux 선택 후 Add 클릭  
        - 상단 Templates 옆 Macro 클릭 - Macro에 {$SNMP_COMMUNITY} 입력, Value에 zabbix 입력
        - 하단 Add 클릭으로 호스트 생성
        - 생성한 ztest2_snmp클릭 후 하단에 Full clone 클릭하여 Hostname과 SNMP interfaces만 변경하여 ztest3_snmp, ztest4_snmp 생성
    - 트래픽 아이템 만들어보기(생략가능)   
        새로 만든 호스트 옆 item 클릭, Create item -> 다음 내용 입력 후 하단 Add클릭  
        Name : Outgoing traffic on interface $1    
        Type : SNMPv2 agent   
        Key : ifOutOctets[ens192]  
        SNMP OID : ifOutOctets["index","ifDescr","ens192"]  
        SNMP commuvity : zabbix  
        Units : Bps  
        Store value : Delta (speed per second)  
        -> Monitering > Lastest data에서 트래픽 정보 표시되면 성공  
    - Monitering > Graphs로 들어가서 Group : SNMP devices, Host : (원하는목록선택), Graphs : (원하는목록선택) 하면 그래프 볼 수 있음
---   
## 2-1. Network 스위치 모니터링(SNMP)
### snmp 설정
- 스위치 연결, 원격 접속 후 (스위치에서 진행)
```
# configure terminal
(config)# snmp-server community v2c zabbix ro
(config)# exit
# copy running-config startup-config
```
- 다른 리눅스 서버에서 스위치 연결 확인
```linux
# snmpstatus -v 2c -c zabbix (switch ip)
# snmpwalk -v 2c -c zabbix (switch ip)
```
    
### zabbix-server -> network 스위치 수집   
- 자빅스에서 snmp 호스트 생성  
    - 자빅스 웹페이지 Configuration > Host 에서 Create host 클릭  
        - Host name : test_switch (임의로 설정함)  
        - Groups - In group : SNMP devices  
        - Agent interfaces는 오른쪽 Remove로 삭제
        - SNMP interfaces에 Add 클릭 후 스위치의 ip 입력  
        - 상단 Host 옆 Templates 클릭 - Link new templates의 Select 클릭 -> Template SNMP Device 선택 후 Add 클릭  
        - 상단 Templates 옆 Macro 클릭 - Macro에 {$SNMP_COMMUNITY} 입력, Value에 zabbix 입력
        - 하단 Add 클릭으로 호스트 생성
       
## 3. zabbix-agent 설치 및 설정, 수집, 비주얼화
- ztest2~4에 zabbix-agent 설치
```linux
# yum install epel-release
# rpm -ivh http://repo.zabbix.com/zabbix/3.0/rhel/7/x86_64/zabbix-release-3.0-1.el7.noarch.rpm
# yum install zabbix-agent
# systemctl start zabbix-agent
# systemctl enable zabbix-agent
``` 
- zabbix 웹페이지 설정에 새 호스트 추가  
Configuration > Hosts 에서 Create host 클릭  
    Host name : ztest2  
    Agent interfaces IP address : 192.168.50.67 입력  
    -> 하단 Add 클릭으로 호스트 생성 (ztest3, ztest4도 동일하게 ip바꿔서 입력)
- zabbix_agentd.conf 파일 수정(ztest2~4에서 진행)  
```linux
# vi /etc/zabbix/zabbix_agentd.conf
Server=192.168.50.66      ->자빅스 서버ip 입력, 95번째 줄
ServerActive=192.168.50.66      ->자빅스 서버ip 입력, 136번째 줄
Hostname=ztest2      ->자신의 hostname 입력, 147번째 줄
# systemctl restart zabbix-agent
```
- ztest1에서 zabbix_agentd.conf 파일 수정
```linux
# vi /etc/zabbix/zabbix_agentd.conf
Hostname=ztest1      ->자신의 hostname 입력, 147번째 줄
# systemctl restart zabbix-agent
```
---
- 자빅스에서 agent 호스트 생성  
    - 자빅스 웹페이지 Configuration > Host 에서 Create host 클릭  
        - Host name : ztest2 (snmp와 agent를 구별하기 위해 임의로 설정함)  
        - Groups - In groups : Linux servers (임의)  
        - Agent interfaces : ztest2의 ip 입력   
        - Agent interfaces에 Add 클릭 후 ztest2의 ip 입력   
        - 상단 Host 옆 Templates 클릭 - Link new templates의 Select 클릭 -> Template OS Linux 선택 후 Add 클릭   
        - 하단 Add 클릭으로 호스트 생성
        - 생성한 ztest2_snmp클릭 후 하단에 Full clone 클릭하여 Hostname과 Agent interfaces만 변경하여 ztest3, ztest4 생성
    - Monitering > Graphs로 들어가서 Group : SNMP devices, Host : (원하는목록선택), Graphs : (원하는목록선택) 하면 그래프 볼 수 있음
## 4. IPMI 설치 및 설정, 수집, 비주얼화   
> 주어진 IPMI 정보(ID:root/PW:calvin)   
192.168.1.222   svr01-m   
192.168.1.223   svr02-m   
192.168.1.224   svr03-m   
192.168.1.225   svr04-m   
- IPMI 모니터링 준비 : IPMI 폴러가 시작되도록 설정   
```linux
# vi /etc/zabbix/zabbix_server.conf
StartIPMIPollers=0 (147번째 줄, 주석 제거 후 3으로 값 변경)

# systemctl restart zabbix-server
```
- IPMI 아이템 설정   
```linux
# yum install ipmitool
# ipmitool -U root -H 192.168.1.222 -I lanplus -L user sdr -P calvin
[root@ztest1 ~]# ipmitool -U root -H 192.168.1.222 -I lanplus -L user sensor get "Inlet Temp" -P calvin
Locating sensor record...
Sensor ID              : Inlet Temp (0x5)
 Entity ID             : 7.1
 Sensor Type (Threshold)  : Temperature
 Sensor Reading        : 20 (+/- 1) degrees C
 Status                : ok
 Lower Non-Recoverable : na
 Lower Critical        : -7.000
 Lower Non-Critical    : 3.000
 Upper Non-Critical    : 28.000
 Upper Critical        : 32.000
 Upper Non-Recoverable : na
 Positive Hysteresis   : 2.000
 Negative Hysteresis   : 2.000
 Assertion Events      : 
 Assertions Enabled    : lnc- lcr- unc+ ucr+ 
 Deassertions Enabled  : lnc- lcr- unc+ ucr+
```
> U:username, H:IPMI host의 ip입력, P:IPMI 설정에서 설정한 암호 입력   
- IPMI 모니터링   
    - 자빅스 서버에서 IPMITOOL 패키지 설치 및 zabbix_server.conf파일 편집   
    ```linux
    # rpm -qa ipmitool   
    ipmitool-1.8.18-9.el7_7.x86_64
    # vi /etc/zabbix/zabbix_server.conf  
      ### Option: StartIPMIPollers
      #       Number of pre-forked instances of IPMI pollers.
      #
      # Mandatory: no
      # Range: 0-1000
      # Default:
      StartIPMIPollers=5            //147번째 줄
    # systemctl restart zabbix-server
    ```
    - 자빅스 서버에서 ipmi 센서 목록 가져오기(ipmi ip입력)
    ```linux
    # ipmitool -I lanplus -H 192.168.1.222 -U root -P calvin sensor
    # ipmitool -I lanplus -H 192.168.1.223 -U root -P calvin sensor
    # ipmitool -I lanplus -H 192.168.1.224 -U root -P calvin sensor
    # ipmitool -I lanplus -H 192.168.1.225 -U root -P calvin sensor
    ```
    - 자빅스 웹페이지 Configuration > Host 에서 Create host 클릭  
        - Host name : IPMI host  
        - Visible name : svr01-m(다른 IPMI host와 구별 위해)   
        - Groups - In groups : Linux servers (임의)  
        - Agent interfaces : Remove버튼으로 삭제   
        - IPMI interfaces에 Add 클릭 후 IPMI host의 ip 입력      
        - 상단 Host 옆 Templates 클릭 - Link new templates 오른쪽 Select 클릭 -> Template IPMI Intel SR1630 클릭 후 하단 Select 클릭 -> 하단 파란색 Add 클릭   
        - 상단 Host 옆옆 IPMI 클릭 - Username과 Password에 IPMI host의 정보 기입 (root/calvin)
        - 하단 Add 클릭으로 host 생성  
    - Inlet Temp 아이템 생성   
        - svr01-m의 Applications 클릭 -> Create application 클릭 -> Name : IPMI 입력 후 Add -> svr01-m의 Items 클릭 -> Create item 클릭 -> 다음과 같이 입력 후 하단의 Add 클릭 
        ```    
            Name : Temperature   
            Type : IPMI agent   
            Key : techexpert.ipmi.temp   
            IPMI sensor : Inlet Temp   
            Type of information : Numeric (float)   
            Units : degrees C   
            Update interval (in sec) : 30   
            Applications : IPMI
        ```
    - Inlet Temp 그래프 생성   
        - svr01-m의 Graphs 클릭 -> Create Graph 클릭 -> Name : Inlet Temp 입력 -> 하단 Items에 위에서 만든 아이템(Temperature) 선택 후 Select -> 하단의 Add 클릭
    - Monitering > Graphs로 들어가서 Group : SNMP devices, Host : (원하는목록선택), Graphs : (원하는목록선택) 하면 그래프 볼 수 있음
   ---
## 추가-1   
> 부하발생 명령어 stress 이용   
- 설치 방법(yum 설치)
    ```linux
    # yum install epel-release
    # yum install stress
    ```
- CPU 부하   
stress -c (코어 수)   
예시) 코어 5개에 부하   
    ```linux
    # stress -c 5 
    ```
- Memory 부하   
stress --vm (프로세스 수) --vm-bytes (크기)   
예시) 2G만큼씩 부하를 1개의 워커로 수행   
    ```linux
    # stress --vm 1 --vm-bytes 2g   
    ```
- HDD 부하   
stress --hdd (하드 수) -hdd-bytes (크기)   
예시) HDD 3개를 1024m만큼 부하   
    ```linux
    # stress --hdd 3 --hdd-bytes 1024m --timeout 60s
    ```
## 추가-2   
> CPU, MEM, Network, DISK 80% 이상 발생 시 event 발생 트리거 생성   
- Disk Trigger : Template OS Linux ->Discovery rules -> Mounted filesystem discovery의 Trigger prototyles -> "Free disk space is less than 20% on volume {#FSNAME}"란 이름으로 20% 아래일시 알람 뜨게 되어있음.   
- Network Trigger :  Template OS Linux ->Discovery rules -> Network interface discovery의 Trigger prototypes -> Create trigger prototype 클릭   
    name : income network traffic over 200Mbps    
    Expression : {Template OS Linux:net.if.in[{#IFNAME}].avg(5m)}>200M   
    Severity : Warning
- Memory Trigger : Template OS Linux -> Trigger    
    name : Lack of available memory on server {HOST.NAME} 클릭하여 내용 수정      
    Expression : {Template OS Linux:vm.memory.size[available].last(0)}/{Template OS Linux:vm.memory.size[total].last(0)}*100<20
- CPU Trigger : Template OS Linux -> Trigger -> Create trigger 클릭   
    name : CPU less than 20 Trigger   
    Expression : {Template OS Linux:system.cpu.util[,idle].avg(1m)}<20   
    Severity : Warning
