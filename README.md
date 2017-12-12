# Lagen's dockerfile

To ease your lagen setup, here is a docker compose file. Make it some changes to fit your needs, and build it :)

## Setup

#### 1. Make sure docker-compose is installed.

#### 2. Clone this repository :

  ```bash
    git clone https://github.com/lagen-project/docker.git && cd docker
  ```

#### 3. Make some changes to fit your needs :

  - Change the JWT passphrases in `nginx/lagen-api.conf` and `docker-compose.yml`
  - Change the "ALLOWED_ORIGINS" in `nginx/lagen-api.conf` and `docker-compose.yml` to fit the address you'll configure for the client
  - Eventually change the "APP_SECRET" in `nginx/lagen-api.conf`
  - Change the "api" conf in `nginx/client-config.json` to fit the address you'll configure for the API
  - Change the user and password in `docker-compose.yml` so that it creates a user with the credentials you want

#### 4. Build & run

  ```bash
    docker-compose build && docker-compose up
  ```

#### 5. Now you can point API and client domain names on localhost, with ports 4242 (API) and 4243 (client). Enjoy.
