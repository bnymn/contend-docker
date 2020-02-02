FROM php:7.2-fpm-alpine

ENV PHP_IDE_CONFIG "serverName=contend.test"
ENV XDEBUG_CONFIG "idekey=PHPSTORM"

RUN docker-php-ext-install pdo_mysql

# Install imagick
RUN set -x \
    && apk add --no-cache --virtual .phpize-deps $PHPIZE_DEPS imagemagick-dev libtool \
    && pecl install imagick-3.4.4 \
    && docker-php-ext-enable imagick \
    && apk add --no-cache --virtual .imagick-runtime-deps imagemagick \
    && apk del .phpize-deps

# Install GD
RUN apk upgrade --update \\
    && apk add --no-cache \
        freetype \
        libpng \
        libjpeg-turbo \
        freetype-dev \
        libpng-dev \
        libjpeg-turbo-dev \
    && docker-php-ext-configure gd \
    --with-gd \
    --with-freetype-dir=/usr/include/ \
    --with-png-dir=/usr/include/ \
    --with-jpeg-dir=/usr/include/ \
    && NPROC=$(grep -c ^processor /proc/cpuinfo 2>/dev/null || 1) \
    && docker-php-ext-install -j${NPROC} gd \
    && apk del --no-cache freetype-dev libpng-dev libjpeg-turbo-dev

# Install Xdebug
RUN apk update \
    && apk add --no-cache g++ make autoconf \
    && docker-php-source extract \
    && pecl install xdebug \
    && echo "xdebug.remote_enable = true" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_host = host.docker.internal" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_port = 9000" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.remote_autostart = off" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && echo "xdebug.idekey = PHPSTORM" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini \
    && docker-php-ext-enable xdebug \
    && docker-php-source delete \
    && apk del --no-cache g++ make autoconf

# Install zip extension
RUN apk add --no-cache libzip-dev \
    && docker-php-ext-configure zip --with-libzip=/usr/include \
    && docker-php-ext-install zip

# Install Git
RUN apk add --no-cache git

# Install Openssh-Clien
RUN apk add --no-cache openssh-client

# Copy scripts
RUN mkdir -p /scripts
COPY ./scripts /scripts
RUN chmod +x /scripts/*

# Install PHP composer rool
RUN /scripts/install-composer.sh

WORKDIR /var/www

CMD ["php-fpm"]

EXPOSE 9000