# ProfHW34
Домашнее задание №3. Работа с LVM
---
1. Настроить LVM в Ubuntu 24.04 Server
```
root@UbuntuTestVirt:/mnt/01# hostnamectl
 Static hostname: UbuntuTestVirt
       Icon name: computer-vm
         Chassis: vm 
      Machine ID: 30209530c4164a2fb1487a120a480866
         Boot ID: e89e38be2bbc4eb4968329ba3a73f76c
  Virtualization: oracle
Operating System: ;;https://www.ubuntu.com/Ubuntu 24.04.2 LTS;;
          Kernel: Linux 6.8.0-60-generic
    Architecture: x86-64
 Hardware Vendor: innotek GmbH
  Hardware Model: VirtualBox
Firmware Version: VirtualBox
   Firmware Date: Fri 2006-12-01
    Firmware Age: 18y 5month 3w 3d
```
```
root@UbuntuTestVirt:~# lvm version
  LVM version:     2.03.16(2) (2022-05-18)
  Library version: 1.02.185 (2022-05-18)
  Driver version:  4.48.0
  Configuration:   ./configure --build=x86_64-linux-gnu --prefix=/usr --includedir=${prefix}/include --mandir=${prefix}/share/man --infodir=${prefix}/share/info --sysconfdir=/etc --localstatedir=/var --disable-option-checking --disable-silent-rules --libdir=${prefix}/lib/x86_64-linux-gnu --runstatedir=/run --disable-maintainer-mode --disable-dependency-tracking --prefix=/usr --exec-prefix=/usr --with-usrlibdir=/usr/lib/x86_64-linux-gnu --with-optimisation=-O2 --with-cache=internal --with-device-uid=0 --with-device-gid=6 --with-device-mode=0660 --with-default-pid-dir=/run --with-default-run-dir=/run/lvm --with-default-locking-dir=/run/lock/lvm --with-thin=internal --with-thin-check=/usr/sbin/thin_check --with-thin-dump=/usr/sbin/thin_dump --with-thin-repair=/usr/sbin/thin_repair --enable-applib --enable-blkid_wiping --enable-cmdlib --enable-dmeventd --enable-editline --enable-lvmlockd-dlm --enable-lvmlockd-sanlock --enable-lvmpolld --enable-notify-dbus --enable-pkgconfig --enable-udev_rules --enable-udev_sync --disable-readline
```
---
2. Создать Physical Volume, Volume Group и Logical Volume  
В системе пристсвтуют несколько pvs, vgs, lvs. Для создания pvs я буду использовать диски sde и sdf. Создам vgs с именем vg-hw. И на ней создам 3 lvs.
```
root@UbuntuTestVirt:~# pvcreate /dev/sde /dev/sdf
  Physical volume "/dev/sde" successfully created.
  Physical volume "/dev/sdf" successfully created.
root@UbuntuTestVirt:~# pvs
  PV         VG        Fmt  Attr PSize    PFree
  /dev/sda3  ubuntu-vg lvm2 a--    17,71g   7,71g
  /dev/sdb   vg0       lvm2 a--  1020,00m      0
  /dev/sdc   vg0       lvm2 a--  1020,00m 140,00m
  /dev/sde             lvm2 ---     1,00g   1,00g
  /dev/sdf             lvm2 ---     1,00g   1,00g
```
```
root@UbuntuTestVirt:~# vgcreate vg-hw /dev/sde /dev/sdf
  Volume group "vg-hw" successfully created
root@UbuntuTestVirt:~# vgs
  VG        #PV #LV #SN Attr   VSize  VFree
  ubuntu-vg   1   1   0 wz--n- 17,71g   7,71g
  vg-hw       2   0   0 wz--n-  1,99g   1,99g
  vg0         2   3   0 wz--n-  1,99g 140,00m
```
```
root@UbuntuTestVirt:~# lvcreate /dev/vg-hw -n hw1 -L 500M
  Logical volume "hw1" created.
root@UbuntuTestVirt:~# lvcreate /dev/vg-hw -n hw2 -L 300M
  Logical volume "hw2" created.
root@UbuntuTestVirt:~# lvcreate /dev/vg-hw -n hw3 -L 300M
  Logical volume "hw3" created.
root@UbuntuTestVirt:~# lvs
  LV        VG        Attr       LSize    Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  ubuntu-lv ubuntu-vg -wi-ao----   10,00g
  hw1       vg-hw     -wi-a-----  500,00m
  hw2       vg-hw     -wi-a-----  300,00m
  hw3       vg-hw     -wi-a-----  300,00m
  lv0       vg0       -wi-ao---- 1000,00m
  lv1       vg0       -wi-ao----  600,00m
  lv2       vg0       -wi-ao----  300,00m
```
---
3. Отформатировать и смонтировать файловую систему
Создаём фс на всех lvs и монтируем их в систему.
```
root@UbuntuTestVirt:~# mkfs.ext4 /dev/vg-hw/hw1
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 128000 4k blocks and 128000 inodes
Filesystem UUID: 09f956d1-8723-4351-91b7-322d9ea58f2d
Superblock backups stored on blocks:
        32768, 98304

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

root@UbuntuTestVirt:~# mkfs.ext4 /dev/vg-hw/hw2
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 76800 4k blocks and 76800 inodes
Filesystem UUID: 39ec65f9-e02f-46f2-af6d-78705b4e138e
Superblock backups stored on blocks:
        32768

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

root@UbuntuTestVirt:~# mkfs.ext4 /dev/vg-hw/hw3
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 76800 4k blocks and 76800 inodes
Filesystem UUID: 715c5d8f-2564-49dd-bc64-774dd003e55d
Superblock backups stored on blocks:
        32768

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done
```
```
root@UbuntuTestVirt:~# mount /dev/vg-hw/hw1 /mnt/01
root@UbuntuTestVirt:~# mount /dev/vg-hw/hw2 /mnt/02
root@UbuntuTestVirt:~# mount /dev/vg-hw/hw3 /mnt/03
root@UbuntuTestVirt:~# findmnt
TARGET                          SOURCE                            FSTYPE      OPTIONS
/                               /dev/mapper/ubuntu--vg-ubuntu--lv ext4        rw,relatime
├─/sys                          sysfs                             sysfs       rw,nosuid,nodev,noexec,relatime
│ ├─/sys/kernel/security        securityfs                        securityfs  rw,nosuid,nodev,noexec,relatime
│ ├─/sys/fs/cgroup              cgroup2                           cgroup2     rw,nosuid,nodev,noexec,relatime,nsdelegate,memory_recursiveprot
│ ├─/sys/fs/pstore              pstore                            pstore      rw,nosuid,nodev,noexec,relatime
│ ├─/sys/fs/bpf                 bpf                               bpf         rw,nosuid,nodev,noexec,relatime,mode=700
│ ├─/sys/kernel/tracing         tracefs                           tracefs     rw,nosuid,nodev,noexec,relatime
│ ├─/sys/kernel/debug           debugfs                           debugfs     rw,nosuid,nodev,noexec,relatime
│ │ └─/sys/kernel/debug/tracing tracefs                           tracefs     rw,nosuid,nodev,noexec,relatime
│ ├─/sys/kernel/config          configfs                          configfs    rw,nosuid,nodev,noexec,relatime
│ └─/sys/fs/fuse/connections    fusectl                           fusectl     rw,nosuid,nodev,noexec,relatime
├─/proc                         proc                              proc        rw,nosuid,nodev,noexec,relatime
│ ├─/proc/sys/fs/binfmt_misc    systemd-1                         autofs      rw,relatime,fd=32,pgrp=1,timeout=0,minproto=5,maxproto=5,direct,pipe_ino=4219
│ │ └─/proc/sys/fs/binfmt_misc  binfmt_misc                       binfmt_misc rw,nosuid,nodev,noexec,relatime
│ └─/proc/fs/nfsd               nfsd                              nfsd        rw,relatime
├─/dev                          udev                              devtmpfs    rw,nosuid,relatime,size=969424k,nr_inodes=242356,mode=755,inode64
│ ├─/dev/pts                    devpts                            devpts      rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=000
│ ├─/dev/shm                    tmpfs                             tmpfs       rw,nosuid,nodev,inode64
│ ├─/dev/hugepages              hugetlbfs                         hugetlbfs   rw,nosuid,nodev,relatime,pagesize=2M
│ └─/dev/mqueue                 mqueue                            mqueue      rw,nosuid,nodev,noexec,relatime
├─/run                          tmpfs                             tmpfs       rw,nosuid,nodev,noexec,relatime,size=201508k,mode=755,inode64
│ ├─/run/lock                   tmpfs                             tmpfs       rw,nosuid,nodev,noexec,relatime,size=5120k,inode64
│ ├─/run/user/1000              tmpfs                             tmpfs       rw,nosuid,nodev,relatime,size=201504k,nr_inodes=50376,mode=700,uid=1000,gid=1000,inode64
│ └─/run/rpc_pipefs             sunrpc                            rpc_pipefs  rw,relatime
├─/boot                         /dev/sda2                         ext4        rw,relatime
├─/mnt/01                       /dev/mapper/vg--hw-hw1            ext4        rw,relatime
├─/mnt/02                       /dev/mapper/vg--hw-hw2            ext4        rw,relatime
└─/mnt/03                       /dev/mapper/vg--hw-hw3            ext4        rw,relatime
```
---
4. Расширить файловую систему за счёт нового диска  
Запишем данные на все lvs. После этого добавим ещё один диск в vg-hw и расширим все lvs.
```
root@UbuntuTestVirt:~# cp /home/starsh/*.mp3 /mnt/01
cp: error writing '/mnt/01/11.mp3': No space left on device
cp: error writing '/mnt/01/12.mp3': No space left on device
cp: error writing '/mnt/01/13.mp3': No space left on device
cp: error writing '/mnt/01/14.mp3': No space left on device
cp: error writing '/mnt/01/15.mp3': No space left on device
root@UbuntuTestVirt:~# cp -r /var/log/* /mnt/02
root@UbuntuTestVirt:~# cp -r /var/log/* /mnt/03
root@UbuntuTestVirt:~# df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              197M  1,2M  196M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  9,8G  5,7G  3,6G  62% /
tmpfs                              984M     0  984M   0% /dev/shm
tmpfs                              5,0M     0  5,0M   0% /run/lock
/dev/sda2                          1,7G  185M  1,5G  12% /boot
tmpfs                              197M   12K  197M   1% /run/user/1000
/dev/mapper/vg--hw-hw1             452M  442M     0 100% /mnt/01
/dev/mapper/vg--hw-hw2             265M  244M  304K 100% /mnt/02
/dev/mapper/vg--hw-hw3             265M  244M  300K 100% /mnt/03
```
```
root@UbuntuTestVirt:~# pvcreate sdd
  No device found for sdd.
root@UbuntuTestVirt:~# pvcreate /dev/sdd
  Physical volume "/dev/sdd" successfully created.
root@UbuntuTestVirt:~# pvs
  PV         VG        Fmt  Attr PSize    PFree
  /dev/sda3  ubuntu-vg lvm2 a--    17,71g   7,71g
  /dev/sdb   vg0       lvm2 a--  1020,00m      0
  /dev/sdc   vg0       lvm2 a--  1020,00m 140,00m
  /dev/sdd             lvm2 ---     1,00g   1,00g
  /dev/sde   vg-hw     lvm2 a--  1020,00m 220,00m
  /dev/sdf   vg-hw     lvm2 a--  1020,00m 720,00m
root@UbuntuTestVirt:~# vgextend /dev/vg-hw /dev/sdd
  Volume group "vg-hw" successfully extended
root@UbuntuTestVirt:~# vgs
  VG        #PV #LV #SN Attr   VSize  VFree
  ubuntu-vg   1   1   0 wz--n- 17,71g   7,71g
  vg-hw       3   3   0 wz--n- <2,99g   1,91g
  vg0         2   3   0 wz--n-  1,99g 140,00m
```
---
5. Выполнить resize
```
root@UbuntuTestVirt:~# lvextend /dev/vg-hw/hw1 -L +1G
  Size of logical volume vg-hw/hw1 changed from 500,00 MiB (125 extents) to <1,49 GiB (381 extents).
  Logical volume vg-hw/hw1 successfully resized.
root@UbuntuTestVirt:~# resize2fs /dev/vg-hw/hw1
resize2fs 1.47.0 (5-Feb-2023)
Filesystem at /dev/vg-hw/hw1 is mounted on /mnt/01; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/vg-hw/hw1 is now 390144 (4k) blocks long.

root@UbuntuTestVirt:~# lvextend /dev/vg-hw/hw2 -L +300M -r
  Size of logical volume vg-hw/hw2 changed from 300,00 MiB (75 extents) to 600,00 MiB (150 extents).
  Logical volume vg-hw/hw2 successfully resized.
resize2fs 1.47.0 (5-Feb-2023)
Filesystem at /dev/mapper/vg--hw-hw2 is mounted on /mnt/02; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/mapper/vg--hw-hw2 is now 153600 (4k) blocks long.

root@UbuntuTestVirt:~# lvextend /dev/vg-hw/hw3 -L +300M -r
  Size of logical volume vg-hw/hw3 changed from 300,00 MiB (75 extents) to 600,00 MiB (150 extents).
  Logical volume vg-hw/hw3 successfully resized.
resize2fs 1.47.0 (5-Feb-2023)
Filesystem at /dev/mapper/vg--hw-hw3 is mounted on /mnt/03; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/mapper/vg--hw-hw3 is now 153600 (4k) blocks long.
```
```
root@UbuntuTestVirt:~# vgs
  VG        #PV #LV #SN Attr   VSize  VFree
  ubuntu-vg   1   1   0 wz--n- 17,71g   7,71g
  vg-hw       3   3   0 wz--n- <2,99g 336,00m
  vg0         2   3   0 wz--n-  1,99g 140,00m
root@UbuntuTestVirt:~# lvs
  LV        VG        Attr       LSize    Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  ubuntu-lv ubuntu-vg -wi-ao----   10,00g
  hw1       vg-hw     -wi-ao----   <1,49g
  hw2       vg-hw     -wi-ao----  600,00m
  hw3       vg-hw     -wi-ao----  600,00m
  lv0       vg0       -wi-a----- 1000,00m
  lv1       vg0       -wi-a-----  600,00m
  lv2       vg0       -wi-a-----  300,00m
root@UbuntuTestVirt:~# df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              197M  1,2M  196M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  9,8G  5,7G  3,6G  62% /
tmpfs                              984M     0  984M   0% /dev/shm
tmpfs                              5,0M     0  5,0M   0% /run/lock
/dev/sda2                          1,7G  185M  1,5G  12% /boot
tmpfs                              197M   12K  197M   1% /run/user/1000
/dev/mapper/vg--hw-hw1             1,4G  442M  896M  34% /mnt/01
/dev/mapper/vg--hw-hw2             553M  244M  282M  47% /mnt/02
/dev/mapper/vg--hw-hw3             553M  244M  282M  47% /mnt/03
```
---
6. Проверить корректность работы  
Проверка сохранности данных. Вывод содержимого /mnt/01 /mnt/02 /mnt/03
```
root@UbuntuTestVirt:~# ll /mnt/01
total 452580
drwxr-xr-x 3 root root     4096 мая 26 01:25 ./
drwxr-xr-x 8 root root     4096 мая 16 00:02 ../
-rw-r--r-- 1 root root  4231965 мая 26 01:25 01.mp3
-rw-r--r-- 1 root root  4629026 мая 26 01:25 02.mp3
-rw-r--r-- 1 root root 33065924 мая 26 01:25 03.mp3
-rw-r--r-- 1 root root 52731948 мая 26 01:25 04.mp3
-rw-r--r-- 1 root root 74349842 мая 26 01:25 05.mp3
-rw-r--r-- 1 root root 22704716 мая 26 01:25 06.mp3
-rw-r--r-- 1 root root 57262626 мая 26 01:25 07.mp3
-rw-r--r-- 1 root root 86963850 мая 26 01:25 08.mp3
-rw-r--r-- 1 root root 57529075 мая 26 01:25 09.mp3
-rw-r--r-- 1 root root 64194479 мая 26 01:25 10.mp3
-rw-r--r-- 1 root root  5709824 мая 26 01:25 11.mp3
-rw-r--r-- 1 root root        0 мая 26 01:25 12.mp3
-rw-r--r-- 1 root root        0 мая 26 01:25 13.mp3
-rw-r--r-- 1 root root        0 мая 26 01:25 14.mp3
-rw-r--r-- 1 root root        0 мая 26 01:25 15.mp3
drwx------ 2 root root    16384 мая 26 01:13 lost+found/
root@UbuntuTestVirt:~# ll /mnt/02
total 1372
drwxr-xr-x 11 root root   4096 мая 26 01:25 ./
drwxr-xr-x  8 root root   4096 мая 16 00:02 ../
-rw-r--r--  1 root root      0 мая 26 01:25 alternatives.log
-rw-r--r--  1 root root   8879 мая 26 01:25 alternatives.log.1
-rw-r--r--  1 root root   2519 мая 26 01:25 alternatives.log.2.gz
-rw-r-----  1 root root      0 мая 26 01:25 apport.log
drwxr-xr-x  2 root root   4096 мая 26 01:25 apt/
-rw-r-----  1 root root   8432 мая 26 01:25 auth.log
-rw-r-----  1 root root  40946 мая 26 01:25 auth.log.1
-rw-r-----  1 root root   2494 мая 26 01:25 auth.log.2.gz
-rw-r-----  1 root root   1090 мая 26 01:25 auth.log.3.gz
-rw-r-----  1 root root   1010 мая 26 01:25 auth.log.4.gz
-rw-r--r--  1 root root  61229 мая 26 01:25 bootstrap.log
-rw-r-----  1 root root      0 мая 26 01:25 btmp
-rw-r-----  1 root root      0 мая 26 01:25 btmp.1
-rw-r-----  1 root root  85170 мая 26 01:25 cloud-init.log
-rw-r-----  1 root root   4592 мая 26 01:25 cloud-init-output.log
drwxr-xr-x  2 root root   4096 мая 26 01:25 dist-upgrade/
-rw-r-----  1 root root  57444 мая 26 01:25 dmesg
-rw-r-----  1 root root  58680 мая 26 01:25 dmesg.0
-rw-r-----  1 root root  17266 мая 26 01:25 dmesg.1.gz
-rw-r-----  1 root root  17053 мая 26 01:25 dmesg.2.gz
-rw-r-----  1 root root  16990 мая 26 01:25 dmesg.3.gz
-rw-r-----  1 root root  17336 мая 26 01:25 dmesg.4.gz
-rw-r--r--  1 root root  24796 мая 26 01:25 dpkg.log
-rw-r--r--  1 root root  54025 мая 26 01:25 dpkg.log.1
-rw-r--r--  1 root root   1642 мая 26 01:25 dpkg.log.2.gz
-rw-r--r--  1 root root  67810 мая 26 01:25 dpkg.log.3.gz
-rw-r--r--  1 root root      0 мая 26 01:25 faillog
drwxr-x---  4 root root   4096 мая 26 01:25 installer/
drwxr-xr-x  3 root root   4096 мая 26 01:25 journal/
-rw-r-----  1 root root  12104 мая 26 01:25 kern.log
-rw-r-----  1 root root  82110 мая 26 01:25 kern.log.1
-rw-r-----  1 root root  58950 мая 26 01:25 kern.log.2.gz
-rw-r-----  1 root root  14708 мая 26 01:25 kern.log.3.gz
-rw-r-----  1 root root  13712 мая 26 01:25 kern.log.4.gz
drwxr-xr-x  2 root root   4096 мая 26 01:25 landscape/
-rw-r--r--  1 root root 292292 мая 26 01:25 lastlog
drwx------  2 root root  16384 мая 26 01:13 lost+found/
drwx------  2 root root   4096 мая 26 01:25 private/
lrwxrwxrwx  1 root root     39 мая 26 01:25 README -> ../../usr/share/doc/systemd/README.logs
-rw-r-----  1 root root  40133 мая 26 01:25 syslog
-rw-r-----  1 root root 253955 мая 26 01:25 syslog.1
-rw-r-----  1 root root 116554 мая 26 01:25 syslog.2.gz
-rw-r-----  1 root root  32725 мая 26 01:25 syslog.3.gz
-rw-r-----  1 root root  24231 мая 26 01:25 syslog.4.gz
drwxr-xr-x  2 root root   4096 мая 26 01:25 sysstat/
-rw-r--r--  1 root root      0 мая 26 01:25 ubuntu-advantage-apt-hook.log
drwxr-x---  2 root root   4096 мая 26 01:25 unattended-upgrades/
-rw-r--r--  1 root root  67584 мая 26 01:25 wtmp
root@UbuntuTestVirt:~# ll /mnt/03
total 1372
drwxr-xr-x 11 root root   4096 мая 26 01:25 ./
drwxr-xr-x  8 root root   4096 мая 16 00:02 ../
-rw-r--r--  1 root root      0 мая 26 01:25 alternatives.log
-rw-r--r--  1 root root   8879 мая 26 01:25 alternatives.log.1
-rw-r--r--  1 root root   2519 мая 26 01:25 alternatives.log.2.gz
-rw-r-----  1 root root      0 мая 26 01:25 apport.log
drwxr-xr-x  2 root root   4096 мая 26 01:25 apt/
-rw-r-----  1 root root   8432 мая 26 01:25 auth.log
-rw-r-----  1 root root  40946 мая 26 01:25 auth.log.1
-rw-r-----  1 root root   2494 мая 26 01:25 auth.log.2.gz
-rw-r-----  1 root root   1090 мая 26 01:25 auth.log.3.gz
-rw-r-----  1 root root   1010 мая 26 01:25 auth.log.4.gz
-rw-r--r--  1 root root  61229 мая 26 01:25 bootstrap.log
-rw-r-----  1 root root      0 мая 26 01:25 btmp
-rw-r-----  1 root root      0 мая 26 01:25 btmp.1
-rw-r-----  1 root root  85170 мая 26 01:25 cloud-init.log
-rw-r-----  1 root root   4592 мая 26 01:25 cloud-init-output.log
drwxr-xr-x  2 root root   4096 мая 26 01:25 dist-upgrade/
-rw-r-----  1 root root  57444 мая 26 01:25 dmesg
-rw-r-----  1 root root  58680 мая 26 01:25 dmesg.0
-rw-r-----  1 root root  17266 мая 26 01:25 dmesg.1.gz
-rw-r-----  1 root root  17053 мая 26 01:25 dmesg.2.gz
-rw-r-----  1 root root  16990 мая 26 01:25 dmesg.3.gz
-rw-r-----  1 root root  17336 мая 26 01:25 dmesg.4.gz
-rw-r--r--  1 root root  24796 мая 26 01:25 dpkg.log
-rw-r--r--  1 root root  54025 мая 26 01:25 dpkg.log.1
-rw-r--r--  1 root root   1642 мая 26 01:25 dpkg.log.2.gz
-rw-r--r--  1 root root  67810 мая 26 01:25 dpkg.log.3.gz
-rw-r--r--  1 root root      0 мая 26 01:25 faillog
drwxr-x---  4 root root   4096 мая 26 01:25 installer/
drwxr-xr-x  3 root root   4096 мая 26 01:25 journal/
-rw-r-----  1 root root  12104 мая 26 01:25 kern.log
-rw-r-----  1 root root  82110 мая 26 01:25 kern.log.1
-rw-r-----  1 root root  58950 мая 26 01:25 kern.log.2.gz
-rw-r-----  1 root root  14708 мая 26 01:25 kern.log.3.gz
-rw-r-----  1 root root  13712 мая 26 01:25 kern.log.4.gz
drwxr-xr-x  2 root root   4096 мая 26 01:25 landscape/
-rw-r--r--  1 root root 292292 мая 26 01:25 lastlog
drwx------  2 root root  16384 мая 26 01:13 lost+found/
drwx------  2 root root   4096 мая 26 01:25 private/
lrwxrwxrwx  1 root root     39 мая 26 01:25 README -> ../../usr/share/doc/systemd/README.logs
-rw-r-----  1 root root  40133 мая 26 01:25 syslog
-rw-r-----  1 root root 253955 мая 26 01:25 syslog.1
-rw-r-----  1 root root 116554 мая 26 01:25 syslog.2.gz
-rw-r-----  1 root root  32725 мая 26 01:25 syslog.3.gz
-rw-r-----  1 root root  24231 мая 26 01:25 syslog.4.gz
drwxr-xr-x  2 root root   4096 мая 26 01:25 sysstat/
-rw-r--r--  1 root root      0 мая 26 01:25 ubuntu-advantage-apt-hook.log
drwxr-x---  2 root root   4096 мая 26 01:25 unattended-upgrades/
-rw-r--r--  1 root root  67584 мая 26 01:25 wtmp
```
