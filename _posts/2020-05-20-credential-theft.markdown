---
layout: post
title:  "Spearphishing Credential Theft"
date:   2020-05-20 10:32:10 -0400
categories: phishing credentialaccess
tags: credentialtheft
---

<p>
Many users use their corporate credentials on non corporate websites which is very dangerous. If that site gets compromised the adversary now has credentials to a corporate network. Credential Theft Prevention controls which categories of URL’s users are allowed to type their corporate credentials.
</p>
  
<p>
In this use case we send the user a targeted email asking the user to reset their corporate credentials due to suspicious behaviour. The hypertext in the email does not match the actual URL. When the user clicks the link they are taken to what looks like a Google sign in page, and anything the adversary captures anything the user types in the form. From there credentials can be abused to access corporate applications. 
</p>
<br>


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


<h3>Step 3 - Configure Social Engineering Toolkit</h3>

<p>From the Kali host or wherever you've install SET start it up with
</p>
<code>
   setoolkit
</code>
<br>

<img src="/images/setoolkit.png" alt="setoolkit">
<br>

<p>
Choose Option 2 - Website Attack Vectors
</p>
<img src="/images/setoolkit-website-attack-vectors.png" alt="setoolkit">
<br>

<p>
Choose Option 3 - Credential Harvester Attack Method
</p>
<img src="/images/setoolkit-credential-harvester.png" alt="setoolkit">
<br>

<p>
Choose Option 1 - Web Templates
</p>
<img src="/images/setoolkit-credential-web-template.png" alt="setoolkit">
<br>

<p>
Choose Option 2 - Google
</p>
<img src="/images/setoolkit-credential-web-google.png" alt="setoolkit">
<br>

<p>You will be asked for the IP to listen on. This doesn't necessarily have to be a FQDN or public IP, it just needs to be somewhere your target can get to.
</p>

<p>
SET will now clone the login form from the source, making it look really good, and listening on TCP port 80 on the IP address provided. You can see that there are connections to the system immediately because this one was on the Internet.
</p>
<img src="/images/setoolkit-credential-listening.png" alt="setoolkit">

<h3>Step 4 - Setup an Email Client</h3>

<p>
Below you will find the what Thunderbird and Mail look like configured.
</p>
<img src="/images/thunderbird-configured.png" alt="Thunderbird Configured">
<i>Thunderbird Configured</i><br>
<img src="/images/mail-configured.png" alt="Mail Configured">
<i>Mail Configured</i><br>


<h3>Step 5 - Send a Malicious Email</h3>

<p>
You can either use the Social Engineering Toolkit, or for a quicker way you can also find the test file phishing-credential-harvest.txt in the freshdemo/smarthost repo. It is already configured to send an email to the phishme account. Feel free to edit the file and send the mail with;
</p>
<code>
   sendmail -t < phishing-credential-harvest.txt
</code>

<b>The harvester URL in the file is s.example.com. You can either change this to your IP/domain or edit the zone file for example.com on the freshdemo/mailanddns</b>
<br>


<h3>Step 6 - View the Email</h3>

<p>
You should have the email in the configured client. 
</p>

<p>
The arrows indicate;
</p>
<ul>
   <li>the link in the body of the email is obfuscated with regular text <i>change your account password immediately</i>
   <li>when you mouse over the link the real address shows up in the status bar at the bottom, s.example.com.
<br>

<img src="/images/credential-theft-phish.png" alt="Spearphishing Link">
<i>Spearphishing Link</i>
<br>
<br>

<h3>Step 7 - Click the Link</h3>

<p>
When the link is clicked you are brought to the harvester page.
</p>
<img src="/images/setoolkit-credential-server.png" alt="SET">
<br>

<p>
When you type your username and password the user gets redirected to google.com, and the credentials appear back in SET.
</p>
<img src="/images/setoolkit-credential-harvested.png" alt="setoolkit">
<br>

<p>
Re-testing this with Credential Theft Protection enabled prevented the HTTP POST. SET did not get the credentials and the user was redirected to this page.
</p>
<img src="/images/setoolkit-credential-blocked.png" alt="setoolkit">
