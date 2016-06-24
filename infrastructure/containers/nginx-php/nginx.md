# Nginx
 
* [Access](#access)
* [Logs](#logs)
* [Configuration files](#configuration-files)
* [CLI](#cli)
    * [Rebooting nginx](#rebooting-nginx)
    * [Show installed modules](#show-installed-modules)

## Access

Nginx is a part of [nginx-php container](README.md).

## Logs

Logs can be found under `/var/log/nginx`.

## Configuration files

Configuration files can be found under `/srv/conf/nginx`.

## CLI 

### Rebooting nginx

```bash
$ nginx -s reload
```

or

```bash
$ s6-svc -h /var/run/s6/services/nginx/
```

### Show installed modules

```bash
$ nginx -V
```