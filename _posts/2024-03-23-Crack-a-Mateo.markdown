---
layout: post
title:  "Crack-a-Mateo (Crypto Challenge Writeup) -- JerseyCTF IV 2024 "
date:   2024-03-23
image: /jersey24/logo.png
---

<p class="intro"><span class="dropcap">O</span>nce again <a href="https://www.jerseyctf.com/">JerseyCTF</a> returns in its IV edition, with a total of 65 challenges so that more than 1600 players in more than 800 teams participated in this event, players of all categories from beginners to experts could enjoy 24 hours of entertainment. I chose this challenge for the write-up because of its simplicity. Many people are unaware that it is possible to create custom unique password candidates based on personal details like birthdays and names, so we must be cautious with the information we share on social networks, and most importantly, we must know how to choose our passwords very well!</p>

#### Challenge Description: 

<figure>
        <img src="/assets/img/jersey24/chall.png" alt="" />
        <figcaption>Crack-a-Mateo</figcaption>
</figure>

This challenge had two files to download, the first one was a screenshot of a message conversation.

<figure>
        <img src="/assets/img/jersey24/challenge-image.png" alt="" />
        <figcaption>challenge-image</figcaption>
</figure>

and the second one was an encrypted PDF file that can be downloaded ([here](https://github.com/leonuz/CTFs/raw/main/stuff/flag.pdf)) 

the challenge also had a free Hint available!!!

<figure>
        <img src="/assets/img/jersey24/hint.png" alt="" />
        <figcaption>Hint</figcaption>
</figure>

---

After reading the description of the challenge, the conversation of the message, and seeing the hint, it was more than evident that the solution to the problem would be to perform a dictionary attack, but a dictionary attack profiled towards the target.  

There are several tools to do this but none that I know of is as easy to implement as [CUPP - Common User Passwords Profiler](https://github.com/Mebus/cupp)  

CUPP is a remarkable tool that proves to be extremely useful in actual security tasks. It is a powerful resource for creating personalized password guesses by profiling individuals through interactive queries. Unlike standard wordlists such as rockyou.txt, CUPP specializes in mixing personal details like birthdays and names to formulate unique password combinations commonly devised by users.

We can run our tool and generate a dictionary based on all the information obtained from the conversation.

<figure>
        <img src="/assets/img/jersey24/cupp.png" alt="" />
        <figcaption>CUPP</figcaption>
</figure>

after a short time, it generated a dictionary called mateo.txt with 16592 words.

then, we use pdf2john to extract the hash of the PDF so we can crack it using [John](https://www.openwall.com/john/)

```
┌──(leonuz㉿sniperhack)-[~/…/ctf/jersey24/crypto/crack-a-mateo]
└─$ john --wordlist=mateo.txt pdf.hash
Using default input encoding: UTF-8
Loaded 1 password hash (PDF [MD5 SHA2 RC4/AES 32/64])
Cost 1 (revision) is 3 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
m3l14!@'#'       (flag.pdf)
1g 0:00:00:00 DONE (2024-03-24 10:33) 2.941g/s 29364p/s 29364c/s 29364C/s jennifer@*!..m3l14$@%
Use the "--show --format=PDF" options to display all of the cracked passwords reliably
Session completed.
```

John was able to decrypt the hash in a short time and obtain the password with which we were able to open the PDF.

<figure>
        <img src="/assets/img/jersey24/flag.png" alt="" />
        <figcaption>flag</figcaption>
</figure>


#### jctf{IMa_7aKe_I7_PER50nA11Y}
---
#### NOTE:
As a reminder when choosing a password, I always think of this [xkcd](https://xkcd.com/) comic that has been published for 13 years now!!!

<figure>
        <img src="https://imgs.xkcd.com/comics/password_strength.png" alt="" />
        <figcaption>password_strength</figcaption>
</figure>

[Explain Password Strength](https://www.explainxkcd.com/wiki/index.php/936:_Password_Strength)


- - -
Thanks to [JerseyCTF Team](https://www.jerseyctf.com), once again doing an excellent job, kudos for the whole team! 

For fun and knowledge, always think out of the box! :)

<figure>
        <img src="/assets/img/jersey24/score.png" alt="" />
</figure>



