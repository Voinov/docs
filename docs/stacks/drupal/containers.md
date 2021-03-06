# Drupal stack containers

## Apache

{!stacks/_includes/containers/php-apache.md!}

## Blackfire

{!stacks/_includes/containers/blackfire.md!}

## Crond

{!stacks/_includes/containers/php-crond.md!}

## Drupal Node.js

Drupal node is a container with a server app for the [Node.js Integration Drupal module](https://www.drupal.org/project/nodejs). You can configure Node.js via environment variables that listed at https://github.com/wodby/drupal-node 

Usage:

1. Install [Node.js Integration Drupal module](https://www.drupal.org/project/nodejs) on your site
2. Visit the Node.js configuration page under the Configuration menu, and enter the connection information for your Node.js server application. Set host to `node` (or `drupal-node` for local environment) and service key can be found on `[Instance] > Stack > Node.js`

## Mailhog

{!stacks/_includes/containers/mailhog.md!}

## MariaDB

See [MariaDB stack documentation](../mariadb/index.md).

## Memcached

{!stacks/_includes/containers/memcached.md!}

## Nginx

!!! important "New Nginx image" 
    Since Drupal stacks 5.2.0+ nginx image `wodby/drupal-nginx` has been replaced with [`wodby/nginx`](https://github.com/wodby/nginx) with `$NGINX_VHOST_PRESET=drupalX`
    
{!stacks/_includes/containers/nginx.md!}

See [details](https://github.com/wodby/nginx#drupal) about virtual host preset.

!!! warning "Do not gzip pages in Drupal"
    We already gzip content on Nginx side and it works faster. Having double gzip may cause issues. 

### PHP endpoints

For security reasons, default nginx config allows executing limited php endpoints. This is how you can add additional php endpoints:

1. Add `*.conf` file to your codebase with locations definition, example:
```
location = /custom-php-endpoint.php {
    fastcgi_pass php;
}
```
2. In nginx service configuration set new environment variable `NGINX_SERVER_EXTRA_CONF_FILEPATH` to your *.conf file (e.g. `/var/www/html/drupal.conf`). It will be included at the end of `/etc/nginx/conf.d/drupal.conf`
3. Restart the service

Alternatively, you can replace your HTTP server to [Apache](#apache) (not recommended) that has less strict rules.

### XML endpoints

By default nginx config requests Drupal backend when `rss.xml` or `sitemap.xml` requested. If you want to add another XML endpoint generated by Drupal just set environment variable `NGINX_DRUPAL_ALLOW_XML_ENDPOINTS` to any value and restart the service.

### Custom config

If the default drupal config and available environment variables are not enough for your customizations you can replace the config with your own:

1. Copy `/etc/nginx/conf.d/drupal.conf` to your codebase, adjust to your needs
2. Deploy code with your config file
3. Add new environment variable `NGINX_CONF_INCLUDE` for nginx service, the value should the path to your *.conf file (e.g. `/var/www/html/nginx.conf`

### Files proxy

You can proxy all requests to files to (similar to what drupal module stage_file_proxy does) by adding the environment variable `NGINX_DRUPAL_FILE_PROXY_URL` to URL of your Drupal instance with files, e.g. `http://example.com`

### Mod pagespeed

Nginx comes with [mod_pagespeed](https://www.modpagespeed.com/) which is disabled by default. To enable it add `NGINX_PAGESPEED=on` environment variable to Nginx service.

## Node.js

{!stacks/_includes/containers/node.md!}

## OpenSMTPD

See [OpenSMTPD stack documentation](../opensmtpd/index.md).

## PHP

{!stacks/_includes/containers/php.md!}

!!! info "SSH and Cron containers"
    For Wodby environments we additionally spin up copies of PHP services with overridden commands to run cron and ssh daemons. All environment variables added to PHP-FPM service will be automatically passed to [SSHd](#sshd) and [Crond](#crond) services.

#### Drupal-specific

Additionally, variable `$DRUPAL_SITE` (previous deprecated name `$WODBY_APP_SUBSITE`) contains Drupal site directory, e.g. `default`.  

!!! warning "WARNING" 
    Some environment variables used by PHP may be overridden in [`wodby.settings.php`](index.md#drupal-settings) file
    
{!stacks/_includes/containers/php-dev.md!}

### PHPUnit

1. Inside your drupal/core directory, copy the file `phpunit.xml.dist` and rename it to `phpunit.xml`
2. Open that file and make sure that you update `SIMPLETEST_BASE_URL` to `http://nginx`
3. In order to make sure that your DB connection is working as well, update `SIMPLETEST_DB` to `mysql://drupal:drupal@mariadb/drupal`

### Drush

PHP container comes with pre-installed drush and drush launcher. Drush launcher will help you use drush that comes with your project without specifying the full path to it. See [how to use](index.md#drush) drush aliases.

### Drupal Console

PHP container comes with installed drupal console launcher (not the same as drupal console), the launcher used to be able run drupal console without specified the full path to it. Please note that starting Drupal Console ~1.0 you have to install it manually (via composer) per project.

## Redis

You can use Redis as a cache storage for your Drupal application. Redis in-memory storage works much faster compared to the default Drupal's cache storage – database.

### Wodby environment

1. Download and install [redis module](https://www.drupal.org/project/redis)
2. Make sure redis service enabled in your stack
3. Additionally for Drupal 8: enable redis integration under `[Instance] > Cache > Settings` page 
4. That's it, all required configuration already provided in [`wodby.settings.php` file](index.md#drupal-settings)

### Local environment

1. Download and install redis module
2. Add the following lines to your `settings.php` file (make sure the path to redis module is correct):

For Drupal 8:

```php
$settings['redis.connection']['host'] = 'redis';
$settings['redis.connection']['port'] = '6379';
//$settings['redis.connection']['password'] = '';
$settings['redis.connection']['base'] = 0;
$settings['redis.connection']['interface'] = 'PhpRedis';
$settings['cache']['default'] = 'cache.backend.redis';
$settings['cache']['bins']['bootstrap'] = 'cache.backend.chainedfast';
$settings['cache']['bins']['discovery'] = 'cache.backend.chainedfast';
$settings['cache']['bins']['config'] = 'cache.backend.chainedfast';
$settings['container_yamls'][] = "modules/contrib/redis/example.services.yml";
```

For Drupal 7:

```php
$conf['redis_client_host'] = 'redis';
$conf['redis_client_port'] = '6379';
//$conf['redis_client_password'] = ';
$conf['redis_client_base'] = 0;
$conf['redis_client_interface'] = 'PhpRedis';
$conf['cache_backends'][] = "sites/all/modules/contrib/redis/redis.autoload.inc";
$conf['cache_default_class'] = 'Redis_Cache';
$conf['cache_class_cache_form'] = 'DrupalDatabaseCache';

$conf['lock_inc'] = "sites/all/modules/contrib/redis/redis.lock.inc";
$conf['path_inc'] = "sites/all/modules/contrib/redis/redis.path.inc";
```

For more information see [Redis stack documentation](../redis/index.md).

## Rsyslog

Rsyslog can be used to stream your applications logs (watchdog). It's similar to using syslog, however there's no syslog in PHP container. Rsyslog will stream all incoming logs to a container output.

Here how you can use it with Monolog:

1. Install [monolog module](https://www.drupal.org/project/monolog). Make sure all dependencies being downloaded
2. Add new handler at `monolog/monolog.services.yml`:
```yml
monolog.handler.rsyslog:
  class: Monolog\Handler\SyslogUdpHandler
  arguments: ['rsyslog']
```
3. Rebuild cache (`drush cr`)
4. Use `rsyslog` handler for your channels
5. Find your logs in [rsyslog container output](../../apps/logs.md)

Read [Logging in Drupal 8](https://www.wellnet.it/en/blog/logging-drupal-8) to learn more.

## Solr 

!!! important "New Solr image"
    Since Drupal stacks 5.2.0+ Varnish image `wodby/drupal-solr` has been replaced with `wodby/solr` and `$SOLR_DEFAULT_CONFIG_SET`, see [versions matrix](https://github.com/wodby/solr#drupal-search-api-solr)

See [Solr for Drupal stack documentation](../solr-drupal/index.md).

## SSHd

{!stacks/_includes/containers/php-sshd.md!}

## Varnish

!!! important "New Varnish image"
    Since Drupal stacks 5.2.0+ Varnish image `wodby/drupal-varnish` has been replaced with [`wodby/varnish`](https://github.com/wodby/varnish) and `$VARNISH_CONFIG_PRESET=drupal`

You can use Varnish to cache dynamic pages and show pages to your visitors without any calls to Drupal (cache stored in memory).  

### Drupal 8

Read [this article](https://wunderkraut.se/blogg/purge-cachetags-varnish) to learn how to use Varnish with Drupal 8.

### Drupal 7

1. Install and enable varnish module (use the dev version). All required configuration for this module already provided in [`wodby.settings.php` file](index.md#drupal-settings)
2. Go to `Home » Administration » Configuration » Development` page of Drupal website and
3. Check Cache pages for anonymous users
4. Check Compress cached pages.
Check Aggregate and compress CSS files.
Check Aggregate JavaScript files.
Also, we recommend to install expire module to configure auto purge of pages when some content has been updated. After installation go to `Home » Administration » Configuration » System` and select External expiration at the "Module status" tab.


For more details see [Varnish stack documentation](../varnish/index.md)

## Webgrind

{!stacks/_includes/containers/webgrind.md!}
