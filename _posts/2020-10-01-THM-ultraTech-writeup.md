---
layout: post
title: THM UltraTech Write-up
excerpt_separator: <!--more-->
---
<img src="/img/ultratech/0.png"/>
<!--more-->



<p>To start I’m gonna run a simple nmap scan against the host IP just on the top 1000 ports.   </p>
<p> The results show ports 21, 22 and 8081 being open </p>
<img src="/img/ultratech/1.nmap-scan-1.png"/>

<p>Let's enumerate these 3 ports for version, service and check the host OS </p>
<img src="/img/ultratech/2.nmap-scan-2.png"/>

<p>The results show port 21 FTP vsftpd 3.0.3, port 22 OpenSSH 7.6p1 and port 8081 http Node.js.</p>
<img src="/img/ultratech/3.namp-scan-3.png"/>

<p>Let’s run nmap again, but this time against all 65k ports. This will take some time, while that’s working  I’m going to check and see if FTP has anonymous login allowed and browse to the web page served on port 8081 </p>
<img src="/img/ultratech/4.nmap-scan-4.png"/>

<p>It appears as if anonymous login isn’t enabled- moving on to the website on port 8081 </p>
<img src="/img/ultratech/5.FTP-anonymous-failed.png"/>

<p>Navigating to the webpage on port 8081 and we get a page that says “UltraTech API v0.1.3” Let’s check the source code</p>
<img src="/img/ultratech/6.webpage-index.png"/>

<p>Nothing here either.</p>
<img src="/img/ultratech/7.source-code-index-page.png"/>

<p>Let’s use gobuster to enumerating for subdomains and/or files (txt,php,html) that may be open to the public</p>
<img src="/img/ultratech/8.gobuster-scan-1.png"/>

<p>Check in on the nmap scan we see that it’s reporting port 31331 as open. This could be a false positive. let’s  verify by running nmap against port 31331</p>
<img src="/img/ultratech/9.nmap-scan-5.png"/>

<p>The results show there's Apache web server v2.4.29 running on this port. </p>
<img src="/img/ultratech/10.apache-webserver.png"/>

<p>This time we find a web page for a pseudo company named UltraTech. There’s no webforms to interact with. However, there are some possible usernames we should take note of. My next stop is to view the source code</p>
<img src="/img/ultratech/11.UltraTech-index.php"/>

<p>Nothing stands out right away in the source code. Let’s keep enumerating</p>
<img src="/img/ultratech/12.UltraTech-index-source.png"/>

<p>Let’s start enumerating for any subdirectories  with gobuster. While we wait on some results, Let’s see if there is a robots.txt and if there’s anything interesting in it</p>
<img src="/img/ultratech/13.gobuster-untraTech.png"/>

<p>The robots.txt file has an entry for a  sitemap.xml. It’s well worth taking a look.</p>
<img src="/img/ultratech/14.robots.txt-file.png"/>

<p>There’s three entries index.html, what.html and partners.html. From my notes we’ve already visited index and what so let’s navigate to partner.html</p>
<img src="/img/ultratech/15.sitemap.png"/>

<p>We found a login web form. Let’s check source and enumerate this further.</p>
<img src="/img/ultratech/16.partner-login-page.png"/>

<p>I used burpsuite to intercept the http request on the login page and in doing so I found a API GET request that was running every few seconds.</p>
<img src="/img/ultratech/17.get-request.png"/>

<p>After fuzzing the get requests for a while I found where I can injection vulnerability by appending  a bash command in the url </p>
<p>I can list the contents of the current directory and we find a sqlite db. Let’s see is we can read the contents</p>
<img src="/img/ultratech/18.injection-test.png"/>

<p>The contents of the database has two usernames and password hashes. Let’s try and crack them.</p>
<img src="/img/ultratech/19.sqli-hashes.png"/>

<p>Before we try John or Hashcat let’s give crackstation.net a shot at this hash</p>
<p>Great! Crackstation was able to crack this password.Let’s try the other password hash</p> 
<img src="/img/ultratech/20.mr00t-cracked-hash.png"/>

Mr00t:n100906

Madmin: mrsheafy


<p>I’m surprised. It cracked that password too. Now we have 3 places we can try these logins: FTP ,SSH and the partners.html login portal. Let’s try them out</p>
<img src="/img/ultratech/21.Madmin-cracked-password.png"/>

<p>I tried both users in FTP, SSH and partner web login page and only one users creditials worked on the FTP and SSH. There was nothing interesting in the partners web portal. Let’s enumerate the server using the SSH connection </p>
<img src="/img/ultratech/23.web-portal-login.png"/>
<img src="/img/ultratech/22.ssh-succes.png"/>

<p>Looking for a quick win with privesc. Thats a no-go… let’s look around the see if we can find any low hanging fruit</p>
<img src="/img/ultratech/24.privesc-attempt1.png"/>


<p>This is interesting.. I’m in a user group named “docker”.Maybe we’re in a docker container?? </p>
<img src="/img/ultratech/25.user-group.png"/>

<p>I ran this command to verify we’re in a docker container and to grab the name of the other(s) containers that may be running. I found one named “bash”</p>
<img src="/img/ultratech/26.Docker-image.png"/>

<p>Let’s spawn a shell in the container we found</p>
<img src="/img/ultratech/27.Docker-container-shell.png"/>

<p>That didn’t work so I exited that shell, checked GTFObins and found this command  </p>
<img src="/img/ultratech/29.GTFO-bins.png"/>



<p>What we need to do is change the default docker container name to the one running “bash” and we have a shell with root privileges. Now we just need to cat the root private ssh key to complete the box</p>
<img src="/img/ultratech/28.root-priesc-docker.png"/>
