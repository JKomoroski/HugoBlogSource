---
title: "Improving My Network: Domains and DNS"
date: 2020-12-22T22:35:07-07:00
tags: [network, devember, domains, dns, homelab]
draft: false
---

This is the second part in my series on taking the [Devember challenge](https://forum.level1techs.com/t/welcome-to-level1-devember/162269)
 and improving my homelab and network. You can find the previous post [here]({{< ref "./improving-my-network-part-1.md" >}}).

### Domain
The first step in re-vamping my homelab was to finally purchase a domain. I was a complete novice on domain, dns, and 
 subnet management going into the challenge, so I asked some friends about it. Fortunately, the process was much simpler
 than I anticipated. It was suggested that I shop based on price alone, with an eye on whois privacy features secondarily.
 I did some shopping around and ended up buying my last name: `komoro.ski`. There are cheaper top level domains, but I 
 thought this was fun. I made my purchase at [porkbun.com](https://porkbun.com/), smaller domain name registrar which 
 has some excellent pricing. After buying the domain, I then needed to sort out DNS.

### DNS Management
After getting some direction from a co-worker, I started by creating a free account at cloudflare, who have a free tier 
 for DNS management among other features. I added my domain as a zone in cloudflare and then went to porkbun and set 
 the suggested cloudflare name servers as the authoritative domain servers for the domain. I went back to my cloudflare 
 account and created my first dns record by creating a CNAM entry for this site (blog.komoro.ski). I pointed it to my 
 github pages domain, which was where I was hosting this blog at the time. I then created several dns records for the 
 servers on my network (synology nas, raspberry pis, main server, etc); everything I had configured with a static ip in 
 my gateway got a dns record. 

### Dynamic DNS
I since I opted to use cloudflare for my dns management, I needed to find a way to update my home network's public ip 
 with cloudflare. After a bit of searching I found [a python script](https://github.com/timothymiller/cloudflare-ddns)
 which calls the cloudflare api to update dns records. The project publishes a docker container which makes deployment 
 easy. I followed the instructions, created an api key, and made a configuration file. To get started I deployed the 
 container to my main server.

In my next blog post I'll go over setting up my Unifi Dream Machine Pro to run a Wireguard VPN Interface and the 
 cloudflare ddns container. 
