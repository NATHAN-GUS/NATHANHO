# TP 4 : Safé d partission
## 3. Vérification

🌞 Lister les périphériques de stockage branchés à la machine
```sh
picasso@NathanDebian:~$ lsblk
NAME                   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                      8:0    0   20G  0 disk
└─sda1                   8:1    0   20G  0 part
  ├─nathan-root        254:0    0  9.3G  0 lvm  /
  ├─nathan-Swap        254:1    0  1.9G  0 lvm  [SWAP]
  ├─nathan-home        254:2    0  1.9G  0 lvm  /home
  ├─nathan-var         254:3    0  2.8G  0 lvm  /var
  └─nathan-Espacelibre 254:4    0  2.8G  0 lvm
sr0                     11:0    1 1024M  0 rom
```

🌞 Lister les partitions en cours d'utilisation
```sh
picasso@NathanDebian:~$ df -h
Filesystem               Size  Used Avail Use% Mounted on
udev                     1.9G     0  1.9G   0% /dev
tmpfs                    392M  1.3M  391M   1% /run
/dev/mapper/nathan-root  9.1G  5.0G  3.6G  59% /
tmpfs                    2.0G     0  2.0G   0% /dev/shm
tmpfs                    5.0M  8.0K  5.0M   1% /run/lock
/dev/mapper/nathan-home  1.8G   24M  1.7G   2% /home
/dev/mapper/nathan-var   2.7G  408M  2.2G  16% /var
tmpfs                    392M  176K  392M   1% /run/user/1000
```

## 4. Taille et inodes

🌞 Lister la quantité d'inodes disponibles sur chaque partition
```sh
picasso@NathanDebian:~$ df -i
Filesystem              Inodes  IUsed  IFree IUse% Mounted on
udev                    492724    421 492303    1% /dev
tmpfs                   501094    773 500321    1% /run
/dev/mapper/nathan-root 610800 182942 427858   30% /
tmpfs                   501094      1 501093    1% /dev/shm
tmpfs                   501094      5 501089    1% /run/lock
/dev/mapper/nathan-home 121920    433 121487    1% /home
/dev/mapper/nathan-var  183264  11766 171498    7% /var
tmpfs                   100218    151 100067    1% /run/user/1000
```

🌞 Déterminer la taille (avec du) et l'inode (avec ls) de chacun des fichiers suivants :

* le kernel ! c'est le fichier qui commence par vmlinuz dans le dossier /boot/
  ```sh
  picasso@NathanDebian:/$ du -h /boot/vmlinuz*
  [7.9M    /boot/vmlinuz-6.1.0-25-amd64
  7.9M    /boot/vmlinuz-6.1.0-27-amd64]
 ```

```sh
picasso@NathanDebian:/$ ls -i /boot | grep vmlinuz
  37 vmlinuz-6.1.0-25-amd64
  45 vmlinuz-6.1.0-27-amd64
  ```

* le shell qu'on utilise (genre le "terminal" de la VM), il s'appelle bash
```sh
picasso@NathanDebian:/$ which bash
/usr/bin/bash
picasso@NathanDebian:/$ du -h "/usr/bin/bash"
1.3M    /usr/bin/bash
```
```sh
picasso@NathanDebian:/$ ls -i "/usr/bin/bash"
392361 /usr/bin/bash
```

* le fichier qui contient la liste des utilisateurs (tu connais son chemin non ?! si non, tu peux regarder ton TP précédent)
  ```sh
  picasso@NathanDebian:/$ du -h /etc/passwd
  4.0K    /etc/passwd
 ```

```sh
picasso@NathanDebian:/$ ls -i /etc/passwd
310182 /etc/passwd
```

## II. Partitioning
### 1. d disk partou

🌞 Repérer le nom des nouveaux disques
```sh
picasso@NathanDebian:~$ lsblk
NAME                   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                      8:0    0   20G  0 disk
└─sda1                   8:1    0   20G  0 part
  ├─nathan-root        254:0    0  9.3G  0 lvm  /
  ├─nathan-Swap        254:1    0  1.9G  0 lvm  [SWAP]
  ├─nathan-home        254:2    0  1.9G  0 lvm  /home
  ├─nathan-var         254:3    0  2.8G  0 lvm  /var
  └─nathan-Espacelibre 254:4    0  2.8G  0 lvm
sdb                      8:16   0   10G  0 disk
sdc                      8:32   0   10G  0 disk
sr0                     11:0    1 1024M  0 rom
```

🌞 Ajouter ces deux disques comme des PV LVM
```sh
picasso@NathanDebian:~$ sudo pvcreate /dev/sdb
[sudo] password for picasso:
  Physical volume "/dev/sdb" successfully created.
picasso@NathanDebian:~$ sudo pvcreate /dev/sdc
  Physical volume "/dev/sdc" successfully created.
```

🌞 Créer un VG nommé cat
```sh
```

🌞 Créer un LV nommé meoooow
```sh
picasso@NathanDebian:~$ sudo lvcreate -L 3G cat -n meoooow
  Logical volume "meoooow" created.
```

🌞 Formater la partition (le LV) en autre chose que ext4
```sh
picasso@NathanDebian:~$ sudo mkfs -t xfs /dev/cat/meoooow
meta-data=/dev/cat/meoooow       isize=512    agcount=4, agsize=196608 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=786432, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

🌞 Créer le point de montage /mnt/meow
```sh
picasso@NathanDebian:~$ sudo mkdir /mnt/meow
```

🌞 Monter la partition meoooow sur le point de montage que vous avez créé
```sh
picasso@NathanDebian:~$ sudo mount /dev/cat/meoooow /mnt/meow
```

🌞 Vérifier avec la commande adaptée que la partition est utilisable et qu'elle fait 3G
```sh
picasso@NathanDebian:~$ df -h
Filesystem               Size  Used Avail Use% Mounted on
udev                     1.9G     0  1.9G   0% /dev
tmpfs                    392M  1.2M  391M   1% /run
/dev/mapper/nathan-root  9.1G  5.0G  3.6G  59% /
tmpfs                    2.0G     0  2.0G   0% /dev/shm
tmpfs                    5.0M  8.0K  5.0M   1% /run/lock
/dev/mapper/nathan-home  1.8G   24M  1.7G   2% /home
/dev/mapper/nathan-var   2.7G  443M  2.1G  18% /var
tmpfs                    392M  140K  392M   1% /run/user/1000
/dev/mapper/cat-meoooow  3.0G   54M  2.9G   2% /mnt/meow
```

## 2. Grow !

🌞 Il reste 2G sur le disque dur principal
```sh

```

🌞 Agrandir la partition meoooow pour qu'elle occupe tout l'espace libre de son VG
```sh
picasso@NathanDebian:~$ sudo lvextend -l +100%FREE /dev/cat/meoooow
  New size (5118 extents) matches existing size (5118 extents).
picasso@NathanDebian:~$ lsblk
NAME                   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
sda                      8:0    0   20G  0 disk
└─sda1                   8:1    0   20G  0 part
  ├─nathan-root        254:1    0  9.3G  0 lvm  /
  ├─nathan-Swap        254:2    0  1.9G  0 lvm  [SWAP]
  ├─nathan-home        254:3    0  3.2G  0 lvm  /home
  ├─nathan-var         254:4    0  2.8G  0 lvm  /var
  └─nathan-Espacelibre 254:5    0  2.8G  0 lvm
sdb                      8:16   0   10G  0 disk
└─cat-meoooow          254:0    0   20G  0 lvm  /mnt/meow
sdc                      8:32   0   10G  0 disk
└─cat-meoooow          254:0    0   20G  0 lvm  /mnt/meow
sr0                     11:0    1 1024M  0 rom
picasso@NathanDebian:~$  sudo xfs_growfs /dev/cat/meoooow
meta-data=/dev/mapper/cat-meoooow isize=512    agcount=4, agsize=196608 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1    bigtime=1 inobtcount=1 nrext64=0
data     =                       bsize=4096   blocks=786432, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=16384, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 786432 to 5240832
```

🌞  Déterminer la taille de la nouvelle partition
```sh
picasso@NathanDebian:~$ df -h /dev/cat/meoooow
Filesystem               Size  Used Avail Use% Mounted on
/dev/mapper/cat-meoooow   20G  176M   20G   1% /mnt/meow
```




## 3. Montage automatique

🌞 Modifier le fichier /etc/fstab pour que la partition soit montée /mnt/meow automatiquement
```sh
  GNU nano 7.2                     /etc/fstab                               # /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# systemd generates mount units based on this file, see systemd.mount(5).
# Please run 'systemctl daemon-reload' after making changes here.
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
/dev/mapper/nathan-root /               ext4    errors=remount-ro 0       1
/dev/mapper/nathan-home /home           ext4    defaults        0       2
/dev/mapper/nathan-var /var            ext4    defaults        0       2
/dev/mapper/nathan-Swap none            swap    sw              0       0
/dev/sr0        /media/cdrom0   udf,iso9660 user,noauto     0       0
/dev/cat/meoooow /mnt/meow            xfs      defaults    0        0





















                             [ Read 16 lines ]
^G Help        ^O Write Out   ^W Where Is    ^K Cut         ^T Execute
^X Exit        ^R Read File   ^\ Replace     ^U Paste       ^J Justify
```

```sh
picasso@NathanDebian:~$ sudo nano /etc/fstab
[sudo] password for picasso:
Sorry, try again.
[sudo] password for picasso:
picasso@NathanDebian:~$ sudo umount /mnt/meow
picasso@NathanDebian:~$ sudo mount -av
/                        : ignored
/home                    : already mounted
/var                     : already mounted
none                     : ignored
/media/cdrom0            : ignored
mount: (hint) your fstab has been modified, but systemd still uses
       the old version; use 'systemctl daemon-reload' to reload.
/mnt/meow                : successfully mounted
picasso@NathanDebian:~$ sudo reboot

Broadcast message from root@NathanDebian on pts/1 (Mon 2024-11-18 03:57:06 CET):

The system will reboot now!

picasso@NathanDebian:~$ Connection to 192.168.202.5 closed by remote host.
Connection to 192.168.202.5 closed.
```

## 4. Bonus : options de montage

⭐ Déterminer quelles options de montage permettrait d'améliorer le niveau de sécurité des données contenues sur une partition.
```sh























  
  




