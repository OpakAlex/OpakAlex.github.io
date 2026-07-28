---
title: "Deterministic Where It Matters"
date: 2026-07-28
draft: false
tags: ["AI", "determinism", "expert systems", "Talon", "OpenTalon", "LLM", "EITL", "agents"]
summary: "LLMs are probabilistic by nature -- probability buys flexibility, never 100% accuracy. So can a Talon + LLM agent pipeline promise same input, same output? The honest answer is a split: the decision core is provably deterministic, the LLM edge is only reproducible. Determinism comes from architecture, not from taming the model."
ShowToc: true
---

LLMs are **probabilistic by their very nature**. The probability theory we all know and love sits at the foundation of the mathematics every language model is built on. Strip away the marketing and a model is a machine that, given everything written so far, estimates a distribution over the next token and draws from it -- again, and again, and again. There is no fact being looked up and no formula being solved; there is a very good guess about what word tends to follow, weighted by how often patterns like it appeared in the training data. That is why the same prompt can yield two different answers, why "temperature" is even a dial you can turn, and why a model will complete a plausible-sounding sentence long past the edge of what it actually knows. Probability buys flexibility -- the ability to handle input no one anticipated -- but it never buys 100% accuracy. The two come from the same mechanism; you cannot keep one and discard the other.

We are used to trusting the calculations of machines. A computer operates on numbers, and numbers, as everyone knows, cannot lie. But can a machine *make up* convenient numbers? It sounds like nonsense -- how do you invent a calculation that a formula can trivially check? Yet with the arrival of the LLM, exactly that changed. A machine can now invent -- *hallucinate* -- as it generates a final result; or one agent in a chain can emit information that never existed and swing the outcome completely.

Should we fight this? The answer to that deceptively simple question is really philosophical: should we fight the **vices of a human being**? Those vices were written into us by evolution and the survival of the species. Society sets boundaries to keep them in check -- and we do the very same thing to the LLM. We try to fence in the vice of hallucination with strict prompts, with human-centric decision-making, with retrieval and verification. It helps. But nature is nature: the vices still leak out. They can ruin a thesis, a report, a business plan -- and they can kill a production system.

You could say that mistakes are normal; everyone makes them. But a **lie** is already a vice. What do you do when the lie *is* the mistake? For an LLM, that is precisely the case -- a confident, fluent, well-formatted falsehood is indistinguishable, at the point of output, from a correct answer. You have to constrain each agent to be as deterministic as possible. And what does determinism best inside a machine? **Program code** -- because determinism is the very foundation of code.

That is why the [Talon language](https://github.com/opentalon/talon-language) was created inside the [OpenTalon](https://github.com/opentalon/opentalon) ecosystem: let the LLM generate the code, then have each agent make its decision and verify it against **facts** -- and facts don't lie. Every run, the agent executes code the LLM authored ahead of time. That is what makes a deterministic result possible.

![Deterministic computation is verifiable and reproducible -- numbers don't lie -- while an LLM generates output by sampling tokens probabilistically; hallucinations can invent non-existent statistics in a business report or approve a drug that does not exist in medicine. Strict prompts, human review, RAG, and source verification limit the risk but do not eliminate it. Use LLMs where flexibility counts; keep critical calculations deterministic.](/images/deterministic-where-it-matters.png)

## The decomposition that changes the question

The question everyone actually asks -- *can I get the same output from the same input?* -- feels unanswerable while the model sits in the middle of the loop, re-deciding on every request. So don't leave it there.

This is the move [opentalon-agents](https://github.com/opentalon/opentalon-agents) makes. It splits the lifetime of an agent into two phases that most systems smear together:

- **Authoring.** A human describes a task in fuzzy natural language. The LLM does what it is genuinely good at -- turning that description into a small Talon program. This happens *once*.
- **Execution.** From then on, the runtime evaluates that Talon program on every tick, over the facts of the moment. **No model call. No sampling. No re-deciding.**

The LLM is great at authoring an automation from a vague request. But you don't want a model re-deciding what to do every five minutes, forever. So it doesn't. Once the Talon is written, the model steps out of the loop entirely.

What runs in its place is deliberately boring, and boring is the point:

- **Edge-triggered firing.** A rule fires when a fact *changes* in a way that matters -- a lab value crossing a threshold once, not on every poll that still reports it out of range.
- **Snapshot state between ticks.** The engine persists the fact snapshot, so replaying the last known value on restart fires nothing. No false re-fires, no drift.
- **Replay-safe restarts.** State is durable; the agent resumes exactly where it left off.

That is the determinism boundary: *no model in the loop.* Everything downstream of authoring is mechanical.

## Two determinisms, not one

Here is the reframe that makes the whole question tractable. "Deterministic pipeline" is not one property -- it is two, and they live in different places.

**Execution determinism** lives in the Talon core. It is the pure evaluation of rules over facts: the same facts always yield the same decision and fire the same actions, on any machine, at any time. This one you get for free -- it is what a deterministic language *is*.

**Authoring determinism** lives at the LLM edge: does the same natural-language input always produce the same Talon program? Honestly -- no, not at the token level. Sampling, model-version drift, even floating-point non-determinism in batched inference at temperature zero all conspire against a bit-for-bit guarantee.

But notice the trap in the original question. "Same output" measured *how?* If you measure at the token level, you will never win -- and you don't need to. Two Talon programs can be textually different and semantically identical: reordered rules, a renamed `define`, a different whitespace style. What you actually care about is whether they produce the **same decision on the same facts.** That is the level at which "same input, same output" is both meaningful and achievable.

So the pipeline has a probabilistic edge and a deterministic core. The engineering is entirely about keeping the boundary between them clean.

## A medical pipeline, end to end

Let me make this concrete with a domain where a hallucinated number is not an inconvenience but a lawsuit -- or a death. A medication-triage agent ingests an unstructured intake note and a lab panel, and must decide dosing, referral, or escalation.

### 1. What the LLM extracts

The model never doses anyone. It never sees a formulary or a contraindication table. Its only job is to turn a messy note into a canonical set of **facts about this patient**:

```talon
// ── Generated by the LLM from intake_note.txt + lab_panel.json ──
// The LLM is an information provider, not a decision-maker.

fact "patient.PT-55240" {
  age_years = 74
  weight_kg = 61
  pregnant = false
  reported_symptoms = ["fever", "productive cough", "dyspnea"]
  known_allergies = ["penicillin"]
  current_meds = ["warfarin", "metformin"]
}

fact "labs.PT-55240" {
  egfr_ml_min = 38          // kidney function
  crp_mg_l = 142
  wbc_10e9_l = 15.8
  potassium_mmol_l = 5.2
  measured_at = "2026-07-28T08:10:00Z"
}
```

That is the entire LLM contribution: two blocks of facts. If it misreads `38` as `88`, it has produced a *wrong fact* -- and a wrong fact is checkable. It has not produced a wrong decision.

### 2. What the clinicians authored, once

The rest of the file is written by pharmacists and physicians, reviewed by clinical governance, and versioned next to the code it governs. It does not change between patients. It is the deterministic layer that actually decides:

```talon
// ── Authored by clinical governance ─────────────────────
// Formulary limits, contraindications, renal adjustments.
// Reviewed by pharmacy, physicians, and compliance.

fact "formulary.amoxicillin" {
  drug_class = "penicillin"
  standard_dose_mg = 500
  max_dose_mg = 1000
}

fact "interactions" {
  warfarin = ["nsaid", "macrolide", "sulfonamide"]
}

// ── Conditions ──────────────────────────────────────────

define "allergic_to_drug_class" {
  "formulary.amoxicillin.drug_class" in "patient.known_allergies"
}

define "renal_impairment" {
  "labs.egfr_ml_min" < 45
}

define "hyperkalemia" {
  "labs.potassium_mmol_l" > 5.0
}

define "sepsis_warning" {
  "labs.crp_mg_l" > 100 and "labs.wbc_10e9_l" > 12
}

// ── Decision rules ──────────────────────────────────────

rule "Block a drug the patient is allergic to" {
  when is "allergic_to_drug_class"
  do set "rx.decision" "blocked"
  do explain "Patient allergic to {formulary.amoxicillin.drug_class}; select alternative"
  do require "review.pharmacist"
}

rule "Halve dose on reduced kidney function" {
  when is "renal_impairment"
    and not is "allergic_to_drug_class"
  do set "rx.dose_mg" "formulary.amoxicillin.standard_dose_mg" / 2
  do explain "eGFR {labs.egfr_ml_min} < 45: renal dose adjustment applied"
}

rule "Flag warfarin interaction" {
  when "warfarin" in "patient.current_meds"
  do comment "rx" "On warfarin -- avoid interacting classes, monitor INR"
}

// ── Edge-triggered escalation to a human ────────────────

rule "Escalate on sepsis warning" {
  when is "sepsis_warning"
  do require "review.physician"
  do notify "team.oncall" "Possible sepsis: CRP {labs.crp_mg_l}, WBC {labs.wbc_10e9_l}"
}

rule "Escalate on hyperkalemia before any renal-cleared drug" {
  when is "hyperkalemia"
  do require "review.physician"
  do explain "Potassium {labs.potassium_mmol_l} mmol/L exceeds 5.0 -- hold and review"
}
```

### 3. The LLM closes the loop -- but gated

Once the core has decided, the model is allowed back for one thing it is good at: writing the instruction sheet in plain language. And it is not trusted to send it. Talon verifies the draft against the recorded facts first:

```talon
rule "Draft patient instructions, verify, then release" {
  when "rx.decision" in ["approved", "adjusted"]
  do llm_draft "instructions" from "rx.decision_facts" tone "plain"
  do verify "instructions" against "rx.decision_facts"
  do send "instructions" to "patient.id" only_if "verify.passed" == true
}
```

Feed the same two fact blocks into this program tomorrow, on another server, and the escalation fires the same way, the renal adjustment computes the same milligrams, the allergy block triggers identically. **Same facts, same decision.** That is execution determinism, and it is provable -- you can read the rule that fired and the facts that triggered it.

## The safety net: extraction is verifiable, decisions are pinned

Look at where the model's uncertainty can and cannot land.

The LLM only ever emits `fact` blocks. A hallucination therefore surfaces as a *wrong fact* -- an eGFR that disagrees with the lab feed, an allergy the record doesn't list -- and wrong facts are exactly what a deterministic layer is built to cross-check and reject. The one thing the model cannot do is hallucinate a *decision*, because it never makes one. Decisions are pinned to rules that a pharmacist signed off on.

And when the model re-enters to draft the patient's instructions, it re-enters behind a gate: `verify "instructions" against "rx.decision_facts"`. If the prose claims a dose the facts don't support, it never sends. Determinism at the seam, not just in the core.

## Making the LLM edge reproducible -- not perfect

You cannot make the authoring step provably deterministic. You can make it *reproducible enough* that it stops being a source of surprise:

- **Greedy decoding** (temperature 0) removes deliberate sampling noise.
- **Grammar-constrained decoding** onto Talon's tiny grammar shrinks the output space dramatically -- the model can only emit valid `fact` blocks, so there is far less room to diverge. As I covered in [Code Mode for MCP](/posts/code-mode-for-mcp/), the whole grammar fits in a forty-line prompt.
- **Pinned model versions** stop silent drift when a provider ships a new checkpoint.
- **Prompt-hash caching** memoizes `input → Talon`: an input you have seen before returns the exact program you produced last time, for free.
- **Canonicalization** normalizes the emitted Talon -- sorted keys, stable formatting -- so that semantic equality becomes textual equality, and you can diff two runs and actually trust the result.

Be honest about what this buys you: higher reproducibility, not a proof. And that is fine, because the proof was never supposed to live here.

## So -- is the pipeline deterministic?

At the **decision layer: yes, provably.** Same facts, same decision, every time, on any machine -- and every decision traces to the rule that fired and the facts that triggered it.

At the **authoring layer: no** -- and you do not need it to be. You need it reproducible and verifiable, and both are within reach.

The determinism does not come from taming the model. It comes from **architecture**: pin the decision core in a deterministic language, push the probabilistic model out to the edges where its output is a fact you can check rather than a verdict you must trust. The real prize is not that the pipeline repeats itself. It is that it can be **audited** -- and for anything touching medicine, money, or safety, an auditable decision beats a merely repeatable one every single time.

Deterministic where it matters. Probabilistic where it helps. That is the whole design.

## Further reading

- [Code Mode for MCP](/posts/code-mode-for-mcp/) -- let the model write the script, and let the expert system run it.
- [Expert in the Loop (EITL)](/posts/expert-in-the-loop/) -- why deterministic rules, authored once, beat a human rubber-stamping every decision.
- [LLM, Neural Network, or Expert System?](/posts/llm-neural-network-or-expert-system/) -- on the boundary between probabilistic and deterministic decisions.
- [Human-in-the-Loop and Vigilance Theory](/posts/human-in-the-loop-vigilance/) -- why "a human approves everything" is not the safety net it looks like.
- [opentalon-agents](https://github.com/opentalon/opentalon-agents) -- persistent, LLM-authored automations that run deterministically with no model in the loop.
- [Talon language](https://github.com/opentalon/talon-language) -- the small deterministic language behind all of this.
