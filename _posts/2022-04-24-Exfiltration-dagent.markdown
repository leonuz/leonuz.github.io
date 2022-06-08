---
layout: post
title:  "Exfiltration d'agent (Network Challenge Writeup) -- Midnightflag INFEKTION 2022"
date:   2022-04-24
image: /midnightflag/midnightflag.png
---
<p class="intro"><span class="dropcap">M</span>idnightflag Team has elaborated a very entertaining and fun CTF named INFEKTION with a specific theme and even an excellent an spectacular <a href="https://www.youtube.com/watch?v=H_FT7KB74KU">Teaser</a> too. We are especially interested in the forensic and networking challenges, particularly this networking challenge seemed very interesting for dealing with information exfiltration issues.</p>

 
Challenges Description:

<figure>
        <img src="/assets/img/midnightflag/chall.png" alt="" />
        <figcaption>Description translate:An agent missed his exfiltration, but he sent us a message of great importance, find it!</figcaption>
</figure>

We were given a file to download ([source here](https://github.com/leonuz/CTFs/raw/main/stuff/transmission.pcapng))

We start opening the file using [Wireshark](https://www.wireshark.org/), after analyzing each protocol, we noticed strange hexadecimals characters in the ICMP data field.

<figure>
        <img src="/assets/img/midnightflag/icmp.png" alt="" />
        <figcaption>Strange data</figcaption>
</figure>

Proceed to copy the string and try to decode it using [CyberChef](https://gchq.github.io/CyberChef)

<figure>
        <img src="/assets/img/midnightflag/cybercheff.png" alt="" />
        <figcaption>Strange data</figcaption>
</figure>

After decoding "from HEX", cybercheff detects a ZIP file, we proceed to download it with the name `chall.zip` 

the file was encrypted, we use [fcrackzip](https://github.com/hyc/fcrackzip) to crack the file.

{%- highlight bash -%}
┌──(leonuz㉿sniper)-[~/Midnightflag/networking/Exfiltration_dAgent]
└─$ fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt chall.zip

PASSWORD FOUND!!!!: pw == G3ars0fwar
{%- endhighlight -%}  

After a few minutes, we find the password!  
then we open the zip file using that password...

{%- highlight bash -%}
┌──(leonuz㉿sniper)-[~/Midnightflag/networking/Exfiltration_dAgent]
└─$ 7z x download.zip

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 

Scanning the drive for archives:
1 file, 190 bytes (1 KiB)

Extracting archive: chall.zip
--
Path = chall.zip
Type = zip
Physical Size = 190


Enter password (will not be echoed):
Everything is Ok

Size:       28
Compressed: 190
{%- endhighlight -%}  

and finally, we have the flag inside the txt file!!

{%- highlight bash -%}
┌──(leonuz㉿sniper)-[~/Midnightflag/networking/Exfiltration_dAgent]
└─$ cat flag.txt
MCTF{g00d_0ld_1cmp_pr070c0l}                                             
{%- endhighlight -%}

#### MCTF{g00d_0ld_1cmp_pr070c0l}     

---

### Final Notes.


Thanks [MidnightFlag Team](https://midnightflag.fr/) for the oportunity of a excellent CTF.  
Give us more time for the next CTF please!!

For fun and knowledge, always think out of the box! :)

<figure>
        <img src="/assets/img/midnightflag/score.png" alt="" />
</figure>


---
