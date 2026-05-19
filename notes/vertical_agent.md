# Vertical Agent — What Makes One Actually Useful

## Summary

This note summarize my thinking on what makes an AI Agent or a Vertical AI Agent actually useful. Everything we call "agent engineering" — tools, MCP, skills, MAS, scratchpad files like `task_plan.md` — is just context-window management aimed at giving the LLM a stable, low-noise working environment. A *vertical* agent is therefore not valuable because it is "smart"; it is valuable because it lowers the cost of obtaining context, stabilizes state across long tasks, and truly understands a specific domain environment.

---

## Table of Contents

- [First Question — Does the LLM Actually Need This?](#first-question--does-the-llm-actually-need-this)
- [Why MAS — Context Window Isolation](#why-mas--context-window-isolation)
- [Why "Primitive" Scratchpad Files Work So Well](#why-primitive-scratchpad-files-work-so-well)
- [What Decides If a Vertical Agent Is Useful](#what-decides-if-a-vertical-agent-is-useful)
- [Why Claude Code Is Strong — It Knows the Environment](#why-claude-code-is-strong--it-knows-the-environment)
- [Future of Agent Competition — Who Understands the Environment Better](#future-of-agent-competition--who-understands-the-environment-better)
- [Personal Takeaways](#personal-takeaways)

---

## First Question — Does the LLM Actually Need This?

Before adding any feature to an agent, ask whether the LLM needs it at all. A lot of "agent design" turns out to be over-engineering around something the model could already do, or worse, something that pollutes its context.

The clean mental model:

- **The LLM is the only actor that decides anything.** Every plan, branch, and tool choice ultimately comes from the model.
- **Agent engineering/Harness = managing the context window the LLM sees and formulate the actions LLM perform.** The goal is to make non-deterministic LLM output more deterministic and more useful.
- **Tools / MCPs** are how we *constrain* model output into deterministic, side-effecting actions — call this tool, with these arguments.
- **Skills** are how we *compress* context — pull in only the rules / instructions / examples relevant to the current sub-task instead of paying for them on every turn.

If a proposed feature doesn't either constrain output (deterministic action) or compress / clean up context (skills, isolation), it probably isn't pulling its weight.

---

## Why MAS — Context Window Isolation

The most underrated reason for Multi-Agent Systems is not "specialization" or "parallelism" — those are real but secondary. The primary reason is **context isolation**:

- prevent sub-task contexts from contaminating each other.
- keep the model's attention from scattering across unrelated material.
- keep irrelevant information out of the main thread.

---

## Why "Primitive" Scratchpad Files Work So Well

Files like `task_plan.md`, `progress.md`, `findings.md` look like a hack — and they are. But the reason they outperform fancier mechanisms is that they target the right failure mode:

- **They prevent cognitive drift.** When the model "forgets" — because context was compacted, a sub-agent returned, or the conversation got long — it can just re-read the file and re-anchor.
- **They reduce repeated cognitive cost.** Instead of rederiving the plan or rediscovering past findings on every turn, the model reads a stable, curated artifact.

In other words, these files act as **externalized working memory** with stable addressing. Cheap to write, cheap to read, hard to corrupt — which is exactly what a long-running agent needs.

---

## What Decides If a Vertical Agent Is Useful

A vertical agent's value reduces to three questions:

1. **Does it lower the cost of obtaining context?** — Can the model reach the right files, docs, tickets, dashboards without burning turns and tokens to find them?
2. **Does it improve state stability?** — Across a long task, does the agent's understanding of "where we are" stay coherent, or drift?
3. **Does it truly understand the domain environment?** — Conventions, approval flows, permission boundaries, the way *this* team actually works.

A "vertical agent" that just wraps GPT with a domain prompt and a couple of APIs hits none of these. A real one invests in retrieval, scratchpads, and domain rules until the model can operate in that environment as confidently as a seasoned engineer would.

---

## Why Claude Code Is Strong — It Knows the Environment

The takeaway I keep coming back to: Claude Code isn't strong because the underlying model is smarter than you. It is strong because **it knows how to give the model a working environment that fits the task**. Concretely, that environment work includes:

- How to **manage context** (compaction, system prompts, memory).
- How to **isolate task pollution** (subagents, worktrees).
- How to **maintain state** (scratchpad files, todos, plans).
- How to **compress information** (skills, summaries).
- How to **keep the model focused on the goal** (planning, structured outputs, periodic re-anchoring).

The model is the same model anyone else can call. The difference is the harness around it.

These look like "engineering details," but they are all variations on the same idea: **reduce the model's cognitive load**. A subagent isn't really "another worker"; it's a disposable context window that absorbs the noise of a sub-task and returns only the distilled result to the parent.

---

## Future of Agent Competition — Who Understands the Environment Better

If this is right, then the next phase of agent competition isn't "whose model feels more human." It's **whose system understands the environment more deeply**. The candidates that win long-term will be the ones that:

- **Embed deeper into real workflows** — GitLab, Jira, CI/CD, on-call, code review — not just chat windows.
- **Maintain cognitive state more stably** — continuous records of task stages, decisions made, decisions deferred, so a fresh session can resume without losing the thread.
- **Acquire context more cheaply** — auto-generated call chains, key entrypoints, dependency maps so the model doesn't have to rediscover the codebase every time.
- **Organize long-term memory better** — historical bugs, architectural decisions, "why we did X this way" precipitated into something retrievable.
- **Understand organizational rules accurately** — approval chains, permission boundaries, process constraints. Knowing what *cannot* be done autonomously is as important as knowing what *can*.

Whoever delivers these is the one who builds a long-term-valuable vertical agent.

---

## Personal Takeaways

1. **Start every feature proposal with "does the LLM need this?"** If the answer is "it lets the LLM look smarter," that's the wrong reason. The right reasons are: it constrains output, isolates context, or compresses information.
2. **Treat MAS as a context-isolation tool first.** Specialization and parallelism are nice side effects. If a sub-task doesn't have its own dirty context to absorb, it probably doesn't need its own agent.
3. **Externalized working memory beats clever prompting.** A boring `progress.md` that the agent re-reads is more reliable than any prompt trick for fighting long-horizon drift.
4. **The vertical-agent moat is environment knowledge, not model quality.** Anyone can call the same API. The differentiator is how deeply the agent is wired into the team's real workflows, conventions, and constraints.



## References
1. [在通用Agent越来越强的今天，如何设计一个真正有价值的面向特定领域的Agent](https://www.zhihu.com/question/2038425178141676715/answer/2038727147288786573)