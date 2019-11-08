FROM php:7.3-fpm-alpine

RUN apk update; \
    apk upgrade;

RUN set -ex \
  && apk --no-cache add \
    postgresql-dev

RUN docker-php-ext-install pdo pdo_pgsql

COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

RUN adduser --uid 1001 --disabled-password --home /home/devuser devuser
RUN mkdir -p /home/devuser/.composer && \
    chown -R devuser:devuser /home/devuser
RUN addgroup devuser www-data
RUN addgroup devuser root

RUN chgrp -R  www-data /var/www/html
RUN chmod -R g+r /var/www/html/

# RUN su - devuser && cd /var/www/html && composer install && php artisan key:generate