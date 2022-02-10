---
layout: post
title:  "HTB Write-Up Paper"
summary: Write-Up to the HackTheBox Machine Paper
author: utnelson
date: '2022-02-09 10:35:23 +0530'
category: ['htb','write-up', 'easy']
thumbnail: /assets/img/posts/paper.png
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

![image](\assets\img\posts\paper_website.PNG)

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
Nice. We found a `x-backend-server` called `office-paper`. Add it to your `/etc/hosts` and see what we got.
A new `Wordpress` Site appears. Version `5.2.3`. Not UpToDate.
There is a interesting comment:

![image](\assets\img\posts\paper_comment.PNG)

Lets try to look if there is a vulnerability about that...

[wpscan.com 5.2.3](https://wpscan.com/vulnerability/3413b879-785f-4c9f-aa8a-5a4a1d5e0ba2)

![imag](assets\img\posts\vuln.PNG)

