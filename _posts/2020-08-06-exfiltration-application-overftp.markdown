---
layout: post
title:  "Application Firewall Data Exfiltration over FTP"
date:   2020-03-14 14:11:15 -0400
categories: exfiltration 
tags: app-id
---
<p>
Once an environment has been compromised and the adversary finds some desirable data the data needs to be removed from the network. In this use case we will assume that FTP is being permitted egress by the firewall.
</p>

<h3>Step 1 - Deploy the FTP Server</h3>

<p>
The instructions to build a preconfigured FTP server with TLS encryption can be found here, <a href="https://freshdemo.github.io/infrastructure/2020/03/15/infrastructure-ftpovertls.html" target="_blank">https://freshdemo.github.io/infrastructure/2020/03/15/infrastructure-ftpovertls.html</a>
</p>

<p>
This server is configured to provide FTP over TLS using self signed certificates. 

<h3>Step 2 - Decrypt TLS</h3>

<p>
Make sure that the NGFW (or other DUT) is decrypting TLS traffic between the source and FTP server. Otherwise you will only see the network connectivity and lose visibility into the files and threats being transferred.
</p>


<h3>Step 3 - Upload a File</h3>


<p>
From a blue host behind the firewall login to the FTP server. This server is running FTP over TLS so you should receive a certificate warning that you need to accept. Pay close attention to approve the root certifcate that is from the NGFW.
</p>

<p>Default settings in popular clients like Filezilla seem to work ok. The server is setup for passive transfer. You may want to force the client to "require explicit FTP over TLS"
</p>


<h3>Step 4 - View the Results</h3>

<p>
Providing TLS decryption and the associated security policies were implemented in a way to permit the traffic you will see something like the following.
</p>


<p>
The first screenshot indicates that the FTP traffic over port 21 was permitted. From there the data transfer was moved over to TCP port 57729. There are 3 log types; Traffic (firewall), data, and vulnerability.
</p>
<img src="/images/exfiltration-application-overftp1.png" alt="ngfw">
<br>
<br>

<p>
The second screenshot is showing the same logs just scrolled over to the right to show the additional data available. The data and vulnerability logs show the filename.</p>

<p>
For further control you could use the data filtering to block uploads to unknown FTP servers to prevent data loss. The vulnerability profile could be configured to block threats rather than alert.
</p>

<br>
<br>
<img src="/images/exfiltration-application-overftp2.png" alt="ngfw">


