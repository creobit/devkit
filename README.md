# DevProxy for local Docker development

Inspired by DevBox https://github.com/lbngoc/DevBox

## Requirements

- [Homebrew](https://brew.sh/)
- [Orb](https://orbstack.dev/) or [Docker](https://docs.docker.com/docker-for-mac/install/)

## Containers

- Dnsmasq
- Traefik v2
- Mailhog

## Quickstart

- Clone this repo

```sh
git clone https://github.com/creobit/devkit.git && \
cd devproxy
```

### 1. Setup local DNS, Root CA

- Setup MacOS to take into account our local docker resolver

```sh
sudo mkdir -p /etc/resolver && \
echo "nameserver 127.0.0.1" | sudo tee -a /etc/resolver/test > /dev/null
```

- Setup a local trusted Root CA and create a TLS certificate for using https in local (shout out to [mkcert](https://github.com/FiloSottile/mkcert)).

```sh
brew install mkcert
brew install nss # only if you use Firefox
mkcert -install
mkcert -cert-file certs/local.crt -key-file certs/local.key localhost "*.localhost" 127.0.0.1 ::1 devkit.test "*.devkit.test"
```

### 2. Docker & Traefik

```sh
touch traefik/acme.json && chmod 600 traefik/acme.json && \
docker network create proxy
```

Setup dashboard authentication (optional)

- Generate `USER:PASSWORD` for access Traefik dashboard - put your own values in <USER> and <PASSWORD> below

```sh
echo $(htpasswd -nb <USER> <PASSWORD>) | sed -e s/\\$/\\$\\$/g
```

- Replace USER:PASSWORD in `docker-compose.yml:42`
- Uncomment `docker-compose.yml:41` `docker-compose.yml:42`

Or we can create new file `docker-compose.override.yml`

```yml
version: "3"

services:
  traefik:
    labels:
      - "traefik.http.middlewares.traefik-auth.basicauth.users=USER:PASSWORD"
      - "traefik.http.routers.traefik.middlewares=traefik-auth"
```

Done, we need to start Traefik at first time

```sh
docker-compose up -d
```

Go on https://localhost or https://traefik.devkit.test you should have the traefik web dashboard serve over https

## Usage

- When starting any new projects, just use this sample config

```yml
version: "3"

services:
  wordpress:
    image: wordpress
    container_name: wordpress
    restart: unless-stopped
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: root
      WORDPRESS_DB_PASSWORD: root
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - ./public_html:/var/www/html
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.wordpress.entrypoints=http"
      - "traefik.http.routers.wordpress.rule=Host(`wordpress.devkit.test`)"

networks:
  proxy:
    external: true
    name: proxy
```

Then run `docker-compose up` then open browser and go to http://wordpress.devkit.test

## Example

I also put a sample config to start whoami container, that will prints OS information and HTTP request to output

```sh
docker-compose -f whoami.yml up
```

Go on https://whoami.devkit.test to see the result

### References

- https://github.com/lbngoc/DevBox
- https://github.com/FiloSottile/mkcert
- https://github.com/SushiFu/traefik-local
- https://medium.com/@containeroo/traefik-2-0-docker-a-simple-step-by-step-guide-e0be0c17cfa5
