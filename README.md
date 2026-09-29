# 🔍 CrewAI with Real Web Search — Two Specialist Agents, One Sequential Pipeline (Agentic AI #5)

The direct sequel to an earlier CrewAI project: the `SERPER_API_KEY` collected but never used there is now **actually wired into a live web search tool**. Two genuinely different specialist agents — a Market Researcher who browses the internet and a Product Strategist who never touches a search tool at all — collaborate on a real market-research pipeline, with CrewAI automatically passing the researcher's findings into the strategist's task as context.

![Python](https://img.shields.io/badge/Python-3.9%2B-blue?logo=python&logoColor=white)
![CrewAI](https://img.shields.io/badge/CrewAI-Multi--Agent%20Framework-FF6F00)
![Serper](https://img.shields.io/badge/Serper-Live%20Web%20Search-4285F4)
![Agentic AI](https://img.shields.io/badge/Series-Agentic%20AI%20%2305-8A2BE2)

---

## 📌 Overview

The earlier CrewAI notebook in this series used one agent for two tasks, and set up (but never used) a Serper web-search key. This project closes both gaps at once: **two distinct agents with different tools and expertise**, one of which genuinely searches the live web, feeding a real, current-market pipeline rather than working from the LLM's static training knowledge alone.

---

## 🏗️ The Complete Pipeline

```
product_name = "energy drink"
        │
        ▼
market_researcher (Agent, equipped with SerperDevTool — REAL web search)
        │
        ▼
gather_market_insights_task ──► browses the internet, returns current market trends
        │
        │   (CrewAI automatically passes this task's output as context
        │    into the next task, since no explicit "context" wiring is needed
        │    under the default sequential process)
        ▼
strategiest (Agent, no search tool — reasons from the researcher's findings)
        │
        ▼
develop_positioning_strategy_task ──► positioning strategy, target audience, impact
        │
        ▼
Crew(agents=[...], tasks=[...], planning=True)
        │
        ▼
await crew.kickoff_async()
```

---

## 🔬 Part 1 — Two Agents, Deliberately Different Capabilities

```python
market_researcher = Agent(
    role="Market Researcher",
    goal="Analyse market trends for the product launch",
    backstory="Experienced in marketing trends and consumer behavior analysis",
    tools=[SerperDevTool(api_key=SERPER_API_KEY)],   # ← genuinely equipped with live search
    verbose=True,
)

strategiest = Agent(
    role="Product Strategist",
    goal="Create effective positioning strategies for the product",
    backstory=f"skilled in competitive positioning {strategist_backstory}",
    verbose=True,   # ← notice: no `tools` argument at all
)
```

This is the notebook's clearest design idea: **not every agent needs the same capabilities.** The researcher is given `SerperDevTool` — a real internet search tool from `crewai_tools` — while the strategist has none, and is expected to reason purely from what the researcher hands it. Tool assignment is per-agent, matching real specialisation rather than giving every agent the same generic toolbox.

---

## 🔬 Part 2 — Tasks That Implicitly Depend on Each Other

```python
gather_market_insights_task = Task(
    description=f"Browse the internet to gather insights on current market trends for the "
                 f"launch of the {product_name} product.",
    expected_output=f"List of relevant market trends and consumer preferences, relevant to {product_name}",
    agent=market_researcher,
)

develop_positioning_strategy_task = Task(
    description=f"Based on the market insights, create a positioning strategy for the "
                 f"{product_name} product, including analysis for the impact and target audience",
    expected_output="A positioning strategy with target audience and impact notes",
    agent=strategiest,
)
```

Notice the second task's description literally says *"Based on the market insights"* — under CrewAI's **default sequential process**, each task automatically receives the prior task's output as context, without any explicit context-passing code. This is a meaningfully different mental model from AutoGen's approach earlier in this series, where passing information between agents means explicitly managing conversation history — here, sequencing tasks *is* the data flow.

---

## 🔬 Part 3 — Assembling and Running the Crew

```python
crew = Crew(agents=[market_researcher, strategiest],
            tasks=[gather_market_insights_task, develop_positioning_strategy_task],
            planning=True)

output = await crew.kickoff_async()
```
Same `planning=True` and async execution pattern introduced in the earlier CrewAI notebook, now applied to a genuinely two-agent, tool-using pipeline instead of a single agent handling both tasks itself.

---

## ⚠️ A Bug Worth Knowing About

```python
crew = Crew(Agents = [market_researcher, strategiest], ...)
```

CrewAI's `Crew` constructor expects a lowercase **`agents`** keyword argument, not `Agents`. Python keyword arguments are case-sensitive, so as written, this line will raise a `TypeError` rather than silently misbehaving — an easy fix (`agents=`, lowercase), but a genuine reminder that a single capitalisation slip is enough to break agent orchestration code that otherwise reads correctly at a glance.

---

## 🗂️ Repository Structure

```
crewai-web-search-market-research-crew/
├── AgenticAI_05_CrewAI_02.ipynb   # Main notebook
├── requirements.txt                 # Dependencies
├── .gitignore                       # Keeps secrets out of git
├── .env.example                     # Template for required environment variables
└── README.md                        # This documentation
```

---

## 🚀 Getting Started

### Prerequisites
- Python 3.9+
- An [OpenAI API key](https://platform.openai.com/api-keys)
- A [Serper API key](https://serper.dev) (free tier available) — **actually used** by `market_researcher` in this notebook

### Installation

```bash
git clone https://github.com/Kailaswadje/crewai-web-search-market-research-crew.git
cd crewai-web-search-market-research-crew

pip install -r requirements.txt

jupyter notebook AgenticAI_05_CrewAI_02.ipynb
```

> ⚠️ Fix the `Agents=` → `agents=` typo in the `Crew(...)` call before running (see above). The notebook already uses `getpass()` for both API keys — good practice, keep it. Clear notebook outputs before pushing.

---

## 🧠 Key Takeaways

- **Not every agent in a crew needs the same tools** — assigning `SerperDevTool` only to the researcher, not the strategist, models real division of labour rather than giving every agent identical capabilities
- **CrewAI's default sequential process passes context automatically** — a later task's description can simply say "based on the [prior] insights," with no explicit conversation-history management required, a real contrast to AutoGen's more conversational context-passing
- **Real web search changes what an agent can know** — the researcher's findings reflect current information the LLM's training data can't provide on its own, a meaningful capability upgrade from Part 4's static, hardcoded example
- **Keyword-argument case sensitivity is a real failure mode in agent code** — `Agents=` vs `agents=` is exactly the kind of small, easy-to-miss error that a quick test run catches immediately but a read-through might not
- This tool-equipped, sequential-context pipeline is a direct step toward the kind of specialised, information-gathering agent roles my dissertation's agentic intelligence platform relies on

---

## 📚 Agentic AI Series Context

| Part | Project | Framework | Focus |
|---|---|---|---|
| 01 | [AutoGen Agent Fundamentals](https://github.com/Kailaswadje/agentic-ai-autogen-introduction) | AutoGen | `ConversableAgent`, peer-to-peer negotiation |
| 02 | [UserProxyAgent & Sequential Chat](https://github.com/Kailaswadje/agentic-ai-userproxyagent-sequential-chat) | AutoGen | Human-facing coordination, pipeline handoffs |
| 03 | [Group Chat, State Flow & Nested Chat](https://github.com/Kailaswadje/agentic-ai-group-chat-state-flow-nested-chat) | AutoGen | Multi-agent teams, deterministic orchestration |
| 04 | [CrewAI Fundamentals](https://github.com/Kailaswadje/crewai-fundamentals-recipe-crew) | CrewAI | Structured agents, task/agent separation, planning |
| **05 (this repo)** | CrewAI with Real Web Search | CrewAI | Tool-equipped agents, automatic sequential context |

---

## 🔮 Possible Extensions

- [ ] Add a third agent (e.g. a content writer) that consumes the strategist's output, extending the sequential chain further
- [ ] Explicitly set `context=[gather_market_insights_task]` on the second task instead of relying on the default sequential behaviour, for clarity
- [ ] Give the strategist its own tool (e.g. a competitor-pricing lookup) and compare output quality
- [ ] Parameterise `product_name` as a CLI or notebook-widget input instead of a hardcoded string

---

## 👤 Author

**Kailas Wadje**
MSc Data Science & AI, University of Liverpool

- GitHub: [@Kailaswadje](https://github.com/Kailaswadje)
- LinkedIn: [linkedin.com/in/kwadaje](https://www.linkedin.com/in/kwadaje/)

---

⭐ If seeing a real web-search-equipped agent crew in action was useful, consider giving it a star!
