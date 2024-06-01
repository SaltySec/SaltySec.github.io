---
title: Packer Paradox
date: 2024-05-21 21:07:00 -0500
category: Malware Analysis
tags: Malware Analysis windows IDA UPX CTF
description: Malware Analysis writeup from May 2024 Cyber Sentinel DoD CTF
image: /assets/img/Packers_paradox/Banner.png
---


## **The Backstory**


This challenge is from a Cyber Sentinel CTF that I participated in, that was hosted by Correlation One and sponsored by the DoD. The CTF was a ton of fun, and I learned a lot from it. I especially had a lot of fun with the 2 OSINT challenges, but I'll get to doing a write up on those at a later date. Out of roughly 3000 people I placed 259th, which while not great, felt pretty good for being the first CTF that I participated in. There were sections with easy, medium, and hard challenges for Web Application testing, Network Enumeration and penetration, Malware Reverse Engineering/Analysis, OSINT, and Forensic Analysis. On to this one.


## **The initial Analysis**


This challenge was called "Packer Paradox", and was the *easy* challenge from the Malware Reverse Engineering section. Each of the challenges had "backstory" to support what our objective was on it, but unfortunately I can't recall what it was, nor can I find them anymore.. Now I know to take screenshots as I'm going so I don't lose the data I need. Anyways, The first thing that I did (after creating a snapshot of my Clean FLARE VM, unzipping the file, and disarming it by adding a second file extension) was check the files md5 hash using the md5 hash drop down from the flare vm's context menu, and upload that to virus total to see if it has been flagged as being malicious previously.  
![screenshot of md5](/assets/img/Packers_paradox/md5.png)
*md5 hash and context menu option*


Virus total came back with 35/73 sources marking this as malicious... This isn't really a necessary step for a CTF, but malware analysis is a fairly new thing to me, and getting the reps in is probably a good thing.
![virus total image](/assets/img/Packers_paradox/virustotal.png)
*virus total report*


My next step was loading the EXE into PE - Bear to snoop around for anything interesting.. and I couldn't find anything here. The magic byte was 4D 5A, which tells me that it is in fact a PE file, but there wasn't anything else that was revealing anywhere else, like ``imports`` or ``resources``.. From here I thought that it might be time to open it with IDA, and decompile the PE file. Before doing this I removed the ``.dud`` file extension that I added to the end of ``Packer_Paradox.exe`` to disarm it, and now the EXE was live. In my experience, sometimes the imports and resources in IDA can get screwed up if you leave the .exe disarmed while trying to decompile it. Then something interesting happened. When I went to decompile the executable, I got an error saying that the file's ``imports`` section seems to have been destroyed, and that it could be due to a ``packer``. Alarm bells went off, due to a ``packer`` on ``Packer Paradox``, which sounds very CTF-esque. Unfortunately I had *no* clue what a "packer" was.


![packed](/assets/img/Packers_paradox/packed.png)
*IDA popup*
## **The Packer**
After a couple of minutes researching, I found that a ``Packer`` is a tool that's used to essentially compress PE/EXE files. From what I could tell, they're not used very often in above ground, kosher, clean programs, but more often used by malware developers in an attempt to obfuscate their code and make it harder to reverse engineer. Through my research I found a tool called ``UPX`` or ``Universal Packer for eXecutables``, and figured that this could be used to *unpack* my packed malware. Closed IDA, opened a cmd prompt, and tried this ``upx -d Packer_Paradox.exe``. This was the result:


![upx](/assets/img/Packers_paradox/upx-d.png)
*upx -d Packer_Paradox.exe*
## **The Solution**
Now let's go into IDA again. This time it acted much closer to expected, and all of the imports are showing up as they should. And there were "a lot" of them. I say a lot in quotes, because I don't really know if it is a lot or not, because I'm still pretty green to this. All I knew is that I saw some imports that looked like they had the potential to be malicious. Now that the code seems to be primarily deobfuscated, I decided to run strings on it, to see if there are any plain-text strings that it decided to spit out that made any sort of sense. And sure enough..
![flag](/assets/img/Packers_paradox/paradox.png)
*The Flag*
The flag was right there in plain text. Turned out to be a pretty simple one. I never really ended up learning what the malware was trying to do, or if it was trying to ping a C2, or really anything about it. But, that's the glory of CTF's, sometimes they're just easier than they seem. Moving forward, I'm really excited to share some more difficult and in depth malware analysis with you all! Despite this one being pretty easy, I still learned a bunch, and am thankful to Correlation 1 for Hosting the CTF.


Thanks everyone, and I hope you enjoyed. If you have any feedback, feel free to reach out to me on [X/twitter](https://www.x.com/SaltySec_). Thanks for your time!


``01110011 01110100 01100001 01111001 00100000 01100011 01110101 01110010 01101001 01101111 01110101 01110011 00101100 00100000 01110011 01110100 01100001 01111001 00100000 01110110 01101001 01100111 01101001 01101100 01100001 01101110 01110100.``




``- SaltySec``

