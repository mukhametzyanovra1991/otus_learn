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

## Уменьшить том под / до 8G  
### install package xfsdump  
[root@lvm ~]# yum install -y xfsdump  
Installed:  
  xfsdump.x86_64 0:3.1.7-4.el7_9  
Dependency Installed:  
  attr.x86_64 0:2.4.46-13.el7  
Complete!  

### prepare temp volume for / partition  
[root@lvm ~]# vgcreate vg_root /dev/sdb  
  Physical volume "/dev/sdb" successfully created.  
  Volume group "vg_root" successfully created  
[root@lvm ~]# lvcreate -n lv_root -l +100%FREE /dev/vg_root  
  Logical volume "lv_root" created.  

### make filesystem and mount it  
[root@lvm ~]# mkfs.xfs /dev/vg_root/lv_root  
meta-data=/dev/vg_root/lv_root   isize=512    agcount=4, agsize=655104 blks  
         =                       sectsz=512   attr=2, projid32bit=1  
         =                       crc=1        finobt=0, sparse=0  
data     =                       bsize=4096   blocks=2620416, imaxpct=25  
         =                       sunit=0      swidth=0 blks  
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1  
log      =internal log           bsize=4096   blocks=2560, version=2  
         =                       sectsz=512   sunit=0 blks, lazy-count=1  
realtime =none                   extsz=4096   blocks=0, rtextents=0  
[root@lvm ~]# mkdir /tmp_root  
[root@lvm ~]# mount /dev/vg_root/lv_root /tmp_root/  

### copy all from / to /tmp_root  
[root@lvm ~]# xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /tmp_root/  
xfsdump: Dump Status: SUCCESS  
xfsrestore: restore complete: 93 seconds elapsed  
xfsrestore: Restore Status: SUCCESS  

### configure grub, imitate current root, do chrot and update grub  
[root@lvm ~]# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /tmp_root/$i; done  
[root@lvm ~]# chroot /tmp_root/  
[root@lvm /]# grub2-mkconfig -o /boot/grub2/grub.cfg  
Generating grub configuration file ...  
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64  
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img  
done  

### update initrd image  
[root@lvm /]# cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; \> s/.img//g"` --force; done  
*** Creating image file ***  
*** Creating image file done ***  
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***  

### change rd.lvm.lv=VolGroup00/LogVol00 to rd.lvm.lv=vg_root/lv_root for mounting right root while loading  
[root@lvm boot]# sed -i "s/rd.lvm.lv=VolGroup00\/LogVol00/rd.lvm.lv=vg_root\/lv_root/g" grub2/grub.cfg  
[root@lvm boot]# cat grub2/grub.cfg | grep 'rd.lvm.lv'  
        linux16 /vmlinuz-3.10.0-862.2.3.el7.x86_64 root=/dev/mapper/vg_root-lv_root ro no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=vg_root/lv_root rd.lvm.lv=VolGroup00/LogVol01 rhgb quiet  
[root@lvm boot]# lsblk  
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT  
sda                       8:0    0   40G  0 disk  
├─sda1                    8:1    0    1M  0 part  
├─sda2                    8:2    0    1G  0 part /boot  
└─sda3                    8:3    0   39G  0 part  
  ├─VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  
  └─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]  
sdb                       8:16   0   10G  0 disk  
└─vg_root-lv_root       253:2    0   10G  0 lvm  /  
sdc                       8:32   0    2G  0 disk  
sdd                       8:48   0    1G  0 disk  
sde                       8:64   0    1G  0 disk  

### change size of old VG and return root on it  
### remove old 40G LV and create new 8G  
[root@lvm ~]# lvremove /dev/VolGroup00/LogVol00  
Do you really want to remove active logical volume VolGroup00/LogVol00? [y/n]: y  
  Logical volume "LogVol00" successfully removed  
[root@lvm ~]# lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00  
WARNING: xfs signature detected on /dev/VolGroup00/LogVol00 at offset 0. Wipe it? [y/n]: y  
  Wiping xfs signature on /dev/VolGroup00/LogVol00.  
  Logical volume "LogVol00" created.  

### create filesystem, mount and copy with xfsdump  
[root@lvm ~]# mkfs.xfs /dev/VolGroup00/LogVol00  
meta-data=/dev/VolGroup00/LogVol00 isize=512    agcount=4, agsize=524288 blks  
         =                       sectsz=512   attr=2, projid32bit=1  
         =                       crc=1        finobt=0, sparse=0  
data     =                       bsize=4096   blocks=2097152, imaxpct=25  
         =                       sunit=0      swidth=0 blks  
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1  
log      =internal log           bsize=4096   blocks=2560, version=2  
         =                       sectsz=512   sunit=0 blks, lazy-count=1  
realtime =none                   extsz=4096   blocks=0, rtextents=0  
[root@lvm ~]# mount /dev/VolGroup00/LogVol00 /tmp_root/  
[root@lvm ~]# xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /tmp_root/  
xfsdump: Dump Status: SUCCESS  
xfsrestore: restore complete: 98 seconds elapsed  
xfsrestore: Restore Status: SUCCESS  

### configure grub  
[root@lvm ~]# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /tmp_root/$i; done  
[root@lvm ~]# chroot /tmp_root/  
[root@lvm /]# grub2-mkconfig -o /boot/grub2/grub.cfg  
Generating grub configuration file ...  
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64  
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img  
done  
[root@lvm /]# cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; \> s/.img//g"` --force; done  
*** Creating image file ***  
*** Creating image file done ***  
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***  

## Выделить том под /var в зеркало  
### create mirror on free disks  
[root@lvm boot]# vgcreate vg_var /dev/sdc /dev/sdd  
  Physical volume "/dev/sdc" successfully created.  
  Physical volume "/dev/sdd" successfully created.  
  Volume group "vg_var" successfully created  
[root@lvm boot]# lvcreate -L 950M -m1 -n lv_var vg_var  
  Rounding up size to full physical extent 952.00 MiB  
  Logical volume "lv_var" created.  

### create FS and move /var  
[root@lvm boot]# mkfs.ext4 /dev/vg_var/lv_var  
mke2fs 1.42.9 (28-Dec-2013)  
...  
Allocating group tables: done  
Writing inode tables: done  
Creating journal (4096 blocks): done  
Writing superblocks and filesystem accounting information: done
[root@lvm boot]# mkdir /tmp_var  
[root@lvm boot]# mount /dev/vg_var/lv_var /tmp_var  
[root@lvm boot]# cp -aR /var/* /tmp_var/  

### save old /var content  
[root@lvm boot]# mkdir /tmp/oldvar && mv /var/* /tmp/oldvar

### mount new var to /var  
[root@lvm boot]# umount /tmp_var  
[root@lvm boot]# mount /dev/vg_var/lv_var /var  

### change fstab for automatic mount /var  
[root@lvm boot]# echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab  
[root@lvm boot]# cat /etc/fstab | grep var  
UUID="4bcf4ef5-38c5-4022-881c-55fccd8bea95" /var ext4 defaults 0 0  

### reboot and remove temporary LV for root dir  
[root@lvm ~]# lvremove /dev/vg_root/lv_root  
Do you really want to remove active logical volume vg_root/lv_root? [y/n]: y  
  Logical volume "lv_root" successfully removed  
[root@lvm ~]# vgremove /dev/vg_root  
  Volume group "vg_root" successfully removed  
[root@lvm ~]# pvremove /dev/sdb  
  Labels on physical volume "/dev/sdb" successfully wiped.  

## Выделить том под /home  
### create LV in VG VolGroup00  
[root@lvm ~]# lvcreate -n LogVol_Home -L 2G /dev/VolGroup00  
  Logical volume "LogVol_Home" created.  

### create FS, mount it and copy files from home  
[root@lvm ~]# mkdir /tmp_home  
[root@lvm ~]# mkfs.xfs /dev/VolGroup00/LogVol_Home  
meta-data=/dev/VolGroup00/LogVol_Home isize=512    agcount=4, agsize=131072 blks  
         =                       sectsz=512   attr=2, projid32bit=1  
         =                       crc=1        finobt=0, sparse=0  
data     =                       bsize=4096   blocks=524288, imaxpct=25  
         =                       sunit=0      swidth=0 blks  
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1  
log      =internal log           bsize=4096   blocks=2560, version=2  
         =                       sectsz=512   sunit=0 blks, lazy-count=1  
realtime =none                   extsz=4096   blocks=0, rtextents=0  
[root@lvm ~]# mount /dev/VolGroup00/LogVol_Home /tmp_home/  
[root@lvm ~]# cp -aR /home/* /tmp_home/  
[root@lvm ~]# rm -rf /home/*  
[root@lvm ~]# umount /tmp_home/  
[root@lvm ~]# mount /dev/VolGroup00/LogVol_Home /home/  

### change fstab for automatic mount /home  
[root@lvm ~]# echo "`blkid | grep Home | awk '{print $2}'` /home xfs defaults 0 0" >> /etc/fstab  
[root@lvm ~]# cat /etc/fstab | grep home  
UUID="7c8586bc-c531-481c-a841-1998c61626c8" /home xfs defaults 0 0  

## Работа со снапшотами  
### generate files in /home  
[root@lvm ~]# touch /home/file{1..20}  

### create snapshot for /LogVol_Home  
[root@lvm ~]# lvcreate -L 100MB -s -n home_snap /dev/VolGroup00/LogVol_Home  
  Rounding up size to full physical extent 128.00 MiB  
  Logical volume "home_snap" created.  

### remove some files  
[root@lvm ~]# rm -f /home/file{11..20}  
[root@lvm ~]# ls /home/  
file1  file10  file2  file3  file4  file5  file6  file7  file8  file9  vagrant  

### recovery from snapshot  
[root@lvm ~]# umount /home  
[root@lvm ~]# lvconvert --merge /dev/VolGroup00/home_snap  
  Merging of volume VolGroup00/home_snap started.  
  VolGroup00/LogVol_Home: Merged: 100.00%  
[root@lvm ~]# mount /home  
[root@lvm ~]# ls -al /home  
total 0  
drwxr-xr-x.  3 root    root    292 Jun 11 05:51 .  
drwxr-xr-x. 21 root    root    286 Jun 11 05:44 ..  
-rw-r--r--.  1 root    root      0 Jun 11 05:51 file1  
-rw-r--r--.  1 root    root      0 Jun 11 05:51 file10  
-rw-r--r--.  1 root    root      0 Jun 11 05:51 file11  
-rw-r--r--.  1 root    root      0 Jun 11 05:51 file12  
