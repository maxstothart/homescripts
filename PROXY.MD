# Caddy Proxy Server

## Caddy Service File
```bash
[Unit]
Description=Caddy HTTP/2 web server
Documentation=https://caddyserver.com/docs
Wants=network-online.target
After=network-online.target

[Service]
Restart=on-failure

; User and group the process will run as.
User=felix
Group=felix

; Letsencrypt-issued certificates will be written to this directory.
Environment=CLOUDFLARE_API_TOKEN=TOKENHERE
Environment=JWT_SECRET=JWTSECRET

; Use graceful shutdown with a reasonable timeout
KillMode=mixed
KillSignal=SIGQUIT
TimeoutStopSec=5s

; Always set "-root" to something safe in case it gets forgotten in the Caddyfile.
ExecStart=/opt/caddy/caddy run --environ --config /opt/caddy/Caddyfile
ExecReload=/opt/caddy/caddy reload --config /opt/caddy/Caddyfile

LimitNOFILE=1048576
LimitNPROC=512
AmbientCapabilities=CAP_NET_BIND_SERVICE

[Install]
WantedBy=multi-user.target
```

## Google Authentication

For now while Caddy updates the automated build process, I download from:

https://caddyserver.com/download

And pick the following plugins as I use CloudFlare and Google oAuth2:

- caddy-dns/cloudflare
- greenpau/caddy-auth-portal
- greenpau/caddy-authorize

There are documentation links on stepping through setup and configuration:

https://authp.github.io/docs/authenticate/oauth/backend-oauth2-0007-github

I use proxy entries for all my items behind and oAuth for my login validation.

## Caddy Plex Configuration

I use [Caddy](https://github.com/mholt/caddy) as a proxy server and if you want to use it with Plex, there is a very simple config below that I used. I have since removed this from my config to help simplify things.

My plex configuration in my CaddyFile as follows:

```bash
plex.domain.us {
tls {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
}
reverse_proxy localhost:32400
log {
        output file /opt/caddy/logs/plex.log
        format single_field common_log
}
}
```

Plex also needs to have a few additional steps to be added so Caddy is used in all situations.

### Plex Configuration

Remote Access - Disable

```
Network - Custom server access URLs = https://<your-domain>:443
```
Network - Secure connections = Required

### Stopping Local Access
By default, Plex regardless of what override URL you set will still connect locally to 32400. To remove this, I use the second option of adding the option to the Preferences.xml. You need to stop Plex and add the line in near the end of your Preferences.xml

allowLocalhostOnly="1" 

UFW or other firewall:
- Deny port 32400 externally (Plex still pings over 32400, some clients may use 32400 by mistake despite 443 and 80 being set).
