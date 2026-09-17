# Foundations of LLM agents and the state of the art — talk design (v4)

NASA Cosmic Origins AI/ML STIG, opening talk of the fall 2026 series. Monday 14 September 2026, 4:00 pm ET, Teams. 50 minutes. Speaker: Josh Speagle (Astronomy & Astrophysics · Statistical Sciences, University of Toronto).

v4 (after Josh's review of v3). Text budget enforced on every slide; more signposting; images split into embedding and combination; tool call and variability split; gradient descent shown as figures; harness part opens with the whole scaffold used for this talk, then components with real examples (a tool definition, a skill description, how memory is loaded), then agents as tools, variability, why multi-agent, then the workflow that built the talk; harness table dropped; John Wu's post cited properly; no link to his tutorial; last slide folded into the overview. All on-slide text below is final.

## Text budget (applies to every slide)

Title: one line, at most six words. Body under or beside a figure: at most two sentences, at most 30 words, set in the `lead` role (22 pt) or `body` (20 pt); never `small` for the main statement. One optional source or aside line in `small`/`mono-d`. Cards and chips: heading plus at most 12 words. Statement slides: at most three lines. Everything else goes to the speaker notes. If a slide's text does not fit within this, split the slide; do not shrink the type.

## Stage 1 and 2

Unchanged from v3: four big ideas (how they work; why they work; the harness; working with them), no polls, four cards shown at the start, revisited at each part, filled in at the end.

## Stage 3 · Slides

### Design rules

`_design-system/design/SLIDE_DESIGN.md`; dark for Teams, light exported. Figures drawn with the system's tokens; figure and text say the same thing. Sub-part dividers (navy, Deck.divider) mark topic changes inside a part. Real artifacts as text in chips.

### Time budget

| Part | Min | Slides |
|---|---|---|
| 0 Opening | 3 | 1–3 |
| 1 How they work | 12 | 4–19 |
| 2 Why they work | 8 | 20–28 |
| 3 The harness | 14 | 29–43 |
| 4 Working with them | 8 | 44–51 |
| 5 What is possible today | 3 | 52–53 |
| 6 Close | 1 | 54 |

### The four cards

- **How they work** — Context in, a distribution out, one draw appended. Tools return text into the context. `Part 1 · 12 min`
- **Why they work** — Varied text, no memorizing, a hard objective: general strategies. `Part 2 · 8 min`
- **The harness** — Everything around the model. You build it and you control it. `Part 3 · 14 min`
- **Working with them** — Clear vision, clear communication, clear execution. `Part 4 · 8 min`

"Where we are" slides (4, 20, 29, 44) re-show the grid with the pad beside the current card; other cards' text in ink-3.

### Reference facts for the builder (real, from this session)

- The pptx skill description as the harness shows it to the model (trim to two lines): "Use this skill any time a .pptx or .potx file is involved in any way — as input, output, or both. This includes: creating slide decks … reading, parsing, or extracting text from any .pptx … editing, modifying, or updating existing presentations …"
- A real tool definition (Read): name `Read`; description "Reads a file from the local filesystem"; input schema `file_path: string (absolute)`, `offset: integer`, `limit: integer`. Another (Bash): "Executes a bash command and returns its output"; `command: string`, `timeout: integer (ms)`.
- The Agent tool definition: name `Agent`; description "Launch a new agent to handle complex, multi-step tasks"; input `prompt: string`, `subagent_type: string`, `model: "sonnet" | "opus" | …`; returns the agent's final message as text.
- Memory mechanics in this harness: at session start the harness injects a snapshot (profile, preferences, and a listing of memory files with one-line descriptions); a file is read in full only when the task calls for it (`memory_read`); the model writes with `memory_write`/`memory_append`. Claude Code's equivalent: `MEMORY.md` index loaded every session, capped at 200 lines / 25 KB.
- The scaffold that built this talk: harness Claude Cowork (desktop app + cloud sandbox); lead model Claude Fable 5.1; tools: Read/Write/Edit/Glob/Grep/Bash, WebSearch/WebFetch, device file tools (list/stage/commit), memory tools, Agent, AskUserQuestion, SendUserFile; skills available: pptx, dataviz, docx, … (loaded by description); memory files: profile, preferences, /areas/talks.md; project files: `TALK_DESIGN.md`, `SESSION_ARTIFACTS.md`, `research/*.md` (4 briefs), `_design-system/design/SLIDE_DESIGN.md`, `slides/lib/speagle_slides.py`, `slides/lib/README.md`, `build/build_stig.py`; subagents: 4 research (Opus/Sonnet), 3 builders (Opus); outputs: pptx ×2, PDF, renders. Human checkpoints: 5.
- Agent statistics: research A 2,472 words · 81 tool uses · 7.2 min; B ~1,900 words · 35 · 6.8 min; C 1,484 words · 16 · 2.1 min; D ~1,700 words · 85 · 11.0 min; builder v1 44 slides · 67 · 17.4 min; v2 46 slides · 107 · 24.5 min; v3 46 slides · 94 · 21.4 min.
- John Wu's post: "Clear Vision, Clear Communications", jwuphysics.github.io/blog, 10 Jul 2025, quoting Javier Grillo-Marxuach, *The Eleven Laws of Showrunning*: "When you're a showrunner, it is on you to define the tone, the story, and the characters. You are not a curator of other people's ideas." John's line: "When working with LLMs, it is on you to define the tone, the core ideas, the new insights." "Be the showrunner, not the audience" is Josh's compression of it.

### Slide plan (final text)

**Part 0**

1. Title (navy). "Foundations of LLM agents,\nand the state of the art". Lead: "How the models work, why they work, and what you build around them." Presenter, affiliation, course line "NASA AI/ML STIG · 14 Sep 2026", room "joshspeagle.com".
2. Statement (navy). "Built with Claude.\nEvery idea is mine.\nNone of the slides are." Gloss: "How this talk was built is the example in part 3."
3. Cards. Title: "Four things to take away".

**Part 1 · How they work**

4. Where we are (card 1).
5. Divider (navy). "Text".
6. Figure. Title: "Text becomes tokens". Real cl100k pieces with indices: `Ast` 62152 · `roph` 22761 · `ysics` 17688 · `_is` 374 · `_a` 264 · `_statistical` 29564 · `_science` 8198. Body: "A token is a piece of text with a fixed index in a vocabulary of about 100,000. The model sees only the indices." Small: "Tokenizer: OpenAI cl100k_base · try one at tiktokenizer.vercel.app".
7. Figure. Title: "The context is all the model sees". Strip with SYSTEM PROMPT · EARLIER TURNS · YOUR MESSAGE brackets and a NEXT square. Body: "Everything the model knows about the problem is in this strip. Nothing carries over between runs unless it is put back in."
8. Figure. Title: "One pass, one distribution". Strip → MODEL → five bars (hydrogen 0.41 … iron 0.05). Body: "One forward pass gives one probability for every token in the vocabulary. That is the whole output." Mono: "Values illustrative."
9. Figure. Title: "The next token is a random draw". Bars with DRAW connector to the NEXT square; three mini-panels `temperature` (p ∝ exp(z/T)), `top-k`, `top-p`. Body: "The draw is why one prompt gives different answers. Temperature, top-k and top-p reshape the bars first; none of them checks anything." Small: "Not repeatable even at T = 0: the answer depends on what else is in the server's batch (Thinking Machines 2025)."
10. Divider (navy). "Images, audio, and other inputs".
11. Figure. Title: "An image becomes vectors". 4 × 4 patch grid → VISION ENCODER → PROJECTOR → four chips `v₁ v₂ v₃ v₄`. Body: "Patches go through a vision encoder, then a learned projector into the model's embedding space. The result is continuous vectors, not vocabulary entries." Mono: "LLaVA-1.5: 576 per image · Qwen2-VL: 256–1024, set by resolution".
12. Figure. Title: "One sequence, several kinds of token". A strip where word tokens pass through an EMBEDDING LOOKUP block and image vectors enter directly from PROJECTOR, both feeding MODEL; a faint third input labelled `audio` the same way. Body: "Text tokens are looked up in a table; image and audio vectors arrive already embedded. From here on the model treats them all the same way." Small: "The cost is real: an image is a few hundred tokens of context."
13. Divider (navy). "Tools".
14. Comparison (two chips). Title: "How a model refers to a tool". Left, heading "Described in the prompt": "The harness lists each tool's name, description and arguments. The model writes `{"name": …, "input": {…}}`; the harness parses it." Right, heading "As special tokens": "Open models have vocabulary entries for it: `<|python_tag|>`, `<tool_call>`, `[TOOL_CALLS]`. End-of-turn and `<think>` are the same kind." Binding line: "Both are learned in post-training. The tool itself is code the harness runs."
15. Figure. Title: "What a tool looks like". A chip with the real `Read` definition: `name: Read` / `description: Reads a file from the local filesystem` / `input: file_path (string), offset (int), limit (int)`; beside it a second, shorter chip for `Bash`. Body: "A tool is a function with a name, a description and typed arguments. That is all the model ever sees of it."
16. Figure. Title: "When the model calls a tool". Strip → tool-call chip → THE HARNESS (stops the model, runs the tool) → result chip appended, marked "text the model did not write". Body: "The harness stops the model at the call, runs the code, and appends the result. The model continues from a context it did not fully write."
17. Figure. Title: "Prompt in, k tokens back". Two strips stacked: top "before": `prompt → next token`; bottom "after a call": `prompt + k tokens you did not write → next token`, the k tokens as chips with a different edge. Body: "Until now every token in the context was chosen by the model or by you. A tool result is neither, and it can be any length."
18. Figure. Title: "The same call, three results". One tool-call chip fanning to three result chips (`15 rows`, `timeout`, `0 rows: no match`). Body: "Results vary from call to call and can contradict what the model assumed. The model has to read them, and errors in them get acted on."
19. Statement. "A model predicts the next token\nfrom its context.\nAn agent is that model with tools,\nand every result goes back into the context."

**Part 2 · Why they work**

20. Where we are (card 2).
21. Figure. Title: "The training task". Masked sentence with fills (`distance` 0.62, `radius` 0.19, `mass` 0.11) and the same sentence continued left to right. Body: "Predict the hidden or next token, across trillions of tokens of text that is different every time." Small: "Masking (BERT 2018) and next-token (GPT-2 2019) both transferred to tasks never trained on."
22. Figure. Title: "Memorizing fails, strategies survive". TRAINING TEXT → MEMORIZE (ink-3 square, "fails on new text") and → REUSABLE STRUCTURE ("works everywhere"). Body: "New text at every step, parts hidden, parts of the network dropped out. Whatever only works on seen examples is punished; whatever works everywhere is kept."
23. Figure. Title: "One layer, one gradient step". Left: a context of `(x₁, y₁) (x₂, y₂) (x₃, y₃) x?` chips; right: MODEL drawn as four stacked layer blocks, each labelled `step`, output `ŷ`. Body: "If the only strategy that works on every regression problem is gradient descent, the network learns gradient descent. One attention layer can implement one step (von Oswald 2022)."
24. Figure. Title: "Generating tokens adds steps". Loop: MODEL → `token` → back to the strip, with a small descending bar chart labelled `loss` beside successive tokens. Body: "A forward pass has fixed depth. Each generated token is another pass, and nothing limits the count. A reasoning trace is the model running a procedure to completion."
25. Figure. Title: "Scale keeps working". Loss-vs-compute line, three squares `GPT-2` `GPT-3` `today`. Body: "Loss falls as a power law over seven orders of magnitude of compute. Architecture details matter far less than scale." Small: "Kaplan 2020; Hoffmann 2022 · attention and pre-training: Oct 5."
26. Figure. Title: "Representations are the by-product". Othello boards: THE BOARD (never shown) · PROBE READOUT · AFTER INTERVENTION. Body: "A model trained only on move sequences keeps a linear map of the board, and editing it changes the moves. The cheapest way to predict text about the world is to represent the world." Small: "Li 2023; Nanda 2023; Gurnee & Tegmark 2023 · interpretability: Dec 7."
27. Figure. Title: "The same bet at every stage". PRE-TRAINING "representations" → FINE-TUNING "format, instructions" → RL ON CHECKED ANSWERS "self-checking, long reasoning" → RL ON TOOLS "strategies that transfer". Body: "Each stage sets a hard, varied objective with no way to memorize. Reasoning appeared under RL without being shown; tool skills learned on one domain transfer to others." Small: "Brown 2020; DeepSeek-R1 2025; RITE 2025 · finetuning and RL: Oct 19."
28. Statement. "Predicting text is not acting.\nPost-training taught the model\nto follow instructions, reason, and call tools.\nNow it needs somewhere to act."

**Part 3 · The harness**

29. Where we are (card 3).
30. Figure. Title: "The harness assembles the prompt". Ordered stack SYSTEM PROMPT · TOOL DEFINITIONS · INSTRUCTION FILES · MEMORY · SKILL DESCRIPTIONS · CONVERSATION · TOOL RESULTS → MODEL → "text → you" / "tool call → run" with return to TOOL RESULTS. Body: "Before you type, the harness has filled most of the context. It decides what goes in, in what order, which tools exist, and who approves an action."
31. Figure. Title: "The scaffold that built this talk". Centre MODEL (`Claude Fable 5.1, lead`) inside a copper-edged rectangle labelled HARNESS · Claude Cowork; left column chips: SYSTEM PROMPT · MEMORY (profile, preferences, talks.md) · SKILLS (pptx, dataviz, …) · PROJECT FILES; right column chips: FILE TOOLS · SHELL · WEB SEARCH · DEVICE FILES · MEMORY TOOLS · AGENT · ASK THE HUMAN; a HUMAN chip above with a connector to ASK. Body: "Everything in this diagram is real. The model is one box. The rest is the harness."
32. Figure. Title: "The files it drew on". A file tree in mono inside a chip, two columns: `TALK_DESIGN.md` · `SESSION_ARTIFACTS.md` · `research/web_brief.md` · `research/why_they_work_brief.md` · `research/own_decks_brief.md` · `research/harness_brief.md` || `_design-system/design/SLIDE_DESIGN.md` · `slides/lib/speagle_slides.py` · `slides/lib/README.md` · `build/build_stig.py` · `memory: profile, preferences, talks.md`. Body: "Specs, research, the slide library, the builder script, and memory. The agents read these; you are looking at what they produced."
33. Figure. Title: "Instruction files". USER · PROJECT · LOCAL chips merging into one trunk "appended every session"; a chip with the real §8 excerpt from `SLIDE_DESIGN.md` ("Never: emoji, icons standing in for words, drop shadows … more than one pad … two ideas on one slide."). Body: "Standing text the harness appends at the start of every session: your rules, standards and specs. The spec behind these slides is one."
34. Figure. Title: "Skills load when needed". Four stages: "one-line description in the prompt" → "task matches" → "SKILL.md loaded" → "its files and scripts read" → ACT. Chip with the real pptx description (two lines). Body: "Only the description sits in the prompt. When a task matches, the file is loaded and tells the model what else to read."
35. Figure. Title: "Memory is read the same way". SESSION 1 → writes MEMORY FILE; SESSION 2 prompt holds "index + one-line descriptions"; a connector "read in full when relevant" to the file. Chip with two real memory lines. Body: "The index is in every prompt; a file is read only when the task calls for it. This is how the same preferences reached every agent tonight." Small: "Retrieval and memory: Oct 26."
36. Figure. Title: "Tools and permissions". TOOL CALL → PERMISSION CHECK (mono line under it: `read-only · ask each time · approve by class · automatic`) → RUN (sandbox) → RESULT. Body: "Every call passes a check the harness owns, from asking about everything to asking about nothing. MCP is the open standard for plugging tools in." Small: "Tool use and MCP: Nov 2."
37. Figure. Title: "An agent can be a tool". A chip with the real `Agent` definition: `name: Agent` / `description: Launch a new agent to handle multi-step tasks` / `input: prompt (string), model (string)` / `returns: the agent's final message`. Beside it a small MODEL block with its own strip. Body: "Same shape as Read or Bash. The 'code' that runs is another model with its own context; what comes back is text."
38. Figure. Title: "The same task, run three times". TASK → RUN 1 · 2 · 3 → three different OUTPUT chips, bracket "similar, not identical". Body: "Outputs are drawn, not computed. Three runs land in similar places, never the same place, and you cannot reproduce one exactly."
39. Figure. Title: "Small changes in context, different behaviour". Two strips identical except one chip (`INSTRUCTION FILE` present / absent) → two MODEL outputs that differ. Body: "The model is the same. Change one line of the context and the behaviour changes. That is a risk in one long conversation and a design tool in many short ones."
40. Figure. Title: "Why split the work across agents". LEAD block → three SUBAGENT chips with their own strips → SUMMARY chips back. Body: "Each agent gets a clean, purpose-built context and returns a summary. Work runs in parallel, detail stays out of the lead's window, and each result can be checked." Small: "Cost: ~4× the tokens of a chat; multi-agent ~15× (Anthropic 2025) · orchestration: Nov 16."
41. Figure. Title: "The workflow that built this talk". Tree: HUMAN (brief, 5 decisions) → LEAD (plan, write, review) → RESEARCH A · B · C · D → LEAD → BUILDER → RENDER → CHECK → LEAD → HUMAN. Mono lines with the four research and three builder statistics. Body: "Cheaper models did the implementation; the lead planned and wrote; the human decided. That rule was written down before anything ran."
42. Figure. Title: "When a result says no". Two chips with the verbatim failures (blocked shell; locked log) and their recoveries. Body: "Two tool results tonight were failures. Both were read and worked around. A check step is what turns a loop into a workflow."
43. Statement. "Same model, different harness,\ndifferent agent.\nThe harness is the part you control."

**Part 4 · Working with them**

44. Where we are (card 4).
45. Figure. Title: "Restrictive to flexible". Axis with chips: read-only · ask each time · plan, then approve · approve by class · autonomous, review after · unattended. Body: "One workflow moves along this axis: restrictive while setting up, flexible once specified, restrictive to check. It shifts as models and your work change."
46. Figure. Title: "Discussion, delegation, distribution". The axis with three navy blocks: "you see every step" · "you audit the result" · "the spec carries the work; you audit the process". Pad on Distribution. Body: "The further right, the less you see and the more the specification has to carry."
47. Figure. Title: "Vision, communication, execution". Three blocks; brackets "yours" under the first two, "the agent's" under the third. Body: "Know what you want. Say it so an agent, or a colleague, could act on it without you. The first two are most of the work; do them first."
48. Figure. Title: "Intent into things the model reads". INTENT → SPEC · PRINCIPLES · SKILLS · MEMORY · INSTRUCTIONS → HARNESS. Body: "Anything every run should respect goes in a file the harness loads. Written once, read every time. Keep it short and prune what stops being true."
49. Figure. Title: "Communication runs both ways". YOU ⇄ MODEL with labels "what you want, in checkable terms" / "what it did, in terms you can check". Body: "Ask for reports you can verify: assumptions stated, sources resolved, changes listed. Do not accept output you cannot read."
50. Object stack. Title: "What it costs to skip this". "Developers with early-2025 AI tools were 19% slower and believed they were 20% faster (METR 2025)." "People who shipped code with AI scored 50% on understanding it, against 67% without (Anthropic 2026)." "Treated like ordinary code, these systems cost time. Treated as systems you design and monitor, they return it."
51. Statement. "Be the showrunner, not the audience." Gloss: "After John Wu, 'Clear Vision, Clear Communications' (2025), quoting Javier Grillo-Marxuach, The Eleven Laws of Showrunning."

**Part 5 · What is possible today**

52. Figure (bars). Title: "How long a task can an agent finish". METR bars. Body: "METR measures the human working time of coding tasks an agent completes half the time. Doubling about every seven months since 2023." Small: "Caveats: coding only; the top bar's interval is 2–20 hours · METR Jan 2026."
53. Table. Title: "What agents did for this talk". Rows: Reorganize 270 files, 3.7 GB · script, run by the human; Index 176 talks · script; Research, four briefs · 4 agents, 2–11 min each; Build 46–55 slides, three versions · 3 agents, 17–25 min each; Plan and every line of text · lead; Scope, structure, what to cut · human, 5 checkpoints. Body: "The unit of work moved from lines of code to whole tasks. The human's work moved to the start and the end."
54. Object stack. Title: "In our field". "ReplicationBench (2025): agents asked to reproduce 20 astrophysics papers, 111 tasks. Best agent: 22%." "Kosmos: 79% of statements accurate, 58% of synthesis statements." "cmbagent: 91% on cosmology questions with retrieval." "AstroMLab, AstroReview, and the STIG textbook, Part V." Pad on 22%.

**Part 6 · Close**

55. Closing layout with cards. Title: "Four things to take away". The four cards, full ink, lines replaced: How they work: "Context in, distribution out, one draw appended. Tool results are text the model did not write." Why they work: "No memorizing, varied text, a hard objective. Representations are the by-product." The harness: "Prompt, tools, permissions, memory, skills, agents. You choose where it sits on the axis." Working with them: "Vision, communication, execution. Intent into files. Monitor the harness." Navy band: lockup; course line "Next: Sep 21 and 28, hands-on agentic coding · ai4astro.org · j.speagle@utoronto.ca". Notes acknowledge John Wu's SSAIL tutorial as an influence on the teaching approach.

### Speaker notes

Every slide: what to say (this is where the longer explanations from v3 go), sources and dates, later-talk pointers. Slides 31, 32 and 41 state what the agents did and what Josh decided.

## Review checklist

Text budget met on every slide (count words). One line per title. One pad at most. Figures in tokens only. Every number sourced in notes. 55 slides for 50 minutes: the dividers and statements are seconds each.
