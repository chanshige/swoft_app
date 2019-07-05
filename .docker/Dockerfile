ARG VERSION=7.1.27
FROM php:${VERSION}-apache

ENV SWOOLE_VERSION=4.3.5

# overwrite timezone
RUN /bin/cp -f /usr/share/zoneinfo/Asia/Tokyo /etc/localtime && date

# php extensions install
RUN apt-get update \
    && apt-get install -y zlib1g-dev libpq-dev libicu-dev libpng-dev libssl-dev curl wget zip unzip git \
    && docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql \
    && docker-php-ext-install intl gd pdo_pgsql pgsql mbstring \
    && cp /usr/local/etc/php/php.ini-development /usr/local/etc/php/php.ini \
    && apt-get clean \
    && apt-get autoremove

# composer install
RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer \
    && composer self-update --clean-backups

# composer setup
ENV COMPOSER_ALLOW_SUPERUSER=1 \
    COMPOSER_NO_INTERACTION=1 \
    COMPOSER_HOME=/composer \
    PATH=$PATH:/composer/vendor/bin

# install xdebug
#RUN pecl install xdebug \
#    && docker-php-ext-enable xdebug

# install ext-redis
RUN pecl install redis \
    && docker-php-ext-enable redis

# swoole
RUN wget https://github.com/swoole/swoole-src/archive/v${SWOOLE_VERSION}.tar.gz -O swoole.tar.gz \
    && mkdir -p swoole-src \
    && tar -xf swoole.tar.gz -C swoole-src --strip-components=1 \
    && rm swoole.tar.gz \
    && cd swoole-src \
    && phpize \
    && ./configure \
    && make \
    && make install \
    && docker-php-ext-enable swoole

# apache:mod_rewrite
RUN a2enmod rewrite

WORKDIR /var/www/html
EXPOSE 18306 18307 18308

ENTRYPOINT ["php", "/var/www/html/bin/swoft", "http:start"]
