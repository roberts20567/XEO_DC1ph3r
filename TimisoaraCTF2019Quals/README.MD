# Timisoara CTF 2019 Quals
## Sections
### Crypto section
[Baby Crypto](#baby_crypto)<br>
[Proof of work](#proof_of_work)<br>
[Alien Alphabet](#alien_alphabet)<br>
[Strange cipher](#strange_cipher)
### Exploit section
[Hiss hiss python](#hiss_hiss_python)<br>
[Swag](#swag)
### Forensics section
[Deleted file](#deleted_file)<br>
[Strange image](#strange_image)<br>
[Tri-color QR](#tri-color_qr)
### Misc section
[Read the rules](#read_the_rules)
### Reverse engineering section
[Baby Rev](#baby_rev)<br>
[Easy Rev](#easy_rev)<br>
[Strange jump](#strange_jump)
### Web section
[Not so empty website](#not_so_empty_website)<br>
[Admin panel](#admin_panel)
<br><br>
## Crypto section
<a name="baby_crypto"></a>
### Baby Crypto
#### Description:
>This file seems... odd

#### Solution:
by mmzmihai

Even if the text is strange we recognize the pattern indicated by the braces and the caps lock chars.

Encoded text:
-> Otil bw amm gwc uilm qb! 
Emtkwum bw bpm ewvlmznct ewztl wn kzgxbwozixpg! 
Pmzm qa gwcz zmeizl:
 BQUKBN{Rctqca_Kimaiz_e0ctl_j3_xzwcl}
 
The main hint is the text file name, "caesar.txt"
We used an online tool to decode the message, finding out 
that the correct key is 8, as 'a' has to turn into letter 'i'.

Decoded text:
-> Glad to see you made it! 
Welcome to the wonderful world of cryptography! 
Here is your reward:
flag: `TIMCTF{Julius_Caesar_w0uld_b3_proud}`


<br>
<a name="proof_of_work"></a>

### Proof of work
#### Description:
>While developing an anti-bot system we thought of a system to test if the users are indeed human. You need to enter a string whose SHA256 has the last 7 digits 0. As this hash is secure you need to use some processing power, thus denying spam. Sort of like mining bitcoin.

>nc 89.38.208.143 21021

#### Solution:
by assassinduke

This challenge was pretty straight forward it only required to brute force text that was hashed with sha256 until a regex found the 6 ending zeros.

Code:
```python
import string
import hashlib
import re
ALLOWED_CHARACTERS = string.printable
NUMBER_OF_CHARACTERS = len(ALLOWED_CHARACTERS)

def characterToIndex(char):
    return ALLOWED_CHARACTERS.index(char)

def indexToCharacter(index):
    if NUMBER_OF_CHARACTERS <= index:
        raise ValueError("Index out of range.")
    else:
        return ALLOWED_CHARACTERS[index]

def next(string):
    """ Get next sequence of characters.
    Treats characters as numbers (0-255). Function tries to increment
    character at the first position. If it fails, new character is
    added to the back of the list.
    It's basically a number with base = 256.
    :param string: A list of characters (can be empty).
    :type string: list
    :return: Next list of characters in the sequence
    :rettype: list
    """
    if len(string) <= 0:
        string.append(indexToCharacter(0))
    else:
        string[0] = indexToCharacter((characterToIndex(string[0]) + 1) % NUMBER_OF_CHARACTERS)
        if characterToIndex(string[0]) is 0:
            return list(string[0]) + next(string[1:])
    return string

def main():
    sequence = list()
    while True:
        sequence = next(sequence)
        str1 = ''.join(sequence)
        hash = hashlib.sha256(str1.encode('utf-8')).hexdigest()
        if re.match('.*[0]{7}$', hash):
            print("{} = {}".format(str1,hash))
            break

if __name__ == "__main__":
    main()
```

flag: `TIMCTF{9e13449f334ded947431aa5001c2e9ab429ab5ddf880f416fe352a96eb2af122}`

<br>
<a name="alien_alphabet"></a>

### Alien Alphabet
#### Description:
>I found this strange text. It is written in some strange alphabet. Can you decode it?

#### Solution:
by roberts2056

<span style="color:red">**(OLD FLAG)**</span>
Being a crypto challenge, there isn't any hidden file or text, so all we can do is to make a frequency analysis on the text.

Or (as we did it), take a paper and a pen and find anything useful.

[![Image Solving](https://raw.githubusercontent.com/roberts20567/XEO_DC1ph3r/master/TimisoaraCTF2019Quals/imagesolv.jpg "Image Solving")](https://raw.githubusercontent.com/roberts20567/XEO_DC1ph3r/master/TimisoaraCTF2019Quals/imagesolv.jpg "Image Solving")
For example, the last word contains 6 letters followed by a point, that is also found at the end of the word. The first letter repeats in positions 1 and 5, so that must be the flag header (TIMCTF)
and from there you can start to guess other words like 'the', 'Hello' etc...

flag: `TIMCTF{ETAOIN,SHRDLU}`
<br>
<a name="strange_cipher"></a>

### Strange cipher
#### Description:
>I have found this strange encryption service. Can you decode it?

>nc 89.38.208.143 21022

#### Solution:
by roberts2056

We are welcomed with a hex string that we need to decode which has been encoded with an unknown algorithm.
The good news is that we can input 256 strings and the program will encode them for us.
After some trial and error I finally managed to understand what the script does.
Consider you have the ascii table. Split it into 4 columns with 32 rows.
The scripts sets a random start value to one row. Then it calculates the next values as follows:
x = start value

| Ascii values 32-63 | Ascii values 64-95  | Ascii values 96-127  |
| ------------ | ------------ | ------------ |
|  x | y= x + 1  | z = x + 2  |
| x1 = x + 8  | y1 = x1 + 1  | z1 = x1 + 2  |

.
.
.
.

>for example if 0 is set as start point we have 0 = 0x00, P=0x01, p=0x02, since P and p are on the same row with 0

After a few tries I got the correct bytes and the flag.

flag: `TIMCTF{Y0u_really_make_A_diff3rence}`

> Note: Because DEL wasn't included in the script, at some point it gets inconsistent but you get the correct char after a few tries

<br><br>

## Exploit section
<a name="hiss_hiss_python"></a>

### Hiss hiss python
#### Description:
> This snake likes to h1ss at its input.

> nc 89.38.208.144 11113

#### Solution:
by roberts2056

The vulnerability is in `input()` function. While `raw_input()` takes the argument as a specified type, `input()` evaluates the argument, so we can easily write valid python code as input:

Payload:  `sys.stdout.write(open("flag.txt").readline())`

>source - https://github.com/Vinnyz/Root-me-challenge-App-Script/blob/master/Python%20-%20input()

flag: `TIMCTF{h1ss_h1ss_shell}`
<br>
<a name="swag"></a>

### Swag
#### Description:
>The server only lets hackers in, not script kiddies.

>nc 89.38.208.144 11111

#### Solution:
by NClaudiu

At first the problem looks like a legit exploit challenge, but simply searching for "TIMCTF" in the compiled file finds the flag.

flag:`TIMCTF{1_am_th3_c00kie_m0nsta}`

<br><br>
## Forensics section
<a name="deleted_file"></a>

### Deleted file
#### Description:
>Help! I accidentally deleted a photo! Can you recover it for me please?

> http://timisoaractf.ro/10MB.img

> Non-standard flag format

#### Solution:
by roberts2056

If we look in the Journal, there seems to be a file named tctf.png, which doesn't appear to be anywhere, but if we open the .img with a hex editor we can find the image at the end of the file (we can easily distinguish the file hence it's PNG file signature)
[![Hex deleted image](https://raw.githubusercontent.com/roberts20567/XEO_DC1ph3r/master/TimisoaraCTF2019Quals/deleted_file_hxd.png "Hex deleted image")](https://raw.githubusercontent.com/roberts20567/XEO_DC1ph3r/master/TimisoaraCTF2019Quals/deleted_file_hxd.png "Hex deleted image")
After we recover the file, the flag is "painted" in the image.
[![TCTF](https://raw.githubusercontent.com/roberts20567/XEO_DC1ph3r/master/TimisoaraCTF2019Quals/tctf.png "TCTF")](https://raw.githubusercontent.com/roberts20567/XEO_DC1ph3r/master/TimisoaraCTF2019Quals/tctf.png "TCTF")

flag: `flag{I_s33_the_uns33n}`
<br>
<a name="strange_image"></a>

### Strange image
#### Description:
>I received this "image" in my mailbox today, but I can not open it, as if it was corrupted or something. Can you fix it and tell me if it has any hidden meaning?

>Note: if you think you "fixed" it and it does not open, try using a few different photo viewers.

#### Solution:
by roberts2056

At first, it appears to be a corrupt image, but if we xor the first 4 bytes with the normal file signature of a PNG we get 0x7a each time.
Next step is to make a python script to xor each byte of the image with 0x7a, and after that we get a half-corrupted valid (everything but the crc) image.
If then we 'strings'  the file, we immediately see a weird line
[![Hatz](https://raw.githubusercontent.com/roberts20567/XEO_DC1ph3r/master/TimisoaraCTF2019Quals/snip_hatz.PNG "Hatz")](https://raw.githubusercontent.com/roberts20567/XEO_DC1ph3r/master/TimisoaraCTF2019Quals/snip_hatz.PNG "Hatz")

Considering that the first 6 letters are uppercase, and W repeats in positions 1 and 5, that must be the header of the flag, TIMCTF.
The flag encoding is as follows: `ord(x) = ord(x) + 3`, so the decoding: 
`ord(x) = ord(x) - 3`. Write that in python.

flag: `TIMCTF{Brav0_j0hnule!}` (right bracket is missing in the final output)
<br>
<a name="tri-color_qr"></a>

### Tri-color QR
#### Description:
>I stumbled upon this strange QR code which seems to be in a new format. Can you help me decode it?

#### Solution:
by roberts2056

After googling about the rgb QR a while, we found a couple of articles about the technique, and it seems that it was utilizing the color channels to overlap the QR codes.
So we found this nice online utility from our dear friends 
> www.dcode.fr --> RGB channels.

After separating the channels and scanning the QR codes, we find the first 3 parts of the flag, but there seems to be 1 more part missing.
`TIMCTF{Th1s_is_A_4_part_*********`

While I was working on strange image, I found a write-up for a similar challenge in which the guy used pngcheck on the file, and found out that there is more information after the IEND chunk.
So I tested that on the QR image and 
[![QR_CODE_CHECK](https://raw.githubusercontent.com/roberts20567/XEO_DC1ph3r/master/TimisoaraCTF2019Quals/tri-color_pcheck.PNG "QR_CODE_CHECK")](https://raw.githubusercontent.com/roberts20567/XEO_DC1ph3r/master/TimisoaraCTF2019Quals/tri-color_pcheck.PNG "QR_CODE_CHECK")

Voila! there was a zip file after the PNG image in which I found the last part of the flag.

flag: `TIMCTF{Th1s_is_A_4_part_flag}`
<br><br>
## Misc section
<a name="read_the_rules"></a>

### Read the rules
#### Description:
>Have you read them?

#### Solution:
by all

Just so you know we read the rules :)

flag: `TIMCTF{sometext}`
<br><br>
## Reverse engineering section
<a name="baby_rev"></a>

### Baby Rev
#### Description:
>This program asks me for the flag, but i don't know it!

#### Solution:
by assassinduke

Another easy challenge to start off the category, for this one opening the file with a reverse engineering app like IDA revealed the code in plane text in the output function


flag: `TIMCTF{Wh0_know5_a5m_kn0ws_R3V}`

<br>
<a name="easy_rev"></a>

### Easy Rev
#### Description:
>Like the last one, but harder

#### Solution:
by assassinduke

This is where it started to get interesting. At a quick glance IDA revealed only assembly code that was pretty difficult to understand in a short amount of time.
But Ghidra's decompiler offered some easier to understand code that, in the end was based of on only one formula and the encrypted text that was stored and easily found by Ghidra and IDA.

[![Ez Update](https://raw.githubusercontent.com/roberts20567/XEO_DC1ph3r/master/TimisoaraCTF2019Quals/ezrev.png "Ez Update")](https://raw.githubusercontent.com/roberts20567/XEO_DC1ph3r/master/TimisoaraCTF2019Quals/ezrev.png "Ez Update")

After a bit of thinking the code was doing a Caesar cipher using '-a' as a key and a bit of obfuscation


flag: `TIMCTF{rotated13flag}`

<br>
<a name="strange_jump"></a>

### Strange jump
#### Description:
>This application likes to jump!

#### Solution:
by roberts2056

The program seems to be a simple decription script, besides all the junk functions imported.
But if we search for strings with Ghidra, there seems to be a "flag" string found in memory which conflicts with the instructions.

[![Flag Jump](https://raw.githubusercontent.com/roberts20567/XEO_DC1ph3r/master/TimisoaraCTF2019Quals/flag_jump.PNG "Flag Jump")](https://raw.githubusercontent.com/roberts20567/XEO_DC1ph3r/master/TimisoaraCTF2019Quals/flag_jump.PNG "Flag Jump")

If we 'strings' the file, we can actually see the flag string and before that, what appears to be a base64 encoded string

[![Base64 Jump](https://raw.githubusercontent.com/roberts20567/XEO_DC1ph3r/master/TimisoaraCTF2019Quals/jump_base64.PNG "Base64 Jump")](https://raw.githubusercontent.com/roberts20567/XEO_DC1ph3r/master/TimisoaraCTF2019Quals/jump_base64.PNG "Base64 Jump")

After we decode it, we get the flag.

flag: `TIMCTF{deC3pt1ve_ExceP0ti0n_h4ndLer}`

## Web section
<a name="not_so_empty_website"></a>

### Not so empty website
#### Description:
>This website looks empty, but trust me, it is not!

>http://89.38.208.143:20001/

#### Solution:
by roberts2056
Opening the page with inspect element, reveals the flag in a comment in `<body>`

flag: `TIMCTF{D0_not_b3_superfic1al}`
<br>
<a name="admin_panel"></a>

### Admin panel
#### Description:
>Your job is to hack this admin panel and login.

>http://89.38.208.143:20003/

#### Solution:
by NClaudiu

The first thing I thought when I saw the page was to do SQL Injection.
Strangely, it worked.
Feeding the Password input with `' or ''='` gets us the flag.

flag: `TIMCTF{SqL_1nj3ct1on_1s_b4ck_1n_town}`
