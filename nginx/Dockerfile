FROM php:7.1-fpm

MAINTAINER Pierre ROLLAND <roll.pierre@gmail.com>

# Retrieves arguments
ARG jwt_passphrase
ARG allowed_origins
ARG user
ARG password

# Set env varibales
ENV APP_ENV prod
ENV DATABASE_PATH var/data/data.sqlite
ENV JWT_PRIVATE_KEY_PATH 'config/jwt/private.pem'
ENV JWT_PUBLIC_KEY_PATH 'config/jwt/private.pem'
ENV JWT_PASSPHRASE "$jwt_passphrase"
ENV ALLOWED_ORIGINS "$allowed_origins"

# Install base packages
RUN apt-get update && \
    apt-get install -y \
    nginx \
    git \
    unzip \
    npm \
    curl \
    gnupg2 \
    apt-transport-https \
    ca-certificates \
    software-properties-common \
    supervisor

# Install docker
RUN curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | apt-key add -
RUN add-apt-repository -y \
   "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
   $(lsb_release -cs) \
   stable"
RUN apt-get update
RUN apt-get install -y docker-ce

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN composer --version

# Install node
RUN npm install -g n
RUN n lts

# Setup nginx conf
ADD nginx.conf /etc/nginx/
ADD lagen-api.conf /etc/nginx/sites-available/
ADD lagen-client.conf /etc/nginx/sites-available/
RUN ln -s /etc/nginx/sites-available/lagen-api.conf /etc/nginx/sites-enabled/lagen-api
RUN ln -s /etc/nginx/sites-available/lagen-client.conf /etc/nginx/sites-enabled/lagen-client
RUN rm /etc/nginx/sites-enabled/default

RUN usermod -u 1000 www-data

# Setup PHP conf
RUN rm /etc/localtime
RUN ln -s /usr/share/zoneinfo/Europe/Paris /etc/localtime
RUN "date"
RUN docker-php-ext-install pdo pdo_mysql

# Setup supervisor conf
ADD supervisord.conf /etc/supervisor/supervisord.conf
ADD install_worker.conf /etc/supervisor/conf.d/install_worker.conf
ADD php_fpm.conf /etc/supervisor/conf.d/php_fpm.conf
ADD sup_nginx.conf /etc/supervisor/conf.d/nginx.conf

# Install client
WORKDIR /var/www/lagen-client
RUN chown -R www-data:www-data .
RUN npm config set cache /var/www/lagen-client/.npm --global
USER www-data
RUN git clone https://github.com/lagen-project/client.git /var/www/lagen-client && \
    rm /var/www/lagen-client/src/config.json
COPY client-config.json /var/www/lagen-client/src/config.json
RUN npm install
RUN npm run setup
USER root

#Install api
WORKDIR /var/www/lagen-api
RUN chown -R www-data:www-data .
USER www-data
RUN git clone https://github.com/lagen-project/api.git /var/www/lagen-api
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

# Put root and www-data in docker and supervisor group
RUN addgroup --system supervisor
RUN usermod -aG docker root
RUN usermod -aG docker www-data
RUN usermod -aG supervisor www-data

EXPOSE 80
EXPOSE 81

CMD ["/usr/bin/supervisord", "-n"]
