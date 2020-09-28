---
layout: post
title: THM Anonymous Write-up
excerpt_separator: <!--more-->
---
<img src="/img/0.header-image.png">
<!--more-->
<p><b>Question 1. Enumerate the machine.  How many ports are open?</b></p>
<p><b>Question 2. What service is running on port 21?</b></p>
<p><b>Question 3. What service is running on ports 139 and 445?</b></p>
<p>Let’s start off by  running a basic nmap scan for the top 1000 ports</b></p>
<p>We have ports 21,22,139 and 445 open. Let enumerate these further. </p>
<img src="/img/1.basic_nmap_scan.png">
<p>I’m running  mnap with the -A flag just on these 4 open ports to get version, service type and OS detection.</p>
<img src="/img/1.basic_nmap_scan.png">
