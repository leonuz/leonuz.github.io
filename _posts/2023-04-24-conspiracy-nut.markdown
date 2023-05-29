---
layout: post
title:  "Conspiracy Nut (Forensic Challenge Writeup) -- Space Heroes CTF 2023 "
date:   2023-04-24
image: /spaceh23/logo.png
---
<p class="intro"><span class="dropcap">T</span>he <a href="https://research.fit.edu/fitsec/">FITSEC</a> has once again organized a very fun event, full of many challenges (40) with many players (more than 500) for all levels. Personally, I really enjoyed the Forensic challenges (I managed to do them all) and this challenge in particular seemed very well planned and that it can be used to teach how to carry out a forensic investigation.</p>


#### Challenge Description: 

<figure>
        <img src="/assets/img/spaceh23/chall.png" alt="" />
        <figcaption>Conspiracy Nut</figcaption>
</figure>

Memory forensics is an essential type of cyber investigation that lets an investigator detect unauthorized and unusual activity on a computer or server. This involves using special software that takes a snapshot of the system’s memory and saves it as a file, also called a memory dump. The investigator can then move this file to another location and search it.

Getting the clues...

We know that the memory dump was done to a true believer in conspiracy theories. And he has withheld "evidence" that we must find.

We need to perform the extracting of digital artifacts (evidence) from volatile memory (RAM) obtained, for that we will start researching using the volatile memory extraction framework [Volatility3](https://github.com/volatilityfoundation/volatility3)  

First, we need to know what operating system the memdump is on. [windows.info](https://volatility3.readthedocs.io/en/latest/volatility3.plugins.windows.info.html)

<figure>
        <img src="/assets/img/spaceh23/info.png" alt="" />
        <figcaption>windows.info</figcaption>
</figure>

We now know that it is a Windows based operating system (Win7SP1x64) we start our analysis by obtaining the users and possible passwords.
[windows.hashdump](https://volatility3.readthedocs.io/en/latest/getting-started-windows-tutorial.html#windows-hashdump)
<figure>
        <img src="/assets/img/spaceh23/hashdump.png" alt="" />
        <figcaption>windows.hashdump</figcaption>
</figure>

Now we know that the username **tinfoil** is in the system, this is an important clue as we know that tinfoil is used to elaborate [tinfoil hat](https://dictionary.cambridge.org/us/dictionary/english/tinfoil-hat) which is associated when talking about people who believe in conspiracy theories, furthermore we know that the memory dump is from a laptop belonging to a conspiracy nut, so we already know the username of our "friend".

We also managed to crack (using [Jhon the Ripper](https://github.com/openwall/john))  the **tinfoil** password in case we need it later in the investigation.

{%- highlight bash -%}
┌──(leonuz㉿sniperhack)-[~/Downloads/ctf/space23]
└─$ john --format=NT hashes.txt
Using default input encoding: UTF-8
Loaded 3 password hashes with no different salts (NT [MD4 128/128 AVX 4x3])
Remaining 1 password hash
Warning: no OpenMP support for this hash type, consider --fork=4
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Warning: Only 5 candidates buffered for the current salt, minimum 12 needed for performance.
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
123456           (tinfoilxi)
1g 0:00:00:00 DONE 2/3 (2023-04-24 21:43) 33.33g/s 31733p/s 31733c/s 31733C/s 123456..pepper
Use the "--show --format=NT" options to display all of the cracked passwords reliably
Session completed.
{%- endhighlight -%}

Our next step is to get the list of the processes that were running, [windows.pslist](https://volatility3.readthedocs.io/en/latest/getting-started-windows-tutorial.html#windows-pslist)

<figure>
        <img src="/assets/img/spaceh23/pslist.png" alt="" />
        <figcaption>windows.pslist</figcaption>
</figure>

We can identify several suspicious processes, such as notepad.exe, firefox.exe, wmplayer.exe and others. 
But we must investigate further before we begin to study each process.

Several ways to perform the research, the one that made the most sense to us was to perform a [windows.filescan](https://volatility3.readthedocs.io/en/latest/volatility3.plugins.windows.filescan.html) to the whole dump and then search for the **tinfoil** username.  

<figure>
        <img src="/assets/img/spaceh23/filescan.png" alt="" />
        <figcaption>windows.filescan</figcaption>
</figure>

After a thorough search in all the **tinfoil** user space (Documents, AppData, Pictures, Downloads and more) we were able to get in "Desktop" a particularly interesting file (another clue)

<figure>
        <img src="/assets/img/spaceh23/desktop.png" alt="" />
        <figcaption>Desktop files</figcaption>
</figure>

Using the Volatility3 module called [windows.dumpfiles](https://volatility3.readthedocs.io/en/latest/volatility3.plugins.windows.dumpfiles.html) we proceed to extract this file, using for this purpose, the memory address obtained in the previous step (0x17eec5670)

<figure>
        <img src="/assets/img/spaceh23/dumpfiles.png" alt="" />
        <figcaption>windows.dumpfiles</figcaption>
</figure>

This new clue refers to a network connection, a web browser connection to http://57.135.219.202/IMG_2930.jpg  
For the next step we do a [windows.netscan](https://volatility3.readthedocs.io/en/v2.0.1/volatility3.plugins.windows.netscan.html) to search for network connections to that IP address 

<figure>
        <img src="/assets/img/spaceh23/netscan.png" alt="" />
        <figcaption>windows.netscan</figcaption>
</figure>

Now we know that **Firefox** is the application that we must investigate in depth, we begin to dump all the processes associated with **Firefox** in order to investigate each one of them. There are 12 processes in total identified with PID 1936, 2404, 1344, 2772, 1608, 800, 2532, 880, 2332, 1964, 3128, 3240. We use the module named [windows.memmap](https://volatility3.readthedocs.io/en/latest/volatility3.plugins.windows.memmap.html)

After some time we found that PID 880 contains the information we are looking for, the "evidence" that our "friend" preaching about.

{%- highlight bash -%}
┌──(leonuz㉿sniperhack)-[~/Downloads/ctf/space23]
└─$ python3 /home/leonuz/Downloads/repos/volA/volatility3/vol.py -f conspiracy_nut.dump -o memmap/ windows.memmap --dump --pid 880
Volatility 3 Framework 2.4.1
Progress:  100.00               PDB scanning finished
Virtual Physical        Size    Offset in File  File output

0x10000 0x4f579000      0x1000  0x0     pid.880.dmp
0x20000 0x4e3c1000      0x1000  0x1000  pid.880.dmp
0x21000 0x4ce32000      0x1000  0x2000  pid.880.dmp
0x30000 0x7aa11000      0x1000  0x3000  pid.880.dmp

REDACTED...
REDACTED...
REDACTED...

0xffffffd0a000  0x6000  0x1000  0x1d1e3000      pid.880.dmp
0xffffffd0b000  0x0     0x1000  0x1d1e4000      pid.880.dmp
0xffffffd0c000  0x2000  0x3000  0x1d1e5000      pid.880.dmp
0xffffffd0f000  0x8000  0x1000  0x1d1e8000      pid.880.dmp

┌──(leonuz㉿sniperhack)-[~/Downloads/ctf/space23]
└─$ ls -lah memmap
total 466M
drwxr-xr-x  2 leonuz leonuz 4.0K Apr 24 23:28 .
drwxr-xr-x 14 leonuz leonuz 4.0K Apr 24 23:27 ..
-rw-------  1 leonuz leonuz 466M Apr 24 23:28 pid.880.dmp
{%- endhighlight -%}

We use [Foremost](https://en.wikipedia.org/wiki/Foremost_(software)) to do the file carving.

<figure>
        <img src="/assets/img/spaceh23/foremost.png" alt="" />
        <figcaption>Foremost</figcaption>
</figure>

And then, opening the file 00019280.jpg we find the evidence that we need to put our "friend" in the Arkham Asylum!


<figure>
        <img src="/assets/img/spaceh23/flag.png" alt="" />
        <figcaption>flag</figcaption>
</figure>


#### shctf{m4D3_1n_A_h0LLYw00d_b45eM3NT}  


- - -
### NOTES:
This writeup was done as a compilation of an investigation to search for a digital artifact within a memory dump file. There are many ways to perform a forensic investigation, this is just one of them.

- - -
Thanks to [FITSEC, Florida Tech's Competitive Cybersecurity Team](https://research.fit.edu/fitsec/), for doing an excellent job! 
Congratulations to the entire team for a job well done. All the recognition you have received is well deserved and I look forward to seeing what you will do at the next CTF.

For fun and knowledge, always think out of the box! :)

<figure>
        <img src="/assets/img/spaceh23/score.png" alt="" />
</figure>
