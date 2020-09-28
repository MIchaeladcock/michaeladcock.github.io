---
layout: post
title: THM Jack of all trades writeup
excerpt_separator: <!--more-->
---
<img src="/img/0.header.png">
<!--more-->
https://tryhackme.com/room/jackofalltrades
<p>Enumerating  ports with an NMAP Scan.</p>
<img src="/img/1.nmap.png">
<p>Well this is odd. SSH is on port 80 and HTTP on port 22??? Let’s make a curl request to the home directory to see what comes back.</p>
<img src="/img/2.curl_reg.png">
<p>After a quick google search I found a setting in FireFox to add port 22 as a non restricted port.  Now let’s navigate to the root directory in our web browser. </p>
<p>Checking  the page source we can see a note with what appears to be a hash. Let’s grab that hash and  navigate to /recovery.php to see what’s on that page. </p>
<img src="/img/3.curl.png">
<p>It's a login page. Let's check the source code.</p>
<img src="/img/4.login.png">
<p>Appears to be another hash in the source code. let's try and decode both hashes from base64.</p>
<img src="/img/5.home_hash.png">
<p>After decoding the "home page" hash we find a note with a password and a possible username</p>
<img src="/img/6.hash_decode.png">
<p>Now to decrypt the "recovery page" hash.</p>
<p>This could be a password or still encoded with something else. First let's try to login with the known password from the "home page" hash.</p>
<img src="/img/7.hash_decode.png">
<p>The credentials  didn't work on the site or ssh</p>
<img src="/img/8.login_try.png">
<img src="/img/9.ssh_try.png">
<p>Checking in on dirbuster: It looks like there maybe something in a directory called "/assets/" let's take a look.</p>
<img src="/img/10.dirbuster.png">
<p>There's an image named stego. Let's download it and try it in steghide.</p>
<img src="/img/11.dir_finding.png">
<p>Using the password found on the "home page" hash we reveal a file names creds.txt</p>
<img src="/img/12.steghinde1.png">
<p>It was a rabbit hole...</p>
<img src="/img/13.rabbit_hole.png">			
<p>Let's try the other images.</p>
<p>Found another file in the header.jpg. The name of this file sounds promising.</p>
<img src="/img/14.steg_creds.png">
<img src="/img/15.steg_creds_find.png">
<p>Let's try these credentials  in the recovery.php page...</p>
<p>And we’re in.</p>
<img src="/img/16.login_text.png">
<p>We have a command injection vulnerability</p>
<img src="/img/17.show_command_inj.png">		
<p>The command injection is  probably our way  to a reverse shell on the server, but just to be on the safe side lets hit it with a dirbuster while we fuzz the website with bash commands.</p>
<img src="/img/18.dirbuster2.png">
<p>Let's try a bash reverse shell payload to see if we can get a shell.</p>
<img src="/img/19.bashshell_code.png">	
<img src="/img/20.bash_shell_inurl.png">			
<p>Before sending the get URL request, let's get ready to catch this on our local machine</p>
<img src="/img/21.nc_attemp.png">	
<p>That didn't work.... maybe if we're lucky Net Cat is installed on the server. Lets try.</p>
<p>Local machine is ready to catch the shell.</p>
<img src="/img/22.low-level-shell.png">
<p>Send the command</p>
<img src="/img/22.nc_inurl.png">	
<p>Success. We have a shell as the  www-data service user.</p>
<img src="/img/23.jacks_pass_finding.png">		
<p>let's start enumerting.</p>
<p>Looks like we have a user named jack (no suprise) and a list of possiable passwords. Is it time fo hyrda?</p>
<img src="/img/24.jacks_pass_file_cat.png">	
<p> Copy and paste this into a text file on our machine and try and bruteforce the jack cresditals on 22 port 80 with Hydra.</p>
<img src="/img/25.hydra_login.png">
<p>We have Jack's password let's ssh into the box.</p>
<img src="/img/26.login_as_jack_ssh.png">	
<p>Now  we're in as a higher privileged user with a stable shell. Let's get the flag and then start enumerating.</p>
<p>Typlicy there's a user.txt file here - instead we have a user.jpg. let's download this image and see what we can find.</p>
<img src="/img/27.jacks_user_img_file.png">	
<p>Setup a simple python server on the server</p>
<img src="/img/28.python_simple_server.png">	
<p>Issue a wget command  on our machine to download the image.</p>
<img src="/img/29.imaged_downloaded.png">		
<p>The user flag is in the image.</p>
<img src="/img/30.image_user_flag.png">		
<p>Now onto enumeration so we can try and elevate our privilege to the root users account.</p>
<p>Looking for an easy win with sudo.. womp, womp. Let's keep enumerating</p>
<img src="/img/31.sudo_l_for_easy_win.png">		
<p>Checking for SUID  permissions.</p>
<p>The strings binay seems out of place. let's check GTFObins.</p>
<img src="/img/32.checkSUID_prem.png">		
<p>GTFObins says we can abuse this SUID binary to read files we don't have permissions to. Lets try and read the root/root.txt file</p>
<img src="/img/33.gtfo_bins.png">		
<p>Although we didn’t elevate our privileges to root we captured the root flag. If we really wanted the root account we could read the /etc/shadow file by abusing the strings binary, copy the root password hash and try to crack it offline. If we are creative enough there’s probably a lot more ways to get root by abusing the strings binary. </p>
<img src="/img/34.root_flag.png">
			
			
			
		
	
	
	

