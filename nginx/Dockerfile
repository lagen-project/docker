FROM debian:jessie

MAINTAINER Pierre ROLLAND <roll.pierre@gmail.com>

RUN apt-get update && apt-get install -y nginx git npm curl gnupg2 apt-transport-https ca-certificates software-properties-common
RUN curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg | apt-key add -
RUN add-apt-repository -y \
   "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") \
   $(lsb_release -cs) \
   stable"
RUN apt-get update
RUN apt-get install -y docker-ce

RUN npm install -g n
RUN n lts

ADD nginx.conf /etc/nginx/
ADD lagen-api.conf /etc/nginx/sites-available/
ADD lagen-client.conf /etc/nginx/sites-available/

RUN ln -s /etc/nginx/sites-available/lagen-api.conf /etc/nginx/sites-enabled/lagen-api
RUN ln -s /etc/nginx/sites-available/lagen-client.conf /etc/nginx/sites-enabled/lagen-client
RUN rm /etc/nginx/sites-enabled/default

RUN echo "upstream php-upstream { server php:9000; }" > /etc/nginx/conf.d/upstream.conf

RUN usermod -u 1000 www-data

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

RUN usermod -aG docker root
RUN usermod -aG docker www-data

EXPOSE 80
EXPOSE 81
EXPOSE 443

CMD ["nginx", "-g", "daemon off;"]
