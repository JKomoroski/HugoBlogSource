---
title: "Improving My Network: A Messy Situation"
date: 2020-12-21T19:48:59-07:00
tags: [network, devember ]
draft: false
---

In November 2020, I decided to take the Level 1 Techs [Devember Challenge](https://forum.level1techs.com/t/welcome-to-level1-devember/162269/135),
 which is a challenge for anyone to spend an hour a day working on a tech project. The challenge is sponsored by Linode,
 which offers some free credit for new users taking the challenge. I opted to take the challenge and finally buy a domain
 and set up my network properly. Since I've never had a domain before, my experience with proper networking and dns 
 management were novice at best, so I figured it would take me plenty of time to get things setup. This is the first in 
 a series of blog posts describing what I've set up, how I've done it, and (in many cases) why. The Level 1 Techs forum 
 has [my challenge thread](https://forum.level1techs.com/t/devember-setting-up-a-domain-and-networking-for-self-hosted-services/165129/8),
 which consists of my regular musings and learnings of things along the way. It's a lot of rambling, and half-baked 
 thoughts. This blog series is intended to distill all of that.

### A Messy Situation

Sometime in the summer of 2020 my home router gave up the ghost, so my home network got some upgrades. I decided to go 
 big in my hardware upgrade and move from a pile of stuff to a proper rackmount system with prosumer grade hardware. I 
 purchased the following at that time:

- [Unifi Dream Machine Pro](https://store.ui.com/collections/unifi-network-routing-switching/products/udm-pro)
- [Unifi Switch POE 16 (150W)](https://store.ui.com/products/unifi-switch-16-150w) 
- [Unifi 6 Lite Access Point](https://store.ui.com/products/unifi-ap-6-lite-beta) 
- [Unifi Flex Camera](https://store.ui.com/collections/unifi-protect-cameras/products/unifi-video-g3-flex-camera)

On my home network I was already running a few raspberry pis, a 4-bay Synology Nas, an old desktop that was running as 
 my main server, and my current desktop. Beyond that were all my wifi connected devices. I run quite a few services on 
 my main server including the usual suspects from the folks at [linuxserver.io](https://fleet.linuxserver.io/), a
 private [gitea](https://gitea.io/en-us/) instance and [plex](https://plex.tv). All of these services were running from 
 docker-compose on my server, and to access them I needed to connect to each service by typing in the ip and port for 
 the service (i.e. 192.168.1.11:8080). I tried to host a static site with links, to keep it straight before installing 
 [heimdall](https://heimdall.site/), but everything was still ip and port, I didn't have a domain, or dynamic dns setup.
 I didn't have a VPN setup either, so managing services while remote was a non-starter. I had great hardware, but my 
 software lagged behind.

My goal for the Level 1 Devember Challenge was to set up a domain as best I could, by giving services and hosts proper
 names, set up web hosting for this blog on my own hardware, and get a vpn solution in place, so I could manage things 
 while away from home.

The next post in this series will begin explaining what I did in detail. 
