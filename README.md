# Private AI Server

A documentation repository for a self-hosted AI inference server running on consumer hardware in Gothenburg, Sweden. The system runs large language models locally with no cloud dependency and no data leaving the network.

**Status:** Phase 1 operational — GPU-accelerated inference and first automated workflow in production (April 2026).

---

## Overview

The goal of this project is to build and operate a private AI platform capable of serving local language models to downstream applications — chat interfaces, automation workflows, document processing pipelines — entirely on owned infrastructure. The project is being built in public as a technical portfolio and as groundwork for a private-AI services business targeting automotive HIL/V&V engineering teams.

The build is deliberately staged across four phases to spread cost and allow each layer to stabilise before the next is added:

- **Phase 1 (complete)** — Core server, GPU, local inference, first automation workflow
- **Phase 2** — RAM expansion to 64 GB for larger context windows
- **Phase 3** — GPU upgrade to 24 GB VRAM for 13B–30B class models
- **Phase 4** — Additional infrastructure (NAS, monitoring cluster, VPN gateway)

---

## Current state (April 2026)

| Layer | Component | Status |
|---|---|---|
| OS | Debian 13 (Trixie) | Operational |
| GPU stack | NVIDIA driver 550 + CUDA 12.4 | Operational |
| Container runtime | Docker 26.1.5 + Compose | Operational |
| Inference engine | Ollama (GPU-accelerated) | Operational |
| Models available | 8 models, 7B–9B parameters | Loaded |
| Web interface | Open WebUI (Docker) | Operational |
| Automation | n8n (Docker) | Operational |
| First workflow | Telegram bot → n8n → Ollama → Telegram | Operational |
| Document Q&A workflow | PDF via Telegram → local inference → reply | In development |
| Static IP + ethernet | — | Pending |
| Persistent public URL | — | Pending (ngrok temporary) |
| Enclosure | — | Pending (open-bench) |

---

## Hardware

All prices in Swedish kronor (SEK), as purchased in early 2026.

| Component | Specification | Source | Price |
|---|---|---|---|
| CPU | AMD Ryzen 9 7900 (12 cores / 24 threads, 65 W) | Webhallen | 3 179 kr |
| Motherboard | ASUS B650E MAX Gaming WiFi (AM5, DDR5, PCIe 5.0) | Webhallen | 1 049 kr |
| RAM | Crucial Pro OC 32 GB (2×16 GB) DDR5-6000 CL36 EXPO | Elgiganten | 3 790 kr |
| Storage | Kingston Fury Renegade 2 TB NVMe Gen 4 | Webhallen | 3 999 kr |
| PSU | ASUS TUF Gaming 750 W Gold | Webhallen | 949 kr |
| CPU cooler | AMD Wraith Stealth (included with CPU) | — | 0 kr |
| Thermal paste | Arctic MX-4 | Webhallen | 99 kr |
| GPU | ASUS RTX 3080 10 GB TUF Gaming (used) | SweClockers | 3 009 kr |
| **Total Phase 1** | | | **16 074 kr** |

A detailed bill of materials, including planned upgrades for Phases 2–4, is documented in [`docs/hardware-bom.md`](docs/hardware-bom.md).

---

## Software stack

The system is built on a foundation-up model where each layer is independent and replaceable:

- **Debian 13** as the host OS, chosen after Ubuntu 24.04 Server was abandoned due to persistent ACPI and cloud-init issues on the AM5 platform. The switch reduced install time from several debugging sessions to under 20 minutes. A full account of the decision is in [`docs/ubuntu-to-debian.md`](docs/ubuntu-to-debian.md).
- **NVIDIA proprietary drivers** (550.163.01) with CUDA 12.4 and the NVIDIA Container Toolkit, enabling GPU passthrough into Docker containers.
- **Ollama** as the inference engine, running as a systemd service bound to `0.0.0.0:11434` to allow other containers on the host to reach it.
- **Docker 26.1.5** hosting the user-facing services: Open WebUI (port 3000) and n8n (port 5678).
- **ngrok** as a temporary HTTPS tunnel for inbound webhooks. A persistent replacement is planned once a fixed internet connection is in place — the chosen approach will be a self-hosted Cloudflare Tunnel or WireGuard, depending on how the network layer evolves.

A sanitised `docker-compose.yml` representing the running stack is available at [`docker/docker-compose.yml`](docker/docker-compose.yml).

---

## Models

Eight models are currently pulled into Ollama and verified on the RTX 3080 (10 GB VRAM). The selection covers the main use cases the project is designed around:

| Model | Size | Role |
|---|---|---|
| Mistral 7B | 7.2 B | General-purpose reasoning and chat |
| Qwen2.5 7B | 7.6 B | Multilingual (Arabic, French, English) |
| Llama 3.1 8B | 8.0 B | Instruction following, agents |
| Gemma 2 9B | 9.2 B | Document and long-form writing |
| DeepSeek Coder 6.7B | 6.7 B | Code generation and review |
| Qwen2.5 Coder 7B | 7.6 B | Multilingual code tasks |
| Phi-3 Mini 3.8B | 3.8 B | Lightweight, low-latency responses |
| LLaVA 7B | 7.0 B | Vision / image understanding |

Each model is loaded on demand and unloaded after a configurable idle period (default 10 minutes). With Mistral 7B warm in VRAM, end-to-end response time from Telegram to Telegram is approximately 2 seconds.

---

## Workflows

### First operational workflow — private AI chat via Telegram

The first production workflow is a private AI chat interface delivered through Telegram:

1. A user sends a message to a Telegram bot
2. An HTTPS tunnel forwards the webhook to n8n running on the server
3. The n8n workflow calls Ollama with the incoming message
4. Mistral 7B generates a response on the local GPU
5. The response is sent back to Telegram as a bot reply

No third-party AI provider is involved at any point in the pipeline. All inference and data processing happen on hardware physically located in the owner's home.

A short demonstration of this workflow is available at [`media/demo.mp4`](media/demo.mp4) *(pending upload)*.

### Document Q&A system — under construction

A document Q&A system is being built in phases. The first pipeline currently under development receives PDF files through Telegram, processes them against a local model, and returns an answer in the same Telegram conversation. Subsequent phases will expand both the set of supported document formats and the set of input channels.

The underlying pattern — untrusted documents in, structured answers out, nothing leaving the local network during inference — is designed to support professional use cases such as technical-specification review, test-report summarisation, and bilingual document triage in engineering contexts.

---

## Documentation

| Document | Status |
|---|---|
| [Architecture overview](docs/architecture.md) | Pending |
| [Ubuntu to Debian: a switch driven by ACPI](docs/ubuntu-to-debian.md) | Pending |
| [Hardware bill of materials (Phases 1–4)](docs/hardware-bom.md) | Pending |
| [Docker Compose (sanitised)](docker/docker-compose.yml) | Pending |

Documents are added as each deliverable is completed. This README is updated in parallel.

---

## Roadmap

**Near-term (next 1–2 months):**
- Migrate from ngrok to a persistent public entry point
- Transition from USB tethering to wired Ethernet
- Deploy Portainer for Docker management
- Install the server in a Corsair 4000D enclosure
- Expand the document Q&A workflow to additional formats and input channels

**Mid-term (3–6 months):**
- Expand RAM to 64 GB (identical Crucial kit)
- Upgrade GPU to a 24 GB class card (used RTX 3090) to run 13B–30B models
- PSU upgrade to 850 W class as a prerequisite for the GPU upgrade
- Introduce automated backups and Prometheus/Grafana monitoring

**Longer-term:**
- NAS integration for model and data storage
- Small auxiliary cluster for gateway, monitoring, and edge-AI tasks
- WireGuard VPN for secure remote access

---

## About

This project is maintained by Anass Derqaoui, a HIL/V&V engineer based in Gothenburg, Sweden, with eight years of experience in automotive testing (dSPACE, VeriStand, CANoe, MATLAB/Simulink, Python). The project is in part groundwork for a small private-AI services practice focused on engineering teams that require local inference in Arabic, French, or English.

If you are exploring something similar or would like to discuss private AI in an engineering context, connect on [LinkedIn](https://www.linkedin.com/in/anass-derqaoui-704a3bb6/).

---

## License

This repository is released under the MIT License. See [`LICENSE`](LICENSE) for details.

---

*Last updated: 24 April 2026*
