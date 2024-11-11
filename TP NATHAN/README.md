# TP2 : Le terminal vous a déjà manqué non

## II. Files and users

## 1. Fichiers
### A. Find me

#### 🌞 Trouver le chemin vers le répertoire personnel de votre utilisateur
 pwd :
 
 /home/nathan

#### 🌞 Vérifier les permissions du répertoire personnel de votre utilisateurs
ls -l /home/nathan :

 total 18056
 
drwxr-xr-x 2 nathan nathan     4096 Nov  9 14:48 Desktop


drwxr-xr-x 2 nathan nathan     4096 Nov  9 14:48 Documents

drwxr-xr-x 2 nathan nathan     4096 Nov  9 14:48 Downloads

drwxr-xr-x 2 nathan nathan     4096 Nov  9 17:37 gameshell

-rwxr-xr-x 1 nathan nathan   219773 Nov  9 13:51 gameshell-save.sh

-rw-r--r-- 1 nathan nathan   210298 Nov  9 13:40 gameshell.sh

-rw-r--r-- 1 nathan nathan 18016044 Jan 21  2024 meow.zip

drwxr-xr-x 2 nathan nathan     4096 Nov  9 14:48 Music

drwxr-xr-x 2 nathan nathan     4096 Nov  9 14:48 Pictures

drwxr-xr-x 2 nathan nathan     4096 Nov  9 14:48 Public

drwxr-xr-x 2 nathan nathan     4096 Nov  9 14:48 Templates

drwxr-xr-x 2 nathan nathan     4096 Nov  9 14:48 Videos

#### 🌞 Trouver le chemin du fichier de configuration du serveur SSH
find /etc/ssh/sshd_config :

/etc/ssh/sshd_config

## 2. Users

#### 🌞 Créer un nouvel utilisateur

useradd -m -d /home/papier_alu/ marmotte

passwd marmotte

New password:

Retype new password:

passwd: password updated successfully

### B. Infos enregistrées par le système

#### 🌞 Prouver que cet utilisateur a été créé
 cat /etc/passwd | grep marmotte
 
marmotte:x:1001:1001::/home/papier_alu/:/bin/sh

#### 🌞 Déterminer le hash du password de l'utilisateur marmotte
cat /etc/shadow | grep marmotte

marmotte:$y$j9T$rNeIV1dLfSitBbgU3vhQS/$Y19D4LxtUeZIGV3yrvv5CfnqIAyqXbpXO.7.b2ADPO7:20038:0:99999:7:::

### C. Hint sur la ligne de commande





 
