---
layout: default
title: Pickle Rick
nav_order: 2
---

#### Nmap Scan
```shell
root@ip-10-114-86-244:~# nmap 10.114.137.31
Starting Nmap 7.94SVN ( https://nmap.org ) at 2026-11-03 11:43 UTC
Nmap scan report for ip-10-114-137-31.eu-central-1.compute.internal (10.114.137.31)
Host is up (0.00014s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 0.18 seconds
```
The Nmap scan revealed an open HTTP port.
#### Website
At http://10.114.137.31:80 I found this:
![[Pasted image 20261103124648.png]]
Upon inspecting the HTML source code I found this comment:
```http  
  <!--
    Note to self, remember username!
    Username: R1ckRul3s
  -->
```
Username: R1ckRul3s
#### Gobuster Scan
```shell
root@ip-10-114-86-244:~# gobuster dir -w /usr/share/wordlists/dirb/common.txt -u 10.114.137.31

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.114.137.31
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/.hta                 (Status: 403) [Size: 278]
/assets               (Status: 301) [Size: 315] [--> http://10.114.137.31/assets/]
/index.html           (Status: 200) [Size: 1062]
/robots.txt           (Status: 200) [Size: 17]
/server-status        (Status: 403) [Size: 278]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================
```
http://10.114.137.31/robots.txt contains "Wubbalubbadubdub", this might be a password.

After being unable to find anything else I decided to use a Gobuster scan again, but this time with the flag "-x php", testing for php files as well. This showed me a new page:
```shell
/login.php            (Status: 200) [Size: 882]
```
After using the previously found credentials:
Username: R1ckRul3s
Password: Wubbalubbadubdub
#### Command Panel
Logging in with those credentials gave me access to the command panel.
I used to ls command to see the files and directories, which revealed 2 new files:
- Sup3rS3cretPickl3Ingred.txt
- clue.txt
The page http://10.114.137.31/Sup3rS3cretPickl3Ingred.txt contains the first answer.

The page clue.txt contains: "Look around the file system for the other ingredient.".

I used `ls /home`, which revealed 2 home directories: rick and ubuntu.
After looking around the file system, I found the file "second ingredients" in /home/rick. I read it with `less "/home/rick/second ingredients"`, giving me the second ingredient.

Using `sudo -l` I was able to find that my user has passwordless sudo permissions:
```shell
User www-data may run the following commands on ip-10-114-137-31:
    (ALL) NOPASSWD: ALL
```
This vulnerability allowed commands to be executed with root privileges, resulting in privilege escalation. I used `sudo ls -a /root`, which revealed the file: 3rd.txt. Using `sudo less /root/3rd.txt` (as cat is disabled) I found the 3rd and final ingredient.
#### Attack Chain
1. I used Nmap to discover the open HTTP port.
2. I found the username in a comment in the source code of the site.
3. I used Gobuster to try to find different directories.
4. I found the password in robots.txt, as well as a login page.
5. I used the previously found credentials to log in and gain access to the command panel.
6. I found 2 ingredients in the file system.
7. For the third, I had to use privilege escalation to gain access to /root.
