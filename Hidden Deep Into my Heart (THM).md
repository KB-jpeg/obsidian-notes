---
layout: default
title: Pickle Rick
---

#### Nmap Scan
```shell
root@ip-10-112-70-35:~# nmap 10.112.184.73
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-11-06 10:42 UTC
Nmap scan report for ip-10-112-184-73.eu-central-1.compute.internal (10.112.184.73)
Host is up (0.00050s latency).
Not shown: 998 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
5000/tcp open  upnp

Nmap done: 1 IP address (1 host up) scanned in 0.10 seconds
```
The Nmap scan revealed 2 open ports: 22 and 5000. Nmap detected port 5000 as UPnP, but this is not the case. The room instruction says this is a website.
#### Webstite
![[Pasted image 20261106114536.png]]
This is what the website looks like. I checked the source code and there's nothing out of the ordinary there.
#### Gobuster Scan
```shell
root@ip-10-112-70-35:~# gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.112.184.73:5000
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.112.184.73:5000
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/console              (Status: 400) [Size: 167]
/robots.txt           (Status: 200) [Size: 70]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================
```
The Gobuster scan revealed "robots.txt", which is a file used on web servers to tell crawlers which URLs they can or cannot access. Contents:
```
User-agent: *
Disallow: /cupids_secret_vault/*

# cupid_arrow_2026!!!
```
`/cupids_secret_vault/` looks like a URL path, but `cupid_arrow_2026!!!` looks more like a password, considering the exclamation marks.

Opening http://10.112.184.73:5000/cupids_secret_vault/ revealed this message:
![[Pasted image 20261106115326.png]]

A Gobuster scan of that directory reveals another site:
```shell
root@ip-10-112-70-35:~# gobuster dir -w /usr/share/wordlists/dirb/common.txt -u http://10.112.184.73:5000/cupids_secret_vault/
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.112.184.73:5000/cupids_secret_vault/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/administrator        (Status: 200) [Size: 2381]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================
```

At http://10.112.184.73:5000/cupids_secret_vault/administrator I am met with a login screen:
![[Pasted image 20261106115700.png]]
I tried these credentials:
Username: *Admin*
Password: *cupid_arrow_2026!!!*
Which worked, giving me the flag.
