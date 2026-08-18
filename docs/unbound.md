# Unbound Documentation

[Unbound](https://nlnetlabs.nl/projects/unbound/about/) is a validating, recursive, and caching DNS resolver used in this stack as the upstream DNS server for Pi-hole.

---

## Simple Description

When you type a website address into your browser, your device needs to look up the numeric address that corresponds to that name — similar to looking up a phone number in a directory. Unbound is the service that performs that lookup on your behalf. Rather than relying on a third-party DNS provider (such as Google or Cloudflare), Unbound resolves addresses by querying the internet's authoritative DNS servers directly, starting from the root of the DNS hierarchy and working its way down until it finds the answer. This means no external provider ever sees all of your DNS queries. Once Unbound has looked up an address, it caches the result so that repeated lookups are answered immediately without going to the internet again.

---

## Technical Description

Unbound is a full recursive DNS resolver with DNSSEC validation. Instead of forwarding queries to a third-party resolver, it performs iterative resolution: it contacts root name servers, then top-level-domain (TLD) servers, then authoritative servers, following referrals until it obtains the final answer. This keeps DNS queries entirely within the local network and the destination authoritative servers, eliminating the privacy exposure of a forwarding-only resolver.

In this stack, Unbound listens on `127.0.0.1:5335` and acts as the upstream resolver for [Pi-hole](./pihole.md). Pi-hole forwards all allowed DNS queries to Unbound, which resolves them recursively and returns the result.

### Key capabilities

| Capability | Detail |
|---|---|
| Recursive resolution | Resolves queries from root servers down without relying on a third-party forwarder |
| DNSSEC validation | Cryptographically validates DNS responses using the chain of trust from root to authoritative servers |
| Response caching | Caches resolved records for the duration of their TTL, reducing latency for repeated queries |
| DNS-over-TLS / DNS-over-HTTPS | Supports encrypted DNS transports when configured |
| Access control | Configurable allow/deny rules for which clients may send queries |
| Local zone overrides | Custom DNS records that override or supplement public DNS data |
| Rate limiting | Protects against amplification attacks by limiting query rates per source |
| Minimal responses | Strips unnecessary additional sections from responses to reduce exposure of internal network information |

### DNS query flow (this stack)

```
Pi-hole (port 53)
    │
    │ Forwarded allowed queries (127.0.0.1:5335)
    ▼
Unbound (port 5335)
    │
    │ Iterative recursive resolution
    ▼
Root name servers (.)
    │
    ▼
TLD name servers (e.g., .com, .net, .org)
    │
    ▼
Authoritative name servers (e.g., ns1.example.com)
    │
    │ Authoritative answer (validated with DNSSEC)
    ▼
Unbound (cached and returned to Pi-hole)
    │
    ▼
Pi-hole → Client device
```

### Relevant configuration (this stack)

| Setting | Value | Description |
|---|---|---|
| Listening address | `127.0.0.1:5335` | Binds to localhost only; Pi-hole forwards queries to this address |
| `network_mode` | `host` | Container shares the host network stack so Pi-hole can reach `127.0.0.1:5335` |
| Configuration file | `./unbound/unbound.conf` | Mounted read-only into the container at `/usr/local/unbound/unbound.conf` |
| Healthcheck script | `/usr/local/unbound/sbin/healthcheck.sh` | Verified before Pi-hole is allowed to start (`depends_on: condition: service_healthy`) |

### Official resources

- [Unbound website](https://nlnetlabs.nl/projects/unbound/about/)
- [Unbound documentation](https://unbound.docs.nlnetlabs.nl/)
- [Unbound GitHub repository](https://github.com/NLnetLabs/unbound)
- [Pi-hole + Unbound setup guide](https://docs.pi-hole.net/guides/dns/unbound/)
