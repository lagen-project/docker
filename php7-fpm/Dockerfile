FROM php:7.0-fpm

MAINTAINER Pierre ROLLAND <roll.pierre@gmail.com>

ARG jwt_passphrase
ARG allowed_origins
ARG user
ARG password

RUN apt-get update && apt-get install -y git unzip

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN composer --version

RUN rm /etc/localtime
RUN ln -s /usr/share/zoneinfo/Europe/Paris /etc/localtime
RUN "date"

RUN docker-php-ext-install pdo pdo_mysql

WORKDIR /var/www/lagen-api

RUN git clone https://github.com/lagen-project/api.git /var/www/lagen-api
RUN export APP_ENV=prod && \
    export DATABASE_PATH=var/data/data.sqlite && \
    export APP_ENV=prod && \
    export JWT_PRIVATE_KEY_PATH='config/jwt/private.pem' && \
    export JWT_PUBLIC_KEY_PATH='config/jwt/private.pem' && \
    export JWT_PASSPHRASE="$jwt_passphrase" && \
    export ALLOWED_ORIGINS="$allowed_origins" && \
    composer install --no-dev --optimize-autoloader && \
    mkdir -p var/data config/jwt && \
    php bin/console doctrine:database:create && \
    php bin/console doctrine:schema:update --force && \
    php bin/console app:create-user $user $password
RUN openssl genrsa -out config/jwt/private.pem -passout pass:$jwt_passphrase
RUN openssl rsa -pubout -in config/jwt/private.pem -out config/jwt/public.pem -passin pass:$jwt_passphrase
