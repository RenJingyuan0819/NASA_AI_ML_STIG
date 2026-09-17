# Lecture 26 — Foundations of LLM agents and the state of the art

**Josh Speagle** (University of Toronto) · NASA AI/ML STIG · Monday, 14 September 2026

Opening lecture of the 2026–2027 season. This folder publishes two of the
working documents behind the talk — not just the slides — because the talk
itself is a worked example of its own subject: it was designed, researched,
and built inside an agent harness (Claude Cowork), with a lead model planning
and writing, four research agents and three builder agents executing, and
Josh deciding at five human checkpoints.

## Files

| File | What it is |
|---|---|
| `TALK_DESIGN.md` | The full design spec for the talk: learning outcomes, the four take-away cards, a per-part time budget, an enforced text budget (≤ 30 words per slide statement, nothing under 20 pt), and the final slide-by-slide plan with speaker notes. This is the document the agents were given — a backwards-designed spec, not a prompt. |
| `web_brief.md` | One of the four research briefs the agents compiled before a single slide was drawn. Every factual claim in the lecture carries a source URL here (or is flagged as unverifiable). A representative example of delegating research to an agent and auditing what comes back. |

## How the talk was made

The session ran on 14 September 2026. The lead model (planning, prose, and
edits) delegated to cheaper models by difficulty: four research agents
(2–11 minutes each) produced briefs on the state of the art, why LLMs work at
all, harness mechanics, and Josh's own earlier decks; three builder agents
(17–25 minutes each) turned the approved design into successive versions of
the deck, which Josh reviewed slide by slide. The final deck is 54 slides.
The whole workflow — including the two tool failures that were read and
worked around — is described in the talk itself (Part 3, "The harness").

## The rest of the guts

Josh's complete working folder — the other three research briefs, his
slide-by-slide feedback on each deck version, the build scripts, and the
earlier deck iterations — is available here:

<https://www.dropbox.com/scl/fo/qrlazliavt7wkvrp3v7d4/AKAcxYIMP4be5c2O0FI082c?rlkey=w9lhr22jjfc2kjou5bbsyjobk&dl=0>
