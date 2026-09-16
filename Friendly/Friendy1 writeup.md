# Friendly1 — HackmyVm Writeup

Welcome to the **Friendly1** writeup! In this machine we cover the full hacking cycle:

- Reconnaissance
- Exploring vulnerabilities
- Maintaining persistence
- Privilege escalation

<img width="685" alt="Hacking cycle overview" src="https://github.com/user-attachments/assets/960e1aa8-c0b0-4fc1-a32f-0ef0511c6559" />

---

## 1. Reconnaissance

Before starting reconnaissance, we need to verify our network interface and find the IP address of the Friendly1 machine.

### Step 1 — Identify the interface

Run the following command to check your network interface (the older equivalent is `ifconfig`):

```bash
ip a
```

As shown below, our interface is `eth0`.

<img width="540" alt="ip a output" src="https://github.com/user-attachments/assets/66d139ea-3822-4daa-9004-ac9ae50717e3" />

We can also ping the target to identify the type of operating system:

```bash
ping <target-ip>
```

This is a **Linux** machine, which we can tell from the **TTL (Time To Live)** value of 64. Most Windows machines have a TTL between 127–150, while Linux machines are typically in the 60–84 range.

<img width="612" alt="ping TTL result" src="https://github.com/user-attachments/assets/a70f3cab-0f30-41f6-a976-e161dae4322b" />

### Step 2 — Find the target's IP address

Now that we know our interface, let's scan the local network for the Friendly1 machine using ARP:

```bash
sudo arp-scan -I eth0 --localnet
```

This displays every device's IP address on our local network via ARP. In this case, the Friendly1 IP address is **192.168.15.14**.

<img width="402" alt="arp-scan results" src="https://github.com/user-attachments/assets/3c550fb0-da09-45fe-945e-6fc2fd1fc152" />
<img width="641" alt="Target IP confirmed" src="https://github.com/user-attachments/assets/e476446e-0f85-408c-ad06-6cf9b6d51f06" />

### Step 3 — Set up the working directory

Create a new directory on the Desktop named after the machine:

```bash
mkdir Friendly1
```

<img width="586" alt="mkdir Friendly1" src="https://github.com/user-attachments/assets/a4eb90f0-47cc-42d9-828c-71ede6487a05" />

### Step 4 — Port scan with Nmap

From inside the `Friendly1` directory, run a full Nmap scan:

```bash
sudo nmap -p- -sS -sC -sV --min-rate 5000 -n -Pn -vvv 192.168.15.14 -oN results.txt
```

After the scan, we find two open ports: **21/TCP (FTP)** and **80/TCP (HTTP)**. We'll start with port 21, since it may allow an FTP connection.

<img width="1142" alt="Nmap scan results" src="https://github.com/user-attachments/assets/61f1c558-c8fe-4abf-8f27-a3b20f41ac7f" />
<img width="733" alt="Open ports 21 and 80" src="https://github.com/user-attachments/assets/9aa19d92-46d9-42a5-b0cc-be4fb9e748df" />

---

## 2. Exploring Vulnerabilities

At this point, we start exploring vulnerabilities. As shown in the previous scan, there's a website running on the target's IP address.

<img width="1379" alt="Website on target IP" src="https://github.com/user-attachments/assets/7393b952-a20d-409a-ac56-f206befc2a3c" />

This machine hosts the website on an **Apache 2.4** server. These servers commonly run PHP — and indeed, we can see PHP files present — which means we can leverage the FTP access to upload a malicious script.

### A. Connect via FTP

```bash
ftp <target-ip>
```

### B. Log in anonymously

When prompted for a username, use `anonymous`; when prompted for a password, just press **Enter**.

Once connected, use the `dir` command to list the contents:

```bash
dir
```

<img width="756" alt="FTP anonymous login" src="https://github.com/user-attachments/assets/6f2c6ecd-ae7b-480b-91da-1a57aa6a1948" />

### C. Generate a reverse shell

We'll use the **Reverse Shell Generator** website to build our payload. Use *your* attacker IP address (not the victim's) and a port of your choice.

**1.** Paste your attacker IP address and port in the **IP/PORT** section.

<img width="1077" alt="Reverse Shell Generator IP/Port" src="https://github.com/user-attachments/assets/a65a9d6e-fbc5-4500-92b9-49c440ab0738" />

**2.** From the left-side menu, select the **PHP PentestMonkey** option.

<img width="1360" alt="Select PHP PentestMonkey" src="https://github.com/user-attachments/assets/fb093103-8291-4227-9187-32f041924252" />

**3.** Copy the entire generated code.

<img width="1360" alt="Copy the generated payload" src="https://github.com/user-attachments/assets/a952e039-d145-4369-8153-f3ef71b2a838" />

**4.** From a terminal inside the `Friendly1` directory, create a new PHP file (e.g. `login.php`):

```bash
nano login.php
```

<img width="499" alt="Create login.php" src="https://github.com/user-attachments/assets/ed685bd6-8a86-43c3-b4f9-8f415d0ec1a6" />

**5.** Paste the generated reverse shell code, then press `Ctrl + O` to save, `Enter` to confirm, and `Ctrl + X` to exit.

Now, inside the `Friendly1` directory, we have two files: `login.php` and `results.txt`.

<img width="468" alt="login.php and results.txt in folder" src="https://github.com/user-attachments/assets/3d8dad2b-7091-4d48-87b1-c6a55f969b64" />

### D. Upload the payload

Back in the FTP session, upload `login.php` to the server:

```bash
put login.php
```

<img width="1572" alt="FTP put login.php" src="https://github.com/user-attachments/assets/b63529c3-39bd-42f6-8cdf-a2bb1157c629" />

### E. Catch the reverse shell

Start a listener on the port you configured in the payload (in this case, 444):

```bash
sudo nc -nlvp 444
```

At the same time, trigger the payload by visiting `/login.php` on the website.

<img width="459" alt="Listener started" src="https://github.com/user-attachments/assets/5a6f197c-642b-4bce-ae34-d6a8acd6e271" />
<img width="1246" alt="Triggering login.php" src="https://github.com/user-attachments/assets/a6b1ce9a-1497-49e7-aff8-66aa9c0bf7b2" />

Once triggered, the listener catches the connection. Running `whoami` confirms we're logged in as **www-data**.

<img width="880" alt="whoami www-data" src="https://github.com/user-attachments/assets/f204286c-846e-4c91-ae7c-e2f6c0795b71" />

---

## 3. Persistence

We've compromised the machine, but we need to make sure we don't lose the connection. If the connection drops on a more complex machine, we may have to start from scratch — so we'll set up persistence in two parts.

### Part I — Stabilize the shell

Run the following commands, in order:

```bash
script /dev/null -c bash
```

Then press `Ctrl + Z`.

<img width="860" alt="script /dev/null -c bash" src="https://github.com/user-attachments/assets/85edf863-0452-4f2c-b67a-66967354f777" />

```bash
stty raw -echo; fg
```

Then set the terminal type:

```bash
reset xterm
export SHELL=bash
export TERM=xterm
```

<img width="498" alt="stty raw -echo; fg" src="https://github.com/user-attachments/assets/7e254b8c-8ff9-4083-8941-82df0285f935" />
<img width="419" alt="export SHELL and TERM" src="https://github.com/user-attachments/assets/7e9eddbe-05b0-48df-b2ef-05d6b2166c02" />

Now, using `Ctrl + C` or `clear` won't drop the connection.

### Part II — Persistence via Crontab

We'll use `crontab -e` to automatically run a command on a schedule — in this case, a reverse shell that fires every minute.

Go back to the Reverse Shell Generator website, change the port to **445**, and select the **bash -i** option this time.

<img width="1257" alt="bash -i option, port 445" src="https://github.com/user-attachments/assets/bcd90343-ba2a-4697-af5f-a02bbad60f39" />

Copy that code and paste it into the crontab file, using the standard cron format so it runs every minute:

```bash
* * * * * bash -c '<reverse-shell-payload>'
```

Save with `Ctrl + O`, `Enter`, then exit with `Ctrl + X`.

<img width="816" alt="Editing crontab" src="https://github.com/user-attachments/assets/4371b7d6-b3ed-4a47-a4eb-141c70a6e9a6" />

To verify the crontab entry works, open a new terminal and start a listener on port 445 — it should catch a new connection automatically every minute:

```bash
sudo nc -nlvp 445
```

<img width="674" alt="Listener catching cron connection" src="https://github.com/user-attachments/assets/ec96fa24-1266-4538-942b-bec98608e386" />

---

## 4. Privilege Escalation

To check for privilege escalation paths, run:

```bash
sudo -l
```

This shows that the `www-data` user can potentially become root via `/usr/bin/vim`.

We check [GTFOBins](https://gtfobins.github.io/) for known privilege escalation techniques involving `vim`.

<img width="1092" alt="GTFOBins vim entry" src="https://github.com/user-attachments/assets/c2d45980-b8e4-4907-aad6-9e4dc9b542ec" />

GTFOBins gives us the sudo escalation command:

```bash
vim -c ':!/bin/sh'
```

Putting it all together:

```bash
sudo -u root vim -c ':!/bin/sh'
```

Then confirm our privilege level:

```
www-data@friendly:/$ sudo -u root vim -c ':!/bin/sh'
# whoami
root
```

We're now root.

### Capturing the flags

You can search for the flags manually, or use `find`:

```bash
find / -iname "user.txt" 2>/dev/null
find / -iname "root.txt" 2>/dev/null
```

| Flag | Location | Value |
|------|----------|-------|
| **User flag** | `/home/RiJaba1/user.txt` | `b8cff8c9008e1c98a1f2937b4475acd6` |
| **Root flag** | `/var/log/apache2/root.txt` | `66b5c58f3e83aff307441714d3e28d2f` |

---

*Machine rooted — full cycle complete: recon → exploitation → persistence → privilege escalation.*





















