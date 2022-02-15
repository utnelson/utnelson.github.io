---
layout: post
title:  "HTB Write-Up Paper"
summary: Write-Up to the HackTheBox Machine Paper
author: utnelson
date: '2022-02-09 10:35:23 +0530'
category: ['htb','write-up', 'easy']
thumbnail: \assets\img\posts\paper\paper.PNG
keywords: paper, htb, box, easy, write-up, walkthrough, polkit
permalink: /blog/write-up-paper/
---

## Enumeration

Run a Nmap Scan

```console
root@machine:~$ nmap 10.10.11.143
Starting Nmap 7.92 ( https://nmap.org ) at 2022-02-10 09:12 CET
Nmap scan report for localhost (10.10.11.143)
Host is up (0.072s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT    STATE SERVICE
22/tcp  open  ssh
80/tcp  open  http
443/tcp open  https
```
Nothing special so i tried to look at the website.

![image](\assets\img\posts\paper\paper_website.PNG)

Gobuster also didn't show anything interesting. So I tried a new tool called `nikto`.

```console
root@machine:~$ nikto -h 10.10.11.143
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.11.143
+ Target Hostname:    10.10.11.143
+ Target Port:        80
+ Start Time:         2022-02-10 09:21:59 (GMT1)
---------------------------------------------------------------------------
+ Server: Apache/2.4.37 (centos) OpenSSL/1.1.1k mod_fcgid/2.3.9
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ Uncommon header 'x-backend-server' found, with contents: office.paper
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Retrieved x-powered-by header: PHP/7.2.24
+ Allowed HTTP Methods: HEAD, GET, POST, OPTIONS, TRACE 
+ OSVDB-877: HTTP TRACE method is active, suggesting the host is vulnerable to XST
...
```
Nice. We found a `x-backend-server` called `office-paper`. Add it to your "/etc/hosts" and see what we got.  
A new `Wordpress` site appears. Version `5.2.3`. Seems to be not Up-To-Date.  

There is a interesting comment:

<div class="text-center">
    <img src="\assets\img\posts\paper\paper_comment.PNG" class="rounded img-fluid">
</div>
<br/>  
  
Lets try to look if there is a vulnerability about that...

This sounds good: [wpscan.com 5.2.3](https://wpscan.com/vulnerability/3413b879-785f-4c9f-aa8a-5a4a1d5e0ba2)

![image](\paper\assets\img\posts\paper\vuln.PNG){:class="img-fluid"}

According to this we can find a nice looking secret register link. `http://chat.office.paper/register/*************`
Before you can follow the link add `chat.office.paper` to your host file.

Register an account and login. We see there is a bot, which accepts commands. We could try to leverage these.
`list` and `file` commands seem to be qual to `ls` and `cat`

With an simple added upgoing command `../` we can see the upper directory. So we can look arround for any secrets.
Inside the `/hubot/.env` file are the bots creds.

<div class="text-center">
    <img src="\assets\img\posts\paper\paper_bot.PNG" class="rounded img-fluid">   
</div>
<br/>

---

## Foothold

With the Creds from recyclops and the username of the /home directory

```console
root@machine:~$ ssh dwight@10.10.11.143
The authenticity of host '10.10.11.143 (10.10.11.143)' can't be established.
ECDSA key fingerprint is SHA256:2eiFA8VFQOZukubwDkd24z/kfLkdKlz4wkAa/lRN3Lg.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.143' (ECDSA) to the list of known hosts.
dwight@10.10.11.143's password: 
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Thu Feb 10 03:56:07 2022 from 10.10.14.3
[dwight@paper ~]$ 
```
Boom. Shell!

---

## Prev Escalation

Discovery with linpeas.sh?

[CVE-2021-3560](https://github.blog/2021-06-10-privilege-escalation-polkit-root-on-linux-with-bug/)

So there is already a complete Exploit on Exploit-db  

[PythonExploit](https://www.exploit-db.com/exploits/50011)

Upload the script. Run it. Boom Root Shell.