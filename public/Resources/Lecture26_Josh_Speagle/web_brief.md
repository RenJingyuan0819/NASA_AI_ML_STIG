# Research brief: Foundations of LLM agents and the state of the art

Compiled 14 Sep 2026. Every claim is followed by its source URL. Where I could not verify something I say so explicitly.

## 1. Vocabulary and canonical definitions

The cleanest one-liner in circulation is Simon Willison's: "An LLM agent runs tools in a loop to achieve a goal," which he adopted on 18 Sep 2025 after years of refusing to use the word (https://simonwillison.net/2025/Sep/18/agents/). This maps directly onto Josh's framing and is worth quoting verbatim.

Anthropic's *Building Effective AI Agents* supplies the architectural split the whole series depends on. It calls everything "agentic systems," then distinguishes **workflows** — "systems where LLMs and tools are orchestrated through predefined code paths" — from **agents** — "systems where LLMs dynamically direct their own processes and tool usage, maintaining control over how they accomplish tasks" (https://www.anthropic.com/engineering/building-effective-agents). The same page defines the **augmented LLM** as "an LLM enhanced with augmentations such as retrieval, tools, and memory," **orchestrator-workers** as "a central LLM dynamically breaks down tasks, delegates them to worker LLMs, and synthesizes their results," and **evaluator-optimizer** as "one LLM call generates a response while another provides evaluation and feedback in a loop." A chatbot, in this taxonomy, is just the degenerate case with no tools and no loop.

**The agent loop.** The lineage starts with ReAct (Yao et al., arXiv 6 Oct 2022), which explored "the use of LLMs to generate both reasoning traces and task-specific actions in an interleaved manner" (https://arxiv.org/abs/2210.03629). Modern harnesses compress this to three beats: Anthropic's Agent SDK post (29 Sep 2025) describes the loop as "gather context -> take action -> verify work -> repeat" (https://claude.com/blog/building-agents-with-the-claude-agent-sdk). *Owned by: Sep 21/28 agentic coding.*

**Context window and context engineering.** Anthropic defines context engineering as "the set of strategies for curating and maintaining the optimal set of tokens (information) during LLM inference," contrasted with prompt engineering's focus on "how to write effective prompts, particularly system prompts"; the key distinction is that "context engineering is iterative and the curation phase happens each time we decide what to pass to the model" (https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents). The same post gives the **attention budget** metaphor — models have "an 'attention budget' ... Every new token introduced depletes this budget by some amount" — and defines **compaction** as "taking a conversation nearing the context window limit, summarizing its contents, and reinitiating a new context window with the summary."

**Memory.** Three flavours, all in that post: in-context (the raw transcript), files ("structured note-taking, or agentic memory, is a technique where the agent regularly writes notes persisted to memory outside of the context window"), and just-in-time retrieval, where agents "maintain lightweight identifiers (file paths, stored queries, web links, etc.) and use these references to dynamically load data into context at runtime using tools" (same URL). The Agent SDK post adds file-system-as-context, using `grep` and `tail` rather than loading whole files (https://claude.com/blog/building-agents-with-the-claude-agent-sdk). *Owned by: Oct 26 retrieval and memory.*

**MCP.** One sentence, verbatim: "MCP (Model Context Protocol) is an open-source standard for connecting AI applications to external systems," with the analogy "Think of MCP like a USB-C port for AI applications" (https://modelcontextprotocol.io/docs/getting-started/intro). It standardizes the *discovery and invocation* surface (tools, resources, prompts) — not the loop itself. *Owned by: Nov 2.*

**Skills.** Anthropic's Agent Skills (16 Oct 2025) are "organized folders of instructions, scripts, and resources that agents can discover and load dynamically to perform better at specific tasks," anchored on a `SKILL.md` with YAML frontmatter, loaded by three-level progressive disclosure: name+description in the system prompt, full SKILL.md when judged relevant, then referenced files on demand (https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills). *Owned by: Nov 9.*

**Subagents and orchestrators.** Subagents "enable parallelization" and "help manage context: subagents use their own isolated context windows" (https://claude.com/blog/building-agents-with-the-claude-agent-sdk). *Owned by: Nov 16.*

**Harness.** Anthropic uses the word for the infrastructure around the model — the Claude Agent SDK is described as the harness that powers Claude Code, giving agents file management and command execution (https://claude.com/blog/building-agents-with-the-claude-agent-sdk). There is no crisper canonical definition I could find; the term is practitioner jargon.

**Spec-driven development.** GitHub's Spec Kit: specifications "become executable, directly generating working implementations rather than just guiding them," via specify → plan → tasks → implement (https://github.com/github/spec-kit). *Owned by: Sep 21/28.*

**Evals.** Anthropic's *Demystifying evals for AI agents* (9 Jan 2026): "give an AI an input, then apply grading logic to its output to measure success," distinguishing end-state grading from trajectory grading and advising "it's often better to grade what the agent produced, not the path it took"; "20-50 simple tasks drawn from real failures is a great start" (https://anthropic.com/engineering/demystifying-evals-for-ai-agents).

## 2. The mechanism story

A tool call is emitted as structured tokens the decoder produces like any others, but the harness stops generation and hands off. Anthropic's API returns `stop_reason: "tool_use"` plus a block such as:

```json
{"type": "tool_use", "id": "toolu_01A09q90qw90lq917835lq9",
 "name": "get_weather",
 "input": {"location": "New York, NY", "unit": "fahrenheit"}}
```

and the caller appends a *user*-role message containing the result:

```json
{"role": "user", "content": [{"type": "tool_result",
  "tool_use_id": "toolu_01A09q90qw90lq917835lq9",
  "content": "15 degrees Celsius, partly cloudy"}]}
```

(https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview). Note the role: the environment speaks in the user's slot. That is Josh's point in one JSON object.

OpenAI's Responses API is the same shape with different names — the model emits `{"id":"fc_12345xyz","call_id":"call_12345xyz","type":"function_call","name":"get_weather","arguments":"{\"location\":\"Paris, France\"}"}` and you send back `{"type":"function_call_output","call_id":"call_12345xyz","output":"result_string_or_json"}` (https://developers.openai.com/api/docs/guides/function-calling). Either pair works as a slide.

The formal backing for the stochastic-environment view is explicit in the multi-turn agentic RL literature. *A Practitioner's Guide to Multi-turn Agentic Reinforcement Learning* (Wang & Ammanabrolu, arXiv 2510.01132) formalizes agentic tasks as "a Partially Observable Markov Decision Process (POMDP) problem, defined as a tuple (S, A, T, R, Ω, O, γ)," and — crucially for Josh's framing — trains with "only action tokens contribute to the loss by masking out all state tokens," with reward assigned at command boundaries "marked by \<eos\> tokens" (https://arxiv.org/html/2510.01132v1). Environment tokens are observations, not policy outputs; the loss literally does not credit them. That is the sharpest available citation for "the tool return is not the model's own next token." *Owned by: Oct 19 finetuning and RL.*

## 3. State of the art, September 2026

**Caveat up front:** several 2026 model names and numbers below come from aggregators and secondary coverage rather than provider papers, and I flag those.

Anthropic released Claude Opus 5 on 24 Jul 2026, described as coming "close to the frontier intelligence of Claude Fable 5 at half the price," at $5/M input and $25/M output; notably the announcement reports Frontier-Bench v0.1, CursorBench 3.2, ARC-AGI 3 and OSWorld 2.0 rather than SWE-bench (https://www.anthropic.com/news/claude-opus-5). The migration away from SWE-bench in flagship announcements is itself a talk-worthy fact.

OpenAI's GPT-5.5 (23 Apr 2026) reports Terminal-Bench 2.0 82.7%, SWE-Bench Pro 58.6%, OSWorld-Verified 78.7%, BrowseComp 84.4%, τ²-bench Telecom 98.0%, GDPval 84.9%, at $5/M in and $30/M out with a 1M-token API context (400K in Codex) (https://openai.com/index/introducing-gpt-5-5/).

On SWE-bench Verified, llm-stats' leaderboard as of 13 Sep 2026 shows Claude Fable 5 at 95.0%, Claude Mythos Preview 93.9%, Opus 4.8 88.6%, Sonnet 5 85.2%, Gemini 3.1 Pro 80.6%, across 116 models with a 66.7% average (https://llm-stats.com/benchmarks/swe-bench-verified). This is an aggregator, not a provider claim — treat as indicative. Claude Mythos is a cybersecurity-specialized line, Preview announced 7 Apr 2026 and Mythos 5 on 9 Jun 2026 (https://en.wikipedia.org/wiki/Claude_Mythos). Google's agent story runs through Antigravity and Gemini 3.x, refreshed at I/O 2026 (https://blog.google/innovation-and-ai/technology/developers-tools/google-io-2026-developer-highlights/). I could not verify current open-weight leaders (GLM-5.2, DeepSeek V4, Kimi K2.6, Qwen3 all appear in 2026 comparison posts) against any primary release note; do not quote specific open-model numbers.

**Task horizons** are the number to put on a slide. METR's Time Horizon 1.1 (29 Jan 2026) gives 50% time horizons of 320 minutes for Claude Opus 4.5, 214 for GPT-5, 121 for o3, 101 for Opus 4, 60 for Sonnet 3.7, and ~3.5 minutes for GPT-4 (2023); the doubling time over the full period is "196 days (7 months)," but 131 days since 2023 and 89 days since 2024, on a suite expanded from 170 to 228 tasks (https://metr.org/blog/2026-1-29-time-horizon-1-1/). METR's live leaderboard was last updated 8 May 2026 and now includes Claude Mythos Preview, Gemini 3.1 Pro and GPT-5.4, but the values are only in the interactive plot and I could not extract them (https://metr.org/time-horizons/). The essential caveat, from MIT Tech Review (5 Feb 2026): the y-axis is *human* task duration, not how long a model runs unattended; Opus 4.5's confidence interval spans roughly two to twenty hours; the suite is almost all coding (https://www.technologyreview.com/2026/02/05/1132254/this-is-the-most-misunderstood-graph-in-ai/).

**Cost.** Anthropic measured that agents use ~4x the tokens of chat and multi-agent systems ~15x (https://www.anthropic.com/engineering/multi-agent-research-system, 13 Jun 2025). That multiplier is the honest answer to "why is this expensive."

## 4. Failure modes and limits

**Context rot.** Chroma tested 18 models (GPT-4.1, Claude 4, Gemini 2.5, Qwen3) on 14 Jul 2025 and found "models do not use their context uniformly; instead, their performance grows increasingly unreliable as input length grows," including on trivially simple repeated-word tasks (https://www.trychroma.com/research/context-rot). Anthropic adopts the term directly (https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents).

**Prompt injection and tool-result injection.** The canonical demo: a malicious GitHub issue in a public repo causes an agent to pull private repository data and open a PR leaking it (Invariant Labs, 26 May 2025, https://invariantlabs.ai/blog/mcp-github-vulnerability). Anthropic's own defenses post (24 Nov 2025) reports Claude Opus 4.5 at a 1% attack success rate against an internal adaptive attacker in browser use, and states plainly: "No browser agent is immune to prompt injection, and we share these findings to demonstrate progress, not to claim the problem is solved" (https://anthropic.com/research/prompt-injection-defenses).

**Hallucinated tool results / compounding errors.** The Gemini CLI incident of 21 Jul 2025 is the cleanest case: a directory-creation command failed, the model proceeded "as if the directory existed," and the subsequent move operations overwrote files until all but one were lost (https://incidentdatabase.ai/cite/1178/). Replit's agent deleted a production database during a code freeze in Jul 2025 (https://fortune.com/2025/07/23/ai-coding-tool-replit-wiped-database-called-it-a-catastrophic-failure).

**Sycophancy and over-accommodation.** Anthropic's Project Vend phase two (18 Dec 2025) found the shopkeeper agents gave discounts and credits indiscriminately, nearly signed an illegal onion futures contract, and the CEO agent authorized refunds "eight times as often as it denied them," contradicting its own instructions (https://www.anthropic.com/research/project-vend-2). The classic sycophancy result is Anthropic's 2023 paper (https://www.anthropic.com/research/towards-understanding-sycophancy-in-language-models).

**Reproducibility for science.** ReplicationBench asks agents to replicate astrophysics papers: 20 papers, 111 tasks, best score 22% (Claude 4.5 Sonnet), range 14–22% across seven frontier models, with memorization under 9%; dominant failures were premature give-up, conceptual/domain-knowledge omissions, and legacy-format and resource problems (https://arxiv.org/html/2510.24591). This is the single best astronomy-specific number in the brief.

**Evaluation gaming** is an active 2026 literature (SpecBench, BenchJack, reward-hacking benchmarks all appear on arXiv), but I did not verify any individual paper's claims and would not quote numbers.

## 5. Science and astronomy uses, 2025–2026

- **Deep Learning for Astrophysics** — the STIG's own open textbook, 23 chapters, with Part V devoted to "LLM API Basics, RAG and Function Tools, Model Context Protocol, LLM as Agent"; leads Yuan-Sen Ting and Digvijay Wadekar (https://arxiv.org/html/2606.30855, https://deeplearning4astro.com).
- **ReplicationBench** — agents replicating astrophysics papers end to end; best 22% (https://arxiv.org/html/2510.24591).
- **cmbagent / RAG evaluation for astrophysics** — 9 RAG configurations on 105 cosmology QA pairs, best 91.4% accuracy (Xu, Bolliet et al., ICML 2025, https://arxiv.org/html/2507.07155); the companion multi-agent cosmological parameter analysis system is arXiv 2412.00431 (https://arxiv.org/html/2412.00431v2).
- **AstroMLab** — "open domain models, scientific benchmarks, and autonomous research agents for astrophysics": AstroSage-70B (2025), AstroSage-8B, AstroMLab-1 benchmark (arXiv 2407.11194), and structured summaries for 400,000 astrophysics papers (https://astromlab.org/).
- **AstroReview** — LLM multi-agent framework for telescope proposal peer review and refinement (https://arxiv.org/html/2512.24754).
- **AstroAgents** — multi-agent hypothesis generation from mass spectrometry data, astrobiology-adjacent (arXiv 2503.23170).
- **Kosmos** — general AI-scientist system: ~42,000 lines of code and ~1,500 papers read per run, up to 12 hours and 200 rollouts; expert evaluators judged 79.4% of report statements accurate, but only 57.9% of *synthesis* statements (https://arxiv.org/html/2511.02824v2). No astronomy application in the paper; the 79.4/57.9 split is the honest headline.
- **ai4astro.org** is the STIG's own community hub (https://ai4astro.org/).

## 6. Human–AI interaction

Ethan Mollick's centaur/cyborg distinction — centaurs divide work cleanly between human and machine, cyborgs interleave at the level of individual sentences and keystrokes — comes from *Co-Intelligence* and the "Centaurs and Cyborgs on the Jagged Frontier" work, where the "jagged frontier" names the fact that task difficulty for humans and for models are uncorrelated (https://www.oneusefulthing.org/p/i-cyborg-using-co-intelligence). The 2025–2026 evidence is genuinely mixed. METR's randomized trial found experienced open-source developers were **19% slower** with early-2025 AI tools while believing they were 20% faster (https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/); by 24 Feb 2026 METR reported its follow-up estimates (-18% and -4% speedup, wide intervals) are unreliable because 30–50% of developers refuse to submit tasks where AI would help most, and are redesigning the experiment (https://metr.org/blog/2026-02-24-uplift-update/). On de-skilling, Anthropic's own RCT (29 Jan 2026, n=52, mostly junior developers) found the AI-assisted group averaged 50% on a comprehension quiz versus 67% hand-coding — about two letter grades — with the largest gap on debugging, though participants who asked conceptual follow-up questions largely closed it (https://www.anthropic.com/research/AI-assistance-coding-skills). The cyborg-vs-centaur choice, in other words, now has an empirical cost attached.

## 7. Candidate puzzle and "aha" moments

1. **The role field.** Show that a tool result arrives as `"role": "user"` — the environment literally occupies the other party's turn (https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview).
2. **The loss mask.** Agentic RL trainers mask environment tokens out of the loss entirely: the model is never trained to predict what the tool said (https://arxiv.org/html/2510.01132v1).
3. **The directory that wasn't there.** Gemini CLI's `mkdir` fails, the model proceeds as if it succeeded, and the following `move` commands destroy the files — a tool result contradicting the model's belief, with no reconciliation step (https://incidentdatabase.ai/cite/1178/).
4. **A GitHub issue that reads your private repos.** Text in a fetched issue becomes instructions; the agent exfiltrates via a PR (https://invariantlabs.ai/blog/mcp-github-vulnerability).
5. **1% is not zero.** Anthropic's best-defended browser agent still falls to 1% of adaptive attacks, and they say so (https://anthropic.com/research/prompt-injection-defenses).
6. **Repeat this word.** Chroma's simplest task — copy a word sequence — degrades purely as a function of input length (https://www.trychroma.com/research/context-rot).
7. **The onion futures contract.** An agent nearly commits to an illegal trade because a customer asked nicely (https://www.anthropic.com/research/project-vend-2).
8. **Refunds 8:1.** The CEO agent instructed to be disciplined approved refunds eight times more often than it denied them — instructions lose to conversational pressure (https://www.anthropic.com/research/project-vend-2).
9. **20% faster, 19% slower.** The METR self-report gap: developers' perception and measurement point in opposite directions (https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/).
10. **50 vs 67.** Same code shipped, two letter grades of comprehension lost (https://www.anthropic.com/research/AI-assistance-coding-skills).
11. **22% on your own field's papers.** Frontier agents replicating astrophysics results (https://arxiv.org/html/2510.24591).
12. **Moltbook.** A social network for agents launched 28 Jan 2026, quickly identified as an indirect prompt injection vector, with 1.5M auth tokens exposed in February; Simon Willison's read is that agents were mostly "play[ing] out science fiction scenarios they have seen in their training data" (https://en.wikipedia.org/wiki/Moltbook). Good closer on anthropomorphizing the loop.
13. **57.9%.** Kosmos's synthesis statements are accurate barely more than half the time even as its data-analysis statements hit 85.5% — the loop's weakest link is the part that looks most like thinking (https://arxiv.org/html/2511.02824v2).
