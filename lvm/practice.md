# “Файловые системы и LVM-1” курса Administrator Linux.Professional

## Что нужно сделать?
на имеющемся образе (centos/7 1804.2)  
1 уменьшить том под / до 8G  
2 выделить том под /home  
3 выделить том под /var (/var - сделать в mirror)  
4 для /home - сделать том для снэпшотов  
5 прописать монтирование в fstab (попробовать с разными опциями и разными файловыми системами на выбор)  
6 Работа со снапшотами:  
	- сгенерировать файлы в /home/  
	- снять снэпшот  
	- удалить часть файлов  
	- восстановиться со снэпшота  

(залоггировать работу можно утилитой script, скриншотами и т.п.)

Задание со звездочкой*  
на нашей куче дисков попробовать поставить btrfs/zfs:  
	- с кешем и снэпшотами  
	- разметить здесь каталог /opt  

## LVM - начало работы  
### get list of block devices  
[root@lvm ~]# lsblk  
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT  
sda      8:0    0  40G  0 disk  
└─sda1   8:1    0  40G  0 part /  
sdb      8:16   0  10G  0 disk  
sdc      8:32   0   2G  0 disk  
sdd      8:48   0   1G  0 disk  
sde      8:64   0   1G  0 disk  
[root@lvm ~]# lvmdiskscan  
  /dev/sda1 [     <40.00 GiB]  
  /dev/sdb  [      10.00 GiB]  
  /dev/sdc  [       2.00 GiB]  
  /dev/sdd  [       1.00 GiB]  
  /dev/sde  [       1.00 GiB]  
  4 disks  
  1 partition  
  0 LVM physical volume whole disks  
  0 LVM physical volumes  

### creane Physical Volume and Volume Group  
[root@lvm ~]# vgcreate otus_lvm /dev/sdb  
  Physical volume "/dev/sdb" successfully created.  
  Volume group "otus_lvm" successfully created  

### create Logical Volume  
[root@lvm ~]# lvcreate -l+80%FREE -n lv1 otus_lvm  
WARNING: ext4 signature detected on /dev/otus_lvm/lv1 at offset 1080. Wipe it? [y/n]: y  
  Wiping ext4 signature on /dev/otus_lvm/lv1.  
  Logical volume "lv1" created.  

### get info about VG
[root@lvm ~]# vgdisplay otus_lvm  
  --- Volume group ---   
  VG Name               otus_lvm  
  System ID  
  Format                lvm2  
  Metadata Areas        1  
  Metadata Sequence No  2  
  VG Access             read/write  
  VG Status             resizable  
  MAX LV                0  
  Cur LV                1  
  Open LV               0  
  Max PV                0  
  Cur PV                1  
  Act PV                1  
  VG Size               <10.00 GiB  
  PE Size               4.00 MiB  
  Total PE              2559  
  Alloc PE / Size       2047 / <8.00 GiB  
  Free  PE / Size       512 / 2.00 GiB  
  VG UUID               EJbLBx-cDJf-BsF3-hp84-nEoX-58P0-OceHIm  

### get info about block devices in VG  
[root@lvm ~]# vgdisplay -v otus_lvm | grep 'PV Name'  
  PV Name               /dev/sdb  

### get info about LV  
[root@lvm ~]# lvdisplay /dev/otus_lvm/lv1  
  --- Logical volume ---  
  LV Path                /dev/otus_lvm/lv1  
  LV Name                lv1  
  VG Name                otus_lvm  
  LV UUID                38j3bf-I1pN-sqsm-e1G8-V3eC-BdtP-EZcLJ8  
  LV Write Access        read/write  
  LV Creation host, time lvm, 2024-06-06 11:18:45 +0000  
  LV Status              available  
  # open                 0  
  LV Size                <8.00 GiB  
  Current LE             2047  
  Segments               1  
  Allocation             inherit  
  Read ahead sectors     auto  
  - currently set to     8192  
  Block device           253:0  
[root@lvm ~]# pvs  
  PV         VG       Fmt  Attr PSize   PFree  
  /dev/sdb   otus_lvm lvm2 a--  <10.00g 2.00g  
[root@lvm ~]# vgs  
  VG       #PV #LV #SN Attr   VSize   VFree  
  otus_lvm   1   1   0 wz--n- <10.00g 2.00g  
[root@lvm ~]# lvs  
  LV   VG       Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert  
  lv1  otus_lvm -wi-a----- <8.00g  

### create another LV size 100Mb  
[root@lvm ~]# lvcreate -L100M -n lv2small otus_lvm  
  Logical volume "lv2small" created.  
[root@lvm ~]# lvs  
  LV       VG       Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert  
  lv1      otus_lvm -wi-a-----  <8.00g  
  lv2small otus_lvm -wi-a----- 100.00m  

### make filesystem on LV and mount it  
[root@lvm ~]# mkfs.ext4 /dev/otus_lvm/lv1  
mke2fs 1.42.9 (28-Dec-2013)  
Filesystem label=  
OS type: Linux  
Block size=4096 (log=2)  
Fragment size=4096 (log=2)  
Stride=0 blocks, Stripe width=0 blocks  
524288 inodes, 2096128 blocks  
104806 blocks (5.00%) reserved for the super user  
First data block=0  
Maximum filesystem blocks=2147483648  
64 block groups  
32768 blocks per group, 32768 fragments per group  
8192 inodes per group  
Superblock backups stored on blocks:  
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632
Allocating group tables: done  
Writing inode tables: done  
Creating journal (32768 blocks): done  
Writing superblocks and filesystem accounting information: done  
[root@lvm ~]# mkdir /mnt/lv1  
[root@lvm ~]# mount /dev/o  
oldmem    otus_lvm/  
[root@lvm ~]# mount /dev/otus_lvm/lv1 /mnt/lv1/  
[root@lvm ~]# mount | grep /mnt/lv1  
/dev/mapper/otus_lvm-lv1 on /mnt/lv1 type ext4 (rw,relatime,seclabel,data=ordered)  

## Расширение LVM  
### create new PV and extend VG  
[root@lvm ~]# vgextend otus_lvm /dev/sdc  
  Physical volume "/dev/sdc" successfully created.  
  Volume group "otus_lvm" successfully extended  
[root@lvm ~]# vgdisplay -v otus_lvm | grep 'PV Name'  
  PV Name               /dev/sdb  
  PV Name               /dev/sdc  
[root@lvm ~]# vgs  
  VG       #PV #LV #SN Attr   VSize  VFree  
  otus_lvm   2   2   0 wz--n- 11.99g <3.90g  

### simulate used disk space with dd  
[root@lvm ~]# dd if=/dev/zero of=/mnt/lv1/test.log bs=1M count=8000 status=progress  
8228175872 bytes (8.2 GB) copied, 136.984393 s, 60.1 MB/s  
dd: error writing ‘/mnt/lv1/test.log’: No space left on device  
7880+0 records in  
7879+0 records out  
8262189056 bytes (8.3 GB) copied, 137.867 s, 59.9 MB/s  
[root@lvm ~]# df -Th /mnt/lv1/  
Filesystem               Type  Size  Used Avail Use% Mounted on  
/dev/mapper/otus_lvm-lv1 ext4  7.8G  7.8G     0 100% /mnt/lv1  

### extend LV  
[root@lvm ~]# lvextend -l+80%FREE /dev/otus_lvm/lv1  
  Size of logical volume otus_lvm/lv1 changed from <8.00 GiB (2047 extents) to <11.12 GiB (2846 extents).  
  Logical volume otus_lvm/lv1 successfully resized.  
[root@lvm ~]# lvs /dev/otus_lvm/lv1  
  LV   VG       Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert  
  lv1  otus_lvm -wi-ao---- <11.12g  
[root@lvm ~]# df -Th /mnt/lv1/  
Filesystem               Type  Size  Used Avail Use% Mounted on  
/dev/mapper/otus_lvm-lv1 ext4  7.8G  7.8G     0 100% /mnt/lv1  

### resize filesystem  
[root@lvm ~]# resize2fs /dev/otus_lvm/lv1  
resize2fs 1.42.9 (28-Dec-2013)  
Filesystem at /dev/otus_lvm/lv1 is mounted on /mnt/lv1; on-line resizing required  
old_desc_blocks = 1, new_desc_blocks = 2  
The filesystem on /dev/otus_lvm/lv1 is now 2914304 blocks long.  
[root@lvm ~]# df -Th /mnt/lv1/  
Filesystem               Type  Size  Used Avail Use% Mounted on  
/dev/mapper/otus_lvm-lv1 ext4   11G  7.8G  2.6G  76% /mnt/lv1  

## decrease LV   
### unmount and check FS  
[root@lvm ~]# umount /mnt/lv1  
[root@lvm ~]# e2fsck -fy /dev/otus_lvm/lv1   
e2fsck 1.42.9 (28-Dec-2013)  
Pass 1: Checking inodes, blocks, and sizes  
Pass 2: Checking directory structure  
Pass 3: Checking directory connectivity  
Pass 4: Checking reference counts  
Pass 5: Checking group summary information  
/dev/otus_lvm/lv1: 12/729088 files (0.0% non-contiguous), 2105907/2914304 blocks  

### resize filesystem to 10 Gb  
[root@lvm ~]# resize2fs /dev/otus_lvm/lv1 10G  
resize2fs 1.42.9 (28-Dec-2013)  
Resizing the filesystem on /dev/otus_lvm/lv1 to 2621440 (4k) blocks.  
The filesystem on /dev/otus_lvm/lv1 is now 2621440 blocks long.  

### reduce LV  
[root@lvm ~]# lvreduce /dev/otus_lvm/lv1 -L 10G  
  WARNING: Reducing active logical volume to 10.00 GiB.  
  THIS MAY DESTROY YOUR DATA (filesystem etc.)  
Do you really want to reduce otus_lvm/lv1? [y/n]: y  
  Size of logical volume otus_lvm/lv1 changed from <11.12 GiB (2846 extents) to 10.00 GiB (2560 extents).  
  Logical volume otus_lvm/lv1 successfully resized.  

### mount FS and get FS and LVM info  
[root@lvm ~]# df -Th /mnt/lv1/  
Filesystem               Type  Size  Used Avail Use% Mounted on  
/dev/mapper/otus_lvm-lv1 ext4  9.8G  7.8G  1.6G  84% /mnt/lv1  
[root@lvm ~]# lvs /dev/otus_lvm/lv1  
  LV   VG       Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert  
  lv1  otus_lvm -wi-ao---- 10.00g  

## Работа со снапшотами  
### create snapshot  
[root@lvm ~]# lvcreate -L 500M -s -n lv3snap /dev/otus_lvm/lv1  
  Logical volume "lv3snap" created.  
[root@lvm ~]# vgs -o +lv_size,lv_name | grep lv  
  otus_lvm   2   3   1 wz--n- 11.99g <1.41g  10.00g lv1  
  otus_lvm   2   3   1 wz--n- 11.99g <1.41g 100.00m lv2small  
  otus_lvm   2   3   1 wz--n- 11.99g <1.41g 500.00m lv3snap  
[root@lvm ~]# lsblk  
NAME                   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT  
sda                      8:0    0   40G  0 disk  
└─sda1                   8:1    0   40G  0 part /  
sdb                      8:16   0   10G  0 disk  
├─otus_lvm-lv2small    253:1    0  100M  0 lvm   
└─otus_lvm-lv1-real    253:2    0   10G  0 lvm  
  ├─otus_lvm-lv1       253:0    0   10G  0 lvm  /mnt/lv1  
  └─otus_lvm-lv3snap   253:4    0   10G  0 lvm  
sdc                      8:32   0    2G  0 disk  
├─otus_lvm-lv1-real    253:2    0   10G  0 lvm  
│ ├─otus_lvm-lv1       253:0    0   10G  0 lvm  /mnt/lv1  
│ └─otus_lvm-lv3snap   253:4    0   10G  0 lvm  
└─otus_lvm-lv3snap-cow 253:3    0  500M  0 lvm  
  └─otus_lvm-lv3snap   253:4    0   10G  0 lvm  
sdd                      8:48   0    1G  0 disk  
sde                      8:64   0    1G  0 disk  

### mount snapshot  
[root@lvm ~]# mkdir /mnt/lv3snap   
[root@lvm ~]# mount /dev/otus_lvm/lv3snap /mnt/lv3snap/  
[root@lvm ~]# ll /mnt/lv3snap/  
total 8068564  
drwx------. 2 root root      16384 Jun  6 11:37 lost+found  
-rw-r--r--. 1 root root 8262189056 Jun  6 11:54 test.log  

### rollback to snapshot  
[root@lvm ~]# umount /mnt/lv3snap  

### remove file test.log  
[root@lvm ~]# rm /mnt/lv1/test.log  
rm: remove regular file ‘/mnt/lv1/test.log’? y  
[root@lvm ~]# ll /mnt/lv1/  
total 16  
drwx------. 2 root root 16384 Jun  6 11:37 lost+found  

### unmount LV
[root@lvm ~]# umount /mnt/lv1  

### rollback to snapshot  
[root@lvm ~]# lvconvert --merge /dev/otus_lvm/lv3snap  
  Merging of volume otus_lvm/lv3snap started.  
  otus_lvm/lv1: Merged: 99.93%  
  otus_lvm/lv1: Merged: 100.00%  

### mount LV  
[root@lvm ~]# mount /dev/otus_lvm/lv1 /mnt/lv1  
[root@lvm ~]# ll /mnt/lv1/  
total 8068564  
drwx------. 2 root root      16384 Jun  6 11:37 lost+found  
-rw-r--r--. 1 root root 8262189056 Jun  6 11:54 test.log  

## Работа с LVM-RAID  
### create PV and VG vg0  
[root@lvm ~]# vgcreate vg0 /dev/sd{d,e}  
  Physical volume "/dev/sdd" successfully created.  
  Physical volume "/dev/sde" successfully created.  
  Volume group "vg0" successfully created  

### create LV  
[root@lvm ~]# lvcreate -l+80%FREE -m1 -n mirror vg0  
  Logical volume "mirror" created.  
[root@lvm ~]# lvs  
  LV       VG       Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert  
  lv1      otus_lvm -wi-ao----  10.00g  
  lv2small otus_lvm -wi-a----- 100.00m  
  mirror   vg0      rwi-a-r--- 816.00m                                    86.36  
[root@lvm ~]# lvs  
  LV       VG       Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert  
  lv1      otus_lvm -wi-ao----  10.00g  
  lv2small otus_lvm -wi-a----- 100.00m  
  mirror   vg0      rwi-a-r--- 816.00m                                    100.00  
