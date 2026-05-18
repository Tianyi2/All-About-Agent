# AI Agent — Technical Development Report Study Note

## Summary

This note distills the *AI Agent 智能体技术发展报告* (China Academy of Industrial Internet, Jan 2026) into a developer-oriented overview of what an AI Agent is, how it is engineered, and where it is being deployed. It walks through the modern definition (autonomy over automation), the four-module cognitive loop (Perception → Brain → Action → Memory), the decision frameworks that drive the "brain" (CoT, ReAct, Plan-and-Execute, Reflection), Multi-Agent Systems (MAS) and the MCP / A2A protocols that let agents interconnect, the open-source frameworks and low-code platforms that ship them, the industry application patterns that pay the bills, and finally the security, ethical, privacy, and legal risks that come with autonomy. For more check the [original report reference](https://mp.weixin.qq.com/s/x3eigelOnJJi-F2qzBxihg?scene=2).

---

## Table of Contents

- [What Is an Agent](#what-is-an-agent)
  - [Agent Components from Engineering View](#agent-components-from-engineering-view)
  - [Agent Components from Functional View](#agent-components-from-functional-view)
    - [Brain](#brain)
      - [Chain-of-Thought (CoT)](#chain-of-thought-cot)
      - [ReAct (Reason + Act)](#react-reason--act)
      - [Plan-and-Execute](#plan-and-execute)
      - [Reflection & Self-Critique](#reflection--self-critique)
      - [Framework comparison](#framework-comparison)
    - [Action](#action)
      - [Function Calling — the core mechanism](#function-calling--the-core-mechanism)
    - [Memory](#memory)
      - [Short-Term Memory (conversation history)](#short-term-memory-conversation-history)
      - [Long-Term Memory (cross-session knowledge)](#long-term-memory-cross-session-knowledge)
- [Multi-Agent Systems (MAS)](#multi-agent-systems-mas)
  - [Why MAS?](#why-mas)
  - [MAS architectures](#mas-architectures)
  - [Inter-agent coordination mechanisms](#inter-agent-coordination-mechanisms)
  - [Communication protocols](#communication-protocols)
- [My Understanding](#my-understanding)
- [Agent Frameworks](#agent-frameworks)
  - [Why use/not use agent frameworks](#why-usenot-use-agent-frameworks)
- [Agent Protocols](#agent-protocols)
- [Agent Development Approaches](#agent-development-approaches)
- [Agent Platforms (Low-Code)](#agent-platforms-low-code)
  - [Selection cheat sheet](#selection-cheat-sheet)
- [Agent Deployment Challenges](#agent-deployment-challenges)
- [In-Business Agent Products — Application Landscape](#in-business-agent-products--application-landscape)
  - [Finance — "the disruptor of intelligent transformation"](#finance--the-disruptor-of-intelligent-transformation)
  - [Industry & Manufacturing — "from automation to autonomy"](#industry--manufacturing--from-automation-to-autonomy)
  - [E-commerce & Customer Service](#e-commerce--customer-service)
  - [Other emerging sectors](#other-emerging-sectors)
  - [ROI snapshot](#roi-snapshot)
- [Challenges with AI Agents](#challenges-with-ai-agents)
  - [Technical Security Risks](#technical-security-risks)
  - [Ethics, Bias, and Social Risks](#ethics-bias-and-social-risks)
  - [Data and Privacy Security](#data-and-privacy-security)
  - [Law and Regulation](#law-and-regulation)
- [Thinking — Personal Takeaways](#thinking--personal-takeaways)
- [Research Areas and Open Needs](#research-areas-and-open-needs)

## What Is an Agent

**Agent loop** is the key of any modern agnet. The loop can seen as a continuous calling of tools and mcps or repeating perception -> brain -> action -> memory.

### Agent Components from Engineering View

When building an agent in practice, you wire together a handful of engineering primitives:

- **Tools** — the agent's capability surface:
  - Filesystem actions (read / write / list)
  - Bash / shell execution
  - `TodoWrite` style task tracking
  - Subagents — delegating work to specialized children
  - Skills — reusable, named capabilities
  - Tasks — long-running background units
  - ...
- **MCP (Model Context Protocol)** — standardized way to expose tools, data, and services to the LLM from external sources (see [Agent Protocols](#agent-protocols--mcp-and-a2a)).
- **Memory** — both per-session conversation history and longer-term persisted knowledge.
  - **Compaction** — automatic summarization to keep the context window healthy.
- **Permissions** — what the agent is allowed to do without asking, and where it must defer to the user.

### Agent Components from Functional View

The same agent, viewed as cognitive science rather than engineering, decomposes into four modules that mirror human cognition:


| Module                 | Role                                                                                                                        | Analogy                |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------- | ------------------------ |
| **Perception** (input) | Encode multimodal external information (text, image, audio, video, structured data) into vectors the brain can reason over. | Five senses            |
| **Brain** (LLM)        | Reason, plan, decompose goals into actionable sub-steps, decide which tool to call.                                         | Central nervous system |
| **Action** (output)    | Execute the brain's decisions by invoking tools — APIs, code interpreters, robots.                                         | Hands and feet         |
| **Memory**             | Store short-term context and long-term knowledge so the agent can learn across tasks.                                       | Memory                 |

The Perception module uses encoders like BERT (text), ViT (image), Whisper (audio) to produce unified embeddings, often fused via **cross-attention** for true multimodal understanding (the "1+1>2" effect).

#### Brain

The Brain is where intelligence lives. Four decision frameworks dominate 2025 practice; they are **not mutually exclusive** and are often composed.

##### Chain-of-Thought (CoT)

Proposed by Google researchers in 2022. Prompt the LLM to *think step by step* before answering. This "slow thinking" dramatically improves accuracy on multi-step reasoning problems and is the substrate every other framework builds on.

> **Zero-shot CoT example.** Q: *"A basket has 5 apples. Xiaoming takes 2, then puts 1 back. How many apples are in the basket now?"* — prompted with "Let's think step by step", the model explicitly reasons: $5 - 2 = 3$; $3 + 1 = 4$. **Final answer: 4.**

##### ReAct (Reason + Act)

Proposed by Princeton + Google. Interleaves reasoning with tool calls in a tight loop:

1. **Thought** — reason about current state and goal.
2. **Action** — pick and invoke a tool.
3. **Observation** — feed the tool result back as input to the next thought.

The Thought → Action → Observation cycle continues until the agent decides the task is done.


| Strengths                                                   | Weaknesses                                               |
| ------------------------------------------------------------- | ---------------------------------------------------------- |
| Dynamic and adaptive — adjusts based on real-time feedback | High latency / cost — many round-trips with LLM + tools |
| Highly explainable — every step is logged                  | A complex task may need 5–10+ loops                     |
| Strong error recovery — can re-try on failure              |                                                          |

##### Plan-and-Execute

Splits the task into two clean phases:

1. **Planning** — a dedicated *Planner Agent* drafts a complete, ordered step list up front.
2. **Execution** — one or more *Executor Agents* walk the plan and call tools, without re-planning.

**Pros:** structured, predictable, fewer LLM calls (lower cost & latency).
**Cons:** brittle to surprises — if the environment changes mid-plan, the whole plan may need to be rebuilt.

##### Reflection & Self-Critique

After producing an initial result, the agent (or a separate *Critic Agent*) **evaluates** the output for completeness, correctness, and quality, then revises. This *act → reflect → improve* loop lets the agent self-iterate without human supervision. Representative frameworks: **Reflexion**, **LATS** (Language Agent Tree Search).

##### Framework comparison


| Framework        | Core idea                       | Strengths                                | Weaknesses                       | Best for                              |
| ------------------ | --------------------------------- | ------------------------------------------ | ---------------------------------- | --------------------------------------- |
| ReAct            | Interleave reasoning and acting | Adaptive, explainable                    | High cost / latency              | Open-ended, exploratory tasks         |
| Plan-and-Execute | Plan first, then execute        | Structured, efficient when goal is clear | Inflexible to surprise           | Well-defined, deterministic workflows |
| Reflection       | Self-evaluate and revise        | Higher quality output, self-improving    | Further amplifies cost / latency | Tasks with strict quality bars        |

In practice these compose: use Plan-and-Execute for the macro plan, ReAct inside each step, and Reflection at key checkpoints.

#### Action

In agent terminology, a **tool** is any external function, API, or service the agent can invoke. Common categories:

- **Information retrieval** — search engines, database queries, weather / stock / news APIs.
- **Compute & analysis** — calculators, code interpreters (Python, SQL), Pandas.
- **Content generation** — image generation (DALL·E 3, Midjourney), TTS.
- **Application control** — send email, create calendar events, drive a CRM.
- **Physical world** — robots, drones, smart-home devices.

##### Function Calling — the core mechanism

Function Calling lets the LLM emit a structured JSON object naming a function and its arguments, rather than executing anything itself.

```
1. Define Tools    → developer registers tools in JSON Schema
2. LLM decision    → on user input, LLM decides which (if any) tool to call
3. Generate args   → LLM emits {"name": "get_weather", "arguments": {"city": "Beijing"}}
4. External exec   → host app parses JSON and runs get_weather(city="Beijing")
5. Return result   → result fed back to LLM
6. Final response  → LLM composes natural-language reply
```

By 2025, native Function Calling is standard across GPT, Gemini, Claude, Qwen, and others.

#### Memory

A stateless agent is a "goldfish" — every conversation starts from scratch. Memory is split into two layers.

##### Short-Term Memory (conversation history)

- **Implementation:** the recent N turns of dialogue, included in the LLM's context window each call.
- **Challenge:** context windows are finite (even Gemini 2.5's multi-million-token window has cost and latency limits).
- **Compression strategies:**
  - **Sliding window** — keep only the last N turns.
  - **Summarization (compaction)** — periodically call an LLM to summarize older turns into a short digest.

##### Long-Term Memory (cross-session knowledge)

Implemented via markdown OR ... OR **RAG (Retrieval-Augmented Generation)**:

1. **Store** — embed the fact (e.g. *"user likes lattes"*) into a vector via an embedding model, persist in a vector DB.
2. **Retrieve** — on a related future query, embed the query and run a similarity search.
3. **Augment** — inject the retrieved memory into the prompt as additional context.
4. **Generate** — LLM produces a personalized answer.

###### Vector database options (2025)


| Database | Type             | Strength                     | Best for                            |
| ---------- | ------------------ | ------------------------------ | ------------------------------------- |
| Pinecone | Managed cloud    | Fully hosted, plug-and-play  | Prototypes, SMB apps                |
| Milvus   | Open source      | Distributed, highly scalable | Large-scale production              |
| Weaviate | Open source      | Multimodal, GraphQL API      | Complex / multimodal retrieval      |
| ChromaDB | Open source      | Lightweight, Python-native   | Local dev, data-science experiments |
| Redis    | OSS / commercial | In-memory, ultra-low latency | Real-time apps already on Redis     |

---

## Multi-Agent Systems (MAS)

A single all-purpose agent buckles under enterprise-scale workflows. **MAS** mirrors how a company organizes specialists — decompose a large goal, hand pieces to role-specialized agents, and aggregate the results.

### Why MAS?

- **Context Window Management** - seperate information between sessions to agents
- **Specialization** — each agent is an expert (data analyst, coder, report writer).
- **Task parallelism** — different sub-tasks run concurrently.
- **Scalability & robustness** — add agents to extend capability; one agent's failure doesn't collapse the system.
- **Complex system simulation** — model traffic, supply chains, financial markets, etc.

### MAS architectures


| Pattern          | How it works                                                                                 | Representative framework         |
| ------------------ | ---------------------------------------------------------------------------------------------- | ---------------------------------- |
| **Hierarchical** | A Manager / Orchestrator decomposes goals and assigns to Worker agents; workers report back. | AutoGen                          |
| **Peer-to-peer** | No central manager — agents communicate and coordinate directly, like an agile team.        | CrewAI                           |
| **Hybrid**       | Hierarchical at macro level, peer-to-peer inside each sub-team.                              | Used in large enterprise systems |

### Inter-agent coordination mechanisms

- **Blackboard** — all agents share a public data area; read tasks from and write results to the board. LangGraph's state-graph mechanism is a generalized blackboard.
- **Contract Net** — market-style bidding: one agent posts a task, others bid based on capability, the poster awards the contract.

### Communication protocols

For agents from different vendors to interoperate, the field standardized on **MCP** (tools / data, late 2024) and **A2A** (agent-to-agent, April 2025) — see the [protocols section](#agent-protocols--mcp-and-a2a).

---

## My Understanding

> An AI Agent is a system that interacts with an LLM to solve a problem or complete a task. The core mechanism is a "brain" that decides what to do and an iterative loop that performs actions until the task is complete. From this core, all the engineering features follow: tools, MCP, memory, skills, permissions — each one extends what the brain can perceive or actuate.

---

## Agent Frameworks

The 2025 framework landscape splits into general-purpose orchestration libraries and MAS-focused libraries. Picking one constrains your tech stack, deployment options, and even commercial model.

**The real-world takeaway:** Many teams start with a framework to validate an idea, then migrate to a leaner, hand-rolled implementation once they understand their actual requirements. The framework essentially becomes a reference, not a foundation.


| Framework                       | Focus                                                                   | Collaboration style              | Best for                                               |
| --------------------------------- | ------------------------------------------------------------------------- | ---------------------------------- | -------------------------------------------------------- |
| **LangChain**                   | General LLM-app framework, the de-facto industry standard               | Chain / Agent                    | Almost any agent — deep customization                 |
| **LangGraph**                   | State-graph for cyclic, long-running flows                              | Blackboard / state machine (MAS) | Workflows needing precise control, loops, persistence  |
| **AutoGen** (Microsoft)         | Multi-agent conversation orchestration                                  | Hierarchical (MAS)               | Code generation, research automation                   |
| **CrewAI**                      | Role + Task team metaphor                                               | Peer-to-peer (MAS)               | Business process automation, content / marketing teams |
| **MetaGPT**                     | Models a software company with SOPs                                     | Hierarchical + waterfall (MAS)   | Generate a full project from a one-line spec           |
| **ChatDev**                     | Simulates a full virtual dev team (CEO, CTO, programmer, tester)        | Hierarchical + waterfall (MAS)   | End-to-end software dev automation, research           |
| **Semantic Kernel** (Microsoft) | Enterprise, multi-language (C#, Java, Python), .NET / Azure integration | —                               | Enterprise apps in Microsoft ecosystem                 |
| **LlamaIndex**                  | RAG / data indexing specialist                                          | —                               | Knowledge bases, document Q&A, research assistants     |
| **Phidata**                     | Productivity-focused, data-engineering toolset                          | —                               | Data-engineering / analysis agents                     |
| **SuperAGI**                    | GUI for building, running, monitoring agents                            | —                               | Users who want UI-driven agent<br />ops                |

### Why use/not use agent frameworks


|                           | Framework (LangChain etc.)                    | Roll Your Own                      |
| --------------------------- | ----------------------------------------------- | ------------------------------------ |
| **Speed**                 | Fast to prototype                             | Slower to build                    |
| **Control**               | Limited — you work within their abstractions | Full — you define every behaviour |
| **Transparency**          | Can be a "black box" — hard to debug         | You know exactly what's happening  |
| **Flexibility**           | Harder to deviate from intended patterns      | Unlimited customisation            |
| **Overhead**              | Heavy dependency, frequent breaking changes   | Only what you need                 |
| **Cost/Token efficiency** | Often adds hidden prompt overhead             | You control every token            |

---

## Agent Protocols

In 2025 two open protocols laid the "TCP/IP" of the Agent Internet:

- **MCP — Model Context Protocol** (Anthropic, late 2024). Standardized "language" between an LLM and external tools / data / services. Replaces bespoke glue code; one tool definition, many agents.
- **A2A — Agent-to-Agent Protocol** (Google, April 2025). First open standard purpose-built for *inter-agent* interoperability — discovery, capability negotiation, message exchange, task coordination. Lets, e.g., Company A's recruiting agent securely coordinate interview times with Company B's calendar agent.

Together MCP + A2A move the ecosystem from "wild-growth silos" to a true Internet of Agents.

---

## Agent Development Approaches

Different layers of abstraction trade flexibility for time-to-market ([reference](https://mp.weixin.qq.com/s/x3eigelOnJJi-F2qzBxihg?scene=2)):

1. **HTTP API** — directly call the model provider's REST endpoint. Maximum control, maximum boilerplate.
2. **Python SDK** (e.g. `openai`, `anthropic`) — typed bindings around the API.
3. **AI Framework** (LangChain, LangGraph, AutoGen, CrewAI, …) — opinionated orchestration, RAG, memory, tool use.
4. **Low-code platform** (Dify, Coze, n8n, FastGPT) — drag-and-drop workflows, fastest path to a working bot.
5. **AI Agent SDK** (e.g. Cursor SDK) — pre-packaged agents you embed into a host product.

> The chosen layer **constrains where the agent can be deployed and where it can run** (作用域).

---

## Agent Platforms (Low-Code)

For non-specialists, low-code platforms package the entire LLMOps stack:

- **Dify** — open-source LLMOps platform: dataset / model / app layers, visual workflow editor, RAG engine, Docker / K8s deploy scripts for on-prem. "Bucket-shaped" — strong across the board.
- **FastGPT** — focused on enterprise knowledge bases; best-in-class RAG pipeline (hybrid retrieval, re-ranking, full-link tracing). Narrower than Dify but deeper on Q&A.
- **Coze (扣子)** — ByteDance's low/no-code Bot factory. Cloud-hosted, plugin marketplace, one-click publish to Doubao / Feishu / WeChat. Lowest barrier but limited custom flexibility and on-prem support.
- **Cloud MaaS platforms** — Alibaba (Bailian), Tencent (智能体平台), Baidu (文心智能体平台 / AI Studio), plus AWS / Azure / Google equivalents. Tight integration with the vendor's models and cloud services; the trade-off is vendor lock-in.

### Selection cheat sheet


| Question                                              | Pick                                       |
| ------------------------------------------------------- | -------------------------------------------- |
| Who builds it? Senior Python engineer                 | LangChain / LangGraph                      |
| Who builds it? Enterprise full-stack team             | Dify                                       |
| Who builds it? PM / ops / no code background          | Coze                                       |
| What problem? High-precision document Q&A             | FastGPT                                    |
| What problem? Well-defined business workflow          | CrewAI                                     |
| What problem? Multi-expert collaboration on open task | AutoGen                                    |
| Where does it deploy? Must be on-prem                 | Dify / FastGPT / self-hosted OSS framework |
| Where does it deploy? Don't care, just ship           | Coze / Dify Cloud / cloud-vendor platform  |

---

## Agent Deployment Challenges

A checklist of what you must address before an agent goes live:

1. **Runnability** — does it actually start and stay up?
2. **User intent capture** — clear contract for what the agent should and should not attempt.
3. **Security policy practices** — least privilege, sandbox configs, audit logs.
4. **Cost** — both economic (tokens, infra) and time (latency).
5. **Maintenance** — model upgrades, prompt drift, tool schema changes.
6. **Runtime security** — defense against prompt injection, tool-call abuse, sandbox escape.
7. **Vendor lock-in** — model, framework, and platform choices all bind you.

---

## In-Business Agent Products — Application Landscape

2025 is the year AI Agent commercial value materialized. Highlights by sector:

### Finance — "the disruptor of intelligent transformation"

- 智能信贷审批 (Smart credit approval) — cross-verifies credit bureau, identity, social and consumption data; cuts approval from hours to minutes.
- 欺诈检测 + 精算模型优化 — one big bank lifted fraud detection accuracy +20%, cut false positives −15%.
- 反洗钱 (AML) 监控 — real-time transaction-graph analysis (analogous to log / observability monitoring).
- 智能投顾 + 智能保单检视 — "personal wealth manager" for the long tail; doubles as the customer-service channel.

### Industry & Manufacturing — "from automation to autonomy"

- 视觉质量检测 Agent (AQI) — finds known and unseen defects; loops back upstream to suggest process tweaks.
- 产线动态调度 — re-plans production when a sensor predicts equipment failure.
- 产品设计 Agent — generative design driving CAE simulations.
- 设备预测性维护 Agent.
- 智能采购 + 智慧物流 Agent.
- ZTE's "Nebula" comms agent: −83% O&M headcount, 5× efficiency. Siemens targets +50% production efficiency in lighthouse factories.

### E-commerce & Customer Service

- 智能客服 — handles 93% of issues without human help (RepAI data); −70% labor cost; complaint-handling +50–150%.
- 运营助手 — analyses listing performance and rewrites titles, images, coupons against a KPI.
- 采购代理 — Alibaba "Accio," Amazon "Project Amelia."
- 直播中控 — real-time barrage analysis to drive host prompts and coupon timing.

### Other emerging sectors

- **Education** — "AI 教师" (teaching aid: lesson prep, grading) + "AI 学伴" (1:1 tutoring, personalized paths). Beijing's "京娃" initiative.
- **Government (政务)** — "数字公务员": proactive policy push, "one-thing one-trip" coordinated cross-department service. National strategy now formally includes "Agent-as-a-Service."
- **Health** — AI 影像诊断 Agent (CT/MRI reading), 新药研发 Agent (cuts early R&D 75–90%), 慢病管理 Agent.

### ROI snapshot


| Sector           | Metric                   | Effect                           |
| ------------------ | -------------------------- | ---------------------------------- |
| Industry         | O&M headcount            | −83%                            |
| Industry         | Production efficiency    | Up to +50%                       |
| Customer service | Unassisted resolution    | 93%                              |
| Customer service | Labor cost               | −70%                            |
| Finance          | Fraud detection accuracy | +20% (top-bank interception 70%) |
| Healthcare       | Early drug R&D time      | −75–90%                        |

PwC's 2025 report: of enterprises that have deployed AI Agents, **66% report productivity gains, 57% cost savings, 57% faster decisions**.

---

## Challenges with AI Agents

Cross-cutting design problems that show up regardless of vertical:

- **Action boundary** — where should agent autonomy stop?
- **Decision reliability** — can we trust the output to be correct often enough?
- **Potential misuse** — autonomous tools amplify harm.
- **Responsibility** — when something breaks, who is accountable?

---

### Technical Security Risks

Per the 360 Vulnerability Research Institute + Tsinghua *Agent Security Practice Report* (July 2025), audits of mainstream OSS agent projects uncovered **20+ CVEs** in a single sweep. Recurring patterns:

- **SSRF (Server-Side Request Forgery)** — frameworks default-bind to `0.0.0.0` for dev convenience, exposing internal services and enabling lateral movement (e.g. LangChain-Chatchat arbitrary file read/write CVE-2025-6853 / 6854 / 6855).
- **Remote Code Execution (RCE)** — unsanitized input flowing into code generation or template engines (PySpur Jinja2 injection CVE-2025-6518; Upsonic CVE-2025-6278).
- **Untrusted LLM output** — most agent systems unconditionally trust LLM output, so a **jailbreaking prompt** can induce dangerous function calls, false instructions, or malicious code injection.
- **MCP poisoning** — malicious tools uploaded to public MCP registries; SSE-mode remote MCP services can broadcast malicious instructions to many agents at once ("cross-agent poisoning").
- **A2A auth gaps** — the OSS A2A implementation ships *without* concrete authentication; if developers don't add it, attackers can impersonate trusted agents ("shadow attack") or poison context between them.
- **Sandbox escape** + **differentiated sandbox** challenge — one-size sandbox configs are either too permissive (insecure) or too strict (breaks the agent). No mature dynamic per-task sandbox-permission solution exists yet.

#### Sample CVE table


| Target             | Vulnerability              | CVE               |
| -------------------- | ---------------------------- | ------------------- |
| LangChain-Chatchat | Arbitrary file read/write  | CVE-2025-6853/4/5 |
| DB-GPT             | Parameter validation error | CVE-2025-6772     |
| SuperAGI           | Arbitrary file write       | CVE-2025-6280     |
| Upsonic            | RCE                        | CVE-2025-6278     |
| PySpur             | RCE (template injection)   | CVE-2025-6518     |
| OpenAgents         | Arbitrary file write       | CVE-2025-6282     |
| Python-a2a         | Parameter validation error | CVE-2025-6167     |

#### Mitigations

- **Framework** — bind to `127.0.0.1` by default; enforce auth; sanitize all input, especially template / code paths.
- **Ecosystem** — review LLM output before action; vet MCP marketplaces; enforce mutual auth and RBAC on A2A.
- **Sandbox** — keep sandbox tech patched; design **dynamic, least-privilege** policies so each task gets only the permissions it strictly needs.

---

### Ethics, Bias, and Social Risks

- **Invisible training-data bias** — gender, race, region, religion, etc. An AI hiring agent trained on male-skewed history will silently prefer male candidates. Agent autonomy doesn't just *expose* such bias, it *executes* it at scale.
- **AI hallucination** — the agent's "Achilles heel." In long decision chains a tiny hallucination cascades; an autonomous trading agent acting on a fabricated earnings number can lose serious money. Fact-checking and consistency-checking before high-impact actions are open problems.
- **Macro social risks** —
  - **Employment disruption** — unlike past automation that mostly hit physical / repetitive work, agents reach white-collar cognitive work (analysis, research, coding, support).
  - **Environmental cost** — training and running giant LLMs is energy- and carbon-intensive.
  - **Erosion of social trust** — agents are force-multipliers for fake news, deepfakes, and coordinated cognitive attacks on social media.

---

### Data and Privacy Security

Agent autonomy depends on broad, persistent access to data — making each agent a potential "data black hole." Specific risks:

- **Over-collection / misuse** — the agent collects far more than the task strictly needs.
- **Accidental leakage** — agent A inadvertently exposes user X's data while answering user Y.
- **Attacker exfiltration** — once the agent is breached, all its accessible data is breached.
- **Identity re-association** — by combining fragmented signals, the agent can re-identify "anonymous" data.

#### Why users feel out of control

The *Agent Survey* report: **over half** of users don't know what data permissions they granted an agent, or how that data is used. The agent's UI typically takes a high-level goal ("plan next week's marketing") and proceeds opaquely — which files, which APIs, which other agents — none of it is shown.

#### Mitigation stack — from tech to governance

- **Privacy-Enhancing Technologies (PETs)** — Federated Learning, Differential Privacy, Homomorphic Encryption.
- **Clear, auditable data-governance framework** — data classification, per-class access rights, immutable audit logs.
- **User-centric data-control UI** — a "privacy dashboard" showing what data the agent is accessing or plans to access, with **real-time grant / reject / revoke** controls. High-risk requests must re-confirm with the user.
- **Compliance** — *Personal Information Protection Law (PIPL)* etc.; honor *informed consent* and *minimum necessary*.

---

### Law and Regulation

- **EU AI Act** (fully effective Aug 1, 2025) — world's first comprehensive, binding AI law. Risk-tiered approach:
  - **Unacceptable risk** — banned (e.g. subliminal manipulation, social scoring).
  - **High risk** — critical infrastructure, education, hiring, credit, judicial, etc. Must satisfy lifecycle obligations: risk management, data governance, technical docs, transparency, human oversight, cybersecurity.
  - **Limited risk** — transparency obligations only (must disclose AI interaction).
  - **Minimal risk** — most apps; unregulated.
  - Penalties run up to tens of millions of euros or a global-revenue percentage.
- **China — agile governance**
  - State Council *Opinions on Deepening Implementation of "AI+" Action* (Aug 2025).
  - *AI Safety Governance Framework v2.0* (CAC, Sept 2025) — introduces concepts like **circuit-breaker mechanism** and **one-click control** specifically for high-autonomy systems; emphasizes agent **controllability**.
  - As of Sept 2025, **31 provinces** have AI-related policies covering data, algorithmic safety, and ethics.
- **Compliance challenges for enterprises**
  - Tracking divergent jurisdictional rules.
  - Translating compliance into technical interfaces (e.g. EU AI Act's "human oversight" → standard interrupt / abort APIs in the agent runtime).
  - Balancing compliance cost against innovation speed — especially painful for startups.

> Going forward, **traceability of responsibility** (责任可追溯性) is becoming the cornerstone of "responsible AI." Without clear accountability, victims have no recourse and social acceptance stalls.

---

## Thinking — Personal Takeaways

1. **The development method bounds the deployment surface (作用域).** Choosing HTTP / SDK / framework / low-code / agent-SDK doesn't just affect velocity — it determines where the resulting agent can run, what infra it needs, and what compliance posture it can achieve. Pick deliberately, not by familiarity.
2. **"What" is the right first question.** Before reaching for frameworks, ask what problem the agent is solving — that decides whether you need a single-agent RAG bot, a MAS team, an Embodied AI, or just a workflow script. The framework follows from the answer, not the other way around.
3. **Security, governance, and observability are not bolt-ons.** Given the 2025 CVE landscape and the regulatory tide, an agent that ships without least-privilege sandboxing, auditable data flow, and a runtime monitor is not production-ready — it is a future incident.

## Research Areas and Open Needs

A personal radar of research / engineering gaps worth investing in:

- **Voice stack for agents** — ASR / NLU / TTS so voice becomes a first-class agent input.
- **MAS frameworks + benchmarks** — both the systems and the evaluation harnesses.
- **Agent memory management** — make memory efficiency and effectiveness for short-term and long-term.
- **Local / on-prem deployment** of both models and the agent framework.
-	**Evaluate Cloud MaaS platforms/Claude managed platform** on security.
- **Post-deployment maintenance** — versioning, prompt drift, regression detection.
- **Observability** — monitoring, debugging, and evaluating live agent actions.
- **Edge deployment** for privacy-sensitive scenarios (model + agent framework).
- **Agent risk and governance** —
  - Technical security across framework / LLM output audit / MCP-A2A audit / sandbox escape / differentiated sandboxes / AI hallucination.
  - Law-compliance scanning baked into the framework (agent framework that *adheres to law X*).
- **Data & privacy security in AI Agents** —
  - Lift the "black box" of agent data collection — what data is collected, what it is used for. A clear execution flow with user-data paths highlighted.
  - User-centric UI: visible data flow + real-time grant / reject / redo / confirm controls.
  - Auditable data governance: data levels, granted level, granted permissions, immutable logs.
- **责任可追溯性** — end-to-end accountability traces.
- **Domain-Specific Language Models (DSLM)** — e.g. IaC-Model: high accuracy, low cost, better compliance.
- **Embodied AI** — bridging digital and physical.
- **AI Secure Platform** — a full runtime safety / guardrail layer for agents.
- **Agent runtime monitoring tools.**
- **Easy fine-tuning for domain experts** to inject expertise into their own DSLM.
---
