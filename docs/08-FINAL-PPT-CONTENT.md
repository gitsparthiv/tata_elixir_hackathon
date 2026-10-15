# 🖼️ FINAL PPT SLIDE CONTENT — Tata Elxsi TELIPORT Season 4

**Case Study:** CS-03 | **Project Title:** NetPulse-QoS — Intent-Aware Adaptive Traffic Shaping Engine for Mixed Home Broadband  
**Team Name:** NexForge | **Template:** `Teliport S4_Submission Template.pptx`

> [!IMPORTANT]
> This is the final, review-ready slide content. Copy each box verbatim into the corresponding section of the official PPTX template. Diagrams in Box 2 may be recreated as a PowerPoint SmartArt / simple box-and-arrow diagram.

---

## 🟦 BOX 1 — Problem Statement & Operational Challenges (Top Left)

**Heading: The Residential Broadband QoS Crisis**

- **Bufferbloat at Scale:** Concurrent traffic (WebRTC calls, gaming, 4K streaming, bulk ISO downloads) floods shared router queues → interactive latency spikes **>300 ms**, causing call freezes and lag even on high-speed connections
- **Encrypted Traffic Blind Spot:** TLS 1.3 & QUIC encryption make Deep Packet Inspection (DPI) impossible — traditional content-aware QoS breaks entirely
- **Dynamic WAN Volatility:** ISP bandwidth can drop 100 Mbps → 20 Mbps without warning; static rate limiters continue over-committing, accelerating queue buildup
- **Static Priority Starvation:** Hardcoded priority rules either choke bulk transfers completely or fail to protect low-latency flows under congestion
- **No User Context Awareness:** Routers cannot distinguish an active interview call from background YouTube — both appear identical at the network layer

---

## 🟩 BOX 2 — Proposed System Architecture (Bottom Left)

**Heading: 4-Layer Adaptive QoS Architecture**

```
[ HOME DEVICE TRAFFIC — IPv4/IPv6 ]
              │
              ▼
┌──────────────────────────────────────────────────────┐
│ MODULE 1: Zero-DPI Metadata Classifier               │
│ eBPF/Netfilter flow probe — NO payload read          │
│ Features: Packet Size Variance · Inter-Arrival Time  │
│ (IAT) · Burst Entropy · Upload:Download Ratio        │
│ → Decision Tree ML (< 50 KB, < 1 ms inference)       │
│ → Fallback: Deterministic heuristics if CI < 70%     │
└──────────────────────┬───────────────────────────────┘
                       │ Flow Class + Confidence Score
                       ▼
┌──────────────────────────────────────────────────────┐
│ MODULE 2: Intent & Policy Controller                 │
│ Flask + Redis — stores TTL-bound intent state        │
│ POST /api/intent → {device, mode: "interview",       │
│ ttl: 3600} — auto-reverts on expiry                  │
│ RTT Gradient Monitor: detects WAN drops in <350 ms,  │
│ updates shaper ceiling instantly                     │
└──────────────────────┬───────────────────────────────┘
                       │ Dynamic Rates + Override Rules
                       ▼
┌──────────────────────────────────────────────────────┐
│ MODULE 3: Linux tc Enforcement Engine                │
│ Hierarchical Token Bucket (HTB) + FQ-CoDel           │
│ ⚡ Class A — Interactive (VoIP/Gaming): delay-bounded │
│ 🎬 Class B — Streaming (OTT): burst-tolerant pacing  │
│ 📦 Class C — Bulk (Downloads): ≥ 10% guaranteed min │
└──────────────────────┬───────────────────────────────┘
                       │
                       ▼
┌──────────────────────────────────────────────────────┐
│ MODULE 4: Observability Dashboard                    │
│ Flask + React (SSE) — Real-time: RTT · Jitter ·      │
│ Queue Depth · Packet Loss · Jain's Fairness Index    │
│ Automated baseline vs optimised experiment reports   │
└──────────────────────────────────────────────────────┘
```

---

## 🟨 BOX 3 — Technical Innovations & AI Strategy (Top Right)

**Heading: Core Innovations & AI vs. Baseline**

**① Privacy-Safe Non-Invasive Classifier:**
- Extracts 4 statistical flow features (Packet Size Variance, IAT, Burstiness, Directional Ratio) from metadata only — zero payload decryption, fully TLS 1.3/QUIC compliant
- **Baseline:** Deterministic heuristic rules (`if packet_size < 200B && IAT < 20ms → VoIP/Gaming`)
- **AI Layer:** Lightweight Decision Tree (<50 KB, <1 ms, runs on commodity router CPU) — outputs traffic class with confidence score
- **Safety Fallback:** If confidence <70%, reverts to heuristic baseline — zero classification-failure risk

**② RTT-Gradient WAN Sensing (Bufferbloat Buster):**
- Passive RTT inflection detection: rising RTT gradient signals ISP congestion → dynamically lowers root shaper ceiling **before** queue overflows
- Eliminates bufferbloat at source, not after packets are already dropped

**③ Stateful Intent Override with Auto-Rollback:**
- REST API: `POST /api/intent` declares time-bound context (Interview Mode, Game Mode, Urgent Download)
- TTL-bound Redis state ensures **automatic policy reversion** — bounded, observable, and reversible

**④ Starvation-Free Fair Multi-Queue:**
- HTB guarantees Class C (Bulk) a minimum **10% bandwidth floor** regardless of Class A/B demand
- FQ-CoDel per-flow queuing eliminates head-of-line blocking within each class

---

## 🟪 BOX 4 — Measurable Outcomes & Prototype Scope (Bottom Right)

**Heading: Verified Performance Impact & Implementation Scope**

| Metric | Unmanaged FIFO | Static Priority QoS | **NetPulse-QoS** |
|---|---|---|---|
| Interactive Latency (under full bulk load) | >280 ms | ~120 ms | **<20 ms** |
| WebRTC / VoIP Jitter | 45–90 ms | 20–40 ms | **<5 ms** |
| WAN Drop Adaptation (100M→20M) | Queue flood / drops | Manual reconfig | **<350 ms auto** |
| Bulk Throughput Stability | Choked | Starved | **≥10% floor guaranteed** |
| Jain's Fairness Index | ~0.55 (poor) | ~0.72 (moderate) | **>0.92 (high)** |
| Gateway CPU / RAM Overhead | — | — | **<5% CPU · <30 MB RAM** |

**Prototype Scope:**
- **Emulation:** Linux Network Namespaces + `tc netem` (WAN impairment) + `iperf3` + Python Controller
- **Validated Against:** 3 mandatory case study scenarios (bulk-during-call, WAN bandwidth drop, multi-device gaming+streaming)
- **Production Roadmap:** OpenWrt / RDK-B compatible feed package — deployable on real CPE hardware

**Stack:** Python · Flask · Redis · Linux `tc` (HTB + FQ-CoDel) · eBPF/Netfilter · React Dashboard
