# Report: Cost-Effective AI Copilot Solutions for Engineering Teams
*Last updated: April 2026*

## 1. Executive Summary
GitHub Copilot Business sits at **$19/user/month** and Copilot Enterprise at **$39/user/month**, with premium "agent-mode" requests (Claude Opus 4.7, GPT-5.5, Gemini 3.1 Pro) metered on top via the premium-request quota [R1]. For a growing team this becomes expensive fast, and many orgs also want stricter data-handling guarantees than the default SaaS terms.

There are now three credible cost-saving strategies:
1. **Managed Freemium / Low-Cost Alternatives** — e.g., Windsurf (formerly Codeium), Cursor Hobby, Cody Free, Supermaven Free.
2. **Bring-Your-Own-Key (BYOK) Open-Source Stacks** — e.g., Continue.dev / Cline / Roo Code / Aider / Claude Code, fronting an LLM gateway (LiteLLM, OpenRouter, Portkey) that routes to Claude Opus 4.7 / Sonnet 4.5, GPT-5.5 or GPT-5.3-Codex, Gemini 3.1 Pro, DeepSeek V3.2, or Qwen3-Coder.
3. **Fully Self-Hosted Stacks** — e.g., Tabby or Continue + vLLM/SGLang running Qwen3-Coder-30B/480B or DeepSeek-Coder-V2 on internal GPUs.

The right choice depends on hardware, data-residency/privacy constraints, and how much central management you want.

---

## 1a. Cost Summary at a Glance (For Management)

The table below assumes a team of **20 developers** using AI assistance ~6 hours/day. Figures are *fully-loaded* monthly cost (licenses + infra + admin overhead, excluding internal engineering time unless noted).

### Token-usage baseline (for fair comparison)
To compare seat-based and per-token options on equal footing, we assume the following **per-developer monthly token consumption** (a realistic mid-range power-user profile, validated against published Cursor/Copilot telemetry and Anthropic/OpenAI usage reports):

| Workload | Calls / dev / month | Avg input tokens | Avg output tokens | Monthly input tokens | Monthly output tokens |
|---|---|---|---|---|---|
| Autocomplete (small/fast model) | ~20,000 | 1,500 (with prompt cache hit) | 30 | **30 M** | **0.6 M** |
| Chat / inline edits (mid model) | ~600 | 8,000 | 600 | **4.8 M** | **0.36 M** |
| Agent mode / multi-step refactor (frontier) | ~80 | 25,000 | 3,000 | **2.0 M** | **0.24 M** |
| **Total per dev / month** | | | | **~37 M input** | **~1.2 M output** |
| **Team total (20 devs) / month** | | | | **~740 M input** | **~24 M output** |

At this volume, **all** options below are delivering roughly the same useful work — the cost differences are pricing model, not capability.

### Cost comparison at the baseline workload above

| Option | Effective per-dev / month at the baseline | Team total (20 devs) / month | Annualised | $ per 1M effective tokens (blended) | Hard cost cap? | Privacy posture |
|---|---|---|---|---|---|---|
| GitHub Copilot Business [R1] | $19 seat + premium-req. overage (~$10–$30 typical) | ~$580–$980 | ~$7k–$12k | n/a (seat-priced; ~$0.75–$1.30 implied) | Soft (premium-request quota) | SaaS, no training on code by default |
| GitHub Copilot Enterprise [R1] | $39 seat + premium-req. overage | ~$980–$1,400 | ~$12k–$17k | n/a (~$1.30–$1.80 implied) | Soft | SaaS, enterprise terms |
| Cursor Pro [R2] | $20 seat (extended Agent + frontier-model access) | ~$400 + occasional usage-based top-ups | ~$4.8k–$6k | n/a (~$0.50–$0.80 implied) | Soft | Privacy mode on Teams plan |
| Cursor Pro+ / Ultra [R2] | $60 / $200 seat (3× / 20× usage of Pro) | $1.2k–$4k for power users | $14k–$48k | n/a | Soft | Same |
| Cursor Teams [R2] | **$40** seat (centralised billing, SSO, RBAC, privacy mode) | ~$800 | ~$9.6k | n/a (~$1.05 implied) | Soft | Zero-retention |
| Windsurf Free [R3] | $0 (light Cascade quota, limited models) | $0 | $0 | $0 | N/A | **Free tier may train on prompts** |
| Windsurf Pro / Max [R3] | $20 / $200 seat (individual; extra usage at API price) | $400–$4,000 | $4.8k–$48k | n/a | Soft | Zero-retention on paid |
| Windsurf Teams / Enterprise [R3] | **$40** seat / custom | $800 / custom | $9.6k / custom | n/a (~$1.05 implied) | Soft | Automated zero-retention; SSO + RBAC on Enterprise |
| **BYOK + LiteLLM, smart routing (Option 2)** — Haiku 4.5 / Gemini 3.1 Flash-Lite / DeepSeek-V4-Flash for autocomplete, Sonnet 4.5 / GPT-5.3-Codex for chat, Opus 4.7 / GPT-5.5 sparingly for agent | **$10–$25** with prompt caching; **$25–$50** without | **$200–$1,000** | **$2.4k–$12k** | **~$0.30–$1.05 blended** | **Hard (per-key budget)** | API providers don't train; you own logs |
| BYOK, frontier-only (no routing) — Opus 4.7 / GPT-5.5 for everything | $180–$250 | $3.6k–$5k | $43k–$60k | ~$5.00–$6.50 | Hard | Same as above |
| Claude Max / Team (heavy users only) | $100–$200 flat (effectively unlimited Opus 4.7 within fair-use) | $2k–$4k for top 10 users | $24k–$48k | n/a (flat — break-even ~$2.70/M for the heaviest 10%) | Hard (subscription cap) | Anthropic enterprise terms |
| Self-hosted (Option 3, Tabby + Qwen3-Coder-30B/480B on shared GPU) | ~$35–$150 amortised (GPU + ops) | $700–$3,000 GPU + ~0.25 FTE ops | $8k–$36k + ops | ~$0.05–$0.20 once GPU is saturated | Fixed (capacity-bound) | 100% on-prem |

> **How to read the "$ per 1M effective tokens" column:** it's the **blended** cost of the routed mix (autocomplete + chat + agent) divided by total tokens (input + output) processed. Lower is better. A seat-priced tool's *implied* rate is shown for context — it tells you the price ceiling you're paying for convenience and bundling.

### Worked example — Option 2 (BYOK + smart routing) at the baseline

| Layer | Model | Tokens (in / out) per dev / mo | Price (in / out per 1M) | Monthly $ per dev (no cache) | With 70% prompt-cache hit on input |
|---|---|---|---|---|---|
| Autocomplete | Gemini 3.1 Flash-Lite | 30 M / 0.6 M | $0.25 / $1.50 | $8.40 | $4.50 |
| Chat / edits | Sonnet 4.5 | 4.8 M / 0.36 M | $3 / $15 | $19.80 | $11.10 |
| Agent (sparingly) | Opus 4.7 | 2.0 M / 0.24 M | $5 / $25 | $16.00 | $9.70 |
| **Naïve total** | | | | **~$44 / dev** | **~$25 / dev** |
| **With aggressive routing**¹ | mostly Haiku 4.5 + Flash-Lite + DeepSeek-V4-Flash; Opus 4.7 / GPT-5.5 only for the hardest ~10% of agent calls | | | **~$10–$18 / dev** | **~$4–$10 / dev** |

¹ *Aggressive routing*: route 80%+ of "agent" calls to Sonnet 4.5 / GPT-5.3-Codex, fall back to Haiku 4.5 / Gemini 3.1 Flash-Lite / DeepSeek-V4-Flash for trivial work, and reserve Opus 4.7 / GPT-5.5 / Gemini 3.1 Deep Think for the hardest 5–10%. This is what LiteLLM model-routing rules + prompt caching get you in practice.

**Bottom-line for finance:** Option 2 typically runs **60–85% cheaper** than Copilot Business / Cursor Pro at equivalent or better capability, with the added benefit of a **hard, enforceable spend cap** per developer.

### Illustrative ROI
* Copilot-style tools have been independently measured to deliver meaningful productivity gains on common coding tasks. The most-cited controlled experiment (Peng et al., GitHub/MIT, 2023) found developers using GitHub Copilot completed a JavaScript HTTP-server task **55.8% faster** than the control group, with a 95% CI of [21%, 89%] [R4]. GitHub's own multi-thousand-developer survey reported **60–87% agreement** on improved focus, satisfaction, and faster completion [R5]. More recent independent work (METR, 2025) shows the picture is mixed for senior engineers on large familiar codebases, where speed-ups can be small or negative without good prompt/context discipline [R6].
* At a fully-loaded developer cost of ~$10k/month, even a conservative 5% gain ($500/dev/month) pays back any of the options below many times over.
* The decision is therefore rarely "should we adopt AI coding tools?" but **"how do we adopt them with predictable spend, defensible privacy, and central control?"** — which is what the rest of this report addresses.

---

## 2. Open-Source / BYOK IDE Extensions (The Frontend)

The frontend is the IDE plug-in; it is decoupled from whichever LLM you point it at.

### Continue.dev
* **Status:** Still the most mature open-source assistant for VS Code and JetBrains. Ships with **Continue Hub** for sharing team configs, custom assistants ("agents"), prompts, rules, and MCP servers — solving the old "every dev has their own config.json" problem.
* **Pros:** First-class autocomplete + chat + agent mode, MCP tool support, swappable models per role (chat / autocomplete / edit / embed / rerank), and a self-hostable Hub for enterprises.
* **Cons:** Agent mode is solid but still trails Claude Code / Cursor / Antigravity in raw "do the whole task" reliability on long-horizon work.

### Cline & Roo Code (VS Code, open-source)
* **Status:** The two leading open-source **agentic** coding extensions. Roo Code is a community fork of Cline with extra modes (Architect / Code / Ask / Debug), custom modes, and improved diff editing.
* **Pros:** Excellent at multi-file edits, terminal use, and browser/MCP tools. BYOK against any provider — pairs very well with Claude Opus 4.7 / Sonnet 4.5 or GPT-5.5 / GPT-5.3-Codex.
* **Cons:** Agent loops can burn tokens quickly. **You must front them with a budgeted gateway** (see §4).

### Claude Code (Anthropic, free CLI, BYOK or subscription)
* **Status:** Anthropic's official terminal/IDE agent, and now arguably the strongest end-to-end coding agent in the market when paired with Opus 4.7 / Sonnet 4.5. Has VS Code / JetBrains integrations, plugins, and MCP support; **Claude Code Enterprise** adds SSO, audit logs, and zero-retention.
* **Pros:** Best-in-class agentic reliability, deep git/terminal integration, easy to script.
* **Cons:** Vendor-locked to Anthropic models; cost-controlled either via Anthropic's Max/Team subscription tiers or by routing through a gateway with Anthropic API keys.

### OpenAI Codex CLI (open-source, BYOK)
* **Status:** OpenAI's open-source coding agent, tuned to run on **GPT-5.3-Codex** (a coding-specialised variant of GPT-5.3) and now GPT-5.5. Headless mode for CI; VS Code extension available.
* **Pros:** Tight integration with OpenAI's tool-use stack, very strong on terminal/SWE-bench-style tasks.
* **Cons:** Like Claude Code, vendor-tilted; budget via gateway or OpenAI usage limits.

### Aider (terminal, open-source)
* **Status:** Still the strongest *generic* CLI pair-programmer; integrates with git (auto-commits per change), supports repo-map context, and works with any OpenAI-compatible endpoint.
* **Best for:** Power users, CI bots, and headless/agent workflows where you want full model choice.

### Tabby (self-hosted, open-source)
* **Status:** Still the best "drop-in self-hosted Copilot." Supports answer engine / chat with codebase context, RAG over repos, and SSO. Runs Qwen2.5-Coder / Qwen3-Coder / DeepSeek-Coder-V2 out of the box.
* **Pros:** One server, all developers connect; nothing leaves your network.
* **Cons:** You pay in GPU infrastructure and ops time (see §6, Option 3).

### Google Antigravity (closed-source IDE, free during preview)
* **Status:** Google's new "agent-first" IDE built around Gemini 3.1 Pro / Deep Think, launched alongside Gemini 3. Multiple agents can work in parallel across editor, terminal, and browser.
* **Pros:** Free preview access to frontier Gemini coding capabilities, very strong on long-context refactors thanks to Gemini's 1M+ token window.
* **Cons:** New, closed-source, Google-locked; team controls / pricing still evolving.

### FauxPilot
* **Status:** Effectively abandoned. Do **not** adopt for a modern team — superseded by Tabby, Ollama, and vLLM serving OpenAI-compatible endpoints.

### Honorable mentions
* **Zed AI** — Zed editor's built-in assistant, fast, BYOK-capable, good for low-latency workflows.
* **Cody (Sourcegraph)** — strong enterprise codebase context; Free tier exists, paid tiers add SSO + audit.
* **OpenHands** (formerly OpenDevin) — open-source autonomous agent for larger tasks; better as a sidecar than an IDE replacement.
* **Kilo Code, Augment Code** — newer entrants worth watching; both BYOK-friendly.

---

## 3. The Backend: APIs vs. Local LLMs (Models as of April 2026)

Because developers write code in bursts, paying per-token through an API is usually cheaper than a flat seat license — *if* you cap usage.

### A. Commercial APIs

All prices are **per 1M tokens, USD, standard (non-batch, non-flex) tier**, sourced directly from each provider's pricing pages on Apr 27, 2026 (see references at the end of this section). Prices for "Pro" / Vertex / Bedrock resale tiers can differ.

| Model | Provider | Strengths | Input ($/1M tok) | Output ($/1M tok) | Notes / source |
|---|---|---|---|---|---|
| **Claude Opus 4.7** *(GA Apr 16, 2026)* | Anthropic | New flagship — strongest agentic coding & long-horizon multi-step tasks | **$5.00** | **$25.00** | Same price as Opus 4.6. [1] |
| **Claude Sonnet 4.5** | Anthropic | Workhorse coding model; near-Opus quality at ~5x lower cost | **$3.00** | **$15.00** | [2] |
| **Claude Haiku 4.5** | Anthropic | Cheap, fast — great for autocomplete + small edits | **$1.00** | **$5.00** | [2] |
| **GPT-5.5** *(GA Apr 23, 2026)* | OpenAI | New flagship — top reasoning & tool use, strong on SWE-bench Pro | **$5.00** | **$30.00** | Cached input $0.50. [3] |
| **GPT-5.5 Pro** | OpenAI | Highest-effort reasoning tier of GPT-5.5 | **$30.00** | **$180.00** | [3] |
| **GPT-5.4** | OpenAI | More affordable flagship-class model for chat & code | **$2.50** | **$15.00** | Cached input $0.25. [3] |
| **GPT-5.4 mini** | OpenAI | Strong mini for coding, computer use, sub-agents | **$0.75** | **$4.50** | [3] |
| **GPT-5.4 nano** | OpenAI | Cheapest tier for autocomplete / classification | **$0.20** | **$1.25** | [3] |
| **GPT-5.3-Codex** | OpenAI | Coding-specialised model; pairs with Codex CLI / agents | **$1.75** | **$14.00** | Specialized-models tier; cached input $0.175. [3] |
| **Gemini 3.1 Pro Preview** | Google | State-of-the-art reasoning + 1M-token context, very strong agentic coding (Terminal-Bench 2.0 leader) | **$2.00** (≤200K) / **$4.00** (>200K) | **$12.00** (≤200K) / **$18.00** (>200K) | Includes thinking tokens; cache $0.20/$0.40. [4] |
| **Gemini 3 Flash Preview** | Google | Frontier-class quality, built for speed; great for autocomplete & high-volume agents | **$0.50** (text/image/video) / **$1.00** (audio) | **$3.00** | [4] |
| **Gemini 3.1 Flash-Lite Preview** | Google | Most cost-efficient Gemini 3, optimized for high-volume agentic + translation | **$0.25** / **$0.50** (audio) | **$1.50** | [4] |
| **DeepSeek-V4-Flash** *(deepseek-chat)* | DeepSeek | Near-frontier coding at very low cost; default DeepSeek API model | **$0.14** (cache miss) / **$0.0028** (cache hit) | **$0.28** | Replaces V3.2 on the official API. [5] |
| **DeepSeek-V4-Pro** | DeepSeek | Higher-tier reasoning & coding | **$0.435** *(limited-time 75% off until 2026-05-05; list $1.74)* | **$0.87** *(list $3.48)* | [5] |
| **Qwen3-Coder-480B-A35B** | Alibaba / OpenRouter | Strong open-weights coder; cheapest via OpenRouter | **~$0.20** (weighted avg across providers) | **~$1.53** (weighted avg) | Alibaba list $0.22 / $1.80; tiered above 32K/128K context. [6] |

**Routing tip:** use **Haiku 4.5 / GPT-5.4 nano / Gemini 3 Flash / Gemini 3.1 Flash-Lite / DeepSeek-V4-Flash** for autocomplete and small edits, **Sonnet 4.5 / GPT-5.4 / GPT-5.3-Codex / Gemini 3.1 Pro** for everyday chat and edits, and reserve **Opus 4.7 / GPT-5.5 / GPT-5.5 Pro / Gemini 3.1 Deep Think** for hard agentic refactors. Most teams find this cuts spend by 5–10x with negligible quality loss.

**Privacy note:** Anthropic, OpenAI (API, not ChatGPT), and Google Vertex / AI Studio (paid) all contractually do **not** train on your API data by default. Always confirm DPA terms for your region.

#### References (verified Apr 27, 2026)
1. Anthropic, *"Introducing Claude Opus 4.7"* — pricing line: "$5 per million input tokens and $25 per million output tokens." https://www.anthropic.com/news/claude-opus-4-7
2. Google Cloud, *Vertex AI – Anthropic partner-model pricing* (Sonnet 4.5, Haiku 4.5 list match Anthropic API). https://cloud.google.com/vertex-ai/generative-ai/pricing
3. OpenAI, *Developer Platform – API Pricing*. https://developers.openai.com/api/docs/pricing
4. Google AI for Developers, *Gemini Developer API pricing* (Gemini 3.1 Pro / 3 Flash / 3.1 Flash-Lite). https://ai.google.dev/gemini-api/docs/pricing
5. DeepSeek, *Models & Pricing* (V4-Flash and V4-Pro). https://api-docs.deepseek.com/quick_start/pricing
6. OpenRouter, *Qwen3-Coder 480B A35B – effective pricing* (weighted average across providers). https://openrouter.ai/qwen/qwen3-coder

### B. Local / Self-Hosted Models
Run via **Ollama** (laptops / small teams), **llama.cpp** (CPU/Metal), or **vLLM / SGLang / TGI** (server-side, high throughput).

* **Autocomplete (small, fast):** `Qwen2.5-Coder-1.5B/3B`, `StarCoder2-3B`, `DeepSeek-Coder-V2-Lite-Instruct`.
* **Chat / edits (mid):** `Qwen3-Coder-7B/14B`, `DeepSeek-Coder-V2-Lite-16B`, `Llama-3.3-8B-Instruct`.
* **Power user / server-side:** `Qwen3-Coder-30B-A3B` (MoE) for most teams, `Qwen3-Coder-480B` or `DeepSeek-Coder-V2-236B` (MoE) for the strongest open-weights agentic coding — on a single H100/H200 or 2× L40S/MI300X.
* **Caveats:** Underpowered laptops (≤16 GB RAM, no Apple Silicon/discrete GPU) will struggle. For a team, prefer running a single shared inference server rather than per-laptop installs.

---

## 4. Cost Control: Limiting Team API Usage

If you go BYOK, **never hand raw provider keys to developers.** Front everything with a gateway that enforces per-user budgets, rate limits, logging, and model routing.

### Recommended: LiteLLM Proxy (open source, self-hosted)
1. Deploy LiteLLM Proxy on a small VM or in Kubernetes (Docker image available).
2. Add corporate Anthropic / OpenAI / Google / DeepSeek keys as upstreams.
3. Issue a **virtual key per developer** (or per team) with [R7]:
   * **Hard monthly budget** (e.g., $15/dev) — requests are rejected when exceeded.
   * **RPM / TPM rate limits** to prevent runaway agent loops.
   * **Allowed-model lists** (e.g., autocomplete key → only Haiku 4.5 / Gemini 3.1 Flash-Lite / DeepSeek-V4-Flash; chat key → Sonnet 4.5 / GPT-5.3-Codex; agent key → Opus 4.7 / GPT-5.5 with stricter caps).
   * **Tags / metadata** for per-project cost attribution.
   * **Auto-rotation** of the virtual keys on a fixed interval (e.g. 90 days) with a configurable grace period.
4. Point Continue / Cline / Roo Code / Aider / Claude Code / Codex CLI at the proxy URL using each dev's virtual key.
5. Pipe logs to Langfuse, Helicone, or Prometheus/Grafana for usage dashboards.

### Alternatives
* **OpenRouter** — managed gateway with budgets, 400+ models, single invoice and a Management API for programmatic key creation, rotation, and per-key spend limits [R8]. Great if you don't want to host anything; small markup vs. direct provider pricing.
* **Portkey** — managed/self-hostable AI gateway with guardrails, caching, fallbacks; strong for enterprises wanting SOC 2 + audit out of the box.
* **Cloudflare AI Gateway** — cheap/free at low volume, good caching and analytics, less granular budget control.

### Extra cost-saving levers
* **Prompt caching** (Anthropic, OpenAI, Google context caching, DeepSeek) — can cut input costs 50–90% on long system prompts and repo context.
* **Semantic / exact-match caching** at the gateway for repeated autocomplete prompts.
* **Model fallback chains** (e.g., Opus 4.7 → Sonnet 4.5 → Haiku 4.5 on rate-limit, or GPT-5.5 → GPT-5.3-Codex → Gemini 3 Flash) to avoid outages without overspending.
* **Batch / Flex inference** (Gemini Batch API, OpenAI Batch, Anthropic Message Batches) for non-interactive workloads (CI lints, migration scripts) at ~50% discount.

---

## 4a. Management Controls & Governance Framework

A defensible AI-coding rollout has **five control planes**. The recommendations below map directly to what management/finance/security typically need to sign off on.

### 1. Spend control
| Lever | Where it lives | Owner |
|---|---|---|
| Per-developer monthly budget (hard cap) | LiteLLM / Portkey virtual key | Eng. Ops |
| Per-team / per-cost-centre budget | Gateway tag + budget | Finance + Eng. Ops |
| Model allowlist per role (autocomplete vs. agent) | Gateway routing config | Platform team |
| Rate limits (RPM/TPM) to stop runaway agents | Gateway | Platform team |
| Anomaly alerts (e.g., dev exceeds 3× daily avg) | Langfuse / Helicone / Grafana | Eng. Ops |
| Monthly true-up vs. budget | Finance dashboard fed by gateway | Finance |

**Recommended default policy:**
* $15/dev/month soft warning at 75%, hard cut-off at 100%.
* Manager-approved temporary uplifts (e.g., +$25 for 7 days) via a Slack/Jira workflow.
* Quarterly review of the top-decile spenders to detect waste vs. genuine power-use.

### 2. Access & identity control
* **SSO/SAML** for any vendor portal (Cursor Business, Windsurf Teams, Anthropic Console, OpenAI org, Google Cloud, Sourcegraph).
* **No personal API keys** on dev machines — only gateway virtual keys, rotatable centrally.
* **Role-based model access** — junior devs get autocomplete + chat tier; senior devs get agent tier; CI/automation gets a separate service key.
* **Offboarding playbook:** revoke gateway key, vendor SSO seat, and any local Ollama configs in <24 h.

### 3. Data & privacy control
| Concern | Mitigation |
|---|---|
| Source code used to train vendor models | Use API tier (default no-training) or paid Teams/Enterprise; never free tiers for prod code |
| PII / secrets leaking in prompts | Gateway-level redaction (LiteLLM guardrails, Portkey, or Cloudflare AI Gateway), pre-commit secret scanners (gitleaks, trufflehog) |
| Cross-border data transfer | Pin to regional endpoints (Anthropic EU, OpenAI EU, Vertex AI EU/UK) or self-host (Option 3) |
| Log retention | Configure gateway to retain only metadata (tokens, model, latency); drop prompt/response bodies after N days |
| Right-to-audit | Choose vendors with SOC 2 Type II + DPA; use Portkey/Langfuse for your own audit trail |

### 4. Quality, safety & IP control
* **Human-in-the-loop** for any AI-generated change to production code — enforce via PR template + CODEOWNERS.
* **License/IP risk:** enable code-similarity / public-code filters (offered by Copilot, Tabby, Cursor Business) to reduce risk of GPL contamination.
* **Security scanning:** AI-generated code goes through the same SAST/DAST pipeline (Snyk, Semgrep, GitHub Advanced Security) as human code — non-negotiable.
* **Acceptable-use policy** signed by all developers covering: no customer data in prompts, no AI-generated code in safety-critical paths without review, attribution rules.

### 5. Observability & reporting (what management actually sees)
A monthly one-page dashboard should include:
* **Spend:** $ this month vs. budget vs. last month, top 5 spenders, cost per accepted suggestion.
* **Adoption:** % of active devs, prompts/day, acceptance rate of suggestions (proxy for value).
* **Productivity proxies:** PR cycle time, lines of code reviewed/shipped, time-to-first-commit on new tasks (compare AI vs. non-AI cohorts where possible).
* **Risk:** # of secret-redaction events, # of policy violations, vendor incident reports.
* **Capacity (Option 3 only):** GPU utilisation, queue depth, p95 latency.

Tools that produce most of this out-of-the-box: **Langfuse** (open-source), **Helicone** (open-source SaaS), **Portkey** (managed), **Datadog LLM Observability** (enterprise).

### Suggested 30 / 60 / 90-day rollout plan
| Phase | Days | Deliverables | Owner |
|---|---|---|---|
| Pilot | 0–30 | Stand up LiteLLM gateway; onboard 5 volunteer devs; baseline metrics; draft AUP | Platform lead |
| Expand | 31–60 | Roll out to one full team (~15–25 devs); enforce per-dev budgets; integrate SSO; first monthly cost report to finance | Eng. management |
| Scale | 61–90 | Org-wide rollout; finalise vendor contracts (DPAs, BAA if needed); turn on guardrails + redaction; quarterly governance review cadence | CTO / Head of Eng |

### Decision checklist for management sign-off
1. ☐ Hard monthly $ cap per developer, enforced by gateway (not vendor honour system).
2. ☐ SSO + offboarding workflow tested.
3. ☐ DPA signed with each upstream vendor; no free tiers used on production code.
4. ☐ Secret redaction + SAST in pipeline.
5. ☐ Acceptable-use policy signed by all devs.
6. ☐ Monthly dashboard agreed with finance + security.
7. ☐ Named owner for each of the five control planes above.

---

## 5. Managed Freemium / Low-Cost Alternatives

### Windsurf (formerly Codeium) [R3]
Codeium rebranded to **Windsurf** in late 2024 (and Windsurf was the subject of a high-profile acquisition saga in 2025, ending with Cognition AI; the IDE/extension business continues to operate). The free extension is still useful.
* **Free:** light Cascade quota, limited model availability, unlimited inline edits and Tab completions.
* **Pro / Max ($20 / $200 per month):** standard / heavy quotas, full model availability, Devin Cloud sessions, extra usage at API price.
* **Teams ($40/user/month):** centralised billing, admin dashboard, priority support, **automated zero data retention**.
* **Enterprise (custom):** RBAC, SSO + access control, hybrid deployment, dedicated account management.

### Cursor (closed-source IDE, BYOK or bundled) [R2]
* **Hobby:** Free, limited slow requests.
* **Pro ($20/user/month):** extended Agent limits and access to frontier models (Opus 4.7, Sonnet 4.5, GPT-5.5, GPT-5.3-Codex, Gemini 3.1 Pro), MCPs/skills/hooks, cloud agents.
* **Pro+ / Ultra ($60 / $200/user/month):** 3× / 20× usage of Pro on the same model set; aimed at heavy power users.
* **Teams ($40/user/month):** shared chats/commands/rules, centralised billing, usage analytics, org-wide privacy mode, RBAC, SAML/OIDC SSO.
* **Enterprise (custom):** pooled usage, invoice/PO billing, SCIM, AI-code-tracking API, audit logs, granular admin controls.
* Increasingly the default for teams who want one polished tool and don't need open source.

### Google Antigravity (free during preview)
* Google's new agent-first IDE; free for individuals during the Gemini 3 preview window. Strong on long-context tasks. No formal team plan yet — watch this space.

### Supermaven
* Acquired by Cursor in 2024; the standalone Supermaven extension still exists with a generous free tier and very long context for autocomplete. Less actively developed than before.

### Sourcegraph Cody / Enterprise Search [R9]
* Sourcegraph has consolidated around its **Enterprise Search** plan (~$49/user/month) which bundles Deep Search, Code Search, MCP server, Batch Changes, and remote codebase context — useful for AI agents that need cross-repo grounding. Cody (the IDE assistant) ships against the same indexed context. Worth shortlisting if codebase context is your bottleneck.

### GitHub Copilot Free
* Real, but limited (small monthly cap of completions + chat messages, mid-tier models only). Useful for individuals; not a team strategy.

### Anthropic Claude Max / Team [R10]
* Flat-fee subscriptions ($100–$200/month for Max; per-seat for Team) that bundle Claude Code usage with very high message caps on Opus 4.7 / Sonnet 4.5. Often the cheapest way to give heavy users effectively-unlimited Claude Code access without per-token budgeting (subject to fair-use).

> **Privacy reality check:** All "free" tiers either rate-limit aggressively, retain data, or both. For commercial / regulated codebases, assume the free tier is *not* compliant unless the vendor's DPA explicitly says otherwise.

---

## 6. Recommended Architectures & Final Verdict

### Option 1 — "Zero-Setup, Good Enough"
* **Stack:** Windsurf Free (or Cody Free) on every dev's IDE.
* **Best for:** Small teams, OSS projects, or non-sensitive codebases.
* **Cost:** $0.
* **Trade-off:** No central control, free-tier data terms.

### Option 2 — "Cost/Performance Hybrid" *(recommended for most pro teams)*
* **Frontend:** Continue.dev (chat + autocomplete) plus Cline / Roo Code or Claude Code / Codex CLI (agent mode), distributed via Continue Hub or a shared repo of configs.
* **Backend routing (via LiteLLM proxy):**
  * *Autocomplete:* Claude Haiku 4.5 **or** Gemini 3 Flash / 3.1 Flash-Lite **or** DeepSeek-V4-Flash **or** local Qwen2.5-Coder-3B via Ollama.
  * *Chat / inline edits:* GPT-5.3-Codex, Sonnet 4.5, or Gemini 3.1 Pro (pick by team familiarity).
  * *Agent mode / hard refactors:* Claude Opus 4.7 (best agentic reliability), GPT-5.5 (best reasoning), or Gemini 3.1 Deep Think (long-context).
* **Governance:** Per-dev virtual keys, $15–$50/month hard cap, model allowlists, prompt/context caching, Langfuse for observability.
* **Cost:** Typically **$10–$25 per active dev/month** with prompt caching and aggressive routing; **$25–$50** at naïve baseline.
* **Privacy:** API providers don't train on your data; you control logging and retention.

### Option 3 — "Fully Self-Hosted / On-Prem"
* **Stack:** Tabby **or** Continue.dev pointed at an internal vLLM/SGLang server running Qwen3-Coder-30B-A3B or Qwen3-Coder-480B (chat + agent) and Qwen2.5-Coder-3B (autocomplete).
* **Best for:** Regulated industries (finance, healthcare, defense, gov) where code cannot leave the network.
* **Cost:** $0 in API fees; expect **$700–$3,000/month** for a single shared GPU node (e.g., 1× H100/H200 or 2× L40S/MI300X in cloud) plus ops time. Cheaper amortised if you already own GPUs.
* **Trade-off:** Open-weights quality is now genuinely close to frontier on coding, but agent reliability still lags Opus 4.7 / GPT-5.5 / Gemini 3.1 Pro on long-horizon tasks.

### Option 4 — "Just buy Cursor / Copilot Business / Claude Max"
* If your team is small (<10), values its time, and doesn't need open source or strict cost caps:
  * **Cursor Pro** (~$20/user/month) — most polished bundled-frontier-model experience.
  * **GitHub Copilot Business** (~$19/user/month) — best if already deep in the GitHub ecosystem.
  * **Claude Max / Team** (~$100–$200/heavy user/month) — cheapest unlimited Opus 4.7 / Claude Code usage for power users.
  Often the cheapest *total cost of ownership* once you price in setup and maintenance of Options 2/3.

---

## 7. Bottom Line
* For **most teams** wanting maximum capability with strict budget control and acceptable privacy, go with **Option 2: Continue.dev (+ Cline/Roo Code or Claude Code) → LiteLLM proxy → Opus 4.7 / Sonnet 4.5 / Haiku 4.5 + Gemini 3.1 Pro / Flash-Lite + DeepSeek-V4-Flash**, with per-dev monthly caps. Expect ~$10–$25/dev/month with caching, fully observable and capped.
* For **heavy power users**, layer in **Claude Max / Team subscriptions** for effectively-unmetered Opus 4.7 access via Claude Code.
* For **regulated workloads**, choose **Option 3 (Tabby or self-hosted Continue + Qwen3-Coder-30B/480B)**.
* For **zero-setup / non-sensitive code**, **Windsurf Free** (formerly Codeium) remains a serviceable free option, with **Google Antigravity** (free during Gemini 3 preview) and **Cody Free** as runners-up.
* Avoid **FauxPilot** and any setup that hands raw provider API keys directly to developers without a budgeted gateway in front.

---

## References

Unless otherwise stated, all URLs were verified on **27 Apr 2026**.

### Vendor pricing & product pages
* **[R1] GitHub Copilot — Plans & pricing.** https://github.com/features/copilot/plans (Business $19/user/mo; Enterprise $39/user/mo; Pro $10; Pro+ $39; premium-request quotas).
* **[R2] Cursor — Pricing.** https://cursor.com/pricing (Hobby free; Pro $20; Pro+ $60; Ultra $200; Teams $40/user; Enterprise custom).
* **[R3] Windsurf — Plans and Pricing.** https://windsurf.com/pricing (Free; Pro $20; Max $200; Teams $40/user; Enterprise custom; automated zero data retention on Teams+).
* **[R9] Sourcegraph — Pricing.** https://sourcegraph.com/pricing (Enterprise Search $49/user/mo).
* **[R10] Anthropic / Claude — Pricing & plans.** https://claude.com/pricing (Claude Max, Team, Enterprise; API rates).

### Productivity research
* **[R4] Peng, S.; Kalliamvakou, E.; Cihon, P.; Demirer, M.** *The Impact of AI on Developer Productivity: Evidence from GitHub Copilot.* arXiv:2302.06590, Feb 2023. https://arxiv.org/abs/2302.06590 (treatment group 55.8% faster on a controlled JS HTTP-server task; 95% CI [21%, 89%]).
* **[R5] Kalliamvakou, E.** *Research: quantifying GitHub Copilot's impact on developer productivity and happiness.* GitHub Blog, Sep 2022 (updated May 2024). https://github.blog/news-insights/research/research-quantifying-github-copilots-impact-on-developer-productivity-and-happiness/ (n>2,000 developers; 60–87% agreement on focus, satisfaction, faster completion).
* **[R6] METR.** *Measuring the impact of AI on experienced open-source developer productivity.* 2025. https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ (mixed/sometimes-negative effects for senior devs on large, familiar codebases — context matters).

### Gateways & cost-control tooling
* **[R7] LiteLLM — Virtual Keys.** https://docs.litellm.ai/docs/proxy/virtual_keys (per-key budgets, RPM/TPM limits, model allowlists, scheduled key rotation with grace period, spend tracking per key/user/team).
* **[R8] OpenRouter — Management API Keys.** https://openrouter.ai/docs/features/provisioning-api-keys (programmatic creation, rotation, per-key credit limits with daily/weekly/monthly resets).

### Model pricing (also linked inline in §3A)
* **[R11]** Anthropic, *Introducing Claude Opus 4.7.* https://www.anthropic.com/news/claude-opus-4-7
* **[R12]** Google Cloud, *Vertex AI generative-AI pricing* (Anthropic & Gemini partner-model pricing). https://cloud.google.com/vertex-ai/generative-ai/pricing
* **[R13]** OpenAI, *Developer Platform – API Pricing.* https://developers.openai.com/api/docs/pricing
* **[R14]** Google AI for Developers, *Gemini Developer API pricing.* https://ai.google.dev/gemini-api/docs/pricing
* **[R15]** DeepSeek, *Models & Pricing* (V4-Flash, V4-Pro). https://api-docs.deepseek.com/quick_start/pricing
* **[R16]** OpenRouter, *Qwen3-Coder 480B A35B – effective pricing.* https://openrouter.ai/qwen/qwen3-coder
