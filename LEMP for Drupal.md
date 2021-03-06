## Update system

    $ apt-get update
    $ apt-get upgrade --show-upgraded

## Drush

Install Drush

    $ apt-get install drush

Upgrade to the latest version

    $ drush dl drush --destination='/usr/share'

## Upload progress

http://pecl.php.net/package/uploadprogress

    apt-get install php-pear
    apt-get install make
    pecl install uploadprogress

Edit php.ini

    nano /etc/php5/fpm/php.ini

Add

    extension=uploadprogress.so

Restart

    $ service php5-fpm restart


## Other packages for Drupal

Most of Drupal sites need GD

    $ apt-get install drush php5-gd

For Commerce Kickstart

    $ apt-get install php5-curl

Other packages that people suggested

    $ apt-get install php-apc
    $ apt-get install php5-common

Other packages

    $ apt-get install git


## Virtual Host Configuration

Edit the nginx config file

    $ nano /etc/nginx/sites-available/example.com

Copy and pasted the Nginx configuration for Drupal. Then create a link.

    $ ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled

Remove the default site

    $ rm /etc/nginx/sites-enabled/default

Reload nginx configuration

    $ /etc/init.d/nginx reload
    or
    $ service nginx reload

Create the root document for this website and start putting files

    $ mkdir -p /var/www/example.com

Test it

    $ echo "<?php phpinfo(); ?>" > /var/www/example.com/index.php

And go to example.com, if it works, remove the file.

    $ rm /var/www/example.com/index.php

Restart Nginx.

    $ /etc/init.d/nginx restart

## Install Drupal directly

Go to the directory

    $ cd /var/www

Download Drupal 7 the latest version

    $ drush dl

Or, if you'd like to install from some distribution:

    $ drush dl commerce_kickstart

Move the files to www, note that the version of commerce kickstart might change.

    $ mv commerce_kickstart-7.x-2.5/* commerce_kickstart-7.x-2.5/.htaccess example.com/

Create the directories for Drupal. Do it in one go:
(replace example.com with your domain)

    $ mkdir /var/www/example.com/sites/default/files; chmod 777 /var/www/example.com/sites/default/files; cp /var/www/example.com/sites/default/default.settings.php /var/www/example.com/sites/default/settings.php; chmod 777 /var/www/example.com/sites/default/settings.php

Or one by one

    $ mkdir /var/www/example.com/sites/default/files
    $ chmod 777 /var/www/example.com/sites/default/files
    $ cp /var/www/example.com/sites/default/default.settings.php /var/www/example.com/sites/default/settings.php
    $ chmod 777 /var/www/example.com/sites/default/settings.php

Then go to example.com to install, after installation,

    $ chmod 444 /var/www/example.com/sites/default/settings.php

## Multisites

    $ cd /var/www/example.com
    $ mv default/settings.php example.com/
    $ mkdir sites/example.com/files
    $ chmod 777 sites/example.com/files

## Migrate Drupal for another server

On the server

    $ cd ~/.ssh
    $ ssh-keygen
    $ cat id_rsa.pub

Copy it!

    ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCttw4qZn5Kw+B8aiB13UUyIprxQQ5CGBtUZDahHT/bw5kr1kw9e90nR+PkJcdduxTFpd69ggPTCCo2l18Akvadzvxcbxqaw412523wryfdshrtFSDRgo7pWBlO2T8WUqfs0ag6Nc+ZHQGpyaI88ZhNlGFECiqHp9mh8+HeDEyyCunnAl9zjibAMZ+fEXSZmycUovQa/+jsvsoMl2tOG1Qi5+JKFGgyb8cpnlD2svIAuyUNkd HANA@ip-10-132-193-87

Add to the other server

Log in via sftp

    $ cd /var/www
    $ sftp root@example.com

    > get /example/PUBLIC.tar.gz
    > get /example/DB_DUMP.sql

Uncompress

    $ tar -jxv -f PUBLIC.tar.gz -C /var/www/example.com
    $ mysql -u DBUSER -pDBPASS bhuntr < DB_DUMP.sql

## Varnish In Drupal

Edit VCL file and paste the code from varnish3-for-drupal7.vcl

    $ nano /etc/varnish/default.vcl

Get the key

    $ cat /etc/varnish/secret

Install varnish

    $ drush dl varnish; drush en varnish -y

And set up on drupal admin, remember to set the life

Edit Drupal 7 settings.php

    $ nano /var/www/example.com/sites/default/settings.php

Make sure these are not commented.

    $conf['reverse_proxy'] = TRUE;
    $conf['reverse_proxy_header'] = 'HTTP_X_CLUSTER_CLIENT_IP';
    $conf['reverse_proxy_addresses'] = array('127.0.0.1');

Add this to the end

    // Varnish
    $conf['cache_backends'][] = 'sites/all/modules/varnish/varnish.cache.inc';
    $conf['cache_class_cache_page'] = 'VarnishCache';
    // Drupal 7 does not cache pages when we invoke hooks during bootstrap.
    // This needs to be disabled.
    $conf['page_cache_invoke_hooks'] = FALSE;

Check response header to see if

    X-Drupal-Cache:HIT
    X-Varnish-Cache:HIT

## Memcache

Install

    apt-get install memcached libmemcached-tools
    apt-get install php5-dev php-pear make
    pecl install memcache

Edit

    nano /etc/php5/conf.d/memcache.ini

Paste

    extension=memcache.so
    memcache.hash_strategy="consistent"


Edit memcached.conf
    nano /etc/memcached.conf

Set the memory

    # Change this default value to ~ 1/4 of your total available RAM
    -m 16

Restart

    service memcached restart
    service nginx restart

Check if Memcached is running:

    sudo netstat -tap | grep memcached

You should see something like:

    tcp 0 0 localhost:11211 *:* LISTEN 25266/memcached

Install memcache

    drush dl memcache; drush en memcache -y

Add this to Drupal's settings.php, change unique_key to something unique.

    $conf['cache_backends'][] = 'sites/all/modules/memcache/memcache.inc';
    $conf['cache_default_class'] = 'MemCacheDrupal';
    $conf['cache_class_cache_form'] = 'DrupalDatabaseCache';
    $conf['memcache_key_prefix'] = 'unique_key';

Check the status and stats with memstat tool

Part of the memcached package is a handy tool called memstat.
You need to specify the host IP and port. In this case the host IP is 127.0.0.1 and the port 1211.
Open the Terminal Window and enter :

    memstat 127.0.0.1:11211


## APC

AB test first and save the result.

    ab -n 10 -c 5 http://example.com/

Install APC.

    apt-get install php-apc
    pecl install apc

Edit php.ini, add this

    extension=apc.so
    apc.enabled = 1
    apc.shm_size = 48

Restart

    service nginx restart
    service php5-fpm restart

Check APC stat

    mv /usr/share/doc/php-apc/apc.php.gz /var/www/example.com/
    gunzup /var/www/example.com/apc.php.gz

edit apc.ini file.

    nano /etc/php5/conf.d/apc.ini

adding apc.enable_cli=1 to apc.ini file.

    apc.enable_cli=1

Now you can go to http://example.com/apc.php to check the stat.

    drush dl apc; drush en apc -y

Add conf to settings.php
http://drupalcode.org/project/apc.git/blob_plain/refs/heads/7.x-1.x:/README.txt

    $conf['cache_backends'] = array('sites/all/modules/apc/drupal_apc_cache.inc');
    $conf['cache_class_cache'] = 'DrupalAPCCache';
    $conf['cache_class_cache_bootstrap'] = 'DrupalAPCCache';
    //$conf['apc_show_debug'] = TRUE;  // Remove the slashes to use debug mode.

And you can do AB Test again.

## References

Configure Varnish 3 for Drupal 7
https://fourkitchens.atlassian.net/wiki/display/TECH/Configure+Varnish+3+for+Drupal+7

Caching in Drupal 7: Varnish, APC, and Memcache
http://www.rklawson.com/blog/2012/04/14/caching-drupal-7-varnish-apc-and-memcache

Linux – How to configure NGINX + PHP-FPM + Drupal 7.0
http://blog.celogeek.com/201209/202/how-to-configure-nginx-php-fpm-drupal-7-0/

Installing Memcached for Drupal 7
http://andrewdunkle.com/how-install-memcached-drupal-7

Memcache Installation
http://drupal.org/node/1131468
