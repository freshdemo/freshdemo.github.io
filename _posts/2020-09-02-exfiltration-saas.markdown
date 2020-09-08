---
layout: post
title:  "Data Exfiltration over SaaS (safe application enablement)"
date:   2020-09-02 09:49:25 -0400
categories: exfiltration 
tags: app-id
---
<p>
Whether it is a user taking data backups into their own hands, they are stealing the data, or an adversary got their hands on data they would like a copy of, SaaS often provides a great solution. There are many file sharing applications designed for this. And in most environments these applications are very difficult to control, so they are effectively allowed. 
</p>

<p>
This use case is focused on File Sharing applications however the exact same process should be followed for personal email, collaboration tools, and pretty much every other category of application permitted.
</p>

<p>
The goal is to permit uploads and downloads to sanctioned applications, meaning applications where IT has access to the data if required. We will be focussing on the tolerated applications that you may want certain functionality from. For instance we may use OneDrive for file sharing so don't want users uploading to sites like Dropbox or Google Drive, but do we care if they download their partners grocery list from those sites? (we shouldn't).
</p>

<h3>Step 1 - Decrypt TLS Traffic</h3>

<p>
Make sure that SSL decryption is enabled for at least your source/client IP. This is what allows the visibility into what's going on inside the stream.
</p>

<h3>Step 1 - Configure Security Policies</h3>

<p>
Typically I leave it to you to configure security policies, but in this use case it's important to be clear about what they look like.
</p>

<p>
Closer to the top of the policy set you'll want to specify the policies for your sanctioned SaaS, by type or app. For example you may have a policy with all of the enterprise Office365 applications for authenticated users in the environment. From there you would want to apply all of the threat signatures available because you still shouldn't trust the content going to or from the services.
</p>

<p>
To permit the rest of these types of File Sharing applications we are going to use an Application Filter, which works similarly to URL Filtering with much more control and accuracy. The Application Filter being used here is filtering on sub-category = file-sharing, then we are automatically permitting risk levels 1-4 of these applications. As new content is created by the manufacturer and updated on the NGFW that traffic will automatically be permitted in the way we want. This ensures a consistent experience and reduces the amount of tickets and complaining.
</p>

<img src="/images/appid-exfiltration-saasfilter.png">

<p>
This policy would be to start dealing with the rest of the File Sharing applications. You can see there are only a few applications appearing in the rule. In the applications section we are using an Application Filter, and the manually adding a couple of risk level 5 apps that are also relevant.
</p>

<img src="/images/appid-exfiltration-saaspolicy.png">
<br>
<br>

<h3>Step 2 - Apply File Blocking</h3>

<p>
For this use case we are permitting all of those applications automatically, and want to prevent users from uploading to prevent data loss. For this a simple File Blocking profile is configured and applied to the security policy.
</p>

<img src="/images/appid-exfiltration-saasfileblocking.png">
<br>
<br>

<p>
Another similar use case is to use Host Intetrity Posture checks to identify users on devices that are not connected to the domain among other things. This would indicate that the user is on BYOD. In that case for the sanctioned SaaS applications you may want to prevent those users to still access the sanctioned SaaS, but do not permit downloading.
</p>


<h3>Step 3 - Upload a File</h3>

<p>
From a blue host behind the firewall login to Dropbox and Google Drive. Upload a file that ideally you have not already uploaded to the service (more on why later).
</p>

<p>
You will see messages similar to this for Dropbox and Google Drive respectively.
</p>

<img src="/images/appid-exfiltration-saasdropboxblock.png">
<br>
<br>
<img src="/images/appid-exfiltration-saagoogledriveblock.png">
<br>
<br>


<h3>Step 4 - View the Results</h3>

<p>
Taking a look at the results we can see that the Security Policy permitted the application traffic and File Blocking profile prevented the data from being uploaded.
</p>

<img src="/images/appid-exfiltration-saasdropboxupload.png">
<br>
<br>
<img src="/images/appid-exfiltration-saasgoogledriveupload.png" alt="ngfw">
<br>
<br>


<h3>Warnings</h3>

<p>
There were a couple observations made.
</p>

<p>
First: Microsoft OneDrive and Google Drive will hash a file before uploading, send that file to their cloud, and if they have seen it before your system will not actually send the file. This certainly worked as expected for the cloud providers, but you will not see a file transfer.
</p>

<p>
Second: Microsoft OneDrive seems to be a little more evasive. Many of the tests resulted in the file being transferred over ms-onedrive-base rather than ms-onedrive-uploading. Blocking ms-onedrive-uploading is one solution to this.
</p>


