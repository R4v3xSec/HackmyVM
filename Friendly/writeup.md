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

we can also run a **ping* command to check what type of machine we will be working for.

This one is a Linux Machine and we see it because of the Time To Life that is equal to 64, most of the Windows Machines are in a range from 127 to 150 and the Linux from 60 to 84. 

<img width="612" height="181" alt="image" src="https://github.com/user-attachments/assets/a70f3cab-0f30-41f6-a976-e161dae4322b" />


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
At this point we will be eploring the vulnerabilities 


If you see the picture above you will notice that there is a website using the IP Address 
<img width="1379" height="851" alt="image" src="https://github.com/user-attachments/assets/7393b952-a20d-409a-ac56-f206befc2a3c" />

Very important this machine has the website in a server that runds Apache2.4, sometimes these type of servers run in php as you see there are two documents in php, so this will allow us to explore with the ftp command 

So we will try the following 

A- Type the command ftp + the IP Address

B- To connect via ftp we need to use it anonymous and it will asks us for a passwordjust hit enter

with the **dir** command we access the machine
<img width="756" height="371" alt="image" src="https://github.com/user-attachments/assets/6f2c6ecd-ae7b-480b-91da-1a57aa6a1948" />


Now we are going to use a website called Reverse Shell Generator to create our reverse Shell

From this website we will need to use our IP Address not the victim´s IP Address and the port that you like
Here the steps

1-Paste your Attack IP Address in the **IP /PORT** section 
<img width="1077" height="559" alt="image" src="https://github.com/user-attachments/assets/a65a9d6e-fbc5-4500-92b9-49c440ab0738" />


2-Scroll down from the left menu and select the option PHO PentestMonkey
<img width="1360" height="569" alt="image" src="https://github.com/user-attachments/assets/fb093103-8291-4227-9187-32f041924252" />

3-Copy the entire code 
<img width="1360" height="569" alt="image" src="https://github.com/user-attachments/assets/a952e039-d145-4369-8153-f3ef71b2a838" />

4-From a terminal inside the Friendly1 directory create a nano.php file in this example I will create a new one called login.php 
<img width="499" height="137" alt="image" src="https://github.com/user-attachments/assets/ed685bd6-8a86-43c3-b4f9-8f415d0ec1a6" />

5-Paste the code generated from ReverseShell, type Ctrl + o to save it, hit enter and Ctrl + X to close it

Now inside the Friendly1 Directory we have two documents login.pho and results.txt 
<img width="468" height="159" alt="image" src="https://github.com/user-attachments/assets/3d8dad2b-7091-4d48-87b1-c6a55f969b64" />

With this login.php file we will use it and upload it to the server from out ftp access.

To complete this we need to use the command **put** + the name of the file in this case login.php 
<img width="1572" height="389" alt="image" src="https://github.com/user-attachments/assets/b63529c3-39bd-42f6-8cdf-a2bb1157c629" />

Once this is uploaded to the server, we will need to start a conection for the file that we uploaded to listen to us

So we need to run the command **sudo nc -nlvp 444** and in paralel we need to add /login.php from the website 
<img width="459" height="144" alt="image" src="https://github.com/user-attachments/assets/5a6f197c-642b-4bce-ae34-d6a8acd6e271" />

<img width="1246" height="521" alt="image" src="https://github.com/user-attachments/assets/a6b1ce9a-1497-49e7-aff8-66aa9c0bf7b2" />


Once run the command and added login.php we can notice that now it is listening 

If we run whoami we will see out user that is **www-data**
<img width="880" height="279" alt="image" src="https://github.com/user-attachments/assets/f204286c-846e-4c91-ae7c-e2f6c0795b71" />


**Persistance**


So far we have vulnarated the machine however we need to make sure we dont loose the connection to it, so for this we will run some commands to keep persistance in the machine.

In some machines the vulnerabilities are more complex and if we loose this connection we need to start from scratch so for this example we will use the stty treatment in two parts 

 Part I

Run the command

- **script /dev/null -c bash**
Ctr + Z 
<img width="860" height="354" alt="image" src="https://github.com/user-attachments/assets/85edf863-0452-4f2c-b67a-66967354f777" />


Run the comamnd

-**stty raw -echo; fg** 

**reset xterm**

<img width="498" height="120" alt="image" src="https://github.com/user-attachments/assets/7e254b8c-8ff9-4083-8941-82df0285f935" />

**export SHELL=bash**

**export TERM=xterm**

<img width="419" height="105" alt="image" src="https://github.com/user-attachments/assets/7e9eddbe-05b0-48df-b2ef-05d6b2166c02" />

Now if we use Crtl + C or clear we dont loose the conection

Part II 

We will using Crontab -e (Crontab will help us to use commands in automatic for example each minute we will running a special command in this case we will use a reverseshell.

For this we need to go back to the revserse shell website and from the port we need to change it to the 445 and we need to look for the option **bash-i**
<img width="1257" height="674" alt="image" src="https://github.com/user-attachments/assets/bcd90343-ba2a-4697-af5f-a02bbad60f39" />

We coppy the code and paste it into our crontab file

using the command  * * * * * bash -c like this example then Crtl + O hit enter and Ctrl +X 
<img width="816" height="534" alt="image" src="https://github.com/user-attachments/assets/4371b7d6-b3ed-4a47-a4eb-141c70a6e9a6" />

To verify that our crontab command works you can run in a new terminal the same **sudo nc -nlvp 445**  that each 5 minutes it will be listening automatically 

<img width="674" height="197" alt="image" src="https://github.com/user-attachments/assets/ec96fa24-1266-4538-942b-bec98608e386" />


**Escalating Privileges** 

to escale priviliges we will be using the command **sudo -ls**

This tell us that the user www-data could be a potential root from the path **/usr/bin/vim**

In this scenario we we be using a website called GTFOBins 

As you see vim is the directory to become root so inside the website we will search for exploits in vim 

<img width="1092" height="473" alt="vim" src="https://github.com/user-attachments/assets/c2d45980-b8e4-4907-aad6-9e4dc9b542ec" />

Then scroll to sudo and you will find the command to run in this case

vim -c ':!/bin/sh' 

and we complete it using the entire command

sudo -u root vim -c ':!/bin/sh' hit enter and whoami to check what user we are 

www-data@friendly:/$ sudo -u root vim -c ':!/bin/sh'

 **whoami**
 
root



















