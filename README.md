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

Создадим файловую систему **xfs** на созданном Logical volume **lv_root**:

`mkfs.xfs /dev/vg_root/lv_root`

<details>
<summary> результат выполнения команды: </summary>

```
meta-data=/dev/vg_root/lv_root   isize=512    agcount=4, agsize=655104 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=2620416, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
Discarding blocks...Done.
```
</details>

Смонтируем раздел **/dev/vg_root/lv_root** в **/mnt**:

`mount /dev/vg_root/lv_root /mnt`

**1.3 Скопируем все данные с раздела / в /mnt:**

`xfsdump -J - /dev/VG_ROOT/root | xfsrestore -J - /mnt`

<details>
<summary> результат выполнения команды: </summary>

```
xfsrestore: using file dump (drive_simple) strategy
xfsrestore: version 3.1.11 (dump format 3.0)xfsdump:
using file dump (drive_simple) strategy
xfsdump: version 3.1.11 (dump format 3.0)
xfsdump: level 0 dump of testdeb1:/
xfsdump: dump date: Wed Nov  6 13:58:18 2024
xfsdump: session id: c49b7587-f750-40b2-80ec-b34861f7d6ab
xfsdump: session label: ""
xfsrestore: searching media for dump
xfsdump: ino map phase 1: constructing initial dump list
xfsdump: ino map phase 2: skipping (no pruning necessary)
xfsdump: ino map phase 3: skipping (only one dump stream)
xfsdump: ino map construction complete
xfsdump: estimated dump size: 1320175232 bytes
xfsdump: creating dump session media file 0 (media 0, file 0)
xfsdump: dumping ino map
xfsdump: dumping directories
xfsrestore: examining media file 0
xfsrestore: dump description:
xfsrestore: hostname: testdeb1
xfsrestore: mount point: /
xfsrestore: volume: /dev/mapper/VG_ROOT-root
xfsrestore: session time: Wed Nov  6 13:58:18 2024
xfsrestore: level: 0
xfsrestore: session label: ""
xfsrestore: media label: ""
xfsrestore: file system id: 13c1b0bd-7496-4aa2-93e5-709d9bd62ed9
xfsrestore: session id: c49b7587-f750-40b2-80ec-b34861f7d6ab
xfsrestore: media id: f81b9a64-698e-4ff9-bba7-829d3d23ad3a
xfsrestore: searching media for directory dump
xfsrestore: reading directories
xfsdump: dumping non-directory files
xfsrestore: 3373 directories and 33174 entries processed
xfsrestore: directory post-processing
xfsrestore: restoring non-directory files
xfsdump: ending media file
xfsdump: media file size 1282148400 bytes
xfsdump: dump size (non-dir files) : 1264793016 bytes
xfsdump: dump complete: 19 seconds elapsed
xfsdump: Dump Status: SUCCESS
xfsrestore: restore complete: 19 seconds elapsed
xfsrestore: Restore Status: SUCCESS
```
</details>

Проверим, что содержимое раздела **/** скопировалось в **/mnt**:

`ls -la /mnt`

<details>
<summary> результат выполнения команды: </summary>

```
total 8
drwxr-xr-x 17 root root  298 Nov  6 13:58 .
drwxr-xr-x 17 root root  298 Nov  5 17:07 ..
lrwxrwxrwx  1 root root    7 Nov  6 13:58 bin -> usr/bin
drwxr-xr-x  2 root root    6 Nov  5 17:06 boot
drwxr-xr-x  4 root root  182 Nov  5 17:06 dev
drwxr-xr-x 68 root root 4096 Nov  6 08:40 etc
drwxr-xr-x  3 root root   18 Nov  5 17:10 home
lrwxrwxrwx  1 root root   30 Nov  6 13:58 initrd.img -> boot/initrd.img-6.1.0-18-amd64
lrwxrwxrwx  1 root root   30 Nov  6 13:58 initrd.img.old -> boot/initrd.img-6.1.0-18-amd64
lrwxrwxrwx  1 root root    7 Nov  6 13:58 lib -> usr/lib
lrwxrwxrwx  1 root root    9 Nov  6 13:58 lib64 -> usr/lib64
drwxr-xr-x  3 root root   33 Nov  5 17:06 media
drwxr-xr-x  2 root root    6 Nov  5 17:06 mnt
drwxr-xr-x  2 root root    6 Nov  5 17:06 opt
drwxr-xr-x  2 root root    6 Jan 29  2024 proc
drwx------  4 root root   84 Nov  5 18:45 root
drwxr-xr-x  2 root root    6 Nov  5 17:11 run
lrwxrwxrwx  1 root root    8 Nov  6 13:58 sbin -> usr/sbin
drwxr-xr-x  2 root root    6 Nov  5 17:06 srv
drwxr-xr-x  2 root root    6 Jan 29  2024 sys
drwxrwxrwt  8 root root  250 Nov  6 03:20 tmp
drwxr-xr-x 12 root root  133 Nov  5 17:06 usr
drwxr-xr-x 11 root root  139 Nov  5 17:06 var
lrwxrwxrwx  1 root root   27 Nov  6 13:58 vmlinuz -> boot/vmlinuz-6.1.0-18-amd64
lrwxrwxrwx  1 root root   27 Nov  6 13:58 vmlinuz.old -> boot/vmlinuz-6.1.0-18-amd64
```
</details>

##### Сконфигурируем grub для того, чтобы при старте перейти в новый /  
Сымитируем текущий root, сделаем в него chroot и обновим grub:

```
for i in /proc/ /sys/ /dev/ /run/ /boot/; \
do mount --bind $i /mnt/$i; done
chroot /mnt/
```
Поищем строки содержащие старый **/** раздел в файлах **/etc/fstab** и **/boot/grub/grub.cfg**

`cat /etc/fstab | grep VG_ROOT`

<details>
<summary> результат выполнения команды: </summary>

```
/dev/mapper/VG_ROOT-root /               xfs     defaults        0       0
/dev/mapper/VG_ROOT-swap none            swap    sw              0       0
```
</details>

`cat /boot/grub/grub.cfg | grep VG_ROOT`

<details>
<summary> результат выполнения команды: </summary>

```
        linux   /vmlinuz-6.1.0-18-amd64 root=/dev/mapper/VG_ROOT-root ro  quiet
                linux   /vmlinuz-6.1.0-18-amd64 root=/dev/mapper/VG_ROOT-root ro  quiet
                linux   /vmlinuz-6.1.0-18-amd64 root=/dev/mapper/VG_ROOT-root ro single
```
</details>

Выполним замену **VG_ROOT-root** на **vg_root-lv_root** в файлах **/etc/fstab** и **/boot/grub/grub.cfg** и проверим что замена произошла:  
`sed -i 's/VG_ROOT-root/vg_root-lv_root/g' /etc/fstab && cat /etc/fstab | grep vg_root-lv_root`

<details>
<summary> результат выполнения команды: </summary>
   
```
/dev/mapper/vg_root-lv_root /               xfs     defaults        0       0
```
</details>

`sed -i 's/VG_ROOT-root/vg_root-lv_root/g' /boot/grub/grub.cfg && cat /boot/grub/grub.cfg | grep vg_root-lv_root`

<details>
<summary> результат выполнения команды: </summary>
   
```
        linux   /vmlinuz-6.1.0-18-amd64 root=/dev/mapper/vg_root-lv_root ro  quiet
                linux   /vmlinuz-6.1.0-18-amd64 root=/dev/mapper/vg_root-lv_root ro  quiet
                linux   /vmlinuz-6.1.0-18-amd64 root=/dev/mapper/vg_root-lv_root ro single
```
</details>

**Обновим образ initrd**

`update-initramfs -c -k all`

<details>
<summary> результат выполнения команды: </summary>
   
```
update-initramfs: Generating /boot/initrd.img-6.1.0-18-amd64
```
</details>
