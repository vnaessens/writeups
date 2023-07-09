https://tryhackme.com/room/vulnversity

From the source code, it seems the password for user "admin" is "sUp3rPaSs1".

`netcat 10.10.61.110 1337`

`Welcome! Please login as the admin!`

`username: admin`

`admin's password: sUp3rPaSs1`

`Not that easy :)`

`Goodbye!`

This is because of :
`def start(server):`

`        key = get_random_bytes(16)`

`        iv = get_random_bytes(16)`

`        send_message(server, 'Welcome! Please login as the admin!\n')`

`        send_message(server, 'username: ')`

`        username = server.recv(4096).decode().strip()`

`        send_message(server, username +"'s password: ")`

`        password = server.recv(4096).decode().strip()`

 `       message = 'access_username=' + username +'&password=' + password`

 `       if "admin&password=sUp3rPaSs1" in message:`
 
 `           send_message(server, 'Not that easy :)\nGoodbye!\n')`
 
 `       else:`
 
`            setup(server,username,password,key,iv)`

So, one cannot login with the good combination of user and password.

Let's have a look at function `setup(server,username,password,key,iv)`.

First, it creates a message = 'access_username=' + username +'&password=' + password

which is encoded by function encrypt_data(data,key,iv) using AES mode CBC and 16-byte padding style 'PKCS7'.

Then, the "leaked" ciphertext is printed and we are asked to enter a ciphertext.
The function decodes OUR ciphertext and compares it with "admin&password=sUp3rPaSs1".

So without knowing the key or the IV, which are random, we must change our encrypted input (which cannot be correct or else the program would have logged us off before) and provided a proper ciphertext for it to contains "admin&password=sUp3rPaSs1" when decoded.

Let's try to modify the ciphertext so that when it gets decrypted it contains the string "admin&password=sUp3rPaSs1".

Conveniently, the plaintext is made of three 16-byte blocks after padding :

access_username=

admin&password=s

Up3rPaSs1.......	(.... for padding)

The username is aligned with the start of the second block. An idea is to pass an incorrect username such as 'zdmin', then modify the ciphertext so that when it gets decrypted the second block starts with 'admin'.

Leaked ciphertext: 

9e90a8b302cf5f03484670a8e49c2da8	// access_username=

20010c20b8eca18336db215712d1971b	// admin&password=s

27a93a3ef62057d9d9c117bb0d027327	// Up3rPaSs1....... (.... for padding)

Based on the CBC-decoding schema, we can see that the last step of the decoding of the second block is a XOR with the ciphertext of the first block.
![alt-title](/TryHackMe/Flip_CBC.png)

It means that by changing the ciphertext first block (which contains "access_username=" i.e. no valuable data in terms of logging/identication), we can modify directly the decoding of the cipher second block where the start of the useful data (i.e. "admin&password") is located.

Let's note '?' the first byte of block2_cipher_decryption.

We know that :
? ^ 9e = 'z' ; ? = 'z'^9e = 7a^9e = e4.

What we want is :
e4^?? = 'a' ; ?? = 'a'^e4 = 61^e4 = 85 where '??' is the modified first byte of block1_ciphertext.

So, let's modify the first byte with '85' :
8590a8b302cf5f03484670a8e49c2da820010c20b8eca18336db215712d1971b27a93a3ef62057d9d9c117bb0d027327

The last block is left untouched and will be decoded as previously with the ramaining part of the good password "Up3rPaSs1".

THM{FliP_DaT_B1t_oR_G3t_Fl1pP3d}
