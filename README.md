# Lagen's dockerfile

To ease your lagen setup, here is a docker compose file. Make it some changes to fit your needs, and build it :)

## Setup

1. Make sure docker-compose is installed.

2. Clone this repository :

  ```bash
    git clone https://github.com/lagen-project/docker.git && cd docker
  ```

3. Make some changes to fit your needs :

  - Change the JWT passphrases in `nginx/lagen-api.conf` and `php7-fpm/Dockerfile` (lines starting with `openssl` and `export JWT_PASSPHRASE`)
  - Change the "ALLOWED_ORIGINS" in `nginx/lagen-api.conf` and `php7-fpm/Dockerfile` to fit the address you'll configure for the client
  - Eventually change the "APP_SECRET" in `nginx/lagen-api.conf`
  - Change the "api" conf in `nginx/client-config.json` to fit the address you'll configure for the API
  - Change the user add command in `php7-fpm/Dockerfile` so that it creates a user with the credentials you want

4. First build & run

  ```bash
    docker-compose build && docker-compose up
  ```

5. Get the PHP's instance IP address

  ```bash
    docker inspect --format '' $(docker ps -f name=php -q) | grep IPAddress
  ```

6. Finalize Nginx conf

  Replace the `fastcgi_pass` line with the given IP address

7. Restart

  Hit ^C and rerun
  ```bash
    docker-compose build && docker-compose up
  ```

8. Now you can point API and client domain names on localhost, with ports 4242 (API) and 4243 (client). Enjoy.
