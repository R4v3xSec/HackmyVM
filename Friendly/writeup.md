Hello welcome to the Friendly writeups!

From this machine we will be covering the full hacking cycle:

- Reconnaissance
- Exploring vulnerabilities
- Keeping persistence
- Elevating privileges
<img width="685" height="83" alt="image" src="https://github.com/user-attachments/assets/960e1aa8-c0b0-4fc1-a32f-0ef0511c6559" />

**Reconnaissence**

Before start with recconaissance we need to verify our interface and the IP Address of the Friendly1 machine

Step# 1 To verify the interface we will need to run the command **"ip a"** previously **"ifconfig"**  command, as you can see our nterface is eth0.

<img width="540" height="50" alt="image" src="https://github.com/user-attachments/assets/66d139ea-3822-4daa-9004-ac9ae50717e3" />





Step# 2 Now that we have the interface it is time to search for the Firnedly1 machine IP Address, using the command 

**sudo arp-scan -I eth0 --localnet**

This will display the IP Addresses for the devices using ARP in our local network. 

In this case de Friendly IP Address is the "192.168.15.14"

<img width="402" height="74" alt="image" src="https://github.com/user-attachments/assets/3c550fb0-da09-45fe-945e-6fc2fd1fc152" />
<img width="641" height="32" alt="image" src="https://github.com/user-attachments/assets/e476446e-0f85-408c-ad06-6cf9b6d51f06" />





Step # 3 we need to create a new directory in our Desktop with the name of Friendly1

To create a new Directory we will be using the **mkdir** command + the name of the machine

So from a new terminal we will create our new directory called Friendly1

<img width="586" height="190" alt="image" src="https://github.com/user-attachments/assets/a4eb90f0-47cc-42d9-828c-71ede6487a05" />

To continue with the scan art we will need to run a **nmap** command

The command will be **sudo nmap -p- -sS -sC -sV --n-min-rate 5000 -n -Pn -vvv 192.168.1514 -oN results.txt** 

We need to run the nmap command from a terminal inside Friendly1

<img width="1142" height="613" alt="image" src="https://github.com/user-attachments/assets/61f1c558-c8fe-4abf-8f27-a3b20f41ac7f" />

After the scan we found two ports open the 21/TCP and the 80/TCP , we will be using the 21 as it may allow ftp connection

<img width="733" height="276" alt="image" src="https://github.com/user-attachments/assets/9aa19d92-46d9-42a5-b0cc-be4fb9e748df" />




**Vulnerabilities**


