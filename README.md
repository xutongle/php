# Supported tags and respective `Dockerfile` links

* `7.1.6-cli`, `7.1-cli`, `7-cli`, `cli`, `7.1.6`, `7.1`, `7`, `latest` [(7.1/Dockerfile)](https://github.com/xutongle/php/blob/master/7.1/Dockerfile)
* `7.1.6-alpine`, `7.1-alpine`, `7-alpine`, `alpine` [(7.1/alpine/Dockerfile)](https://github.com/xutongle/php/blob/master/7.1/alpine/Dockerfile)
* `7.1.6-fpm`, `7.1-fpm`, `7-fpm`, `fpm` [(7.1/fpm/Dockerfile)](https://github.com/xutongle/php/blob/master/7.1/fpm/Dockerfile)
* `7.1.6-fpm-alpine`, `7.1-fpm-alpine`, `7-fpm-alpine`, `fpm-alpine` [(7.1/fpm/alpine/Dockerfile)](https://github.com/xutongle/php/blob/master/7.1/fpm/alpine/Dockerfile)
* `7.0.20-cli`, `7.0-cli`, `7.0.20`, `7.0` [(7.0/Dockerfile)](https://github.com/xutongle/php/blob/master/7.0/Dockerfile)
* `7.0.20-alpine`, `7.0-alpine` [(7.0/alpine/Dockerfile)](https://github.com/xutongle/php/blob/master/7.0/alpine/Dockerfile)
* `7.0.20-fpm`, `7.0-fpm` [(7.0/fpm/Dockerfile)](https://github.com/xutongle/php/blob/master/7.0/fpm/Dockerfile)
* `7.0.20-fpm-alpine`, `7.0-fpm-alpine` [(7.0/fpm/alpine/Dockerfile)](https://github.com/xutongle/php/blob/master/7.0/fpm/alpine/Dockerfile)
* `5.6.30-cli`, `5.6-cli`, `5-cli`, `5.6.30`, `5.6`, `5` [(5.6/Dockerfile)](https://github.com/xutongle/php/blob/master/5.6/Dockerfile)
* `5.6.30-alpine`, `5.6-alpine`, `5-alpine` [(5.6/alpine/Dockerfile)](https://github.com/xutongle/php/blob/master/5.6/alpine/Dockerfile)
* `5.6.30-fpm`, `5.6-fpm`, `5-fpm` [(5.6/fpm/Dockerfile)](https://github.com/xutongle/php/blob/master/5.6/fpm/Dockerfile)
* `5.6.30-fpm-alpine`, `5.6-fpm-alpine`, `5-fpm-alpine` [(5.6/fpm/alpine/Dockerfile)](https://github.com/xutongle/php/blob/master/5.6/fpm/alpine/Dockerfile)

# What is PHP?

PHP is a server-side scripting language designed for web development, but which can also be used as a general-purpose programming language. PHP can be added to straight HTML or it can be used with a variety of templating engines and web frameworks. PHP code is usually processed by an interpreter, which is either implemented as a native module on the web-server or as a common gateway interface (CGI).

# How to use this image.

## With Command Line

For PHP projects run through the command line interface (CLI), you can do the following.

Create a `Dockerfile` in your PHP project

```bash
FROM xutongle/php:7.0-cli
COPY . /usr/src/myapp
WORKDIR /usr/src/myapp
CMD [ "php", "./your-script.php" ]
```

Then, run the commands to build and run the Docker image:

```bash
$ docker build -t my-php-app .
$ docker run -it --rm --name my-running-app my-php-app
```

# Run a single PHP script

For many simple, single file projects, you may find it inconvenient to write a complete Dockerfile. In such cases, you can run a PHP script by using the PHP Docker image directly:

    $ docker run -it --rm --name my-running-script -v "$PWD":/usr/src/myapp -w /usr/src/myapp xutongle/php:7.0-cli php your-script.php

# How to install more PHP extensions

We provide the helper scripts `docker-php-ext-configure`, `docker-php-ext-install`, and `docker-php-ext-enable` to more easily install PHP extensions.

In order to keep the images smaller, PHP's source is kept in a compressed tar file. To facilitate linking of PHP's source with any extension, we also provide the helper script docker-php-source to easily extract the tar or delete the extracted source. Note: if you do use docker-php-source to extract the source, be sure to delete it in the same layer of the docker image.

```bash
FROM php:7.0-apache
RUN docker-php-source extract \
    # do important things \
    && docker-php-source delete
```

# PHP Core Extensions

For example, if you want to have a PHP-FPM image with `iconv`, `mcrypt` and `gd` extensions, you can inherit the base image that you like, and write your own `Dockerfile` like this:

```bash
FROM xutongle/php:7.0-fpm
RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng12-dev \
    && docker-php-ext-install -j$(nproc) iconv mcrypt \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd
```

Remember, you must install dependencies for your extensions manually. If an extension needs custom configure arguments, you can use the `docker-php-ext-configure` script like this example. There is no need to run `docker-php-source` manually in this case, since that is handled by the configure and install scripts.

# PECL extensions

Some extensions are not provided with the PHP source, but are instead available through PECL. To install a PECL extension, use `pecl install` to download and compile it, then use `docker-php-ext-enable` to enable it:

```bash
FROM xutongle/php:7.1-fpm
RUN pecl install redis-3.1.0 \
    && pecl install xdebug-2.5.0 \
    && docker-php-ext-enable redis xdebug
    
FROM xutongle/php:5.6-fpm
RUN apt-get update && apt-get install -y libmemcached-dev zlib1g-dev \
    && pecl install memcached-2.2.0 \
    && docker-php-ext-enable memcached
```
