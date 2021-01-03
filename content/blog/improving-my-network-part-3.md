---
title: "Improving My Network: Public Proxy and Wireguard VPN"
date: 2021-01-01T18:16:49-07:00
tags: [network, devember, linode, vpn, homelab]
draft: false
---

This is the third part in my series on taking the [Devember challenge](https://forum.level1techs.com/t/welcome-to-level1-devember/162269)
 and improving my homelab and network. You can find the previous post [here]({{< ref "./improving-my-network-part-2.md" >}}).

### Unifi Dream Machine Pro Problems
I upgraded my router to a Unifi Dream Machine Pro over the summer, which has treated me well, but running custom 
 scripts and applications on the box has been a challenge because the UDM Pro will wipe out any customizations whenever 
 a firmware upgrade is applied, and some changes get wiped out in a reboot. I wanted my Dynamic DNS for Cloudflare to 
 update anytime my router lost connection or rebooted, so this limitation needed to be overcome.

### The Fix 
Enter [boostchicken's udm utility project](https://github.com/boostchicken/udm-utilities). The project allows scripts
 and services to persist through upgrades and reboots. I spent the time to install the deb package from the unifi-os
 shell, and then put in the work to migrate the cloudflare-ddns container (described in the previous post). I made a 
 [contribution](https://github.com/boostchicken/udm-utilities/pull/80) so others could also use the script on their 
 devices.

### A Linode Proxy
I wanted to create a public proxy for all my private services, so I wouldn't have to deal with opening ports on my UDM
 or needlessly expose my public IP. I decided to stet up a linode account since they were sponsoring the devember 
 challenge. I signed up, set up an instance, and began investigating VPN and proxy options.

### WireGuard VPN
I tried setting up an IPSec and an OpenVPN via the UDM Pro gui, but getting it to suitably proxy traffic required more 
 configuration and knowledge than I was prepared. It's likely possible, I just wasn't willing to invest the time for 
 either solution. This was in part because I learned about [wireguard](https://www.wireguard.com/). Wireguard 
 is incredibly simple to configure for the topology I was hoping to create. The UDM-Pro uses an older Linux Kernel, so
 WireGuard isn't available by default. To overcome this, I installed the WireGuard-Go container using the boostchicken 
 utility script. I installed wg-quick in my Linode instance, and pointed a DNS record to point to the public IP of the 
 linode instance.

### Configuration

I aimed for the following topology out of my vpn:

```
 +-----------------------+
 |                       |
 |    Public Internet    |
 |                       |
 |  +-------------+      |
 |  |             |      |
 |  | Workstation |      |
 |  |             |      |
 |  +---^---------+      |
 |      |                |
 |      |                |
 |      |                |
 |      |                |
 |  +---v--------+       |
 |  |            |       |      +--------------+       +--------------+
 |  | Linode-VPN <-------------->              |       |              |
 |  |            |       |      | Home-Gateway <-------> Home-Network |
 |  +------------+       |      |              |       |              |
 |                       |      +--------------+       +--------------+
 |  +-----------------+  |          |
 |  |                 |  |          |
 |  | Cloudflare-DDNS <-------------+
 |  |                 |  |
 |  +-----------------+  |
 |                       |
 +-----------------------+
```
Basically, I allow the linode and udmpro to act as Nat routers
I enabled ipv4 traffic forwarding on my linode instance:

```sysctl -w net.ipv4.ip_forward=1```

Then used the following WireGuard configuration in my Linode instance:

```
# Linode VPN Configuration
[Interface]
Address = 10.222.0.1/24
ListenPort = 51820
PrivateKey = LINODE_VPN_PRIVATE_KEY
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

# UDM PRO Peer
[Peer]
PublicKey = UDM_PRO_PUBLIC_KEY
AllowedIPs = 10.222.0.2/32, 192.168.1.0/24
PersistentKeepalive = 25

# Workstation Peer
[Peer]
PublicKey = WORKSTATION_PUBLIC_KEY
AllowedIps = 10.222.0.3/32
PersistentKeepalive = 25

```

I used the following for my UDM-Pro (Home-Gateway):

```
[Interface]
# Configuration for the UDM-PRO
# The IP address that this client will have on the WireGuard network.
Address = 10.222.0.2/32
PrivateKey = MY_UDM_SECRET_KEY_HERE
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o br0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o br0 -j MASQUERADE


[Peer]
# Configuration for the UDM-PRO to connect to the Linode instance
# The linode public key
PublicKey = MY_LINODE_INSTANCE_PUBLIC_KEY
# The Lindode WireGuard server to connect to.
Endpoint = MY_LINODE_DNS_SUB_DOMAIN.komoro.ski:51820

# The subnet for the WireGuard VPN
AllowedIPs = 10.222.0.1/24

PersistentKeepalive = 25

```

Finally, I used the following for my workstation:

```
[Interface]
# Configuration for the Workstation Interface

# The IP address that this client will have on the WireGuard network.
Address = 10.222.0.3/32
ListenPort = 51820
PrivateKey = MY_WORKSTATION_SECRET_KEY_HERE

[Peer]
# Configuration to peer with the linode vpn instance

# The linode public key
PublicKey = MY_LINODE_INSTANCE_PUBLIC_KEY 

# The WireGuard server to connect to.
Endpoint = MY_LINODE_DNS_SUB_DOMAIN.komoro.ski:51820

# The subnet WireGuard VPN is in control of, and the subnet for the home network
AllowedIPs = 10.222.0.1/24, 192.168.1.0/24

# Ensures that a nat router does not kill the tunnel, by sending a ping every 25 seconds.
PersistentKeepalive = 25

```

For the server and my workstation, I used systemd to manage the wg-quick service. A detail guide can be found 
 [here](https://zach.bloomqu.ist/blog/2019/11/site-to-site-wireguard-vpn.html). I use Fedora linux on my workstation, 
 so I added a Gnome [extension]((https://github.com/atareao/wireguard-indicator)) to quickly toggle my WireGuard 
 connection when I'm not on my home network. This allows to me connect via SSH to any ip on my home server.

That's the quick and dirty of my WireGuard setup. In the next part I'll go over setting up
 [Caddy](https://caddyserver.com/v2) on the same Linode instance to host static pages, and act as a reverse proxy for 
 services on my home network.
