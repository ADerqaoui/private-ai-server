# Hardware Bill of Materials

*Full hardware inventory for Server 1, across all four phases. All prices are in Swedish kronor (SEK) at actual or expected retail, as of April 2026.*

---

## Phase 1 — Core build (complete)

The hardware below is installed and operational. The total represents real money spent, not estimates.

| Component | Specification | Source | Price |
|---|---|---|---|
| CPU | AMD Ryzen 9 7900 (12 cores / 24 threads, 65 W) | Elgiganten | 3 179 kr |
| Motherboard | ASUS B650E MAX Gaming WiFi (AM5, DDR5, PCIe 5.0) | Elgiganten | 1 049 kr |
| RAM | Crucial Pro OC 32 GB (2×16 GB) DDR5-6000 CL36 EXPO | Elgiganten | 3 790 kr |
| Storage | Kingston Fury Renegade 2 TB NVMe Gen 4 | Inet.se (open-box) | 3 999 kr |
| PSU | ASUS TUF Gaming 750 W Gold | Elgiganten | 949 kr |
| CPU cooler | AMD Wraith Stealth (included with CPU) | — | 0 kr |
| Thermal paste | Arctic MX-4 | Elgiganten | 99 kr |
| GPU | ASUS RTX 3080 10 GB TUF Gaming (used) | SweClockers auction | 3 009 kr |
| **Total Phase 1** | | | **16 074 kr** |

**Notes:**
- The GPU was acquired via auction on SweClockers at minimum bid, delivered via DHL. Price includes shipping.
- The SSD was purchased as an open-box item from Inet.se.
- The CPU cooler is the stock AMD Wraith Stealth included in the Ryzen 9 7900 box. It is adequate for a 65 W TDP chip under typical inference workloads but will likely be replaced when the GPU is upgraded and total system heat increases.
- Thermal paste was applied during assembly; a small amount remains for future re-seats.

---

## Phase 2 — RAM expansion

| Component | Specification | Planned source | Expected price |
|---|---|---|---|
| RAM (second kit) | Crucial Pro OC 32 GB (2×16 GB) DDR5-6000 CL36 EXPO — **identical to current kit** (part CP2K16G60C36U5B or exact equivalent) | Inet.se or Webhallen | 3 500–4 200 kr |

**Target timing:** Month 3–4 from project start. Purchase when budget allows.

**Configuration after install:**
- 4 DIMMs total: slots A1 + B1 (new) and A2 + B2 (existing)
- 64 GB total capacity
- Expected stable speed: DDR5-6000 (EXPO profile retained)

**Risks and constraints:**
- The new kit must be the **exact same model** as the installed kit. Mixing RAM brands, speeds, or timings on AM5 commonly causes instability or forces the memory controller to downclock to the lowest common denominator.
- Running 4 DIMMs on AM5 sometimes reduces maximum stable frequency even with matched kits. If DDR5-6000 does not hold with four DIMMs populated, falling back to DDR5-5600 is acceptable — the bandwidth loss is small relative to the benefit of 64 GB capacity.
- Elgiganten pricing on this kit has been inconsistent. Inet.se and Webhallen should be checked first before purchase.
- **2026 DDR5 market:** prices are rising, not falling. Budget accordingly. Waiting for a price drop is not a useful strategy.

---

## Phase 3 — GPU upgrade

| Component | Specification | Planned source | Expected price |
|---|---|---|---|
| GPU (replacement) | NVIDIA RTX 3090 24 GB — used, any AIB brand | SweClockers, Tradera, Blocket | 7 000–9 000 kr |
| PSU upgrade | 850 W 80+ Gold modular | Webhallen, Inet.se, Elgiganten | 800–1 000 kr |
| GPU sell-back | RTX 3080 10 GB TUF Gaming (current card) | SweClockers | 2 500–2 800 kr *recovered* |
| **Net Phase 3 cost** | | | **5 300–7 200 kr** |

**Target timing:** Month 4–6.

**Why the 3090:**
- 24 GB VRAM enables comfortable inference on 13 B and 20 B class models, and usable (quantised) inference on 30 B class models. The current 3080 is limited to approximately 7–9 B class models fully in VRAM.
- Memory bandwidth on the 3090 (936 GB/s on its 384-bit GDDR6X bus) is substantially higher than the 3080, which improves token generation speed even on models that both cards can fit.
- The used market for 3090s has stabilised; supply is reliable.

**Why not a newer GPU (4090, 5090):**
- 4090 prices remain in the 15 000–20 000 kr range for the used market. The jump in capability does not justify the price increase for single-user inference on models up to 30 B.
- 5090 prices are significantly higher still, and availability is limited. Not appropriate for this phase.
- A 70 B class model target would require either a 5090 or a multi-GPU workstation — both of which belong to the Server 2 scope, not Server 1.

**Why a PSU upgrade is required:**
- The installed 750 W PSU is at the edge of stable operation for a Ryzen 9 7900 plus an RTX 3090 under sustained inference load. 3090 cards draw up to 350 W at peak with transient spikes well above that.
- 850 W provides headroom for transient spikes and quiet operation. 80+ Gold efficiency is the sensible floor; Platinum is unnecessary at this tier.
- The PSU should be purchased and installed before the GPU.

**Sell-back of the RTX 3080:**
- SweClockers is the most reliable channel for used GPU sales in Sweden. Expected recovery is 2 500–2 800 kr depending on condition and market timing at the time of sale.
- The card will have approximately 6–9 months of use at the time of sale, all under moderate inference loads (not gaming). This is a favourable usage profile for resale.

---

## Phase 4 — Infrastructure completion

| Component | Specification | Planned source | Expected price |
|---|---|---|---|
| Case | Corsair 4000D Airflow (ATX mid-tower) | Webhallen, Inet.se | ~1 050 kr |
| NAS | Synology DS224+ or TrueNAS on mini-PC | Dustin, Inet.se, secondary market | 4 000–6 000 kr |
| NAS drives (initial) | 2× 4 TB HDD (Seagate IronWolf or WD Red Plus) | Inet.se, Webhallen | ~2 000 kr |
| Auxiliary cluster | Platform TBD (Pi 5, mini-PC, Jetson, or similar) — 1–3 nodes | Varies by platform | 3 000–7 000 kr |
| Gigabit switch | 8-port managed or unmanaged | Inet.se, Webhallen | 300–700 kr |
| Cabling | Cat 6 patch cables (short runs) | Various | 100–200 kr |
| **Phase 4 range** | | | **10 450–16 950 kr** |

**Target timing:** Month 10–14.

**Role of each Phase 4 component:**

- **Case.** Moves the build off the current open-bench configuration. The Corsair 4000D Airflow is chosen for good front-to-rear airflow, which matters more once a 3090 is installed. Other candidates (Fractal Design North, Lian Li Lancool) are equally valid; the 4000D is selected primarily for availability and price.
- **NAS.** Holds model files, ingested documents, backups, and any outputs that should persist outside the primary server. Makes the primary server stateless in principle — a catastrophic failure on the primary server does not mean losing the models or accumulated documents.
- **Auxiliary cluster.** Runs gateway services (Nginx reverse proxy, WireGuard VPN) and monitoring (Prometheus, Grafana, Uptime Kuma) on low-power hardware that can stay on 24/7 without affecting the primary server's power budget. The cluster also provides a 24/7 always-reachable entry point for the system even when the primary server is being maintained or rebooted.
- **Switch and cabling.** Gigabit ethernet for all nodes. WiFi is explicitly not used for any server-to-server traffic because latency and reliability matter for the NAS-to-inference-server link.

**Platform decision for the cluster is deliberately deferred:**

The following are all acceptable platforms for the cluster role. Final selection will be made closer to purchase, based on pricing and availability at that time:

- **Raspberry Pi 5** (8 or 16 GB variants) — proven, well-documented, ARM architecture. Pricing in 2026 is volatile.
- **Orange Pi 5 Plus** (16 GB) — more RAM than Pi 5 8 GB, includes NPU. ARM architecture.
- **NVIDIA Jetson Orin Nano Super** — CUDA-capable edge AI. Useful if the cluster is expected to do any lightweight local inference.
- **Mini PCs (Intel N100 / N150, Beelink, Minisforum)** — x86 architecture, often cheaper than ARM single-board computers in 2026, no driver surprises.
- **Small used office PCs** — cheapest option. Higher power draw and larger physical footprint are the trade-offs.

---

## Complete Server 1 budget, all phases

| Phase | Range |
|---|---|
| Phase 1 (complete) | 16 074 kr |
| Phase 2 (RAM) | 3 500–4 200 kr |
| Phase 3 (GPU + PSU, net after 3080 sale) | 5 300–7 200 kr |
| Phase 4 (case, NAS, cluster, network) | 10 450–16 950 kr |
| **Full Server 1 total** | **35 324–44 424 kr** |

This total is the amount that cumulative business profit must exceed before Server 2 becomes eligible for planning. That is a deliberate discipline, not a hard deadline — the gate is on profit, not time.

---

## Separately: Server 2 (future, conditional)

A second server is planned but **not approved and not funded**. It is documented in the project brief as a conditional future build, gated on business profitability. Its hardware — workstation-class platform (Threadripper or EPYC), 128 GB RAM, dual GPU or high-VRAM single card, 1200+ W PSU, larger case — is not tracked in this bill of materials.

Consumer AM5 hardware cannot host a proper dual-GPU AI configuration because the motherboard has only one full-bandwidth PCIe slot. Server 2 exists as a separate build rather than a Server 1 upgrade for this reason, among others. Full rationale is in the project brief.

---

*Last updated: 24 April 2026*
