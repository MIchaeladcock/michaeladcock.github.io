---
layout: post
title: THM RootMe
excerpt_separator: <!--more-->
---
<img src="/img/rootme/0.head.png">
<!--more--> 

<a href="https://tryhackme.com/room/rrootme" target="_blank" > Click here to access this box on tryHackMe.com</a>
<hr>

<p> This box simulates how a local file upload web form that isn’t properly sanitized and a miconfiguard user permission can lead to a complete system take over . In this write up I demonstrate how I would identify and abuse these mistakes. </p>    

<H1> ENUMERATION:</H1>

<hr>

<p> Start by enumerating  ports with a NMAP Scan.</p>
<p>Nmap reports that we have port 22 OpenSSH ver. 7.6 and port 80 Apache 2.4.29 open. Let’s open a web browser and see what's on port 80</p>
<img src="/img/rootme/1.nmap-scan.png">

<p>Not much here… let’s check the page source.</p>
<img src="/img/rootme/2.defualt-webpage.png">

<p>Not a lot in the source code either. There’s basic HTML and CSS with a small javascript function. Let’s see if there are any hidden subdirectories by useing gobuster and common wordlist.</p>
<img src="/img/rootme/4.gobuster-common.png">

<p>There’s a directory named panel and uploads. When we navigate there we are presented with a file upload form.</p>
<img src="/img/rootme/5.gobuster-panel.png">

<p>So we know we’re running Linux Ubuntu on Apache. So I’m thinking we should try and upload a php reverse shell payload and execute it from the upload page. </p>
<img src="/img/rootme/6.uploads-page.png">

<H1> Gaining a foothold:</H1>
<hr>
<p>First let’s test to see if the file extension .php is allowed.</p>
<img src="/img/rootme/7.test-php">
<img src="/img/rootme/8.test-upload.png">

<p>My Pourtguese isn’t very good but I believe this says file extention php isn’t allowed. Lol</p>
<img src="/img/rootme/9.test-upload-fail.png">

<p>Let’s intercept this request in Burp Suite and fuzz the file extension. </p>
<img src="/img/rootme/10.Burp-request.png">

<p>I captured the upload post request and sent it to intruder. </p>
<img src="/img/rootme/11.Burp-intruder.png">

<p>First let’s try some basic php extension payloads.</p>
<img src="/img/rootme/12.Burp-payloads.png">

<p>Let's start the attack and see what comes back. It appears that there's a lot of file extensions are allowed. .php5 is a common one - let’s try a payload with that extension. </p>
<img src="/img/rootme/13.files-success.png">

<p>Here I’m using a reverse shell php code from PentestMonkey. If you use this script all you really need to change is the ip address to reflect your machine's IP and change the extension to .php5. I’ll use port 4444 to setup a listener next</p>
<img src="/img/rootme/15.pentsetMonky-rev-shell.png">

<p>Here I setup the listener. </p>
<img src="/img/rootme/14.nc-listner.png">

<p>Upload the php reverse shell code.</p>
<img src="/img/rootme/16.payload-upload.png">

<p>I was listening on the wrong port so I got an error when I tried to execute the file in the uploads folder.</p>
<img src="/img/rootme/17.worng-port.png.png">

<p>Let’s change our port to 4444 and try again... and we have a shell.</p>
<img src="/img/rootme/18.first-shell.png">

<h1> Privilege Escalation:  </h1>
<hr>
<p>As expected we dropped in as www-data. We’ll need to enumerate to find an privesc path.</p>
<img src="/img/rootme/19.whoami.png">

<p>It appears that we only have one other user other than root. I suspect we’ll need to move laterally to the rootme user before we can move the root user.Let’s keep enumerating. First I’m going for the easy wins by password huning, checking suid, guid and contab </p>
<img src="/img/rootme/20.users.png">

<H1>Gaining root </H1>
<hr>
<p>While checking the SUID the python binary was set.</p>
<img src="/img/rootme/21.python-setuid.png">


<p>Easy privesc. Just run this command from the /usr/bin directory. The “-p” retains  the SUID bit permissions  set and does not drop the elevated privileges ( in our case root). And elevates the shells permissions to root . </p>
<img src="/img/rootme/22.root-esclation..png">


<p>cat the root flag and the box is done.</p>
<img src="/img/rootme/23.root.txt.png">
