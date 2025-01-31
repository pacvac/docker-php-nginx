# Docker PHP-FPM 7.4 & Nginx 1.20 on Alpine Linux

Example PHP-FPM 7.4 & Nginx 1.20 container image for Docker, build on [Alpine Linux](https://www.alpinelinux.org/).

Repository: https://github.com/pacvac/alpine-nginx-php

-   Built on the lightweight and secure Alpine Linux distribution
-   Multi-platform, supporting AMD4, ARMv6, ARMv7, ARM64
-   Very small Docker image size (+/-40MB)
-   Uses PHP 7.4 for better performance, lower CPU usage & memory footprint
-   Optimized for 100 concurrent users
-   Optimized to only use resources when there's traffic (by using PHP-FPM's `on-demand` process manager)
-   The services Nginx, PHP-FPM and supervisord run under a non-privileged user (nobody) to make it more secure
-   The logs of all the services are redirected to the output of the Docker container (visible with `docker logs -f <container name>`)
-   Follows the KISS principle (Keep It Simple, Stupid) to make it easy to understand and adjust the image to your needs

[![Docker Pulls](https://img.shields.io/docker/pulls/pacvac/alpine-nginx-php.svg)](https://hub.docker.com/r/pacvac/alpine-nginx-php/)
![nginx 1.20](https://img.shields.io/badge/nginx-1.20-brightgreen.svg)
![php 7.4](https://img.shields.io/badge/php-7.4-brightgreen.svg)
![License MIT](https://img.shields.io/badge/license-MIT-blue.svg)

## Goal of this project

The goal of this container image is to provide an example for running Nginx and PHP-FPM in a container which follows
the best practices and is easy to understand and modify to your needs.

## Usage

Start the Docker container:

    docker run -p 80:8080 pacvac/alpine-nginx-php

See the PHP info on http://localhost, or the static html page on http://localhost/test.html

Or mount your own code to be served by PHP-FPM & Nginx

    docker run -p 80:8080 -v ~/my-codebase:/var/www/html pacvac/alpine-nginx-php

## Configuration

In [config/](config/) you'll find the default configuration files for Nginx, PHP and PHP-FPM.
If you want to extend or customize that you can do so by mounting a configuration file in the correct folder;

Nginx configuration:

    docker run -v "`pwd`/nginx-server.conf:/etc/nginx/conf.d/server.conf" pacvac/alpine-nginx-php

PHP configuration:

    docker run -v "`pwd`/php-setting.ini:/etc/php8/conf.d/settings.ini" pacvac/alpine-nginx-php

PHP-FPM configuration:

    docker run -v "`pwd`/php-fpm-settings.conf:/etc/php8/php-fpm.d/server.conf" pacvac/alpine-nginx-php

_Note; Because `-v` requires an absolute path I've added `pwd` in the example to return the absolute path to the current directory_

## Adding composer

If you need [Composer](https://getcomposer.org/) in your project, here's an easy way to add it.

```Dockerfile
FROM pacvac/alpine-nginx-php:latest

# Install composer from the official image
COPY --from=composer /usr/bin/composer /usr/bin/composer

# Run composer install to install the dependencies
RUN composer install --optimize-autoloader --no-interaction --no-progress
```

### Building with composer

If you are building an image with source code in it and dependencies managed by composer then the definition can be improved.
The dependencies should be retrieved by the composer but the composer itself (`/usr/bin/composer`) is not necessary to be included in the image.

```Dockerfile
FROM composer AS composer

# copying the source directory and install the dependencies with composer
COPY <your_directory>/ /app

# run composer install to install the dependencies
RUN composer install \
  --optimize-autoloader \
  --no-interaction \
  --no-progress

# continue stage build with the desired image and copy the source including the
# dependencies downloaded by composer
FROM pacvac/alpine-nginx-php
COPY --chown=nginx --from=composer /app /var/www/html
```
