# Letsencrypt automated request and renewal (with cloudflare implementation).

## Dependencies
Thanks to these really useful projects:
- https://github.com/lukas2511/dehydrated
- https://github.com/kappataumu/letsencrypt-cloudflare-hook

## Installation
### Base script
#### Install
~~~
mkdir /opt/letsencrypt.sh
cd /opt/letsencrypt.sh && git clone https://github.com/lukas2511/dehydrated.git
~~~
#### First launch and terms agreements
The first launch will output you   
~~~
# INFO: Using main config file config

To use dehydrated with this certificate authority you have to agree to their terms of service which you can find here: https://letsencrypt.org/documents/LE-SA-v1.1.1-August-1-2016.pdf

To accept these terms of service run `./dehydrated --register --accept-terms`.
~~~

So run the following to accept terms
~~~
cd /opt/letsencrypt.sh/dehydrated && ./dehydrated --register --accept-terms
~~~


### Letsencrypt Cloudflare hooks (skip if you don't use Cloudflare)
#### Install
~~~
cd /opt/letsencrypt.sh/dehydrated
git clone https://github.com/kappataumu/letsencrypt-cloudflare-hook hooks/cloudflare
sudo apt-get install build-essential libssl-dev libffi-dev python-dev
pip install --upgrade pip
pip install -r hooks/cloudflare/requirements-python-2.txt
~~~

Note: If you have any trouble installing requirements with pip   
~~~
export LC_ALL="en_US.UTF-8"
export LC_CTYPE="en_US.UTF-8"
sudo dpkg-reconfigure locales
~~~

#### Config
Put this content in "config" file in cd /opt/letsencrypt.sh/dehydrated, replace by the correct values.   
~~~
export CF_EMAIL='user@example.com'
export CF_KEY='K9uX2HyUjeWg5AhAb'
~~~

## One time usage (testing installation)
### With Cloudflare implementation
cd /opt/letsencrypt.sh/dehydrated && ./dehydrated -f config -c -d <yourdomain.com> -d <sub1.yourdomain.com> -d <sub2.yourdomain.com> -t dns-01 -k 'hooks/cloudflare/hook.py' -o </your/certificates/path>   
### Without Cloudflare implementation
cd /opt/letsencrypt.sh/dehydrated && ./dehydrated -f config -c -d <yourdomain.com> -d <sub1.yourdomain.com> -d <sub2.yourdomain.com> -t dns-01 -k 'hooks/cloudflare/hook.py' -o </your/certificates/path>

## Now making an automatic renewal with cron
Creating a script called by the crontab, add the following content to /opt/letsencrypt.sh/cron.sh
Note: Remove "-k 'hooks/cloudflare/hook.py'" if you don't use Cloudflare
~~~
#!/usr/bin/env bash

cd /opt/letsencrypt.sh/dehydrated && ./dehydrated -f config -c -d <yourdomain.com> -d <sub1.yourdomain.com> -d <sub2.yourdomain.com> -t dns-01 -k 'hooks/cloudflare/hook.py' -o </your/certificates/path>
~~~   

Make this executable
~~~
chmod +x /opt/letsencrypt.sh/cron.sh
~~~   

Creating the log folder
~~~
mkdir /var/log/letsencrypt.sh/
~~~   

Add this line in the crontab
~~~
0 1 * * * /opt/letsencrypt.sh/cron.sh >> /var/log/letsencrypt.sh/cron.log 2>&1
~~~
