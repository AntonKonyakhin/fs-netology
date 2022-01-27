# Домашнее задание к занятию "3.5. Файловые системы"  

1. Узнайте о sparse (разряженных) файлах.  
выполнено
2. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?  
ответ: нет, так файл и жесткая ссылка имеют одинаковый дескриптор(inode)

3. создать новую вм
```
vagrant@vagrant:~$ lsblk
NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                    8:0    0   64G  0 disk
├─sda1                 8:1    0  512M  0 part /boot/efi
├─sda2                 8:2    0    1K  0 part
└─sda5                 8:5    0 63.5G  0 part
  ├─vgvagrant-root   253:0    0 62.6G  0 lvm  /
  └─vgvagrant-swap_1 253:1    0  980M  0 lvm  [SWAP]
sdb                    8:16   0  2.5G  0 disk
sdc                    8:32   0  2.5G  0 disk
```
4. Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.  
```
root@anton-v-m:/home/anton# fdisk /dev/sdb
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
First sector (2048-5242879, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-5242879, default 5242879): +2G

Created a new partition 1 of type 'Linux' and of size 2 GiB.

Select (default p): p
Partition number (2-4, default 2): 
First sector (4196352-5242879, default 4196352): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5242879, default 5242879): 

Created a new partition 2 of type 'Linux' and of size 511 MiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.


```
5. Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.  

используем команду `sfdisk -d /dev/sdb | sfdisk /dev/sdc`
```
root@anton-v-m:/home/anton# sfdisk -d /dev/sdb | sfdisk /dev/sdc
Checking that no-one is using this disk right now ... OK

Disk /dev/sdc: 2,51 GiB, 2684354560 bytes, 5242880 sectors
Disk model: VMware Virtual S
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x6625ed92

Old situation:

Device     Boot Start     End Sectors Size Id Type
/dev/sdc1        2048 4196351 4194304   2G 83 Linux

>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Script header accepted.
>>> Created a new DOS disklabel with disk identifier 0x6625ed92.
/dev/sdc1: Created a new partition 1 of type 'Linux' and of size 2 GiB.
/dev/sdc2: Created a new partition 2 of type 'Linux' and of size 511 MiB.
/dev/sdc3: Done.

New situation:
Disklabel type: dos
Disk identifier: 0x6625ed92

Device     Boot   Start     End Sectors  Size Id Type
/dev/sdc1          2048 4196351 4194304    2G 83 Linux
/dev/sdc2       4196352 5242879 1046528  511M 83 Linux

The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.


```

таблица скопировалась
```
root@anton-v-m:/home/anton# lsblk | grep sd
sda      8:0    0    20G  0 disk 
├─sda1   8:1    0   512M  0 part /boot/efi
├─sda2   8:2    0     1K  0 part 
└─sda5   8:5    0  19,5G  0 part /
sdb      8:16   0   2,5G  0 disk 
├─sdb1   8:17   0     2G  0 part 
└─sdb2   8:18   0   511M  0 part 
sdc      8:32   0   2,5G  0 disk 
├─sdc1   8:33   0     2G  0 part 
└─sdc2   8:34   0   511M  0 part 
```

6. Соберите mdadm RAID1 на паре разделов 2 Гб.  
```
root@anton-v-m:/home/anton# mdadm --create --verbose /dev/md0 -l 1 -n 2 /dev/sdb1 /dev/sdc1 
mdadm: Note: this array has metadata at the start and
    may not be suitable as a boot device.  If you plan to
    store '/boot' on this device please ensure that
    your boot-loader understands md/v1.x metadata, or use
    --metadata=0.90
mdadm: size set to 2094080K
Continue creating array? y
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.

```
7. Соберите mdadm RAID0 на второй паре маленьких разделов.  
```
root@anton-v-m:/home/anton# mdadm --create --verbose /dev/md1 -l 0 -n 2 /dev/sdb2 /dev/sdc2 
mdadm: chunk size defaults to 512K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md1 started.
```

информация о рейдах:

```
root@anton-v-m:/home/anton# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Wed Jan 26 22:59:57 2022
        Raid Level : raid1
        Array Size : 2094080 (2045.00 MiB 2144.34 MB)
     Used Dev Size : 2094080 (2045.00 MiB 2144.34 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Wed Jan 26 23:00:08 2022
             State : clean 
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

Consistency Policy : resync

              Name : anton-v-m:0  (local to host anton-v-m)
              UUID : f9286472:1322f110:596871e5:fbfb7286
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       17        0      active sync   /dev/sdb1
       1       8       33        1      active sync   /dev/sdc1
root@anton-v-m:/home/anton# mdadm -D /dev/md1
/dev/md1:
           Version : 1.2
     Creation Time : Wed Jan 26 23:00:49 2022
        Raid Level : raid0
        Array Size : 1042432 (1018.00 MiB 1067.45 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Wed Jan 26 23:00:49 2022
             State : clean 
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

            Layout : -unknown-
        Chunk Size : 512K

Consistency Policy : none

              Name : anton-v-m:1  (local to host anton-v-m)
              UUID : b44316c4:0abe4d07:9f78edb9:0d949603
            Events : 0

    Number   Major   Minor   RaidDevice State
       0       8       18        0      active sync   /dev/sdb2
       1       8       34        1      active sync   /dev/sdc2

```

8. Создайте 2 независимых PV на получившихся md-устройствах.  
```
root@anton-v-m:/home/anton# pvcreate /dev/md0 
  Physical volume "/dev/md0" successfully created.
root@anton-v-m:/home/anton# pvcreate /dev/md1
  Physical volume "/dev/md1" successfully created.
root@anton-v-m:/home/anton# pvdisplay 
  "/dev/md0" is a new physical volume of "<2,00 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/md0
  VG Name               
  PV Size               <2,00 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               RyGmhW-88xv-G2Kx-SlTr-d3cd-ujuc-Pox80g
   
  "/dev/md1" is a new physical volume of "1018,00 MiB"
  --- NEW Physical volume ---
  PV Name               /dev/md1
  VG Name               
  PV Size               1018,00 MiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               bKJ6Wf-idio-uOoJ-f6sQ-Feuv-5ujG-dgcvW0
  
```

9. Создайте общую volume-group на этих двух PV.

```
root@anton-v-m:/home/anton# vgcreate vol_grp01 /dev/md0 /dev/md1
  Volume group "vol_grp01" successfully created
root@anton-v-m:/home/anton# vgdisplay 
  --- Volume group ---
  VG Name               vol_grp01
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  1
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                0
  Open LV               0
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               <2,99 GiB
  PE Size               4,00 MiB
  Total PE              765
  Alloc PE / Size       0 / 0   
  Free  PE / Size       765 / <2,99 GiB
  VG UUID               uCEYKR-U226-jSB0-Dkf1-NjFl-xH59-FoTU0S

```
10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.  
```
root@anton-v-m:/home/anton# lvcreate -L 100M -n logical_vol1 vol_grp01 /dev/md1
  Logical volume "logical_vol1" created.
```
```
root@anton-v-m:/home/anton# lsblk
NAME                         MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
sda                            8:0    0    20G  0 disk  
├─sda1                         8:1    0   512M  0 part  /boot/efi
├─sda2                         8:2    0     1K  0 part  
└─sda5                         8:5    0  19,5G  0 part  /
sdb                            8:16   0   2,5G  0 disk  
├─sdb1                         8:17   0     2G  0 part  
│ └─md0                        9:0    0     2G  0 raid1 
└─sdb2                         8:18   0   511M  0 part  
  └─md1                        9:1    0  1018M  0 raid0 
    └─vol_grp01-logical_vol1 253:0    0   100M  0 lvm   
sdc                            8:32   0   2,5G  0 disk  
├─sdc1                         8:33   0     2G  0 part  
│ └─md0                        9:0    0     2G  0 raid1 
└─sdc2                         8:34   0   511M  0 part  
  └─md1                        9:1    0  1018M  0 raid0 
    └─vol_grp01-logical_vol1 253:0    0   100M  0 lvm   
sr0                           11:0    1   2,9G  0 rom   

```

11. Создайте mkfs.ext4 ФС на получившемся LV.

```shell
root@anton-v-m:/home/anton# mkfs.ext4 /dev/vol_grp01/logical_vol1
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 25600 4k blocks and 25600 inodes

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (1024 blocks): done
Writing superblocks and filesystem accounting information: done

```

12. Смонтируйте этот раздел в любую директорию, например, /tmp/new.  
```
root@anton-v-m:/home/anton# mount /dev/vol_grp01/logical_vol1 /tmp/new/
```
```
sdb                            8:16   0   2,5G  0 disk  
├─sdb1                         8:17   0     2G  0 part  
│ └─md0                        9:0    0     2G  0 raid1 
└─sdb2                         8:18   0   511M  0 part  
  └─md1                        9:1    0  1018M  0 raid0 
    └─vol_grp01-logical_vol1 253:0    0   100M  0 lvm   /tmp/new
sdc                            8:32   0   2,5G  0 disk  
├─sdc1                         8:33   0     2G  0 part  
│ └─md0                        9:0    0     2G  0 raid1 
└─sdc2                         8:34   0   511M  0 part  
  └─md1                        9:1    0  1018M  0 raid0 
    └─vol_grp01-logical_vol1 253:0    0   100M  0 lvm   /tmp/new
sr0                           11:0    1   2,9G  0 rom   

```

13. Поместите туда тестовый файл, например wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz.  
сделано:
```
root@anton-v-m:/home/anton# cd /tmp/new/
root@anton-v-m:/tmp/new# wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz
--2022-01-27 21:18:34--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
Resolving mirror.yandex.ru (mirror.yandex.ru)... 213.180.204.183, 2a02:6b8::183
Connecting to mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 22106598 (21M) [application/octet-stream]
Saving to: ‘/tmp/new/test.gz’

/tmp/new/test.gz                      100%[========================================================================>]  21,08M  7,48MB/s    in 2,8s    

2022-01-27 21:18:37 (7,48 MB/s) - ‘/tmp/new/test.gz’ saved [22106598/22106598]

```
14. Прикрепите вывод lsblk.  
```
root@anton-v-m:/tmp/new# lsblk
NAME                         MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINT
sda                            8:0    0    20G  0 disk  
├─sda1                         8:1    0   512M  0 part  /boot/efi
├─sda2                         8:2    0     1K  0 part  
└─sda5                         8:5    0  19,5G  0 part  /
sdb                            8:16   0   2,5G  0 disk  
├─sdb1                         8:17   0     2G  0 part  
│ └─md0                        9:0    0     2G  0 raid1 
└─sdb2                         8:18   0   511M  0 part  
  └─md1                        9:1    0  1018M  0 raid0 
    └─vol_grp01-logical_vol1 253:0    0   100M  0 lvm   /tmp/new
sdc                            8:32   0   2,5G  0 disk  
├─sdc1                         8:33   0     2G  0 part  
│ └─md0                        9:0    0     2G  0 raid1 
└─sdc2                         8:34   0   511M  0 part  
  └─md1                        9:1    0  1018M  0 raid0 
    └─vol_grp01-logical_vol1 253:0    0   100M  0 lvm   /tmp/new
sr0                           11:0    1   2,9G  0 rom   

```

15. Протестируйте целостность файла:  
выполнено:
```
root@anton-v-m:/tmp/new# gzip -t /tmp/new/test.gz
root@anton-v-m:/tmp/new# echo $?
0
```

16. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.
команда pvdisplay отображает названия pv
```
root@anton-v-m:/tmp/new# pvdisplay
  --- Physical volume ---
  PV Name               /dev/md0
  VG Name               vol_grp01
  ...
   
  --- Physical volume ---
  PV Name               /dev/md1
  VG Name               vol_grp01
...

```
теперь выполним команду pvmove
```
root@anton-v-m:/tmp/new# pvmove /dev/md1 /dev/md0
  /dev/md1: Moved: 24,00%
  /dev/md1: Moved: 100,00%

```
17. Сделайте --fail на устройство в вашем RAID1 md.  
```
root@anton-v-m:/tmp/new# mdadm /dev/md0 --fail /dev/sdc1
mdadm: set /dev/sdc1 faulty in /dev/md0

```
18. Подтвердите выводом dmesg, что RAID1 работает в деградированном состоянии.  
```
root@anton-v-m:/tmp/new# dmesg

...
[ 1251.360594] md/raid1:md0: Disk failure on sdc1, disabling device.
               md/raid1:md0: Operation continuing on 1 devices.

```

19. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:  
выполнено:
```
root@anton-v-m:/tmp/new# gzip -t /tmp/new/test.gz
root@anton-v-m:/tmp/new# echo $?
0

```

20. Погасите тестовый хост, vagrant destroy.  
готово
```
PS F:\vagrant_project\vm-3> vagrant.exe destroy
    default: Are you sure you want to destroy the 'default' VM? [y/N] y
==> default: Destroying VM and associated drives...
```