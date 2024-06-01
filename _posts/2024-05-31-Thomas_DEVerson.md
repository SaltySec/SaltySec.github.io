---
title: Thomas DEVerson
date: 2024-05-31 21:45:00 -0500
category: CTF
tags: Web Security CTF Nahamcon flask cookies
description: CTF writeup from May 2024 NahamCon CTF
image: /assets/img/Thomas_DEVerson/backup.png
---

## About
Thomas DEVerson was a Medium box that was part of the Nahamcon CTF in May 2024. It was 1 of only a handful of boxes that I completed, and certainly the hardest. The box was tricky, but I learned an absolute ton from it. All in all this CTF was awesome and hugely fun to take part in. I saved and even backed up a ton of screenshots for this writeup. Unfortunately, not all of those screenshots made it to my `PC Backup` folder before doing a complete format of both my drive and reinstall of windows. So this writeup is going to be a little more text heavy, and lighter on the screens. I will do my best to be better about this moving forward (who doesn't love more screenshots!). I am actively working on tracking down some more screens from this from other folk who participated in this CTF. 

## Initial Enumeration
After nagvigating to the site, I was greeted by a login screen, and text that said stuff about it being 1797, and using modern technology to hide plans from the federalists. There was also a spot to enter a username, but no password field. I tried a handful of random usernames off the rip and the all send you to the directory /messages, with the text **"Hey there federalist... Keep out of our buisness!"** This let me know that there was some kind of user authenitcation happening behind the scenes without the use of a password, potentially using cookies. Looking at the pages source, I saw a comment that was left that said something about the directory `/backup`
![screenshot of source](/assets/img/Thomas_DEVerson/source.png)
*source of page*

Navigating to the /backup directory shows this file:
![backup](/assets/img/Thomas_DEVerson/backup.png)
*screen of /backup directory*

We learn a decent handfull of things from this page... Come to find out later on, we learn *everything* we need from this page, but it took my a little longer to get to that spot while I was actually participating in the CTF. Hindsight is 20/20 ;)
Firstly - This is a flask server.
Secondly - There are 3 allowed users. Jefferson, Madison, and Burr. 
Lastly - there is some code that is running *(was ran... hindsight)* on the server, that is appending a timestamp to the `secret key`. 

Secret key took a little bit of digging into for me to figure out.. But eventually came across [this page from hacktricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/flask) What I learned was that `flask` servers can use a secret key and store it inside of, or use it to encrypt a session cookie, so that without the secrey key, the cookie can not be altered successfully. Thankfully for be (bad for Thomas Deverson), I have the secret key. 

Now came the (way too long) process of figuring out how to use it. I looked at the sessionID using the inspect function in firefox, and saw that it had the same format as a JSON web token. I took the cookie and slapped it into [JWT](https://jwt.io/) and got the result of `{"user": "guest"}`. Now I now that for my authentication, this will have to get changed to be one of the authorized users, then encrypted using the secret key. 

## The exploitation
Now that I know what I have to do, it's just a matter of doing it. Using what I found from [hacktricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/flask), I knew that I could use flask unsign to write my own JSON data to a cookie, and sign it with the secret key. All of the information that I needed was laid out right in from of me, and I just had to figure out how to make it do what I wanted it to. It started very slow at first and I was trying things like `flask unsign --sign --cookie "{'name': 'Jefferson'}" --secret "THE_REYNOLDS_PAMPHLET"`,`flask unsign --sign --cookie "{'name': 'Jefferson'}" --secret "THE_REYNOLDS_PAMPHLET-{f}"` etc, before realizeing that I needed to.. y'know.. solve for the f string. Like I said.. Hindsight, haha. So thats what I did but.. I did it as if I was generating the secret cookie, rather than the computer that was executing the code. This then took me a while, but also ended up uncovering some stuff that I was going to need for the end result. The first thing I did, since I thought that the secret cookie would be updating every muinute, was come up with a python scrip that would solve for the F string, and then ouput the entire secret key. When I say came up with a python script, I mean go to /backlup, ctrl+c, crtl+v and add `print (app.secret_key)` to the bottom of it. So now that I had everything set up, I could drop that into the `cookie` parameter in burp fairly quickly. Quicker than the 60 seconds I though I had with the rolling key haha. Heres how that setup looked. 
![burp](/assets/img/Thomas_DEVerson/burp.png)
*the beginning of my decent into madness*

Obviously *(ooooobviousllyyyy.. insert eye roll)* that didn't work, and I was still being met with Thomas Deverson calling me a federalist, which mean that my authentication failed. Something wasn't right here and now I had to track down what. My first step was going back to the original session cookie. Lookig at it, and when it was last accessed, I found that it was in a different time zone. I shifted my secret key to fit that time zone, and still nada. I got stuck here for quite a while. Took a break, got some food, cleared my head. When I came back and started thinking about it with a clear and level head, I got it almost right away. If you've made it this far into reading this, I'm sure you already realize what I've done wrong. 

*I am not generating the secret key.. The server is.* But the server isn't continually executing this code to generate a new secret key. Although, I suppose it could via a cron job, looking at the web UI I didn't think that was incredibly likely... Bit how could I figure out when the server was last initalized to execute the code?.. Well, how about the /stats tab that shows up on the home page, and shows server uptime. 
![status](/assets/img/Thomas_DEVerson/status.png)
*facepalm.. Knew it was there for a reson.*

Chcuked that day into a good old google, got the date that it was in the format that I needed (the year was... you guessed it, 1797.. they told me), slapped that bad boy into the secrey spot of the flask unsign command and **BOOM**: still nothing. 

I shifted the timestamp to account for timezone. Then it worked perfectly. Let me into /messages and I was greeted with a beautiful, beautiful flag. 

I learned a TON from this box in particular. JSON web tokens, Flask cookies, so on and so forth. It was extremely challenging for me, and took me almost 2 days to finish. Although I stated to lose my mind a bit in the end there, that feeling of getting a flag never gets old. Pretty sure that my lady heard my shout "Yes! F#$*(&$) finally!!" from all the way across the house when I finished this one. 

Thanks everyone, and hope you enjoyed. If you have any feedback, feel free to reach out to me on [X/twitter](https://www.x.com/SaltySec_). Thanks for your time!

``01110011 01110100 01100001 01111001 00100000 01100011 01110101 01110010 01101001 01101111 01110101 01110011 00101100 00100000 01110011 01110100 01100001 01111001 00100000 01110110 01101001 01100111 01101001 01101100 01100001 01101110 01110100.``


``- SaltySec``