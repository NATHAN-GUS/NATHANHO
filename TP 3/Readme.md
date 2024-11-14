
# TP 3
#  I. Utilisateurs

#### 1. Liste des users
 🌞 Afficher la ligne du fichier qui concerne votre utilisateur
```sh
nathan@ServeurDebian:/$ sudo cat /etc/passwd | grep nathan
nathan:x:1000:1000:nathan,,,:/home/nathan:/bin/bash
 ```
🌞 Afficher la ligne du fichier qui concerne votre utilisateur ET celle de root en même temps
 ```sh  
nathan@ServeurDebian:/$ cat /etc/passwd | grep -E '^(root|nathan):'
root:x:0:0:root:/root:/bin/bash
nathan:x:1000:1000:nathan,,,:/home/nathan:/bin/bash
```
🌞 Afficher la liste des groupes d'utilisateurs de la machine
* ptite recherche par vous-mêmes pour trouver quel fichier c'est !
```sh
nathan@ServeurDebian:/$ cat /etc/group
root:x:0:
daemon:x:1:
bin:x:2:
sys:x:3:
adm:x:4:
tty:x:5:
disk:x:6:
lp:x:7:
mail:x:8:
news:x:9:
uucp:x:10:
man:x:12:
proxy:x:13:
kmem:x:15:
dialout:x:20:
fax:x:21:
voice:x:22:
cdrom:x:24:nathan
floppy:x:25:nathan
tape:x:26:
sudo:x:27:nathan
audio:x:29:pulse,nathan
dip:x:30:nathan
www-data:x:33:
backup:x:34:
operator:x:37:
list:x:38:
irc:x:39:
src:x:40:
shadow:x:42:
utmp:x:43:
video:x:44:nathan
sasl:x:45:
plugdev:x:46:nathan
staff:x:50:
games:x:60:
users:x:100:nathan
nogroup:x:65534:
systemd-journal:x:999:
systemd-network:x:998:
crontab:x:101:
input:x:102:
sgx:x:103:
kvm:x:104:
render:x:105:
netdev:x:106:nathan
messagebus:x:107:
_ssh:x:108:
ssl-cert:x:109:
avahi-autoipd:x:110:
bluetooth:x:111:nathan
avahi:x:112:
lpadmin:x:113:nathan
pulse:x:114:
pulse-access:x:115:
scanner:x:116:saned,nathan
saned:x:117:
lightdm:x:118:
polkitd:x:997:
rtkit:x:119:
colord:x:120:
nathan:x:1000:
marmotte:x:1001:
```
🌞 Afficher la ligne du fichier qui concerne votre utilisateur ET celle de root en même temps
```su
nathan@ServeurDebian:/$ cat /etc/passwd | cut -d":" -f1,6 |  grep -E "nathan|root"
root:/root
nathan:/home/nathan
```

### 2. Hash des passwords
🌞 Afficher la ligne qui contient le hash du mot de passe de votre utilisateur
```sh
nathan@ServeurDebian:/$ sudo cat /etc/shadow | grep nathan
nathan:$y$j9T$QSQK4oL/cSJlyQ0PYf/Yc.$FaTPvwuVDv0FjiSiunMX19fcHLIFgzMdGcwVvpV6ycB:20036:0:99999:7:::
```
# 3. Sudo

## B. Practice
🌞 Créer un groupe d'utilisateurs
```sh
nathan@ServeurDebian:/$ sudo groupadd stronk_admins
```
🌞 Créer un utilisateur
```sh
nathan@ServeurDebian:/$ sudo adduser imbob
Adding user `imbob' ...
Adding new group `imbob' (1003) ...
Adding new user `imbob' (1003) with group `imbob (1003)' ...
adduser: The home directory `/home/imbob' already exists.  Not touching this directory.
adduser: Warning: The home directory `/home/imbob' does not belong to the user you are currently creating.
New password:
Retype new password:
passwd: password updated successfully

nathan@ServeurDebian:/$ sudo usermod -aG stronk_admins imbob

```

🌞 Prouver que l'utilisateur imbob est créé
```sh
nathan@ServeurDebian:/$ sudo cat /etc/passwd | grep imbob
imbob:x:1003:1003:,,,:/home/imbob:/bin/bash
```
🌞 Prouver que l'utilisateur imbob a un password défini
```sh
nathan@ServeurDebian:/$ sudo cat /etc/shadow | grep imbob
imbob:$y$j9T$Tn6IX93O6WF6Zrm0v0d.p.$ZYL9/nrpfV3pibPZZiaZ87AEXBENJOX3KA.gjXQ1WcD:20041:0:99999:7:::
```
🌞 Prouver que l'utilisateur imbob appartient au groupe stronk_admins
```sh
nathan@ServeurDebian:/$ sudo cat /etc/group | grep stronk_admins
stronk_admins:x:1002:imbob
```
🌞 Créer un deuxième utilisateur
```sh
nathan@ServeurDebian:/$ sudo adduser imnotbobsorry
Adding user `imnotbobsorry' ...
Adding new group `imnotbobsorry' (1004) ...
Adding new user `imnotbobsorry' (1004) with group `imnotbobsorry (1004)' ...
adduser: The home directory `/home/imnotbobsorry' already exists.  Not touching this directory.
New password:
Retype new password:
passwd: password updated successfully
```

nathan@ServeurDebian:/$ sudo usermod -aG stronk_admins imnotbobsorry


🌞 Modifier la configuration de sudo 
```sh

# User privilege specification
root    ALL=(ALL:ALL) ALL
imbob ALL=(ALL:ALL) ALL
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL
```

🌞 Créer le dossier /home/goodguy (avec une commande)
```sh
nathan@ServeurDebian:/$ sudo mkdir /home/goodguy
```
🌞 Changer le répertoire personnel de imbob
```sh
nathan@ServeurDebian:/$ sudo usermod -d /home/goodguy imbob
nathan@ServeurDebian:/$ sudo cat /etc/passwd | grep imbob
imbob:x:1003:1003:,,,:/home/goodguy:/bin/bash
```

🌞 Créer le dossier /home/badguy
```sh
nathan@ServeurDebian:/$ sudo mkdir /home/badguy
```
🌞 Changer le répertoire personnel de imnotbobsorry
```sh
nathan@ServeurDebian:/$ sudo usermod -d /home/badguy imnotbobsorry
nathan@ServeurDebian:/$ sudo cat /etc/passwd | grep imnotbobsorry
imnotbobsorry:x:1004:1004:,,,:/home/badguy:/bin/bash
```

🌞 Prouver que les permissions du dossier /home/gooduy sont incohérentes
```sh
imbob@ServeurDebian:~$ ls -ld /home/goodguy
drwxr-xr-x 2 root root 4096 Nov 14 00:53 /home/goodguy
imbob@ServeurDebian:~$ cat file1 file2 file3 > bigfile
-bash: bigfile: Permission denied
```

🌞 Modifier les permissions de /home/goodguy
```sh
nathan@ServeurDebian:/$ sudo chown -R imbob /home/goodguy
[sudo] password for nathan:
```
🌞 Modifier les permissions de /home/badguy
```sh
nathan@ServeurDebian:/$ sudo chown -R imnotbobsorry /home/badguy
```

🌞 Connectez-vous sur l'utilisateur imbob
```sh
nathan@ServeurDebian:/$ su - imbob
Password:
imbob@ServeurDebian:~$ pwd
/home/goodguy
imbob@ServeurDebian:~$ sudo echo meow
[sudo] password for imbob:
meow
```

🌞 Connectez-vous sur l'utilisateur imnotbobsorry
```sh
imbob@ServeurDebian:~$ su - imnotbobsorry
Password:
imnotbobsorry@ServeurDebian:~$ pwd
/home/badguy
imnotbobsorry@ServeurDebian:~$ sudo echo meow
[sudo] password for imnotbobsorry:
Sorry, user imnotbobsorry is not allowed to execute '/usr/bin/echo meow' as root on ServeurDebian.info.

imnotbobsorry@ServeurDebian:~$ sudo apt update
[sudo] password for imnotbobsorry:
Get:1 http://security.debian.org/debian-security bookworm-security InRelease [48.0 kB]
Hit:2 http://deb.debian.org/debian bookworm InRelease
Get:3 http://deb.debian.org/debian bookworm-updates InRelease [55.4 kB]
Get:4 https://packages.mozilla.org/apt mozilla InRelease [1,522 B]
Get:5 https://packages.mozilla.org/apt mozilla/main all Packages [5,605 kB]
Get:6 https://packages.mozilla.org/apt mozilla/main amd64 Packages [97.7 kB]
Reading package lists... Done
E: Release file for http://security.debian.org/debian-security/dists/bookworm-security/InRelease is not valid yet (invalid for another 3h 39min 49s). Updates for this repository will not be applied.
E: Release file for http://deb.debian.org/debian/dists/bookworm-updates/InRelease is not valid yet (invalid for another 1h 41min 18s). Updates for this repository will not be applied.
W: Target Packages (main/binary-amd64/Packages) is configured multiple times in /etc/apt/sources.list.d/mozilla.list:1 and /etc/apt/sources.list.d/mozilla.list:2
W: Target Packages (main/binary-all/Packages) is configured multiple times in /etc/apt/sources.list.d/mozilla.list:1 and /etc/apt/sources.list.d/mozilla.list:2
W: Target Translations (main/i18n/Translation-en_US) is configured multiple times in /etc/apt/sources.list.d/mozilla.list:1 and /etc/apt/sources.list.d/mozilla.list:2
W: Target Translations (main/i18n/Translation-en) is configured multiple times in /etc/apt/sources.list.d/mozilla.list:1 and /etc/apt/sources.list.d/mozilla.list:2
W: Target Packages (main/binary-amd64/Packages) is configured multiple times in /etc/apt/sources.list.d/mozilla.list:1 and /etc/apt/sources.list.d/mozilla.list:2
W: Target Packages (main/binary-all/Packages) is configured multiple times in /etc/apt/sources.list.d/mozilla.list:1 and /etc/apt/sources.list.d/mozilla.list:2
W: Target Translations (main/i18n/Translation-en_US) is configured multiple times in /etc/apt/sources.list.d/mozilla.list:1 and /etc/apt/sources.list.d/mozilla.list:2
W: Target Translations (main/i18n/Translation-en) is configured multiple times in /etc/apt/sources.list.d/mozilla.list:1 and /etc/apt/sources.list.d/mozilla.list:2
```

# II. Processes
🌞 Affichez les processus bash
```sh
nathan@ServeurDebian:/$ ps aux | grep bash
nathan      5538  0.0  0.1   7328  4068 pts/2    Ss   Nov13   0:01 -bash
nathan      7709  0.0  0.0   6332  2136 pts/2    S+   02:19   0:00 grep bash
   ```
 🌞 Affichez tous les processus lancés par votre utilisateur
 ```sh
nathan@ServeurDebian:/$ ps -u nathan
    PID TTY          TIME CMD
    948 ?        00:00:00 systemd
    949 ?        00:00:00 (sd-pam)
    964 ?        00:00:00 pulseaudio
    966 ?        00:00:00 gnome-keyring-d
    973 ?        00:00:11 dbus-daemon
    974 ?        00:00:00 xfce4-session
   1026 ?        00:00:00 ssh-agent
   1036 ?        00:00:00 at-spi-bus-laun
   1042 ?        00:00:00 dbus-daemon
   1051 ?        00:00:00 at-spi2-registr
   1062 ?        00:00:00 gpg-agent
   1064 ?        00:00:03 xfwm4
   1067 ?        00:00:00 gvfsd
   1095 ?        00:00:00 xfsettingsd
   1103 ?        00:00:06 xfce4-panel
   1107 ?        00:00:00 Thunar
   1112 ?        00:00:03 xfdesktop
   1115 ?        00:00:01 xfce4-power-man
   1116 ?        00:00:00 polkit-gnome-au
   1117 ?        00:00:00 nm-applet
   1119 ?        00:00:00 xiccd
   1121 ?        00:00:00 panel-6-systray
   1122 ?        00:00:01 light-locker
   1123 ?        00:00:50 panel-8-pulseau
   1126 ?        00:00:00 xfce4-notifyd
   1127 ?        00:00:11 panel-9-power-m
   1128 ?        00:00:00 applet.py
   1129 ?        00:00:00 panel-10-notifi
   1174 ?        00:00:00 dconf-service
   1180 ?        00:00:00 panel-14-action
   1218 ?        00:00:00 gvfs-udisks2-vo
   1279 ?        00:00:00 gvfsd-trash
   1299 ?        00:00:00 gvfsd-metadata
   5528 ?        00:00:07 sshd
   5538 pts/2    00:00:01 bash
   7754 ?        00:00:00 xfconfd
   7762 pts/2    00:00:00 ps
```
 🌞 Affichez le top 5 des processus qui utilisent le plus de RAM
   ```sh
nathan@ServeurDebian:/$ ps aux --sort=-rss | head -n 6
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
lightdm     6381  0.0  2.8 616204 115876 ?       Ssl  Nov13   0:06 /usr/sbin/lightdm-gtk-greeter
nathan      1064  0.0  2.7 1822356 108540 ?      Sl   Nov13   0:03 xfwm4
root         773  0.0  2.5 354860 102696 tty7    Ssl+ Nov13   0:14 /usr/lib/xorg/Xorg :0 -seat seat0 -auth /var/run/lightdm/root/:0 -nolisten tcp vt7 -novtswitch
root        6320  0.0  1.9 353376 76480 tty8     Ssl+ Nov13   0:01 /usr/lib/xorg/Xorg :1 -seat seat0 -auth /var/run/lightdm/root/:1 -nolisten tcp vt8 -novtswitch
nathan      1112  0.0  1.4 479980 58312 ?        Sl   Nov13   0:03 xfdesktop
```

🌞 Affichez le PID du processus du service SSH
```sh
nathan@ServeurDebian:/$ ps aux | grep sshd
root        5358  0.0  0.2  15432  9340 ?        Ss   Nov13   0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
root        5512  0.0  0.2  17336 10812 ?        Ss   Nov13   0:00 sshd: nathan [priv]
nathan      5528  0.0  0.1  17596  6592 ?        S    Nov13   0:07 sshd: nathan@pts/2
nathan      7727  0.0  0.0   6332  2140 pts/2    S+   02:27   0:00 grep sshd
```
🌞 Affichez le nom du processus avec l'identifiant le plus petit
```sh
nathan@ServeurDebian:/$ ps -aux --sort pid | head -2
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.3 169460 13844 ?        Ss   Nov13   0:11 /sbin/init
```
# 2. Parent, enfant, et meurtre
 🌞 Déterminer le PID de votre shell actuel
 ```sh
nathan@ServeurDebian:/$ echo $$
5538
 ```
   🌞 Déterminer le PPID de votre shell actuel
   ```sh
nathan@ServeurDebian:/$  ps -o ppid= -p 5538
   5528
   ```
   🌞 Déterminer le nom de ce processus
   ```sh
nathan@ServeurDebian:~$ ps -p $PPID -o comm=
sshd
```
🌞 Lancer un processus sleep 9999 en tâche de fond
```sh
nathan@ServeurDebian:~$ sleep 9999 &
[1] 8179
```
* déterminer avec une commande ps son PID et son PPID
```sh
nathan@ServeurDebian:~$ ps -o pid,ppid,cmd -C sleep
    PID    PPID CMD
     8179    8172 sleep 9999
 ```
 🌞 Fermez votre session SSH
   ```sh
 nathan@ServeurDebian:/$ exit
logout
Connection to 192.168.202.3 closed.
  ```
```sh
PS C:\Users\gusta> ssh nathan@192.168.202.3
nathan@192.168.202.3's password:
Linux ServeurDebian 6.1.0-26-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.112-1 (2024-09-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Nov 14 03:26:24 2024 from 192.168.202.1
```
* est-ce que le processus sleep lancé en tâche de fond s'exécute toujours ?
  | OUI |
sh```
nathan@ServeurDebian:~$ ps -ef | grep sleep
nathan      8179       1  0 03:28 ?        00:00:00 sleep 9999
nathan      8385    8377  0 03:35 pts/3    00:00:00 grep sleep
```

# III. Services
 ## 2. Analyser un service existant
 🌞 S'assurer que le service ssh est démarré
 ```sh
nathan@ServeurDebian:/$ systemctl status ssh
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; preset: enabled)
     Active: active (running) since Thu 2024-11-14 03:25:51 EST; 23min ago
       Docs: man:sshd(8)
             man:sshd_config(5)
    Process: 8136 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
   Main PID: 8138 (sshd)
      Tasks: 1 (limit: 4620)
     Memory: 3.1M
        CPU: 115ms
     CGroup: /system.slice/ssh.service
             └─8138 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"
```
🌞 Isolez la ligne qui indique le nom du programme lancé
```sh
nathan@ServeurDebian:/$ sudo systemctl status ssh | grep "Main PID"
   Main PID: 8138 (sshd)
   ```
   🌞 Déterminer le port sur lequel écoute le service SSH
   ```sh
   nathan@ServeurDebian:/$  sudo ss -lnpt | grep ssh
LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=8138,fd=3))
LISTEN 0      128             [::]:22           [::]:*    users:(("sshd",pid=8138,fd=4)
```

🌞 Consulter les logs du service SSH
* donnez une commande journalctl qui permet de consulter les logs du service SSH
```sh
nathan@ServeurDebian:/$ sudo journalctl -xe -u ssh
░░ Support: https://www.debian.org/support
░░
░░ A stop job for unit ssh.service has finished.
░░
░░ The job identifier is 8842 and the job result is done.
Nov 14 03:25:51 ServeurDebian systemd[1]: Starting ssh.service - OpenBSD Secure Shell server...
░░ Subject: A start job for unit ssh.service has begun execution
░░ Defined-By: systemd
░░ Support: https://www.debian.org/support
░░
░░ A start job for unit ssh.service has begun execution.
░░
░░ The job identifier is 8842.
Nov 14 03:25:51 ServeurDebian sshd[8138]: Server listening on 0.0.0.0 port 22.
Nov 14 03:25:51 ServeurDebian sshd[8138]: Server listening on :: port 22.
Nov 14 03:25:51 ServeurDebian systemd[1]: Started ssh.service - OpenBSD Secure Shell server.
░░ Subject: A start job for unit ssh.service has finished successfully
░░ Defined-By: systemd
░░ Support: https://www.debian.org/support
░░
░░ A start job for unit ssh.service has finished successfully.
░░
░░ The job identifier is 8842.
Nov 14 03:26:24 ServeurDebian sshd[8153]: Accepted password for nathan from 192.168.202.1 port 62088 ssh2
Nov 14 03:26:24 ServeurDebian sshd[8153]: pam_unix(sshd:session): session opened for user nathan(uid=1000) by (uid=0)
Nov 14 03:26:24 ServeurDebian sshd[8153]: pam_env(sshd:session): deprecated reading of user environment enabled
Nov 14 03:33:02 ServeurDebian sshd[8352]: Accepted password for nathan from 192.168.202.1 port 62363 ssh2
Nov 14 03:33:02 ServeurDebian sshd[8352]: pam_unix(sshd:session): session opened for user nathan(uid=1000) by (uid=0)
Nov 14 03:33:02 ServeurDebian sshd[8352]: pam_env(sshd:session): deprecated reading of user environment enabled
lines 460-488/488 (END)
```


## 3. Modification du service
### A. Configuration du service SSH
🌞 Identifier le fichier de configuration du serveur SSH
```sh
nathan@ServeurDebian:/$ ls -l /etc/ssh | grep config
-rw-r--r-- 1 root root   1650 Jun 22 15:38 ssh_config
drwxr-xr-x 2 root root   4096 Jun 22 15:38 ssh_config.d
-rw-r--r-- 1 root root   3230 Nov 14 03:25 sshd_config
drwxr-xr-x 2 root root   4096 Jun 22 15:38 sshd_config.d
-rw-r--r-- 1 root root   3221 Nov 13 12:47 sshd_config.save
```
🌞 Modifier le fichier de conf
```sh
nathan@ServeurDebian:~$ echo $RANDOM
14040
nathan@ServeurDebian:~$ cd /etc/ssh
nathan@ServeurDebian:/etc/ssh$ cat sshd_config | grep Port
Port 14040
#GatewayPorts no
```


🌞 Effectuer une connexion SSH sur le nouveau port
```sh
nathan@ServeurDebian:~$ exit
logout
Connection to 192.168.202.3 closed.
PS C:\Users\gusta> ssh nathan@192.168.202.3
ssh: connect to host 192.168.202.3 port 22: Connection refused
```
```sh
PS C:\Users\gusta> ssh nathan@192.168.202.3 -p 14040
nathan@192.168.202.3's password:
Linux ServeurDebian 6.1.0-26-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.112-1 (2024-09-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Nov 14 04:12:45 2024 from 192.168.202.1
```
✨ Bonus : affiner la conf du serveur SSH
```sh
X11Forwarding	no
MaxAuthTries	3
ClientAliveCountMax 2
AllowUsers nathan,root
```
## B. Le service en lui-même
🌞 Trouver le fichier ssh.service
```sh
nathan@ServeurDebian:~$ sudo find / -name ssh.service
/var/lib/systemd/deb-systemd-helper-enabled/multi-user.target.wants/ssh.service
/etc/systemd/system/multi-user.target.wants/ssh.service
/snap/core22/1663/usr/lib/systemd/system/multi-user.target.wants/ssh.service
/snap/core22/1663/usr/lib/systemd/system/ssh.service
/sys/fs/cgroup/system.slice/ssh.service
/usr/lib/systemd/system/ssh.service
/usr/share/doc/avahi-daemon/examples/ssh.service
nathan@ServeurDebian:~$
```
```sh
nathan@ServeurDebian:~$ sudo find / -name "ssh.service" | grep -x "/usr/lib/systemd/system/ssh.service"
/usr/lib/systemd/system/ssh.service
```

🌞 Déterminer quel est le programme lancé
```sh
nathan@ServeurDebian:/usr/lib/systemd/system$ cat ssh.service | grep ExecStart=
ExecStart=/usr/sbin/sshd -D $SSHD_OPTS
```

# 4. Créez votre propre service
🌞 Déterminer le dossier qui contient la commande python3
```sh
nathan@ServeurDebian:~$ which python3
/usr/bin/python3
```
🌞 Créez un fichier /etc/systemd/system/meow_web.service
```sh
nathan@ServeurDebian:~$ cd /etc/systemd/system
nathan@ServeurDebian:/etc/systemd/system$ sudo touch meow_web.service
nathan@ServeurDebian:/etc/systemd/system$ sudo nano meow_web.service
nathan@ServeurDebian:/etc/systemd/system$ sudo systemctl daemon-reload
nathan@ServeurDebian:/etc/systemd/system$  systemctl status meow_web.service
● meow_web.service - Super serveur web MEOW
     Loaded: loaded (/etc/systemd/system/meow_web.service; enabled; preset: enabled)
     Active: active (running) since Wed 2024-11-13 19:19:16 EST; 9h ago
   Main PID: 6631 (python3)
      Tasks: 1 (limit: 4620)
     Memory: 8.9M
        CPU: 6.893s
     CGroup: /system.slice/meow_web.service
             └─6631 /usr/bin/python3 -m http.server 8888
```

🌞 Déterminer le PID du processus Python en cours d'exécution
```sh
nathan@ServeurDebian:/$ ps -ef | grep http.server | grep root
root        6631       1  0 Nov13 ?        00:00:07 /usr/bin/python3 -m http.server 8888
```
🌞 Prouvez que le programme écoute derrière le port 8888
```sh
nathan@ServeurDebian:/$ sudo ss -lnpt | grep python
LISTEN 0      5            0.0.0.0:8888      0.0.0.0:*    users:(("python3",pid=6631,fd=3))
```
🌞 Faire en sorte que le service se lance automatiquement au démarrage de la machine
```sh
nathan@ServeurDebian:/$ sudo systemctl enable meow_web.service
Created symlink /etc/systemd/system/multi-user.target.wants/meow_web.service → /etc/systemd/system/meow_web.service.
```


















   
 
   
   



 



   




























   
