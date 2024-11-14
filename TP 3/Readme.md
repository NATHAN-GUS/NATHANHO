
# TP 3
#  I. Utilisateurs

#### 1. Liste des users
 🌞 Afficher la ligne du fichier qui concerne votre utilisateur
```sh
nathan@ServeurDebian:~$ cat /etc/passwd | grep nathan 
|nathan:x:1000:1000:nathan,,,:/home/nathan:/bin/bash| 
 ```
🌞 Afficher la ligne du fichier qui concerne votre utilisateur ET celle de root en même temps
 ```sh 
nathan@ServeurDebian:~$ cat /etc/passwd | grep -e nathan -e root
root:x:0:0:root:/root:/bin/bash
nathan:x:1000:1000:nathan,,,:/home/nathan:/bin/bash
```
🌞 Afficher la liste des groupes d'utilisateurs de la machine
* ptite recherche par vous-mêmes pour trouver quel fichier c'est !
```sh
nathan@ServeurDebian:~$ cat /etc/group
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
users:x:100:nathan,imbob,imnotbobsorry
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
stronk_admins:x:1002:imbob,imnotbobsorry
imbob:x:1003:
imnotbobsorry:x:1004:
```
🌞 Afficher la ligne du fichier qui concerne votre utilisateur ET celle de root en même temps
```su
nathan@ServeurDebian:~$ cat /etc/passwd | cut -d":" -f1,6 |  grep -E "nathan|root"
root:/root
nathan:/home/nathan
```

### 2. Hash des passwords
🌞 Afficher la ligne qui contient le hash du mot de passe de votre utilisateur
```sh
[nathan@ServeurDebian:]~$ sudo cat /etc/shadow | grep nathan
[sudo] password for nathan:
nathan:$y$j9T$QSQK4oL/cSJlyQ0PYf/Yc.$FaTPvwuVDv0FjiSiunMX19fcHLIFgzMdGcwVvpV6ycB:20036:0:99999:7:::
```
# 3. Sudo
## A. Intro


## B. Practice
🌞 Créer un groupe d'utilisateurs
```sh
nathan@ServeurDebian:~$ sudo groupadd stronk_admins
```
🌞 Créer un utilisateur
```sh
nathan@ServeurDebian:~$ sudo useradd -m imbob
nathan@ServeurDebian:~$ sudo passwd imbob
New password:
Retype new password:
passwd: password updated successfully
```
🌞 Prouver que l'utilisateur imbob est créé
```sh
nathan@ServeurDebian:~$ cat /etc/passwd | grep imbob
imbob:x:1002:1002:,,,:/home/imbob:/bin/bash
```
🌞 Prouver que l'utilisateur imbob a un password défini
```sh
nathan@ServeurDebian:~$  sudo cat /etc/shadow | grep imbob
imbob:$y$j9T$KdTxmpPqDDUOA.tD9PW5b1$QQBF7ImEx.z1i13rqxvVBqvtouVIv4H071cA67oYSO.:20040:0:99999:7:::
```
🌞 Prouver que l'utilisateur imbob appartient au groupe stronk_admins
```sh
nathan@ServeurDebian:~$  cat /etc/group | grep stronk_admins
stronk_admins:x:1005:imbob
```
🌞 Créer un deuxième utilisateur
```sh
nathan@ServeurDebian:~$ sudo useradd imnotbobsorry
nathan@ServeurDebian:~$ sudo passwd imnotbobsorry
New password:
Retype new password:
Sorry, passwords do not match.
passwd: Authentication token manipulation error
passwd: password unchanged
nathan@ServeurDebian:~$ sudo passwd imnotbobsorry
New password:
Retype new password:
passwd: password updated successfully
```

🌞 Modifier la configuration de sudo 
```sh
sudo visudo
%stronk_admins ALL=(ALL) NOPASSWD: /usr/bin/apt, /usr/bin/apt-get
```

🌞 Créer le dossier /home/goodguy (avec une commande)
```sh
nathan@ServeurDebian:~$ sudo mkdir /home/goodguy
```
🌞 Changer le répertoire personnel de imbob
```sh
nathan@ServeurDebian:~$ sudo usermod -d /home/goodguy imbob
```
* prouvez que le changement est effectif en affichant le contenu du fichier passwd
 ```sh
nathan@ServeurDebian:~$ cat /etc/passwd | grep imbob
imbob:x:1002:1002:,,,:/home/goodguy:/bin/bash
```
🌞 Créer le dossier /home/badguy
```sh 
nathan@ServeurDebian:~$ sudo mkdir /home/badguy
```
🌞 Changer le répertoire personnel de imnotbobsorry
```sh
nathan@ServeurDebian:~$  sudo usermod -d /home/badguy imnotbobsorry
```
* prouvez que le changement est effectif en affichant le contenu du fichier passwd
```sh
nathan@ServeurDebian:~$ cat /etc/passwd | grep imnotbobsorry
imnotbobsorry:x:1003:1003:,,,:/home/badguy:/bin/bash
```
🌞 Prouver que les permissions du dossier /home/gooduy sont incohérentes
```sh
nathan@ServeurDebian:~$ ls -ld /home/goodguy
drwxr-xr-x 2 root root 4096 Nov 13 06:11 /home/goodguy
```

🌞 Modifier les permissions de /home/goodguy
```sh

```
🌞 Modifier les permissions de /home/badguy
```sh




🌞 Connectez-vous sur l'utilisateur imbob
```sh
nathan@ServeurDebian:~$ su - imbob
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
Hit:1 http://deb.debian.org/debian bookworm InRelease
Hit:2 http://security.debian.org/debian-security bookworm-security InRelease
Hit:3 http://deb.debian.org/debian bookworm-updates InRelease
Hit:4 https://packages.mozilla.org/apt mozilla InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
56 packages can be upgraded. Run 'apt list --upgradable' to see them.
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
nathan@ServeurDebian:~$ ps aux | grep bash | grep -v grep
nathan      1440  0.0  0.0   7196  3960 pts/0    Ss+  05:08   0:00 bash
nathan      1477  0.0  0.1   7324  4096 pts/1    Ss   05:11   0:01 -bash
imbob       2500  0.0  0.0   7196  3984 pts/1    S    06:55   0:00 -bash
imnotbo+    2516  0.0  0.0   7196  3888 pts/1    S+   06:58   0:00 -bash
nathan      3097  0.0  0.1   7196  4052 pts/2    Ss   08:35   0:00 -bash
   ```
 🌞 Affichez tous les processus lancés par votre utilisateur
 ```sh
 nathan@ServeurDebian:~$ ps -fu nathan
UID          PID    PPID  C STIME TTY          TIME CMD
nathan       948       1  0 05:04 ?        00:00:00 /lib/systemd/systemd --user
nathan       949     948  0 05:04 ?        00:00:00 (sd-pam)
nathan       964     948  0 05:04 ?        00:00:00 /usr/bin/pulseaudio --daemonize=no --log-target=journal
nathan       966     948  0 05:04 ?        00:00:00 /usr/bin/gnome-keyring-daemon --foreground --components=pkcs11,secre
nathan       973     948  0 05:04 ?        00:00:04 /usr/bin/dbus-daemon --session --address=systemd: --nofork --nopidfi
nathan       974     942  0 05:04 ?        00:00:00 xfce4-session
nathan      1026     974  0 05:04 ?        00:00:00 /usr/bin/ssh-agent x-session-manager
nathan      1036     948  0 05:04 ?        00:00:00 /usr/libexec/at-spi-bus-launcher
nathan      1042    1036  0 05:04 ?        00:00:00 /usr/bin/dbus-daemon --config-file=/usr/share/defaults/at-spi2/acces
nathan      1051     948  0 05:04 ?        00:00:00 /usr/libexec/at-spi2-registryd --use-gnome-session
nathan      1062     948  0 05:04 ?        00:00:00 /usr/bin/gpg-agent --supervised
nathan      1064     974  0 05:04 ?        00:00:01 xfwm4
nathan      1067     948  0 05:04 ?        00:00:00 /usr/libexec/gvfsd
nathan      1095     974  0 05:04 ?        00:00:00 xfsettingsd
nathan      1103     974  0 05:04 ?        00:00:03 xfce4-panel
nathan      1107     974  0 05:04 ?        00:00:00 Thunar --daemon
nathan      1112     974  0 05:04 ?        00:00:01 xfdesktop
nathan      1115     974  0 05:04 ?        00:00:00 xfce4-power-manager
nathan      1116     974  0 05:04 ?        00:00:00 /usr/lib/policykit-1-gnome/polkit-gnome-authentication-agent-1
nathan      1117     974  0 05:04 ?        00:00:00 nm-applet
nathan      1119     974  0 05:04 ?        00:00:00 xiccd
nathan      1121    1103  0 05:04 ?        00:00:00 /usr/lib/x86_64-linux-gnu/xfce4/panel/wrapper-2.0 /usr/lib/x86_64-li
nathan      1122     974  0 05:04 ?        00:00:00 light-locker
nathan      1123    1103  0 05:04 ?        00:00:15 /usr/lib/x86_64-linux-gnu/xfce4/panel/wrapper-2.0 /usr/lib/x86_64-li
nathan      1126     974  0 05:04 ?        00:00:00 /usr/lib/x86_64-linux-gnu/xfce4/notifyd/xfce4-notifyd
nathan      1127    1103  0 05:04 ?        00:00:03 /usr/lib/x86_64-linux-gnu/xfce4/panel/wrapper-2.0 /usr/lib/x86_64-li
nathan      1128     974  0 05:04 ?        00:00:00 /usr/bin/python3 /usr/share/system-config-printer/applet.py
nathan      1129    1103  0 05:04 ?        00:00:00 /usr/lib/x86_64-linux-gnu/xfce4/panel/wrapper-2.0 /usr/lib/x86_64-li
nathan      1174     948  0 05:04 ?        00:00:00 /usr/libexec/dconf-service
nathan      1180    1103  0 05:04 ?        00:00:00 /usr/lib/x86_64-linux-gnu/xfce4/panel/wrapper-2.0 /usr/lib/x86_64-li
nathan      1218     948  0 05:04 ?        00:00:00 /usr/libexec/gvfs-udisks2-volume-monitor
nathan      1279    1067  0 05:04 ?        00:00:00 /usr/libexec/gvfsd-trash --spawner :1.14 /org/gtk/gvfs/exec_spaw/0
nathan      1299     948  0 05:04 ?        00:00:00 /usr/libexec/gvfsd-metadata
nathan      1415       1  0 05:08 ?        00:00:01 xfce4-terminal
nathan      1440    1415  0 05:08 pts/0    00:00:00 bash
nathan      3306    3292  0 09:05 ?        00:00:00 sshd: nathan@pts/3
nathan      3311    3306  0 09:05 pts/3    00:00:00 -bash
nathan      3712     948  0 11:49 ?        00:00:00 /usr/lib/x86_64-linux-gnu/xfce4/xfconf/xfconfd
nathan      3737    3311  0 11:54 pts/3    00:00:00 ps -fu nathan
```
 🌞 Affichez le top 5 des processus qui utilisent le plus de RAM
   ```sh
nathan@ServeurDebian:~$ 7
COMMAND %MEM%
/usr/sbin/lightdm-gtk-greeter 2.8%
xfwm4 2.6%
/usr/lib/xorg/Xorg 2.5%
/usr/lib/xorg/Xorg 1.9%
xfdesktop 1.5%
```
🌞 Affichez le PID du processus du service SSH
```sh
```
🌞 Affichez le nom du processus avec l'identifiant le plus petit
```sh
nathan@ServeurDebian:~$  ps -ef --sort pid | head -2
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 05:03 ?        00:00:09 /sbin/init
```
# 2. Parent, enfant, et meurtre
 🌞 Déterminer le PID de votre shell actuel
 ```sh
 nathan@ServeurDebian:~$  echo $$
3850
 ```
   🌞 Déterminer le PPID de votre shell actuel
   ```sh
nathan@ServeurDebian:~$ ps -o ppid= -p 3850
   3845
   ```
   🌞 Déterminer le nom de ce processus
   ```sh
nathan@ServeurDebian:~$ ps -p $PPID -o comm=
sshd
```
🌞 Lancer un processus sleep 9999 en tâche de fond
```sh
nathan@ServeurDebian:~$ sleep 9999 &
[1] 4005
```
* déterminer avec une commande ps son PID et son PPID
```sh
nathan@ServeurDebian:~$ ps -o pid,ppid,comm -C sleep --sort=start_time | tail -n 1
   4005    3850 sleep
 ```
 🌞 Fermez votre session SSH
   ```sh
   nathan@ServeurDebian:~$ exit
logout
Connection to 192.168.202.3 closed.
  ```
* puis reconnecte-toi avec une nouvelle connexion SSH
```sh
PS C:\Users\gusta> ssh nathan@192.168.202.3
nathan@192.168.202.3's password:
Linux ServeurDebian 6.1.0-26-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.112-1 (2024-09-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Nov 13 11:58:59 2024 from 192.168.202.1 
nathan@ServeurDebian:~$ ps -ef | grep sleep
nathan      4005       1  0 12:23 ?        00:00:00 sleep 9999
nathan      4097    4077  0 12:27 pts/2    00:00:00 grep sleep
```
 # III. Services
 ## 2. Analyser un service existant
 🌞 S'assurer que le service ssh est démarré
 ```sh
nathan@ServeurDebian:~$ systemctl status ssh
● ssh.service - OpenBSD Secure Shell server
     Loaded: loaded (/lib/systemd/system/ssh.service; enabled; preset: enabled)
     Active: active (running) since Wed 2024-11-13 05:04:12 EST; 7h ago
       Docs: man:sshd(8)
             man:sshd_config(5)
    Process: 753 ExecStartPre=/usr/sbin/sshd -t (code=exited, status=0/SUCCESS)
   Main PID: 763 (sshd)
      Tasks: 1 (limit: 4620)
     Memory: 5.7M
        CPU: 1.509s
     CGroup: /system.slice/ssh.service
             └─763 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Warning: some journal files were not opened due to insufficient permissions.
```
🌞 Isolez la ligne qui indique le nom du programme lancé
```sh
nathan@ServeurDebian:~$ sudo systemctl status ssh | grep "Main PID"
   Main PID: 763 (sshd)
   ```
   🌞 Déterminer le port sur lequel écoute le service SSH
   ```sh
   nathan@ServeurDebian:~$ sudo ss -lnpt | grep ssh
[sudo] password for nathan:
LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=763,fd=3))
LISTEN 0      128             [::]:22           [::]:*    users:(("sshd",pid=763,fd=4))
```
🌞 Consulter les logs du service SSH
* donnez une commande journalctl qui permet de consulter les logs du service SSH
```sh
nathan@ServeurDebian:~$ sudo journalctl -xe -u ssh
~
~
Nov 13 05:04:08 ServeurDebian systemd[1]: Starting ssh.service - OpenBSD Secure Shell server...
░░ Subject: A start job for unit ssh.service has begun execution
░░ Defined-By: systemd
░░ Support: https://www.debian.org/support
░░
░░ A start job for unit ssh.service has begun execution.
░░
░░ The job identifier is 134.
Nov 13 05:04:12 ServeurDebian sshd[763]: Server listening on 0.0.0.0 port 22.
Nov 13 05:04:12 ServeurDebian sshd[763]: Server listening on :: port 22.
Nov 13 05:04:12 ServeurDebian systemd[1]: Started ssh.service - OpenBSD Secure Shell server.
░░ Subject: A start job for unit ssh.service has finished successfully
░░ Defined-By: systemd
░░ Support: https://www.debian.org/support
░░
░░ A start job for unit ssh.service has finished successfully.
░░
░░ The job identifier is 134.
Nov 13 05:11:08 ServeurDebian sshd[1458]: Accepted password for nathan from 192.168.202.1 port 64675 ssh2
Nov 13 05:11:08 ServeurDebian sshd[1458]: pam_unix(sshd:session): session opened for user nathan(uid=1000) by (uid=0)
Nov 13 05:11:08 ServeurDebian sshd[1458]: pam_env(sshd:session): deprecated reading of user environment enabled
Nov 13 08:35:23 ServeurDebian sshd[3078]: Accepted password for nathan from 192.168.202.1 port 50867 ssh2
Nov 13 08:35:23 ServeurDebian sshd[3078]: pam_unix(sshd:session): session opened for user nathan(uid=1000) by (uid=0)
Nov 13 08:35:23 ServeurDebian sshd[3078]: pam_env(sshd:session): deprecated reading of user environment enabled
Nov 13 09:05:28 ServeurDebian sshd[3292]: Accepted password for nathan from 192.168.202.1 port 51659 ssh2
Nov 13 09:05:28 ServeurDebian sshd[3292]: pam_unix(sshd:session): session opened for user nathan(uid=1000) by (uid=0)
Nov 13 09:05:28 ServeurDebian sshd[3292]: pam_env(sshd:session): deprecated reading of user environment enabled
Nov 13 11:58:59 ServeurDebian sshd[3831]: Accepted password for nathan from 192.168.202.1 port 61840 ssh2
Nov 13 11:58:59 ServeurDebian sshd[3831]: pam_unix(sshd:session): session opened for user nathan(uid=1000) by (uid=0)
Nov 13 11:58:59 ServeurDebian sshd[3831]: pam_env(sshd:session): deprecated reading of user environment enabled
Nov 13 12:26:37 ServeurDebian sshd[4067]: Accepted password for nathan from 192.168.202.1 port 62339 ssh2
Nov 13 12:26:37 ServeurDebian sshd[4067]: pam_unix(sshd:session): session opened for user nathan(uid=1000) by (uid=0)
Nov 13 12:26:37 ServeurDebian sshd[4067]: pam_env(sshd:session): deprecated reading of user environment enabled
```

## 3. Modification du service
### A. Configuration du service SSH
🌞 Identifier le fichier de configuration du serveur SSH
```sh
nathan@ServeurDebian:~$ ls -l /etc/ssh | grep config
-rw-r--r-- 1 root root   1650 Jun 22 15:38 ssh_config
drwxr-xr-x 2 root root   4096 Jun 22 15:38 ssh_config.d
-rw-r--r-- 1 root root   3223 Nov 13 04:31 sshd_config
drwxr-xr-x 2 root root   4096 Jun 22 15:38 sshd_config.d
```
🌞 Modifier le fichier de conf
```sh
nathn@ServeurDebian:~$ echo $RANDOM
1624
nathan@ServeurDebian:~$ sudo cat /etc/ssh/sshd_config | grep Port
Port 1624
#GatewayPorts no
```

🌞 Effectuer une connexion SSH sur le nouveau port
```sh
PS C:\Users\gusta> ssh nathan@192.168.202.3
ssh: connect to host 192.168.202.3 port 22: Connection refused
PS C:\Users\gusta>  ssh nathan@192.168.202.3 -p 1624
nathan@192.168.202.3's password:
Linux ServeurDebian 6.1.0-26-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.112-1 (2024-09-30) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Nov 13 13:42:00 2024 from 192.168.202.3
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
nathan@ServeurDebian:~$ sudo find / -name "ssh.service"
[sudo] password for nathan:
/var/lib/systemd/deb-systemd-helper-enabled/multi-user.target.wants/ssh.service
/etc/systemd/system/multi-user.target.wants/ssh.service
/snap/core22/1663/usr/lib/systemd/system/multi-user.target.wants/ssh.service
/snap/core22/1663/usr/lib/systemd/system/ssh.service
/sys/fs/cgroup/system.slice/ssh.service
/usr/lib/systemd/system/ssh.service
/usr/share/doc/avahi-daemon/examples/ssh.service

```sh
nathan@ServeurDebian:~$ sudo find / -name "ssh.service" | grep -x "/usr/lib/systemd/system/ssh.service"
/usr/lib/systemd/system/ssh.service
```

🌞 Déterminer quel est le programme lancé
```sh 



# 4. Créez votre propre service
🌞 Déterminer le dossier qui contient la commande python3
```sh
nathan@ServeurDebian:~$ which python3
/usr/bin/python3
```
🌞 Créez un fichier /etc/systemd/system/meow_web.service
```sh
nathan@ServeurDebian:/etc/systemd/system$ sudo touch meow_web.service
nathan@ServeurDebian:/etc/systemd/system$ sudo nano meow_web.service
nathan@ServeurDebian:/etc/systemd/system$ sudo systemctl daemon-reload
nathan@ServeurDebian:/etc/systemd/system$ sudo systemctl start meow_web
nathan@ServeurDebian:/etc/systemd/system$ systemctl status meow_web.service
● meow_web.service - Super serveur web MEOW
     Loaded: loaded (/etc/systemd/system/meow_web.service; disabled; preset: enabled)
     Active: active (running) since Wed 2024-11-13 19:19:16 EST; 11s ago
   Main PID: 6631 (python3)
      Tasks: 1 (limit: 4620)
     Memory: 8.9M
        CPU: 47ms
     CGroup: /system.slice/meow_web.service
             └─6631 /usr/bin/python3 -m http.server 8888
```

🌞 Déterminer le PID du processus Python en cours d'exécution
`` sh
nathan@ServeurDebian:/$ ps -ef | grep http.server | grep root
root        6631       1  0 19:19 ?        00:00:00 /usr/bin/python3 -m http.server 8888
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


















   
 
   
   



 



   




























   
