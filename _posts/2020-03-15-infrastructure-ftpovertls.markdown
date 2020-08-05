---
layout: post
title:  "FTP Infrastructure"
date:   2020-03-15 13:24:10 -0400
categories: infrastructure 
tags: infrastructure
---
<p>
Getting infrastructure deployed and configured correctly has been one of the most time consuming things when demonstrating the cybersecurity attack chain.
</p>

<h3>Option 1 - Build a Docker Container</h3>

<p>
The primary goal was to deploy this in a Docker container. Reason being it is infrstructure that is not needed all the time, and may need to move from a laptop, to Google Cloud Platform, to Azure, etc.
</p>

<p>
Get it here, <a href="https://github.com/freshdemo/ftpovertls" target="_blank">https://github.com/freshdemo/ftpovertls</a>.
</p>
<br>

<h3>Option 2 - Build Your Own</h3>

<p>
You can easily interpret the Dockerfile to deploy all of the software and configurations manually on a host.
</p>
<br>

<h3>Option 3 - Run the Built Container</h3>

<p>
From a security perspective it is not a great idea to use an already built container as you have no idea how it was built or if any backdoor or data exfiltration configurations are pre-built. This container is automatically being built on Docker Hub any time a commit is made to GitHub, so what you see is what you get.
</p>

<code>
   docker pull freshdemo/ftpovertls<br>
   docker run  -it -p 21:21 -p 30000-30020:30000-30020-d <image ID>
</code>
<br>

