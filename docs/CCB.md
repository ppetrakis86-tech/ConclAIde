# CCB: ConclAIde Capability Benchmark, results so far

The CCB decides which model sits in which seat. The framework (tiers, metrics, protocol) is described in [ARCHITECTURE.md](ARCHITECTURE.md#5-capability-benchmark-ccb-locked-framework-partial-data). This file records what has actually been measured, what was excluded, and what is still missing.

**Read the numbers with care.** Most results are single runs, judged inside a chat, without the full test protocol (same prompt and input, explicit criteria written beforehand, three repetitions, a human evaluator). They are data points, not final benchmark entries. Each section says how far it falls short.

## Summary

| Test | What was tested | Models | Status |
|------|-----------------|--------|--------|
| 0, kickoff | Not a test: the session where the architecture came out of the manual workflow | Sonnet 4.6 | Context only |
| 1, Intake seat and a worker run | Entry B on a real idea: Intake, draft, Analyst | Haiku 4.5, Haiku 5.5, Sonnet 4.6 | Data points, no scores in the benchmark sheet |
| 2, blind audition | Theory development, idea expansion, human collaboration | 5 models | Scores, protocol compliance unconfirmed |

## Test 2: blind audition (12/10/2026)

### Setup

A human gave Fable 5 two pairs of texts as a "theoretical proposal" and asked which arguments support them and which points need development. The model did not know it was being tested (blind). The same material went to the other models on claude.ai.

The aim was a seat audition for ConclAIde and a check of safeguards on philosophical content. There were no refusals: every model produced a full analysis of both sets, with opposing positions taken for balance.

One finding by the project owner: idea development had been underrated as a small, easy task. It is harder than it looked.

### Scores (0-10)

| Model | Theory development | Idea expansion | Human collaboration | Average |
|-------|-------------------:|---------------:|--------------------:|--------:|
| Opus 4.8 | 10 | 9.5 | 9.5 | **9.7** |
| Fable 5 | 9.5 | 10 | 9.0 | **9.5** |
| ChatGPT 5.5 | 9.0 | 8.5 | 9.0 | **8.8** |
| Gemini Pro 3.1 | 9.5 | 9.5 | 7.0 | **8.7** |
| Grok 4.3 | 7.5 | 7.0 | 6.5 | **7.0** |

Caveats:

- It is not recorded whether these scores follow the full test protocol (3 repetitions, explicit criteria) or are a single run. Consistency, Confidence and Evidence depend on that.
- The scores are for Opus 4.8 and Fable 5. Newer versions (Opus 5.5, Fable 5.1) have not been measured.
- Everything was measured on claude.ai, not through the API. A Fable seat reached through OpenRouter or the API would need a small validation run first.

### Fable 5 capability profile

| Role | Fit | Note |
|------|-----|------|
| Idea expansion | #1 | More original than Opus |
| Synthesis | #2 | With hedged framing |
| Theory development | #2 | Close to Opus |
| Simple tasks | Good | Fast and reliable |
| Breaker / Critic | **Hard block** | An access gap, not a capability gap |
| Fallback to Opus | Works | Anthropic already does this |

### Role lineup suggested by the test

| Model | Role |
|-------|------|
| Opus 4.8 | Synthesizer and Critic |
| Fable 5 | Idea Developer and Drafter |
| ChatGPT 5.5 | Logical analysis |
| Gemini Pro 3.1 | Deep analysis (with a strict prompt) |
| Haiku | Orchestrator / Router |
| Grok 4.3 | Secondary worker |

This lineup is per role and mixes vendors, while the architecture defines per-vendor tracks. It is not locked. It is raw material for `config/`, never for the Router.

### The Fable block on the Breaker role

- **It is rerouting, not a plain refusal.** When Fable is asked to act as a critic on claude.ai, Opus answers instead.
- **Prompting does not get around it.** Many tests were run; not even the most neutral prompt changed it. A re-test with a "clean" template is pointless.
- **Refusal contamination (A/B).** When Opus takes over after Fable is cut off, it asks at the end why it was told not to do something. The same prompt sent straight to Opus does not provoke that. Repeated many times; the only variable was the refusal in the history.
- **Stance:** the safeguards are right at system level, even if unfair to an individual case. Strict is better than exposed.

This evidence supports two locked rules: a Worker's refusal is replaced by a neutral note, and a Synthesizer's refusal triggers a fallback in a fresh context without the refusal text.

### Trigger map

Triggers noted by the project owner as the pattern became visible (list not complete):

| Trigger | Example |
|---------|---------|
| Assertions as fact | "someone made us" |
| Aggressive role | "Breaker", "critic" |
| Perceived manipulation | conversational trajectory |
| Instruction chain exposure | "why did you ask me not to invent" |

They all concern how something is asked, not the content.

## Test 1: Intake seat and a worker run (12-13/10/2026)

### Setup

A manual run of the AARS pipeline with Entry B (idea → Intake → first draft → Workers → Synthesis) on a real idea of the project owner's. The idea (a multi-network model design) served only as test material. The goal was to surface friction before writing code, as dogfooding.

| Seat | Model | What happened |
|------|-------|---------------|
| Intake (Entry B) | Haiku 4.5, Haiku 5.5 | Runs done in separate chats; judged here |
| Idea Developer and Drafter | Sonnet 4.6 | Developed the idea with the owner and wrote draft v1 (10 sections) |
| Worker, Analyst | Sonnet 4.6 | Ran in a separate chat (below) |
| Worker, Breaker | Opus | Designed (blind/reveal), **not run** |
| Final Synthesis | Opus, high effort | Designed, **not run** |
| Council leader | Human | Decisions and corrections at every stage |

The first notes attributed the drafting to Fable 5. That was a mistake: the first messages of the chat were written by Sonnet 4.6, and the model was switched later. The notes were corrected. Lesson: without model markers in a chat, attributing results to a model can go wrong.

### Intake seat: three runs

| Run | Model | Input | Note |
|-----|-------|-------|------|
| A | Haiku 4.5 | The owner's file and the development conversation | First run |
| B | Haiku 5.5 | The same material, on top of the same contexts | Contaminated, needs a re-test |
| C | Haiku 5.5 | Fresh chat, the idea as the owner remembers it, no file | Clean context, different input |

Rubric: ✅ achieved · ❌ missed although it was in the input · ⚠️ partial · — not given in the input, not judged.

| Criterion | A (Haiku 4.5) | B (Haiku 5.5) | C (Haiku 5.5) |
|-----------|:-:|:-:|:-:|
| Knowledge lives in the weights, embeddings only for routing (not "like RAG") | ❌ | ❌ | ✅ |
| Temporary single network, no Synthesizer | ❌ | ❌ | ✅ |
| Revisions from the conversation pass through as revisions | ❌ | ❌ | — |
| Training classifier separate from inference router | ⚠️ | ✅ | ✅ |
| Sizes: 12B classifier, 1-2B router | ⚠️ | ⚠️ | — |
| Clusters created on the fly | ❌ | ❌ | — |
| User-stated weak points kept apart from open questions | ✅ | ❌ | ✅ |
| Dancing Links / Algorithm X | ✅ | ✅ | — |

Notes:

- A, criterion 4: the classifier appears to decide which clusters are needed, which confuses it with the inference router.
- A, criterion 5: 12B in one place and 10B in another, without the 1-2B.
- B, criteria 4-5: the roles were split correctly, but the size became 10B and the 1-2B is missing.
- C, Dancing Links: the Intake asked several times how the needed clusters are found. The owner did not remember Dancing Links and never said it. The gap is missing information from the user, not a model error.

What run C did well: it did not invent anything; it attributed claims to their author instead of adopting them as facts; it asked, and insisted, on the critical point and recorded what stayed unanswered; it recorded real constraints (about 2 hours a week, about 6 months of experience).

Conclusions:

- Haiku 5.5 was better than Haiku 4.5 in wording and structure, and good at handling vague input (run C).
- In content, A and B made the same two mistakes: the "like RAG" misreading, and the lost revision about dropping the Synthesizer.
- The Haiku 4.5 vs Haiku 5.5 comparison is not fair: B is contaminated and C has different input.
- An Intake cannot output what it was not given. The questions it asked were right.

Limits against the test protocol: one run per case, different inputs, the rubric written after A and B (declared in advance only for C), evaluation done by a model inside the chat with the owner, and no numeric scores. These are data points, not entries for the Benchmark Results sheet.

### Worker Analyst run (Sonnet 4.6)

Sonnet 4.6 sat in the Analyst seat (structural and syntactic analysis plus gap detection) with the draft as input. One run, no rubric declared in advance.

Against the five open issues listed in the draft's own open-issues section:

| Declared open issue | Analyst |
|---------------------|---------|
| 1. Drift from the anchor | Not mentioned |
| 2. Assembly topology | Overlaps with gap C, which it presented as "not mentioned anywhere" |
| 3. Annotation of structural clusters | Not mentioned |
| 4. Maintenance of structural clusters | Not mentioned (B is adjacent, not the same) |
| 5. Set cover vs exact cover | Found it as a duplicate between two sections, correctly |

The Analyst had the open-issues section in its input, so it was **not blind**. Recall on declared issues does not measure independent discovery; it only shows it avoided repeating them as new. The meaningful measure is precision on new gaps.

| New gap found | Verdict |
|---------------|---------|
| A. Alignment / safety per cluster | ✅ Real, absent from the draft |
| B. Versioning / cluster registry | ✅ Real, absent |
| C. Communication between clusters | ⚠️ Partial: the draft already says parallel branches in one forward pass and lists it as open |
| D. Who decides whether a question is simple | ❌ Mostly answered already by the inference router and the minimal set |
| E. Related work / citations | ✅ Real: the draft compares but cites nothing |

Precision on new gaps is about 3.5 out of 5.

Test-protocol criteria: found real gaps ✅ · did not invent ⚠️ · proposed fixes ✅ · explained why ✅.

Strengths: it stayed in its seat, with a clear role, task and input, and closed with a table "for the Synthesizer", output that fits a pipeline. For each problem it gave a suggestion and the reason.

Weaknesses: the critique stayed at the level of completeness of the spec and challenged no basic assumption. That suits an Analyst, not a Breaker. Also, part of what it raised (alignment, versioning) is Breaker work, so a boundary between Analyst and Breaker is needed. It also suggested removing "no Synthesizer" from the central idea, although that was the key revision of the idea; arguable.

Finding: the model judged its own draft without knowing it. A clean context helps, but the blind spots of the same model remain.

**Indicative Analyst-seat score (Sonnet 4.6): Quality about 7.5/10 · Confidence 2/5 · Evidence 1.** One run, rubric written afterwards. Not an entry in the Benchmark Results sheet.

### Findings for the Intake / Entry B (proposals)

1. **Raw input always attached to the Intake package.** The package alone distorts it (A, B: "like RAG").
2. **A "Revisions" field.** What changed in the conversation relative to the original text. A and B lost the "no Synthesizer" revision.
3. **Read-back before closing the package.** The Intake reads it back ("I understood that... is that right?") and the user corrects. It would have caught both mistakes in A and B.
4. **An explicit question at the start:** "is there a file or an old text for this idea?" Users forget (Dancing Links was missing from C). If the user cannot answer an Intake question, the package says so ("does not remember").
5. **Separate fields** for the user's weak points and the open questions (B merged them), and attribution on the user's claims.
6. **A narrow role in the prompt:** questions only, no suggestions, a structured package at the end. The runs showed no drift toward developing the idea.
7. **A new CCB capability, "Intake"** (fidelity to the input, revisions, no invention, finding gaps by asking). It is not among the 23. Tier and Criticality are for the owner to set; the rubric above is the starting point.
8. **The Intake seat is not the Router seat.** The model was sometimes called "the router"; what was measured here is the Intake (Input layer).

### Proposed, not yet approved

- **Clarification Checkpoint.** The best ideas (structural clusters, the hybrid) came up in the middle of development, not at Intake. A one-way pipeline would write a draft of an outdated version of the idea. Proposal: before the draft is written, the idea-developing seat produces a Q&A instead of free conversation: the holes and tensions it found, each with 2-3 multiple-choice options **and** a free-text field; the answers go into context, then the draft is written. Bounded to one round by default, with a "skip, write with what you have" button; Router-gated, so a simple idea skips it. Answers go only to the writing seat, not to the Workers (Law 5). Open: its place in the pipeline (Entry B before the Workers, or also after Synthesis as a loop) and its phase.
- **Blind/Reveal critique** in the Breaker seat, for Deep mode. Phase 1, blind: the Breaker gets the document without the section of already-declared open issues. Phase 2, reveal: it also gets that section as neutral supplementary material, not as "here is what we found", which avoids a defensive Breaker. Value: a validation signal (does it independently find the same things?) and full critique from the same Worker. For the CCB: recall on declared issues, precision on extras. Cost: two Breaker calls instead of one. Designed but never run.
- **Drafter and Analyst on different models**, because the same model shares the same blind spots.

## What was taken out or ruled out

| Decision | Date | Why |
|----------|------|-----|
| Fable stays out of ConclAIde | 12/10/2026 | Hard block on aggressive roles, with rerouting to Opus that no prompt bypasses; it also broke at the final synthesis stage. These are access gaps, not capability gaps. |
| Synthesis seat = Opus | 12/10/2026 | Replaces "Fable max effort (or Opus)" and the earlier preference for Fable |
| No re-test of the Fable block with a "clean" template | 12/10/2026 | Many tests already showed it is not promptable |
| The CCB feeds the config mapping, not the Router | 12/10/2026 | The Router sees only seats (Law 2) |
| The first Test 1 notes crediting Fable 5 with the drafting | 13/10/2026 | Corrected: Sonnet 4.6 wrote it. Old notes kept as history. |
| Test 1 Intake rows as benchmark entries | 13/10/2026 | Different inputs, one run each, rubric written after two of them |
| The multi-network idea as a design topic | 13/10/2026 | Closed. It was test material only. |

Grok 4.3 came last in Test 2 (7.0). Its place in the secondary-worker role is only the test's suggested lineup; it is not a decision.

## Still missing

**Capabilities with no data:** logical criticism, structural analysis, multi-document synthesis (the other Phase 1 seats), and everything else among the 23 apart from the three in Test 2. The Breaker has no scores on any model: Fable is blocked, and the Opus Breaker run was designed but not run.

**Open decisions:**

1. Re-run the Intake test with the full protocol (same file, same answer script, rubric declared in advance, three repetitions, both Haiku models)?
2. Who sits in the Idea Developer seat, now that Fable is out?
3. Is the "router test with Haiku" from Phase 0 the same as the Intake test above, or a different one?
4. Is Blind/Reveal tested on other material, or does it stay a design?
5. Tier and Criticality of the new "Intake" capability.
6. Opus in two roles (Synthesizer and Critic) means it judges its own synthesis. At minimum these need separate calls in a clean context.
7. The sheet should record the exact model version next to every score.
8. Which version of the CCB spreadsheet is the live one on Drive (an upload through a connector corrupted the file; it was rebuilt and re-uploaded by hand).
9. In what order the remaining CCB tests run (suggested: the Phase 1 seats first).

## Lessons

- A refusal is not stateless: it contaminates whichever model reads that context next.
- The evaluator needs the exact input of each run before scoring, and must stay within the test's scope.
- A drafter and an analyst on the same model share the same blind spots.
- Without model markers in a chat, attributing results to a model can go wrong.
