# Adaptive Traffic Steering in O-RAN via Hybrid Rule-Based and Reinforcement Learning

> **Companion repository for the paper:**
> *"Hybrid Rule-Based and Reinforcement Learning Traffic Steering in Open RAN"*
> Utkarsh Sharma, Yuchen Liu — Computer Networks (Elsevier), 2025 (under review)

---

## Overview

Open Radio Access Networks (O-RAN) disaggregate the RAN stack and expose programmable control interfaces (xApps, rApps) that enable data-driven optimization. **Traffic Steering (TS)** — deciding which base station each UE connects to — is a central control problem in O-RAN.

This repository accompanies a journal paper that proposes a **hybrid traffic-steering controller** combining:

- A **rule-based policy** — low-latency, interpretable, and safe-by-construction
- A **Dueling Deep Q-Network (DQN)** — trained to maximize long-run network utility
- A **hybrid controller** with hysteresis-based switching — selects the best policy per network condition and prevents oscillation

The system is evaluated end-to-end in **ns-3** using a realistic heterogeneous multi-cell, multi-slice O-RAN scenario.

---

## Key Contributions

| # | Contribution |
|---|---|
| C1 | Enhanced rule-based TS policy with three-phase capacity management (reserve → RRC → commit/rollback) |
| C2 | Dueling DQN agent with per-slice preference shaping, fairness incentives, and handover penalties |
| C3 | Hybrid controller with hysteresis band (ρ_L = 50%, ρ_H = 70%) and health-trigger override |
| C4 | Zero-shot transfer from Python PoC to ns-3 via ONNX Runtime C++ inference (~180 μs, ~250 KB) |
| C5 | Comprehensive ns-3 O-RAN simulation covering 8 load scenarios (50–1500 UEs), 3 slices, 3 BS tiers |

---

## Architecture

```
┌──────────────────────────────────────────────┐
│              Non-RT RIC (offline)            │
│   ┌──────────────────────────────────────┐   │
│   │      RL Training Pipeline           │   │
│   │  State → Dueling DQN → Q(s,a)       │   │
│   │  Export trained model → ONNX        │   │
│   └──────────────────────────────────────┘   │
└───────────────────┬──────────────────────────┘
                    │ ONNX model
┌───────────────────▼──────────────────────────┐
│              Near-RT RIC (online)            │
│  ┌───────────────────────────────────────┐   │
│  │         Hybrid Controller             │   │
│  │  load < ρ_L  → Rule-Based Policy     │   │
│  │  load > ρ_H  → RL Policy (ONNX)      │   │
│  │  Health triggers override at any load │   │
│  └───────────────────────────────────────┘   │
│                    │ E2 interface             │
└───────────────────┬──────────────────────────┘
                    │
┌───────────────────▼──────────────────────────┐
│              ns-3 O-RAN Simulator            │
│   Macro / Pico / Anchored BS tiers           │
│   eMBB · URLLC · mMTC slices                 │
│   8 load scenarios: 50 – 1500 UEs            │
└──────────────────────────────────────────────┘
```

---

## System Model

**Network topology:** Three-tier heterogeneous network — Macro, Pico, and Anchored base stations — simulated in ns-3.45.

**Traffic slices:**

| Slice | Mix | Demand profile |
|-------|-----|----------------|
| eMBB  | majority | High throughput, burst-tolerant |
| URLLC | moderate | Consistent, low-latency |
| mMTC  | minority | Sensor/IoT — very small per-UE footprint |

**Evaluation scenarios:** S1–S8, spanning 50 to 1500 UEs.

**Metrics:**
- Unmanaged UEs (capacity violations)
- First-preference satisfaction rate
- Jain's Fairness Index
- Handover count

---

## Repository Structure

```
oran-adaptive-traffic-steering/
├── README.md
│
├── rl_agent/               # Dueling DQN implementation (Python / PyTorch)
│   ├── model.py            #   Network architecture (shared trunk + value/advantage heads)
│   ├── train.py            #   Training loop, replay buffer, ε-greedy exploration
│   └── export_onnx.py      #   Export trained weights to ONNX for ns-3 inference
│
├── rule_based/             # Rule-based traffic steering policy
│   └── policy.py           #   Three-phase capacity management logic
│
├── hybrid_controller/      # Hybrid switching logic
│   └── controller.py       #   Hysteresis + health-trigger override
│
├── ns3_simulation/         # ns-3.45 simulation scripts and helpers
│   ├── scenario.cc         #   Main simulation entry point
│   ├── hybrid.py           #   Batch runner for hybrid policy evaluation
│   └── eval_results.csv    #   Raw simulation results
│
└── plots/                  # Result visualisation scripts
    └── plot_results.py
```

> **Note:** Code files are being cleaned and documented; full release will follow paper acceptance.
> For questions about a specific component, open an issue or contact the authors directly.

---

## Results Summary

Evaluated across 8 load scenarios in ns-3:

| Scenario | UEs  | RL unmanaged | Rule unmanaged | RL Jain's Fairness |
|----------|------|-------------|---------------|-------------------|
| S1       | 50   | 0           | 3             | 0.136             |
| S2       | 100  | 2           | 8             | 0.326             |
| S3       | 250  | 6           | 56            | 0.820             |
| S4       | 350  | 9           | 114           | 0.871             |
| S5       | 450  | 41          | 139           | 0.914             |
| S6       | 600  | 34          | 239           | 0.907             |
| S7       | 1000 | 243         | 449           | 0.924             |
| S8       | 1500 | 731         | 797           | 0.842             |

The RL agent substantially reduces unmanaged UEs under high load. The hybrid controller captures the best of both policies while avoiding oscillation through hysteresis.

---

## Citation

If you use this work, please cite:

```bibtex
@article{sharma2025hybrid,
  title   = {Hybrid Rule-Based and Reinforcement Learning Traffic Steering in Open RAN},
  author  = {Sharma, Utkarsh and Liu, Yuchen},
  journal = {Computer Networks},
  year    = {2025},
  note    = {Under review}
}
```

---

## Contact

**Utkarsh Sharma** · PhD Student, NC State University (CSC)
usharma3@ncsu.edu · [GitHub](https://github.com/utkarsh-us-sharmaa)

**Yuchen Liu** · Assistant Professor, NC State University (CSC)

---

*This research was conducted as part of the NC State University Computer Science PhD program.*
