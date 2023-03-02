---
layout: single
title: "RHCSA Practice Test with answer"
categories: os
tags: [2023/03/02, 운영체제, RHCSA, CentOS, 레드햇]
toc: true
toc_sticky: true
author_profile: true
comments: true
share: true
related: true
published: true
hidden: false
sidebar: 
    nav: "docs"
---

## 1. 서론  

&nbsp;&nbsp;&nbsp;&nbsp; [RCHSA 기술개요](https://www.redhat.com/ko/services/training/ex200-red-hat-certified-system-administrator-rhcsa-exam?section=%EB%AA%A9%ED%91%9C) RHCSA를 취득하기 위해 필요한 기술들이 나와있는 공홈 페이지다. 그에 따라 기존에 공부한 내용들을 연습하고 수정할 부분은 수정하여 최종적으로 모아 연습해볼 수 있도록 만든 포스팅이다.

## 2. 본론  

```
# systemctl get-defalut
# systemctl set-default graphical.target //재부팅시 반영
# systemctl isolate graphical.target //현재 세션에 즉시 반영
```

### 1-1. Emegerncy Mode

```
Enter emergency mode and do system recovery process.
Root Passwd : 'keduit'
```

```
Fine line end with linux, add 'systemd.unit=emergency.target'
ctrl + d
keduit
```

### 1-2. Configure Host Name, IP Address, Gateway and DNS

```
IPv4 address: 172.25.250.10
Subnet mask: 255.255.255.0
Default Gateway: 172.25.250.254
DNS Server: 172.25.250.254
Host Name: node1.domain250.example.com
```

```
# hostnamectl status
# hostnamectl set-hostname node1.domain250.example.com 
# hostnamectl status
# ip a
# nmtui
Change hostname as above.
Select ethernet one and then change all as above.
Automatic -> Manual
Check 'Automatically Connect' //X표시=enable
# nmcli con up 'Wired connection 1'
# systemctl restart NetworkManager
# systemctl status NetworkManager
# ip a
# ip route
# cat /etc/resolv.conf
# ssh root@172.25.250.10 //check
```

### 1-3. Base Repo

```
Base = https://mirror.stream.centos.org/9-stream/BaseOS/
Appstream = https://mirror.stream.centos.org/9-stream/AppStream/
gpgkey = x.x.x.x
Configure base repository with appstream. And then activate gpgkey as above.
```

```
# cd /etc/yum.repos.d
# mv *.repo /tmp/
# vim local.repo //파일 생성 후내용수정. 확실하진 않으나 기존에 연습했던 dnf config-manager, yum config-manager는 시험에서 사용 못한다는 글을봄

[BaseOS]
name=BaseOS
baseurl=https://mirror.stream.centos.org/9-stream/BaseOS/
enabled=1
gpgcheck=0
gpgkey=x.x.x.x //gpgkey 주어지면 gpgcheck 1로 변경

[AppStream]
name=AppStream
baseurl=https://mirror.stream.centos.org/9-stream/AppStream/
enabled=1
gpgcheck=0
gpgkey=x.x.x.x //gpgkey 주어지면 gpgcheck 1로 변경

# dnf clean all //remove cache
# dnf repolist all
# dnf update -y

//Optional
# dnf config-manager --add-repo=https://mirror.stream.centos.org/9-stream/BaseOS/
# dnf config-manager --add-repo=https://mirror.stream.centos.org/9-stream/AppStream/
```

### 1-4. SELinux

```
httpd's default directory is /var/www/html, do not delete related files and directories.
Connect to this server through web browser, and check if html files in /var/www/html/ are visible normally.
httpd's default port is 82, do not change.
httpd have to be on after reboot.
SELinux is Enforcing now, this makes httpd failed.
```

```
# dnf install -y httpd
# systemctl enable --now httpd
# systemctl status httpd
# semanage port -a -t http_port_t -p tcp 82
# semanage port -l //SELinux에 등록된 port 중 82번이 http용으로 추가됐는지 확인
# firewall-cmd --list-all //firewall통과 가능한 목록 조회
# firewall-cmd --add-service=http --permanent
# firewall-cmd --add-port=82/tcp --permanent
# firewall-cmd --reload
# man semanage fcontext | grep \#
# semanage fcontext -m -t httpd_sys_contect_t "/var/www/html/file1"
# restorecon -Rv /var/www/html/
# systemctl restart httpd
# systemctl enable httpd
# systemctl status httpd
// 웹브라우저 상에서 ip:82/file1 으로 잘 보이나 확인 
```

### 1-5. Add User(Group)

```
Add 3 users: kim, kang, choi(user id=1111).
Those 3 user's password is 'keduit'
The requirements: The Additional group of the two users: kim, kang is the keduit group(group id=111). The user: choi's login shell should be non-interactive.
Add user 'teacher' who can use sudo command without passwd.
```

```
# groupadd -g 111 keduit
# useradd -G keduit kim
# useradd -G keduit kang
# useradd -u 1111 choi
# usermod -s /sbin/nologin choi
# cat /etc/group
# echo 'keduit' | passwd --stdin kim
# echo 'keduit' | passwd --stdin kang
# echo 'keduit' | passwd --stdin choi
# useradd teacher
# echo 'teacher ALL=NOPASSWD:ALL' >> /etc/sudoers
# su teacher
# sudo useradd testuser
```

### 1-6. Crontab

```
Configure a task: plan to run below command every 2mins by user kim
logger RH200 exam
```

```
# systemctl status crond
# stytemctl enable --now crond
# crontab -u kim -e
*/2 * * * * logger RH200 exam
# crontab -u kim -l
```

### 1-7. Shared directory

```
Make directory named '/home/shared'
Only 'keduit' group can read and write on above dir.
Made files in shared dir are automatically owned by group 'keduit'
Any other users can do nothing on that.
But root can do anything.
```

```
# mkdir -p /home/shared
# ls -ltr /home
//기본적으로 drwxr-xr-x 권한으로 파일이 생기는것을 확인 가능
# chwon -R root:keduit /home/shared
# chmod -R 2770 /home/shared
# ls -ltr /home
# touch /home/shared/testfile01
# ls -ltr /home/shared
```

### 1-8. File permission

```
Copy /etc/fstab to /var/tmp. Change owner group to 'keduit', the user 'kim' could read, write and modify it, while user 'kang' and any other user without any permission. 
```

```
# cp /etc/fstab /var/tmp
# chgrp keduit /var/tmp/fstab
# setfacl -m u:kim:rwx /var/tmp/fstab
# setfacl -m u:kang:- /var/tmp/fstab
# setfacl -m o::- /var/tmp/fstab
# getfacl /var/tmp/fstab
```

### 1-9. NTP(Chrony)

```
Configure ntp client to sync time through kr.pool.ntp.org
```

```
# timedatectl
# systemctl status chronyd
# firewall-cmd --list-all
# firewall-cmd --add-service=ntp --permananet
# firewall-cmd --reload
# firewall-cmd --list-all
# find / -name chrony.*
# vim /etc/chrony.conf
server kr.pool.ntp.org iburst //추가
# systemctl enable --now chronyd
# systemctl restart chronyd
# hwclock -w
# chronyc sources
# timedatectl
// System clock synchronized:yes, NTP service: active 2가지 확인
```

### 1-10. Find files

```
Make directory /root/findfiles
Find the files owned by user 'kim', and copy it to /root/findfiles
```

```
# mkdir -p /root/findfiles
# find / -user kim | xargs -t cp /root/findfiles
```

### 1-11. Find String

```
Find the rows that contain pizza from file /etc/pizzafile, and write it to the /tmp/pizza.txt
```

```
# grep pizza /etc/pizzafile > /tmp/pizza.txt
```

### 1-12. nfs / autofs 

```
다음 요구 사항에 따라 servera에서 autofs 자동 마운트를 구성합니다.
1) serverb는 NFS 공유 디렉토리 /rhome을 통해 시스템에 연결합니다. 이 파일 시스템에는 사용자 ldapuser0에 대해 사전 구성된 홈 디렉토리가 포함됩니다.
2) 기본 사용자 ldapuser0의 홈 디렉토리는 다음과 같아야 합니다. 로컬 /rhome/ldapuser0 디렉토리에 자동으로 마운트됩니다.
3) 기본 사용자 ldapuser0의 홈 디렉토리는 serverb.lab.example.com:/rhome/ldapuser0입니다.
4) 기본 사용자 ldapuser0의 홈 디렉토리는 다음 위치에 자동으로 마운트되어야 합니다. local /rhome/ldapuser0 디렉토리
5) 마운트된 홈 디렉토리는 읽고 쓸 수 있어야 합니다.
```

```
# showmount -e serverb.lab.example.com
# dnf install -y autofs
# systemctl enable --now autofs
# systemctl status autofs
# find / -name auto.*
# vim /etc/auto.master
/rhome  /etc/auto.nfs
# vim /etc/auto.nfs
ldapuser0 -rw,sync serverb.lab.example.com:/rhome/ldapuser0
# systemctl restart autofs
# mkdir /rhome/
# cd /rhome/
# ls
# cd ldapuser0
# pwd
# df -h
```

```
//Optional
//server side. 공유하고자 하는 폴더 /share
# dnf install -y nfs-utils
# dnf install -y autofs
# systemctl status nfs-server
# systemctl status autofs
# mkdir -p /share
# echo "test" > /share/share1.txt
# echo "test2" > /share/share2.txt
# chmod 777 /share
# vim /etc/exports
/share xxx.xxx.xxx.xxx(rw,sync,no_root_squash) //client ip
# firewall-cmd --add-service={nfs,mountd,rpc-bind} --permanent
# firewall-cmd --reload
# systemctl enable --now nfs-server
# systemctl enable --now autofs
# exportfs -avr 
```

```
//client side
# dnf install -y nfs-utils
# dnf install -y autofs
# systemctl status nfs-server
# systemctl enable --now nfs-server
# systemctl status autofs
# systemctl enable --now autofs
# firewall-cmd --add-service={nfs,mountd,rpc-bind} --permanent
# firewall-cmd --reload
# showmount -e xxx.xxx.xxx.xxx //client ip
# vim /etc/auto.master
/misc /etc/auto.misc //해당 라인 주석처리
/auto_mount /etc/auto.misc //auto_mount는 나한테 보여질 디렉토리
# vim /etc/auto.misc
accessnfs -rw,soft,intr xxx.xxx.xxx.xxx:/share //server ip
# ls /
# systemctl restart autofs
# ls /
# cd /auto_mount
# ls
# cd accessnfs
```

### 1-13. podman

```
//문제1
자동으로 시작하도록 컨테이너 구성(볼륨)
레지스트리 서버의 rsyslog이미지를 logserver이름이 지정된 컨테이너를 만듭니다.
wallah사용자를 위해 systemd 서비스를 구성합니다.
서비스 이름이 지정되고 container-logserver개입 없이 시스템 재부팅 시 자동으로 시작됩니다.
```

```
//문제2
컨테이너에 대한 영구 스토리지(A 볼륨) 구성

이전 작업의 서비스를 다음과 같이 확장합니다.

시스템 재부팅 시 데이터를 보존하도록 호스트 시스템의 저널 저널을 구성하고 로깅 서비스를 다시 시작합니다.
호스트 /var/log/journal디렉토리 아래로 시작하는 모든 파일을 다음으로*.journal 복사합니다./home/wallah/container_logfile
시작 시 컨테이너 아래에 자동으로 /home/wallah/container_logfile마운트/var/log/journal
```

```
# systemctl list-unit-files | greep journal
# systemctl status systemd-journald
# man journald.conf
# vim /etc/systemd/journald.conf
Storage = auto
# man journald.conf
# ll -d /run/log/journal
# mkdir /var/log/journal
# chown root:systemd-journal /var/log/journal
# chmod g+s /var/log/journal
# systemctl restart systemd-journald
# ls /var/log/journal
# ls /run/log/journal
# cp /var/log/journal/*/*.journal /home/wallah/container_logfile/
# chown -R wallah ~wallah
# ll -Z /home/wallah/container_logfile/

# ssh wallah@localhost
# podman login registry.domain250.example.com
# podman search registry.domain250.example.com
# podman run -d --name logserver \
# podman stop logserver
# loginctl enable-linger
# loginctl show-user wallah
# man systemd.unit | grep config.*user
# mkdir -p ~/.config/systemd/user/
# cd ~/.config/systemd/user/
# podman generate systemd -n logserver -f
# systemctl --user enable --now container-logserver
# systemctl --user status container-logserver
# podman exec logserver ls /var/log/journal
# reboot
# ps aux | grep -n podman
```

```
//문제3
레지스트리 서버의 rsyslog이미지를 logger이름이 지정된 컨테이너를 만듭니다.
wallah사용자를 위해 systemd 서비스를 구성합니다.
서비스 이름이 지정되고 container-logger개입 없이 시스템 재부팅 시 자동으로 시작됩니다.
시작 시 컨테이너 아래에 자동으로 /home/wallah/var_log마운트/var/log
컨테이너에서 명령 실행podman exec logger logger -p authpriv.info SUIBIAN
```

```
# ssh wallah@localhost
# podman login -u admin -p redhat321
# podman search registry.domain250.example.com/
# podman run -d --name logger \
# podman stop logger
# loginctl enable-linger
# loginctl show-user wallah
# man systemd.unit | grep =A 20 \\--user
# mkdir -p ~/.confing/systemd/user/
# cd ~/.config/systemd/user/
# podman generate systemd -n logger -f
# systemctl --user enable --now container-logger
# systemctl --user status container-logger
# podman exec logger logger -p authpriv.info SUIBIAN
# grep authpriv /etc/rsyslog.conf
# grep SUIBIAN /home/wallah/var_log/secure
# reboot
# ps -aux | grep podman
```

### 1-14 Password Aging Control

```
Change PASS_MAX_DAYS to 20
```

```
# find / -name login.*
# vim /etc/login.defs
/PASS_MAX로 해당 라인 찾아서 20으로 바꿔주기
```

### 1-15 Archive(Backup)

```
Create a backup file named /root/backuptest.tar, contains the content of /usr/localtest, tar must use bzip2 to compress.
```

```
# man tar | grep bzip2
# tar -cfj /root/backuptest.tar /usr/localtest
// c(create), j(bzip2), tar [파일명.tar] [대상 폴더명] 
```

### 2-1. Reset root password

```
Reset root passwd to 'keduit'
```

```
grub -> e
In linux line change ro(crash~ 앞) to rw and add init=/sysroot/bin/bash at the end of the line. 
ctrl + x
# mount -o remount,rw /sysroot
# chroot /sysroot
# passwd
keduit
keduit
# touch /.autorelabel
ctrl + d //twice
```

```
//Optional
grub -> rescue mode -> e
In linux~ line, add rd.break to the end of the line.
ctrl + x
# mount -o remount,rw /sysroot
# chroot /sysroot
# passwd
# touch /.autorelabel
ctrl + d //twice
```

### 2-2. Base Repo

```
Base = https://mirror.stream.centos.org/9-stream/BaseOS/
Appstream = https://mirror.stream.centos.org/9-stream/AppStream/
gpgkey = x.x.x.x
Configure base repository with appstream. And then activate gpgkey as above.
```

```
# cd /etc/yum.repos.d
# mv *.repo /tmp/
# vim local.repo //파일 생성 후내용수정. 확실하진 않으나 기존에 연습했던 dnf config-manager, yum config-manager는 시험에서 사용 못한다는 글을봄

[BaseOS]
name=BaseOS
baseurl=https://mirror.stream.centos.org/9-stream/BaseOS/
enabled=1
gpgcheck=0
gpgkey=x.x.x.x //gpgkey 주어지면 gpgcheck 1로 변경

[AppStream]
name=AppStream
baseurl=https://mirror.stream.centos.org/9-stream/AppStream/
enabled=1
gpgcheck=0
gpgkey=x.x.x.x //gpgkey 주어지면 gpgcheck 1로 변경

# dnf clean all //remove cache
# dnf repolist all
# dnf update -y

//Optional
# dnf config-manager --add-repo=https://mirror.stream.centos.org/9-stream/BaseOS/
# dnf config-manager --add-repo=https://mirror.stream.centos.org/9-stream/AppStream/
```

### 2-3. LVM(Logical Volume Management)

물리 저장소 추가 -> (fdisk로 파티션 생성, 시스템의 안정성을 위해 필요하다고 하는데 요즘은 물리저장소 추가할때는 굳이 안하는 경우도 많다고 함) -> pvcreate로 -> vgcreate -> lvcreate

#### 1. 예제1

```
Change the logical volume capacity 'lvo1' from 190M to 300M. And the size of the floating range should set between 280 and 320. (This logical volume has been mounted in advance.)
```

```
// 직접 만드는 과정
# lsblk //list block devices
# pvcreate /dev/sdb
# pvcreate /dev/sdc
# pvs
# vgcreate VG1 /dev/sdb /dev/sdc
# vgs
# lvcreate -n lvo1 -L 190M VG1
# lvs
# mkdir /lv
# mkfs.xfs /dev/VG1/lvo1
# vim /etc/fstab
/dev/VG1/lvo1    /lv xfs defaults    0 0
# mount -a
```

```
# vgs //check available space
# df -h
# lvs
# lvextend -L +110M /dev/VG1/lvo1
# vgs 
# resize2fs /dev/VG1/lvo1
```

#### 2. 예제2

```
Remove logical volume and group we made above, and then make new one set 8M as a extends. Divided a volume group containing 50 extends(named 'lvshare') on volume group 'vgtest'. And the size of the floating range should set between 380M and 400M.
```

```
# lsblk //list block devices
# pvdisplay //남는공간 확인
# umount /lv
# lvremove /dev/VG1/lvo1
# vim /etc/fstab
위에 추가했던 부분 주석처리
# vgremove VG1
# vgcreate -s 8M VG1 /dev/sdb /dev/sdc
# vgdisplay // Check PE Size(Physical Extent, 물리볼륨의 블록단위)
# lvcreate -n lvshare -l 50 /dev/VG1
# mkfs.xfs /dev/VG1/lvshare
# vim /etc/fstab
/dev/VG1/lvshare    /lv xfs defaults    0 0
# mount -a
```

### 2-4. Swap Partition

#### 1. 예제1

```
Create a 1G partition and which take effect automatically at boot-start. 
```

```
# lsblk
# fdisk /dev/sdd
n -> enter -> enter -> enter -> +1G -> w
# partprobe /dev/sdd
# lsblk
# mkdir /newswap
# mkfs.xfs /dev/sdd1 // /dev/sdd의 첫번째 파티션에 시스템파일 생성
# mount /dev/sdd1 /newswap
# vim /etc/fstab
/dev/sdd1 /newswap  xfs defaults    0 0
```

#### 2. 예제2

```
Add a swap partition of size 750Mb. Do not delete the existing swap
```

```
# free -h
# fdisk /dev/sdd
n -> enter -> enter -> enter -> +750Mb -> t -> 2 -> L -> 82(Linux swap) -> w
# partprobe /dev/sdd
# lsblk
# mkswap /dev/sdd2
# swapon /dev/sdd2
# vim /etc/fstab
/dev/sdd2    swap    swap    defaluts    0 0
# free -h
```

### 2-5. tuned

```
Change tuned to recommended
```

```
# dnf install -y tuned
# systemctl enable --now tuned
# tuned-adm recommend //tuned recommend the most suitable profile for my system
# tuned-adm profile virtual-guest //virtual-guest profile 적용
# tuned-adm active //View the current active Tuned profile
# reboot
# tuned-adm verify
```

### 2-6. ip forward

```
Please open the ip_forward, and take effect permanently.
```

```
# sysctl net.ipv4.ip_forward
// '0' means off, '1' means on
# sysctl -w net.ipv4.ip_forward=1 
# vim /etc/sysctl.conf
net.ipv4.ip_forward=1 //Add this line to the bottom of the file
```

### 2-7. Hard Link, Soft(Symbolic) Link

```
Make a /etc/linktest.txt 
Make a Hard Link named hlink at /etc/linkexam/
Make a Soft Link named slink at /etc/linkexam/
```

```
# ln /etc/linktest.txt /etc/linkexam/hlink 
# ln -s /etc/linktest.txt /etc/linkexam/slink 
```

### 2-8. UMASK

```
Set a permission for file to 622, for dir to 733 permanently.
It's only affect user 'kim'
```

```
# su kim
# umask 044 //file's default umask is 666, dir's 777
```

## 3. 결론  

&nbsp;&nbsp;&nbsp;&nbsp; 쉽지 않다.

```bash
클라우드 개발자를 넘어 데브옵스를 꿈꾸는 이제 막 공부를 시작한 초보 엔지니어입니다. 틀린 점이 있으면 친절하게 댓글 부탁드립니다. :)
```

---
