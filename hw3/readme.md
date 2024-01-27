- создайте виртуальную машину c Ubuntu 20.04/22.04 LTS в GCE/ЯО/Virtual Box/докере
- поставьте на нее PostgreSQL 15 через sudo apt
- проверьте что кластер запущен через sudo -u postgres pg_lsclusters
```shell
focus@postgres-hw-3:~$ sudo -u postgres pg_lsclusters 
Ver Cluster Port Status Owner    Data directory              Log file
15  main    5432 online postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
```
- зайдите из под пользователя postgres в psql и сделайте произвольную таблицу с произвольным содержимым
```shell
postgres=# create table test(c1 text);
CREATE TABLE
postgres=# 
postgres=# 
postgres=# insert into test values ('1');
INSERT 0 1
postgres=# select * from test;
 c1 
----
 1
(1 row) 
```
- остановите postgres например через sudo -u postgres pg_ctlcluster 15 main stop
```shell
focus@postgres-hw-3:~$ sudo -u postgres pg_ctlcluster 15 main stop
Warning: stopping the cluster using pg_ctlcluster will mark the systemd unit as failed. Consider using systemctl:
  sudo systemctl stop postgresql@15-main
focus@postgres-hw-3:~$ sudo -u postgres pg_lsclusters 
Ver Cluster Port Status Owner    Data directory              Log file
15  main    5432 down   postgres /var/lib/postgresql/15/main /var/log/postgresql/postgresql-15-main.log
```
- создайте новый диск к ВМ размером 10GB

- добавьте свеже-созданный диск к виртуальной машине - надо зайти в режим ее редактирования и дальше выбрать пункт attach existing disk
```shell
focus@postgres-hw-3:~$ sudo lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT,LABEL
NAME   FSTYPE     SIZE MOUNTPOINT        LABEL
loop0  squashfs  63.3M /snap/core20/1822 
loop1  squashfs 111.9M /snap/lxd/24322   
loop2  squashfs  49.8M /snap/snapd/18357 
vda                18G                   
├─vda1              1M                   
└─vda2 ext4        18G /                 
vdb                20G  
```
- проинициализируйте диск согласно инструкции и подмонтировать файловую систему, только не забывайте менять имя диска на актуальное, в вашем случае это скорее всего будет /dev/sdb - https://www.digitalocean.com/community/tutorials/how-to-partition-and-format-storage-devices-in-linux
```shell
focus@postgres-hw-3:~$ sudo apt update
focus@postgres-hw-3:~$ sudo apt install parted
focus@postgres-hw-3:~$ sudo parted -l | grep Error
Error: /dev/vdb: unrecognised disk label
focus@postgres-hw-3:~$ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0  63.3M  1 loop /snap/core20/1822
loop1    7:1    0 111.9M  1 loop /snap/lxd/24322
loop2    7:2    0  49.8M  1 loop /snap/snapd/18357
vda    252:0    0    18G  0 disk 
├─vda1 252:1    0     1M  0 part 
└─vda2 252:2    0    18G  0 part /
vdb    252:16   0    20G  0 disk
focus@postgres-hw-3:~$ sudo parted /dev/vdb mklabel gpt
Information: You may need to update /etc/fstab.
focus@postgres-hw-3:~$ sudo parted -a opt /dev/vdb mkpart primary ext4 0% 100%
Information: You may need to update /etc/fstab.
focus@postgres-hw-3:~$ lsblk                                              
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0  63.3M  1 loop /snap/core20/1822
loop1    7:1    0 111.9M  1 loop /snap/lxd/24322
loop2    7:2    0  49.8M  1 loop /snap/snapd/18357
vda    252:0    0    18G  0 disk 
├─vda1 252:1    0     1M  0 part 
└─vda2 252:2    0    18G  0 part /
vdb    252:16   0    20G  0 disk 
└─vdb1 252:17   0    20G  0 part
focus@postgres-hw-3:~$ sudo mkfs.ext4 -L datapartition /dev/vdb1
mke2fs 1.46.5 (30-Dec-2021)
Creating filesystem with 5242368 4k blocks and 1310720 inodes
Filesystem UUID: 603f2ab9-82d9-4ac7-a6e4-a81d7d4e1877
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
focus@postgres-hw-3:~$ sudo lsblk --fs
NAME   FSTYPE   FSVER LABEL         UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
loop0  squashfs 4.0                                                            0   100% /snap/core20/1822
loop1  squashfs 4.0                                                            0   100% /snap/lxd/24322
loop2  squashfs 4.0                                                            0   100% /snap/snapd/18357
vda                                                                                     
├─vda1                                                                                  
└─vda2 ext4     1.0                 ed465c6e-049a-41c6-8e0b-c8da348a3577   11.7G    29% /
vdb                                                                                     
└─vdb1 ext4     1.0   datapartition 603f2ab9-82d9-4ac7-a6e4-a81d7d4e1877
focus@postgres-hw-3:~$ sudo mkdir -p /mnt/data
focus@postgres-hw-3:~$ sudo mount -o defaults /dev/vdb1 /mnt/data
focus@postgres-hw-3:~$ df -h -x tmpfs
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda2        18G  5.2G   12G  31% /
/dev/vdb1        20G   24K   19G   1% /mnt/data
focus@postgres-hw-3:~$ echo "success" | sudo tee /mnt/data/test_file
success
focus@postgres-hw-3:~$ cat /mnt/data/test_file
success
```
- перезагрузите инстанс и убедитесь, что диск остается примонтированным (если не так смотрим в сторону fstab)
```shell
focus@postgres-hw-3:~$ df -h -x tmpfs
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda2        18G  5.2G   12G  31% /
/dev/vdb1        20G   28K   19G   1% /mnt/data 
focus@postgres-hw-3:~$ cat /mnt/data/test_file
success
```
- сделайте пользователя postgres владельцем /mnt/data - chown -R postgres:postgres /mnt/data/
```shell
focus@postgres-hw-3:~$ ls -l /mnt/data/
total 20
drwx------ 2 root root 16384 Jan 27 10:10 lost+found
-rw-r--r-- 1 root root     8 Jan 27 10:15 test_file
focus@postgres-hw-3:~$ sudo chown -R postgres:postgres /mnt/data/
focus@postgres-hw-3:~$ ls -l /mnt/data/
total 20
drwx------ 2 postgres postgres 16384 Jan 27 10:10 lost+found
-rw-r--r-- 1 postgres postgres     8 Jan 27 10:15 test_file 
```

- перенесите содержимое /var/lib/postgres/15 в /mnt/data - mv /var/lib/postgresql/15/mnt/data
```shell
focus@postgres-hw-3:~$ sudo mv /var/lib/postgresql/15 /mnt/data
focus@postgres-hw-3:~$ ls -l /mnt/data/
total 24
drwxr-xr-x 3 postgres postgres  4096 Jan 27 09:41 15
drwx------ 2 postgres postgres 16384 Jan 27 10:10 lost+found
-rw-r--r-- 1 postgres postgres     8 Jan 27 10:15 test_file
focus@postgres-hw-3:~$ ls -l /var/lib/postgresql/15
ls: cannot access '/var/lib/postgresql/15': No such file or directory 
```
- попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
  напишите получилось или нет и почему? Директория пустая. Нужно задать путь в /mnt.
```shell
focus@postgres-hw-3:~$ sudo -u postgres pg_ctlcluster 15 main start
Error: /var/lib/postgresql/15/main is not accessible or does not exist 
```

- задание: найти конфигурационный параметр в файлах раположенных в /etc/postgresql/15/main который надо поменять и поменяйте его
  напишите что и почему поменяли
```shell
focus@postgres-hw-3:~$ sudo vim /etc/postgresql/15/main/postgresql.conf
data_directory = '/mnt/data/15/main' 
```
- попытайтесь запустить кластер - sudo -u postgres pg_ctlcluster 15 main start
  напишите получилось или нет и почему
```shell
focus@postgres-hw-3:~$ sudo -u postgres pg_ctlcluster 15 main start
Warning: the cluster will not be running as a systemd service. Consider using systemctl:
  sudo systemctl start postgresql@15-main
Removed stale pid file.

```
- зайдите через через psql и проверьте содержимое ранее созданной таблицы
```
postgres=# select * from test;
 c1 
----
 1
(1 row)

```