# Dasandata Standard Recipes of GPU Node on OpenHPC Cluster (v1.3.3-CentOS7.4 Base OS)[2018.02]


## # Check dasan_ohpc_variable.sh
```bash
[root@master:~]#
[root@master:~]# pwd
/root
[root@master:~]#
[root@master:~]# cat dasan_ohpc_variable.sh
#!/bin/bash

# 클러스터 이름.
export CLUSTER_NAME=OpenHPC-Dasandata

# MASTER 의 이름 과 IP.
export MASTER_HOSTNAME=$(hostname)
export MASTER_IP=10.1.1.1
export MASTER_PREFIX=24

# 인터페이스 이름.
export EXT_NIC=em2 # 외부망.
export INT_NIC=p1p1 # 내부망.
export NODE_INT_NIC=p1p1 # node 들의 내부망 인터페이스 명.

# NODE 의 이름, 수량, 사양.
export NODE_NAME=node
export NODE_NUM=3
export NODE_RANGE="[1-3]"  # 전체 노드가 3개일 경우 1-3 / 5대 일 경우 [1-5]

# NODE 의 CPU 사양에 맞게 조정.
# 물리 CPU가 2개 이고, CPU 당 코어가 10개, 하이퍼스레딩은 켜진(Enable) 상태 인 경우.  
export SOCKETS=2          ## 물리 CPU 2개
export CORESPERSOCKET=6  ## CPU 당 코어 10개
export THREAD=2           ## 하이퍼스레딩 Enable

# 노드 배포 이미지 경로.
export CHROOT=/opt/ohpc/admin/images/centos7.4

# end of file.
[root@master:~]#
[root@master:~]#
```

## # Load to $CHROOT variable
```bash
[root@master:~]# source dasan_ohpc_variable.sh
[root@master:~]#
[root@master:~]# echo ${CHROOT}
/opt/ohpc/admin/images/centos7.4
[root@master:~]#
```



## # Install cuda-repo to Master
```bash
curl  -L -o  cuda-repo-rhel7-8.0.61-1.x86_64.rpm \
 http://developer.download.nvidia.com/compute/cuda/repos/rhel7/x86_64/cuda-repo-rhel7-8.0.61-1.x86_64.rpm

yum -y install cuda-repo-rhel7-8.0.61-1.x86_64.rpm \
 >> dasan_log_ohpc_cudarepo-on-master.txt
tail dasan_log_ohpc_cudarepo-on-master.txt

cat /etc/yum.repos.d/cuda.repo
```

## # Install cuda-repo to node vnfs images
```bash
yum -y install --installroot ${CHROOT} cuda-repo-rhel7-8.0.61-1.x86_64.rpm \
 >> dasan_log_ohpc_cudarepo-on-node.txt
tail dasan_log_ohpc_cudarepo-on-node.txt

cat ${CHROOT}/etc/yum.repos.d/cuda.repo

```
*output example>*
```bash
[root@master:~]# curl  -L -o  cuda-repo-rhel7-8.0.61-1.x86_64.rpm \
>  http://developer.download.nvidia.com/compute/cuda/repos/rhel7/x86_64/cuda-repo-rhel7-8.0.61-1.x86_64.rpm
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  6363  100  6363    0     0  21969      0 --:--:-- --:--:-- --:--:-- 22248
[root@master:~]#
[root@master:~]# yum -y install --installroot ${CHROOT} cuda-repo-rhel7-8.0.61-1.x86_64.rpm \
>  >> dasan_log_ohpc_cudarepo-on-node.txt
[root@master:~]#
[root@master:~]# tail dasan_log_ohpc_cudarepo-on-node.txt
Running transaction test
Transaction test succeeded
Running transaction
  Installing : cuda-repo-rhel7-8.0.61-1.x86_64                              1/1
  Verifying  : cuda-repo-rhel7-8.0.61-1.x86_64                              1/1

Installed:
  cuda-repo-rhel7.x86_64 0:8.0.61-1                                             

Complete!
[root@master:~]#
[root@master:~]# cat ${CHROOT}/etc/yum.repos.d/cuda.repo
[cuda]
name=cuda
baseurl=http://developer.download.nvidia.com/compute/cuda/repos/rhel7/x86_64
enabled=1
gpgcheck=1
gpgkey=http://developer.download.nvidia.com/compute/cuda/repos/rhel7/x86_64/7fa2af80.pub
[root@master:~]#
```

## # Check to available list cuda repo
```bash

yum --disablerepo="*" --enablerepo="cuda" list available

chroot ${CHROOT} yum --disablerepo="*" --enablerepo="cuda" list available

```
*output example>*
```bash
[root@master:~]# chroot ${CHROOT} yum --disablerepo="*" --enablerepo="cuda" list available
Loaded plugins: fastestmirror
cuda                                                     | 2.5 kB     00:00     
cuda/primary_db                                            | 125 kB   00:00     
Loading mirror speeds from cached hostfile
Available Packages
cuda.x86_64                                      9.1.85-1                   cuda
cuda-7-0.x86_64                                  7.0-28                     cuda
cuda-7-5.x86_64                                  7.5-18                     cuda
cuda-8-0.x86_64                                  8.0.61-1                   cuda
cuda-9-0.x86_64                                  9.0.176-1                  cuda
cuda-9-1.x86_64                                  9.1.85-1                   cuda
cuda-command-line-tools-7-0.x86_64               7.0-28                     cuda
cuda-command-line-tools-7-5.x86_64   
<일부 생략>
cuda-visual-tools-9-1.x86_64                     9.1.85-1                   cuda
gpu-deployment-kit.x86_64                        352.93-0                   cuda
nvidia-kmod.x86_64                               1:390.30-2.el7             cuda
nvidia-uvm-kmod.x86_64                           1:352.99-3.el7             cuda
xorg-x11-drv-nvidia.x86_64                       1:390.30-1.el7             cuda
xorg-x11-drv-nvidia-devel.x86_64                 1:390.30-1.el7             cuda
xorg-x11-drv-nvidia-diagnostic.x86_64            1:390.30-1.el7             cuda
xorg-x11-drv-nvidia-gl.x86_64                    1:390.30-1.el7             cuda
xorg-x11-drv-nvidia-libs.x86_64                  1:390.30-1.el7             cuda
[root@master:~]#
```


## # (Optional) Install libGLU.so libX11.so libXi.so libXmu.so to Master
```bash
yum -y install libXi-devel mesa-libGLU-devel \
libXmu-devel libX11-devel freeglut-devel libXm*   openmotif*  \
  >> dasan_log_ohpc_libGLU,libX-on-master.txt
tail dasan_log_ohpc_libGLU,libX-on-master.txt
```


## # Install  libGLU.so libX11.so libXi.so libXmu.so to node vnfs images
```bash
yum -y install --installroot ${CHROOT} libXi-devel mesa-libGLU-devel \
libXmu-devel libX11-devel freeglut-devel libXm*   openmotif*  \
 >> dasan_log_ohpc_libGLU,libX-on-node.txt 2>&1
tail dasan_log_ohpc_libGLU,libX-on-node.txt
```


## # (Optional) Install cuda 8.0, 9.0 to Master
```bash
yum -y install cuda-8-0 cuda-9-0 \
>> dasan_log_ohpc_cuda8,9-master.txt 2>&1
tail dasan_log_ohpc_cuda8,9-master.txt
```


## # Install cuda 8.0, 9.0 to node vnfs images
```bash
yum -y install --installroot ${CHROOT} cuda-8-0 cuda-9-0 \
>> dasan_log_ohpc_cuda8,9-node-vnfs.txt 2>&1
tail dasan_log_ohpc_cuda8,9-node-vnfs.txt
```


## # Install Cudnn to node vnfs images
\# 먼저 cudnn 압축파일을 ~/cudnn 에 다운로드 한 후 진행 합니다.


```
cd ~/cudnn
pwd
ls -l

tar xvzf cudnn-8.0-linux-x64-v5.0.tgz  
tar xvzf cudnn-8.0-linux-x64-v6.0.tgz   
tar xvzf cudnn-8.0-linux-x64-v7.0.tgz

ls -l cuda/include/
ls -l cuda/lib64/

chmod a+r  cuda/include/*
chmod a+r  cuda/lib64/*

mv  cuda/include/cudnn.h  ${CHROOT}/usr/local/cuda-8.0/include/
mv  cuda/lib64/libcudnn*  ${CHROOT}/usr/local/cuda-8.0/lib64/

cd
updatedb ; locate libcudnn.so
```

## # wwsh vnfs.con 파일 수정.
```bash
[root@master:~]# grep -n -v '^$\|^#' /etc/warewulf/vnfs.conf
15:gzip command = /usr/bin/pigz -9
26:cpio command = cpio --quiet -o -H newc
31:build directory = /var/tmp/
41:exclude += /tmp/*
42:exclude += /var/log/*
43:exclude += /var/chroots/*
44:exclude += /var/cache
45:exclude += /usr/src
46:exclude += /usr/local
68:hybridize += /usr/X11R6
72:hybridize += /usr/share/man
73:hybridize += /usr/share/doc
[root@master:~]#
```


## # /usr/local/ 을 NFS Service 에 추가.
```bash
echo "/usr/local *(ro,no_subtree_check)"  >> /etc/exports
systemctl restart nfs-server
exportfs

echo "dfnc-master:/usr/local /usr/local nfs nfsvers=3 0 0" >> ${CHROOT}/etc/fstab
```


## # Update to Node nvfs image.
```bash
wwvnfs --chroot ${CHROOT}

wwsh vnfs list
```


## # Apply update imgae to nodes (rebooting)
```bash
ssh node1 reboot
```


## # Add Cuda Module for GPU Node
### # Download Module Template of CUDA
```bash
git clone https://github.com/dasandata/open_hpc
GIT_CLONE_DIR="/root/open_hpc"
echo ${GIT_CLONE_DIR}


MODULES_DIR="/opt/ohpc/pub/modulefiles"
mkdir -p ${MODULES_DIR}/cuda
```

### # Add CUDA Module File by each version
```bash
for VERSION in 7.0 7.5 8.0 9.0 9.1; do
cp -a ${GIT_CLONE_DIR}/Module_Template/cuda.lua ${MODULES_DIR}/cuda/${VERSION}.lua ;
sed -i "s/{version}/${VERSION}/" ${MODULES_DIR}/cuda/${VERSION}.lua ;
done
```

### # Refresh modules
```bash
rm -rf  ~/.lmod.d/.cache

module av
```
*output example>*
```bash
------------------------ /opt/ohpc/pub/moduledeps/gnu7 -------------------------
   R/3.4.2        mvapich2/2.2       openmpi/1.10.7 (L)    scotch/6.0.4
   gsl/2.4        numpy/1.13.1       openmpi3/3.0.0        superlu/5.2.1
   metis/5.1.0    ocr/1.0.1          pdtoolkit/3.24
   mpich/3.2      openblas/0.2.20    plasma/2.8.0

------------------------- /opt/ohpc/admin/modulefiles --------------------------
   spack/0.10.0

-------------------------- /opt/ohpc/pub/modulefiles ---------------------------
   EasyBuild/3.4.1        cuda/9.0            papi/5.5.1
   autotools       (L)    cuda/9.1     (D)    pmix/1.2.3
   cmake/3.9.2            gnu/5.4.0           prun/1.2        (L)
   cuda/7.0               gnu7/7.2.0   (L)    singularity/2.4
   cuda/7.5               hwloc/1.11.8        valgrind/3.13.0
   cuda/8.0               ohpc         (L)

  Where:
   D:  Default Module
   L:  Module is loaded

Use "module spider" to find all possible modules.
Use "module keyword key1 key2 ..." to search for all possible modules matching
any of the "keys".
```
### # CUDA Sample Compile
\# Master 에서 Sample 을 Compile 한 후 실행은 node 에서 합니다.





## # Install Python 3.5.4 to Master

### # Pre installation package
```bash
yum -y install zlib-devel bzip2-devel sqlite sqlite-devel openssl-devel
```

### # Download Python 3.5.4
```bash
wget https://www.python.org/ftp/python/3.5.4/Python-3.5.4.tgz
tar  xvzf  Python-3.5.4.tgz
mv ~/Python-3.5.4 ~/Python-3.5.4-gnu4
cp -r ~/Python-3.5.4-gnu4 ~/Python-3.5.4-gnu5
cp -r ~/Python-3.5.4-gnu4 ~/Python-3.5.4-gnu7
```
### # Install Python For each compiler version

#### # For gnu4
```bash
module purge
gcc --version

cd ~/Python-3.5.4-gnu4
./configure \
--enable-optimizations \
--with-ensurepip=install \
--enable-shared \
--prefix=/opt/ohpc/pub/apps/python3/gnu4

make -j$(grep -c processor /proc/cpuinfo) ; make install

cd ..
rm -rf ~/Python-3.5.4-gnu4
```

#### # For gnu5
```bash
module purge
module load gnu
gcc --version

cd ~/Python-3.5.4-gnu5
./configure \
--enable-optimizations \
--with-ensurepip=install \
--enable-shared \
--prefix=/opt/ohpc/pub/apps/python3/gnu5

make -j$(grep -c processor /proc/cpuinfo) ; make install

cd ..
rm -rf ~/Python-3.5.4-gnu5
```

#### # For gnu7
```bash
module purge
module load gnu7
gcc --version

cd ~/Python-3.5.4-gnu7
./configure \
--enable-optimizations \
--with-ensurepip=install \
--enable-shared \
--prefix=/opt/ohpc/pub/apps/python3/gnu7

make -j$(grep -c processor /proc/cpuinfo) ; make install

cd ~
rm -rf ~/Python-3.5.4-gnu7
```

## # Add Python Module for GPU Node
### # Download Module Template of Python
```bash
cd /root/
git clone https://github.com/dasandata/open_hpc
GIT_CLONE_DIR="/root/open_hpc"
echo ${GIT_CLONE_DIR}

MODULES_DIR="/opt/ohpc/pub/modulefiles"
mkdir -p ${MODULES_DIR}/python3

MODULE_DEPS_DIR="/opt/ohpc/pub/moduledeps"
mkdir -p ${MODULE_DEPS_DIR}/gnu/python3
mkdir -p ${MODULE_DEPS_DIR}/gnu7/python3
VERSION=3.5.4
```

### # Add Python Module File by each version
```bash
GNU_VERSION=4
cp -a ${GIT_CLONE_DIR}/Module_Template/python3.txt        ${MODULES_DIR}/python3/${VERSION}
sed -i "s/{VERSION}/${VERSION}/"          ${MODULES_DIR}/python3/${VERSION}
sed -i "s/{GNU_VERSION}/${GNU_VERSION}/"  ${MODULES_DIR}/python3/${VERSION}
```

```bash
GNU_VERSION=5
cp -a ${GIT_CLONE_DIR}/Module_Template/python3.txt        ${MODULE_DEPS_DIR}/gnu/python3/${VERSION}
sed -i "s/{VERSION}/${VERSION}/"          ${MODULE_DEPS_DIR}/gnu/python3/${VERSION}
sed -i "s/{GNU_VERSION}/${GNU_VERSION}/"  ${MODULE_DEPS_DIR}/gnu/python3/${VERSION}
```

```bash
GNU_VERSION=7
cp -a ${GIT_CLONE_DIR}/Module_Template/python3.txt        ${MODULE_DEPS_DIR}/gnu7/python3/${VERSION}
sed -i "s/{VERSION}/${VERSION}/"          ${MODULE_DEPS_DIR}/gnu7/python3/${VERSION}
sed -i "s/{GNU_VERSION}/${GNU_VERSION}/"  ${MODULE_DEPS_DIR}/gnu7/python3/${VERSION}
```

### # Refresh modules
```bash
rm -rf  ~/.lmod.d/.cache

module av
```
*output example>*
```bash
------------------------ /opt/ohpc/pub/moduledeps/gnu7 -------------------------
   R/3.4.2        mvapich2/2.2       openmpi/1.10.7 (L)    python3/3.4.5 (D)
   gsl/2.4        numpy/1.13.1       openmpi3/3.0.0        scotch/6.0.4
   metis/5.1.0    ocr/1.0.1          pdtoolkit/3.24        superlu/5.2.1
   mpich/3.2      openblas/0.2.20    plasma/2.8.0

------------------------- /opt/ohpc/admin/modulefiles --------------------------
   spack/0.10.0

-------------------------- /opt/ohpc/pub/modulefiles ---------------------------
   EasyBuild/3.4.1        cuda/9.0            papi/5.5.1
   autotools       (L)    cuda/9.1     (D)    pmix/1.2.3
   cmake/3.9.2            gnu/5.4.0           prun/1.2        (L)
   cuda/7.0               gnu7/7.2.0   (L)    python3/3.4.5
   cuda/7.5               hwloc/1.11.8        singularity/2.4
   cuda/8.0               ohpc         (L)    valgrind/3.13.0

  Where:
   D:  Default Module
   L:  Module is loaded

Use "module spider" to find all possible modules.
Use "module keyword key1 key2 ..." to search for all possible modules matching
any of the "keys".
```
### # Test of Python Module
```bash
[root@master:~]# ml purge
[root@master:~]# ml list
No modules loaded
[root@master:~]#
[root@master:~]# gcc --version | head -1
gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-16)
[root@master:~]#
[root@master:~]# which python3
/usr/bin/which: no python3 in (/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/opt/dell/srvadmin/bin:/opt/dell/srvadmin/sbin:/root/bin)
[root@master:~]#
[root@master:~]# ml load python3
[root@master:~]#
[root@master:~]# which python3
/opt/ohpc/pub/apps/python3/gnu4/bin/python3
[root@master:~]#
[root@master:~]# python3 -V
Python 3.5.4
[root@master:~]#
```

```bash
[root@master:~]# ml purge
[root@master:~]#
[root@master:~]# ml load gnu
[root@master:~]#
[root@master:~]# gcc --version | head -1
gcc (GCC) 5.4.0
[root@master:~]#
[root@master:~]# ml load python3
[root@master:~]#
[root@master:~]# which python3
/opt/ohpc/pub/apps/python3/gnu5/bin/python3
[root@master:~]#
[root@master:~]# python3 -V
Python 3.5.4
[root@master:~]#
```

```bash
[root@master:~]# ml purge
[root@master:~]#
[root@master:~]# ml load gnu7
[root@master:~]#
[root@master:~]# gcc --version | head -1
gcc (GCC) 7.2.0
[root@master:~]#
[root@master:~]# ml load python3
[root@master:~]#
[root@master:~]# which python3
/opt/ohpc/pub/apps/python3/gnu7/bin/python3
[root@master:~]#
[root@master:~]# python3 -V
Python 3.5.4
[root@master:~]#
```

## # Install pip for Python 2.7.5 to Master
```bash
which  python
rpm -qa  |  grep ^python-2.7
python -V
rpm -ql  python-2.7.5

easy_install pip
rpm -qa | grep  setuptools

pip -V
```
*output example>*
```bash
[root@master:~]# which  python
/bin/python
[root@master:~]# rpm -qa  |  grep ^python-2.7
python-2.7.5-58.el7.x86_64
[root@master:~]# python -V
Python 2.7.5
[root@master:~]#
[root@master:~]# rpm -ql  python-2.7.5
/usr/bin/pydoc
/usr/bin/python
/usr/bin/python2
/usr/bin/python2.7
/usr/share/doc/python-2.7.5
/usr/share/doc/python-2.7.5/LICENSE
/usr/share/doc/python-2.7.5/README
/usr/share/man/man1/python.1.gz
/usr/share/man/man1/python2.1.gz
/usr/share/man/man1/python2.7.1.gz
[root@master:~]#
[root@master:~]# easy_install pip
Searching for pip
Reading https://pypi.python.org/simple/pip/
Best match: pip 9.0.1
Downloading https://pypi.python.org/packages/11/b6/abcb525026a4be042b486df43905d6893fb04f05aac21c32c638e939e447/pip-9.0.1.tar.gz#md5=35f01da33009719497f01a4ba69d63c9

<일부 생략>

Installed /usr/lib/python2.7/site-packages/pip-9.0.1-py2.7.egg
Processing dependencies for pip
Finished processing dependencies for pip
[root@master:~]# rpm -qa | grep  setuptools
python-setuptools-0.9.8-7.el7.noarch
[root@master:~]#
[root@master:~]# pip -V
pip 9.0.1 from /usr/lib/python2.7/site-packages/pip-9.0.1-py2.7.egg (python 2.7)
[root@master:~]#
```


## # Install pip for Python 2.7.5 to node vnfs imgae

```bash
echo ${CHROOT}

wget https://pypi.python.org/packages/5f/ad/1fde06877a8d7d5c9b60eff7de2d452f639916ae1d48f0b8f97bf97e570a/distribute-0.7.3.zip#md5=c6c59594a7b180af57af8a0cc0cf5b4a

mv ~/distribute-0.7.3.zip  ${CHROOT}/root/

cd ${CHROOT}/root/
unzip distribute-0.7.3.zip

cd
chroot ${CHROOT}
```

*output example>*
```bash
[root@master:~]# echo $CHROOT
/opt/ohpc/admin/images/centos7.4
[root@master:~]#
[root@master:~]# mv ~/distribute-0.7.3.zip  ${CHROOT}/root/
[root@master:~]# cd ${CHROOT}/root/
[root@master:root]# unzip distribute-0.7.3.zip
[root@master:root]# cd
[root@master:~]# chroot $CHROOT
[root@master:/]#
```

```bash
cd ~
cd distribute-0.7.3
python setup.py install

```



## # Install to TensorFlow





# END.
