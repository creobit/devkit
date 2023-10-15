# Reverse Proxy for local Docker development with SSL

Inspired by DevBox https://github.com/lbngoc/DevBox

- Use \*.dev.local for all your projects for easy SSL
- DNSmasq for dev.local domain resolution
- Traefik for easy reverse proxy for your docker projects
- Redirect http to https by default (defined in traefik/traefik.yml)
- Mailhog as a bonus

## Requirements

- [Homebrew](https://brew.sh/)
- [Orb](https://orbstack.dev/) or [Docker Desktop](https://docs.docker.com/docker-for-mac/install/)

## Containers

- Dnsmasq
- Traefik v2
- Mailhog

## Quickstart

- Clone this repo

```sh
git clone https://github.com/creobit/devkit.git && \
cd devkit
```

### 1. Setup local DNS, Root CA

- Setup MacOS to use dnsmasq for domain: \*.dev.local

```sh
sudo mkdir -p /etc/resolver && \
echo "nameserver 127.0.0.1" | sudo tee -a /etc/resolver/dev.local > /dev/null
```

- Setup a local trusted Root CA and create a TLS certificate for using https in dev.local

```sh
brew install mkcert
brew install nss # only if you use Firefox
mkcert -install
mkcert -cert-file certs/local.crt -key-file certs/local.key localhost "*.localhost" 127.0.0.1 ::1 dev.local "*.dev.local"
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

Or we can create new file `docker-compose.private.yml`

```yml
version: "3"

services:
  traefik:
    labels:
      - "traefik.http.middlewares.traefik-auth.basicauth.users=USER:PASSWORD"
      - "traefik.http.routers.traefik.middlewares=traefik-auth"
```

You are ready to start your containers

```sh
docker-compose up -d
```

Go to https://traefik.dev.local or https://localhost

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
      - "traefik.http.routers.wordpress.rule=Host(`wordpress.dev.local`)"
    networks:
      - proxy

networks:
  proxy:
    external: true
```

Then run `docker-compose up -d` then open browser and go to http://wordpress.dev.local

## Example

I also put a sample config to start whoami container, that will prints OS information and HTTP request to output

```sh
docker-compose -f whoami.yml up -d
```

Go on https://whoami.dev.local to see the result

### References

- https://github.com/lbngoc/DevBox
- https://github.com/FiloSottile/mkcert
- https://github.com/SushiFu/traefik-local
- https://medium.com/@containeroo/traefik-2-0-docker-a-simple-step-by-step-guide-e0be0c17cfa5
