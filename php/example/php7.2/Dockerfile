FROM php:7.2.34-apache

ENV COMPOSER_ALLOW_SUPERUSER=1

# Debian Buster est EOL : utiliser les archives Debian
RUN sed -i 's/deb.debian.org/archive.debian.org/g' /etc/apt/sources.list \
    && sed -i 's/security.debian.org/archive.debian.org/g' /etc/apt/sources.list \
    && sed -i '/buster-updates/d' /etc/apt/sources.list \
    && sed -i '/buster\/updates/d' /etc/apt/sources.list \
    && echo 'Acquire::Check-Valid-Until "false";' > /etc/apt/apt.conf.d/99no-check-valid-until

RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        locales \
        git \
        wget \
        sudo \
        libicu-dev \
        libpng-dev \
        libxml2-dev \
        libzip-dev \
        libonig-dev \
        libxslt-dev \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libssl-dev \
        librabbitmq-dev \
    && rm -rf /var/lib/apt/lists/*

# Locales
RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen \
    && echo "fr_FR.UTF-8 UTF-8" >> /etc/locale.gen \
    && locale-gen

ENV LANG=fr_FR.UTF-8 \
    LANGUAGE=fr_FR:fr \
    LC_ALL=fr_FR.UTF-8

# PHP extensions
RUN docker-php-ext-configure gd \
        --with-freetype-dir=/usr/include/ \
        --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) \
        bcmath \
        calendar \
        dom \
        gd \
        intl \
        mbstring \
        opcache \
        pdo \
        pdo_mysql \
        xsl \
        zip

# APCu - version compatible PHP 7.2
RUN wget -q https://pecl.php.net/get/apcu-5.1.21.tgz \
    && tar -xzf apcu-5.1.21.tgz \
    && cd apcu-5.1.21 \
    && phpize \
    && ./configure \
    && make -j$(nproc) \
    && make install \
    && docker-php-ext-enable apcu \
    && cd / \
    && rm -rf /apcu-5.1.21 /apcu-5.1.21.tgz

# AMQP
ADD https://github.com/mlocati/docker-php-extension-installer/releases/latest/download/install-php-extensions \
    /usr/local/bin/install-php-extensions

RUN chmod +x /usr/local/bin/install-php-extensions \
    && install-php-extensions amqp

# Xdebug - PHP 7.2 compatible
RUN pecl install xdebug-3.1.6 \
    && docker-php-ext-enable xdebug

# Composer
COPY --from=composer:2.7.1 /usr/bin/composer /usr/bin/composer

# Composer directory
RUN mkdir -p /var/www/.composer \
    && chown -R www-data:www-data /var/www/.composer

# Match host user UID
RUN usermod -u 1000 www-data

WORKDIR /var/www/app