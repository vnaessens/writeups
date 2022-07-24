# Reversing ELF #

## Task 1 : Crackme1 ##

Workaround to upload missing `crackme` files on AttackBox :
1. Download task file
2. Download crackme1
3. Open it in Hxd.exe or any hexadecimal editor.
4. Select all, copy.
5. In AttackBox, open Sublime text editor, paste, save as crackme1.txt in /home/Downloads
6. `root@ip-10-10-22-54:~# xxd -r -p Downloads/crackme1.txt > crackme1`

`root@ip-10-10-22-54:~# ./crackme1`

`bash: ./crackme1: Permission denied`

`root@ip-10-10-22-54:~# chmod 777 crackme1`

`root@ip-10-10-22-54:~# ./crackme1`

`flag{not_that_kind_of_elf}`

## Task 2 : Crackme2 ##

Repeat steps 1-6 of Task 1 to upload and enable execution of crackme file.

The program prints "Usage : %s password\n".

Open file in Ida or any disassembler.

Look at function `main`. The password is in clear. When given the correct password, the programs calls function `giveFlag`.

Execute the program with argument `super_secret_password` to get the flag.

## Task 3 : Crackme3 ##

Repeat steps 1-6 of Task 1 to upload and enable execution of crackme file.

`file crackme3` -> ELF 32-bit LSB executable

`strings crackme3` -> One string stands out : "ZjByX3kwdXJfNWVjMG5kX2xlNTVvbl91bmJhc2U2NF80bGxfN2gzXzdoMW5nNQ=="

It looks like Base64 encoding -> f0r_y0ur_5ec0nd_le55on_unbase64_4ll_7h3_7h1ng5

## Task 4 : Crackme4 ##

Repeat steps 1-6 of Task 1 to upload and enable execution of crackme file.

`file crackme4 -> ELF 64-bit LSB executable`

`strings crackme4` -> One string stands out : "This time the string is hidden and we used strcmp"

`r2 -d crackme4`

`aa`

`afl`

`pdf @main` -> check the number of arguments ; exits if < 2 ; else calls sym.compare_pwd

`pdf @sym.compare_pwd` -> string comparison @0x004006d5

`exit`

Load program `crackme4` in debug with argument `password` : `r2 -d rarun2 program=crackme4 arg1=password`

Put a breakpoint where the string comparison is called : `db 0x004006d5`

And continue execution : `dc`

Print registries : `dr`

`px @rsi` -> password

`px @rdi` -> my_m0r3_secur3_pwd

## Task 5 : Crackme5 ##

Repeat steps 1-6 of Task 1 to upload and enable execution of crackme file.

Same as previous task :
- load the program with Radare2
- `aa`
- `afl`
- `pdf @main`
- put a breakpoint where `strcmp` is called
- `dc`
- enter a password when asked
- prints rsi and rdi with `px @rsi` and `px @rdi` when this breakpoint is reached.

=> OfdlDSA|3tXb32~X3tX@sX`4tXtz


## Task 6 : Crackme6 ##

Repeat steps 1-6 of Task 1 to upload and enable execution of crackme file.

`r2 -d crackme6`

`aa`

`afl`

`pdf @main`

Again, the program checks the number of arguments and exits if different than 2.

`pdf @sym.compare_pwd`

`pdf @sym.my_secure_test`

`my_secure_text` seems to implement a sort of string comparison with characters '1', '3', '3', '7', '_', 'p', 'w', 'd'.

=> 1337_pwd


## Task 7 : Crackme7 ##

Repeat steps 1-6 of Task 1 to upload and enable execution of crackme file.

`r2 -d crackme7`

`aa`

`afl`

`pdf @main`

`pdf @sym.giveFlag`

In function `main`, the number of user inputs read by scanf() is stored in EAX.
The program exits if EAX <> 1.

The value of the user choice is in `local_ch`.
There are 4 comparisons, the first three corresponding to the choice shown in the "menu" + 1 hidden choice `0x7a69 = 31337`.

If `local_ch == 31337` then `sym.giveFlag` is called displaying the flag.

Thus, enter 31337 when prompted by the program.

=> flag{much_reversing_very_ida_wow}


## Task 8 : Crackme8 ##

Repeat steps 1-6 of Task 1 to upload and enable execution of crackme file.

`r2 -d crackme8`

`aa`

`afl`

`pdf @main`

`pdf @sym.giveFlag`

The usage is : crackme8 password

Looking at `main`, the program checks the number of arguments, then calls `sym.giveFlag` if `atoi(password) == 0xcafef00d`.

Keep in mind, `atoi` returns a signed 32-bit integer, so 0xcafef00d is -889262067.

Thus, execute `crackme8 -889262067` to have `sym.giveFlag` called and the flag shown.
