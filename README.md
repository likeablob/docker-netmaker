# docker-netmaker

A slightly modified version of `docker-compose`-ed [Netmaker](https://github.com/gravitl/netmaker) stack.

Modifications:

- Stick to Traefik instead of Caddy, to make things kept in a single yaml file and customizable through `.env`.
- DuckDNS integration:
  - DNS-01 challenge with Traefik (`acme.dnschallenge.provider=duckdns`)
  - Update A records with [`linuxservers/duckdns`](https://hub.docker.com/r/linuxserver/duckdns)
- HTTPS on the non standard port (`:2443`) (mainly for hosting other services at `:443`).
- No HTTP (`:80`) required.
- Bind mounts (`./data`, `./letsencrypt`) for better portability.
- Log rotation.

## Setup

- Ensure you have opened `2443/tcp` and `51821-51830/udp` on your server.

```sh
$ cp .env.example .env
$ editor .env

$ rsync -avP ../docker-netmaker/ remote:~/.netmaker
$ ssh remote
$ docker network create traefik
$ cd ~/.netmaker && docker compose up -d
```

Then the resources will be available at;

- Dashboard: https://nmui.${NETMAKER_BASE_DOMAIN}:2443
- API: https://nmapi.${NETMAKER_BASE_DOMAIN}:2443
- Broker: wss://mq.${NETMAKER_BASE_DOMAIN}:2443
- Traefik dashboard: http://localhost:2080

_Tips:_ You may want to comment out the following lines to secure the dashboard during the setup.

```yaml
- traefik.http.middlewares.secure-ips-ui.ipwhitelist.sourcerange=192.168.0.0/16,172.16.0.0/12,10.0.0.0/8
- traefik.http.routers.netmaker-ui.middlewares=secure-ips-ui@docker
```

## Upstream

- `docker-compose.yml`
  - https://github.com/gravitl/netmaker/blob/5384ff14e2317360fa38ee63cef5ba0809b1f85f/compose/docker-compose.reference.yml
- `config/mosquitto.conf`
  - https://raw.githubusercontent.com/gravitl/netmaker/v0.17.1/docker/mosquitto.conf
- `config/wait.sh`
  - https://raw.githubusercontent.com/gravitl/netmaker/v0.17.1/docker/wait.sh
