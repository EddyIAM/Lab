# Docker Setup

The cluster is setup in a docker swarm and using the NAS as a common mount point and traefik as a reverse proxy for access.  

It currently runs...

## Guacamole

Access to the Lab boxes from the production network.

```yaml
version: '3.2'
services:
  guacd:
    image: guacamole/guacd
  guacamole:
    image: guacamole/guacamole
    depends_on:
      - guacd
      - postgres
    ports:
      - "8083:8080"
    environment:
      GUACD_HOSTNAME: "guacd"
      POSTGRES_HOSTNAME: "postgres.lab.net"
      POSTGRES_DATABASE: "guacamole_db"
      POSTGRES_USER: "guacamole_user"
      POSTGRES_PASSWORD: "password"
    labels:
      - "traefik.enable=true"
      - "traefik.http.services.guacamole.loadbalancer.server.port=8080"
      - "traefik.http.routers.guacamole.rule=Host(`guacamole.lab.net`)"
      - "traefik.http.routers.guacamole.entrypoints=web"
  postgres_db:
    image: postgres
    ports:
      - 5432:5432
    environment:
      - POSTGRES_PASSWORD=password
      - PGDATA=/var/lib/pgsql/data
    volumes:
      - '/mnt/docker/postgres/data:/var/lib/pgsql/data'
    deploy:
    placement:
      constraints:
        - node.labels.postgres == true
  adminer:
    image: adminer
    networks:
      - traefik-public
    ports:
      - 8082:8080
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.http.services.adminer.loadbalancer.server.port=8080"
        - "traefik.http.routers.adminer.rule=Host(`adminer.lab.net`)"
        - "traefik.http.routers.adminer.entrypoints=web"
      placement:
        constraints:
          - node.labels.postgres == true
networks:
  traefik-public:
    external: true
```

## Homer

Menu System for all the web based applications.

```yaml
version: "3"

services:
  homer:
    image: b4bz/homer:22.02.2
    networks:
      - traefik-public
    ports:
      - "8081:8080"
    environment:
      TZ: 'America/Los_Angeles'
    volumes:
      - '/mnt/docker/homer/assets:/www/assets'
    deploy:
    # placement:
      # constraints:
      # - node.labels.homer == true
    labels:
      # the most import label to tell traefik that it should pick up this service
      - "traefik.enable=true"
      # by default, traefik picks up the first exposed port, we can explicitly set it
      # to something else here
      - "traefik.http.services.homer.loadbalancer.server.port=8080"
      # tell traefik to send all requests to `homer.lab.net` to this service
      - "traefik.http.routers.homer.rule=Host(`homer.lab.net`)"
      # the default entry point is `web` which is HTTP
      - "traefik.http.routers.homer.entrypoints=web"
networks:
  traefik-public:
    external: true
```

## Dozzle

Log Reader.

```yaml
version: "3"
services:
  dozzle:
    container_name: dozzle
    image: amir20/dozzle:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    ports:
      - 9999:8080
    deploy:
      placement:
        constraints:
          - node.role == manager
```

## Grafana, InfluxDB, Telegraf, and Prometheus

Open source analytics & monitoring.

```yaml
version: "3.3"

services:
  telegraf:
    image: telegraf:latest
    hostname: "{{.Node.Hostname}}-Docker"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    configs:
      - source: telegraf.conf
        target: /etc/telegraf/telegraf.conf
    deploy:
      mode: global
  influxdb:
    image: influxdb:1.8.10-alpine
    networks:
    - traefik-public
    user: "0"
    ports:
      - 8086:8086
    configs:
      - source: influxdb.conf
        target: /etc/influxdb/influxdb.conf
    volumes:
      - /mnt/docker/influxdb:/var/lib/influxdb
    deploy:
      labels:
      - "traefik.enable=true"
      - "traefik.http.services.influxdb.loadbalancer.server.port=8086"
      - "traefik.http.routers.influxdb.rule=Host(`influxdb.lab.net`)"
      - "traefik.http.routers.influxdb.entrypoints=web"
      placement:
        constraints:
          - node.labels.influxdb == true
  grafana:
    image: grafana/grafana:latest
    networks:
    - traefik-public
    user: "0"
    ports:
      - 3000:3000
    volumes:
      - /mnt/docker/grafana:/var/lib/grafana
    deploy:
      labels:
      - "traefik.enable=true"
      - "traefik.http.services.grafana.loadbalancer.server.port=3000"
      - "traefik.http.routers.grafana.rule=Host(`grafana.lab.net`)"
      - "traefik.http.routers.grafana.entrypoints=web"
      placement:
        constraints:
          - node.labels.grafana == true
  prometheus:
    image: prom/prometheus:latest
    networks:
    - traefik-public
    ports:
    - 9090:9090
    command:
    - --config.file=/etc/prometheus/prometheus.yml
    configs:
      - source: prometheus.yml
        target: /etc/prometheus/prometheus.yml
    deploy:
      labels:
      - "traefik.enable=true"
      - "traefik.http.services.prometheus.loadbalancer.server.port=9090"
      - "traefik.http.routers.prometheus.rule=Host(`prometheus.lab.net`)"
      - "traefik.http.routers.prometheus.entrypoints=web"
      placement:
        constraints:
          - node.labels.prometheus == true
configs:
  influxdb.conf:
    file: ./influxdb/influxdb.conf
  telegraf.conf:
    file: ./telegraf/telegraf.conf
  prometheus.yml:
    file: ./prometheus/prometheus.yml
networks:
  traefik-public:
    external: true
```

## Nessus

Security Scanner.

```yaml
version: "3"

services:
  nessus:
    image: tenableofficial/nessus:latest
    # networks:
    #  - traefik-public
    ports:
      - "8834:8834/tcp"
    environment:
      TZ: 'America/Los_Angeles'
      ACTIVATION_CODE: 'ABCD-ABCD-ABCD-ABCD-ABCD'
      USERNAME: 'username'
      PASSWORD: 'password'
    deploy:
    placement:
      constraints:
        - node.labels.nessus == true
          # labels:
          # - "traefik.enable=true"
          # - "traefik.http.services.nessus.loadbalancer.server.port=8834"
          # - "traefik.http.routers.nessus.rule=Host(`nessus.lab.net`)"
          # - "traefik.http.routers.nessus.entrypoints=websecure"
          # networks:
          # traefik-public:
          # external: true
```

## PiHole

DNS

```yaml
version: "3"

# More info at https://github.com/pi-hole/docker-pi-hole/ and https://docs.pi-hole.net/
services:
  pihole:
    image: pihole/pihole:latest
    networks:
      - traefik-public
    ports:
      - "53:53/tcp"
      - "53:53/udp"
        #  - "67:67/udp"
      - "81:80/tcp"
    environment:
      TZ: 'America/Los_Angeles'
      WEBPASSWORD: 'password'
    # Volumes store your data between container upgrades
    volumes:
      - '/mnt/docker/pihole/etc/pihole:/etc/pihole'
      - '/mnt/docker/pihole/etc/dnsmasq.d:/etc/dnsmasq.d'
    # Recommended but not required (DHCP needs NET_ADMIN)
    #   https://github.com/pi-hole/docker-pi-hole#note-on-capabilities
    cap_add:
      - NET_ADMIN
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.http.services.pihole.loadbalancer.server.port=80"
        - "traefik.http.routers.pihole.rule=Host(`pihole.lab.net`)"
        - "traefik.http.routers.pihole.entrypoints=web"
      placement:
        constraints:
          - node.labels.pihole == true
networks:
  traefik-public:
    external: true
```

## Planka

Task board.

```yaml
version: '3'

services:
  planka:
    image: ghcr.io/plankanban/planka:latest
    command: >
      bash -c
        "for i in `seq 1 30`; do
          ./start.sh &&
          s=$$? && break || s=$$?;
          echo \"Tried $$i times. Waiting 5 seconds...\";
          sleep 5;
        done; (exit $$s)"
    restart: unless-stopped
    volumes:
      - user-avatars:/mnt/docker/planka/user-avatars
      - project-background-images:/mnt/docker/planka/project-background-images
      - attachments:/mnt/docker/planka/attachments
    ports:
      - 3030:1337
    environment:
      - BASE_URL=http://planka.lab.net:3030
      - TRUST_PROXY=0
      - DATABASE_URL=postgresql://postgres@postgres/planka
      - SECRET_KEY=SecretKey
    depends_on:
      - postgres
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.http.services.planka.loadbalancer.server.port=1337"
        - "traefik.http.routers.planka.rule=Host(`planka.lab.net`)"
        - "traefik.http.routers.planka.entrypoints=web"

  postgres:
    image: postgres:alpine
    restart: unless-stopped
    volumes:
      - '/mnt/docker/planka/postgresql/data:/var/lib/postgresql/data'
    environment:
      - POSTGRES_DB=planka
      - POSTGRES_HOST_AUTH_METHOD=trust

volumes:
  user-avatars:
  project-background-images:
  attachments:
```

## Portainer

Docker Manager.

```yaml
version: '3.2'
services:
  agent:
    image: portainer/agent:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    networks:
      - agent_network
    deploy:
      mode: global
      placement:
        constraints: [node.platform.os == linux]

  portainer:
    image: portainer/portainer-ce:latest
    command: -H tcp://tasks.agent:9001 --tlsskipverify
    ports:
      - "9443:9443"
      - "9000:9000"
      - "8000:8000"
    volumes:
      - portainer_data:/data
    networks:
      - agent_network
      - traefik-public
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints: [node.role == manager]
      labels:
      - "traefik.enable=true"
      - "traefik.http.services.portainer.loadbalancer.server.port=9000"
      - "traefik.http.routers.portainer.rule=Host(`portainer.lab.net`)"
      - "traefik.http.routers.portainer.entrypoints=web"

networks:
  agent_network:
    driver: overlay
    attachable: true
  traefik-public:
    external: true

volumes:
  portainer_data:
```

## SmokePing

Tracks Internet connectivity.

```yaml
version: "3"
services:
  smokeping:
    image: lscr.io/linuxserver/smokeping:latest
    container_name: smokeping
    environment:
      - PUID=1000
      - PGID=1000
      - 'America/Los_Angeles'
    volumes:
      - '/data/smokeping/config:/config'
      - '/data/smokeping/data:/data'
    ports:
      - 82:80
    restart: unless-stopped
```

## Traefik

Reverse Proxy.

```yaml
version: '3'

services:
  reverse-proxy:
    image: traefik:v2.3.4
    command:
      - "--providers.docker.endpoint=unix:///var/run/docker.sock"
      - "--providers.docker.swarmMode=true"
      - "--providers.docker.exposedbydefault=false"
      - "--providers.docker.network=traefik-public"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--api.insecure=true" # allow insecure access to the dashboard
      - "--log.level=DEBUG"
    ports:
      - 80:80
      - 443:443
      - 8080:8080
    volumes:
      # So that Traefik can listen to the Docker events
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - /data/traefik/certs:/tools/certs
      - /data/traefik/config.yml:/etc/traefik/dynamic_conf/conf.yml:ro
    networks:
      - traefik-public
    deploy:
      placement:
        constraints:
          - node.role == manager

networks:
  traefik-public:
    external: true
```

## Velociraptor

Digital forensic and incident response.

```yaml
version: '3'
services:
  velociraptor:
    container_name: velociraptor
    image: velociraptor-docker_velociraptor 
    build:
      context: ./
      dockerfile: Dockerfile
    volumes:
      - /mnt/docker/velociraptor:/velociraptor/:rw
    environment:
      - VELOX_USER=${VELOX_USER}
      - VELOX_PASSWORD=${VELOX_PASSWORD}
      - VELOX_ROLE=${VELOX_ROLE}
      - VELOX_SERVER_URL=${VELOX_SERVER_URL}
      - VELOX_FRONTEND_HOSTNAME=${VELOX_FRONTEND_HOSTNAME}
    ports:
      - "8003:8000"
      - "8001:8001"
      - "8889:8889"
    restart: unless-stopped
    deploy:
      labels:
      - "traefik.enable=true"
        #- "traefik.http.services.velociraptor.loadbalancer.server.port=8000"
      - "traefik.http.routers.velociraptor.rule=Host(`velociraptor.lab.net`)"
      - "traefik.http.routers.velociraptor.entrypoints=web"
```

.env file;

```text
VELOX_USER=admin
VELOX_PASSWORD=password
VELOX_ROLE=administrator
VELOX_SERVER_URL=https://velociraptor.lab.net:8000/
VELOX_FRONTEND_HOSTNAME=Velociraptor
```

## VSCode

VS Code web version.

```yaml
version: "3.2"
services:
  code-server:
    image: lscr.io/linuxserver/code-server:latest
    container_name: code-server
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Los_Angeles
      - PASSWORD= #optional
      - HASHED_PASSWORD= #optional
      - SUDO_PASSWORD= #optional
      - SUDO_PASSWORD_HASH= #optional
      - PROXY_DOMAIN=code-server.my.domain #optional
      - DEFAULT_WORKSPACE=/config/workspace #optional
    labels:
      - "traefik.enable=true"
      - "traefik.http.services.code-server.loadbalancer.server.port=8443"
      - "traefik.http.routers.code-server.rule=Host(`code-server.lab.net`)"
      - "traefik.http.routers.code-server.entrypoints=web"
    volumes:
      - /mnt/docker/code-server/config:/config
    ports:
      - 8443:8443
    restart: unless-stopped
```

## Whoogle

Google replacement.

```yaml
version: "3.7"
services:
  whoami:
    image: benbusby/whoogle-search:latest
    networks:
      - traefik-public
    ports:
      - 5001:5000
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.http.services.whoogle-search.loadbalancer.server.port=5000"
        - "traefik.http.routers.whoogle-search.rule=Host(`whoogle-search.lab.net`)"
        - "traefik.http.routers.whoogle-search.entrypoints=web"
networks:
  traefik-public:
    external: true
```
