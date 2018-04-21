# Deployment

There are usually a couple parts to our deployment:

## Nginx

A sub-domain allows us to have different `*.adicu.com` websites (e.g.
`density.adicu.com` or `learn.adicu.com`). We use
[nginx](https://www.nginx.com/) to configure this domain.

The two interesting directories are `/etc/nginx/sites-available` (which
defines all the availale websites) and `/etc/nginx/sites-enabled` (which
has all of our currently running websites). All the files in
`/etc/nginx/sites-enabled` are symlinks to files in
`/etc/nginx/sites-available`.

These usually point to directories in `/srv/`, which holds the logs and
source code for our different services.

## Let's Encrypt

For SSL security, we use [certbot-auto](https://certbot.eff.org/) to
manage [Let's Encrypt](https://letsencrypt.org/) certificates.

## Docker

For static websites, we usually put the appropriate files into
`/srv/<name>/public_html` (as dictated by the nginx setup).

For non-static websites, we use Docker, which handles automatic restarts
for us.
