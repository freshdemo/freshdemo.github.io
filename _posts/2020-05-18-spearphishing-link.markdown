---
layout: post
title:  "Spearphishing Link"
date:   2020-05-28 12:54:10 -0400
categories: phishing initialaccess
---
<p>
One of the first steps during an incident or breach is phishing, where a user will be lured to a website that will either convince users to input their credentials, have them execute malware, or take advantage of a vulnerability to run further commands to gain a better foothold.
</p>

<h3>Step 1 - Deploy the Phishy Infrastructure</h3>

<p>
Using the following Docker container will allow you to run SMTP, IMAP, and DNS to facilitate these attacks, <a href="https://github.com/freshdemo/mailanddns" target="_blank">https://github.com/freshdemo/mailanddns</a>. See the README on how to deploy and run the container.
</p>


<h3>Step 2 - Deploy Kali/Social Engineering Toolkit</h3>

<p>
Deploy this in the same environment as the Phishy Infrastrucutre. Azure has a Kali image you can add directly to your Resource Group.
</p>
<p>
Once your red host is running deploy and configure a local MTA that will send emails through your phishy mailserver, <a href="https://github.com/freshdemo/smarthost" target="_blank">https://github.com/freshdemo/smarthost</a>. The steps to set this up and in the README.
</p>


<h3>Step 3 - Setup an Email Client</h3>

<p>
Below you will find the what Thunderbird and Mail look like configured.
</p>
<img src="/images/thunderbird-configured.png" alt="Thunderbird Configured">
<i>Thunderbird Configured</i><br>
<img src="/images/mail-configured.png" alt="Mail Configured">
<i>Mail Configured</i><br>


<h3>Step 4 - Send a Malicious Email</h3>

<p>
You can either use the Social Engineering Toolkit, or for a quicker way you can also find the test file fullmail.txt in the freshdemo/smarthost repo. It is already configured to send an email to the phishme account. Feel free to edit the file and send the mail with;
</p>
<code>
   sendmail -t < fullmail.txt
</code>

<h3>Step 5 - View the Email</h3>

<p>
You should have the email in the configured client. Be careful of the link as it is live fire at the time of this writing, and is currently being blocked in this environment due to enterprise network security services.
</p>

<p>
The arrows indicate;
</p>
<ul>
   <li>the link in the body of the email is obfuscated with regular text <i>follow these instructions</i>
   <li>when you mouse over the link, the real address shows up in the status bar at the bottom.
   <li>when the link is clicked (don't do it) the page can't currently be reached.
</ul>
<br>

<img src="/images/spearphished-link.png" alt="Spearphished Link">
<i>Spearphished Link</i>
<br>
<br>

<h3>Step 6 - View the Protection on the NGFW</h3>

<p>
The log below indicates that the domain clicked from the email was sinkholed. This means that when this target computer was asking what the IP address is of the domain even that traffic was dropped. Because it was sinkholed the real IP of the malicious server will not be known to this client so any attempt to connect by name is prevented.
</p>

<img src="/images/spearphished-link-sinkhole.png" alt="Spearphished Link Sinkhole">
<i>Spearphished Link Sinkhole</i>
