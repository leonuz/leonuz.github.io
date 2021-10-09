---
layout: post
title:  "Router-Pwn (Writeup DEF CON Red Team Village CTF Quals)"
date:   2021-10-04
---
<a href="https://redteamvillage.io/ctf.html" target="_blank"><img src="/assets/img/cintillo.png" alt="Qries"></a>
<p class="intro"><span class="dropcap">L</span>ast August, the qualification roud for the DEF CON 29 Red Team Village CTF took place, it was an excellent event, with very well thought challenges and an impeccable organization. One of those challenges, called "Router-Pwn" was especially interesting, because solving this challenge requires knowledge of: networking, forensics, cracking and basic fundamentals of routing with Cisco IOS. </p>

We were given a pcap file (download source [here](https://github.com/leonuz/CTFs/raw/main/stuff/rtr-pwneip.pcap)), which made the challenge more interesting.

We start by analyzing the pcap file, to do this, we load the file in Wireshark and perform a Protocol Hierarchy Statistics. (Fig 1) 
 
(Wireshark -> Statistics -> Protocol Hierarchy)
<figure>
        <img src="/assets/img/wphs.png" alt="" />
        <figcaption>Fig 1 .- Wireshark Protocol Hierarchy Statistics</figcaption>
</figure>

We can identify several known protocols, such as TFTP, SNMP, Telnet and Cisco Host Standby Router Protocol (HSRP).  

Now let's get started.    

<figure>
        <img src="/assets/img/rp1.png" alt="" />
        <figcaption>Challenge 1</figcaption>
</figure>

We know for sure, that in the pcap file has some [HSRP](https://community.cisco.com/t5/networking-documents/hsrp-overview-and-basic-configuration/ta-p/3131590) comunication, and we know too that (thanks to this [twitt](https://twitter.com/_johnhammond/status/1246427342894968832?lang=en)) we can use pcap2john.py, to extract the hash for the encrypted HSRP channel and then pass it to a cracking tool (John the Ripper or Hashcat).

{%- highlight bash -%}
┌──(leonuz㉿sniper)-[~/CTFs/DEFCON29_RedTeamVillage_Quals/Router-Pwn]
└─$ python2.7 /usr/share/john/pcap2john.py rtr-pwneip.pcap > hsrp-hashes.txt                                          

┌──(leonuz㉿sniper)-[~/CTFs/DEFCON29_RedTeamVillage_Quals/Router-Pwn]
└─$ cat hsrp-hashes.txt                                                                                                                                               
$hsrp$000010050f64010000000000000000000a0d2565041c010000000a0d25640000000000000000000000000000000000000000$f19b33e766d742c90c031324baeddfc7
$hsrp$000010050f64010000000000000000000a0d2565041c010000000a0d25640000000000000000000000000000000000000000$f19b33e766d742c90c031324baeddfc7
$hsrp$000010050f64010000000000000000000a0d2565041c010000000a0d25640000000000000000000000000000000000000000$f19b33e766d742c90c031324baeddfc7
REDACTED

┌──(leonuz㉿sniper)-[~/CTFs/DEFCON29_RedTeamVillage_Quals/Router-Pwn]
└─$ john --wordlist=/usr/share/wordlists/rockyou.txt hsrp-hashes.txt 
Using default input encoding: UTF-8
Loaded 1 password hash (hsrp, "MD5 authentication" HSRP, HSRPv2, VRRP, GLBP [MD5 32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
catch22$         (?)
1g 0:00:00:09 DONE (2021-08-06 20:51) 0.1002g/s 927961p/s 927961c/s 927961C/s cathy525..catahermosa*
Warning: passwords printed above might not be all those cracked
Use the "--show" option to display all of the cracked passwords reliably
Session completed

┌──(㉿sniperkal)-[~/CTFs/DEFCON29_RedTeamVillage_Quals/Router-Pwn]
└─$ john --show hashes.txt 
?:catch22$

{%- endhighlight -%}
And we have the first flag!
``` 
Flag #1: catch22$
``` 

<figure>
        <img src="/assets/img/rp23.png" alt="" />
        <figcaption>Challenges 2 and 3</figcaption>
</figure>

From the pcap file we were able to extract the router configuration file, specifically the startup-config (Fig 2).

(Wireshark -> File -> Export Objets -> TFTP -> Save)

<figure>
        <img src="/assets/img/tftp.png" alt="" />
        <figcaption>Fig 2.- Wiresharks Export TFTP Objets List</figcaption>
</figure>

We reviewed the startup-config and found the next lines particularly interesting after reading [more info](https://networklessons.com/cisco/ccie-routing-switching/hsrp-hot-standby-routing-protocol) about HSRP, which is what challenges 2 and 3 are all about. 

{%- highlight bash -%}
!
interface GigabitEthernet1
 ip address 10.13.37.100 255.255.255.0
 standby 1 ip 10.13.37.101
 standby 1 timers 5 15
 standby 1 preempt
 standby 1 authentication md5 key-string 7 094F4F1D1A0D45404F
 standby 1 name thebruceleeband
 negotiation auto
 no mop enabled
!
{%- endhighlight -%}

By studying this Cisco document [Configuring HSRP](https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst3560/software/release/12-1_19_ea1/configuration/guide/3560scg/swhsrp.pdf), we know that the above lines, contain information to obtain the "HSRP group" and the "HSRP virtual address" whitch are the flags from the challenges 2 and 3. 

``` 
Flag #2: thebruceleeband
Flag #3: 10.13.37.101
``` 
**Note:** Another way to obtain the HSRP Virtual Address, is directly from the pcap file in Wireshark, selecting any "HSRP" packet and analyzing the content (Fig 3).

<figure>
        <img src="/assets/img/vip.png" alt="" />
        <figcaption>Fig 3 .- HSRP Virtul IP Address</figcaption>
</figure>
<figure>
        <img src="/assets/img/rp45.png" alt="" />
        <figcaption>Challenges 4 and 5</figcaption>
</figure>

In these challenges we are asked to get the "enable password" and the "enable secret". We have the config-startup file of the router, so we can get the hashes and crack them. For this, we follow this great guide from InfosecMatter called [Cisco Password Cracking and Decrypting Guide](https://www.infosecmatter.com/cisco-password-cracking-and-decrypting-guide/).

The following table shows the information necessary to crack the hashes of the Cisco IOS-based network devices.

|Cisco Password | Crackability | John the Ripper | Hashcat | 
| ------------- | ------------ | --------------- | ------- |
|type 0 | instant | n/a | n/a |
|type 7 | instant | n/a | n/a |
|type 4 | easy | –format=Raw-SHA256 | -m 5700 |
|type 5 | medium | --format=md5crypt | -m 500 |
|type 8 | hard | --format=pbkdf2-hmac-sha256 | -m 9200|
|type 9 | very hard | --format=scrypt | -m 9300 |
 


Near the top of the config-startup file we have:


``` bash
!
enable secret 8 $8$uazfxavvplFUvE$Ms16BRtAFP9cdjqf240xdHgcddNxS.V39W6zy7jz3tc
enable password 7 06140A24404C001E031E0103
!
```

for cracking the "enable password" we know this tool [ciscot7](https://github.com/theevilbit/ciscot7), we install it, run it and get the decrypted password which is our flag 4.


``` bash
┌──(leonuz㉿sniper)-[~/CTFs/DEFCON29_RedTeamVillage_Quals/Router-Pwn/crack]
└─$ git clone https://github.com/theevilbit/ciscot7.git
Cloning into 'ciscot7'...
remote: Enumerating objects: 19, done.
remote: Counting objects: 100% (4/4), done.
remote: Compressing objects: 100% (4/4), done.
remote: Total 19 (delta 0), reused 0 (delta 0), pack-reused 15
Receiving objects: 100% (19/19), 6.75 KiB | 863.00 KiB/s, done.
Resolving deltas: 100% (5/5), done.

┌──(leonuz㉿sniper)-[~/CTFs/DEFCON29_RedTeamVillage_Quals/Router-Pwn/crack]
└─$ cd ciscot7/

┌──(leonuz㉿sniper)-[~/CTFs/DEFCON29_RedTeamVillage_Quals/Router-Pwn/crack]
└─$ python2.7 ciscot7.py -d -p 06140A24404C001E031E0103
Decrypted password: reelbigfish
```
```bash
Flag # 4: reelbigfish
```
Cracking the "secret password" is not so easy, but it is not so difficult, we see in the table above, that its crackability is "Hard" (password type 8) but from the description of the challenge we know that all passwords are in rockyou wordlist, so it will be just a matter of time.

To start cracking it we first need to "assemble" the hash, we do that as follows:


```bash
┌──(leonuz㉿sniper)-[~/CTFs/DEFCON29_RedTeamVillage_Quals/Router-Pwn/crack]
└─$ echo 'enablesecret:$8$uazfxavvplFUvE$Ms16BRtAFP9cdjqf240xdHgcddNxS.V39W6zy7jz3tc' > enablesecret.hash
```

and then, knowing the type of encryption used by the Cisco type 8 password, we proceed to crack it using Jhon the ripper.


```bash
┌──(leonuz㉿sniper)-[~/CTFs/DEFCON29_RedTeamVillage_Quals/Router-Pwn/crack]
└─$ john --format=pbkdf2-hmac-sha256 --fork=8 --wordlist=/usr/share/wordlists/rockyou.txt enablesecret.hash                          
Using default input encoding: UTF-8
Loaded 1 password hash (PBKDF2-HMAC-SHA256 [PBKDF2-SHA256 256/256 AVX2 8x])
Cost 1 (iteration count) is 20000 for all loaded hashes
Node numbers 1-8 of 8 (fork)
Press 'q' or Ctrl-C to abort, almost any other key for status
lessthanjake12   (enablesecret)
1 1g 0:00:01:18 DONE (2021-10-06 23:07) 0.01274g/s 79.95p/s 79.95c/s 79.95C/s booney..180486
Waiting for 7 children to terminate
7 0g 0:00:01:18 DONE (2021-10-06 23:07) 0g/s 81.12p/s 81.12c/s 81.12C/s janito..chubs
2 0g 0:00:01:18 DONE (2021-10-06 23:07) 0g/s 79.01p/s 79.01c/s 79.01C/s iloverudy..branca
4 0g 0:00:01:18 DONE (2021-10-06 23:07) 0g/s 79.38p/s 79.38c/s 79.38C/s boobies2..170994
6 0g 0:00:01:18 DONE (2021-10-06 23:07) 0g/s 78.53p/s 78.53c/s 78.53C/s morgan4..iloveyou02
8 0g 0:00:01:18 DONE (2021-10-06 23:07) 0g/s 78.98p/s 78.98c/s 78.98C/s ilovejenny..booty69
5 0g 0:00:01:18 DONE (2021-10-06 23:07) 0g/s 79.74p/s 79.74c/s 79.74C/s 160386..vocalist
3 0g 0:00:01:18 DONE (2021-10-06 23:07) 0g/s 79.34p/s 79.34c/s 79.34C/s boodie..17101992
Use the "--show --format=PBKDF2-HMAC-SHA256" options to display all of the cracked passwords reliably
Session completed

┌──(leonuz㉿sniper)-[~/CTFs/DEFCON29_RedTeamVillage_Quals/Router-Pwn/crack]
└─$ john --show --format=PBKDF2-HMAC-SHA256 enablesecret.hash 
enablesecret:lessthanjake12

1 password hash cracked, 0 left
```
```bash
Flag # 5: lessthanjake12
```

<figure>
        <img src="/assets/img/rp67.png" alt="" />
        <figcaption>Challenges 6 and 7</figcaption>
</figure>

From the config-startup file we have:

```bash
!
username zzyzzx privilege 15 secret 5 $1$qdkB$TJp7PCYE.5UWYce9GyElJ0
username rayhan privilege 14 secret 9 $9$sBkPUtoEWx6hxl$cU3orT6Y9btuI98Abtyy4ROKbplFBnVMnDEuXI9gF1E
```
Let's use the table above to know the parameters that we are going to pass to Jhon the Ripper to crack the passwords we are asked for. We start assembling the hashes for each one.


```bash
echo 'zzyzzx:$1$qdkB$TJp7PCYE.5UWYce9GyElJ0' > zzyzzx.hash

echo 'rayhan:$9$sBkPUtoEWx6hxl$cU3orT6Y9btuI98Abtyy4ROKbplFBnVMnDEuXI9gF1E' > rayhan.hash
```

having the hashes we proceed to crack the passwords using Jhon the ripper


```bash
┌──(leonuz㉿sniper)-[~/CTFs/DEFCON29_RedTeamVillage_Quals/Router-Pwn/crack]
└─$ john --format=md5crypt --fork=8 --wordlist=/usr/share/wordlists/rockyou.txt zzyzzx.hash                                          
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 256/256 AVX2 8x3])
Node numbers 1-8 of 8 (fork)
Press 'q' or Ctrl-C to abort, almost any other key for status
mustardplug      (zzyzzx)
5 1g 0:00:00:45 DONE (2021-10-07 00:14) 0.02206g/s 14407p/s 14407c/s 14407C/s musti1938..mustaqueem
2 0g 0:00:01:00 DONE (2021-10-07 00:14) 0g/s 14365p/s 14365c/s 14365C/s jidpond1..jicleb53
4 0g 0:00:01:00 DONE (2021-10-07 00:14) 0g/s 14404p/s 14404c/s 14404C/s jg122791..jg041257
3 0g 0:00:01:00 DONE (2021-10-07 00:14) 0g/s 14364p/s 14364c/s 14364C/s jiggan1..jigg123
8 0g 0:00:01:00 DONE (2021-10-07 00:14) 0g/s 14468p/s 14468c/s 14468C/s jerjef25..jerinazel
7 0g 0:00:01:00 DONE (2021-10-07 00:14) 0g/s 14550p/s 14550c/s 14550C/s jeanchristophe..jeanashel
1 0g 0:00:01:00 DONE (2021-10-07 00:14) 0g/s 15151p/s 15151c/s 15151C/s ilovepop13..ilovepittaway
6 0g 0:00:01:00 DONE (2021-10-07 00:14) 0g/s 14461p/s 14461c/s 14461C/s jerrytao..jerrymarcelo
Waiting for 7 children to terminate
Use the "--show" option to display all of the cracked passwords reliably
Session completed

┌──(leonuz㉿sniper)-[~/CTFs/DEFCON29_RedTeamVillage_Quals/Router-Pwn/crack]
└─$ john --format=md5crypt --show zzyzzx.hash                                          
zzyzzx:mustardplug

1 password hash cracked, 0 left

```
```bash
Flag # 6: mustardplug
```

```bash
┌──(leonuz㉿sniper)-[~/CTFs/DEFCON29_RedTeamVillage_Quals/Router-Pwn/crack]
└─$ john --format=scrypt --fork=8 --wordlist=/usr/share/wordlists/rockyou.txt rayhan.hash
Using default input encoding: UTF-8
Loaded 1 password hash (scrypt [Salsa20/8 128/128 AVX])
Cost 1 (N) is 16384 for all loaded hashes
Cost 2 (r) is 1 for all loaded hashes
Cost 3 (p) is 1 for all loaded hashes
Node numbers 1-8 of 8 (fork)
Press 'q' or Ctrl-C to abort, almost any other key for status
thespecials      (rayhan)
8 1g 0:00:12:12 DONE (2021-10-06 18:38) 0.001364g/s 51.18p/s 51.18c/s 51.18C/s thespecials
2 0g 0:00:13:00 DONE (2021-10-06 18:38) 0g/s 51.26p/s 51.26c/s 51.26C/s FUCKL0V3
7 0g 0:00:13:00 DONE (2021-10-06 18:38) 0g/s 53.39p/s 53.39c/s 53.39C/s spike16
4 0g 0:00:13:00 DONE (2021-10-06 18:38) 0g/s 50.86p/s 50.86c/s 50.86C/s angeles07
6 0g 0:00:13:00 DONE (2021-10-06 18:38) 0g/s 50.88p/s 50.88c/s 50.88C/s andre4ever
3 0g 0:00:13:00 DONE (2021-10-06 18:38) 0g/s 50.93p/s 50.93c/s 50.93C/s alomdra
5 0g 0:00:13:00 DONE (2021-10-06 18:38) 0g/s 51.08p/s 51.08c/s 51.08C/s Scorpions
1 0g 0:00:13:00 DONE (2021-10-06 18:38) 0g/s 53.44p/s 53.44c/s 53.44C/s solesita
Waiting for 7 children to terminate
Use the "--show --format=scrypt" options to display all of the cracked passwords reliably
Session completed

┌──(leonuz㉿sniper)-[~/CTFs/DEFCON29_RedTeamVillage_Quals/Router-Pwn/crack]
└─$ john --show --format=scrypt rayhan.hash                                                                                            
rayhan:thespecials

1 password hash cracked, 0 left

```
```bash
Flag # 7: thespecials
```

