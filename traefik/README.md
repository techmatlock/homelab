For reference, the following folder structure was used:

```json
./traefik
├── data
│   ├── config.yaml
│   └── traefik.yaml
└── docker-compose.yaml
└── .env
```

Create Docker network

```bash
docker network create proxy
```

Make sure you have htpasswd installed.

```
sudo apt update
sudo apt install apache2-utils
```

```
echo $(htpasswd -nB user) | sed -e s/\\$/\\$\\$/g
```

```
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

#On the traefik server:
curl -k https://10.10.10.3:8006
```
