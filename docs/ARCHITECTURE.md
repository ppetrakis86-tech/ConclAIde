# ConclAIde: Architecture and Technical Overview

This document describes what ConclAIde is and how it is designed to work. It reflects Master Design v1.1 (12/10/2026).

**Status: design only.** The architecture below is locked, but none of it is implemented yet. The next step is the Go backend.

Each section carries one of three labels:

- **Locked**: decided with an explicit yes.
- **Open**: still under discussion or waiting on work.
- **Proposal**: suggested, not yet approved. (None remain open at the time of writing.)

## 1. What it is (Locked)

ConclAIde is a multi-agent system that automates multi-AI review of documents. It replaces a manual workflow in which a document was passed by hand through several AIs in turn and the results compared.

It is a council, not a factory. It is not a document generator; it is a mechanism for dialectic. The system proposes, the human decides, and the goal is to improve the human's thinking, not replace it.

### Laws

1. The human always has the final say.
2. Roles are independent of models. Role → model mapping is configuration, not code.
3. Agents collaborate, they do not compete.
4. Critique adapts to the document: a primary-school exercise is not judged like a business plan.
5. The loop only ever returns to the Synthesizer, never to the Workers.
6. Cost awareness: each stage uses the cheapest model that does the job correctly.
7. The aim is to improve human thinking, not to replace it.
8. The head of the council is the human.

If the whole implementation changes tomorrow, the laws do not.

## 2. Pipeline: Input → Processing → Output (Locked)

Expensive intelligence sits in the middle; cheap models sit at the edges.

```
                ┌─────────────────────────────┐
 Entry A ──────▶│                             │
 Entry B ──────▶│  Input adapters (cheap)     │
 Entry C ──────▶│  → normalized document      │
                └──────────────┬──────────────┘
                               ▼
                    Router / Orchestrator
                               ▼
                       Workers ×2-3 (parallel)
                               ▼
                       Synthesis Agent (2-3 options)
                               ▼
                       Breaker (on / off / auto)
                               ▼
                         Human picks
                               ▼   ▲
              changes ─────────┘   │ (loop: Synthesizer only)
                               ▼
                       Finalizer (cheap)
```

### Input layer

A cheap model adapts the input. The entries are adapters: the core never knows where the input came from. All entries emit the same thing, a **normalized document plus context**, so Entry B and C can plug in later without changing the core.

| Entry | Input | Path |
|-------|-------|------|
| A | Finished document | Check / improve → Router |
| B | Raw idea | Intake Agent (asks the user questions) → first draft → Router |
| C | Several texts | Merge / mapping → Router |

### Processing: AARS

Analysis → Assignmentation → Research → Synthesizing → (Breaking, conditional)

- **Router / Orchestrator**: analyzes the input, prepares assignments, and in Auto mode decides the mode. It judges by the attention the job needs, not the topic (a program for a market launch needs legal attention).
- **Workers ×2-3**: run in parallel and produce observations, suggestions and corrections.
- **Synthesis Agent**: deep thinking, critique, gap finding. Produces 2-3 options.
- **Breaker** (conditional): a stress test. The Synthesizer fixes what it finds.
- **Human picks**: the user chooses an option or asks for changes.
- **Loop**: change requests go **only** to the Synthesizer.

### Output layer

The **Finalizer** is a cheap model that copies and formats the chosen option. It needs accuracy, not intelligence.

### Modes

| Mode | Behaviour |
|------|-----------|
| Auto (default) | The Router decides |
| Light | 2 Workers, fast and cheap |
| Deep | 3 Workers with different "minds", Synthesis at max effort |

### Breaker toggle

A separate stage after Synthesis, with a user toggle: on / off / auto. In auto the Router decides by job type: a primary-school exercise skips it; a business plan, an architecture or a contract turns it on.

### Live progress (SSE)

Progress streams to the UI over Server-Sent Events:

- Analysis, Assignmentation: fast, no substages.
- Research: Analyzing (Workers) and Researching (Synthesis Agent).
- Synthesizing: Thinking, Synthesis, Finalizing.
- Breaking, when active.

## 3. Seats, not models (Locked)

The system has **seats**, each with declared skills. The Router sees only seats and skills, never models. Which model sits in which seat is defined in settings (`config/`). This makes Law 2 structural: the Router literally cannot pick a model.

Settings come at two levels: global defaults (set and forget) and a per-run override before the pipeline starts.

### Escalation: no suitable seat

1. **Level 1, Analyst (Sonnet)**: the Router does not improvise. It sends the Analyst the message "a seat was requested that does not exist, split the job across existing seats". A new assignment plan comes back and the flow continues.
2. **Level 2, human decision**: if the Analyst cannot either, the system stops and asks: share the work across all agents; create an extra seat for this job; the seat is not needed, proceed without it; or propose something else. Creating an extra seat makes a temporary seat for the current run only, with a fallback model from config. It is not saved to settings.

### Company tracks

A provider track fills the seats with one vendor's models.

| Stage | Anthropic track | Google track |
|-------|-----------------|--------------|
| Router / Input | Haiku | Flash Lite |
| Workers | Sonnet ×2 (Deep: plus a third, different model) | Flash plus a Grok critic (Deep: plus GPT) |
| Synthesis | **Opus** | Gemini Pro |
| Finalizer | Haiku | Flash Lite |

Synthesis on the Anthropic track is Opus (decided 12/10/2026). Fable stays out of ConclAIde: in the benchmark it hard-blocked aggressive roles such as the Breaker, rerouting to Opus in a way no prompt could bypass, and it also broke at the final stage of synthesis. These are access gaps, not capability gaps.

**Open:** a later decision may replace fixed tracks with "employees with capabilities" chosen per task from benchmark data. Who chooses, the user in the mapping or the Router at run time, is unresolved. The second option would conflict with Law 2.

## 4. Technical characteristics (Locked)

| Layer | Technology |
|-------|-----------|
| Frontend | HTML + CSS + vanilla JS |
| Backend | Go (`net/http`), no frameworks |
| API | REST, JSON |
| Live progress | Server-Sent Events |
| Model access | OpenRouter API (single key) plus Google API |
| Development | Free model pool (OpenRouter free models, Gemini free tier) |

The model mapping lives in `config/`. The benchmark results end up there too.

## 5. Capability benchmark: CCB (Locked framework, partial data)

The ConclAIde Capability Benchmark is the empirical basis for the seat mapping. The goal is not "which AI is best" but: for capability X, at this budget and with these requirements, which model gives the best quality for the cost?

Findings feed the **config mapping**, not the Router.

### Tiers

| Tier | Meaning | Rule | Capabilities |
|------|---------|------|--------------|
| A, Commodity | Models score about equal | If scores match, take the cheapest | Grammar and syntax, rewriting, summarization, translation, formatting, simple Q&A |
| B, Core cognitive | Models differ | A pricier model is worth it if the gain is real | Structural analysis, logical criticism, Breaker, idea expansion, theory development, multi-document synthesis, long context, human collaboration |
| C, Specialized | Not every user needs them | Case by case | Coding, debugging, code review, software architecture, legal awareness, research, fact checking, educational, creative writing |

Each capability also has a **Criticality** (Low / Medium / High). At Low, a cheaper model is acceptable even with a small quality loss; at High, pay for the best.

### Metrics, per capability × model

- Quality (0-10), Cost ($ per run), Speed (seconds), Consistency (0-10, same result across repeats).
- Confidence (1-5): how sure we are of the score.
- Evidence: the number of tests behind the row.

### Collaboration Score

Measured per model overall, not per capability. Eight traits scored 0-10: follows instructions; makes assumptions (fewer is better); admits uncertainty; proposes alternatives; accepts feedback; cooperates; explains why; keeps context.

### Test protocol

For each capability: same prompt, same input file, explicit evaluation criteria, three repetitions, a human evaluator. For the Breaker, for example: found real gaps, did not invent problems, proposed fixes, explained why.

### Results so far

Test 2 covered three capabilities across five models (blind, on claude.ai): theory development, idea expansion, human collaboration. Opus 4.8 averaged 9.7, Fable 5 9.5, ChatGPT 5.5 8.8, Gemini Pro 3.1 8.7, Grok 4.3 7.0. Fable ranked first in idea expansion, and was unusable in the Breaker role. The rest of the capabilities have no data yet.

## 6. Failure handling

A refusal is a different case from a timeout. A timeout leaves no trace; a refusal leaves text that **contaminates** whichever model reads it next (refusal contamination, observed repeatedly in tests).

| Case | Status | Behaviour |
|------|--------|-----------|
| Worker refuses | Locked | The output is replaced with a neutral note ("Worker X did not deliver"). Optionally re-run on another model of the same seat in a clean context. |
| Synthesizer refuses | Locked | Automatic fallback to another model in a fresh context, without the refusal text. **Open:** Opus is now the primary Synthesizer, so a new fallback model must be chosen. |
| Router asks for a missing seat | Locked | Escalation (Section 3) |
| Worker fails or times out | Open | |
| Very large document (token limits) | Open | |
| Many documents at once | Open | |
| Synthesizer returns 1 option instead of 2-3 | Open | |
| Cap on loops back to the Synthesizer | Open | |

## 7. Roadmap

| Phase | Scope | Status |
|-------|-------|--------|
| 0, prerequisites | CCB for the Phase 1 seats (logical criticism, structural analysis, multi-document synthesis); router test; remaining open decisions | Open |
| 1, core | Entry A only; AARS core with fixed seats, no escalation; Anthropic and Google tracks; basic web UI with SSE; no Breaker | Locked |
| 2 | Breaker (toggle plus auto-decision); escalation (Levels 1 and 2); Entry B (Intake Agent) | Locked |
| 3 | Entry C (multiple texts); voice interface; a hidden Easter egg | Locked |

Future features, not scheduled:

- **Project memory**: decisions, definitions, rejected ideas and standing axioms of a project. A simple JSON file is enough to start.
- **Batch API processing**, once a concrete need appears.
- **Clarification checkpoint** and **blind/reveal critique**, which came out of an Intake test and still need a decision.

## 8. Intake Agent notes (from CCB Test 1)

A manual simulation of Entry B on a real idea produced concrete requirements for the Intake Agent:

- Always keep the raw input attached to the intake package.
- Add a "revisions" field so changes of mind are not lost.
- Read back the understanding to the user before proceeding.
- Ask whether a file for the idea already exists.
- Use separate fields rather than one blob.
- Treat "Intake" as its own benchmarked capability.

## 9. Operating the design

Locked decisions are never erased. If the decision changes, the new one is written with its reason and the old one stays as history. Proposals lock only with an explicit yes from the project owner.
