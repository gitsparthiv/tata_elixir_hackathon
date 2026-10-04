# ❓ Doubts & Conceptual Q&A Log

This document maintains a continuous log of concepts discussed, clarifications provided, and key takeaways for quick reference and interview prep.

---

### Q1: What is an Embedded System?
**Answer:**  
An embedded system is a combination of dedicated hardware and specialized software (firmware) built inside a larger system to perform a specific dedicated task (e.g., a Wi-Fi router routing traffic, a washing machine controller, or an automotive Engine Control Unit). Unlike a PC or laptop, it is not general-purpose.

---

### Q2: What is Firmware and where does it live physically?
**Answer:**  
Firmware is the software stored in non-volatile storage (such as a **Flash Memory** chip) inside the device. It contains the bootloader, kernel/OS, device drivers, and applications that initialize and control the hardware when powered on.

---

### Q3: What kind of company is Tata Elxsi, and how is it different from TCS?
**Answer:**  
Tata Elxsi is a **product engineering and design-led technology** services company focusing on automotive (ADAS, SDV, EV), media/communications, and healthcare engineering. Unlike general enterprise IT service firms (TCS), Tata Elxsi works extensively on core software architecture, AI/ML embedded into products, and systems engineering.

---

### Q4: Is TELIPORT purely for hardware/embedded engineers?
**Answer:**  
No. While themes often relate to industries like Automotive, Avionics, or Telecom, previous winning solutions have been software-first (e.g., GenAI code generation pipelines, predictive diagnostic engines, AI vision algorithms, and simulation frameworks). CSE students with strong software/AI/systems design backgrounds are well-suited.

---

### Q5: What is the primary strategy for Round 1?
**Answer:**  
Round 1 requires a **1-slide idea submission**. Success depends on demonstrating a deep understanding of the problem statement, proposing an architecturally sound and innovative software/AI solution, and presenting clear feasibility and metrics rather than attempting to build a full physical device immediately.


---

### Q6: Do I need to use or run Linux `tc` right now for Round 1?
**Answer:**  
**No, absolutely not.** 
For Round 1, you do not need to run commands, configure Linux environments, or write code. You only need to **mention Linux `tc` (Traffic Control)** in your proposed architecture slide as the underlying enforcement tool. 

**Why we mention it:**
1. The Tata Elxsi problem statement explicitly requires: *"Use Linux traffic control or another open and inspectable enforcement mechanism."*
2. Linux `tc` is the industry-standard built-in networking tool in Linux kernels (and router OS like OpenWrt) for queue management and rate shaping.
3. In Round 1, judges only evaluate your **system design on paper/slide**. You will only ever touch or run `tc` if your slide gets shortlisted for Round 2.


---

### Q7: Will traffic classification be done by ML models?
**Answer:**  
Yes, but in a **very practical, lightweight way** (a small Decision Tree / Random Forest) combined with a **rule-based heuristic baseline**.

**Why the case study asks for this:**
The Tata Elxsi problem statement specifically states:
> *"Where AI is used, the project should compare it with a deterministic baseline and show that the model improves a measurable outcome rather than merely adding a conversational interface."*

**How we explain the ML part (Zero deep-learning jargon needed):**
1. **The Deterministic Baseline (Rules)**:
   - Simple `if-else` rules on packet size and frequency:
     - `if (packet_size < 200 bytes and frequency > 50 pkts/sec) -> VoIP / Gaming`
     - `if (packet_size == 1500 bytes continuously) -> Bulk Download`
2. **The ML Model (Lightweight Decision Tree / Random Forest)**:
   - Takes 4 simple statistical numbers (Features):
     - Average packet size
     - Packet size variance (spread)
     - Inter-arrival time (time gap between packets)
     - Flow duration & direction ratio
   - Outputs: `Class: Interactive` with `Confidence: 94%`.
3. **Why this design is strong in an interview**:
   - You don't need a heavy GPU or neural network. A Decision Tree has a tiny memory footprint ($<50\text{ KB}$), runs in $<1\text{ ms}$ on any standard router CPU, and falls back to deterministic rules if confidence is low.


---

### Q8: Why do we have an Intent API if the ML model is already classifying and prioritizing traffic?
**Answer:**  
**Exactly right! The Intent API is an explicit override & contextual intent layer.**

**Why the ML model alone isn't enough:**
1. **The ML model only knows *traffic category*, not *user importance*:**
   - The ML model sees a YouTube video stream on the TV, and another YouTube video stream on a kid's tablet. To the ML model, both are just "Video Streaming" with equal priority.
   - If total bandwidth drops, how does the system know that Dad is watching a work training video on his laptop while the TV is just background noise? It can't—unless the user expresses **Intent**.
2. **Special temporary events (The "Interview / Work Call" scenario):**
   - If you have an interview for the next 1 hour, you can trigger:
     `POST /api/intent {"device": "192.168.1.50", "priority_mode": "critical_interview", "duration_seconds": 3600}`
   - The Intent API overrides standard heuristic/ML weights, placing your laptop at the absolute highest priority tier, while ensuring household background downloads are throttled (without being killed).
3. **Problem statement alignment:**
   - Tata Elxsi explicitly mandated: *"The controller should support a temporary user intent such as prioritizing a work call while retaining fairness."*


---

### Q9: Who triggers the Intent API and how does it work?
**Answer:**  
**Yes, the user triggers it directly!**

**How the user interacts with it:**
1. **User Action**: The user opens a simple Web Dashboard / Mobile App on their phone or laptop.
2. **One-Click Intent Preset**: They click a button like:
   - 💼 *"Prioritize Work Call (1 Hour)"*
   - 🎮 *"Game Mode (2 Hours)"*
   - ⚡ *"Urgent Work Download"*
3. **Frontend $\to$ Backend Flow**:
   - The UI makes a simple HTTP POST request:
     `POST /api/intent {"device_ip": "192.168.1.15", "intent": "work_call", "duration_minutes": 60}`
   - The backend sets a timer in memory/Redis.
   - The QoS controller temporarily re-weights the traffic queues in Linux `tc` for that device.
   - When the 60-minute timer expires, the system automatically reverts back to the standard balanced ML policy without requiring any manual user cleanup.
