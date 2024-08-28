# Методическое пособие по выполнению домашнего задания курса «Администратор Linux. Professional»

## Стенд с Vagrant c ZFS

## 1. Цели домашнего задания

### Научится самостоятельно устанавливать ZFS, настраивать пулы, изучить основные возможности ZFS.

## 2. Описание домашнего задания

1. Определить алгоритм с наилучшим сжатием:  
	- Определить какие алгоритмы сжатия поддерживает zfs (gzip, zle, lzjb, lz4);  
	- создать 4 файловых системы на каждой применить свой алгоритм сжатия;  
	- для сжатия использовать либо текстовый файл, либо группу файлов.  
2. Определить настройки пула.  
С помощью команды zfs import собрать pool ZFS.  
Командами zfs определить настройки:  
    - размер хранилища;  
    - тип pool;  
    - значение recordsize;  
    - какое сжатие используется;  
    - какая контрольная сумма используется.  
3. Работа со снапшотами:  
	- скопировать файл из удаленной директории;  
	- восстановить файл локально. zfs receive;  
	- найти зашифрованное сообщение в файле secret_message.  

## Определение алгоритма с наилучшим сжатием  
### list of block devices  
[root@zfs ~]# lsblk  
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT  
sda      8:0    0  512M  0 disk  
sdb      8:16   0  512M  0 disk  
sdc      8:32   0  512M  0 disk  
sdd      8:48   0  512M  0 disk  
sde      8:64   0  512M  0 disk  
sdf      8:80   0  512M  0 disk  
sdg      8:96   0  512M  0 disk  
sdh      8:112  0  512M  0 disk  
sdi      8:128  0   40G  0 disk  
└─sdi1   8:129  0   40G  0 part /  

### create 4 zpools of 2 disks  
[root@zfs ~]# zpool create otus1 mirror /dev/sda /dev/sdb  
[root@zfs ~]# zpool create otus2 mirror /dev/sdd /dev/sdc  
[root@zfs ~]# zpool create otus3 mirror /dev/sde /dev/sdf  
[root@zfs ~]# zpool create otus4 mirror /dev/sdg /dev/sdh  
[root@zfs ~]# zpool list  
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT  
otus1   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -  
otus2   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -  
otus3   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -  
otus4   480M  91.5K   480M        -         -     0%     0%  1.00x    ONLINE  -  

### add compression algoritm to every filesystem  
[root@zfs ~]# zfs set compression=lzjb otus1  
[root@zfs ~]# zfs set compression=lz4 otus2  
[root@zfs ~]# zfs set compression=gzip-9 otus3  
[root@zfs ~]# zfs set compression=zle otus4  
[root@zfs ~]# zfs get all | grep compression  
otus1  compression           lzjb                   local  
otus2  compression           lz4                    local  
otus3  compression           gzip-9                 local  
otus4  compression           zle                    local  

### download same file in all pools  
[root@zfs ~]# for i in {1..4}; do wget -P /otus$i https://gutenberg.org/cache/epub/2600/pg2600.converter.log; done  
--2024-06-14 11:03:26--  https://gutenberg.org/cache/epub/2600/pg2600.converter.log  
Resolving gutenberg.org (gutenberg.org)... 152.19.134.47, 2610:28:3090:3000:0:bad:cafe:47  
Connecting to gutenberg.org (gutenberg.org)|152.19.134.47|:443... connected.  
HTTP request sent, awaiting response... 200 OK  
Length: 41052631 (39M) [text/plain]  
Saving to: ‘/otus1/pg2600.converter.log’  
[root@zfs ~]# ls -l /otus*  
/otus1:  
total 22079  
-rw-r--r--. 1 root root 41052631 Jun  2 08:03 pg2600.converter.log  

/otus2:  
total 17999  
-rw-r--r--. 1 root root 41052631 Jun  2 08:03 pg2600.converter.log  

/otus3:  
total 10962  
-rw-r--r--. 1 root root 41052631 Jun  2 08:03 pg2600.converter.log  

/otus4:  
total 40118  
-rw-r--r--. 1 root root 41052631 Jun  2 08:03 pg2600.converter.log  

### check size of file and compression rate in diffrent pools  
[root@zfs ~]# zfs list  
NAME    USED  AVAIL     REFER  MOUNTPOINT  
otus1  21.7M   330M     21.6M  /otus1  
otus2  17.7M   334M     17.6M  /otus2  
otus3  10.8M   341M     10.7M  /otus3  
otus4  39.3M   313M     39.2M  /otus4  
[root@zfs ~]# zfs get all | grep compressratio | grep -v ref  
otus1  compressratio         1.82x                  -  
otus2  compressratio         2.23x                  -  
otus3  compressratio         3.66x                  -  
otus4  compressratio         1.00x                  -  

## Определение настроек пула  
### download archive in home dir and unzip it  
[root@zfs ~]# wget -O archive.tar.gz --no-check-certificate 'https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download'  
--2024-06-14 11:08:50--  https://drive.usercontent.google.com/download?id=1MvrcEp-WgAQe57aDEzxSRalPAwbNN1Bb&export=download  
Resolving drive.usercontent.google.com (drive.usercontent.google.com)... 74.125.205.132, 2a00:1450:4010:c02::84  
Connecting to drive.usercontent.google.com (drive.usercontent.google.com)|74.125.205.132|:443... connected.  
HTTP request sent, awaiting response... 200 OK  
Length: 7275140 (6.9M) [application/octet-stream]  
Saving to: ‘archive.tar.gz’  
100%[===================================================================================>] 7,275,140   6.65MB/s   in 1.0s  
2024-06-14 11:08:57 (6.65 MB/s) - ‘archive.tar.gz’ saved [7275140/7275140]  
[root@zfs ~]# tar -xzvf archive.tar.gz  
zpoolexport/  
zpoolexport/filea  
zpoolexport/fileb  

### check if possible to import dir in pool  
[root@zfs ~]# zpool import -d zpoolexport/  
   pool: otus  
     id: 6554193320433390805  
  state: ONLINE  
 action: The pool can be imported using its name or numeric identifier.  
 config:  
        otus                         ONLINE  
          mirror-0                   ONLINE  
            /root/zpoolexport/filea  ONLINE  
            /root/zpoolexport/fileb  ONLINE  

### import pool in OS  
[root@zfs ~]# zpool import -d zpoolexport/ otus  
[root@zfs ~]# zpool status  
  pool: otus  
 state: ONLINE  
  scan: none requested  
config:  
        NAME                         STATE     READ WRITE CKSUM  
        otus                         ONLINE       0     0     0  
          mirror-0                   ONLINE       0     0     0  
            /root/zpoolexport/filea  ONLINE       0     0     0  
            /root/zpoolexport/fileb  ONLINE       0     0     0  
errors: No known data errors  

### check config otus pool  
[root@zfs ~]# zfs get all otus  
NAME  PROPERTY              VALUE                  SOURCE  
otus  type                  filesystem             -  
otus  creation              Fri May 15  4:00 2020  -  

### check available space  
[root@zfs ~]# zfs get available otus  
NAME  PROPERTY   VALUE  SOURCE  
otus  available  350M   -  

### check type  
[root@zfs ~]# zfs get readonly otus  
NAME  PROPERTY  VALUE   SOURCE  
otus  readonly  off     default  

### check recordsize  
[root@zfs ~]# zfs get recordsize otus  
NAME  PROPERTY    VALUE    SOURCE  
otus  recordsize  128K     local  

### check compression type  
[root@zfs ~]# zfs get compression otus  
NAME  PROPERTY     VALUE     SOURCE  
otus  compression  zle       local  

### check checksum type  
[root@zfs ~]# zfs get checksum otus  
NAME  PROPERTY  VALUE      SOURCE  
otus  checksum  sha256     local  

## Работа со снапшотом, поиск сообщения от преподавателя  
### download file https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download   
[root@zfs ~]# wget -O otus_task2.file --no-check-certificate https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI&export=download  
[1]+  Done                    wget -O otus_task2.file --no-check-certificate https://drive.usercontent.google.com/download?id=1wgxjih8YZ-cqLqaZVa0lA3h3Y029c3oI  

### restore filesystem from snapshot  
[root@zfs ~]# zfs receive otus/test@today < otus_task2.file  
 
### find file secret_message in dir /otus/test  
[root@zfs ~]# find /otus/test -name "secret_message"  
/otus/test/task1/file_mess/secret_message  

### secret message from teacher  
[root@zfs ~]# cat /otus/test/task1/file_mess/secret_message  
https://otus.ru/lessons/linux-hl/  
