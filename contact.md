---
layout: default
title: Contact me
---

<div id="contact">
  <h1 class="pageTitle">Contact Me</h1>
  <div class="contactContent">
    <p class="intro">If for any reason you want to contact me, feel free to do so. I am always willing to help.</p>
    <p>For fun and knowledge, think outside the box!</p>
    <p><img src="{{ '/assets/img/mgreen.jpg' }}" alt=""></p>
  </div>
  <form action="https://formspree.io/f/mleavljz" method="POST">
    <label for="name">Name</label>
    <input type="text" id="name" name="name" class="full-width"><br>
    <label for="email">Email Address</label>
    <input type="email" id="email" name="_replyto" class="full-width"><br>
    <label for="message">Message</label>
    <textarea name="message" id="message" cols="30" rows="10" class="full-width"></textarea><br>
    <input type="submit" value="Send" class="button">
  </form>
</div>
