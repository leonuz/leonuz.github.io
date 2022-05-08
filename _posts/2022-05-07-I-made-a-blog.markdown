---
layout: post
title:  "I made a Blog! (Web Challenge Writeup) -- EZ-CTF 2022"
date:   2022-05-07
image: /cafe22/cafe.png
---
<p class="intro"><span class="dropcap">E</span>Z-CTF 2022 (The first CTF competition using a new and innovative platform called <a href="https://www.linkedin.com/company/ctf-cafe/">CTF Cafe</a>) took place on May 4 and 5, 2022. It was 48 hours of all kinds of well-crafted challenges and exceptional support from the organizers.  
We chose this WEB challenge for the writeup as we can show one of the most common exploitation techniques widely used all over the world.</p>


#### Challenge Description: 

<figure>
        <img src="/assets/img/cafe22/chall.png" alt="" />
        <figcaption>Web Challenge</figcaption>
</figure>

Give us a hint:  

<figure>
        <img src="/assets/img/cafe22/hint.png" alt="" />
        <figcaption>Hint</figcaption>
</figure>

We started by doing a site reconnaissance.  

<figure>
        <img src="/assets/img/cafe22/mainsite.png" alt="" />
        <figcaption>Main site</figcaption>
</figure>

We inspect robots.txt, and this is its content:  

<figure>
        <img src="/assets/img/cafe22/robots.png" alt="" />
        <figcaption>robots.txt</figcaption>
</figure>

let's go to that restricted address:

<figure>
        <img src="/assets/img/cafe22/flag_filtered.png" alt="" />
        <figcaption>flag.php(filtered)</figcaption>
</figure>

So, the application detected the entry, and the filter give us `“How do you filter you coffe”` on page.    
At this point we know that, in order to get the flag, we need bypass the protection filter, and for that we need some techniques to recover the flag.  

Let's keep on digging...  

If we done a web app security assessment, found that the site may be vulnerable to a Local File Inclusion [LFI](https://www.aptive.co.uk/blog/local-file-inclusion-lfi-testing/) vulnerability.    

*LFI vulnerabilities are typically easy to identify and exploit. Any script that includes a file from a web server is a good candidate for further LFI testing.*  

<figure>
        <img src="/assets/img/cafe22/blog.png" alt="" />
        <figcaption>Blog</figcaption>
</figure>

Let's try to show the content of the /etc/passwd file.  

<figure>
        <img src="/assets/img/cafe22/lfi.png" alt="" />
        <figcaption>LFI</figcaption>
</figure>

Bingo!!! A successful exploitation of an LFI vulnerability in a web application!!  

To try to recover the flag we need a technique known as [`PHP Wrapper`](https://brightsec.com/blog/local-file-inclusion-lfi/#lfi-prevention)  

#### PHP Wrappers  
LFI vulnerabilities usually give attackers read-only access to sensitive data, granted from the host server. There are, however, ways to turn this read-only access into a fully compromised host. This type of attack is called Remote Code Execution (RCE). 
Attackers create RCE vulnerabilities by combining an LFI vulnerability with PHP wrappers.  

A wrapper is an entity that surrounds another entity (in this case – code). The wrapper can contain functionality added to the
original code. PHP uses built-in wrappers, which are implemented alongside file system functions. Attackers use this native
functionality of PHP to add vulnerabilities into wrappers.  

#### Commonly used wrappers:  

- PHP filter – provides access to a local file system. Attackers use this filter to read PHP files containing source code,  
typically for the purpose of identifying sensitive information, including credentials.  
- PHP ZIP – this wrapper was designed to manipulate files compressed into a ZIP format. However, its behavior enables attackers 
to create a malicious ZIP file and upload it to the server.  

To try to successfully exploit the LFI we use the payload described here [https://book.hacktricks.xyz/pentesting-web/file-inclusion#lfi-rfi-using-php-wrappers](https://book.hacktricks.xyz/pentesting-web/file-inclusion#lfi-rfi-using-php-wrappers)    
`page=php://filter/convert.base64-encode/resource=flag.php`

This payload forces PHP to base64 encode the file before it is used or rendered in the response. Now we replace page parameter value with above-mentioned payload and check output.  

<figure>
        <img src="/assets/img/cafe22/flag64.png" alt="" />
        <figcaption>flag.php(base64 encode)</figcaption>
</figure>

This strings: `PD9waHAKCWVjaG8gJ0hvdyBkbyB5b3UgZmlsdGVyIHlvdXIgY29mZmVlPyc7ICAgIAoJLy8gRVotQ1RGe0xGSV8xU18zWn0KPz4K`  
it's the base64 representation of the content of `flag.php`.  
let's decode it.. 

{%- highlight bash -%}
leonuz@sniper:~$ echo "PD9waHAKCWVjaG8gJ0hvdyBkbyB5b3UgZmlsdGVyIHlvdXIgY29mZmVlPyc7ICAgIAoJLy8gRVotQ1RGe0xGSV8xU18zWn0KPz4K" | base64 -d > flag.php
leonuz@sniper:~$ cat flag.php
<?php
        echo 'How do you filter your coffee?';
        // EZ-CTF{LFI_1S_3Z}
?>

{%- endhighlight -%}

And we get the flag!  

#### EZ-CTF{LFI_1S_3Z}  

- - -

Thanks [CTF Cafe](https://www.linkedin.com/company/ctf-cafe/) for the excellent EZ-CTF, congratulations to the whole team!.  

For fun and knowledge, always think out of the box! :)

<figure>
        <img src="/assets/img/cafe22/cert.png" alt="" />
</figure>
