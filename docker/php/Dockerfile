ARG PHP_VERSION=8.4
FROM php:${PHP_VERSION}-cli-alpine

ENV COMPOSER_ALLOW_SUPERUSER=1 \
    COMPOSER_NO_INTERACTION=1

RUN apk add --no-cache bash git linux-headers unzip $PHPIZE_DEPS \
    && docker-php-ext-install sockets pcntl \
    && apk del $PHPIZE_DEPS

COPY --from=composer:2 /usr/bin/composer /usr/local/bin/composer

# Composer cache lives on a named volume mounted here; world-writable so the
# container can run as the calling user and never create root-owned files on
# the host bind mount (/app).
RUN mkdir -p /tmp/composer && chmod 777 /tmp/composer

WORKDIR /app
