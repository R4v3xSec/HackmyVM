
Friendly2 - Writeup


<img width="670" height="77" alt="image" src="https://github.com/user-attachments/assets/27dc3846-205e-41e3-b075-e69d4355f647" />

We are going to continuew with teh firnedly2 machine

Recconaissence

We are going to use a command sudo arp-sca -I eth0 --localnet

Result

192.168.15.11   08:00:27:0c:8f:a4       PCS Systemtechnik GmbH

Then we will use a ping command to determine what is this machine. 

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

The machine is a Linux device as show in time to life is equal to 64

We are going to create a directory with the name of Friendly2 

we will run a nmap command to complete the recconaissence 

┌──(clandhacker㉿kali)-[~/Desktop/Friendly2]

└─$ sudo nmap -p- -sS -sC -sV -n -Pn --min-rate 5000 -vvv 192.168.15.11 -oN results.txt 

-Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-16 18:29 -0500

PORT   STATE SERVICE REASON         VERSION

22/tcp open  ssh     syn-ack ttl 64 OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 74:fd:f1:a7:47:5b:ad:8e:8a:31:02:fe:44:28:9f:d2 (RSA)
| ssh-rsa 

AAAAB3NzaC1yc2EAAAADAQABAAABgQCzieRbxwfRD6zuOrOmgPocWFr6Ufu9oCqOlt/Da5dqgRIZwctsaB6P5+6aDoCtBvFAzQXZQSMmT4GmIWR7eZ/Obou3fBSMU4X8R+C/VLyx1wifxNHy5LZ0+6djQX5cl5qhBseWQX3XIqPt+4DzRILCiMZSm9J8dnC0KEe14a8vkSfgV7Zn7xGOaw9R+KldazraLdT3zlzVuvjZjItIBjnA9tBorwY2u/RgMX++HXD3uySm1qt8w+pFGI7WFd/ktfwp3RhcdKMEYmqWhjAO3L9A9arf2vDYL9y/t53XIs+FAOXzoBc2A5gxxVBe7sMsuQCSF0Jw0z5Qf11Zj9si//6WG2KfihR7rKLEIfgeGFGvnilw88HT6sZQGTew1VpfRFLgMZTPpAOwzxlqUYIRWEEvmPrW7DGqzuY+8NpJQpiOhdjhuiS0/SW6PfHVB/nsNs1pWWwo/q+HxyAAS3WjCrkd1xMf92KMs1yheQHKUGNxV/zVuTbt9puXnVhIZGzzhsE=

|   256 16:f0:de:51:09:ff:fc:08:a2:9a:69:a0:ad:42:a0:48 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBFE+bBFz/3QsD9M4Nt6is2iJpFKhlUCSEqpUtATmeiN6jNBE245wbyIk7h3JqOxldcKyfhn7uysTo8NG4AqhPEA=


|   256 65:0e:ed:44:e2:3e:f0:e7:60:0c:75:93:63:95:20:56 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIKxSz6doeuMiydUVbE7ZwrdP8GW46iJYY3JxJPcNuvnA
80/tcp open  http    syn-ack ttl 64 Apache httpd 2.4.56 ((Debian))


| http-methods: 
|_  Supported Methods: POST OPTIONS HEAD GET
|_http-server-header: Apache/2.4.56 (Debian)
|_http-title: Servicio de Mantenimiento de Ordenadores
MAC Address: 08:00:27:0C:8F:A4 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

We can see that during the scan we found a SSH-RSA in the port 22.

Also If we take a look at the page from the IP Address is shows a page called Servicio de Mantenimiento de Ordenadores

<img width="1516" height="552" alt="image" src="https://github.com/user-attachments/assets/033ed806-af26-4454-8662-db69e02e920a" />

In this machine we need to run a WFUZZ with the command 

wfuzz -c --hc 404 -w /usr/share-wordlist/dirbuster/directory-list-lowercase-2.3-medium.txt -u 'http://192.168.15.11/FUZZ' 

Results:

└─$ wfuzz -c --hc 404 -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -u 'http://192.168.15.11/FUZZ'  
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://192.168.15.11/FUZZ
Total requests: 207643

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                   
=====================================================================

000000003:   200        91 L     262 W      2698 Ch     "# Copyright 2007 James Fisher"                                                           
000000001:   200        91 L     262 W      2698 Ch     "# directory-list-lowercase-2.3-medium.txt"                                               
000000007:   200        91 L     262 W      2698 Ch     "# license, visit http://creativecommons.org/licenses/by-sa/3.0/"                         
000000014:   200        91 L     262 W      2698 Ch     "http://192.168.15.11/"                                                                   
000000012:   200        91 L     262 W      2698 Ch     "# on atleast 2 different hosts"                                                          
000000013:   200        91 L     262 W      2698 Ch     "#"                                                                                       
000000011:   200        91 L     262 W      2698 Ch     "# Priority ordered case insensative list, where entries were found"                      
000000005:   200        91 L     262 W      2698 Ch     "# This work is licensed under the Creative Commons"                                      
000000002:   200        91 L     262 W      2698 Ch     "#"                                                                                       
000000004:   200        91 L     262 W      2698 Ch     "#"                                                                                       
000000009:   200        91 L     262 W      2698 Ch     "# Suite 300, San Francisco, California, 94105, USA."                                     
000000010:   200        91 L     262 W      2698 Ch     "#"                                                                                       
000000006:   200        91 L     262 W      2698 Ch     "# Attribution-Share Alike 3.0 License. To view a copy of this"                           
000000008:   200        91 L     262 W      2698 Ch     "# or send a letter to Creative Commons, 171 Second Street,"                              
000000121:   301        9 L      28 W       314 Ch      "tools"                                                                                   
000000287:   301        9 L      28 W       315 Ch      "assets"  

The WFUZZ gave us 2 options Tools and Assets so I[m gonna use the Tools options so we will add the /tools in the website 

<img width="1588" height="527" alt="image" src="https://github.com/user-attachments/assets/1b77510b-1a8b-4856-a7f1-481cf5ed895e" />

Vilnerabilities

Now it is time to start with a code inspection of the website with right click to see what we have. 

<img width="1189" height="655" alt="image" src="https://github.com/user-attachments/assets/1d1d8c51-f32a-4c9f-b808-dcf5c490b98f" />

we found something interesting from this website, usually when we have a code with doc= we could have a potential Path Transversal attack 

but firstly we will check of the page  check_if_exist.php?doc=keyboard.html exists.

And it exists

<img width="1383" height="754" alt="image" src="https://github.com/user-attachments/assets/104da5e8-44a9-46c3-9ccd-ea72dfda60a9" />

So we will use our Path Transversal attack by erasing the keyboad.html and adding ../../../../../../etc/passwd to navigate through the directories of the website and see if we find something with passwords or users from these directories. 

and we found something 

<img width="1562" height="407" alt="image" src="https://github.com/user-attachments/assets/d7862ba5-5334-4378-b339-da3362334158" />

Then we will use Ctrl U to order the information. 













