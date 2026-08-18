# Pi-hole Documentation

[Pi-hole](https://pi-hole.net/) is a network-level ad and tracking blocker that acts as the DNS server for a local network.

---

## Simple Description

Pi-hole works like a gatekeeper for every website lookup on your network. When a device — a phone, laptop, smart TV, or anything else connected to your Wi-Fi — wants to visit a website, it first asks a DNS server to translate the site's name (for example, `example.com`) into a numeric address. Pi-hole sits in that path and checks each lookup against a list of known ad servers, trackers, and malicious domains. If the name is on the block list, Pi-hole simply refuses to answer, and the ad or tracker is never loaded. Lookups for legitimate websites pass through normally. Because blocking happens at the network level, every device on the network benefits without needing to install anything on each device individually.

---

## Technical Description

Pi-hole is a DNS sinkhole that runs on the local network and intercepts DNS queries from all connected clients. When a client resolves a hostname, the query is directed to Pi-hole instead of a public DNS resolver. Pi-hole evaluates the queried domain against one or more block lists (gravity lists) maintained in a local SQLite database. If the domain matches a block-list entry, Pi-hole returns a sinkhole response (typically `0.0.0.0` or `NXDOMAIN`) so the client never establishes a connection to the blocked host. Queries that do not match are forwarded to a configured upstream DNS resolver — in this stack, [Unbound](./unbound.md) running on `127.0.0.1:5335`.

Pi-hole's core DNS and DHCP engine is **FTL** (Faster Than Light), a fork of `dnsmasq` extended with real-time query logging, statistics collection, and a built-in web server that serves the admin dashboard. Query data is persisted to a local database, enabling per-client analytics and long-term trend views.

### Key capabilities

| Capability | Detail |
|---|---|
| DNS sinkholing | Blocks ad, tracker, and malicious domains network-wide at the DNS layer |
| Gravity (block lists) | Aggregates and deduplicates multiple upstream block lists into a single SQLite database |
| Per-client statistics | Tracks query counts, block rates, and query types per client IP |
| Allow/deny lists | Custom per-domain overrides that take precedence over gravity entries |
| Upstream forwarding | Forwards allowed queries to a configurable upstream resolver (Unbound in this stack) |
| Web admin dashboard | Real-time query log, statistics charts, and configuration UI served over HTTP |
| DHCP server (optional) | Can assign IP addresses to clients, enabling per-hostname tracking |
| API | REST API for querying statistics and managing lists programmatically |

### DNS query flow (this stack)

```
Client device
    │
    │ DNS query (port 53)
    ▼
Pi-hole (FTL/dnsmasq)
    │
    ├─ Domain on block list? → return sinkhole response (0.0.0.0 / NXDOMAIN)
    │
    └─ Domain allowed? → forward to Unbound (127.0.0.1:5335)
                              │
                              │ Recursive resolution
                              ▼
                        Authoritative DNS servers (internet)
```

### Relevant configuration (this stack)

| Setting | Value | Description |
|---|---|---|
| `FTLCONF_dns_upstreams` | `127.0.0.1#5335` | Upstream DNS resolver (Unbound on localhost port 5335) |
| `FTLCONF_dns_listeningMode` | set in `.env` | Controls which interfaces Pi-hole's DNS server listens on |
| `FTLCONF_webserver_api_password` | set in `.env` | Password for the web admin interface and REST API |
| `network_mode` | `host` | Container shares the host network stack to listen on port 53 |

### Official resources

- [Pi-hole website](https://pi-hole.net/)
- [Pi-hole documentation](https://docs.pi-hole.net/)
- [Pi-hole GitHub repository](https://github.com/pi-hole/pi-hole)
- [FTL (dnsmasq fork) source](https://github.com/pi-hole/FTL)
