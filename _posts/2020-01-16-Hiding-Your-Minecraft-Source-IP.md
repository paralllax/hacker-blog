---
title: Setting up Minecraft behind CloudFlare and a reverse Proxy
published: false
---

Note that this guide expects that you have purchased a domain name, and have an existing minecraft server already set up. 

The guide will go as follows:

> 1. Set up an Always Free Compute Instance with Oracle Cloud (or any cloud provider)
> 2. Import your records (if any exist) into cloudflare
> 3. Create an SRV record
> 4. Setup a reverse proxy with sslh

# [](#header-1)Setting up a Cloud server

You can set up a cloud server with any provider (aws, azure, google, digitalocean, etc...). I have chosen Oracle's "Always Free Tier" because it is Free. Other providers have similar services, however most relevant products expire after a year or less. Only Google and Oracle have servers/nodes that render free _FOREVER_. I've included links to their listings below if you would like to pick a different one:

[https://cloud.google.com/free/](Google Cloud Free Tiers)
[https://aws.amazon.com/free/](AWS Free Tiers)
[https://azure.microsoft.com/en-us/free/](Azure Free Tiers)
[https://www.oracle.com/cloud/free/](Oracle Cloud Free Tiers)

If you have picked another cloud provider, spin up a small centos 7 instance and skip to the next step. 

We are using a cloud server as another buffer between the client and our network. Once you have created an account navigate to 



# [](#header-1)Transfer your DNS records to CloudFlare


Cloudflare is a CDN (Content Delivery Network). A CDN is a large distributed network of servers around the globe. They provide several advantages for hosting content, such as caching static images, reducing bandwidth, hides the origin IP and more. In this case however, most of those features will be overlooked as cloudflare doesn't support games unless you are willing to shell out a lot of $$. 

We will be adding an SRV record, which has the draw back of revealing your origin IP. However, this will be sent to our cloud server, which will proxy the traffic back to our actual minecraft server. The main reason for adding the record with cloudflare is it should get rid of any potential beginner script kiddies as cloudflare will stop DDOS attacks with it's free tier. Most cloud providers also offer the same. The two combined, considering they are free, add a little more security and the benefit of allowing clients to connect directly over a domain name instead of via an IP adress and port. 

Sign up with CloudFlare [](here)

