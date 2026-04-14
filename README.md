**Uchū Gijutsu v2.0 — Space AI ASIC Simulator**  
**宇宙技術** | *Uchū Gijutsu*  

**Full-fidelity simulator for radiation-hardened AI accelerators in orbital environments.**

[![Python](https://img.shields.io/badge/Python-3.11-blue)](https://www.python.org)  
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)  
**Creator:** Wombat Rider  
**Version:** 2.0 (April 2026)

---

### Overview

Uchū Gijutsu v2.0 is a **physics-first, end-to-end simulator** for custom space-grade AI ASICs. It models a systolic-array accelerator under real orbital conditions, coupling:

- Compute (FP8/FP16/INT8/INT4 + realistic 2:4 structured sparsity)
- Memory hierarchy (HBM + on-chip SRAM + Roofline model)
- Thermal dynamics (capacitance, radiator view factor, mission degradation, temperature-dependent DVFS)
- Radiation effects (SEU/SET/SEL, per-region rates, MBU clustering, critical-path SET filtering, RHBDv3)
- Orbital environment (LEO/MEO/GEO/Deep Space, solar flares, eclipses, CREME96-inspired SEU scaling)

v2.0 is a major upgrade from v1.0, delivering production-grade performance, parallel execution, multi-objective optimization, global sensitivity analysis, and Weibull-based mission lifetime prediction.

**Key slogan:** *“Simulate once. Fly forever.”*

---

### Features

**Core Simulation**
- Accurate per-iteration orbital time advancement (no floating-point drift)
- Thermal capacitance ODE with solar load, radiator degradation, and smooth DVFS throttling
- Pure-NumPy radiation hot paths (10–30× faster Monte-Carlo)
- Realistic 2:4 structured sparsity speedup model
- Area/power scaling with `array_size²` + process-node leakage
- Full transformer workload generator

**Analysis & Optimization Engine**
- Parallel parameter sweeps (`joblib`)
- Single-objective optimization (SciPy Differential Evolution)
- Multi-objective Pareto front (pymoo NSGA-II)
- Global sensitivity analysis (SALib Sobol indices)
- Monte-Carlo reliability with isolated RNGs + Weibull lifetime model
- Built-in benchmarking against real chips (H100, Groq LPU, Cerebras WSE-3, etc.)

**Engineering Excellence**
- Hierarchical OmegaConf config (with JSON/YAML fallback)
- Singleton logger + immutable stats
- Comprehensive unit tests (32 tests covering v2 improvements)
- Clean export: CSV, JSON, interactive HTML dashboard, Plotly/Matplotlib visuals

---

### Requirements

**Core Dependencies** (see `requirements.txt`)

```txt
numpy>=1.24
scipy>=1.11
matplotlib>=3.7
plotly>=5.17
pandas>=2.0
torch>=2.1 (CPU only)
wandb>=0.16 (optional)
pyyaml>=6.0
psutil>=5.9
dataclasses-json>=0.6
omegaconf>=2.3
joblib>=1.3
SALib>=1.4
pymoo>=0.6
numba>=0.58
onnx / onnxruntime (optional)
```

**Recommended Environment**

```bash
conda create -n uchugijutsu2 python=3.11
conda activate uchugijutsu2
pip install -r requirements.txt
# or use environment.yml
conda env create -f environment.yml
```

**Optional for full features**
- `wandb` for sweep logging
- GPU not required (all hot paths are NumPy/CPU)

---

### Installation

```bash
git clone https://github.com/your-org/uchugijutsu.git
cd uchugijutsu
pip install -e .
```

Place the full codebase in a single file (`uchugijutsu.py`) or split into modules as needed.

---

### Quick Start

```bash
python uchugijutsu.py \
  --array-size 256 \
  --freq 1.8 \
  --dtype int8 \
  --sparsity 0.5 \
  --orbit LEO \
  --solar moderate \
  --transformer-layers 8 \
  --iterations 10 \
  --monte-carlo
```

**Example output**
```
TFLOPS (avg)          : 1847.32
TFLOPS / Watt         : 4.821
Reliability Score     : 0.9987
SDC Rate              : 2.4e-05 / hr
Peak Temperature      : 312.4 K (39.2 °C)
Mission lifetime      : 14230 hrs (Weibull shape=1.5)
Ranked #2 by TFLOPS/W among 6 real chips
```

HTML report + interactive dashboard automatically generated in `./uchugijutsu_results/`.

---

### Architecture Overview

```
Uchū Gijutsu v2.0
├── Config Layer (OmegaConf + dataclasses)
├── Memory Subsystem + Roofline Model
├── Thermal & Power Model (DVFS v2 + capacitance)
├── Radiation Effects (RHBDv3 with MBU & SET filtering)
├── Orbital Environment (eclipse + flare Poisson process)
├── Compute Array (FP8/FP16/INT8 + 2:4 sparsity)
├── Workload Runner (transformer layers)
├── Analysis Engine
│   ├── Parameter Sweep (joblib)
│   ├── Pareto Optimization (pymoo NSGA-II)
│   ├── Sensitivity (SALib Sobol)
│   └── Monte-Carlo Reliability (Weibull)
├── Visualization Dashboard (Matplotlib + Plotly)
└── Export (CSV / JSON / HTML)
```

Full module-by-module documentation is embedded in the source code (see Table of Contents at top of `Uchu Gijutsu V2.txt`).

---

### Strengths

- **Unparalleled realism** for space AI: couples thermal inertia, radiator physics, radiation clustering, and orbital dynamics in a single loop.
- **Blazing-fast Monte-Carlo** thanks to pure-NumPy error injection and isolated RNGs.
- **True design-space exploration**: Pareto fronts, Sobol sensitivity, and parallel sweeps make it perfect for hardware-software co-design.
- **Production-grade engineering**: immutable stats, proper seeding, singleton logger, comprehensive tests.
- **Extensible**: clean dataclass architecture and modular design.

---

### Use Cases

1. **Space AI Hardware Design** — Optimize array size, process node, sparsity, and DVFS for maximum TFLOPS/W under radiation and thermal constraints.
2. **Mission Reliability Prediction** — Forecast SDC rates and Weibull lifetime for 5–15 year LEO/GEO/Deep-space missions.
3. **Radiation Hardening Studies** — Compare RHBD techniques (ECC, TMR, scrubbing) with realistic MBU and SET models.
4. **Trade-off Analysis** — Run global sensitivity to identify which parameters (frequency, area, HBM BW, etc.) dominate performance/reliability.
5. **Benchmarking New Ideas** — Quickly evaluate novel dtypes, sparsity schemes, or interconnect designs before tape-out.
6. **Academic / Research** — Generate publication-ready plots, Pareto fronts, and Sobol indices.

---

### How to Deploy & Run

**1. Basic simulation**
```bash
python uchugijutsu.py --array-size 512 --dtype fp8 --monte-carlo
```

**2. Full parameter sweep**
```bash
python uchugijutsu.py --sweep --n-jobs -1
```

**3. Multi-objective Pareto optimization**
```bash
python uchugijutsu.py --pareto-optimize
```

**4. Sensitivity analysis**
```bash
python uchugijutsu.py --sensitivity
```

**5. Run unit tests**
```bash
python uchugijutsu.py --unit-tests
```

All outputs land in `./uchugijutsu_results/` (configurable via `--output-dir`).

**Docker support** (optional)
A `Dockerfile` can be added in <5 minutes using the conda environment.

---

### Suggested Improvements & Extensions

#### High-priority (easy wins)
- Adaptive Runge-Kutta time-stepping for thermal model (instead of fixed 1 s Euler)
- Scale dummy-array error injection by actual MAC/register count
- Gradual SRAM cache-hit model (instead of binary fit/no-fit)
- Add Total Ionizing Dose (TID) leakage growth over mission years

#### Medium-term extensions
- Additional workloads (CNN, ViT, RNN, GNN)
- GPU-accelerated Monte-Carlo (CuPy or Numba CUDA)
- Web dashboard (Streamlit/Gradio) for interactive design exploration
- Integration with real radiation test data (e.g., CERN/GSFC datasets)
- Support for CORDIC, analog, or photonic accelerators
- Proton direct-ionization model for MEO/GEO

#### Long-term vision
- Co-simulation with SPICE-level power grid
- Fault-injection campaign export for FPGA/ASIC emulation
- Cloud-scale sweeps (Kubernetes + Dask)
- Open-source radiation database connector (CREME96/SPENVIS API)

Contributions welcome — especially domain experts in radiation effects or space thermal control.

---

### License

MIT License. See `LICENSE` file.

**Citation** (suggested)
```bibtex
@misc{uchugijutsu2026,
  author = {Wombat Rider},
  title = {Uchū Gijutsu v2.0 — Space AI ASIC Simulator},
  year = {2026},
  note = {https://github.com/your-org/uchugijutsu}
}
```

---

**Made for the stars.**  
*Fly safe — and simulate smarter.* 🚀

*Questions? Figure it out yourselves and do not annoy Wombat Rider.*
