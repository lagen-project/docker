FROM debian:jessie

MAINTAINER Pierre ROLLAND <roll.pierre@gmail.com>

RUN apt-get update && apt-get install -y nginx git npm curl
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

CMD ["nginx", "-g", "daemon off;"]

RUN git clone https://github.com/lagen-project/client.git /var/www/lagen-client && \
    rm /var/www/lagen-client/src/config.json
WORKDIR /var/www/lagen-client
COPY client-config.json /var/www/lagen-client/src/config.json
RUN npm install
RUN npm run setup

EXPOSE 80
EXPOSE 81
EXPOSE 443
