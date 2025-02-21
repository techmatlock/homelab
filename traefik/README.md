For reference, the following folder structure was used:

```json
./traefik
├── data
│   ├── acme.json
│   ├── config.yml
│   └── traefik.yml
└── cf_api_token.txt
└── docker-compose.yml
```

Create a folder to hold traefik files

```json
mkdir traefik
cd traefik
```

Create docker compose file and add contents

```
---
services:
  traefik:
    image: traefik:v3.3.3
    container_name: traefik
    restart: unless-stopped
    security_opt:
      - no-new-privileges:true
    networks:
      - proxy
    ports:
      - 80:80
      - 443:443
      # - 443:443/tcp # Uncomment if you want HTTP3
      # - 443:443/udp # Uncomment if you want HTTP3
    environment:
      CF_DNS_API_TOKEN_FILE: /run/secrets/cf_api_token # note using _FILE for docker secrets
      # CF_DNS_API_TOKEN: ${CF_DNS_API_TOKEN} # if using .env
      TRAEFIK_DASHBOARD_CREDENTIALS: ${TRAEFIK_DASHBOARD_CREDENTIALS}
    secrets:
      - cf_api_token
    env_file: .env # use .env
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - ./data/traefik.yaml:/traefik.yaml:ro
      - ./data/acme.json:/acme.json
      - ./data/config.yaml:/config.yaml:ro
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.traefik.entrypoints=http"
      - "traefik.http.routers.traefik.rule=Host(`traefik-dashboard.homelab.lan`)"
      - "traefik.http.middlewares.traefik-auth.basicauth.users=${TRAEFIK_DASHBOARD_CREDENTIALS}"
      - "traefik.http.middlewares.traefik-https-redirect.redirectscheme.scheme=https"
      - "traefik.http.middlewares.sslheader.headers.customrequestheaders.X-Forwarded-Proto=https"
      - "traefik.http.routers.traefik.middlewares=traefik-https-redirect"
      - "traefik.http.routers.traefik-secure.entrypoints=https"
      - "traefik.http.routers.traefik-secure.rule=Host(`traefik-dashboard.local.example.com`)"
      - "traefik.http.routers.traefik-secure.middlewares=traefik-auth"
      - "traefik.http.routers.traefik-secure.tls=true"
      - "traefik.http.routers.traefik-secure.tls.certresolver=cloudflare"
      - "traefik.http.routers.traefik-secure.tls.domains[0].main=local.example.com"
      - "traefik.http.routers.traefik-secure.tls.domains[0].sans=*.local.example.com"
      - "traefik.http.routers.traefik-secure.service=api@internal"

secrets:
  cf_api_token:
    file: ./cf_api_token.txt

networks:
  proxy:
    external: true
```

Data folder

```
mkdir data
cd data
touch acme.json
chmod 600 acme.json
```

Create docker network

```bash
docker network create proxy
```

CloudFlare API Token Secret

```
touch cf_api_token.txt
vim cf_api_token.txt
```

Traefik Dashboard Password & .env
Make sure you have htpasswd installed.

To install on Linux

```
sudo apt update
sudo apt install apache2-utils
```

```
echo $(htpasswd -nB user) | sed -e s/\\$/\\$\\$/g
```

```
touch .env
vim .env
```

paste your credential pair:

e.g.

```
TRAEFIK_DASHBOARD_CREDENTIALS=user:$$2y$$05$$lSaEi.G.aIygyXRdiFpt7OqmUMW9QUG5I1N.j0bXoXxIjxQmoGOWu
```

Start the stack

```bash
docker compose up -d --force-recreate
```

Troubleshooting

```
docker ps
docker logs traefik
docker exec -it traefik /bin/sh
```
