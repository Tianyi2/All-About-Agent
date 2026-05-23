# Agent in Production Survey

> **Paper:** Measuring Agents in Production (MAP)
> **Venue:** Workshop paper at "Agentic AI in the Wild", ICLR 2026
> **Affiliations:** UC Berkeley, Intesa Sanpaolo, UIUC, Stanford, IBM Research
> **Source:** `resources/academic_papers/MEASURING AGENTS IN PRODUCTION.pdf`

## Keywords
- LLM Agents, Production Deployment, Empirical Study, Reliability, Human-in-the-Loop, System-Level Design, Evaluation, Benchmark Scarcity

## What they done?
- First large-scale **systematic study of AI Agents in production**.
- **20 case studies** via in-depth interviews with deployment teams (30–90 min, semi-structured) + **survey of 306 practitioners** across **26 domains**, filtered to **86 deployed agents** (production / pilot phases).
- Study window: **April – November 2025**.
- Four research questions:
  - RQ1: What are the applications of agents?
  - RQ2: What models, architectures, and methods are used?
  - RQ3: How are agents evaluated?
  - RQ4: What are the top challenges in deployment?

## Findings
- **Benchmarks are not enough** to cover the diverse use cases (especially rare cases / niche industries).
- **[Why] Why organizations build agents**
  - Increase **productivity (80%)**, primarily for human users.
  - **66%** allow response time of **minutes or longer** (latency-tolerant).
  - Risk mitigation (12%) and interdisciplinary expertise reduction (18%) are less common — harder to quantify.
  - **83%** of practitioners who evaluated alternatives prefer agents over non-agentic solutions.
- **[What] Applications of agents**
  - **26 distinct domains** observed; 17 design dimensions characterized (Figure 2, Table 1, Table 3).
  - Top sectors: **Technology (48%) + Finance & Banking (44%) + Corporate Services (42%)**.
  - **[Who] Agent End-users:** Internal employees (**52%**), external customers (**40%**), non-human systems (**8%**). 92.5% serve humans.
- **[How] Models, architectures, and methods**
  - **68%** agents execute **≤10 steps** before human intervention; **47%** execute fewer than 5.
  - **70%** rely on **prompting off-the-shelf models** rather than weight tuning (SFT / RL).
  - Prompt construction: **44.6% Manual + AI** & **33.9% Fully Manual** (humans dominate; automated optimization only 9%).
  - **80% (16/20) cases run agents in structured workflows** rather than open-ended autonomous planning.
  - **85% (17/20) build custom in-house implementations with direct API calls**; survey shows **61% use agent frameworks** (LangChain/LangGraph at 25%) — 2 teams **migrated away** from frameworks (e.g., CrewAI) to custom.
  - **17/20** rely on **proprietary frontier models** (Claude Sonnet 4 / Opus 4.1, GPT o3); open-source only when constrained by cost or regulation.
  - **59%** of deployed agents coordinate multiple models (cost optimization + modality + operational lock-in from upgrades).
  - **Deliberately trade capability for controllability to maintain reliability.**
- **[When-evaluate] How are agents evaluated**
  - Rely primarily on **human-in-the-loop evaluation (74%)** with **LLM-as-a-judge (52%)** as complement.
  - **75%** of teams **forgo formal benchmarking** — no benchmark exists; rely on **A/B testing + expert feedback**.
  - Benchmarks are hard to use (different scenario, runtime complexity, regulated-domain data restrictions, client-specific customization).
  - Only **39%** compare to non-agentic baselines; **26%** report no meaningful baseline exists.
- **Top challenges in deployment**
  - **Core Technical Performance (reliability, robustness, scalability)** ranked top by **37.9%** — far above governance (3.4%) or compliance (17%). → *Opens question: fuzzing-style benchmark generation?*
  - **Reliability** is the primary bottleneck — addressed via **system-level design** (see §7.1): read-only modes, sandboxed verification, internal-first deployment, wrapper APIs, RBAC, autonomy bounding.
  - **Evaluation Challenge:** benchmark scarcity + real-world tasks hard to verify (no ground truth, delayed feedback signals).
  - **Security & Privacy (see §7.3):** **69%** of agents retrieve confidential data. Security is currently achieved implicitly via system-level design (read-only, internal-only). Remains an open challenge as deployments expand outward.

## Advantages
- First empirical, engineering-level cross-organizational dataset on production agents — fills the gap left by commercial surveys (no technical depth) and single-system papers (no breadth).
- Triangulates two data sources (deep case studies + broad survey) over a 7-month window.
- Provides **13 findings** structured against 4 RQs; offers reusable taxonomy across 17 design dimensions.
- Captures both **what works** and **what is deliberately avoided** (e.g., open-ended autonomy, weight tuning, frameworks at scale).

## Limitations
- **Geographic bias:** case-study teams concentrated in Americas and Europe.
- **Participation bias:** survey skewed toward professional networks (LinkedIn, Discord, X, conferences) of the authors.
- **Temporal bias:** 7-month rolling collection may have introduced drift as model capabilities and tooling evolved during the window.
- Findings bounded to practitioners **willing to share** implementation details — proprietary teams may be under-represented.
- Workshop paper scope — limited deep-dive into per-domain quantitative comparisons.

## Thinking
- The paper validates a counterintuitive direction: **production maturity ≠ more autonomy**. The most "agentic" deployments deliberately strip autonomy down to bounded workflows.
- The **<10-step constraint (68%)** is striking — research benchmarks (SWE-bench, etc.) reward long-horizon autonomy, but production rewards short, verifiable loops. There is a research-practice gap to close.
- **Agent frameworks** appear to add value at prototype stage but become a liability at production scale (flexibility + simplicity + security all push toward custom).
- **Human-in-the-loop is treated as a permanent architectural primitive, not a temporary crutch** — this should reshape how research frames "fully autonomous" as the goal.
- Model migration brittleness (newer ≠ better when scaffolds lock onto specific behaviors) is an under-discussed operational cost.

## Key Takeaways
- **Designing agents for production (esp. enterprise workflows): deliberately trade capability for controllability to maintain reliability.**
- **Reliability of agents:** practitioners achieve reliability through best practices in **system-level design**, not model-level or algorithmic advances.
- **Deploy agent internally first** to mitigate reliability and security risks (52% serve internal employees).
- **Trade speed for correctness** — 66% of agents tolerate minute-scale or longer latency; asynchronous designs are first-class.
- **Agent frameworks may not be ideal for production / enterprise workflows** (areas requiring heavy customization, security boundaries, and dependency control) — **85% of production teams build custom**.
- **Structured workflows beat open-ended planning** (80% of cases) — bounded autonomy is the design principle.
- **Human-in-the-loop is a design principle, not a workaround** — 74% of evaluation and most runtime verification still depend on humans.
