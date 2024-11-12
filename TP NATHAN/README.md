# II. Files and users
## 1. A Find me
🌞 Trouver le chemin vers le répertoire personnel de votre utilisateur
```sh
 pwd
/home/nathan
 ```
🌞 Vérifier les permissions du répertoire personnel de votre utilisateurs
 ```sh 
 ls -l /home/nathan
total 18056
drwxr-xr-x 2 nathan nathan     4096 Nov  9 14:48 Desktop
drwxr-xr-x 2 nathan nathan     4096 Nov  9 14:48 Documents
drwxr-xr-x 2 nathan nathan     4096 Nov  9 14:48 Downloads
drwxr-xr-x 2 nathan nathan     4096 Nov  9 17:37 gameshell
-rwxr-xr-x 1 nathan nathan   219773 Nov  9 13:51 gameshell-save.sh
-rw-r--r-- 1 nathan nathan   210298 Nov  9 13:40 gameshell.sh
-rw-r--r-- 1 nathan
drwxr-xr-x 2 nathan nathan     4096 Nov  9 14:48 Pictures
drwxr-xr-x 2 nathan nathan     4096 Nov  9 14:48 Public
drwxr-xr-x 2 nathan nathan     4096 Nov  9 14:48 Templates
drwxr-xr-x 2 nathan nathan     4096 Nov  9 14:48 Videos
```

 🌞 Trouver le chemin du fichier de configuration du serveur SSH
```sh
Last login: Mon Nov 11 17:47:47 2024 from 192.168.202.1
nathan@ServeurDebian:~$ find / -name sshd_config 2> /dev/null
/etc/ssh/sshd_config
/snap/core22/1663/etc/ssh/sshd_config
/snap/core22/1663/usr/share/openssh/sshd_config
/usr/share/openssh/sshd_config
```


## 2. Users

### A. Nouveau user
 🌞 Créer un nouvel utilisateur

```sh
root@ServeurDebian:~# useradd -m -d /home/papier_alu/ marmotte
root@ServeurDebian:~# passwd marmotte
New password:
Retype new password:
passwd: password updated successfully
```
### B. Infos enregistrées par le système
 🌞 Prouver que cet utilisateur a été créé
```sh
 root@ServeurDebian:~# cat /etc/passwd | grep marmotte
marmotte:x:1001:1001::/home/papier_alu/:/bin/sh
```
 🌞 Déterminer le hash du password de l'utilisateur marmotte
```sh
root@ServeurDebian:~# cat /etc/shadow | grep marmotte
marmotte:$y$j9T$rNeIV1dLfSitBbgU3vhQS/$Y19D4LxtUeZIGV3yrvv5CfnqIAyqXbpXO.7.b2ADPO7:20038:0:99999:7:::
```
### D. Connexion sur le nouvel utilisateur
 🌞 Tapez une commande pour vous déconnecter : fermer votre session utilisateur
```sh
nathan@ServeurDebian:~$ logout
Connection to 192.168.202.3 closed.
```
🌞 Assurez-vous que vous pouvez vous connecter en tant que l'utilisateur marmotte
```sh
nathan@```ServeurDebian:~$ su marmotte
Password:
$ ls
ls: cannot open directory '.': Permission denied
$
```
# III. Programmes et paquets
## 1. Programmes et processus
### A. Run then kill
 🌞 Lancer un processus sleep

```sh
nathan@ServeurDebian:~$  sleep 1000
```
* PID DU PROCESSUS SLEEP
```sh
Last login: Mon Nov 11 08:40:35 2024 from 192.168.202.1
nathan@ServeurDebian:~$ ps -a
    PID TTY          TIME CMD
   2847 pts/0    00:00:00 su
   2902 pts/0    00:00:00 sh
   3600 pts/3    00:00:00 sleep
   3659 pts/1    00:00:00 ps
nathan@ServeurDebian:~$ ps -a | grep sleep
   3600 pts/3    00:00:00 sleep
   ```

 🌞 Terminez le processus sleep depuis le deuxième terminal
```sh
nathan@ServeurDebian:~$ kill sleep 3665
-bash: kill: sleep: arguments must be process or job IDs
nathan@ServeurDebian:~$ sleep 1000
Terminated
```

### B. Tâche de fond
 🌞 Lancer un nouveau processus sleep, mais en tâche de fond

```sh
nathan@ServeurDebian:~$ sleep 1000 &
```
 🌞 Visualisez la commande en tâche de fond
```sh
nathan@ServeurDebian:~$ ps -ef | grep sleep
```
### C. Find paths
🌞 Trouver le chemin où est stocké le programme sleep
```sh
nathan@ServeurDebian:~$ find / -name sleep 2> /dev/null
/snap/core22/1663/usr/bin/sleep
/usr/lib/klibc/bin/sleep
/usr/bin/sleep
```
 🌞 Tant qu'on est à chercher des chemins : trouver les chemins vers tous les fichiers qui s'appellent .bashrc

```sh
nathan@ServeurDebian:~$ find / -name .bashrc 2> /dev/null
/mnt/testuser/.bashrc
/etc/skel/.bashrc
/home/papier_alu/.bashrc
/home/nathan/.bashrc
/snap/core22/1663/etc/skel/.bashrc
```
### D. La variable PATH

🌞 Vérifier que 
* les commandes sleep, ssh, et ping sont bien des programmes stockés dans l'un des dossiers listés dans votre PATH
```sh 
nathan@ServeurDebian:~$ which sleep
/usr/bin/sleep
```
## 2.Paquets
🌞 Installer le paquet firefox
```sh
root@ServeurDebian:~# sudo install -d -m 0755 /etc/apt/keyrings
root@ServeurDebian:~# wget -q https://packages.mozilla.org/apt/repo-signing-key.gpg -O- | sudo tee /etc/apt/keyrings/packages.mozilla.org.asc > /dev/null
root@ServeurDebian:~# gpg -n -q --import --import-options import-show /etc/apt/keyrings/packages.mozilla.org.asc | awk '/pub/{getline; gsub(/^ +| +$/,""); if($0 == "35BAA0B33E9EB396F59CA838C0BA5CE6DC6315A3") print "\nThe key fingerprint matches ("$0").\n"; else print "\nVerification failed: the fingerprint ("$0") does not match the expected one.\n"}'

The key fingerprint matches (35BAA0B33E9EB396F59CA838C0BA5CE6DC6315A3).

root@ServeurDebian:~# echo "deb [signed-by=/etc/apt/keyrings/packages.mozilla.org.asc] https://packages.mozilla.org/apt mozilla main" | sudo tee -a /etc/apt/sources.list.d/mozilla.list > /dev/null
root@ServeurDebian:~# echo '
Package: *
Pin: origin packages.mozilla.org
Pin-Priority: 1000
' | sudo tee /etc/apt/preferences.d/mozilla

Package: *
Pin: origin packages.mozilla.org
Pin-Priority: 1000

root@ServeurDebian:~# sudo apt-get update && sudo apt-get install firefox
Get:1 https://packages.mozilla.org/apt mozilla InRelease [1,522 B]
Get:2 https://packages.mozilla.org/apt mozilla/main amd64 Packages [95.6 kB]
Get:3 https://packages.mozilla.org/apt mozilla/main all Packages [5,503 kB]
Hit:4 http://deb.debian.org/debian bookworm InRelease
Get:5 http://security.debian.org/debian-security bookworm-security InRelease [48.0 kB]
Get:6 http://deb.debian.org/debian bookworm-updates InRelease [55.4 kB]
Get:7 http://security.debian.org/debian-security bookworm-security/main Sources [125 kB]
Get:8 http://security.debian.org/debian-security bookworm-security/main amd64 Packages [204 kB]
Get:9 http://security.debian.org/debian-security bookworm-security/main Translation-en [125 kB]
Fetched 6,157 kB in 11s (572 kB/s)
Reading package lists... Done
W: Target Packages (main/binary-amd64/Packages) is configured multiple times in /etc/apt/sources.list.d/mozilla.list:1 and /etc/apt/sources.list.d/mozilla.list:2
W: Target Packages (main/binary-all/Packages) is configured multiple times in /etc/apt/sources.list.d/mozilla.list:1 and /etc/apt/sources.list.d/mozilla.list:2
W: Target Translations (main/i18n/Translation-en_US) is configured multiple times in /etc/apt/sources.list.d/mozilla.list:1 and /etc/apt/sources.list.d/mozilla.list:2
W: Target Translations (main/i18n/Translation-en) is configured multiple times in /etc/apt/sources.list.d/mozilla.list:1 and /etc/apt/sources.list.d/mozilla.list:2
W: Target Packages (main/binary-amd64/Packages) is configured multiple times in /etc/apt/sources.list.d/mozilla.list:1 and /etc/apt/sources.list.d/mozilla.list:2
W: Target Packages (main/binary-all/Packages) is configured multiple times in /etc/apt/sources.list.d/mozilla.list:1 and /etc/apt/sources.list.d/mozilla.list:2
W: Target Translations (main/i18n/Translation-en_US) is configured multiple times in /etc/apt/sources.list.d/mozilla.list:1 and /etc/apt/sources.list.d/mozilla.list:2
W: Target Translations (main/i18n/Translation-en) is configured multiple times in /etc/apt/sources.list.d/mozilla.list:1 and /etc/apt/sources.list.d/mozilla.list:2
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
firefox is already the newest version (132.0.1~build2).
The following package was automatically installed and is no longer required:
  libevent-2.1-7
Use 'sudo apt autoremove' to remove it.
0 upgraded, 0 newly installed, 0 to remove and 55 not upgraded.
```

 🌞 Utiliser une commande pour lancer Firefox
* comme d'hab, une commande, c'est un programme hein
```sh
root@ServeurDebian:~# firefox
```
* determiner le chemin vers le programme firefox
```sh
[root@ServeurDebian]:~# which firefox
/usr/bin/firefox
```
 🌞 Mais aussi déterminer...
 
  Sur quelle(s) URL(s) je me connectes pour télécharger des paquets ?
  ```sh
root@ServeurDebian:~# cat /etc/apt/sources.list
#deb cdrom:[Debian GNU/Linux 12.7.0 _Bookworm_ - Official amd64 NETINST with firmware 20240831-10:38]/ bookworm contrib main non-free-firmware

deb http://deb.debian.org/debian/ bookworm main non-free-firmware
deb-src http://deb.debian.org/debian/ bookworm main non-free-firmware

deb http://security.debian.org/debian-security bookworm-security main non-free-firmware
deb-src http://security.debian.org/debian-security bookworm-security main non-free-firmware

# bookworm-updates, to get updates before a point release is made;
# see https://www.debian.org/doc/manuals/debian-reference/ch02.en.html#_updates_and_backports
deb http://deb.debian.org/debian/ bookworm-updates main non-free-firmware
deb-src http://deb.debian.org/debian/ bookworm-updates main non-free-firmware

# This system was installed using small removable media
# (e.g. netinst, live or single CD). The matching "deb cdrom"
# entries were disabled at the end of the installation process.
# For information about how to configure apt package sources,
# see the sources.list(5) manual.
```

# IV. Poupée russe
🌞 Récupérer le fichier meow
```sh
[root@ServeurDebian]:~# file /root/meow
/root/meow: Zip archive data, at least v2.0 to extract, compression method=deflate
[root@ServeurDebian]:~# rm meow
[root@ServeurDebian]:~# wget https://gitlab.com/it4lik/b1-os/-/raw/main/tp/2/meow
--2024-11-11 14:53:32--  https://gitlab.com/it4lik/b1-os/-/raw/main/tp/2/meow
Resolving gitlab.com (gitlab.com)... 172.65.251.78, 2606:4700:90:0:f22e:fbec:5bed:a9b9
Connecting to gitlab.com (gitlab.com)|172.65.251.78|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 18016947 (17M) [application/octet-stream]
Saving to: ‘meow’

meow               100%[================>]  17.18M  3.55MB/s    in 5.7s

2024-11-11 14:53:38 (3.00 MB/s) - ‘meow’ saved [18016947/18016947]
```
🌞 Trouver le dossier dawa/

* Determiner le type du ficher 
```sh
root@ServeurDebian:~# file /root/meow
/root/meow: Zip archive data, at least v2.0 to extract, compression method=deflate
```
* renommez-le fichier correctement
```sh
nathan@ServeurDebian:~$ mv meow meow.zip
```
* extraire l'archive avec une commande
  ```sh
  root@ServeurDebian:~# unzip meow.zip
  Archive:  meow.zip
  inflating: meow
  
* répétez ces opérations jusqu'à trouver le dossier dawa/
```sh
root@ServeurDebian:~# mv meow meow.xz
root@ServeurDebian:~# xz -dk meow.xz
root@ServeurDebian:~# mv meow meow.bz2
root@ServeurDebian:~# bzip2 -dk meow.bz2
root@ServeurDebian:~# mv meow meow.rar
root@ServeurDebian:~# unrar-free meow.rar

unrar-free 0.1.3  Copyright (C) 2004  Ben Asselstine, Jeroen Dekkers


Extracting from /root/meow.rar

Extracting  meow                                                      OK    
All OK
root@ServeurDebian:~# mv meow meow.gz
root@ServeurDebian:~# gzip -dk meow.gz
root@ServeurDebian:~# mv meow meow.tar
root@ServeurDebian:~# tar xvf archive.tar
```

```sh
dawa/folder16/16/file25
dawa/folder16/16/file6
dawa/folder16/16/file13
dawa/folder16/16/file31
dawa/folder16/16/file3
dawa/folder16/16/file39
dawa/folder16/16/file30
dawa/folder16/16/file48
dawa/folder16/16/file16
dawa/folder16/16/file32
dawa/folder16/16/file38
dawa/folder16/45/
dawa/folder16/45/file34
dawa/folder16/45/file18
dawa/folder16/45/file33
dawa/folder16/45/file7
dawa/folder16/45/file45
dawa/folder16/45/file27
dawa/folder16/45/file22
dawa/folder16/45/file35
dawa/folder16/45/file19
dawa/folder16/45/file44
dawa/folder16/45/file26
dawa/folder16/45/file21
dawa/folder16/45/file50
dawa/folder16/45/file23
dawa/folder16/45/file10
dawa/folder16/45/file37
dawa/folder16/45/file24
```
🌞 Dans le dossier dawa/, déterminer le chemin vers
* le seul fichier de 15Mo
```sh
  root@ServeurDebian:~# find dawa -size 15M
dawa/folder31/19/file39
```
* le seul ficher que ne contient que des 7
  ```sh
  
  
* le seul fichier qui est nommé cookie
```sh
root@ServeurDebian:~# find dawa -name cookie
dawa/folder14/25/cookie
```
* le seul fichier caché (un fichier caché c'est juste un fichier dont le nom commence par un .)
  ```sh
  root@ServeurDebian:~# find dawa -type f -iname ".*"
  dawa/folder32/14/.hidden_file
  ```
* le seul fichier qui date de 2014
```sh
root@ServeurDebian:~# find dawa -mtime +2014
dawa/folder36/40/file43
```
* le seul fichier qui a 5 dossiers-parents
  ```sh
  root@ServeurDebian:~# find dawa -type f |awk -F / 'NF==7'
  dawa/folder37/45/23/43/54/file43
  




  
  












 









 
