---
title: "Setting up NextCloud"
tags: programming
date: 2021-02-24
---

Mostly notes for myself on how I setup nextcloud on [Time4VPS](https://www.time4vps.com/).
I was mostly following [this article](https://dev.to/bajicdusko/setting-up-nextcloud-as-alternative-to-google-services-2i5p) but found it was slightly wrong in a couple of places and missed out the odd thing.

## Install the dependencies

```bash
sudo apt install apache2 mariadb-server libapache2-mod-php unzip

sudo apt install php-gd php-json php-mysql php-curl php-mbstring php-intl php-imagick php-xml php-zip
```


## Download nextcloud

Navigate [here](https://nextcloud.com/install/#instructions-server) to find out the latest verison.
Then (substituting in the latest version for 21.0.0):


```bash
cd /var/www
sudo wget https://download.nextcloud.com/server/releases/nextcloud-21.0.0.zip
sudo unzip nextcloud-21.0.0.zip
```

## Prepare the database




```bash
sudo mysql
```
```sql
CREATE USER 'nextcloud' IDENTIFIED BY 'nextcloud';
CREATE DATABASE nextcloud;
GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@localhost IDENTIFIED BY ‘nextcloud’; and then flush'em FLUSH PRIVILEGES;
quit;
```

The `IDENTIFIED BY 'nextcloud'` part is basically setting the password,  which you will need later.


## Configuring Apache

This is just giving permissions to the user 'nextcloud' which our apache 

```bash
cd /var/www
sudo chmod 750 nextcloud -R
sudo chown www-data:www-data nextcloud -R
```


## Configure next cloud

Use whatever editor you like, let's pretent it is `nvim`:

```bash
cd /etc/apache2/sites-available/
nvim nextcloud.conf
```

Here is the one I used, but obviously you'll need to substitute in your own host:

```xml
Alias /nextcloud "/var/www/nextcloud/"

<Directory /var/www/nextcloud/>
        Require all granted
        AllowOverride All
        Options FollowSymLinks MultiViews

        <IfModule mod_dav.c>
                Dav off
        </IfModule>

</Directory>
<IfModule mod_ssl.c>

    <VirtualHost nextcloud.poleprediction.com:443>

        ServerAdmin my_email.com
        ServerName nextcloud.poleprediction.com

        DocumentRoot /var/www/nextcloud

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        SSLEngine on
        SSLCertificateFile      /etc/ssl/certs/poleprediction.com.pem
        SSLCertificateKeyFile   /etc/ssl/private/poleprediction.com.key.pem

        <FilesMatch "\.(cgi|shtml|phtml|php)$">
            SSLOptions +StdEnvVars
        </FilesMatch>
        <Directory /usr/lib/cgi-bin>
            SSLOptions +StdEnvVars
        </Directory>
    </VirtualHost>
</IfModule>
```

## Finally enable nextcloud

Well enable nextcloud and restart apache:


```bash
sudo a2ensite nextcloud
sudo a2enmod rewrite headers env dir mime
sudo systemctl restart apache2
```

You should be able to now visit the root domain and setup nextcloud via its web interface.


## Cron

I am using the nexcloud rss app, so I needed to setup cron mode. 

```bash
crontab -u www-data -e
```

Add the following simple line which will mean the cron job is run every 5 minutes:

```
*/5  *  *  *  * php -f /var/www/nextcloud/cron.php
```


## SSL

I'm not sure I understand everything here, but [this was the guide](https://bayton.org/docs/nextcloud/installing-nextcloud-on-ubuntu-16-04-lts-with-redis-apcu-ssl-apache/) I found most useful.

I just installed the `certbot` using `apt-get`

```bash
apt-get install certbot python3-certbot-apache
```

Then I basically just ran the `certbot` and it mostly does everything for us:

```bash
certbot --apache --agree-tos --rsa-key-size 4096 --email allan@poleprediction.com --redirect -d nextcloud.poleprediction.com
```

Finally once that has done its business it's worth adding to the configuration (`/etc/apache2/sites-available/nextcloud.conf`:

```
        <VirtualHost *:80>
           ServerName cloud.nextcloud.com
           Redirect permanent / https://cloud.nextcloud.com/
        </VirtualHost>
```

This just redirects all incoming `http` traffic to `https`.
