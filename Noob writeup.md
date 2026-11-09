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

We must enumarate with go buster and check the options that we have 

but first we need to clone a repository from github that contains directory lists 




