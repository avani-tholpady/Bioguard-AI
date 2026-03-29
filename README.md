# BioGuard AI 🛡️
### Agentic Continuous Identity Verification for Zero Trust Environments

> *"Most teams make login harder. We make the session safer."*

Built by **Team TRISPECTRA** — Thanush J, Sri Prerana, Avani Tholpady, Ayush H

---

## The Problem

Traditional MFA verifies identity **once — at login**. After that, the session is trusted indefinitely.

If an attacker steals a session token, they move freely across your entire cloud infrastructure with zero resistance. No existing tool continuously checks *who is actually using the session right now*.

---

## What BioGuard AI Does

BioGuard AI closes the **Post-Login Gap** by running three continuous verification layers simultaneously, fusing them into a **Dynamic Trust Score (0–100)** that updates every few seconds.

| Layer | Signal | Tool |
|-------|--------|------|
| **Layer 1** | BLE proximity — phone must be within 1.5m | `bleak` + `pyotp` |
| **Layer 2** | Keystroke rhythm — Hold Time + Flight Time fingerprint | `pynput` |
| **Layer 3** | Context check — IP changes, device swaps, nav anomalies | `psutil` / `usb-monitor` |

When the trust score drops, the system acts **autonomously** — no human in the loop.

| Score | Status | Action |
|-------|--------|--------|
| 80–100 | SAFE | Normal access |
| 50–79 | WARNING | Shadow Mode — attacker silently redirected to honeypot UI |
| 0–49 | CRITICAL | Hard Lock — session token revoked and quarantined |

---

## Why It's Hard to Beat

To hijack a session, an attacker needs **all three simultaneously**:
- The **physical phone** (BLE proximity within 1.5m)
- The **live rotating TOTP token** (changes every 30 seconds)
- The **owner's typing muscle memory** (impossible to replicate)

Stealing the session cookie alone is useless.

---

## Architecture

```
User Active Session
        │
        ▼
┌─────────────────────────────────────────────┐
│  TIER 1 — Always Running (<1% CPU)          │
│  pynput          bleak + pyotp    psutil     │
│  Keystroke       BLE Proximity    IP + USB   │
│  Dynamics        + TOTP           Posture    │
└──────────────────────┬──────────────────────┘
                       │ Anomaly Detected
                       ▼
┌─────────────────────────────────────────────┐
│  TIER 2 — CrewAI Agentic Brain              │
│  (On-Demand via Ollama + Llama 3)           │
│                                             │
│  Telemetry    Pattern    Proximity    Risk  │
│  Scout        Analyst    Sentry       Gov.  │
└──────────────────────┬──────────────────────┘
                       │
                       ▼
            Dynamic Trust Score (0–100)
           /           |              \
      SAFE (80+)  WARNING (50-79)  CRITICAL (0-49)
      Normal      Shadow Mode       Hard Lock
      Access      (Honeypot UI)     (Quarantine)
```

**Privacy by design** — all biometric processing runs locally via Llama 3. Zero data leaves the device.

---

## Tech Stack

### Tier 1 — Perception (Always Running)
- **pynput** — captures keystroke Hold Time + Flight Time via background listener
- **bleak** — monitors BLE RSSI for physical proximity detection
- **pyotp** — silent rotating TOTP heartbeat every 30 seconds
- **psutil / usb-monitor** — real-time IP reputation + unauthorized peripheral detection

### Tier 2 — Reasoning (On-Demand)
- **CrewAI** — orchestrates the 4-agent reasoning loop (Scout, Analyst, Sentry, Governor)
- **Ollama + Llama 3** — locally hosted LLM for private, low-latency reasoning
- **NumPy / Pandas** — Z-score anomaly detection against user baseline

### Interface
- **Streamlit** — live Security Command Center dashboard with Trust Gauge + agent monologue

---

## Setup & Installation

### Prerequisites
- Python 3.10+
- Ollama installed and running locally
- Llama 3 model pulled via Ollama
- Bluetooth-enabled device

### 1. Clone the repo
```bash
git clone https://github.com/your-username/bioguard-ai.git
cd bioguard-ai
```

### 2. Install dependencies
```bash
pip install -r requirements.txt
```

### 3. Pull Llama 3 locally
```bash
ollama pull llama3
```

### 4. Run baseline enrollment
```bash
python enroll.py
```
Type naturally for ~5 minutes. This builds your `profile.json` — the keystroke baseline.

### 5. Launch the system
```bash
# Start Tier 1 listeners
python tier1/listener.py

# In a new terminal, launch the dashboard
streamlit run dashboard.py
```

---

## How It Works — Step by Step

**Phase 1 — Enrollment**
`enroll.py` runs keyboard listeners for 5 minutes, capturing Hold Time and Flight Time per key. Simultaneously initializes BLE scanning and calibrates the TOTP shared secret. Outputs `profile.json`.

**Phase 2 — Live Monitoring**
Tier 1 listeners run silently in the background. Every signal feeds into the statistical anomaly engine, which computes a rolling Z-score against your baseline. If any signal crosses the deviation threshold, the CrewAI agents wake up.

**Phase 3 — Agentic Response**
The 4 CrewAI agents reason over the anomaly:
- **Telemetry Scout** — aggregates raw sensor events
- **Pattern Analyst** — scores typing rhythm deviation
- **Proximity Sentry** — cross-validates BLE + TOTP
- **Risk Governor** — decides the response (Normal / Shadow / Hard Lock)

---

## Key Metrics

| Metric | Value |
|--------|-------|
| Accuracy (combined biometrics) | 97–99% (internal testing, 3 users) |
| Response time | < 200ms |
| CPU impact (Tier 1) | < 1% |
| Data leaving device | Zero |

---

## Known Limitations

- **Cold-start:** ~5 minutes of typing needed before baseline is reliable. Enrollment UX is still a work in progress.
- **BLE noise:** RSSI fluctuates in crowded environments. Rolling average helps but occasional false drops still occur in open offices or cafes.
- **BLE-restricted environments:** Some enterprise policies block Bluetooth entirely. System falls back to keystroke + TOTP only — still strong, not the full triple-check.
- **Threshold tuning:** We're currently testing SD multipliers between 1.8x and 2.5x to balance false positives vs missed detections.

---

## Project Structure

```
bioguard-ai/
├── enroll.py              # Baseline enrollment script
├── dashboard.py           # Streamlit Security Command Center
├── profile.json           # Generated user baseline (gitignored)
├── tier1/
│   ├── listener.py        # pynput keystroke listener
│   ├── ble_scanner.py     # bleak BLE proximity monitor
│   └── context_watch.py   # psutil IP + device monitor
├── tier2/
│   ├── agents.py          # CrewAI agent definitions
│   ├── scoring.py         # Z-score anomaly engine
│   └── governor.py        # Risk Governor response logic
├── defense/
│   ├── shadow_mode.py     # Honeypot UI redirect
│   └── hard_lock.py       # Session revocation
└── requirements.txt
```

---

## Requirements

```
crewai
ollama
pynput
bleak
pyotp
psutil
streamlit
numpy
pandas
```

---



---

*"Stealing the session cookie used to be game over. With BioGuard AI, it's just the beginning of an impossible challenge."*
