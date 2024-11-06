# OTUS PRO Homework 06-LVM

### Домашняя работа 6: Файловые системы и LVM (реализована на Debian 12)
---
### Домашнее задание:
   - Уменьшить том под / до 8GB.  
   - Выделить том под /home.  
   - Выделить том под /var - сделать в mirror.
   - /home - сделать том для снапшотов. 
   - Прописать монтирование в fstab. Попробовать с разными опциями и разными файловыми системами (на выбор).
   - Работа со снапшотами:
     - сгенерить файлы в /home/;
     - снять снапшот;
     - удалить часть файлов;
     - восстановится со снапшота.
   - *На дисках попробовать поставить btrfs/zfs — с кешем, снапшотами и разметить там каталог /opt.
---
### Выполнение задания:
посмотрим на диски, выполнив команды: `df -hT` и `lsblk`
<details>
<summary>
   результат выполнения команд 
</summary>
   
`df -hT`

```
Filesystem               Type      Size  Used Avail Use% Mounted on
udev                     devtmpfs  457M     0  457M   0% /dev
tmpfs                    tmpfs      97M  508K   96M   1% /run
/dev/mapper/VG_ROOT-root xfs       3.3G  1.3G  2.0G  40% /
tmpfs                    tmpfs     481M     0  481M   0% /dev/shm
tmpfs                    tmpfs     5.0M     0  5.0M   0% /run/lock
/dev/sda1                ext2      444M   60M  380M  14% /boot
tmpfs                    tmpfs      97M     0   97M   0% /run/user/0
```
`lsblk`
```
NAME             MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                8:0    0   20G  0 disk
├─sda1             8:1    0  476M  0 part /boot
└─sda2             8:2    0 19.5G  0 part
  ├─VG_ROOT-root 254:0    0  3.4G  0 lvm  /
  └─VG_ROOT-swap 254:1    0  1.2G  0 lvm  [SWAP]
sdb                8:16   0   10G  0 disk
sdc                8:32   0    2G  0 disk
sdd                8:48   0    1G  0 disk
sde                8:64   0    1G  0 disk
sr0               11:0    1  3.7G  0 rom
```
</details>

---
### 1) Уменьшим том под / до 8G:
   
**1.1 Установим пакет xfsdump:**
   
`apt update -y && apt install -y xfsdump`
<details>
<summary> результат выполнения команды: </summary>
   
```
Hit:1 http://deb.debian.org/debian bookworm InRelease
Hit:2 http://deb.debian.org/debian-security bookworm-security InRelease
Hit:3 http://deb.debian.org/debian bookworm-updates InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
1 package can be upgraded. Run 'apt list --upgradable' to see it.
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  libgpm2 libncurses6
Suggested packages:
  gpm acl attr quota
The following NEW packages will be installed:
  libgpm2 libncurses6 xfsdump
0 upgraded, 3 newly installed, 0 to remove and 1 not upgraded.
Need to get 372 kB of archives.
After this operation, 1,257 kB of additional disk space will be used.
Get:1 http://deb.debian.org/debian bookworm/main amd64 libgpm2 amd64 1.20.7-10+b1 [14.2 kB]
Get:2 http://deb.debian.org/debian bookworm/main amd64 libncurses6 amd64 6.4-4 [103 kB]
Get:3 http://deb.debian.org/debian bookworm/main amd64 xfsdump amd64 3.1.11-0.1 [254 kB]
Fetched 372 kB in 0s (1,306 kB/s)
Selecting previously unselected package libgpm2:amd64.
(Reading database ... 28654 files and directories currently installed.)
Preparing to unpack .../libgpm2_1.20.7-10+b1_amd64.deb ...
Unpacking libgpm2:amd64 (1.20.7-10+b1) ...
Selecting previously unselected package libncurses6:amd64.
Preparing to unpack .../libncurses6_6.4-4_amd64.deb ...
Unpacking libncurses6:amd64 (6.4-4) ...
Selecting previously unselected package xfsdump.
Preparing to unpack .../xfsdump_3.1.11-0.1_amd64.deb ...
Unpacking xfsdump (3.1.11-0.1) ...
Setting up libgpm2:amd64 (1.20.7-10+b1) ...
Setting up libncurses6:amd64 (6.4-4) ...
Setting up xfsdump (3.1.11-0.1) ...
Processing triggers for man-db (2.11.2-2) ...
Processing triggers for libc-bin (2.36-9+deb12u8) ...

```
   
</details>

**1.2 Подготовка вреенного тома для раздела::**
   
`pvcreate /dev/sdb`
<details>
<summary> результат выполнения команды: </summary>

```
  Physical volume "/dev/sdb" successfully created.
```
</details>

Создадим Volume group **vg_root** на разделе **/dev/sdb**:

`vgcreate vg_root /dev/sdb`

<details>
<summary> результат выполнения команды: </summary>

```
  Volume group "vg_root" successfully created
```
</details>

Создадим Logical Volume **lv_root** на Volume Group **vg_root**:

`lvcreate -n lv_root -l +100%FREE /dev/vg_root`

<details>
<summary> результат выполнения команды: </summary>

```
  Logical volume "lv_root" created.
```
</details>
