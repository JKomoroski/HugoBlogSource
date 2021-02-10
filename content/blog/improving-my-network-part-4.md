---
title: "Improving My Network: Static Hosting and Reverse Proxy with Caddy"
date: 2021-01-04T10:15:24-07:00
tags: [network, devember, linode, caddy, homelab]
draft: false
---

This is the fourth part in my series on taking the [Devember challenge](https://forum.level1techs.com/t/welcome-to-level1-devember/162269)
 and improving my homelab and network. You can find the previous post [here]({{< ref "./improving-my-network-part-3.md" >}}).

### Requirements
I had the following requirements when selecting a reverse proxy:
- Let's Encrypt Certificate Support
- Static Hosting Support
- Reverse Proxy Support
- Simple configuration

Preferably, these features would work without any third party plugins. I looked into Nginx, Apache, Traefik, and some
 others, but I landed on [Caddy](https://caddyserver.com/v2).

### Caddy Set Up Script

I use the following script to set up my caddy proxy. The script downloads a caddy binary with selected plugins. A custom
 build of caddy can be configured [here](https://caddyserver.com/download). The script then installs the downloaded 
 binary to `/usr/local/bin`, and grants it permission to bind to low ports. It then creates a caddy system user, and 
 sets up the various system directories. The script then creates a systemd unit file, which contains a service 
 configuration. The script then creates a caddy configuration file, and enables the caddy service.

```shell
#!/usr/bin/env bash

wget -q -O caddy "https://caddyserver.com/api/download?os=linux&arch=amd64&p=github.com%2Fabiosoft%2Fcaddy-exec&p=github.com%2Fabiosoft%2Fcaddy-hmac&p=github.com%2Fcaddy-dns%2Fcloudflare"
mv "./caddy" "/usr/local/bin/caddy"
chmod +x "/usr/local/bin/caddy"
setcap cap_net_bind_service+ep /usr/local/bin/caddy

useradd --shell /bin/false --home-dir /etc/caddy --system caddy
mkdir -p /etc/caddy/.config /etc/caddy/.local
mkdir -p /var/log/caddy
mkdir -p /var/caddy/html
chown -R caddy: /etc/caddy /var/log/caddy

cat <<EOF >> /etc/systemd/system/caddy.service
[Unit]
Description=Caddy web server
After=network-online.target

[Service]
User=caddy
Group=caddy
Type=exec
WorkingDirectory=/var/caddy/html

ExecStart=/usr/local/bin/caddy run -config /etc/caddy/Caddyfile
ExecReload=/usr/local/bin/caddy reload -config /etc/caddy/Caddyfile
ExecStop=/usr/local/bin/caddy stop

LimitNOFILE=1048576
LimitNPROC=512

PrivateTmp=true
PrivateDevices=true
ProtectHome=true
ProtectSystem=strict
ReadWritePaths=/etc/caddy/.local /etc/caddy/.config /var/log

CapabilityBoundingSet=CAP_NET_BIND_SERVICE
AmbientCapabilities=CAP_NET_BIND_SERVICE
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target

EOF

cat <<EOF >> /etc/caddy/Caddyfile
https://blog.komoro.ski {
    encode gzip
    file_server {
        hide 404.html .git
    }
    
    handle_errors {
        @404 {
            expression {http.error.status_code} == 404
        }
    
        rewrite @404 /404.html
        file_server
    }
    
    @github {
        path /webhook
        header_regexp X-Hub-Signature "[a-z0-9]+\=([a-z0-9]+)"
    }
    
    @hmac {
        expression {hmac.signature} == {http.regexp.1}
    }

    route @github {
        hmac sha1 MY_SECRET_WEBHOOK_KEY_HERE
        exec @hmac {
            command git
            args \
                pull \
                origin \
                self_hosted
            directory /var/caddy/html
        }
    }

    tls {
        dns cloudflare MY_SECRET_CLOUDFLARE_KEY_HERE
    }

    log {
        output file /var/log/caddy/access.log {
            roll_size 10MB
            roll_keep 5
            roll_keep_for 240h
        }
    }
}

https://git.komoro.ski {
    reverse_proxy MY_SECRET_PRIVATE_DOMAIN.komoro.ski:3000
}

EOF

systemctl enable --now caddy.service

```

### Caddyfile Description
The Caddyfile has two sections. It hosts a static website, and enables a reverse proxy to a private gitea instance. The
 reverse proxy is quite simple:

```
https://git.komoro.ski {
    reverse_proxy MY_SECRET_PRIVATE_DOMAIN.komoro.ski:3000
}
```

The private domain points to a private IP available via the previously configured WireGuard VPN. Caddy takes care of 
 enabling and configuring a TLS cert, and redirecting http traffic to https.

The static site is a bit more involved. The core configuration is:
```
https://blog.komoro.ski {
    fileserver
    encode gzip
}
```
This creates a static file server out of the working directory defined in the systemd unit file, and allows gzip 
 compression for all served files. Again, TLS certs and http redirect are provided. Everything else in the blog 
 configuration is optional. 

The simple `log` option rotates the caddy access log file:

```
https://blog.komoro.ski {
    ...
    ...
    log {
        output file /var/log/caddy/access.log {
            roll_size 10MB
            roll_keep 5
            roll_keep_for 240h
        }
    }
}
```

The file server options hides some files and redirects the default 404 errors to a custom 404 page:

```
https://blog.komoro.ski {
    ...
    file_server {
        hide 404.html .git
    }
    
    handle_errors {
        @404 {
            expression {http.error.status_code} == 404
        }
    
        rewrite @404 /404.html
        file_server
    }
    ...
}
```

The `tls` directive uses the [cloudflare-dns](https://github.com/caddy-dns/cloudflare) plugin to leverage cloudflare dns
 for acme challenges.
```
https://blog.komoro.ski {
    ...
    tls {
        dns cloudflare MY_SECRET_CLOUDFLARE_KEY_HERE
    }
    ...
}
```

The last portion of this config is support to receive a webhook at the `/webhook` path. It then validates the HMAC 
 header value authentication. If validated, the exec plugin executes a `git pull` command which pulls a new version of 
 the static site files from github.

```
https://blog.komoro.ski {
    ...
    ...
    @github {
        path /webhook
        header_regexp X-Hub-Signature "[a-z0-9]+\=([a-z0-9]+)"
    }
    
    @hmac {
        expression {hmac.signature} == {http.regexp.1}
    }

    route @github {
        hmac sha1 MY_SECRET_WEBHOOK_KEY_HERE
        exec @hmac {
            command git
            args \
                pull \
                origin \
                self_hosted
            directory /var/caddy/html
        }
    }
    ...
    ...
}
```

