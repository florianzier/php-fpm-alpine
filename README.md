# Docker PHP7-FPM with composer support

Alpine based PHP7-FPM docker image with support for composer command, often used extensions and proxy support.

**Table of contents**
<!-- TOC depthFrom:2 depthTo:4 withLinks:1 updateOnSave:1 orderedList:0 -->

- [Installed extensions](#installed-extensions)
- [Build Image](#build-image)
- [Build behind a (company) proxy](#build-behind-a-company-proxy)
- [Run behind a Proxy](#run-behind-a-proxy)
- [Example build arguments](#example-build-arguments)

<!-- /TOC -->

## Installed extensions

- mysqli
- pdo-mysql
- zip
- bzip2
- intl
- Xdebug
- OPcache
- APCu

## Build Image

You can use the `docker-compose.yml` file for an easy setup of build arguments and optional temporary proxy settings.

Trigger a build with following command:
```bash
docker-compose build php
```
In order to do it without compose-file, the associated command line switch would be `--build-arg`:
```bash
docker build \
  --build-arg TZ="Europe/Berlin" \
  --build-arg INTL="de_DE" \
  --build-arg HTTP_PROXY=proxy.domain.tld:8080 \
  --build-arg […] \
  ./
```

System timezone and locale for PHP's `ICU/intl` extension can be set via build-arguments `TZ` and `INTL`. Otherwise they have a fallback to the defaults `TZ="UTC"` and `INTL="en"`.

The `[Date]` and `[intl]` sections will be appended to the `php.ini` file during build. They aren't necessary in the `/conf/php.ini` file.

The template `/conf/xdebug.ini` stores default `Xdebug` settings.

## Build behind a (company) proxy

The `HTTP_PROXY`, `HTTPS_PROXY`, `FTP_PROXY`, `NO_PROXY` arguments aren't persisted as environment variables. A proxy (and it's excluded hosts) is only set during build time to accomplish a successful connection to the download servers with extensions and composer.

Drop your own certificates into `./conf/certs` folder if you have to connect through a SSL secured (company) proxy and therefore need a certificate file. Certificates are permanently stored in the image file, make sure it's only the public part and not containing the private key for more security.

## Run behind a Proxy

The advantage to not store the `HTTP_PROXY`, `HTTPS_PROXY`, `FTP_PROXY`, `NO_PROXY` variables permanently in the environment is, that you can enable/disable the proxy anytime by setting the corresponding environment variables at runtime via command line or within a project's `docker-compose.yml`. Otherwise you would have to explicitly unset or override them as soon the proxy address is changed. And multiple instances behind different proxies would require a certain amount of image rebuilds.

This is where the permanent stored certificates become handy. Simply drop several in the designated folder for an arbitrary amount of proxies. You can then decide for every single container, whether it should use a proxy and which one (since you have all necessary public keys built-in).

## Example build arguments

Following argument list is an example to build an image with system timezone settings for Germany and suitable locale, used for formatting, transliteration, conversion, … with PHP's `ICU/intl` extension.
```
TZ:          "Europe/Berlin"
INTL:        "de_DE"
HTTP_PROXY:  "http://proxy.domain.tld:8080"
HTTPS_PROXY: "https://proxy.domain.tld:4443"
FTP_PROXY:   "ftp://proxy.domain.tld:2121"
NO_PROXY:    "localhost,127.0.0.1,.domain.local"
HTTP_PROXY_REQUEST_FULLURI:  "false"
HTTPS_PROXY_REQUEST_FULLURI: "false"
```
If you don't need some variables or connect through a proxy at all, simply prefix these lines with a `#` to comment them out.
