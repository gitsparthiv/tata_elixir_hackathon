# 📑 Tata Elxsi TELIPORT Season 4: 1-Slide Presentation Blueprint

**Case Study**: CS-03: Adaptive QoS Engine for Mixed Home Broadband Traffic  
**Project Title**: **NetPulse-QoS: Intent-Aware Adaptive Traffic Shaping & Multi-Queue Controller for Next-Gen Home Gateways**  
**Team Name**: **NexForge**  
**Target Template**: `Teliport S4_Submission Template.pptx`

---

## 🖼️ Slide Visual Layout (4-Quadrant Grid)

```
┌──────────────────────────────────────────────┬──────────────────────────────────────────────┐
│ [TOP LEFT]                                   │ [TOP RIGHT]                                  │
│ 1. Problem Statement & Operational Challenges│ 2. Core Innovations & Algorithmic Strategy   │
├──────────────────────────────────────────────┼──────────────────────────────────────────────┤
│ [BOTTOM LEFT]                                │ [BOTTOM RIGHT]                               │
│ 3. Proposed End-to-End System Architecture   │ 4. Measurable Outcomes, Benchmarks & Scope   │
└──────────────────────────────────────────────┴──────────────────────────────────────────────┘
```

---

## 📝 Exact Content for Each Section of the Slide

### 🟦 Quadrant 1: Problem Statement & Key Challenges (Top Left)
- **The Bufferbloat Crisis**: Concurrent residential traffic (Zoom calls, gaming, 4K video, 50 GB bulk downloads) causes massive bufferbloat ($>300\text{ ms}$ latency spikes & jitter), severely degrading interactive user experience.
- **Dynamic WAN Fluctuations**: ISP bandwidth drops (e.g., $100\text{ Mbps} \to 20\text{ Mbps}$) cause unmanaged router queues to overflow.
- **Privacy & Encryption Barrier**: Modern traffic uses TLS 1.3/QUIC; Deep Packet Inspection (DPI) is impossible and violates user privacy.
- **Static QoS Limitations**: Traditional static priority starvation occurs when low-priority bulk downloads completely stall.

---

### 🟩 Quadrant 2: Core Innovations & Algorithmic Strategy (Top Right)
- **Zero-DPI Non-Invasive Flow Classifier**:
  - Extracts 4 metadata features (Packet size variance, Inter-Arrival Time, flow duration, burstiness) without payload decryption.
  - **Lightweight ML + Heuristic Fallback**: Fast Decision Tree ($<50\text{ KB}$ footprint, $<1\text{ ms}$ inference) with automated fallback to deterministic rules if confidence $<70\%$.
- **Intent-Aware REST API & State Engine**:
  - Exposes `POST /api/intent` for one-click temporary prioritization (e.g., *"Prioritize Work Call on Laptop for 60 Mins"*).
  - Automatically reverts to balanced policy upon timer expiration.
- **Dynamic WAN Capacity Estimator**:
  - Tracks Round-Trip Time (RTT) gradients to detect ISP congestion before buffers choke, automatically updating root rate limits.
- **Starvation-Free Fair Queuing**:
  - Employs Hierarchical Token Bucket (HTB) + FQ-CoDel with a guaranteed $10\%$ bandwidth slice for bulk traffic.

---

### 🟨 Quadrant 3: Proposed System Architecture (Bottom Left)

*Include this clean text/flow diagram on the slide:*

```
 [ INGRESS / EGRESS TRAFFIC ]
              │
              ▼
 ┌────────────────────────────────────────────────────────┐
 │ 1. Telemetry & Metadata Classifier (Zero-DPI / Python) │
 │    • Features: Packet Size, Inter-Arrival Time (IAT)   │
 │    • ML Decision Tree + Deterministic Heuristics       │
 └───────────────────────────┬────────────────────────────┘
                             │ Flow Category + Confidence
                             ▼
 ┌────────────────────────────────────────────────────────┐
 │ 2. Intent-Aware Controller & State Store (Redis/Flask) │
 │    • User Intent API (Temporary Work Call / Game Mode) │
 │    • Dynamic WAN Speed Sensor (RTT Congestion Monitor) │
 └───────────────────────────┬────────────────────────────┘
                             │ Dynamic Class Rates
                             ▼
 ┌────────────────────────────────────────────────────────┐
 │ 3. Linux Kernel Enforcement Engine (tc + HTB/FQ-CoDel) │
 │    • ⚡ Fast Lane (Gaming/VoIP) → Delay-bounded        │
 │    • 🎬 Medium Lane (Streaming) → Burst-tolerant       │
 │    • 📦 Bulk Lane (Downloads)  → Guaranteed min 10%    │
 └────────────────────────────────────────────────────────┘
```

---

### 🟪 Quadrant 4: Measurable Outcomes & Prototype Scope (Bottom Right)

#### 📊 Baseline vs. Adaptive QoS Benchmark Matrix:

| Key Performance Indicator | Unmanaged / Default FIFO | NetPulse-QoS Engine | Improvement |
| :--- | :--- | :--- | :--- |
| **Interactive Latency under Bulk Load** | $> 280\text{ ms}$ (Lagging/Choppy) | **$< 20\text{ ms}$ (Real-time)** | **$\approx 93\%$ Latency Reduction** |
| **VoIP / WebRTC Audio Jitter** | $45\text{–}90\text{ ms}$ (Audio drops) | **$< 4\text{ ms}$ (Crystal clear)** | **Zero packet starvation** |
| **WAN Drop Adaptation Time ($100\to 20\text{M}$)** | Fails / Dropped packets | **$< 350\text{ ms}$ automatic rate shift** | **Self-healing link control** |
| **Bulk Download Progress** | Chokes call or stalls | **Continuous steady throughput** | **Guaranteed fairness** |
| **Gateway Resource Overhead** | N/A | **$< 4\%$ CPU, $< 25\text{ MB}$ RAM** | **Edge-gateway ready** |

#### 🛠️ Technology Stack & Scope:
- **Core Engine**: Python (Controller/Daemon) + Linux `tc` (Traffic Control / HTB / FQ-CoDel).
- **APIs & State**: Flask REST API + Redis In-Memory State Store.
- **Simulation Harness**: Linux Network Namespaces (`netem`) + `iperf3` + `ping`.
- **Production Path**: Deployable as an OpenWrt package on home broadband routers.
