# Research Area 4: Phased Commercialization & Learning Architecture

**Status:** Research Complete  
**Date:** March 20, 2026  
**Objective:** Define practical phases from simulation to hardware, including learning loops at each stage

---

## Executive Summary

This research defines a **phased commercialization roadmap** that de-risks the analog KV cache accelerator through progressive validation. Each phase builds on the previous, with clear go/no-go decision gates and learning mechanisms.

**Core Insight:** The analog memory domain requires **hardware/software co-evolution** — we cannot validate analog architectures in pure simulation. The phases bridge this gap through increasingly realistic testing environments.

**Total Timeline:** ~7 years from concept to product  
**Total Investment:** $100M+ (aligned with business case model)  

---

## Phase Structure Overview

| Phase | Name | Duration | Cost | Key Output | Go/No-Go |
|-------|------|----------|------|------------|----------|
| **0** | Simulation & Modeling | 6 months | $500K | Validated Spice + ML co-simulation | Architecture viability |
| **1** | FPGA Emulation | 12 months | $3M | Behavioral prototype on FPGA | Real-world constraints |
| **2** | Test Chip (MPW) | 18 months | $8M | 64-256 analog cell test vehicle | Device characterization |
| **3** | Proof-of-Concept Chip | 24 months | $25M | Full KV cache subsystem (1-4MB analog) | End-to-end validation |
| **4** | Production Silicon | 36 months | $60M+ | Commercial product (256MB-1GB analog) | Market validation |

---

## Phase 0: Simulation & Modeling (Months 1-6)

### Objective
Validate the hybrid architecture through high-fidelity simulation before any hardware commitment.

### Technical Approach

**Spice-Level Analog Modeling:**

**Tools (Validated from Research):**
| Tool | Vendor | Purpose | Cost |
|------|--------|---------|------|
| Spectre XPS | Cadence | SPICE simulation, Monte Carlo | $50K+/license |
| PrimeSim | Synopsys | GPU-accelerated SPICE (5× speedup) | $75K+/license |
| HSPICE | Synopsys | Foundry-certified signoff | $100K+/license |
| ngSPICE | Open source | Prototyping, model development | Free |

**Device Models Required:**
- **RRAM:** Knowm model, Stanford RRAM model, or foundry-provided
- **PCM:** IBM PCM model or foundry-specific
- **Variability:** Monte Carlo decks for process variation
- **Noise:** Time-dependent noise models (1/f, thermal)
- **Aging:** Drift models for retention characterization

**ML Framework Integration:**
```python
# Concept: PyTorch Analog Memory Layer
class AnalogKVCache(nn.Module):
    def __init__(self, noise_profile, drift_model):
        self.noise_profile = noise_profile  # From Spice characterization
        self.drift_model = drift_model      # Time-dependent drift
    
    def forward(self, q, k, v):
        # Simulate analog noise during training
        k_noisy = k + self.noise_profile.sample(k.shape)
        v_noisy = v + self.noise_profile.sample(v.shape)
        
        # Simulate drift over inference duration
        k_drifted = self.drift_model.apply(k_noisy, time=self.inference_time)
        v_drifted = self.drift_model.apply(v_noisy, time=self.inference_time)
        
        return attention(q, k_drifted, v_drifted)
```

**Co-Simulation Platform:**
```
┌─────────────────────────────────────────────────────────────┐
│                 HYBRID SIMULATION STACK                     │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────────┐     ┌──────────────┐     ┌──────────┐    │
│  │   PyTorch    │◄───►│   Analog     │◄───►│  Spice   │    │
│  │   Model      │     │   Wrapper    │     │  Netlist │    │
│  └──────────────┘     └──────────────┘     └──────────┘    │
│         │                    │                    │         │
│         └────────────────────┼────────────────────┘         │
│                              ▼                              │
│                    ┌──────────────────┐                     │
│                    │  Noise Injection │                     │
│                    │  (matched to HW) │                     │
│                    └──────────────────┘                     │
└─────────────────────────────────────────────────────────────┘
```

### Key Research Questions
1. What noise levels can be tolerated before accuracy degradation?
2. How does analog drift affect KV cache over inference duration?
3. What's the optimal digital/analog split ratio?
4. Which precision scaling tiering provides best ROI?

### Deliverables
- Validated analog device models
- ML framework integration library
- Simulation infrastructure for Phases 1-3
- Go/No-Go recommendation for Phase 1

### Learning Loop
- **Metric:** Model accuracy vs. analog noise level
- **Feedback:** Tune hybrid architecture parameters
- **Decision:** Proceed to FPGA if accuracy >95% at target noise levels

---

## Phase 1: FPGA Emulation (Months 7-18)

### Objective
Validate the hybrid architecture with real-world timing and noise injection on FPGA.

### Technical Approach

**FPGA Platform Selection:**

| Platform | Vendor | Key Features | Price |
|----------|--------|--------------|-------|
| **Versal AI Core VC1902** | AMD/Xilinx | 128 TOPS INT8, PCIe Gen5, CXL 2.0 | $8K-12K |
| **Agilex I-Series** | Intel | PCIe Gen5, CXL 1.1, ~50% lower power | $10K-15K |
| **Versal Gen 2** | AMD/Xilinx | PCIe Gen6, CXL 3.1, AI Engines (newest) | TBD |

**Research Finding:** Intel Agilex has ~2× performance/watt advantage over Versal for pure FPGA workloads; Versal has better NoC architecture for heterogeneous compute. For our use case (KV cache controller + external analog), Agilex's power efficiency and CXL support make it the preferred choice.

**FPGA-Based Emulation Architecture:**
```
┌─────────────────────────────────────────────────────────────┐
│                   FPGA EMULATION SYSTEM                     │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────┐   │
│  │              INTEL AGILEX FPGA                      │   │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────┐  │   │
│  │  │  KV Cache    │  │   CXL 2.0    │  │  Noise   │  │   │
│  │  │  Controller  │  │  Controller  │  │  Inject  │  │   │
│  │  │  (RTL)       │  │  (Hard IP)   │  │  (Digital)│  │   │
│  │  └──────────────┘  └──────────────┘  └──────────┘  │   │
│  └─────────────────────────────────────────────────────┘   │
│                              │                              │
│                         ┌────┴────┐                         │
│                         │         │                         │
│  ┌──────────────────────▼──┐   ┌──▼──────────────────────┐  │
│  │   ANALOG FRONT-END      │   │      HOST SYSTEM        │  │
│  │  ┌──────────────────┐   │   │  ┌──────────────────┐   │  │
│  │  │  Commercial RRAM │   │   │  │  GPU + PyTorch   │   │  │
│  │  │  or PCM chips    │   │   │  │  Real LLM models │   │  │
│  │  │  (for char)      │   │   │  └──────────────────┘   │  │
│  │  └──────────────────┘   │   │                         │  │
│  │  Programmable noise     │   │  End-to-end inference   │  │
│  │  circuits for injection │   │  with analog KV cache   │  │
│  └─────────────────────────┘   └─────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

**Why FPGA Before Silicon:**
| Risk | Mitigation |
|------|------------|
| Algorithm errors | Fix in software, no silicon respin |
| Interface timing | Validate with real PCIe/CXL traffic |
| Integration issues | Test system-level behavior |
| Performance modeling | Measure actual vs. simulated |

### Deliverables
- FPGA-based emulator (Intel Agilex I-Series)
- Analog characterization board
- Validated controller RTL
- Performance/power models updated with real data

### Learning Loop
- **Metric:** End-to-end latency vs. simulation predictions
- **Feedback:** Refine analog timing models
- **Decision:** Proceed to test chip if <10% deviation from simulation

---

## Phase 2: Test Chip (MPW - Multi-Project Wafer) (Months 19-36)

### Objective
Characterize actual analog devices in our target technology node.

### MPW Cost Research (Validated)

**Europractice MPW Pricing (2025):**
| Foundry | Node | MPW Cost (per 20 dies) | Notes |
|---------|------|------------------------|-------|
| GlobalFoundries | 22nm FDSOI | ~$50K-100K | Good for analog/mixed-signal |
| GlobalFoundries | 12nm LP+ | ~$75K-150K | Leading edge available |
| TSMC | 22nm ULL | ~$100K-200K | Premium, best analog support |
| TSMC | 16nm FinFET | ~$200K-400K | Highest density |
| GF/TSMC | 28nm | ~$30K-60K | Mature, cost-effective |

**Research Finding:** 28nm/22nm is the sweet spot for analog NVM integration — mature enough for stable analog devices, advanced enough for meaningful density. GlobalFoundries 22nm FDSOI offers excellent analog characteristics and is ~30% cheaper than TSMC equivalent.

**Foundry NVM Capabilities (Validated from Research):**

| Foundry | RRAM | MRAM | PCM | Notes |
|---------|------|------|-----|-------|
| **TSMC** | 40nm, 28nm, 22nm | 22nm, 16nm (2025) | — | Most advanced RRAM, MRAM roadmap |
| **Samsung** | — | 14nm, 8nm (2026), 5nm (2027) | — | Leading MRAM, no RRAM disclosed |
| **GlobalFoundries** | 22nm (CBRAM) | 22nm FDSOI, 12nm | — | Acquired Adesto CBRAM, GF-oriented |
| **STMicro** | — | — | 28nm, 18nm FDSOI | Leading PCM for automotive |

**Recommendation:** Start with **TSMC 22nm RRAM** (proven in Infineon AURIX TC4) or **GlobalFoundries 22nm FDSOI** (cost advantage). Both have production NVM solutions.

### Test Vehicle Design

**MPW Test Chip Specifications:**
| Parameter | Value |
|-----------|-------|
| Analog cells | 64-256 cells |
| Array configurations | 8×8, 16×16, crossbar |
| Device types | 2-3 RRAM variants |
| Digital control | SPI interface |
| On-chip sensors | Temperature, voltage |
| Die size | <10 mm² (for MPW cost) |
| Package | QFP or bare die for probing |

**Characterization Focus:**
| Parameter | Method | Target |
|-----------|--------|--------|
| Write endurance | Cycling test | 10⁶+ cycles |
| Retention | High-temperature bake | 10 years @ 85°C |
| Noise distribution | Statistical sampling | <5% variation |
| Drift over time | Accelerated aging | <1% per decade |
| Crossbar sneak paths | Array-level testing | <10% degradation |

### Deliverables
- Fabricated test chip via MPW
- Comprehensive device characterization report
- Calibrated Spice models from silicon data
- RRAM vs. PCM selection recommendation

### Learning Loop
- **Metric:** Device parameters vs. simulation assumptions
- **Feedback:** Update all models, potentially revise architecture
- **Decision:** Proceed to PoC if devices meet specs within 20%

---

## Phase 3: Proof-of-Concept Chip (Months 37-60)

### Objective
Full KV cache subsystem integration with end-to-end validation.

### PoC Chip Architecture
```
┌─────────────────────────────────────────────────────────────┐
│              PROOF-OF-CONCEPT CHIP                          │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────┐   │
│  │          DIGITAL CONTROLLER (22nm CMOS)             │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────────┐   │   │
│  │  │   PCIe   │  │   CXL    │  │  KV Cache Mgmt   │   │   │
│  │  │   PHY    │  │  Logic   │  │    Engine        │   │   │
│  │  └──────────┘  └──────────┘  └──────────────────┘   │   │
│  └─────────────────────────────────────────────────────┘   │
│                            │                                │
│  ┌─────────────────────────┼─────────────────────────┐     │
│  │      ANALOG ARRAY (BEOL RRAM or PCM)             │     │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ │     │
│  │  │  Tile 0 │ │  Tile 1 │ │  Tile 2 │ │ Tile N  │ │     │
│  │  │  256KB  │ │  256KB  │ │  256KB  │ │  256KB  │ │     │
│  │  │  analog │ │  analog │ │  analog │ │  analog │ │     │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ │     │
│  │          Total: 1-4MB Analog KV Cache             │     │
│  └───────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

**Target Specifications:**
| Parameter | Target |
|-----------|--------|
| Analog capacity | 1-4 MB |
| Access latency | <500ns |
| Power | <5W active |
| Interface | PCIe 5.0 x4 or CXL 2.0 |
| Form factor | PCIe card |
| Die size | ~50-100 mm² |
| Estimated cost | $5M-8M (masks + fab) |

### Software Stack
- Linux kernel driver
- PyTorch extension for analog KV cache
- Profiling and diagnostics tools
- Reference LLM implementations (Llama, GPT-style)

### Deliverables
- Functional PoC chip
- Linux driver and ML framework integration
- Benchmark results on real LLMs
- Customer demo capability

### Learning Loop
- **Metric:** Real LLM throughput vs. projection
- **Feedback:** Final architecture tuning for production
- **Decision:** Proceed to production if customers validate value

---

## Phase 4: Production Silicon (Months 61-96)

### Objective
Commercial product with manufacturing scale.

### Production Architecture
- Advanced process node (7nm or 5nm digital + analog BEOL)
- 256MB-1GB analog KV cache capacity
- Multi-chip modules or 2.5D stacking
- Full CXL 3.0 support

**Manufacturing Strategy:**
- Fabless model with foundry partners (TSMC, Samsung)
- Analog cells in BEOL (back-end-of-line) — compatible with standard CMOS
- Known-good-die (KGD) testing for analog arrays
- Yield learning from PoC data

**Product Variants:**
| Variant | Capacity | Target Market | Est. Price |
|---------|----------|---------------|------------|
| A100-AM | 256MB | Edge inference | $2K-4K |
| A500-AM | 512MB | Cloud inference | $4K-8K |
| A1000-AM | 1GB | Training + inference | $8K-15K |

### Go-to-Market
- Cloud partnerships (AWS, Azure, GCP) for early access
- Direct sales to AI labs (OpenAI, Anthropic, DeepMind)
- Reference designs for system integrators

---

## Cross-Phase Learning Architecture

### Feedback Loops

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Phase 0    │────►│  Phase 1    │────►│  Phase 2    │
│ Simulation  │     │  FPGA       │     │  Test Chip  │
└─────────────┘     └─────────────┘     └─────────────┘
      ▲                   ▲                   ▲
      │                   │                   │
      └───────────────────┴───────────────────┘
                Continuous Feedback
                        │
                        ▼
              ┌─────────────────────┐
              │  Architecture       │
              │  Evolution Engine   │
              │  (automated tuning) │
              └─────────────────────┘
```

### Automated Tuning Loop
At each phase, data feeds into architecture optimization:
1. **Characterization data** → Update device models
2. **Performance measurements** → Tune system parameters
3. **Accuracy results** → Adjust training procedures
4. **Customer feedback** → Prioritize features

### Risk Management

| Risk | Phase 0 | Phase 1 | Phase 2 | Phase 3 |
|------|---------|---------|---------|---------|
| Device variability | Model | Inject | Measure | Compensate |
| Interface timing | Simulate | Validate | Confirm | Optimize |
| Thermal issues | Estimate | Emulate | Test | Solve |
| Integration bugs | Find in sim | Find on FPGA | Fix in RTL | Verify |

---

## Resource Requirements by Phase

### Personnel
| Phase | Analog IC | Digital IC | Software | ML | Validation | Total |
|-------|-----------|------------|----------|----|------------|-------|
| 0 | 1 | 1 | 2 | 2 | 0 | 6 |
| 1 | 2 | 2 | 3 | 2 | 1 | 10 |
| 2 | 3 | 2 | 2 | 1 | 2 | 10 |
| 3 | 4 | 4 | 4 | 3 | 4 | 19 |
| 4 | 6 | 6 | 6 | 4 | 6 | 28 |

### Equipment & Facilities
- **Phase 0:** Standard compute cluster ($50K)
- **Phase 1:** FPGA boards ($50K), lab equipment ($100K)
- **Phase 2:** MPW fabrication ($500K-2M), probe station ($200K)
- **Phase 3:** Full mask set ($5M), ATE testing ($1M)
- **Phase 4:** Production masks ($15M+), volume test capacity

---

## Partnership Strategy

### Foundry Partners
| Foundry | Strength | NVM Offering | Best For |
|---------|----------|--------------|----------|
| **TSMC** | Leading edge, high volume | RRAM, MRAM | High-performance products |
| **Samsung** | Memory expertise | MRAM | Future MRAM roadmap |
| **GlobalFoundries** | Cost-effective, analog focus | CBRAM, MRAM | Cost-sensitive, analog-intensive |
| **STMicro** | Automotive, PCM | PCM | Automotive applications |

### Technology Partners
- **EDA:** Cadence, Synopsys for analog simulation and physical design
- **IP:** Synopsys DesignWare for PCIe/CXL controllers
- **Packaging:** ASE, Amkor for advanced packaging

### Academic Partnerships
- **MIT, Stanford:** Next-gen device research
- **UC Berkeley:** Architecture and systems research
- **ETH Zurich:** ML optimization for analog systems

---

## Success Criteria

| Milestone | Target Date | Criteria |
|-----------|-------------|----------|
| Phase 0 Complete | Month 6 | Simulation accuracy within 10% of theory |
| Phase 1 Complete | Month 18 | FPGA validates latency/power projections |
| Phase 2 Complete | Month 36 | Devices meet spec, models calibrated |
| Phase 3 Complete | Month 60 | End-to-end LLM demo with customers |
| Product Launch | Month 84 | Production volume, customer adoption |

---

## Open Research Questions

1. **Foundry selection:** Confirm TSMC vs. GF through MPW results
2. **RRAM vs PCM:** Final technology selection based on characterization
3. **Packaging:** 2.5D vs. monolithic integration trade-offs
4. **Software ecosystem:** Deep integration with vLLM, Triton
5. **Customer validation:** Secure design partners for Phase 3

---

## References

### MPW and Foundry
1. Europractice IC — "2025 Run Schedules and Prices"
2. GlobalFoundries — "GlobalShuttle Multi-Project Wafer Program"
3. AnySilicon — "MPW Cost Explained: What You Actually Pay"

### NVM Technology
4. Yole Group — "Embedded and Stand-Alone NVM: Two Different Futures?"
5. SemiWiki — "Memory Matters: The State of Embedded NVM 2025"
6. EE Times — "Emerging NVMs Reaffirm Potential"

### FPGA Platforms
7. Intel — "Agilex FPGA Device Overview"
8. AMD/Xilinx — "Versal ACAP Technical Reference"
9. Wevolver — "FPGA Architecture: A Comprehensive Guide"

### EDA Tools
10. Cadence — "Spectre Simulation Platform"
11. Synopsys — "PrimeSim HSPICE"
12. SemiEngineering — "Synopsys Leads AI-Powered Analog Design Innovation"

---

*Document Version: 1.0 — Research Complete  
Last Updated: March 20, 2026  
Research Lead: Orthia*
