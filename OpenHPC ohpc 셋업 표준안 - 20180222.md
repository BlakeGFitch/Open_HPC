# 다산데이타 OpenHPC 1.3 (Centos 7.4) 셋업 표준안 (2018-02)

\# 참조 링크 : http://openhpc.community/
\# Root 로 로그인하여 설치를 시작 합니다.  

***

## # 1. 변수 정의 및 적용 (파일로 작성)

```bash
vi ~/dasan_ohpc_variable.sh

```

### # '~/dasan_ohpc_variable.sh' 파일 내용.
```bash
#!/bin/bash

# 클러스터 이름.
CLUSTER_NAME=OpenHPC-Dasandata # 변경 필요

# MASTER 의 이름 과 IP.
MASTER_HOSTNAME=$(hostname)
MASTER_IP=10.1.1.1
MASTER_PREFIX=24

# 인터페이스 이름.
EXT_NIC=em2 # 외부망.
INT_NIC=p1p1 # 내부망.

# NODE 의 이름, 수량, 사양.
NODE_NAME=node
NODE_NUM=${전체 노드 수}
NODE_RANGE="[1-3]"  # 전체 노드가 3개일 경우 1-3 / 5대 일 경우 [1-5]

# NODE 의 CPU 사양에 맞게 조정.
# 물리 CPU가 2개 이고, CPU 당 코어가 10개, 하이퍼스레딩은 켜진(Enable) 상태 인 경우.  
SOCKETS=2          ## 물리 CPU 2개
CORESPERSOCKET=10  ## CPU 당 코어 10개
THREAD=2           ## 하이퍼스레딩 Enable

# 노드 배포 이미지 경로.
export CHROOT=/opt/ohpc/admin/images/centos7.4

# end of file.
```

### # 변수 적용.
```bash
source  ~/dasan_ohpc_variable.sh

```

***

## # 2. Network, Firewall Setup.

### # 2-1. 외부망 및 내부망 인터페이스 설정.

```bash
ip a    # 인터페이스 목록 확인

```
출력 예)
> 1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1  
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00  
    inet 127.0.0.1/8 scope host lo  
       valid_lft forever preferred_lft forever  
2: em1: <BROADCAST,MULTICAST> mtu 1500 qdisc mq state DOWN qlen 1000  
    link/ether ================ brd ff:ff:ff:ff:ff:ff  
3: *em2*: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP qlen 1000  
    link/ether ================ brd ff:ff:ff:ff:ff:ff  
    inet *192.168.0.116/24* brd 192.168.0.255 scope global em2  
       valid_lft forever preferred_lft forever  
4: em3: <BROADCAST,MULTICAST> mtu 1500 qdisc mq state DOWN qlen 1000  
    link/ether ================ brd ff:ff:ff:ff:ff:ff  
5: em4: <BROADCAST,MULTICAST> mtu 1500 qdisc mq state DOWN qlen 1000  
    link/ether ================ brd ff:ff:ff:ff:ff:ff  
6: *p1p1*: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 1000  
    link/ether ================ brd ff:ff:ff:ff:ff:ff  
7: p1p2: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 1000  
    link/ether ================ brd ff:ff:ff:ff:ff:ff  

### # 2-2. Master 의 외부/내부 인터페이스 설정내용 확인
```bash
cat /etc/sysconfig/network-scripts/ifcfg-${EXT_NIC}

```
출력 예)
>NAME=*em2*  
ONBOOT=yes  
BOOTPROTO=none  
IPADDR=192.168.0.116  
PREFIX=24  
GATEWAY=192.168.0.1  
DNS1=168.126.63.1  
DNS2=8.8.8.8  
DEFROUTE=yes  
<일부 값 생략>  

```bash
cat /etc/sysconfig/network-scripts/ifcfg-${INT_NIC}

```
출력 예)
>NAME=*p1p1*  
ONBOOT=no  
BOOTPROTO=dhcp  
<일부 값 생략>  

### # 2-3. 가독성 향상을 위해, 불 필요한 IPV6 항목 삭제.
```bash
sed -i '/IPV6/d' /etc/sysconfig/network-scripts/ifcfg-${EXT_NIC}
sed -i '/IPV6/d' /etc/sysconfig/network-scripts/ifcfg-${INT_NIC}

```

### # 2-4. Master 의 내부망 인터페이스의 설정 변경.
```bash
perl -pi -e 's/BOOTPROTO=dhcp/BOOTPROTO=none/' /etc/sysconfig/network-scripts/ifcfg-${INT_NIC}
perl -pi -e 's/ONBOOT=no/ONBOOT=yes/' /etc/sysconfig/network-scripts/ifcfg-${INT_NIC}
```

### # 2-5. Master 의 내부망 ip 설정
```bash
echo "IPADDR=${MASTER_IP}"  >>  /etc/sysconfig/network-scripts/ifcfg-${INT_NIC}
echo "PREFIX=${MASTER_PREFIX}"  >>  /etc/sysconfig/network-scripts/ifcfg-${INT_NIC}

cat /etc/sysconfig/network-scripts/ifcfg-${INT_NIC}

```

### # 2-6. ip 변경 설정 적용
```bash
ifdown ${INT_NIC} && ifup ${INT_NIC}

ip a

```
출력 예)
>1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1  
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00  
    inet 127.0.0.1/8 scope host lo  
       valid_lft forever preferred_lft forever  
2: em1: <BROADCAST,MULTICAST> mtu 1500 qdisc mq state DOWN qlen 1000  
    link/ether ================= brd ff:ff:ff:ff:ff:ff  
3: em2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP qlen 1000  
    link/ether ================= brd ff:ff:ff:ff:ff:ff  
    inet 192.168.0.116/24 brd 192.168.0.255 scope global em2  
       valid_lft forever preferred_lft forever  
4: em3: <BROADCAST,MULTICAST> mtu 1500 qdisc mq state DOWN qlen 1000  
    link/ether ================= brd ff:ff:ff:ff:ff:ff  
5: em4: <BROADCAST,MULTICAST> mtu 1500 qdisc mq state DOWN qlen 1000  
    link/ether ================= brd ff:ff:ff:ff:ff:ff  
6: *p1p1*: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state *UP* qlen 1000  
    link/ether ================= brd ff:ff:ff:ff:ff:ff  
    *inet 10.1.1.1/24* brd 10.1.1.255 scope global p1p1  
       valid_lft forever preferred_lft forever  
7: p1p2: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN qlen 1000  
    link/ether ================= brd ff:ff:ff:ff:ff:ff  

### # 2-7. 방화벽 설정 변경
```bash
firewall-cmd --change-interface=${EXT_NIC}  --zone=external  --permanent
firewall-cmd --change-interface=${INT_NIC}  --zone=trusted   --permanent
firewall-cmd --reload

firewall-cmd --list-all --zone=external
```
출력 예)
>*external* (active)  
target: *default*  
icmp-block-inversion: no  
interfaces: *em2*  

```bash
firewall-cmd --list-all --zone=trusted

```
출력 예)
>*trusted* (active)  
  target: ACCEPT  
  icmp-block-inversion: no  
  interfaces: em1 *p1p1*  

***

## # 3. Install OpenHPC Components

### # 3-1. Enable OpenHPC repository for local use
#### # 현재 repolist 확인
```bash
yum repolist

```

출력 예)  
>Loaded plugins: fastestmirror, langpacks, priorities  
Loading mirror speeds from cached hostfile  
 \* base: data.nicehosting.co.kr  
 \* epel: mirror01.idc.hinet.net  
 \* extras: data.nicehosting.co.kr  
 \* updates: data.nicehosting.co.kr  
116 packages excluded due to repository priority protections  
repo id             repo name                                         status  
!base/7/x86_64      CentOS-7 - Base                                        9,591  
!epel/x86_64        Extra Packages for Enterprise Linux 7 - x86_64    12,182+116  
!extras/7/x86_64    CentOS-7 - Extras                                        388  
!updates/7/x86_64   CentOS-7 - Updates                                     1,929  
repolist: 24,090  

#### # Install to OpenHPC repository.
```bash
yum -y install \
http://build.openhpc.community/OpenHPC:/1.3/CentOS_7/x86_64/ohpc-release-1.3-1.el7.x86_64.rpm \
>> ~/dasan_log_ohpc_openhpc_repository.txt

tail ~/dasan_log_ohpc_openhpc_repository.txt
```

출력 예)  
> Running transaction test  
Transaction test succeeded  
Running transaction  
  Installing : ohpc-release-1.3-1.el7.x86_64                                1/1   
  Verifying  : ohpc-release-1.3-1.el7.x86_64                                1/1   
Installed:  
  ohpc-release.x86_64 0:1.3-1.el7                                               
Complete!  

#### # repolist 확인

```bash
yum repolist
```
출력 예)
>Loaded plugins: fastestmirror, langpacks, priorities  
OpenHPC                                                  | 1.6 kB     00:00     
OpenHPC-updates                                          | 1.2 kB     00:00     
(1/3): OpenHPC/group_gz                                    | 1.7 kB   00:00     
(2/3): OpenHPC/primary                                     | 155 kB   00:01     
(3/3): OpenHPC-updates/primary                             | 192 kB   00:01     
Loading mirror speeds from cached hostfile  
 \* base: data.nicehosting.co.kr  
 \* epel: mirror.rise.ph  
 \* extras: data.nicehosting.co.kr  
 \* updates: data.nicehosting.co.kr  
OpenHPC                                                                 821/821  
OpenHPC-updates                                                       1010/1010  
139 packages excluded due to repository priority protections  
repo id             repo name                                         status  
OpenHPC             OpenHPC-1.3 - Base                                   321+500  
OpenHPC-updates     OpenHPC-1.3 - Updates                                428+582  
base/7/x86_64       CentOS-7 - Base                                        9,591  
epel/x86_64         Extra Packages for Enterprise Linux 7 - x86_64    12,182+116  
extras/7/x86_64     CentOS-7 - Extras                                        388  
updates/7/x86_64    CentOS-7 - Updates                                     1,929  
repolist: 24,839  


### # 3-2. NTP Server 설정

```bash
cat /etc/ntp.conf | grep -v "#\|^$"

```
출력 예)
>driftfile /var/lib/ntp/drift  
restrict default nomodify notrap nopeer noquery  
restrict 127.0.0.1   
restrict ::1  
server 0.centos.pool.ntp.org iburst  
server 1.centos.pool.ntp.org iburst  
server 2.centos.pool.ntp.org iburst  
server 3.centos.pool.ntp.org iburst  
includefile /etc/ntp/crypto/pw  
keys /etc/ntp/keys  
disable monitor  

```bash
echo "server time.bora.net" >> /etc/ntp.conf

cat /etc/ntp.conf | grep -v "#\|^$"

systemctl enable ntpd.service
systemctl restart ntpd

```

## # 4. OpenHPC base Install.

### # 4-1. 클러스터 마스터 IP 와 HOSTNAME 을 hosts 에 등록.
```bash
echo "${MASTER_IP}     ${MASTER_HOSTNAME}"  >>  /etc/hosts
cat /etc/hosts
```
출력 예)
>127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4  
::1          localhost localhost.localdomain localhost6 localhost6.localdomain6  
*10.1.1.1    master*  

### # 4-2. Open-HPC Base 설치

```bash
yum -y install ohpc-base ohpc-warewulf  >>  ~/dasan_log_ohpc_base,warewulf.txt
tail ~/dasan_log_ohpc_base,warewulf.txt  
```
출력 예)  
>  tftp-server.x86_64 0:5.2-13.el7                     
  warewulf-cluster-ohpc.x86_64 0:3.8pre-9.2                                     
  warewulf-common-ohpc.x86_64 0:3.8pre-11.1                                     
  warewulf-ipmi-ohpc.x86_64 0:3.8pre-9.1                                        
  warewulf-provision-ohpc.x86_64 0:3.8pre-28.1                                  
  warewulf-provision-server-ohpc.x86_64 0:3.8pre-28.1                           
  warewulf-vnfs-ohpc.x86_64 0:3.8pre-19.1                                       
  xinetd.x86_64 2:2.3.15-13.el7                                                 
Complete!  

***

## # 5. Resource Management Services Install.
\# **주의!** Resource Manager는 Slurm 과 PBS Pro 중 선택하여 진행 합니다.  
\# GPU Cluster 의 경우 63-A. Slurm 을 설치해야 합니다.  

## # 5-A. (Slurm) Resource Management Services Install.
### # 5-A-1. Install to ohpc-slurm-server.
```bash
yum -y install ohpc-slurm-server  >> ~/dasan_log_ohpc_resourcemanager_slurm.txt
tail ~/dasan_log_ohpc_resourcemanager_slurm.txt  
```
출력 예)
>  pmix-ohpc.x86_64 0:1.2.3-20.1                                                 
  slurm-devel-ohpc.x86_64 0:17.02.9-69.2                                        
  slurm-munge-ohpc.x86_64 0:17.02.9-69.2                                        
  slurm-ohpc.x86_64 0:17.02.9-69.2                                              
  slurm-perlapi-ohpc.x86_64 0:17.02.9-69.2                                      
  slurm-plugins-ohpc.x86_64 0:17.02.9-69.2                                      
  slurm-slurmdbd-ohpc.x86_64 0:17.02.9-69.2                                     
  slurm-sql-ohpc.x86_64 0:17.02.9-69.2                                          
Complete!  

### # 5-A-2. Slurm config file 설정 (/etc/slurm/slurm.conf)

#### # ClusterName 과 ControlMachine 변경
```bash
grep 'ClusterName\|ControlMachine' /etc/slurm/slurm.conf

```
출력 예)
>ClusterName=linux  
ControlMachine=linux0   

```bash
perl -pi -e "s/ClusterName=\S+/ClusterName=${CLUSTER_NAME}/"  /etc/slurm/slurm.conf
perl -pi -e "s/ControlMachine=\S+/ControlMachine=${MASTER_HOSTNAME}/" /etc/slurm/slurm.conf
grep ClusterName /etc/slurm/slurm.conf

```
출력 예)
>ClusterName=*OpenHPC-Dasandata*  
ControlMachine=*master*  

#### # NodeName, CPU & Memory 속성 설정

\# slurm.conf 파일의 NodeName을 클러스터 노드의 Hostname 과 동일하게 변경하고,  
\# 사양에 맞추어 Sockets, Cores, Thread, RealMemory 값을 변경 합니다.  
\# Sockets = 노드의 물리적인 CPU 갯수.  
\# CoresPerSocket = 각 물리 CPU의 논리적 코어 갯수.  
\# Thread = 하이퍼스레딩 사용 여부. (사용시 2, 미사용시 1)  
\# RealMemory = 노드의 실제 메모리 보다 약간 작게 합니다.
\# 메모리 단위는 megabyte (예를 들어 64GB 라면 60GB = 60000)  

#### # NodeName 설정
```bash
echo "NodeName="${NODE_NAME}${NODE_RANGE} &&  # 선언 되어 있는 변수 확인
grep NodeName= /etc/slurm/slurm.conf          # slurm 설정 파일의 기본 값 확인.
```
출력 예)
>NodeName=node[1-3]  
NodeName=c[1-4] Sockets=2 CoresPerSocket=8 ThreadsPerCore=2 State=UNKNOWN  

```bash
perl -pi -e "s/NodeName=\S+/NodeName=${NODE_NAME}${NODE_RANGE}/" /etc/slurm/slurm.conf
grep NodeName= /etc/slurm/slurm.conf
```
출력 예) **노드 이름이 'node' 이고, 총 수량은 3대 일 경우**
>NodeName=*node[1-3]* Sockets=2 CoresPerSocket=8 ThreadsPerCore=2 State=UNKNOWN  

#### # Node CPU, Memory 속성 설정

\# 앞서 정의한 변수 값을 다시 한번 검토 한 후 진행 합니다.
```bash
cat ~/dasan_ohpc_variable.sh
bash ~/dasan_ohpc_variable.sh
```

\# slurm 설정파일(slurm.conf)의  기존 설정 값 확인.
```bash
grep NodeName= /etc/slurm/slurm.conf
```
출력 예)
>NodeName=node[1-3] Sockets=2 CoresPerSocket=8 ThreadsPerCore=2 State=UNKNOWN  

```bash
perl -pi -e "s/Sockets=\S+/Sockets=${SOCKETS}/"  /etc/slurm/slurm.conf
perl -pi -e "s/CoresPerSocket=\S+/CoresPerSocket=${CORESPERSOCKET}/"  /etc/slurm/slurm.conf
perl -pi -e "s/ThreadsPerCore=\S+/ThreadsPerCore=${THREAD}/"  /etc/slurm/slurm.conf

grep NodeName= /etc/slurm/slurm.conf  
```
출력 예)
>NodeName=node[1-3] Sockets=*2* CoresPerSocket=*10* ThreadsPerCore=*2* State=UNKNOWN  


## # 5-B. (PBS Pro) Resource Management Services Install

### # 5-B-1. Install to pbspro-server-ohpc
```bash
yum -y install pbspro-server-ohpc >> ~/dasan_log_ohpc_resourcemanager_pbspro.txt
tail ~/dasan_log_ohpc_resourcemanager_pbspro.txt
```

***

## # 6. (Optional) Add InfiniBand support services on master node

```bash
yum -y groupinstall "InfiniBand Support"
yum -y install infinipath-psm
```

### # 6-1. (Optional) Load InfiniBand drivers
```bash
systemctl start rdma

cp /opt/ohpc/pub/examples/network/centos/ifcfg-ib0     /etc/sysconfig/network-scripts
```

### # 6-2. (Optional) Define local IPoIB(IP Over InfiniBand) address and netmask
```bash
perl -pi -e "s/master_ipoib/${sms_ipoib}/" /etc/sysconfig/network-scripts/ifcfg-ib0
perl -pi -e "s/ipoib_netmask/${ipoib_netmask}/" /etc/sysconfig/network-scripts/ifcfg-ib0

echo  “MTU=4096”  >>  /
```

### # 6-3. (Optional) Initiate ib0 (InfiniBand Interface 0)
```bash
ifup ib0

rdma   tunning

udaddy -s  10.1.1.1

```
***

## # 7. Complete basic Warewulf setup for master node

### # 7-1. 클러스터 내부망 인터페이스 변경.
\# 내부망 인터페이스 설정 내용 확인.  
```bash
echo ${INT_NIC}

ifconfig ${INT_NIC}
```
출력 예)
>p1p1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500  
        inet 10.1.1.1  netmask 255.255.255.0  broadcast 10.1.1.255  
        ether ================  txqueuelen 1000  (Ethernet)  
        RX packets 12612  bytes 3974604 (3.7 MiB)  
        RX errors 0  dropped 0  overruns 0  frame 0  
        TX packets 29576  bytes 1362244 (1.2 MiB)  
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0  

\# warewulf provision.conf 파일의 기본값 확인.  
```bash
grep device /etc/warewulf/provision.conf
```
출력 예)
>\# What is the default network device that the master will use to  
network device = **eth1**  

\# 인터페이스 명 변경
```bash

perl -pi -e "s/device = eth1/device = ${INT_NIC}/" /etc/warewulf/provision.conf
grep device /etc/warewulf/provision.conf
```

출력 예)
>\# What is the default network device that the master will use to  
network device = **p1p1**  

### # 7-2. tftp 서비스 Enable.
```bash
grep disable /etc/xinetd.d/tftp
perl -pi -e "s/^\s+disable\s+= yes/ disable = no/" /etc/xinetd.d/tftp
grep disable /etc/xinetd.d/tftp
```

출력 예)
>[root@master:\~]# grep disable /etc/xinetd.d/tftp  
	disable			= yes  
[root@master:\~]# perl -pi -e "s/^\s+disable\s+= yes/ disable = no/" /etc/xinetd.d/tftp  
[root@master:\~]# grep disable /etc/xinetd.d/tftp  
 disable = no  
[root@master:\~]#   

### # 7-3. 관련 서비스 부팅시 시작 되도록 변경(enable) 및 Restarting.
```bash
systemctl enable dhcpd
systemctl restart xinetd

systemctl enable mariadb.service
systemctl restart mariadb

systemctl enable httpd.service
systemctl restart httpd
```

### # 7-4. Define NODE image for provisioning.

#### # Check chroot location.
```bash
echo ${CHROOT}
```
출력 예)
>/opt/ohpc/admin/images/centos7.4

#### # Build initial node image.
```bash
wwmkchroot centos-7 ${CHROOT}
```
출력 예)
>Loaded plugins: fastestmirror, langpacks, priorities  
os-base                                                  | 3.6 kB     00:00     
(1/2): os-base/x86_64/group_gz                             | 156 kB   00:01     
(2/2): os-base/x86_64/primary_db                           | 5.7 MB   01:55     
Determining fastest mirrors  
Resolving Dependencies  
--> Running transaction check  
--> Package basesystem.noarch 0:10.0-7.el7.centos will be installed  
--> Package bash.x86_64 0:4.2.46-28.el7 will be installed  
<일부 생략>  
--> Finished Dependency Resolution  
Dependencies Resolved  
================================================================================  
 Package                      Arch    Version                    Repository  
                                                                           Size  
================================================================================  
Installing:  
 basesystem                   noarch  10.0-7.el7.centos          os-base  5.0 k  
 bash                         x86_64  4.2.46-28.el7              os-base  1.0 M  
 centos-release               x86_64  **7-4.1708.el7.centos**        os-base   23 k  
<일부 생략>  
Transaction Summary  
================================================================================  
Install  43 Packages (+132 Dependent packages)  
Total download size: 85 M  
Installed size: 363 M  
Downloading packages:  
<일부 생략>  
Complete!  

#### # Build 된 node image 와 master 의 kernel version 비교.

```bash
uname -r

chroot ${CHROOT} uname -r
```
출력 예)
>[root@master:~]# uname -r  
3.10.0-693.17.1.el7.x86_64  
[root@master:~]#   
[root@master:~]# chroot ${CHROOT} uname -r  
3.10.0-693.17.1.el7.x86_64  

#### # Build 된 node provision image 의 기본 glibc 라이브러리 업데이트.  
\# glibc 라이브러리 버젼 차이에 의한 locale 오류를 방지하기 위해 업데이트를 실행 합니다.  
\# 업데이트 전 버젼 확인 및 비교.  

```bash
rpm -qa | grep glibc-common
chroot ${CHROOT} rpm -qa | grep glibc-common
```

출력 예)
>[root@master:~]# rpm -qa | grep glibc-common  
**glibc-2.17-196.el7_4.2.x86_64**  
[root@master:~]# chroot ${CHROOT} rpm -qa | grep glibc-common  
**glibc-2.17-196.el7.x86_64**  

\# 업데이트
```bash
yum -y --installroot=${CHROOT} update
```

출력 예)
>Loaded plugins: fastestmirror, langpacks, priorities  
OpenHPC                                                  | 1.6 kB     00:00     
OpenHPC-updates                                          | 1.2 kB     00:00     
base                                                     | 3.6 kB     00:00     
epel/x86_64/metalink                                     | 7.0 kB     00:00     
epel                                                     | 4.7 kB     00:00     
extras                                                   | 3.4 kB     00:00     
updates                                                  | 3.4 kB     00:00     
(1/2): epel/x86_64/updateinfo                              | 892 kB   00:02     
(2/2): epel/x86_64/primary_db                              | 6.2 MB   00:11     
Determining fastest mirrors  
 \* base: ftp.daumkakao.com  
 \* epel: mirror01.idc.hinet.net  
 \* extras: ftp.daumkakao.com  
 \* updates: ftp.daumkakao.com  
OpenHPC                                                                 821/821  
OpenHPC-updates                                                       1010/1010  
139 packages excluded due to repository priority protections  
Resolving Dependencies  
--> Running transaction check  
<일부 생략>  
---> Package glibc.x86_64 0:2.17-196.el7 will be updated  
---> Package glibc.x86_64 0:2.17-196.el7_4.2 will be an update  
---> Package glibc-common.x86_64 0:2.17-196.el7 will be updated  
---> Package glibc-common.x86_64 0:2.17-196.el7_4.2 will be an update  
<일부 생략>  
Complete!  

\# 업데이트 후 버젼 확인 및 비교.
```bash
rpm -qa | grep glibc-common
chroot ${CHROOT} rpm -qa | grep glibc-common
```

출력 예)
>[root@master:\~]# rpm -qa | grep glibc-common  
**glibc-common-2.17-196.el7_4.2.x86_64**  
[root@master:\~]# chroot ${CHROOT} rpm -qa | grep glibc-common  
**glibc-common-2.17-196.el7_4.2.x86_64**  


#### # Install compute node base meta-package.
\# 기본 적으로 필요한 패키지를 node image 에 설치 합니다.
```bash
yum -y --installroot=${CHROOT} install \
 ohpc-base-compute  parted  xfsprogs  python-devel  yum  htop >> ~/dasan_log_ohpc_meta-package.txt
tail ~/dasan_log_ohpc_meta-package.txt  

```
출력 예)
>  python-urlgrabber.noarch 0:3.10-8.el7                                         
  pyxattr.x86_64 0:0.5.1-5.el7                                                  
  rpm-build-libs.x86_64 0:4.11.3-25.el7                                         
  rpm-python.x86_64 0:4.11.3-25.el7                                             
  xorg-x11-proto-devel.noarch 0:7.7-20.el7                                      
  yum-metadata-parser.x86_64 0:1.1.4-10.el7                                     
  yum-plugin-fastestmirror.noarch 0:1.1.31-42.el7                               
  zlib-devel.x86_64 0:1.2.7-17.el7                                              
Complete!  

#### # node image 에 dns 설정 (master 와 동일하게).
```bash
cat /etc/resolv.conf
cp -p /etc/resolv.conf ${CHROOT}/etc/resolv.conf  

```

***

#### # Add Slurm client support meta-package
\# **주의!** - Resource Manager 로 **Slurm** 을 사용하는 경우에만 실행 합니다.
```bash
yum -y --installroot=${CHROOT} install ohpc-slurm-client >> ~/dasan_log_ohpc_slurmclient.txt
tail ~/dasan_log_ohpc_slurmclient.txt

```

***

#### # Add PBS Professional client support
\# **주의!** - Resource Manager 로 **PBS** 를 사용하는 경우에만 실행 합니다.
```bash
yum -y --installroot=${CHROOT} install pbspro-execution-ohpc

grep PBS_SERVER ${CHROOT}/etc/pbs.conf
perl -pi -e "s/PBS_SERVER=\S+/PBS_SERVER=${MASTER_HOSTNAME}/" ${CHROOT}/etc/pbs.conf

chroot ${CHROOT}/opt/pbs/libexec/pbs_habitat

grep clienthost ${CHROOT}/var/spool/pbs/mom_priv/config
perl -pi -e "s/\$clienthost \S+/\$clienthost ${MASTER_HOSTNAME}/" ${CHROOT}/var/spool/pbs/mom_priv/config
grep clienthost ${CHROOT}/var/spool/pbs/mom_priv/config

echo "\$usecp *:/home /home" >> ${CHROOT}/var/spool/pbs/mom_priv/config
cat    ${CHROOT}/var/spool/pbs/mom_priv/config
chroot ${CHROOT}/opt/pbs/libexec/pbs_habitat

chroot ${CHROOT} systemctl enable pbs
```

***

#### # (Optional) Add IB support and enable
\# 인피니밴드(InfiniBand) 를 사용하는 경우에만 실행 합니다.
```bash
yum -y --installroot=${CHROOT} groupinstall "InfiniBand Support"
yum -y --installroot=${CHROOT} install infinipath-psm
chroot ${CHROOT} systemctl enable rdma
```

***

#### # Add Network Time Protocol (NTP) support, kernel drivers, modules user environment.
```bash
yum -y --installroot=${CHROOT} install ntp kernel lmod-ohpc  >> ~/dasan_log_ohpc_ntp,kernel,modules.txt
tail ~/dasan_log_ohpc_ntp,kernel,modules.txt  

```

## # 8. Customize system configuration.

### # 8-1. Initialize warewulf database and add new cluster key to base image.
```bash
wwinit database
wwinit ssh_keys
cat ~/.ssh/cluster.pub >> ${CHROOT}/root/.ssh/authorized_keys

```
출력 예)
>[root@master:\~]# wwinit database  
database:     Checking to see if RPM 'mysql-server' is installed             NO  
database:     Checking to see if RPM 'mariadb-server' is installed           OK  
database:     Activating Systemd unit: mariadb  
database:      + /bin/systemctl -q enable mariadb.service                    OK  
database:      + /bin/systemctl -q restart mariadb.service                   OK  
database:      + mysqladmin --defaults-extra-file=/tmp/0.QNPo6o3qrQus/my.cnf OK  
database:     Database version: UNDEF (need to create database)  
database:      + mysql --defaults-extra-file=/tmp/0.QNPo6o3qrQus/my.cnf ware OK  
database:      + mysql --defaults-extra-file=/tmp/0.QNPo6o3qrQus/my.cnf ware OK  
database:      + mysql --defaults-extra-file=/tmp/0.QNPo6o3qrQus/my.cnf ware OK  
database:     Checking binstore kind                                    SUCCESS  
Done.  
[root@master:\~]# wwinit ssh_keys  
ssh_keys:     Checking ssh keys for root                                     OK  
ssh_keys:     Checking root's ssh config                                     OK  
ssh_keys:     Checking for default RSA1 host key for nodes                   NO  
ssh_keys:     Creating default node ssh_host_key:  
                                                                             OK  
ssh_keys:     Checking for default RSA host key for nodes                    NO  
ssh_keys:     Creating default node ssh_host_rsa_key:  
ssh_keys:      + ssh-keygen -q -t rsa -f /etc/warewulf/vnfs/ssh/ssh_host_rsa OK  
ssh_keys:     Checking for default DSA host key for nodes                    NO  
ssh_keys:     Creating default node ssh_host_dsa_key:  
ssh_keys:      + ssh-keygen -q -t dsa -f /etc/warewulf/vnfs/ssh/ssh_host_dsa OK  
ssh_keys:     Checking for default ECDSA host key for nodes                  NO  
ssh_keys:     Creating default node ssh_host_ecdsa_key:  
                                                                             OK  
Done.  
[root@master:\~]# cat \~/.ssh/cluster.pub >> ${CHROOT}/root/.ssh/authorized_keys  
[root@master:\~]#   

### # 8-2. Add NFS client mounts of /home and /opt/ohpc/pub and /{ETC} to base image.

```bash
df -hT | grep -v tmpfs
echo ${MASTER_HOSTNAME}
cat  ${CHROOT}/etc/fstab

echo "${MASTER_HOSTNAME}:/home /home nfs nfsvers=3,rsize=1024,wsize=1024,cto 0 0" >> ${CHROOT}/etc/fstab
echo "${MASTER_HOSTNAME}:/opt/ohpc/pub /opt/ohpc/pub nfs nfsvers=3 0 0" >> ${CHROOT}/etc/fstab

# 아래는 data 디렉토리를 별도로 구성하는 경우에만.
#echo "${MASTER_HOSTNAME}:/data /data nfs nfsvers=3 0 0" >> ${CHROOT}/etc/fstab
cat  ${CHROOT}/etc/fstab  

```
출력 예)
>[root@master:\~]# df -hT | grep -v tmpfs  
Filesystem                     Type      Size  Used Avail Use% Mounted on  
/dev/mapper/centos_master-root xfs       898G  9.6G  889G   2% /  
/dev/sda1                      xfs      1014M  189M  826M  19% /boot  
/dev/mapper/vg_home-lv_home    xfs        39T   55M   39T   1% /home  
[root@master:\~]#  
[root@master:\~]# cat  ${CHROOT}/etc/fstab  
\#GENERATED_ENTRIES#  
tmpfs /dev/shm tmpfs defaults 0 0  
devpts /dev/pts devpts gid=5,mode=620 0 0  
sysfs /sys sysfs defaults 0 0  
proc /proc proc defaults 0 0  
master:/home /home nfs nfsvers=3,rsize=1024,wsize=1024,cto 0 0  
master:/opt/ohpc/pub /opt/ohpc/pub nfs nfsvers=3 0 0  
[root@master:\~]#  


### # 8-3. Export /home and OpenHPC public packages from master server.

```bash
cat /etc/exports

echo "/home *(rw,no_subtree_check,fsid=10,no_root_squash)" >> /etc/exports
echo "/opt/ohpc/pub *(ro,no_subtree_check,fsid=11)" >> /etc/exports

# 아래는 data 디렉토리를 별도로 구성하는 경우에만.  
#echo "/data *(rw,no_subtree_check,fsid=10,no_root_squash)" >> /etc/exports

cat /etc/exports
```
출력 예)
>/home \*(rw,no_subtree_check,fsid=10,no_root_squash)
/opt/ohpc/pub \*(ro,no_subtree_check,fsid=11)


```bash
exportfs -a

systemctl enable nfs-server
systemctl restart nfs-server

exportfs
```
출력 예)
>/home         	<world>
/opt/ohpc/pub 	<world>


### # 8-4. Enable NTP time service on computes and identify master host as local NTP server.

```bash
chroot ${CHROOT} systemctl enable ntpd
echo "server ${MASTER_HOSTNAME}" >> ${CHROOT}/etc/ntp.conf
```


***

### # 8-5. (Optional Custom) Increase locked memory limits
\# 기본 네트워크 구성이 InfiniBand 또는 Omni-Path를 로 되어 있을 경우에만 수행 합니다.

\# Update memlock settings on master and compute
```bash
perl -pi -e 's/# End of file/\* soft memlock unlimited\n$&/s' /etc/security/limits.conf
perl -pi -e 's/# End of file/\* hard memlock unlimited\n$&/s' /etc/security/limits.conf
perl -pi -e 's/# End of file/\* soft memlock unlimited\n$&/s' ${CHROOT}/etc/security/limits.conf
perl -pi -e 's/# End of file/\* hard memlock unlimited\n$&/s' ${CHROOT}/etc/security/limits.conf

tail /etc/security/limits.conf
tail ${CHROOT}/etc/security/limits.conf
```

***

### # 8-6. Add Ganglia monitoring.
\# Ganglia Monitoring System : https://en.wikipedia.org/wiki/Ganglia_(software)

#### # Install Ganglia meta-package on master
```bash
yum -y install ohpc-ganglia >> ~/dasan_log_ohpc_ganglia.txt
tail ~/dasan_log_ohpc_ganglia.txt  

```

#### # Install Ganglia compute node daemon
```bash
yum -y --installroot=${CHROOT} install ganglia-gmond-ohpc >> ~/dasan_log_ohpc_ganglia-node.txt
tail ~/dasan_log_ohpc_ganglia-node.txt  

```

#### # Use example configuration script to enable unicast receiver on master host
```bash
/usr/bin/cp  /opt/ohpc/pub/examples/ganglia/gmond.conf  /etc/ganglia/gmond.conf

grep 'host =' /etc/ganglia/gmond.conf
perl -pi -e "s/<sms>/${MASTER_HOSTNAME}/" /etc/ganglia/gmond.conf
grep 'host ='  /etc/ganglia/gmond.conf  

```

#### # Add configuration to compute image and provide gridname
```bash
/usr/bin/cp   /etc/ganglia/gmond.conf  ${CHROOT}/etc/ganglia/gmond.conf
echo "gridname MySite" >> /etc/ganglia/gmetad.conf  

```
#### # Start and enable Ganglia services
```bash
systemctl enable gmond
systemctl enable gmetad
systemctl start gmond
systemctl start gmetad

chroot ${CHROOT} systemctl enable gmond  
  
```

# Restart web server
`systemctl try-restart httpd`

\# Open to http://localhost/ganglia

***

### # Import files
```bash
wwsh file import /etc/passwd
wwsh file import /etc/group
wwsh file import /etc/shadow
```
### # optional support for controlling IPoIB interfaces
```bash
wwsh file import /opt/ohpc/pub/examples/network/centos/ifcfg-ib0.ww
wwsh -y file set ifcfg-ib0.ww --path=/etc/sysconfig/network-scripts/ifcfg-ib0
```

## # Finalizing provisioning configuration

# Assemble bootstrap image
```bash
wwbootstrap  `uname -r`
```

# (Optional) Include BeeGFS/Lustre drivers; needed if enabling additional kernel modules on computes
```bash
export WW_CONF=/etc/warewulf/bootstrap.conf
echo "drivers += updates/kernel/" >> $WW_CONF
```

# (Optional) Include overlayfs drivers; needed by Singularity
`echo "drivers += overlay" >> $WW_CONF`

## Assemble Virtual Node File System (VNFS) image

### # Assemble Virtual Node File System (VNFS) image
```bash
export CHROOT=/opt/ohpc/admin/images/centos7.4
wwvnfs --chroot ${CHROOT}
```

***

## # 64. Register nodes for provisioning
### # Set provisioning interface as the default networking device
```bash
echo "GATEWAYDEV=eth0" > /tmp/network.$$
wwsh -y file import /tmp/network.$$ --name network
wwsh -y file set network --path /etc/sysconfig/network --mode=0644 --uid=0
```

 # Add new nodes to Warewulf data store
```bash
wwsh -y node new n01  --netdev  eth0 --ipaddr=10.1.1.1 --hwaddr=******** --gateway ${MASTER_IP}  --netmask=255.255.255.0
```


### # Define provisioning image for hosts
```bash
wwsh -y provision set "n01" --vnfs=centos7.3 --bootstrap=` uname -r ` --files=dynamic_hosts,passwd,group,shadow,slurm.conf,munge.key,network

wwsh  node list
wwsh  node print
wwsh  object  print  -p :all
wwsh  file list
wwsh  pxe  update
wwsh  dhcp  update
wwsh  dhcp   restart
wwsh  bootstrap  list
wwsh  vnfs  list
```

### # Restart dhcp / update PXE

```bash
systemctl restart dhcpd
wwsh pxe update
```

### # node  n01 boot on pxe
```bash
ping n01
ssh n01
df -hT

su - dasan
ssh n01
```

## 65. OpenHPC - Add Node (Clone)

```bash
wwsh node list
wwsh -y node clone n01 n02
wwsh node list
```

```bash
wwsh -y node set n02 --netdev=eth0 --ipaddr=10.1.1.2 --hwaddr=70:85:c2:4d:a8:b7  --gateway=10.1.1.254  --netmask=255.255.255.0
```

## # 66. WOL - Wake Nodes

`vi node_on_by_WOL.sh`
`cat node_on_by_WOL.sh`

============================

# Node List Example
```bash
n0[1]=70:85:c2:4d:a8:9a
n0[2]=70:85:c2:4d:a8:b7
n0[3]=70:85:c2:4d:a7:d4
n0[4]=70:85:c2:4d:a8:28
n0[5]=70:85:c2:4d:a8:a7
n0[6]=70:85:c2:4d:a8:b5
n0[7]=70:85:c2:4d:a8:b1
n0[8]=70:85:c2:4d:a7:ac
```

# Node Num + 1
NODE_NUM=9

# Wake On Command
```bash
for ((i=1 ; i<$NODE_NUM ; i++))
do
ether-wake -i em2 ${n0[$i]}
done

=============================

sh node_on_by_WOL.sh

pdsh -w n[01-08] date | sort
```

## # 64. OpenHPC - Add User
```bash
adduser test
passwd test

echo  “Y”  | wwsh file import /etc/passwd
echo  “Y” |  wwsh file import /etc/group
echo  “Y” |  wwsh file import /etc/shadow

wwinit ssh_keys
wwsh   file  resync  passwd  shadow  group

pdsh -w n[01-02] /warewulf/bin/wwgetfiles

su - test
ssh compute01
```

## 65. OpenHPC - Node Boot Image Update (after add Program)
```bash
[root@ohpc-master:~]# # Define chroot location
[root@ohpc-master:~]# export CHROOT=/opt/ohpc/admin/images/centos7.4
[root@ohpc-master:~]# echo ${CHROOT}
/opt/ohpc/admin/images/centos7.3
[root@ohpc-master:~]#
[root@ohpc-master:~]# yum -y --installroot=${CHROOT} install tmux
Loaded plugins: fastestmirror, langpacks, priorities

<생략>

Complete!
[root@ohpc-master:~]#
[root@ohpc-master:~]# wwvnfs --chroot /opt/ohpc/admin/images/centos7.3
<생략>

[root@ohpc-master:~]#
[root@ohpc-master:~]# ssh compute01
Last login: Mon Aug 28 10:00:18 2017 from ohpc-master
[root@compute01 ~]#
[root@compute01 ~]# tmux
-bash: tmux: command not found
[root@compute01 ~]#
[root@compute01 ~]# reboot
Connection to compute01 closed by remote host.
Connection to compute01 closed.
[root@ohpc-master:~]#
[root@ohpc-master:~]#
[root@ohpc-master:~]# ssh compute01
[root@compute01 ~]#
[root@compute01 ~]# tmux
[exited]
[root@compute01 ~]# [root@compute01 ~]# which tmux
/usr/bin/tmux
```


## # 제외 // @@. OpenHPC - user's ssh login to compute nodes

```bash
vi /opt/ohpc/admin/images/centos7.3/etc/pam.d/sshd
tail -5 /opt/ohpc/admin/images/centos7.3/etc/pam.d/sshd
```
session    include      password-auth
session    include      postlogin
# Used with polkit to reauthorize users in remote sessions
-session   optional     pam_reauthorize.so prepare
#account required pam_slurm.so

wwvnfs --chroot /opt/ohpc/admin/images/centos7.3

ssh compute01 reboot
ssh compute02 reboot


66. OpenHPC -  Node Local Disk Mount (/scratch)


# Define chroot location
```bash
echo ${CHROOT}

yum -y --installroot=${CHROOT} install parted xfsprogs
mkdir ${CHROOT}/scratch
wwvnfs --chroot=${CHROOT}

pdsh -w n[01-08] reboot
```

===== reboot ======
```bash
pdsh -w n[01-08] date

pdsh -w n[01-08] parted  -s /dev/sda "mklabel  GPT "
pdsh -w n[01-08] parted  -s /dev/sda "mkpart primary 0% 100%"
pdsh -w n[01-08] mkfs.xfs  -f  -L SCRATCH   -i  size=1024  -s  size=4096   /dev/sda1
```

```bash
export CHROOT=/opt/ohpc/admin/images/centos7.4
echo "LABEL="SCRATCH"  /scratch   xfs  defaults  0 0 "  >>  ${CHROOT}/etc/fstab
cat ${CHROOT}/etc/fstab

vi ${CHROOT}/etc/rc.d/rc.local
tail -3 ${CHROOT}/etc/rc.d/rc.local
```

# dasandata add
chmod 1777 /scratch

ll ${CHROOT}/etc/rc.d/rc.local
chmod +x ${CHROOT}/etc/rc.d/rc.local

ll ${CHROOT}/etc/rc.d/rc.local

wwvnfs --chroot=${CHROOT}

pdsh -w n[01-08] reboot

===== reboot ======

```bash
pdsh -w n[01-08] date

pdsh -w n[01-08] lsblk
pdsh -w n[01-08] 'df -hT | grep -v tmpfs' | sort | grep -v Filesystem

pdsh -w n[01-08] 'll / | grep scratch'

```


## # 67.  Install OpenHPC Development Components

# Install autotools meta-package (Default)
```bash
yum -y  install ohpc-autotools EasyBuild-ohpc hwloc-ohpc spack-ohpc valgrind-ohpc
```

# Compilers ## gcc ver 7 and 5.4
```bash
yum -y  install gnu7-compilers-ohpc   gnu-compilers-ohpc
```

# MPI Stacks
```bash
yum -y  install openmpi-gnu7-ohpc mvapich2-gnu7-ohpc mpich-gnu7-ohpc
```

# Install perf-tools meta-package
```bash
yum -y  install ohpc-gnu7-perf-tools
yum -y  groupinstall  ohpc-perf-tools-gnu
```

# Setup default development environment
```bash
yum -y  install lmod-defaults-gnu7-openmpi-ohpc
```
### yum -y install lmod-defaults-gnu-openmpi-ohpc


# Install 3rd party libraries/tools meta-packages built with GNU toolchain
```bash
yum -y  install   ohpc-gnu7-serial-libs   ohpc-gnu7-io-libs   ohpc-gnu7-python-libs  ohpc-gnu7-runtimes
```

# Install parallel lib meta-packages for all available MPI toolchains
yum -y  install   ohpc-gnu7-mpich-parallel-libs   ohpc-gnu7-mvapich2-parallel-libs   ohpc-gnu7-openmpi-parallel-libs

# Install gnu5  MPI Stacks & lib & meta-packages
yum -y  groupinstall    ohpc-io-libs-gnu    ohpc-parallel-libs-gnu  ohpc-parallel-libs-gnu-mpich    \
ohpc-python-libs-gnu  ohpc-runtimes-gnu    ohpc-serial-libs-gnu
