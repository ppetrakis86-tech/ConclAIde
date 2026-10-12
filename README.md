# ConclAIde

A multi-agent AI system that reviews documents as a council, not a factory.
Several models work in separate roles, and a human makes the final call.

ConclAIde automates what was previously a manual workflow: running a document through several AIs one after another (Grok → Gemini → Claude) and comparing the results by hand.

## What it does
1. **Idea → Draft**: analyzes a raw idea and produces a structured first draft.
2. **Gap finding**: finds holes and missing steps in a workflow document.
3. **Cross-document review**: compares documents and flags gaps and conflicts between them.
4. **Expansion & deep analysis**: develops a document further and analyzes it in depth.

## How it works: AARS
- **Analysis**: understand the input and what is being asked.
- **Assignmentation**: assign each part of the work to the right agent seat.
- **Research**: agents investigate their part independently.
- **Synthesizing**: results are merged into one conclusion for human review.
- **Breaker** (conditional): an adversarial pass that attacks the synthesized options before the human sees them.

```
Input (A: document · B: idea · C: several texts)
  → Router / Orchestrator
  → Workers ×2-3
  → Synthesis Agent (2-3 options)
  → Breaker (on / off / auto)
  → Human picks
  → Finalizer (formatting)
```

The loop after the human's pick goes back **only to the Synthesizer**, never to the workers.

## Design principles
1. The human always has the final say, and is the head of the council.
2. Roles are independent of models. Role → model mapping is configuration, not code.
3. Agents collaborate rather than work in isolation.
4. Critique adapts to the document being reviewed.
5. Cost-aware: cheap models at the edges (input, output), expensive ones in the middle.
6. It improves the human's thinking, it does not replace it.

The Router only knows **seats** and skills, never specific models. Escalation has two levels: Level 1 (Analyst) and Level 2 (human decision).

## Modes
- **Auto**: the Router decides the depth.
- **Light**: fewer workers, cheaper models.
- **Deep**: full pipeline.

Progress is streamed live to the UI over SSE.

## Planned stack
HTML/CSS/vanilla JS frontend, Go `net/http` backend, REST/JSON, SSE, OpenRouter + Google API for model access.

## Easter egg: Greg
Greg is a joke agent. He sits on the side of the AARS pipeline and drops comments on what the council is doing. He never touches the results. Born from a voice-chat glitch, he is a mascot for every STT/TTS mishap.

## Roadmap
| Phase | Scope |
|-------|-------|
| 1 | Entry A (existing document) + AARS core, no Breaker |
| 2 | Breaker, escalation, Entry B (idea → Intake Agent → draft) |
| 3 | Entry C (multiple texts), voice, Greg |

## Status
Design phase. No application code yet. The architecture is locked (see [Design principles](#design-principles) and the pipeline above); what exists so far is the design and a model-evaluation effort.

### CCB: ConclAIde Capability Benchmark
To decide which model sits in which seat, we built a benchmark rather than guess: 23 capabilities in tiers A/B/C, scored on quality, cost, speed, consistency, confidence and evidence, plus a collaboration score. Every test uses the same prompt and input, explicit criteria, three repetitions, and a human evaluator.

- **Test 0, kickoff**: the session where the architecture came out of the manual workflow.
- **Test 1, Entry B on a real idea (MNN spec)**: simulated by hand in separate chats (Intake → Draft → Analyst). Produced concrete improvements for the Intake Agent (raw input always attached, a revisions field, read-back confirmation).
- **Test 2, blind model audition for the Synthesis/Breaker seats**: five models scored on theory development, idea expansion and human collaboration.

### Decisions so far
- Synthesis runs on Opus. Fable stays out of ConclAIde: it blocks aggressive roles (such as the Breaker) and silently reroutes to Opus.
- The CCB fills the role → model mapping in `config/`, never the Router.
- A Worker refusal becomes a neutral note, with an optional re-run on another model in a clean context.
- A Synthesizer refusal triggers an automatic fallback in a fresh context without the refusal text. A fallback model still has to be chosen.

### Lessons learned
- A refusal is not stateless: it contaminates whichever model reads that context next.
- The evaluator needs the exact input of each run before scoring.
- A drafter and an analyst on the same model share the same blind spots.

## License
TBD
