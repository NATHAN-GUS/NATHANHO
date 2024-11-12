# TP1 : Plip plap votre OS

## 1. Programme, service, processus


## 🌞 Lister tous les processus en cours d'exécution sur votre machine

```sh
PS C:\Users\gusta> get-process | select ProcessName, Id| sort-object -proper
ty ProcessName

ProcessName             --     Id
-----------                    --
AAADSvc        --              4224

ACCStd          ----            20892

ACCSvc                ----       4192

ACCUserPS            ----       13680

AcerCCAgent           ---       4276

AcerDIAgent          --      4468

AcerEZService         --       4244

AcerPixyService       --       4232

AcerQAAgent            --      4360

AcerSense             --      17296

AcerSense             --      17344

AcerSense              --     16520

AcerSense              --     16720

AcerService             --     6476
```


## 🌞 Trouver les 3 processus qui ont le plus petit identifiant

```sh

PS C:\Users\gusta> Get-Process | Sort-Object Id | Select-Object -First 3 | Format-Table ProcessName, ID

ProcessName   -- Id

Idle         -------------   0

System      ----------    4

Secure System-- 172
```


 ## 🌞 Lister tous les services de la machine...
```sh
#PS C:\Users\gusta> Get-Service | Where-Object { $_.Status -eq "Running" }

Status  - Name       --        DisplayName
------   ----               -----------
Running  ACCSvc             ACC Service

Running  AcerARTAIMMXDri... AAADSvc

Running  AcerARTAIMMXSer... ARTAimmxService

Running  AcerCCAgentSvis    Acer Care Center

Running  AcerDeviceEnabl... Acer Device Enabling Sevice V2

Running  AcerDIAgentSvis    Acer Device Info

Running  AcerEZSvc          Acer Experience Zone Component Service

Running  AcerPixyService    AcerPixyService

Running  AcerQAAgentSvis    Acer Quick Access

Running  AcerServiceSvc     Acer Service Component Service

Running  Appinfo            Informations d’application

Running  AppXSvc            Service de déploiement AppX (AppXSVC)

Running  ASMSvc             Acer System Monitor Service

Running  AudioEndpointBu... Générateur de points de terminaison...

Running  Audiosrv           Audio Windows

Running  BDESVC             Service de chiffrement de lecteur B...

Running  BFE                Moteur de filtrage de base
```
```sh

#PS C:\Users\gusta> Get-Service | Where-Object { $_.Status -eq "Stopped" }

Status  - Name       --------        DisplayName
------   ----               -----------
Stopped  AarSvc_726c0       AarSvc_726c0

Stopped  AJRouter           Service de routeur AllJoyn

Stopped  ALG                Service de la passerelle de la couc...

Stopped  AppIDSvc           Identité de l’application

Stopped  AppReadiness       Préparation des applications

Stopped  autotimesvc        Heure cellulaire

Stopped  AxInstSV           Programme d’installation ActiveX (A...

Stopped  BcastDVRUserSer... BcastDVRUserService_726c0

Stopped  BITS               Service de transfert intelligent en...

Stopped  CaptureService_... CaptureService_726c0

Stopped  CertPropSvc        Propagation du certificat

Stopped  ClipSVC            Service de licences de client (Clip...

Stopped  CloudBackupRest... CloudBackupRestoreSvc_726c0

Stopped  COMSysApp          Application système COM+

Stopped  ConsentUxUserSv... ConsentUxUserSvc_726c0

Stopped  CredentialEnrol... CredentialEnrollmentManagerUserSvc_...

Stopped  dbupdate           Service Mise à jour Dropbox (dbupdate)

Stopped  dbupdatem          Service Mise à jour Dropbox (dbupda...

Stopped  dcsvc              Service de configuration (DC) déclaré
```



# 2. Mémoire et CPU
```sh

-🌞 RAM

* afficher la quantité de RAM totale de la machine : (Get-CimInstance -ClassName Win32_ComputerSystem).TotalPhysicalMemory / 1MB
16088,0546875
  
* afficherla quantité de RAM libre sur la machine 


PS C:\Users\gusta> Get-WmiObject -Class Win32_OperatingSystem | Select-Object FreePhysicalMemory

FreePhysicalMemory
------------------
           2972464
```

```sh
-🌞 CPU

* afficher l'utilisation du CPU

PS C:\Users\gusta> Get-WmiObject Win32_Processor | Select-Object LoadPercentage

LoadPercentage

             3
```

# 3. Stockage
```sh

🌞 Périphériques

 PS C:\Users\gusta> Get-Disk | Select-Object -Property FriendlyName, Size

FriendlyName, Size
                          
                          NVMe Micron_2450_MTFDKBA1T0TFK 1024209543168
-------------------------------------------------------------
```

-🌞 Partitions
```sh

PS C:\Users\gusta> Get-partition


   DiskPath : \\?\scsi#disk&ven_nvme&prod_micron_2450_mtfd#4&38f1a3f7&0&020000#{53f56307-b6bf-11d0-94f2-00a0c91efb8b}


PartitionNumber  DriveLetter Offset                                        Size Type
---------------  ----------- ------                                        ---- ----
1                           1048576                                     260 MB System

2                           273678336                                    16 MB Reserved

3                C           290455552                                 952.6 GB Basic

4                           1023135449088                                 1 GB Recovery
 
 	```

 -🌞 Espace disque
```sh
 PS C:\Users\gusta> $ListDisk = Get-WmiObject -Class Win32_LogicalDisk
PS C:\Users\gusta>
PS C:\Users\gusta> Foreach($Disk in $ListDisk){
>>    $DiskFreeSpace = ($Disk.freespace/1GB).ToString('F2')
>>    Write-Host "L'espace disque restant sur $($Disk.DeviceID) est de $DiskFreeSpace Go"
>> }
L'espace disque restant sur C: est de 851,06 Go
```

# 4. Réseau

-🌞 Cartes réseau
```sh

PS C:\Users\gusta> Get-NetAdapter | Select-Object Name, @{Name="IP Address";Expression={(Get-NetIPAddress -InterfaceAlias $_.Name).IPAddress}}

Name                       IP Address
----                       ----------
Connexion réseau Bluetooth 169.254.133.111

Ethernet                   169.254.246.24

Ethernet 2                 {fe80::cd5a:d4a4:7842:8f5b%5, 192.168.56.1}

Wi-Fi                      10.188.125.136
```

 
 -🌞 Connexions réseau
 ```sh 
 
 
PS C:\Users\gusta> Get-NetTCPConnection -State Established | Select-Object LocalAddress, State, @{Name="ProcessName";Expression={(Get-Process -Id $_.OwningProcess).Name}}


LocalAddress         ----------State --------ProcessName
------------         ----- -----------


10.188.125.136 ------Established-- msedgewebview2


10.188.125.136 ------Established ----msedge

10.188.125.136 ------Established ----chrome

10.188.125.136 -----Established -----ONENOTE

-------127.0.0.1      -----Established-- ADESv2Svc

10.188.125.136 -----Established -----svchost

10.188.125.136 ---- Established ------msedge

10.188.125.136 -----Established -----Dropbox

10.188.125.136 -----Established -----Spotify

10.188.125.136 -----Established -----WhatsApp

10.188.125.136 -----Established -----Dropbox

10.188.125.136 -----Established -----msedgewebview2

10.188.125.136 -----Established -----OneDrive

10.188.125.136 -----Established ----------msedgewebview2

-------127.0.0.1      -----Established ----------Dropbox

-------127.0.0.1      -----Established ------Dropbox

-------127.0.0.1      -----Established -----Dropbox

-------127.0.0.1      -----Established  ----Dropbox

10.188.125.136 -----Established -----Spotify

10.188.125.136 -----Established -----Spotify

10.188.125.136 -----Established -----Spotify

10.188.125.136 -----Established -----ms-teams

-------127.0.0.1      -----Established -----AcerSense

-------127.0.0.1      -----Established -----AcerSense

-------127.0.0.1      -----Established -----AQAUserPS

-------127.0.0.1      -----Established -----ipfsvc

-------127.0.0.1      -----Established -----ipfsvc

-------127.0.0.1      -----Established -----AcerSysMoni...

-------127.0.0.1      -----Established -----AcerSysMoni...

-------127.0.0.1      -----Established -----WUDFHost

10.188.125.136 -----Established -----svchost

10.188.125.136 -----Established -----msedge

10.188.125.136 -----Established -----msedgewebview2

10.188.125.136 -----Established -----msedge

------127.0.0.1      -----Established -----AcerSysMoni...

------127.0.0.1      -----Established------AcerQAAgent
```


# 5. Utilisateurs

🌞 Lister les utilisateurs de la machine
``` sh

PS C:\Users\gusta> Get-LocalUser

Name               Enabled Description
----               ------- -----------
Administrateur     False   Compte d’utilisateur d’administration

DefaultAccount     False   Compte utilisateur géré par le système.

gusta              True

Invité             False   Compte d’utilisateur invité

WDAGUtilityAccount False   Compte d’utilisateur géré et utilisé par le système pour les scénarios Windows Defender A...
```


-🌞 Heure de login
```sh


PS C:\Users\gusta> (Get-WmiObject Win32_LogonSession | Where-Object { $_.LogonType -eq 2 }).StartTime

2024 11 07 11 40 57.691228+060
```


-🌞 Lister les processus en cours d'exécution
```sh

PS C:\Users\gusta> Get-WmiObject Win32_Process | Select-Object Name, @{Name="UserName";Expression={(Get-WmiObject Win32_Process -Filter "ProcessId=$($_.ProcessId)").GetOwner().User}}

Name                ------------UserName
----                --------

mc-fw-host.exe

ipf_uf.exe


svchost.exe

AcerSysMonitorSe...

svchost.exe

SurSvc.exe

MsMpEng.exe

WMIRegistrationS...

svchost.exe

svchost.exe

WmiPrvSE.exe

AcerService.exe

conhost.exe

gamingservicesne...

gamingservices.exe

ipf_helper.exe     ---------- gusta

mc-fw-host.exe      ----------gusta

AggregatorHost.exe

svchost.exe

sihost.exe          ----------gusta

svchost.exe         ----------gusta

svchost.exe         ----------gusta

svchost.exe        ---------- gusta

svchost.exe         ----------gusta

svchost.exe

taskhostw.exe       ----------gusta

DropboxUpdate.exe

DropboxCrashHand...

explorer.exe        ---------gusta
```


-🌞 Sur un fichier random qui se trouve dans votre dossier Téléchargements/

```sh
PS C:\Users\gusta> Get-Acl "C:\wamp" | Select Owner

Owner

BUILTIN\Administrateurs
```
 
 
 
 # 6.Random
 ```sh
-🌞 Uptime

PS C:\Users\gusta> (Get-CimInstance -ClassName Win32_OperatingSystem).LastBootUpTime

mercredi 6 novembre 2024 15:02:34
```


-🌞 Device]
```sh
* afficher le modèle du processeur de la machine

PS C:\Users\gusta> Get-CimInstance -ClassName Win32_Processor | Select-Object -ExpandProperty Name
[12th Gen Intel(R) Core(TM) i7-12650H]
```

-🌞 Version
```sh

PS C:\Users\gusta> Get-CimInstance -Class Win32_OperatingSystem | Select-Object Caption, BuildNumber

[Caption                      BuildNumber
Mi-crosoft Windows 11 Famille 22631]
```

-🌞 Mise à jour
```sh

PS C:\Users\gusta> function Get-WuaHistory {
>>     $session = (New-Object -ComObject 'Microsoft.Update.Session')
>>     $history = $session.QueryHistory("",0,50) | ForEach-Object {
>>         $Result = Convert-WuaResultCodeToName -ResultCode $_.ResultCode
>>         $_ | Add-Member -MemberType NoteProperty -Value $Result -Name Result
>>         $Product = $_.Categories | Where-Object {$_.Type -eq 'Product'} | Select-Object -First 1 -ExpandProperty Name
>>         $_ | Add-Member -MemberType NoteProperty -Value $_.UpdateIdentity.UpdateId -Name UpdateId
>>         $_ | Add-Member -MemberType NoteProperty -Value $_.UpdateIdentity.RevisionNumber -Name RevisionNumber
>>         $_ | Add-Member -MemberType NoteProperty -Value $Product -Name Product -PassThru
>>         Write-Output $_ 
>>     }
>>     $history |
>>     Where-Object {![String]::IsNullOrWhiteSpace($_.title)} |
>>     Select-Object Result, Date, Title, SupportUrl, Product, UpdateId, RevisionNumber |
>>     Sort-Object Date -Descending |
>>     Select-Object -First 1
>> }
PS C:\Users\gusta>
PS C:\Users\gusta> (Get-WuaHistory).Date

mercredi 6 novembre 2024 14:13:28
```

# 7. Ptit amusement
```sh
-🌞 Lister les connexions actives

Get-NetTCPConnection | Where-Object {$.LocalAddress -eq "10.188.125.136"} | ForEach-Object { $process = Get-Process -Id $.OwningProcess [PSCustomObject]@{ LocalAddress = $.LocalAddress RemoteAddress = $.RemoteAddress ProcessName = $process.ProcessName } } | Format-Table -AutoSize

[LocalAddress RemoteAddress ProcessName

10.188.125.136 23.33.90.78 DSAService

10.188.125.136 152.199.19.160 Code

10.188.125.136 35.186.224.26 Spotify

10.188.125.136 140.82.113.26 chrome

10.188.125.136 216.58.214.174 chrome

10.188.125.136 216.239.32.36 chrome

10.188.125.136 142.250.75.238 chrome

10.188.125.136 104.26.9.72 brave

10.188.125.136 172.64.69.88 msedge

10.188.125.136 172.64.69.88 msedge

10.188.125.136 162.125.21.2 Dropbox

10.188.125.136 162.125.21.3 Dropbox

10.188.125.136 157.240.203.60 WhatsApp

10.188.125.136 185.60.219.60 WhatsApp

10.188.125.136 163.70.128.60 WhatsApp

10.188.125.136 31.13.86.51 WhatsApp

10.188.125.136 199.232.170.217 msedge

10.188.125.136 104.18.1.46 msedge

10.188.125.136 35.186.224.24 Spotify]
```



-🌞 En apprendre + sur le processus en cours d'exécution
```sh

PS C:\Users\gusta> Get-Process -Name "Spotify" | ForEach-Object {
>>     $wmi = Get-WmiObject Win32_Process -Filter "ProcessId = $($_.Id)"
>>     $user = $wmi.GetOwner().User
>>     $_ | Select-Object Name, Id, WorkingSet, @{Name="User";Expression={$user}}
>> }

Name       Id ----WorkingSet User
----       -- ---------- ----
Spotify --10008   53407744 gusta

Spotify --12176   83566592 gusta
------           ------   ---
```


-🌞 En apprendre + sur le programme
```sh

PS C:\Users\gusta> Get-Acl "C:\Program Files\Google\Chrome\Application" | Select Owner

Owner
-----
BUILTIN\Administrateurs
```

🌞 En apprendre + sur l'adresse IP
```sh
PS C:\Users\gusta> Invoke-RestMethod -Method Get -Uri http://ip-api.com/json/13.37.13.37

status      : success
country     : France
countryCode : FR                                                                
region      : IDF                                                                       regionName  : Île-de-France 
city        : Paris                                                                 
zip         : 75000                                                                     lat         : 48,8566                                                               
lon         : 2,35222                                                                   timezone    : Europe/Paris  
isp         : Amazon Technologies Inc.                                                  org         : AWS EC2 (eu-west-3)                                                   
as          : AS16509 Amazon.com, Inc.                                          
query       : 13.37.13.37
```


-🌞 Délai
```sh
PS C:\Users\gusta> Test-NetConnection -ComputerName 13.37.13.37 -InformationLevel Detailed
AVERTISSEMENT : Ping to 13.37.13.37 failed with status: TimedOut


ComputerName           : 13.37.13.37

RemoteAddress          : 13.37.13.37

NameResolutionResults  : 13.37.13.37

ec2-13-37-13-37.eu-west-3.compute.amazonaws.com

InterfaceAlias         : Wi-Fi

SourceAddress          : 10.188.125.136

NetRoute (NextHop)     : 10.188.0.1

PingSucceeded          : False

PingReplyDetails (RTT) : 0 ms
```







