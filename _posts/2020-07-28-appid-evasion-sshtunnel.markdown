---
layout: post
title:  "Application Firewall Application Evasion over SSH"
date:   2020-07-28 11:11:15 -0400
categories: exfiltration 
tags: app-id
---
<p>
When behind a NGFW there are several reasons to need to bypass egress controls. Privacy, data exfiltration, content filtering evasion, etc.
</p>
<p>
One way to do that is with SSH. The client behind the NGFW will need to SSH into a server outside the NGFW, and specify a local port to open up to proxy the traffic. Client software (web browser/ftp client/entire system) can be configured to send traffic to the local proxy port which will push all of that traffic through the SSH tunnel.
</p>
<p>
If the NGFW is permitting SSH we may be able to use that port to bypass other controls, and appear on the Internet to be coming from the SSH remote SSH server.
</p>

<h3>Step 1 - Start the SSH client</h3>

<p>
In this case the client is being directed to start a proxy on port 8080 (-D 8080), compress the traffic (-C), do not output any messages to the local console (-q), do not execute remote commands (-N), and finally username@IP address.
</p>
<br>
<code>
  ssh -D 8080 -C -q -N 13.88.250.188
</code>

<h3>Step 2 - Configure System Proxy Settings</h3>

<p>
Navigate to the settings of your web browser (File, Preferences) and search for proxy.
</p>

<p>
Ensure you configure it as a SOCKS5 proxy.
</p>
<p>
Specify localhost and port 8080 (or whatever you specified in Step 1 as the -D port.
</p>
<br>

<br>
<br>
<img src="/images/app-id-evastion-sshtunnel-settings.png" alt="settings">

<h3>Step 3 - Validate it's Working</h3>

<p>
Use your web browser to search "what's my ip". It should report back the IP address of your SSH server outside the NGFW.
</p>

<h3>Step 4 - View the Results</h3>


<p>
Looking at the following NGFW log we can see that the NGFW permitted the session by catch all policy. The traffic is identified as ssh-tunnel.
</p>
<p>
SSH should be restricted to use by administrators. There is also a very low liklihood that ssh-tunnel is a relevant application for them to use, so this should have been prevented by permitting only legitimate SSH traffic for administration by administrators and not the tunneling functionality.
</p>
<br>
<br>
<img src="/images/appid-evasion-sshtunnel-decrypted.png" alt="decrypted">


<p>
The only way we can identify that tunneling is ocurring inside of SSH is by decrypting the SSH traffic. You can see that the SSH traffic has been decrypted in the image above.
<p>

<p>
The following image shows what it looks like to run the exact same command to an SSH server where the NGFW is not decrypting the traffic. Only the SSH application is identified and we are blind to the tunneling that is ocurring.
</p>
<br>
<br>
<img src="/images/appid-evasion-sshtunnel-nodecypt.png" alt="nodecrypt">
