# Research Notes — Strategic Synthesis & Research Guidance

**Date:** March 19, 2026  
**Purpose:** Notes from re-reading initial material to guide deep research

---

## The Core Narrative Arc

### NVIDIA's Confession (The Hook for Investors)
GTC 2026 wasn't product launches—it was a **public admission of limits**.
- Jensen: *"The bottleneck of AI is no longer intelligence… it's energy + scale + cost of inference"*
- NVIDIA acknowledged KV cache is outgrowing HBM *today*
- GPU underutilization is happening *now*

**Investment thesis framing:** We're not pitching speculative tech. We're solving a problem NVIDIA admitted they can't solve digitally.

---

## Key Physics & Trends to Verify

### 1. The Memory Gap (Critical for Business Case)

**Current trajectory:**
| Year | HBM Capacity | KV Cache Needed | Gap |
|------|--------------|-----------------|-----|
| 2024 | ~80 GB | 20-120 GB | Manageable |
| 2025 | ~120 GB | 100-500 GB | 4x shortfall |
| 2026 (Rubin) | ~150-200 GB | 0.5-5 TB | 10-25x shortfall |
| 2027 | ~200 GB | 5-20 TB | 25-100x shortfall |

**Research needed:**
- [ ] Verify HBM capacity roadmap (HBM3, HBM4, HBM5)
- [ ] Verify KV cache scaling math (is it really 5-20 TB by 2027?)
- [ ] What are hyperscalers actually seeing in production?
- [ ] Quantify "GPU underutilization" — what percentage idle time?

**Connection to Research Area 1 (Business Case):** These numbers ARE the market opportunity.

---

### 2. The Energy Economics (The Killer Argument)

**Key claim:** Moving 1 bit costs more energy than computing it.

**Current AI energy breakdown (need to verify):**
- Data movement: ~60-90% of energy?
- Computation: ~10-40%?

**Analog potential:**
- Memristor crossbar: Matrix multiply "for free" via Ohm's/Kirchhoff's laws
- No data movement for attention computation
- Only ADC/DAC conversion costs

**Research needed:**
- [ ] Find actual energy numbers for HBM access vs. computation
- [ ] Quantify ADC/DAC overhead (how much does conversion cost?)
- [ ] Find memristor crossbar energy per MAC (multiply-accumulate)
- [ ] Compare: Digital HBM access vs. Analog crossbar access

**The $/million tokens calculation:**
- Current: $X (need baseline)
- With analog: $X/Y (need to calculate Y)
- Error correction overhead: Z% of gains

**Connection to Research Area 1 (Business Case):** This is the TCO model foundation.

---

### 3. Sun-Ni Law — The Governing Principle

**Statement:** As compute increases, problem size grows until memory becomes the bottleneck.

**AI-specific twist:** KV cache grows with:
- Context length (linear)
- Concurrent users (linear)
- Model size (indirectly)

**Result:** Exponential growth in memory demand vs. linear growth in memory supply.

**Research needed:**
- [ ] Sun-Ni Law citations and formal definition
- [ ] Historical precedents (what happened in HPC when memory hit walls?)
- [ ] AI-specific memory scaling research papers

**Connection to Research Area 1:** This is the "why now" — the law catching up to AI.

---

## The Analog Advantage — Specific Claims to Validate

### Claim 1: 10-100x Density Improvement

**Basis:** Analog cells store multi-bit values (resistance levels) vs. digital binary.

**Questions:**
- [ ] How many distinct resistance levels can memristors reliably hold? (4-bit = 16 levels? 6-bit = 64 levels?)
- [ ] What's the practical limit given noise/drift?
- [ ] How does this compare to HBM density (GB/mm²)?

**Research needed:**
- [ ] Memristor crossbar density specs from research papers
- [ ] Knowm or Crossbar technical specifications
- [ ] HBM3/HBM4 density numbers

**Connection to Research Area 2 (Hybrid Architecture):** Determines analog capacity per die.

---

### Claim 2: In-Place Computation

**Basis:** Ohm's Law (V=IR) + Kirchhoff's Current Law = matrix-vector multiply.

**How it works:**
1. Store weight matrix as conductances (1/R) in crossbar
2. Apply input voltages to rows
3. Read currents at columns (sum of I = V×G)
4. Result is matrix-vector product

**Questions:**
- [ ] What precision can this achieve? (How many bits?)
- [ ] What's the latency? (Nanoseconds? Microseconds?)
- [ ] How does noise affect computation accuracy?

**Research needed:**
- [ ] Academic papers on crossbar array computation
- [ ] Precision/latency/energy trade-offs
- [ ] Comparison to digital MAC units

**Connection to Research Area 2:** Core of the architecture advantage.

---

### Claim 3: KV Cache Tolerates Noise

**Basis:** AI models already use quantized weights (FP4, INT8, etc.), so they're robust to precision loss.

**Questions:**
- [ ] What's the actual precision needed for KV cache? (Not FP16, but what?)
- [ ] Studies on quantization impact on attention mechanisms?
- [ ] How much noise/error can attention tolerate before quality degrades?

**Research needed:**
- [ ] Papers on KV cache quantization
- [ ] Attention mechanism precision requirements
- [ ] Model accuracy vs. precision trade-offs

**Connection to Research Area 2:** Determines error correction requirements.

---

## The Hard Problems — How the Hybrid Approach Solves Them

### Problem 1: Noise & Drift

**Digital-only approach:** Prevent noise (impossible with analog).

**Hybrid approach:** Allow noise, detect with digital, correct as needed.

**Key insight:** We don't need perfect analog—we need analog that's "good enough" that correction costs don't erase gains.

**Research needed:**
- [ ] Memristor drift rates (how fast do values change?)
- [ ] What causes drift? (Temperature, time, read/write cycles?)
- [ ] Error detection mechanisms (periodic sampling? continuous monitoring?)
- [ ] Correction strategies (refresh from digital copy? predictive models?)

**Connection to Research Area 2:** The error correction protocol is the core innovation.

---

### Problem 2: ADC/DAC Bottleneck

**The risk:** Converting analog ↔ digital costs energy and adds latency.

**Hybrid mitigation:**
- Keep data in analog domain as long as possible
- Only convert when necessary (final results, not intermediate)
- Use low-precision ADC/DAC (not 16-bit, maybe 4-6 bit)

**Research needed:**
- [ ] ADC/DAC energy per conversion (various precision levels)
- [ ] How often do we need to convert in attention computation?
- [ ] Can we architect to minimize conversions?

**Connection to Research Area 2:** Latency/energy budget calculations.

---

### Problem 3: Programming Variability

**Challenge:** Writing precise analog values is hard because each cell varies.

**Hybrid approach:**
- Use digital to calibrate and compensate
- Closed-loop programming (write, verify, adjust)
- Accept some variability, correct in digital domain

**Research needed:**
- [ ] Memristor programming techniques
- [ ] Calibration algorithms
- [ ] Write endurance (how many cycles before degradation?)

**Connection to Research Area 2:** Manufacturing/calibration strategy.

---

### Problem 4: Software Stack Gap

**Current state:** No CUDA for analog.

**Hybrid opportunity:**
- Analog looks like a "memory accelerator" to software
- CUDA extensions for KV cache offloading
- PyTorch/TensorRT integration

**Research needed:**
- [ ] Existing analog computing software stacks (Mythic? IBM?)
- [ ] How to integrate with CUDA ecosystem
- [ ] API design for KV cache acceleration

**Connection to Research Area 4:** Commercialization/software strategy.

---

## The Phased Rollout — Strategic Logic

### Why Phased?

**De-risking through learning:**
- Phase 1: Learn drift patterns in real workloads
- Phase 2: Optimize based on learnings
- Phase 3: Scale with proven technology

**Customer progression:**
- Phase 1: Research labs (tolerant, curious)
- Phase 2: Cloud providers (sophisticated, cost-sensitive)
- Phase 3: Mass market (requires proven solution)

**Connection to Research Area 4:** The learning architecture is the de-risking mechanism.

---

## The Competitive Landscape — Key Players

### Knowm
- Memristor-based computing
- Status: Startup, limited traction
- Differentiation: General analog computing, not AI-specific

### Crossbar Inc.
- ReRAM/memristor arrays
- Status: Commercial products
- Differentiation: General memory, not compute-in-memory

### Mythic AI
- Analog compute-in-memory
- Status: Shifted from AI to edge
- Lesson: Market timing matters

### IBM Research
- Analog AI accelerators
- Status: Research phase
- Threat: If they commercialize, major competition

### NVIDIA
- Status: Acknowledged problem, no analog solution announced
- Strategy: Complement, not compete

**Research needed:**
- [ ] Deep dive on each competitor's IP portfolio
- [ ] Patent landscape analysis
- [ ] Why did Mythic pivot? (What can we learn?)

**Connection to Research Area 5:** Competitive intelligence and positioning.

---

## The NVIDIA Partnership — Strategic Considerations

### Why NVIDIA Won't Build This Themselves (Yet)

1. **Focus:** They're in "yearly cadence" mode (Blackwell → Rubin → Feynman)
2. **Risk:** Analog is unproven at scale
3. **Philosophy:** They build platforms, not specialized accelerators

### Why NVIDIA Needs This

1. **Rubin gap:** KV cache problem will hurt Rubin performance
2. **Energy:** Data center power limits are real
3. **Competition:** Groq, Cerebras gaining ground

### Partnership Strategy

**Phase 1:** Prove value with sidecar (low risk for NVIDIA)
**Phase 2:** Co-design for Rubin successor (deeper integration)
**Phase 3:** Acquisition or long-term licensing

**Connection to Research Area 5:** Partnership and IP strategy.

---

## Research Priorities (Ranked)

### Critical (Must Have for Investor Doc)
1. **Memory gap quantification** — HBM roadmap vs. KV cache growth
2. **Energy economics** — $/million tokens comparison
3. **Analog density/precision** — Can we actually achieve 10-100x?
4. **Competitive landscape** — Who else is doing this?

### Important (Strengthens Case)
5. **Error correction protocols** — How much overhead?
6. **ADC/DAC costs** — Do they erase gains?
7. **Software integration** — How hard is CUDA extension?
8. **Patent landscape** — What's already claimed?

### Nice to Have (Adds Detail)
9. Manufacturing partnerships (TSMC, Intel, etc.)
10. Specific analog cell comparisons (memristor vs. PCM vs. capacitive)
11. Historical precedents (when other tech hit memory walls)

---

## Open Questions for Deep Research

1. **What is the actual KV cache size for current frontier models?** (GPT-4, Claude, Gemini)
2. **What are hyperscalers spending on memory vs. compute?**
3. **How much of GPU time is spent waiting for memory?**
4. **What's the state of memristor manufacturing?** (Yield, reliability, cost)
5. **Who are the key researchers in analog AI acceleration?** (Academic labs)
6. **What government funding is available?** (CHIPS Act, etc.)
7. **What did Mythic learn from their pivot?** (Avoid their mistakes)
8. **How does Groq's approach compare?** (LPU vs. analog)
9. **What is NVIDIA's actual stance on analog?** (Any internal research?)
10. **What are the hard physics limits of HBM?** (When does it truly stop scaling?)

---

## Narrative Themes to Weave Through Research

### Theme 1: The Inevitability
Analog isn't speculative—it's the only solution that scales to meet AI's memory demands.

### Theme 2: The Timing
Rubin (2026-2027) is the inflection point. We ship before Feynman (2028) to establish position.

### Theme 3: The Pragmatism
Hybrid approach de-risks the technology. We don't need perfect analog—we need good enough analog with digital backup.

### Theme 4: The Partnership
We're not competing with NVIDIA. We're making Rubin better.

### Theme 5: The Learning
Phased rollout lets us learn from production data, improving with each generation.

---

## Connection to 5 Research Areas

### Area 1: Business Case
- Memory gap quantification = market size
- Energy economics = TCO advantage
- Token economics = unit of value

### Area 2: Hybrid Architecture
- Error correction = core innovation
- Precision trade-offs = technical feasibility
- Rubin integration = go-to-market path

### Area 3: Technology Patterns
- IP moat = defensibility
- Patent landscape = freedom to operate
- Step-change innovations = future value

### Area 4: Commercialization
- Phased rollout = de-risking
- Learning architecture = continuous improvement
- Partnership milestones = credibility

### Area 5: IP Strategy
- Competitive analysis = positioning
- Patent strategy = defensibility
- NVIDIA relationship = path to market

---

## Research Methodology Notes

**Sources to prioritize:**
1. Academic papers (IEEE, arXiv) — Technical depth
2. NVIDIA technical docs — Integration requirements
3. Industry reports (Gartner, IDC) — Market sizing
4. Patent databases — Competitive intelligence
5. Financial filings — TCO data

**Search terms:**
- "KV cache scaling"
- "Memristor crossbar array"
- "Analog in-memory computing"
- "AI accelerator memory bottleneck"
- "HBM roadmap"
- "Compute-in-memory"
- "Neuromorphic computing"
- "Analog neural network accelerator"

---

**Next Action:** Begin systematic research on critical questions above, starting with memory gap quantification and energy economics.
