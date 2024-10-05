# Install Certbot on Debian with Cloudflare DNS

This guide will help you install Certbot on a Debian system and configure it to use the Cloudflare DNS plugin to manage SSL certificates.

## Step 1: sudo apt install snapd

First, update your package list and install snapd:

```sh
sudo apt update
sudo apt update
```

Once snapd is installed, exit the ssh terminal and re-login to ensure snap’s paths are updated properly.

## Step 2: Update snapd

```sh
sudo snap install core
```

## Step 3: Remove certbot-auto and any Certbot OS packages

If you have previously installed Certbot via other methods, such as using apt, you'll need to remove those versions to avoid conflicts:

```sh
sudo apt-get remove certbot
```

## Step 4: Install certbot

Now, install Certbot via the snap package:

```sh
sudo snap install --classic certbot
```

To ensure the certbot command is available globally, create a symbolic link to the snap binary:

```sh
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

Run this command on the command line on the machine to acknowledge that the installed plugin will have the same classic containment as the Certbot snap.

```sh
sudo snap set certbot trust-plugin-with-root=ok
```

## Step 5: Install Cloudflare DNS plugin

To manage DNS challenges with Cloudflare, install the Cloudflare DNS plugin:

```sh
sudo snap install certbot-dns-cloudflare
```

## Step 6: Set up credentials

Go to the [Cloudflare dashboard](https://dash.cloudflare.com/?to=/:account/profile/api-tokens) and create a new API token. The Token needed by Certbot requires `Zone:DNS:Edit` permissions for only the zones you need certificates for. Copy the new API token value and save it in a file at `~/.secrets/certbot/cloudflare.ini`

```
# Cloudflare API token used by Certbot
dns_cloudflare_api_token = 0123456789abcdef0123456789abcdef01234567
```

Make sure the file has restricted permissions

```sh
chmod 600 ~/.secrets/certbot/cloudflare.ini
```

## Step 7: Request SSL Certificates

You can now use Certbot to request certificates for your domains using the Cloudflare DNS plugin.

**For a single domain (e.g., example.com):**

```sh
certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials ~/.secrets/certbot/cloudflare.ini \
  -d example.com
```

**For multiple domains (e.g., example.com and www.example.com):**

```sh
certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials ~/.secrets/certbot/cloudflare.ini \
  -d example.com \
  -d www.example.com
```

**For a wildcard certificate (e.g., \*.example.com):**

```sh
certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials ~/.secrets/certbot/cloudflare.ini \
  -d *.example.com
```

If you're using Apache and want Certbot to automatically configure SSL for your sites, add the `-i apache` flag:

```
certbot \
  --dns-cloudflare \
  --dns-cloudflare-credentials ~/.secrets/certbot/cloudflare.ini \
  -i apache
  -d *.example.com
```

After obtaining the SSL certificate, you need to configure Apache to serve your site over HTTPS. Ignore this step if you run certbot with `-i apache` option.

1. Redirect HTTP to HTTPS: Add the following rewrite rules to the <VirtualHost \*:80> block in your Apache configuration file (e.g., /etc/apache2/sites-available/example.com.conf):

```sh
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com

    RewriteEngine on
    RewriteCond %{SERVER_NAME} =*.example.com.vn [OR]
    RewriteCond %{SERVER_NAME} =wildcard.example.com
    RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>
```

2. Enable SSL: In the <VirtualHost \*:443> block, specify the paths to your SSL certificates:

```sh
<VirtualHost *:443>
    ServerName example.com
    DocumentRoot /var/www/example.com

    SSLCertificateFile /etc/letsencrypt/live/example.com/fullchain.pem
    SSLCertificateKeyFile /etc/letsencrypt/live/example.com/privkey.pem
    Include /etc/letsencrypt/options-ssl-apache.conf
</VirtualHost>
```

## Step 8: Test automatic renewal

Certbot certificates are valid for 90 days, but you can set up automatic renewal. First, test if automatic renewal works by running a dry run:

```sh
sudo certbot renew --dry-run
```

To check the status of the renewal timer, use the following command:

```sh
systemctl list-timers
```
