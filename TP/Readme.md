```sh
PS C:\Users\gusta> ssh nathan@10.1.1.1
nathan@10.1.1.1's password:
Last login: Sun Dec  1 17:09:57 2024
[nathan@web ~]$ sudo systemctl status sshd
[sudo] password for nathan:
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)
     Active: active (running) since Sun 2024-12-01 17:08:21 CET; 9min ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 704 (sshd)
      Tasks: 1 (limit: 11083)
     Memory: 4.9M
        CPU: 128ms
     CGroup: /system.slice/sshd.service
             └─704 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Dec 01 17:08:20 localhost.localdomain systemd[1]: Starting OpenSSH server daemon...
Dec 01 17:08:21 localhost.localdomain sshd[704]: Server listening on 0.0.0.0 port 22.
Dec 01 17:08:21 localhost.localdomain sshd[704]: Server listening on :: port 22.
Dec 01 17:08:21 localhost.localdomain systemd[1]: Started OpenSSH server daemon.
Dec 01 17:17:34 web.tp1.b1 sshd[1331]: Accepted password for nathan from 10.1.1.100 port 57996 ssh2
Dec 01 17:17:34 web.tp1.b1 sshd[1331]: pam_unix(sshd:session): session opened for user nathan(uid=1000) by nathan(uid=0)
[nathan@web ~]$ ps -ef | grep sshd
root         704       1  0 17:08 ?        00:00:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
root        1331     704  0 17:17 ?        00:00:00 sshd: nathan [priv]
nathan      1335    1331  0 17:17 ?        00:00:00 sshd: nathan@pts/0
nathan      1439    1336  0 19:22 pts/0    00:00:00 grep --color=auto sshd
[nathan@web ~]$ ps -aux | grep sshd
root         704  0.0  0.5  16796  9344 ?        Ss   17:08   0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
root        1331  0.0  0.6  20156 11520 ?        Ss   17:17   0:00 sshd: nathan [priv]
nathan      1335  0.0  0.3  20352  7096 ?        S    17:17   0:00 sshd: nathan@pts/0
nathan      1441  0.0  0.1   6408  2432 pts/0    S+   19:22   0:00 grep --color=auto sshd
[nathan@web ~]$ sudo ss lnpt | grep sshd
[sudo] password for nathan:
Error: an inet prefix is expected rather than "lnpt".
Cannot parse dst/src address.
[nathan@web ~]$ sudo ss alnpt | grep sshd
^C
[nathan@web ~]$ sudo ss -alnpt | grep sshd
LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=704,fd=3))
LISTEN 0      128             [::]:22           [::]:*    users:(("sshd",pid=704,fd=4))
[nathan@web ~]$ sudo ss -alnpt sshd
^C
[nathan@web ~]$ /var/log/auth.log
-bash: /var/log/auth.log: No such file or directory
[nathan@web ~]$ sudo cat /var/log/auth.log
cat: /var/log/auth.log: No such file or directory
[nathan@web ~]$ ls /var/log/auth.log
ls: cannot access '/var/log/auth.log': No such file or directory
[nathan@web ~]$ ls /var/log
anaconda  btmp    cron             dnf.log      firewalld   lastlog  messages  README  spooler  tallylog
audit     chrony  dnf.librepo.log  dnf.rpm.log  hawkey.log  maillog  private   secure  sssd     wtmp
[nathan@web ~]$ sudo tail /var/log/auth.log
tail: cannot open '/var/log/auth.log' for reading: No such file or directory
[nathan@web ~]$ sudo tail /var/log/secure
Dec  1 19:25:38 vbox sshd[1460]: pam_unix(sshd:session): session opened for user nathan(uid=1000) by nathan(uid=0)
Dec  1 19:27:22 vbox sudo[1493]:  nathan : TTY=pts/1 ; PWD=/home/nathan ; USER=root ; COMMAND=/bin/journalctl -xe -u sshd
Dec  1 19:27:22 vbox sudo[1493]: pam_unix(sudo:session): session opened for user root(uid=0) by nathan(uid=1000)
Dec  1 19:27:22 vbox sudo[1493]: pam_unix(sudo:session): session closed for user root
Dec  1 19:29:35 vbox sudo[1501]:  nathan : TTY=pts/0 ; PWD=/home/nathan ; USER=root ; COMMAND=/bin/cat /var/log/auth.log
Dec  1 19:29:35 vbox sudo[1501]: pam_unix(sudo:session): session opened for user root(uid=0) by nathan(uid=1000)
Dec  1 19:29:35 vbox sudo[1501]: pam_unix(sudo:session): session closed for user root
Dec  1 19:31:21 vbox sudo[1508]:  nathan : TTY=pts/0 ; PWD=/home/nathan ; USER=root ; COMMAND=/bin/tail /var/log/auth.log
Dec  1 19:31:21 vbox sudo[1508]: pam_unix(sudo:session): session opened for user root(uid=0) by nathan(uid=1000)
Dec  1 19:31:21 vbox sudo[1508]: pam_unix(sudo:session): session closed for user root
[nathan@web ~]$ which ssh
/usr/bin/ssh
[nathan@web ~]$ which sshd
/usr/sbin/sshd
[nathan@web ~]$ ls /etc/ssh
moduli      ssh_config.d  sshd_config.d       ssh_host_ecdsa_key.pub  ssh_host_ed25519_key.pub  ssh_host_rsa_key.pub
ssh_config  sshd_config   ssh_host_ecdsa_key  ssh_host_ed25519_key    ssh_host_rsa_key
[nathan@web ~]$ ls /etc/ssh | grep sshd
sshd_config
sshd_config.d
[nathan@web ~]$ ls /etc/ssh | grep sshd_config
sshd_config
sshd_config.d
[nathan@web ~]$ ls /etc/ssh/sshd_config
/etc/ssh/sshd_config
[nathan@web ~]$ echo $RANDOM
8709


[nathan@web ~]$ sudo cat /etc/ssh/sshd_config | grep Port
Port 8709
#GatewayPorts no
[nathan@web ~]$ sudo firewall-cmd --permanent --add-port=22/tcp
success
[nathan@web ~]$ sudo firewall-cmd reload
usage: 'firewall-cmd --help' for usage information or see firewall-cmd(1) man page
firewall-cmd: error: unrecognized arguments: reload
[nathan@web ~]$ sudo firewall-cmd --reload
success
[nathan@web ~]$ sudo firewall-cmd --permanent --remove-port=22/tcp
success
[nathan@web ~]$ sudo firewall-cmd --reload
success
[nathan@web ~]$ sudo firewall-cmd --permanent --add-port=8709/tcp
success
[nathan@web ~]$ sudo firewall-cmd --reload
success
[nathan@web ~]$ sudo firewall-cmd --list-all | grep Port
[nathan@web ~]$ sudo firewall-cmd --list-all | grep 8709
  ports: 8709/tcp
[nathan@web ~]$ sudo systemctl restart sshd
[nathan@web ~]$ ls /var/log
anaconda  btmp    cron             dnf.log      firewalld   lastlog  messages  private  secure   sssd      wtmp
audit     chrony  dnf.librepo.log  dnf.rpm.log  hawkey.log  maillog  nginx     README   spooler  tallylog
[nathan@web ~]$ ls /var/www
ls: cannot access '/var/www': No such file or directory
[nathan@web ~]$ systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Sun 2024-12-01 20:06:51 CET; 8h ago
    Process: 1809 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 1810 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 1811 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 1812 (nginx)
      Tasks: 2 (limit: 11083)
     Memory: 2.0M
        CPU: 63ms
     CGroup: /system.slice/nginx.service
             ├─1812 "nginx: master process /usr/sbin/nginx"
             └─1813 "nginx: worker process"

Dec 01 20:06:51 web.tp1.b1 systemd[1]: Starting The nginx HTTP and reverse proxy server...
Dec 01 20:06:51 web.tp1.b1 nginx[1810]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Dec 01 20:06:51 web.tp1.b1 nginx[1810]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Dec 01 20:06:51 web.tp1.b1 systemd[1]: Started The nginx HTTP and reverse proxy server.
[nathan@web ~]$ cat /usr/share/
aclocal/             factory/             hwdata/              microcode_ctl/       selinux/
appdata/             file/                i18n/                mime-info/           sounds/
applications/        fish/                icons/               misc/                sssd/
audit/               fontconfig/          idl/                 nano/                sssd-kcm/
augeas/              fonts/               ima/                 nginx/               systemd/
authselect/          games/               info/                omf/                 systemtap/
awk/                 gawk/                kdump/               os-prober/           tabset/
backgrounds/         gcc-11/              keyutils/            p11-kit/             terminfo/
bash-completion/     gdb/                 libgpg-error/        pam.d/               testpage/
cracklib/            gettext/             libreport/           pixmaps/             themes/
crypto-policies/     gettext-0.21/        licenses/            pkgconfig/           vim/
dbus-1/              glib-2.0/            locale/              pki/                 wayland-sessions/
desktop-directories/ gnome/               lua/                 polkit-1/            X11/
dict/                gnupg/               magic                publicsuffix/        xfsprogs/
doc/                 groff/               makedumpfile/        python3-wheels/      xsessions/
empty/               grub/                man/                 redhat-release/      zoneinfo/
empty.sshd/          help/                metainfo/            rocky-release/       zsh/
[nathan@web ~]$ cat /usr/share/
aclocal/             factory/             hwdata/              microcode_ctl/       selinux/
appdata/             file/                i18n/                mime-info/           sounds/
applications/        fish/                icons/               misc/                sssd/
audit/               fontconfig/          idl/                 nano/                sssd-kcm/
augeas/              fonts/               ima/                 nginx/               systemd/
authselect/          games/               info/                omf/                 systemtap/
awk/                 gawk/                kdump/               os-prober/           tabset/
backgrounds/         gcc-11/              keyutils/            p11-kit/             terminfo/
bash-completion/     gdb/                 libgpg-error/        pam.d/               testpage/
cracklib/            gettext/             libreport/           pixmaps/             themes/
crypto-policies/     gettext-0.21/        licenses/            pkgconfig/           vim/
dbus-1/              glib-2.0/            locale/              pki/                 wayland-sessions/
desktop-directories/ gnome/               lua/                 polkit-1/            X11/
dict/                gnupg/               magic                publicsuffix/        xfsprogs/
doc/                 groff/               makedumpfile/        python3-wheels/      xsessions/
empty/               grub/                man/                 redhat-release/      zoneinfo/
empty.sshd/          help/                metainfo/            rocky-release/       zsh/
[nathan@web ~]$ cat /usr/share/
aclocal/             factory/             hwdata/              microcode_ctl/       selinux/
appdata/             file/                i18n/                mime-info/           sounds/
applications/        fish/                icons/               misc/                sssd/
audit/               fontconfig/          idl/                 nano/                sssd-kcm/
augeas/              fonts/               ima/                 nginx/               systemd/
authselect/          games/               info/                omf/                 systemtap/
awk/                 gawk/                kdump/               os-prober/           tabset/
backgrounds/         gcc-11/              keyutils/            p11-kit/             terminfo/
bash-completion/     gdb/                 libgpg-error/        pam.d/               testpage/
cracklib/            gettext/             libreport/           pixmaps/             themes/
crypto-policies/     gettext-0.21/        licenses/            pkgconfig/           vim/
dbus-1/              glib-2.0/            locale/              pki/                 wayland-sessions/
desktop-directories/ gnome/               lua/                 polkit-1/            X11/
dict/                gnupg/               magic                publicsuffix/        xfsprogs/
doc/                 groff/               makedumpfile/        python3-wheels/      xsessions/
empty/               grub/                man/                 redhat-release/      zoneinfo/
empty.sshd/          help/                metainfo/            rocky-release/       zsh/
[nathan@web ~]$ cd /etc/nginx/
[nathan@web nginx]$
[nathan@web nginx]$
[nathan@web nginx]$ g
-bash: g: command not found
[nathan@web nginx]$ rep -nri root
-bash: rep: command not found
[nathan@web nginx]$ grep -nri root
fastcgi.conf:2:fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
fastcgi.conf:11:fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi.conf.default:2:fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
fastcgi.conf.default:11:fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_params:10:fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_params.default:10:fastcgi_param  DOCUMENT_ROOT      $document_root;
nginx.conf:42:        root         /usr/share/nginx/html;
nginx.conf:62:#        root         /usr/share/nginx/html;
nginx.conf.default:44:            root   html;
nginx.conf.default:54:            root   html;
nginx.conf.default:66:        #    root           html;
nginx.conf.default:73:        # deny access to .htaccess files, if Apache's document root
nginx.conf.default:90:    #        root   html;
nginx.conf.default:112:    #        root   html;
scgi_params:8:scgi_param  DOCUMENT_ROOT      $document_root;
scgi_params.default:8:scgi_param  DOCUMENT_ROOT      $document_root;
uwsgi_params:9:uwsgi_param  DOCUMENT_ROOT      $document_root;
uwsgi_params.default:9:uwsgi_param  DOCUMENT_ROOT      $document_root;
[nathan@web nginx]$
[nathan@web nginx]$ history
    1  sudo systemctl status sshd
    2  ps -ef | grep sshd
    3  ps -aux | grep sshd
    4  sudo ss lnpt | grep sshd
    5  sudo ss alnpt | grep sshd
    6  sudo ss -alnpt | grep sshd
    7  sudo ss -alnpt sshd
    8  /var/log/auth.log
    9  sudo cat /var/log/auth.log
   10  ls /var/log/auth.log
   11  ls /var/log
   12  sudo tail /var/log/auth.log
   13  sudo tail /var/log/secure
   14  which ssh
   15  which sshd
   16  ls /etc/ssh
   17  ls /etc/ssh | grep sshd
   18  ls /etc/ssh | grep sshd_config
   19  ls /etc/ssh/sshd_config
   20  echo $RANDOM
   21  sudo nano /etc/ssh/sshd_config
   22  sudo cat /etc/ssh/sshd_config | grep Port
   23  sudo firewall-cmd --permanent --add-port=22/tcp
   24  sudo firewall-cmd reload
   25  sudo firewall-cmd --reload
   26  sudo firewall-cmd --permanent --remove-port=22/tcp
   27  sudo firewall-cmd --reload
   28  sudo firewall-cmd --permanent --add-port=8709/tcp
   29  sudo firewall-cmd --reload
   30  sudo firewall-cmd --list-all | grep Port
   31  sudo firewall-cmd --list-all | grep 8709
   32  sudo systemctl restart sshd
   33  ls /var/log
   34  ls /var/www
   35  systemctl status nginx
   36  cd /etc/nginx/
   37  g
   38  rep -nri root
   39  grep -nri root
   40  history
[nathan@web nginx]$ sudo mkdir -p /var/www/
[sudo] password for nathan:
[nathan@web nginx]$ ls /var/www
[nathan@web nginx]$ cd ..
[nathan@web etc]$ cd ..
[nathan@web /]$ cd ..
[nathan@web /]$ ls /var/www
[nathan@web /]$ cd /var/www
[nathan@web www]$ sudo mkdir /var/www/tp1_parc
[nathan@web www]$ cd ..
[nathan@web var]$ cd ..
[nathan@web /]$ cd ..
[nathan@web /]$ ls /var/www
tp1_parc
[nathan@web /]$  /var/www/tp1_parc
-bash: /var/www/tp1_parc: Is a directory
[nathan@web /]$ cd tp1_parc
-bash: cd: tp1_parc: No such file or directory
[nathan@web /]$ cd /var/www
[nathan@web www]$ ls
tp1_parc
[nathan@web www]$  cd /var/www/tp1_parc
[nathan@web tp1_parc]$ cat > index.html
-bash: index.html: Permission denied
[nathan@web tp1_parc]$ sudo cat > index.html
-bash: index.html: Permission denied
[nathan@web tp1_parc]$ sudo nano index.html
[sudo] password for nathan:
[nathan@web tp1_parc]$ ls
index.html
[nathan@web tp1_parc]$ cd
[nathan@web ~]$ cd

```

```sh
[nathan@web ~]$ sudo journalctl -xe -u sshd
[sudo] password for nathan:
~
~
~
~
~
~
~
Dec 01 17:08:20 localhost.localdomain systemd[1]: Starting OpenSSH server daemon...
░░ Subject: A start job for unit sshd.service has begun execution
░░ Defined-By: systemd
░░ Support: https://wiki.rockylinux.org/rocky/support
░░
░░ A start job for unit sshd.service has begun execution.
░░
░░ The job identifier is 223.
Dec 01 17:08:21 localhost.localdomain sshd[704]: Server listening on 0.0.0.0 port 22.
Dec 01 17:08:21 localhost.localdomain sshd[704]: Server listening on :: port 22.
Dec 01 17:08:21 localhost.localdomain systemd[1]: Started OpenSSH server daemon.
░░ Subject: A start job for unit sshd.service has finished successfully
░░ Defined-By: systemd
░░ Support: https://wiki.rockylinux.org/rocky/support
░░
░░ A start job for unit sshd.service has finished successfully.
░░
░░ The job identifier is 223.
Dec 01 17:17:34 web.tp1.b1 sshd[1331]: Accepted password for nathan from 10.1.1.100 port 57996 ssh2
Dec 01 17:17:34 web.tp1.b1 sshd[1331]: pam_unix(sshd:session): session opened for user nathan(uid=1000) by nathan(uid=0)
Dec 01 19:25:38 web.tp1.b1 sshd[1460]: Accepted password for nathan from 10.1.1.100 port 60706 ssh2
Dec 01 19:25:38 web.tp1.b1 sshd[1460]: pam_unix(sshd:session): session opened for user nathan(uid=1000) by nathan(uid=0)
[nathan@web ~]$ sudo systemctl status sshd
[sudo] password for nathan:
● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)
     Active: active (running) since Sun 2024-12-01 20:20:01 CET; 2min 23s ago
       Docs: man:sshd(8)
             man:sshd_config(5)
   Main PID: 1873 (sshd)
      Tasks: 1 (limit: 11083)
     Memory: 1.4M
        CPU: 19ms
     CGroup: /system.slice/sshd.service
             └─1873 "sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups"

Dec 01 20:20:01 web.tp1.b1 systemd[1]: Starting OpenSSH server daemon...
Dec 01 20:20:01 web.tp1.b1 sshd[1873]: Server listening on 0.0.0.0 port 8709.
Dec 01 20:20:01 web.tp1.b1 sshd[1873]: Server listening on :: port 8709.
Dec 01 20:20:01 web.tp1.b1 systemd[1]: Started OpenSSH server daemon.
[nathan@web ~]$ sudo mkdir /var/www/tp1_parc
[sudo] password for nathan:
mkdir: cannot create directory ‘/var/www/tp1_parc’: No such file or directory
[nathan@web ~]$ ls /var/www
ls: cannot access '/var/www': No such file or directory
[nathan@web ~]$ cd /var/www/
-bash: cd: /var/www/: No such file or directory
[nathan@web ~]$ cd var/www
-bash: cd: var/www: No such file or directory
[nathan@web ~]$ ls
[nathan@web ~]$ ps -ef | grep nginx
root        1812       1  0 Dec01 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx       1813    1812  0 Dec01 ?        00:00:00 nginx: worker process
root        1893    1618  0 Dec01 pts/2    00:00:00 sudo nano /etc/nginx/nginx.conf
root        1897    1893  0 Dec01 pts/2    00:00:00 nano /etc/nginx/nginx.conf
nathan      2084    1465  0 05:24 pts/1    00:00:00 grep --color=auto nginx
[nathan@web ~]$ sudo chown nginx /var/www/tp1_parc
[sudo] password for nathan:
[nathan@web ~]$ ls -al /var/www/tp1_parc
total 4
drwxr-xr-x. 2 nginx root 24 Dec  2 05:13 .
drwxr-xr-x. 3 root  root 22 Dec  2 04:55 ..
-rw-r--r--. 1 root  root 38 Dec  2 05:13 index.html
[nathan@web ~]$ sudo nano /etc/nginx/nginx.conf
[sudo] password for nathan:
[nathan@web ~]$
[nathan@web ~]$
[nathan@web ~]$ sudo nano /etc/nginx/nginx.conf
[sudo] password for nathan:
[nathan@web ~]$ sudo systemctl restart nginx
[nathan@web ~]$ sudo nano /etc/nginx/conf.d/nathan.conf
[nathan@web ~]$ ls /etc/nginx/conf.d
nathan.conf
[nathan@web ~]$ cd /etc/nginx/
[nathan@web nginx]$ cd /etc/nginx
[nathan@web nginx]$ cd ..
[nathan@web etc]$ cd ..
[nathan@web /]$ cd ..
[nathan@web /]$ ls /etc/nginx
conf.d        fastcgi.conf.default    koi-utf     mime.types.default  scgi_params          uwsgi_params.default
default.d     fastcgi_params          koi-win     nginx.conf          scgi_params.default  win-utf
fastcgi.conf  fastcgi_params.default  mime.types  nginx.conf.default  uwsgi_params
[nathan@web /]$ sudo nano /etc/nginx/nginx.conf
[sudo] password for nathan:
[nathan@web /]$ ls conf.d
ls: cannot access 'conf.d': No such file or directory
[nathan@web /]$ ls /etc/nginx/conf.d
nathan.conf
[nathan@web /]$ sudo systemctl restart nginx
[sudo] password for nathan:
[nathan@web /]$ sudo nano /etc/nginx/conf.d/nathan.conf
[nathan@web /]$ echo $RANDOM
4269
[nathan@web /]$ sudo nano /etc/nginx/conf.d/nathan.conf
[nathan@web /]$ sudo nano /etc/nginx/conf.d/nathan.conf
[nathan@web /]$ sudo firewall-cmd --remove-port=80/tcp --permanent
success
[nathan@web /]$ sudo firewall-cmd --add-port=4269/tcp --permanent
success
[nathan@web /]$ sudo firewall-cmd --reload
success
[nathan@web /]$ sudo systemctl restart nginx
[nathan@web /]$ sudo ss -lnpt |grep nginx
[sudo] password for nathan:
LISTEN 0      511          0.0``.0.0:4269      0.0.0.0:*    users:(("nginx",pid=2210,fd=6),("nginx",pid=2209,fd=6))
[nathan@web /]$

```

```sh
Windows PowerShell
Copyright (C) Microsoft Corporation. Tous droits réservés.

Installez la dernière version de PowerShell pour de nouvelles fonctionnalités et améliorations ! https://aka.ms/PSWindows

PS C:\Users\gusta> ssh nathan@10.1.1.1
nathan@10.1.1.1's password:
Last login: Sun Dec  1 19:25:38 2024 from 10.1.1.100
[nathan@web ~]$ exit
logout
Connection to 10.1.1.1 closed.
PS C:\Users\gusta> ssh -p 8709 nathan@10.1.1.1
nathan@10.1.1.1's password:
Last login: Sun Dec  1 19:52:48 2024 from 10.1.1.100
[nathan@web ~]$ dnf install nginx
Error: This command has to be run with superuser privileges (under the root user on most systems).
[nathan@web ~]$ sudo dnf install nginx
[sudo] password for nathan:
Rocky Linux 9 - BaseOS                                                                  8.5 kB/s | 4.1 kB     00:00
Rocky Linux 9 - AppStream                                                                15 kB/s | 4.5 kB     00:00
Rocky Linux 9 - Extras                                                                   10 kB/s | 2.9 kB     00:00
Dependencies resolved.
========================================================================================================================
 Package                         Architecture         Version                             Repository               Size
========================================================================================================================
Installing:
 nginx                           x86_64               2:1.20.1-20.el9.0.1                 appstream                36 k
Installing dependencies:
 nginx-core                      x86_64               2:1.20.1-20.el9.0.1                 appstream               566 k
 nginx-filesystem                noarch               2:1.20.1-20.el9.0.1                 appstream               8.4 k
 rocky-logos-httpd               noarch               90.15-2.el9                         appstream                24 k

Transaction Summary
========================================================================================================================
Install  4 Packages

Total download size: 634 k
Installed size: 1.8 M
Is this ok [y/N]: y
Downloading Packages:
(1/4): nginx-filesystem-1.20.1-20.el9.0.1.noarch.rpm                                    113 kB/s | 8.4 kB     00:00
(2/4): rocky-logos-httpd-90.15-2.el9.noarch.rpm                                         261 kB/s |  24 kB     00:00
(3/4): nginx-1.20.1-20.el9.0.1.x86_64.rpm                                               335 kB/s |  36 kB     00:00
(4/4): nginx-core-1.20.1-20.el9.0.1.x86_64.rpm                                          2.6 MB/s | 566 kB     00:00
------------------------------------------------------------------------------------------------------------------------
Total                                                                                   1.2 MB/s | 634 kB     00:00
Running transaction check
Transaction check succeeded.
Running transaction test
Transaction test succeeded.
Running transaction
  Preparing        :                                                                                                1/1
  Running scriptlet: nginx-filesystem-2:1.20.1-20.el9.0.1.noarch                                                    1/4
  Installing       : nginx-filesystem-2:1.20.1-20.el9.0.1.noarch                                                    1/4
  Installing       : nginx-core-2:1.20.1-20.el9.0.1.x86_64                                                          2/4
  Installing       : rocky-logos-httpd-90.15-2.el9.noarch                                                           3/4
  Installing       : nginx-2:1.20.1-20.el9.0.1.x86_64                                                               4/4
  Running scriptlet: nginx-2:1.20.1-20.el9.0.1.x86_64                                                               4/4
  Verifying        : rocky-logos-httpd-90.15-2.el9.noarch                                                           1/4
  Verifying        : nginx-filesystem-2:1.20.1-20.el9.0.1.noarch                                                    2/4
  Verifying        : nginx-2:1.20.1-20.el9.0.1.x86_64                                                               3/4
  Verifying        : nginx-core-2:1.20.1-20.el9.0.1.x86_64                                                          4/4

Installed:
  nginx-2:1.20.1-20.el9.0.1.x86_64                              nginx-core-2:1.20.1-20.el9.0.1.x86_64
  nginx-filesystem-2:1.20.1-20.el9.0.1.noarch                   rocky-logos-httpd-90.15-2.el9.noarch

Complete!
[nathan@web ~]$ sudo systemctl status nginx
○ nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: inactive (dead)
[nathan@web ~]$ sudo systemctl start nginx
[nathan@web ~]$ sudo systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Sun 2024-12-01 20:06:51 CET; 2s ago
    Process: 1809 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 1810 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 1811 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 1812 (nginx)
      Tasks: 2 (limit: 11083)
     Memory: 2.0M
        CPU: 40ms
     CGroup: /system.slice/nginx.service
             ├─1812 "nginx: master process /usr/sbin/nginx"
             └─1813 "nginx: worker process"

Dec 01 20:06:51 web.tp1.b1 systemd[1]: Starting The nginx HTTP and reverse proxy server...
Dec 01 20:06:51 web.tp1.b1 nginx[1810]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Dec 01 20:06:51 web.tp1.b1 nginx[1810]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Dec 01 20:06:51 web.tp1.b1 systemd[1]: Started The nginx HTTP and reverse proxy server.
[nathan@web ~]$ sudo ss -lnpt
State  Recv-Q  Send-Q   Local Address:Port   Peer Address:Port Process
LISTEN 0       511            0.0.0.0:80          0.0.0.0:*     users:(("nginx",pid=1813,fd=6),("nginx",pid=1812,fd=6))
LISTEN 0       128            0.0.0.0:8709        0.0.0.0:*     users:(("sshd",pid=1612,fd=3))
LISTEN 0       511               [::]:80             [::]:*     users:(("nginx",pid=1813,fd=7),("nginx",pid=1812,fd=7))
LISTEN 0       128               [::]:8709           [::]:*     users:(("sshd",pid=1612,fd=4))
[nathan@web ~]$ sudo ss -lnpt | grep nginx
LISTEN 0      511          0.0.0.0:80        0.0.0.0:*    users:(("nginx",pid=1813,fd=6),("nginx",pid=1812,fd=6))
LISTEN 0      511             [::]:80           [::]:*    users:(("nginx",pid=1813,fd=7),("nginx",pid=1812,fd=7))
[nathan@web ~]$ sudo firewall-cmd --remove-port=8709/tcp --permanent
success
[nathan@web ~]$ sudo firewall-cmd --add-port=80/tcp --permanent
success
[nathan@web ~]$ sudo firewall-cmd --reload
success
[nathan@web ~]$ sudo firewall-cmd --list-all | grep 80
  ports: 80/tcp
[nathan@web ~]$ ps -ef | grep nginx
root        1812       1  0 20:06 ?        00:00:00 nginx: master process /usr/sbin/nginx
nginx       1813    1812  0 20:06 ?        00:00:00 nginx: worker process
nathan      1843    1618  0 20:11 pts/2    00:00:00 grep --color=auto nginx
[nathan@web ~]$ ls /etc/passwd
/etc/passwd
[nathan@web ~]$ cd /etc/passwd
-bash: cd: /etc/passwd: Not a directory
[nathan@web ~]$ sudo cat /etc/passwd | grep nginx
nginx:x:996:993:Nginx web server:/var/lib/nginx:/sbin/nologin
[nathan@web ~]$ sudo cat /etc/passwd | grep nathan
nathan:x:1000:1000:nathan:/home/nathan:/bin/bash
[nathan@web ~]$ sudo cat /etc/passwd | grep root
root:x:0:0:root:/root:/bin/bash
operator:x:11:0:operator:/root:/sbin/nologin
[nathan@web ~]$ sudo nano/etc/ssh/sshd_config
sudo: nano/etc/ssh/sshd_config: command not found
[nathan@web ~]$ sudo nano /etc/ssh/sshd_config
[nathan@web ~]$ sudo systemctl restart sshd
Job for sshd.service failed because the control process exited with error code.
See "systemctl status sshd.service" and "journalctl -xeu sshd.service" for details.
[nathan@web ~]$ sudo nano /etc/ssh/sshd_config
[nathan@web ~]$ ls /etc
adjtime                  exports      ld.so.conf.d              os-release              ssh
aliases                  filesystems  libaudit.conf             pam.d                   ssl
alternatives             firewalld    libibverbs.d              passwd                  sssd
anacrontab               fonts        libnl                     passwd-                 statetab.d
audit                    fstab        libreport                 pkcs11                  subgid
authselect               gcrypt       libssh                    pki                     subgid-
bash_completion.d        gnupg        libuser.conf              pm                      subuid
bashrc                   GREP_COLORS  locale.conf               popt.d                  subuid-
binfmt.d                 groff        localtime                 printcap                sudo.conf
chrony.conf              group        login.defs                profile                 sudoers
chrony.keys              group-       logrotate.conf            profile.d               sudoers.d
cifs-utils               grub2.cfg    logrotate.d               protocols               sudo-ldap.conf
cron.d                   grub.d       lvm                       rc.d                    sysconfig
cron.daily               gshadow      machine-id                rc.local                sysctl.conf
cron.deny                gshadow-     magic                     redhat-release          sysctl.d
cron.hourly              gss          makedumpfile.conf.sample  request-key.conf        systemd
cron.monthly             host.conf    man_db.conf               request-key.d           system-release
crontab                  hostname     microcode_ctl             resolv.conf             system-release-cpe
cron.weekly              hosts        mke2fs.conf               rocky-release           terminfo
crypto-policies          inittab      modprobe.d                rocky-release-upstream  tmpfiles.d
crypttab                 inputrc      modules-load.d            rpc                     tpm2-tss
csh.cshrc                iproute2     motd                      rpm                     trusted-key.key
csh.login                issue        motd.d                    rsyslog.conf            udev
dbus-1                   issue.d      mtab                      rsyslog.d               vconsole.conf
default                  issue.net    nanorc                    rwtab.d                 vimrc
depmod.d                 kdump        NetworkManager            sasl2                   virc
dhcp                     kdump.conf   networks                  security                X11
DIR_COLORS               kernel       nftables                  selinux                 xattr.conf
DIR_COLORS.lightbgcolor  keys         nginx                     services                xdg
dnf                      keyutils     nsswitch.conf             sestatus.conf           yum
dracut.conf              krb5.conf    nsswitch.conf.bak         shadow                  yum.conf
dracut.conf.d            krb5.conf.d  nvme                      shadow-                 yum.repos.d
environment              ld.so.cache  openldap                  shells
ethertypes               ld.so.conf   opt                       skel
[nathan@web ~]$ ls /etc | grep nginx
nginx
[nathan@web ~]$ ls /etc/nginx
conf.d        fastcgi.conf.default    koi-utf     mime.types.default  scgi_params          uwsgi_params.default
default.d     fastcgi_params          koi-win     nginx.conf          scgi_params.default  win-utf
fastcgi.conf  fastcgi_params.default  mime.types  nginx.conf.default  uwsgi_params
[nathan@web ~]$ ls /etc/nginx/nginx.conf
/etc/nginx/nginx.conf
[nathan@web ~]$ cd /etc/nginx/nginx.conf
-bash: cd: /etc/nginx/nginx.conf: Not a directory
[nathan@web ~]$ cd etc/nginx/nginx.conf
-bash: cd: etc/nginx/nginx.conf: No such file or directory
[nathan@web ~]$ sudo nano /etc/nginx/nginx.conf
[sudo] password for nathan:
[nathan@web ~]$
```
