# Домашнее задание: работа с mdadm

## Задание:

• добавить в Vagrantfile еще дисков

• собрать R0/R5/R10 на выбор

• прописать собранный рейд в конф, чтобы рейд собирался при загрузке

• сломать/починить raid 

• создать GPT раздел и 5 партиций и смонтировать их на диск.

В качестве проверки принимается - измененный Vagrantfile, скрипт для создания рейда, конф для автосборки рейда при загрузке.

* Доп. задание - Vagrantfile, который сразу собирает систему с подключенным рейдом

## create RAID
### vagrant up vm centos/7 with 5 hdds
  [vagrant@otuslinux ~]$ sudo lshw -short | grep disk  
  /0/100/1.1/0.0.0    /dev/sda   disk        42GB VBOX HARDDISK  
  /0/100/d/0          /dev/sdb   disk        262MB VBOX HARDDISK  
  /0/100/d/1          /dev/sdc   disk        262MB VBOX HARDDISK  
  /0/100/d/2          /dev/sdd   disk        262MB VBOX HARDDISK  
  /0/100/d/3          /dev/sde   disk        262MB VBOX HARDDISK  
  /0/100/d/0.0.0      /dev/sdf   disk        262MB VBOX HARDDISK  

### nullify the superblock
  mdadm --zero-superblock --force /dev/sd{b,c,d,e,f}

### create RAID6
  [vagrant@otuslinux ~]$ sudo mdadm --create --verbose /dev/md0 -l 6 -n 5 /dev/sd{b,c,d,e,f}  
  mdadm: layout defaults to left-symmetric  
  mdadm: layout defaults to left-symmetric  
  mdadm: chunk size defaults to 512K  
  mdadm: size set to 253952K  
  mdadm: Defaulting to version 1.2 metadata  
  mdadm: array /dev/md0 started.  

### check that RAID creates normally
  [vagrant@otuslinux ~]$ cat /proc/mdstat  
  Personalities : [raid6] [raid5] [raid4]  
  md0 : active raid6 sdf[4] sde[3] sdd[2] sdc[1] sdb[0]  
      761856 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/5] [UUUUU]  
  unused devices: <none>  

  [vagrant@otuslinux ~]$ sudo mdadm -D /dev/md0  
  /dev/md0:  
           Version : 1.2  
     Creation Time : Tue May 28 05:29:04 2024  
        Raid Level : raid6  
        Array Size : 761856 (744.00 MiB 780.14 MB)  
     Used Dev Size : 253952 (248.00 MiB 260.05 MB)  
      Raid Devices : 5  
     Total Devices : 5  
       Persistence : Superblock is persistent  
       Update Time : Tue May 28 05:29:20 2024  
             State : clean  
    Active Devices : 5  
   Working Devices : 5  
    Failed Devices : 0  
     Spare Devices : 0  
            Layout : left-symmetric  
        Chunk Size : 512K  
  Consistency Policy : resync  
              Name : otuslinux:0  (local to host otuslinux)  
              UUID : 6f0cbe87:7c8e15c8:ea3bc5d3:a860f857  
            Events : 17  
    Number   Major   Minor   RaidDevice State  
       0       8       16        0      active sync   /dev/sdb  
       1       8       32        1      active sync   /dev/sdc  
       2       8       48        2      active sync   /dev/sdd  
       3       8       64        3      active sync   /dev/sde  
       4       8       80        4      active sync   /dev/sdf  

### create config file for mdadm
### scan /proc/mdstat to get a list of array devices from /proc/mdstat  
  [vagrant@otuslinux ~]$ sudo mdadm --detail --scan --verbose  
  ARRAY /dev/md0 level=raid6 num-devices=5 metadata=1.2 name=otuslinux:0 UUID=6f0cbe87:7c8e15c8:ea3bc5d3:a860f857  
     devices=/dev/sdb,/dev/sdc,/dev/sdd,/dev/sde,/dev/sdf  

### create file mdadm.conf
### create dir /etc/mdadm
  [vagrant@otuslinux ~]$ sudo mkdir /etc/mdadm  

### create file /etc/mdadm/mdadm.conf
  [vagrant@otuslinux ~]$ sudo touch /etc/mdadm/mdadm.conf  

### change owner and group owner of file to vagrant:vagrant
  [vagrant@otuslinux ~]$ sudo chown vagrant:vagrant /etc/mdadm/mdadm.conf  
  [vagrant@otuslinux ~]$ sudo ls -la /etc/mdadm/mdadm.conf  
  -rw-r--r--. 1 vagrant vagrant 0 May 28 06:13 /etc/mdadm/mdadm.conf  

### write information about array to mdadm.conf
  [vagrant@otuslinux ~]$ echo "DEVICE partitions" > /etc/mdadm/mdadm.conf  
  [vagrant@otuslinux ~]$ sudo mdadm --detail --scan --verbose | awk '/ARRAY/ {print}' >> /etc/mdadm/mdadm.conf  
  [vagrant@otuslinux ~]$ cat /etc/mdadm/mdadm.conf  
  DEVICE partitions  
  ARRAY /dev/md0 level=raid6 num-devices=5 metadata=1.2 name=otuslinux:0 UUID=6f0cbe87:7c8e15c8:ea3bc5d3:a860f857  

## fail RAID
### fail one of block device
  [vagrant@otuslinux ~]$ sudo mdadm /dev/md0 --fail /dev/sde  
  mdadm: set /dev/sde faulty in /dev/md0  

  [vagrant@otuslinux ~]$ cat /proc/mdstat  
  Personalities : [raid6] [raid5] [raid4]  
  md0 : active raid6 sdf[4] sde[3](F) sdd[2] sdc[1] sdb[0]  
        761856 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/4] [UUU_U]  
  unused devices: <none>  

  [vagrant@otuslinux ~]$ sudo mdadm -D /dev/md0  
  /dev/md0:  
             Version : 1.2  
       Creation Time : Tue May 28 05:29:04 2024  
          Raid Level : raid6  
          Array Size : 761856 (744.00 MiB 780.14 MB)  
       Used Dev Size : 253952 (248.00 MiB 260.05 MB)  
        Raid Devices : 5  
       Total Devices : 5  
         Persistence : Superblock is persistent  
         Update Time : Tue May 28 06:29:05 2024  
               State : clean, degraded  
      Active Devices : 4  
     Working Devices : 4  
      Failed Devices : 1  
       Spare Devices : 0  
              Layout : left-symmetric  
          Chunk Size : 512K  
  Consistency Policy : resync  
                Name : otuslinux:0  (local to host otuslinux)  
                UUID : 6f0cbe87:7c8e15c8:ea3bc5d3:a860f857  
              Events : 19  
      Number   Major   Minor   RaidDevice State  
         0       8       16        0      active sync   /dev/sdb  
         1       8       32        1      active sync   /dev/sdc  
         2       8       48        2      active sync   /dev/sdd  
         -       0        0        3      removed  
         4       8       80        4      active sync   /dev/sdf  
         3       8       64        -      faulty   /dev/sde  

### remove faulty disk from array
  [vagrant@otuslinux ~]$ sudo mdadm /dev/md0 --remove /dev/sde  
  mdadm: hot removed /dev/sde from /dev/md0  

### add new disk to RAID
  [vagrant@otuslinux ~]$ sudo mdadm /dev/md0 --add /dev/sde  
  mdadm: added /dev/sde  

### rebuild process
  [vagrant@otuslinux ~]$ cat /proc/mdstat  
  Personalities : [raid6] [raid5] [raid4]  
  md0 : active raid6 sde[5] sdf[4] sdd[2] sdc[1] sdb[0]  
        761856 blocks super 1.2 level 6, 512k chunk, algorithm 2 [5/4] [UUU_U]  
        [==>..................]  recovery = 12.4% (31752/253952) finish=0.1min speed=31752K/sec  
  unused devices: <none>  

## create GPT partition
### create GPT on RAID
  [vagrant@otuslinux ~]$ sudo parted -s /dev/md0 mklabel gpt

### create partitions
  [vagrant@otuslinux ~]$ sudo parted /dev/md0 mkpart primary ext4 0% 20%  
  Information: You may need to update /etc/fstab.

  [vagrant@otuslinux ~]$ sudo parted /dev/md0 mkpart primary ext4 20% 40%  
  Information: You may need to update /etc/fstab.

  [vagrant@otuslinux ~]$ sudo parted /dev/md0 mkpart primary ext4 40% 60%  
  Information: You may need to update /etc/fstab.

  [vagrant@otuslinux ~]$ sudo parted /dev/md0 mkpart primary ext4 60% 80%  
  Information: You may need to update /etc/fstab.

  [vagrant@otuslinux ~]$ sudo parted /dev/md0 mkpart primary ext4 80% 100%  
  Information: You may need to update /etc/fstab.

### create filesystem
  [vagrant@otuslinux ~]$ for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done  
  mke2fs 1.42.9 (28-Dec-2013)  
  Filesystem label=  
  OS type: Linux  
  Block size=1024 (log=0)  
  Fragment size=1024 (log=0)  
  Stride=512 blocks, Stripe width=1536 blocks  
  37696 inodes, 150528 blocks  
  7526 blocks (5.00%) reserved for the super user  
  First data block=1  
  Maximum filesystem blocks=33816576  
  19 block groups  
  8192 blocks per group, 8192 fragments per group  
  1984 inodes per group  
  Superblock backups stored on blocks:  
          8193, 24577, 40961, 57345, 73729  
  Allocating group tables: done  
  Writing inode tables: done  
  Creating journal (4096 blocks): done  
  Writing superblocks and filesystem accounting information: done  

### create dirs to mount partitions
  [vagrant@otuslinux ~]$ sudo mkdir -p /raid/part{1,2,3,4,5}  

### mount partitions
  [vagrant@otuslinux ~]$ for i in $(seq 1 5); do sudo mount /dev/md0p$i /raid/part$i; done

### df -h
  [vagrant@otuslinux ~]$ df -h  
  Filesystem      Size  Used Avail Use% Mounted on  
  devtmpfs        489M     0  489M   0% /dev  
  tmpfs           496M     0  496M   0% /dev/shm  
  tmpfs           496M  6.8M  489M   2% /run  
  tmpfs           496M     0  496M   0% /sys/fs/cgroup  
  /dev/sda1        40G  5.1G   35G  13% /  
  tmpfs           100M     0  100M   0% /run/user/1000  
  tmpfs           100M     0  100M   0% /run/user/0  
  /dev/md0p1      139M  1.6M  127M   2% /raid/part1  
  /dev/md0p2      140M  1.6M  128M   2% /raid/part2  
  /dev/md0p3      142M  1.6M  130M   2% /raid/part3  
  /dev/md0p4      140M  1.6M  128M   2% /raid/part4  
  /dev/md0p5      139M  1.6M  127M   2% /raid/part5  
