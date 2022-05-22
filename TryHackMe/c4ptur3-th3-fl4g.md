https://tryhackme.com/room/c4ptur3th3fl4g

Useful sites :

[Aperisolve](http://aperisolve.fr/)

[dCode](https://www.dcode.fr/)

### Translation & Shifting

#### c4n y0u c4p7u23 7h3 f149?

It is plain and simple l33t.

#### 01101100 01100101 01110100 ...

The 0s and 1s point out to binary encoding.
Convert the numbers to characters following ASCII encoding.

#### MJQXGZJTGIQGS4ZAON2XAZLSEBRW63LNN5XCA2LOEBBVIRRHOM======

Only uppercase letters, digits and ending with ====== ? It must be Base32 encoding.

#### RWFjaCBCYXNlNjQgZGlnaXQgcmVwcmVzZW50cyBleGFjdGx5IDYgYml0cyBvZiBkYXRhLg==

Sequence of digits and letters (both lowercase and uppercase) ending with == ? It must be Base64 encoding.

#### 68 65 78 61 64 65 63 69 6d 61 6c 20 6f 72 20 62 61 73 65 31 36 3f

It looks like a serie of hexadecimal numbers.
Convert these numbers to characters following ASCII encoding.

#### Ebgngr zr 13 cynprf!

It's [ROT-13](https://www.dcode.fr/chiffre-rot-13) encoded.

#### *@F DA:? >6 C:89E C@F?5 323J C:89E C@F?5 Wcf E:>6DX

It's [ROT-47](https://www.dcode.fr/chiffre-rot-47) encoded.

#### - . .-.. . -.-. --- -- -- ..- -. .. -.-. .- - .. --- -. . -. -.-. --- -.. .. -. --.

The dashes and dots point out to Morse encoding.

#### 85 110 112 97 99 107 32 116 104 105 115 32 66 67 68

It looks a lot like ASCII-values of letters.

#### LS0tLS0gLi0tLS0gLi0tLS0gLS0tLS0gLS0tLS0gLi0t(...)=

Very long sequence of digits and letters (both lowercase and uppercase) ending with a '=' ?

It must be Base64-encoded.

It turns into a long list of dots and dashes (i.e. Morse encoding) :

----- .---- .---- ----- ----- .---- .---- -----
----- .---- .---- ----- ----- .---- ----- .----
----- ----- .---- ----- ----- ----- ----- -----
----- .---- .---- ----- ----- ----- ----- -----

(...)

01100110
01100101 
00100000 
01100000 
01011111 
01100000 
00100000 
(...)

Let's ASCII decode these binary-encoded numbers :

```fe `_` ``e bh ``d ba `_h hf `_f `_` ba ``e `_c `_d ``d ba hf ba hg `_d ``e ba ``e ``c `_d hh `_f `_d `_` ``c ce ce ce```

This looks like ROT-47 encoding.

76 101 116 39 115 32 109 97 107 101 32 116 104 105 115 32 97 32 98 105 116 32 116 114 105 99 107 105 101 114 46 46 46

Finally, we get a list of values that are in the ASCII-range of letters.

Solution `Let's make this a bit trickier...`

### Spectograms
Download the task file and open it in [Sonic Visualizer](https://sonicvisualiser.org/download.html).

### Steganography

Use http://aperisolve.fr/

### Security through obscurity

Use http://aperisolve.fr/
