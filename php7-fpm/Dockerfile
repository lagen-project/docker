FROM php:7.0-fpm

MAINTAINER Pierre ROLLAND <roll.pierre@gmail.com>

ARG jwt_passphrase
ARG allowed_origins
ARG user
ARG password

RUN apt-get update && apt-get install -y git unzip gnupg2 apt-transport-https ca-certificates software-properties-common supervisor
RUN curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | apt-key add -
RUN add-apt-repository -y \
   "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
   $(lsb_release -cs) \
   stable"
RUN apt-get update
RUN apt-get install -y docker-ce

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN composer --version

RUN rm /etc/localtime
RUN ln -s /usr/share/zoneinfo/Europe/Paris /etc/localtime
RUN "date"

RUN docker-php-ext-install pdo pdo_mysql

ADD install_worker.conf /etc/supervisor/conf.d/install_worker.conf
ADD php_fpm.conf /etc/supervisor/conf.d/php_fpm.conf

WORKDIR /var/www/lagen-api
RUN chown -R www-data:www-data .

USER www-data

RUN git clone https://github.com/lagen-project/api.git /var/www/lagen-api

ENV APP_ENV prod
ENV DATABASE_PATH var/data/data.sqlite
ENV JWT_PRIVATE_KEY_PATH 'config/jwt/private.pem'
ENV JWT_PUBLIC_KEY_PATH 'config/jwt/private.pem'
ENV JWT_PASSPHRASE "$jwt_passphrase"
ENV ALLOWED_ORIGINS "$allowed_origins"

RUN composer install --no-dev --optimize-autoloader
RUN mkdir -p var/data config/jwt
RUN php bin/console doctrine:database:create
RUN php bin/console doctrine:schema:update --force
RUN php bin/console app:user:create $user $password
RUN openssl genrsa -out config/jwt/private.pem -passout pass:$jwt_passphrase
RUN openssl rsa -pubout -in config/jwt/private.pem -out config/jwt/public.pem -passin pass:$jwt_passphrase
RUN ./bin/console cache:warmup --env=prod
RUN chown -R www-data:www-data var

USER root

RUN usermod -aG docker root
RUN usermod -aG docker www-data

CMD ["/usr/bin/supervisord", "-n"]
