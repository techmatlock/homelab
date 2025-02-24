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

Create docker network

```bash
docker network create proxy
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
