# docker-compose.yml Documentation

This document describes the `docker-compose.yml` file used to run the DNS infrastructure stack, which consists of an [Unbound](https://nlnetlabs.nl/projects/unbound/about/) recursive DNS resolver and a [Pi-hole](https://pi-hole.net/) DNS sinkhole.

## Full File

```yaml
services:
  unbound:
    image: madnuttah/unbound:1.25.2-0
    container_name: unbound
    restart: unless-stopped
    network_mode: host
    environment:
      TZ: ${TZ}
    volumes:
      - ./unbound/unbound.conf:/usr/local/unbound/unbound.conf:ro
    healthcheck:
      test: /usr/local/unbound/sbin/healthcheck.sh
      interval: 60s
      timeout: 15s
      retries: 3
      start_period: 15s

  pihole:
    image: pihole/pihole:2026.07.2
    container_name: pihole
    restart: unless-stopped
    network_mode: host
    depends_on:
      unbound:
        condition: service_healthy
    environment:
      TZ: ${TZ}
      FTLCONF_webserver_api_password: ${FTLCONF_webserver_api_password}
      FTLCONF_dns_upstreams: '127.0.0.1#5335'
      FTLCONF_dns_listeningMode: ${FTLCONF_dns_listeningMode}
    volumes:
      - ./etc-pihole:/etc/pihole
      - ./etc-dnsmasq.d:/etc/dnsmasq.d
    cap_add:
      - NET_ADMIN
      - SYS_TIME
      - SYS_NICE
```

---

## `services`

The top-level `services` key defines each container that Docker Compose will create and manage as part of this stack.

- **`services`** — Declares the set of containers (services) that make up the application. Each named entry under `services` becomes a separate container.

---

## `unbound` Service

The `unbound` service runs a local recursive DNS resolver that Pi-hole uses as its upstream DNS server.

- **`image: madnuttah/unbound:1.25.2-0`** — Specifies the Docker image and tag to use for this container. This is a community image that packages the Unbound DNS resolver.
- **`container_name: unbound`** — Assigns an explicit name to the running container, overriding the default auto-generated name. Makes it easier to reference the container in logs and other commands.
- **`restart: unless-stopped`** — Configures the container restart policy. The container will automatically restart after failures or host reboots unless it was explicitly stopped by the user.
- **`network_mode: host`** — Attaches the container directly to the host's network stack instead of an isolated Docker network. The container shares the host's IP address and ports.
- **`environment`** — Passes environment variables into the container.
  - **`TZ: ${TZ}`** — Sets the container's timezone. The value is read from the `TZ` variable in the `.env` file.
- **`volumes`** — Mounts host paths or named volumes into the container filesystem.
  - **`./unbound/unbound.conf:/usr/local/unbound/unbound.conf:ro`** — Bind-mounts the local Unbound configuration file into the container at the expected path. The `:ro` flag makes the mount read-only inside the container.
- **`healthcheck`** — Defines a command Docker runs periodically to determine whether the container is healthy.
  - **`test: /usr/local/unbound/sbin/healthcheck.sh`** — The command executed to check health. A zero exit code means healthy; non-zero means unhealthy.
  - **`interval: 60s`** — How often Docker runs the health check command.
  - **`timeout: 15s`** — How long Docker waits for the health check command to complete before considering it a failure.
  - **`retries: 3`** — Number of consecutive failures required before the container is considered unhealthy.
  - **`start_period: 15s`** — Grace period after container start during which health check failures are not counted toward the retry limit.

---

## `pihole` Service

The `pihole` service runs Pi-hole, a network-level ad and tracking blocker that acts as the DNS server for the local network.

- **`image: pihole/pihole:2026.07.2`** — Specifies the official Pi-hole Docker image and tag to use.
- **`container_name: pihole`** — Assigns an explicit name to the running container.
- **`restart: unless-stopped`** — Automatically restarts the container after failures or host reboots unless explicitly stopped.
- **`network_mode: host`** — Shares the host's network stack, allowing Pi-hole to listen on the host's DNS port (53) directly.
- **`depends_on`** — Declares startup dependencies between services.
  - **`unbound`** — Pi-hole will not start until the `unbound` service satisfies the specified condition.
    - **`condition: service_healthy`** — Docker Compose waits until the `unbound` container's health check reports a healthy status before starting `pihole`.
- **`environment`** — Passes environment variables into the container.
  - **`TZ: ${TZ}`** — Sets the container's timezone, read from the `.env` file.
  - **`FTLCONF_webserver_api_password: ${FTLCONF_webserver_api_password}`** — Sets the password for the Pi-hole web interface and API. The value is read from the `.env` file to keep secrets out of version control.
  - **`FTLCONF_dns_upstreams: '127.0.0.1#5335'`** — Configures Pi-hole's upstream DNS resolver. Requests are forwarded to Unbound running on the host loopback address at port 5335.
  - **`FTLCONF_dns_listeningMode: ${FTLCONF_dns_listeningMode}`** — Controls which network interfaces Pi-hole's DNS server listens on. The value is read from the `.env` file.
- **`volumes`** — Mounts host directories into the container to persist data across container restarts.
  - **`./etc-pihole:/etc/pihole`** — Persists Pi-hole's configuration and data (block lists, custom DNS entries, etc.) on the host.
  - **`./etc-dnsmasq.d:/etc/dnsmasq.d`** — Persists custom dnsmasq configuration snippets used by Pi-hole's FTL DNS engine.
- **`cap_add`** — Grants additional Linux capabilities to the container beyond the default set.
  - **`NET_ADMIN`** — Allows the container to perform network administration tasks such as modifying routing tables and firewall rules, required by Pi-hole for DNS and DHCP management.
  - **`SYS_TIME`** — Allows the container to set the system clock, which Pi-hole's FTL engine may use for accurate timekeeping.
  - **`SYS_NICE`** — Allows the container to adjust process scheduling priorities, enabling Pi-hole's FTL process to raise its priority for lower-latency DNS responses.

---

## Official Docker Compose Documentation

The following links point to the official Docker Compose reference documentation for each key and setting used in this file.

| Key / Setting | Documentation Link |
|---|---|
| `services` | https://docs.docker.com/reference/compose-file/services/ |
| `image` | https://docs.docker.com/reference/compose-file/services/#image |
| `container_name` | https://docs.docker.com/reference/compose-file/services/#container_name |
| `restart` | https://docs.docker.com/reference/compose-file/services/#restart |
| `network_mode` | https://docs.docker.com/reference/compose-file/services/#network_mode |
| `environment` | https://docs.docker.com/reference/compose-file/services/#environment |
| `volumes` | https://docs.docker.com/reference/compose-file/services/#volumes |
| `healthcheck` | https://docs.docker.com/reference/compose-file/services/#healthcheck |
| `healthcheck.test` | https://docs.docker.com/reference/compose-file/services/#test |
| `healthcheck.interval` | https://docs.docker.com/reference/compose-file/services/#interval-timeout-start_period-start_interval |
| `healthcheck.timeout` | https://docs.docker.com/reference/compose-file/services/#interval-timeout-start_period-start_interval |
| `healthcheck.retries` | https://docs.docker.com/reference/compose-file/services/#retries |
| `healthcheck.start_period` | https://docs.docker.com/reference/compose-file/services/#interval-timeout-start_period-start_interval |
| `depends_on` | https://docs.docker.com/reference/compose-file/services/#depends_on |
| `depends_on.condition` | https://docs.docker.com/reference/compose-file/services/#condition |
| `cap_add` | https://docs.docker.com/reference/compose-file/services/#cap_add |
