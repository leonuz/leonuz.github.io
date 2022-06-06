---
layout: post
title:  "Jason's Web Tarot 2 (Web Challenge Writeup) -- BCACTF 3.0 2022"
date:   2022-06-06
image: /bcactf22/logo.png
---
<p class="intro"><span class="dropcap">B</span>CACTF 2022 took place on June 3 to 6, 2022. It was 72 hours of challenges and exceptional support from the organizers.  
We chose this WEB challenge for the writeup because it's a very good practical example of why it is necessary to use a good shared-key in all applications.</p>


#### Challenge Description: 

<figure>
        <img src="/assets/img/bcactf22/chall.png" alt="" />
        <figcaption>Web Challenge</figcaption>
</figure>

We started loading the page.  

<figure>
        <img src="/assets/img/bcactf22/web1.png" alt="" />
        <figcaption>Main site</figcaption>
</figure>

If we hit the "Pull Card" Button, the tarot card changes.

<figure>
        <img src="/assets/img/bcactf22/web2.png" alt="" />
        <figcaption>Card Changes</figcaption>
</figure>

Let's start intercepting traffic using [Burpsuite](https://portswigger.net/burp)  

<figure>
        <img src="/assets/img/bcactf22/Burp-captura_token.png" alt="" />
        <figcaption>Trafic Intercepted</figcaption>
</figure>

We can see a token, a [JSON Web Token](https://jwt.io/introduction)  

#### But, What is JSON Web Token?  
JSON Web Token (JWT) is an open standard [(RFC 7519)](https://datatracker.ietf.org/doc/html/rfc7519) that defines a compact and self-contained way for securely transmitting information between parties as a JSON object. This information can be verified and trusted because it is digitally signed. JWTs can be signed using a secret (with the HMAC algorithm) or a public/private key pair using RSA or ECDSA.

Many tools are available to decode the header and payload parts in a JWT. We use a tool in linux called [jwt_tool](https://github.com/ticarpi/jwt_tool)


<figure>
        <img src="/assets/img/bcactf22/jwt_tool.png" alt="" />
        <figcaption>jwt_tool</figcaption>
</figure>

We need to become subscribed users to the application, to do this, we need to change (in the JWT) the value of the `"isSubscriber"` variable from its current value, `False`, to a new one, `True`. 

But when changing the payload content, without modifying the signature, we get an error (Invalid Signature). Therefore, in order to modify the payload it is necessary to know the key with which the JWT was signed.

We can see that the token uses *HS256* as a Signing Algorithm
 
#### HS256  
Hash-based Message Authentication Code (HMAC) is an algorithm that combines a certain payload with a secret using a cryptographic hash function like SHA-256. The result is a code that can be used to verify a message only if both the generating and verifying parties know the secret. In other words, HMACs allow messages to be verified through shared secrets.

*At this point we will assume that the key used for the JWT signature is very weak and therefore it is possible to crack the JWT key.*

### Brute Forcing a HS256 JSON Web Token
As secure as HS256 is, especially when implemented the right way, brute-forcing a JSON web token signed with small and medium sized shared-secrets using HS256 is still very possible.

Using [Jonh the Ripper](https://www.openwall.com/john/) (or any other tools) we can find the secret key of a HS256 JSON Web token.

We have the original token:

{%- highlight bash -%}
┌──(leonuz㉿sniperhack)-[~/Downloads/ctf/bcactf]
└─$ cat jwt.hash
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc1N1YnNjcmliZXIiOmZhbHNlLCJpYXQiOjE2NTQzMDA1ODd9.FeOzxBet7HJ3ry34my5cDjMnTY2zoRVjPlWAyiAHLS0
{%- endhighlight -%}

and we cracked it!

{%- highlight bash -%}
┌──(leonuz㉿sniperhack)-[~/Downloads/ctf/bcactf]
└─$ john jwt.hash
Using default input encoding: UTF-8
Loaded 1 password hash (HMAC-SHA256 [password is key, SHA256 128/128 AVX 4x])
Will run 4 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
Proceeding with incremental:ASCII
38r4             (?)
1g 0:00:02:50 DONE 3/3 (2022-06-04 18:15) 0.005862g/s 5855Kp/s 5855Kc/s 5855KC/s no3k..3c2E
Use the "--show" option to display all of the cracked passwords reliably
Session completed.

{%- endhighlight -%}

Now we know that `38r4` its the key for signing the JWT

Know we can change the paylod part in the JWT using [jwt.io](https://jwt.io), and signing the JWT with the found key.  


<figure>
        <img src="/assets/img/bcactf22/jwt-io.png" alt="" />
        <figcaption>New JWT token</figcaption>
</figure>


Now we have a new token

{%- highlight bash -%}
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc1N1YnNjcmliZXIiOnRydWUsImlhdCI6MTY1NDMwMDU4N30.e2O2Ph8cRm-DEFGz3sbcIhvnGbJ_9jpfWQvcqNK4RdM
{%- endhighlight -%}

Now, all that remains is to substitute the new token in the request and send the new request. 
We use the function ["Repeatear" of Burpsuit](https://portswigger.net/burp/documentation/desktop/tools/repeater/using) to do that.

<figure>
        <img src="/assets/img/bcactf22/Burp-new.png" alt="" />
        <figcaption>Send new Token and get the Flag</figcaption>
</figure>


And we get the flag!  

<figure>
        <img src="/assets/img/bcactf22/flag.png" alt="" />
        <figcaption>Flag</figcaption>
</figure>


#### bcactf{hm@c_256_yeeeeah_24u9402}

- - -
## Final Notes.  

Many developers are unaware of the importance of knowing the guidelines of the protocols they implement, The JSON Web Algorithms (JWA) defined in [RFC 7518](https://datatracker.ietf.org/doc/html/rfc7518) in statement 3.2 tells us: A key of the same size as the hash output (for instance, 256 bits for "HS256") or larger MUST be used with this algorithm. 
As a rule, make sure to pick a shared-key as long as the length of the hash. 
For HS256 that would be a 256-bit key (or 32 bytes) minimum!!


- - -
Thanks [BCACTF](https://www.bcactf.com/) for this great CTF, congratulations to the whole team!.  

For fun and knowledge, always think out of the box! :)

<figure>
        <img src="/assets/img/bcactf22/cert.png" alt="" />
</figure>
