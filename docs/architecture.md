# Architecture

*Two views of the system: how it runs today, and what the Phase 4 endgame looks like.*

---

## Current state (April 2026)

The server is physically installed on an open-bench configuration with a single user-facing role: serve local inference to downstream applications over the local network and through a temporary public tunnel. All AI processing happens on the host itself — no external AI provider is involved at any point.

```mermaid
flowchart TB
    subgraph EXT["External"]
        USER[User on Telegram]
        PHONE[Phone hotspot<br/>USB tethering]
        ISP[Fiber connection<br/>Bredband2 1000/1000<br/><i>planned</i>]
    end

    subgraph SERVER["Server — Debian 13"]
        TUNNEL[ngrok tunnel<br/><i>temporary</i>]
        N8N[n8n<br/>Docker :5678]
        OWUI[Open WebUI<br/>Docker :3000]
        OLLAMA[Ollama<br/>systemd :11434]
        GPU[RTX 3080 10 GB<br/>NVIDIA + CUDA 12.4]
    end

    USER -->|webhook| TUNNEL
    TUNNEL --> N8N
    N8N -->|HTTP| OLLAMA
    OWUI -->|HTTP| OLLAMA
    OLLAMA -->|GPU passthrough| GPU
    PHONE -.->|current| SERVER
    ISP -.->|planned| SERVER

    classDef planned stroke-dasharray: 5 5,stroke:#888
    classDef temporary stroke-dasharray: 2 2,stroke:#c60
    class ISP planned
    class TUNNEL temporary
```

**Notes on the current state:**

- The internet connection is currently USB tethering from a phone. A Bredband2 1000/1000 fiber subscription has been identified as the intended replacement. Until the fiber connection is installed, the server's public IP changes per session and the static-IP configuration is blocked.
- The ngrok tunnel is a temporary HTTPS entry point for inbound webhooks from Telegram. Its URL resets on every restart of the tunnel, which means the Telegram bot's webhook must be reconfigured each time. This is the single largest source of fragility in the current deployment and is next on the list to replace.
- Ollama runs as a native systemd service rather than in a container, bound to `0.0.0.0:11434` so the n8n and Open WebUI containers can reach it over the host network. This is a deliberate choice: GPU passthrough is simpler when the inference engine is on the host, and model files stored on the host SSD do not need to be mapped into a container.
- Open WebUI and n8n are each in their own Docker container. Both expose a web interface on a dedicated port and are only accessible from the local network today.

---

## Phase 4 endgame

Phase 4 adds shared infrastructure — storage, monitoring, secure remote access — and upgrades the inference capacity of the primary server. It does not add a second compute server; that is a separate, conditional future build.

```mermaid
flowchart TB
    subgraph EXT["External"]
        USER2[Users on Telegram,<br/>web, API clients]
        INTERNET[Internet via fiber]
    end

    subgraph GATEWAY["Auxiliary cluster — platform TBD"]
        NGINX[Nginx reverse proxy]
        WG[WireGuard VPN<br/>endpoint]
        PROM[Prometheus]
        GRAFANA[Grafana]
        UPTIME[Uptime Kuma]
    end

    subgraph SERVER2["Primary server — Debian 13"]
        N8N2[n8n]
        OWUI2[Open WebUI]
        OLLAMA2[Ollama]
        GPU2[RTX 3090 24 GB]
        RAM[64 GB DDR5]
    end

    subgraph STORAGE["NAS"]
        MODELS[/models/]
        DOCS[/documents/]
        BACKUPS[/backups/]
    end

    USER2 --> INTERNET
    INTERNET --> NGINX
    INTERNET --> WG
    NGINX --> N8N2
    NGINX --> OWUI2
    WG -.->|secure access| SERVER2
    N8N2 -->|HTTP| OLLAMA2
    OWUI2 -->|HTTP| OLLAMA2
    OLLAMA2 -->|GPU passthrough| GPU2
    OLLAMA2 -.->|model storage| MODELS
    N8N2 -.->|document input| DOCS
    SERVER2 -.->|nightly| BACKUPS
    PROM -.->|metrics| SERVER2
    PROM -.->|metrics| STORAGE
    GRAFANA -->|dashboards| PROM
    UPTIME -.->|health checks| SERVER2
```

**What changes between today and Phase 4:**

| Layer | Current | Phase 4 |
|---|---|---|
| Internet | Phone hotspot (USB) | Fiber 1000/1000 |
| Public entry | ngrok (temporary) | Nginx reverse proxy on the cluster |
| Remote access | SSH over LAN only | WireGuard VPN from anywhere |
| Inference GPU | RTX 3080 10 GB | RTX 3090 24 GB |
| System RAM | 32 GB | 64 GB |
| PSU | 750 W | 850 W (upgrade required for 3090) |
| Storage | Local NVMe only | Local NVMe + NAS (RAID) |
| Monitoring | `nvidia-smi` by hand | Prometheus + Grafana + Uptime Kuma |
| Enclosure | Open-bench | Corsair 4000D |
| Model ceiling | 9 B (comfortable) / 13 B (tight) | 30 B (comfortable) |

**Design principles for Phase 4:**

- **Compute and infrastructure are separated.** The primary server does one job — run inference. The auxiliary cluster does the everything-else work: proxy, VPN, monitoring. This means a failure in the monitoring stack cannot take down inference, and a reboot of the primary server does not break public access to other services.
- **The NAS is shared storage, not a backup.** Models, ingested documents, and outputs live on the NAS so that the primary server becomes stateless in principle — if the primary server dies, a replacement can re-mount the NAS and resume. Backups are a separate concern handled by nightly snapshots to a different volume on the NAS.
- **Nothing new is consumer-cloud dependent.** The public entry point is a self-hosted Nginx instance behind the fiber connection, not a third-party tunnel service. The VPN is WireGuard, self-hosted. The monitoring stack is self-hosted. This is consistent with the core value proposition of the project: private AI, on owned infrastructure, with no cloud dependency for the inference path.
- **The auxiliary cluster platform is deliberately unspecified.** It could be Raspberry Pi 5, Orange Pi 5 Plus, a Jetson Orin Nano, a mini PC (Intel N100 / N150), or a small used office PC. The decision is deferred until closer to purchase because 2026 small-compute pricing is volatile. The roles are fixed; the hardware is not.

---

## What is deliberately not in Phase 4

The diagram above stops at the boundary of what Server 1 (this build) can reasonably support. Everything beyond this point belongs to a separate, future, conditional build (Server 2) and is not documented here. In particular:

- **70 B class models** are outside the capacity of a 24 GB GPU and require either multi-GPU setups or different hardware entirely.
- **Dual-GPU configurations** are not possible on this motherboard at full bandwidth — the B650E MAX has only one PCIe x16 slot wired at full width.
- **Fine-tuning workloads** are not in scope for Server 1.

These belong to Server 2 planning, which is gated on business profitability rather than infrastructure readiness.

---

*Last updated: 24 April 2026*
