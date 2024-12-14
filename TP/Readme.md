Windows PowerShell
Copyright (C) Microsoft Corporation. Tous droits réservés.

Installez la dernière version de PowerShell pour de nouvelles fonctionnalités et améliorations ! https://aka.ms/PSWindows
```sh 

PS C:\Users\gusta> ssh nathan@10.2.1.1
PS C:\Users\gusta>
PS C:\Users\gusta> ssh nathan@10.1.1.100
ssh: connect to host 10.1.1.100 port 22: Connection refused
PS C:\Users\gusta> ^C
PS C:\Users\gusta> ssh nathan@10.1.1.1
ssh: connect to host 10.1.1.1 port 22: Connection timed out
PS C:\Users\gusta> ssh nathan@10.1.1.1
ssh: connect to host 10.1.1.1 port 22: Connection timed out
PS C:\Users\gusta>
PS C:\Users\gusta>
PS C:\Users\gusta>
PS C:\Users\gusta>
PS C:\Users\gusta>
PS C:\Users\gusta>
PS C:\Users\gusta> ssh nathan@10.1.1.1
ssh: connect to host 10.1.1.1 port 22: Connection timed out
PS C:\Users\gusta>
PS C:\Users\gusta> ssh nathan@10.1.1.1
PS C:\Users\gusta> ssh nathan@10.2.1.1
ssh: connect to host 10.2.1.1 port 22: Connection timed out
PS C:\Users\gusta> ssh nathan@10.2.1.1
The authenticity of host '10.2.1.1 (10.2.1.1)' can't be established.
ED25519 key fingerprint is SHA256:zK5EtW3LDzSnpXR7yWF+y16OSagl6eNKlce0S+E+EmI.
This host key is known by the following other names/addresses:
    C:\Users\gusta/.ssh/known_hosts:19: 10.1.1.1
    C:\Users\gusta/.ssh/known_hosts:22: 10.1.1.2
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.2.1.1' (ED25519) to the list of known hosts.
nathan@10.2.1.1's password:
Last login: Mon Dec  9 15:51:54 2024
[nathan@node1 ~]$
[nathan@node1 ~]$ df -h | grep 'Mem:' | tr -s ' ' | cut -d' ' -f7.
cut: invalid field value ‘.’
Try 'cut --help' for more information.
[nathan@node1 ~]$
[nathan@node1 ~]$ free -mh
               total        used        free      shared  buff/cache   available
Mem:           1.7Gi       304Mi       1.4Gi       4.0Mi       191Mi       1.4Gi
Swap:          1.0Gi          0B       1.0Gi
[nathan@node1 ~]$ df -h
Filesystem                Size  Used Avail Use% Mounted on
devtmpfs                  4.0M     0  4.0M   0% /dev
tmpfs                     888M     0  888M   0% /dev/shm
tmpfs                     355M  5.0M  350M   2% /run
/dev/mapper/rl_vbox-root  8.0G  1.3G  6.7G  17% /
/dev/sda1                 960M  230M  731M  24% /boot
tmpfs                     178M     0  178M   0% /run/user/1000
[nathan@node1 ~]$ df -h  | grep /dev/mapper
/dev/mapper/rl_vbox-root  8.0G  1.3G  6.7G  17% /
[nathan@node1 ~]$ df -h  | grep /dev/mapper | cut -d" "  -f1
/dev/mapper/rl_vbox-root
[nathan@node1 ~]$ df -h  | grep /dev/mapper | cut -d" "  -f2

[nathan@node1 ~]$ df -h  | grep /dev/mapper | cut -d" "  -f3
8.0G
[nathan@node1 ~]$ df -h  | grep /dev/mapper | tr -s " "
/dev/mapper/rl_vbox-root 8.0G 1.3G 6.7G 17% /
[nathan@node1 ~]$ df -h  | grep /dev/mapper | tr -s " " | cut -d' ' -f1
/dev/mapper/rl_vbox-root
[nathan@node1 ~]$ df -h  | grep /dev/mapper | tr -s " " | cut -d' ' -f2
8.0G
[nathan@node1 ~]$ df -h  | grep /dev/mapper | tr -s " " | cut -d' ' -f3
1.3G
[nathan@node1 ~]$ df -h  | grep /dev/mapper | tr -s " " | cut -d' ' -f5
17%
[nathan@node1 ~]$ df -h  | grep /dev/mapper | tr -s " " | cut -d' ' -f4
6.7G
[nathan@node1 ~]$ df -i
Filesystem                Inodes IUsed   IFree IUse% Mounted on
devtmpfs                  221678   390  221288    1% /dev
tmpfs                     227160     1  227159    1% /dev/shm
tmpfs                     819200   593  818607    1% /run
/dev/mapper/rl_vbox-root 4192256 33315 4158941    1% /
/dev/sda1                 524288   358  523930    1% /boot
tmpfs                      45432    14   45418    1% /run/user/1000
[nathan@node1 ~]$ df -i | grep /dev/mapper
/dev/mapper/rl_vbox-root 4192256 33315 4158941    1% /
[nathan@node1 ~]$ df -i | grep /dev/mapper | tr -s " " | cut -d' ' -f1
/dev/mapper/rl_vbox-root
[nathan@node1 ~]$ df -i | grep /dev/mapper | tr -s " " | cut -d' ' -f5
1%
[nathan@node1 ~]$ df -i | grep /dev/mapper | tr -s " " | cut -d' ' -f3
33315
[nathan@node1 ~]$ df -i | grep /dev/mapper | tr -s " " | cut -d' ' -f4
4158941
[nathan@node1 ~]$ date +"%d/%m/%y %H:%M:%S"
09/12/24 17:00:55
[nathan@node1 ~]$ source /etc/os-release && echo $PRETTY_NAME
Rocky Linux 9.5 (Blue Onyx)
[nathan@node1 ~]$ uname -r
5.14.0-503.14.1.el9_5.x86_64
[nathan@node1 ~]$ which python3
/usr/bin/python3
[nathan@node1 ~]$ echo $user

[nathan@node1 ~]$ echo $usernqme

[nathan@node1 ~]$ echo $username

[nathan@node1 ~]$ echo $USER
nathan
[nathan@node1 ~]$ cat /etc/passwd | grep $USER | cut -d: -f7
/bin/bash
[nathan@node1 ~]$ cat /etc/passwd | grep $USER
nathan:x:1000:1000:nathan:/home/nathan:/bin/bash
[nathan@node1 ~]$ cat /etc/passwd | grep $USER | cut -d' ' -f7
nathan:x:1000:1000:nathan:/home/nathan:/bin/bash


```sh
[nathan@node1 ~]$ rpm --query --queryformat "%{NAME} %{ARCH}\n" -a | grep ssh
libssh-config noarch
libssh x86_64
openssh x86_64
openssh-clients x86_64
openssh-server x86_64
[nathan@node1 ~]$ rpm --query --queryformat "%{NAME} %{ARCH}\n" -a | grep -c ssh
5
```
[nathan@node1 ~]$ cat /etc/passwd | grep $USER | tr -s " " |  cut -d' ' -f7
nathan:x:1000:1000:nathan:/home/nathan:/bin/bash
[nathan@node1 ~]$ cat /etc/passwd | grep $USER | cut -d: -f7
/bin/bash
[nathan@node1 ~]$
```
