Let's Encrypt is a relatively new (public beta) free, automated, and open source certificate authority.

---

## Installation
Open a terminal on your webserver:

```language-bash
$ cd ~
$ git clone https://github.com/letsencrypt/letsencrypt
$ cd letsencrypt
$ ./letsencrypt-auto --help
```

This will install all required dependencies.

Let's Encrypt supports Apache out of the box, but I use Nginx.

## Preparing Nginx
GitHub user renchap wrote a guide for using Let's Encrypt for Nginx and it worked for me.

First up, the Nginx configuration. I'm using the DigitalOcean Ghost image, so editing my Nginx configuration is done with:

```language-bash
$ vim /etc/nginx/sites-enabled/ghost
```

In the server block, add a new location block:

```language-nginx
location '/.well-known/acme-challenge' {
    default_type "text/plain";
    root /tmp/letsencrypt-auto;
}
```

and restart Nginx:

```language-bash
$ service nginx restart
```

## Generate the certificate
Next, we'll set up our environment:

```language-bash
$ export DOMAINS="-d your_site.com"
$ export DIR=/tmp/letsencrypt-auto
$ mkdir -p $DIR && ~/letsencrypt/letsencrypt-auto certonly --server https://acme-v01.api.letsencrypt.org/directory -a webroot --webroot-path=$DIR --agree-dev-preview $DOMAINS
$ service nginx reload
```

My key was placed in `/etc/letsencrypt/live/celeodor.com`. Let's update Nginx to use that key now.

## Nginx configuration
Back in your site's Nginx configuration file, edit the server block to add these lines:

```language-nginx
listen 443 ssl;
ssl_certificate /etc/letsencrypt/live/your_site.com/cert.pem;
ssl_certificate_key /etc/letsencrypt/live/your_site.com/privkey.pem;
```

And restart Nginx:

```language-bash
$ service nginx restart
```

Navigate to https://your_site.com and verify that the certificate is being read.

## Automatically renew certificate
The Let's Encrypt certificate only lasts for 90 days, but we can renew it before that. Renchap's guide recommends doing so every 60 days.

We'll put the required commands in a script:

```language-bash
$ vim ~/ssl_renew.sh
```

```
export DOMAINS="-d your_site.com"
export DIR=/tmp/letsencrypt-auto
mkdir -p $DIR && letsencrypt --renew certonly --server https://acme-v01.api.letsencrypt.org/directory -a webroot --webroot-path=$DIR --agree-dev-preview $DOMAINS
```

```language-bash
$ service nginx reload
```
Mark it as executable:

```language-bash
$ chmod +x ~/ssl_renew.sh
```

Now we'll toss that into a cron job to run every 60 days:

```language-bash
$ crontab -e
0 0 */60 * * /root/ssl_update.sh > /root/ssl_update.log 2>&1
```

## Redirect all HTTP traffic to HTTPS
If your new certificate is working correctly and you'd like to direct all HTTP traffic to HTTPS, your Nginx site configuration file will need some editing. We'll have 2 server blocks: 1 for catching the HTTP traffic and redirecting it to HTTPS, and the other for actual serving our website over HTTPS:

```language-nginx
server {
    listen 80;
    server_name your_site.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name your_site.com;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_certificate /etc/letsencrypt/live/your_site.com/cert.pem;
    ssl_certificate_key /etc/letsencrypt/live/your_site.com/privkey.pem;

    root /usr/share/nginx/html;
    index index.html index.htm;

    client_max_body_size 10m;

    location '/.well-known/acme-challenge' {
        default_type "text/plain";
        root /tmp/letsencrypt-auto;
    }

    location / {
        proxy_pass http://localhost:2368;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_buffering off;
    }
}
```

Edit to your tastes, and then restart Nginx:

```language-bash
$ service nginx restart
```

Navigate to your site's homepage in HTTP and verify that you're redirected to the HTTPS URL.

## SSL Labs Rating
With my self-signed SSL cert, this site scored a "T" on SSL Labs. This time around, it scored a "B".
