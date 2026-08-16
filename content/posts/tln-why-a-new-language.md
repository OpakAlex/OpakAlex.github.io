---
title: "Tln: Why a New Language?"
titleHtml: '<a href="https://tln-lang.org/">Tln</a>: Why a New Language?'
date: 2026-08-16
draft: false
tags: ["AI", "expert systems", "Tln", "Prolog", "LLM", "logic programming"]
summary: "The world moves faster than it did in 1995. Prolog gave us a beautiful way to reason over facts, but it was built for a world without LLMs, without live data feeds, without MCP. Tln is a modern reimagining of the expert-system language -- deterministic, testable, data-native, and built to sit next to a language model rather than pretend one doesn't exist."
ShowToc: true
---

The world moves faster than it did in 1995. We have new languages, new frameworks, and -- above all -- LLMs. A developer starting today reaches for TypeScript, Rust, or Go without a second thought, deploys to the edge in minutes, and asks a model to draft the first version of almost anything. The tooling around us has been rebuilt, layer by layer, for a world that streams, scales, and increasingly *thinks* out loud.

And yet the moment you need a machine to make an actual **decision** -- a real, auditable, defensible decision -- you fall back to tools that were essentially finished before the web existed. Expert systems. Rule engines. And underneath many of them, one language that has quietly outlived nearly everything around it: Prolog.

## What Prolog Got Right

Prolog is old, and that is not an insult. It is old the way a good theorem is old. Born in 1972, standardized as ISO/IEC 13211-1 in 1995, it encoded an idea that still has no better expression: you describe *what is true*, and the machine figures out *what follows*.

You don't write a loop. You don't manage state. You state facts and rules, and the resolution engine does the reasoning:

```prolog
parent(tom, bob).
parent(bob, ann).

grandparent(X, Z) :- parent(X, Y), parent(Y, Z).

?- grandparent(tom, ann).
true.
```

That declarative core is genuinely powerful, and it has quietly run serious systems for decades:

- **Medical diagnosis and decision support** -- MYCIN's descendants, drug-interaction checkers, clinical guideline engines.
- **Fraud and compliance** -- rule sets that flag suspicious transactions, enforce KYC/AML policy, and never "forget" a regulation the way a fine-tuned model might.
- **Configuration and scheduling** -- airline crew rostering, telecom network configuration, product configurators where thousands of constraints must all hold at once.
- **Natural-language processing** -- parsers and grammars, back when NLP meant rules rather than weights.
- **Aerospace and industrial control** -- the layer of expert rules that sits on top of the math and the physics, deciding what the system should actually *do*.

None of this is legacy trivia. These systems still run. They pay your claims, approve your credit, and keep planes in the air. They are just invisible, because you can't have a conversation with them and they don't hand their creators a feeling of divine omniscience.

## What Prolog Never Anticipated

Here is the problem. Prolog was designed for a world that no longer exists.

It assumed the facts were already in the database, hand-curated by a knowledge engineer. It assumed the program was the whole system, not one deterministic core wired into a dozen live services. It assumed nobody would ever need to *generate* rules from a vague sentence typed by a warehouse manager. It had no notion of a tool call, an HTTP endpoint, a streaming event, or a language model sitting one hop away, eager to author facts for it.

In practice this means that whenever you try to use classical Prolog for a modern decision system, you spend most of your time on everything *around* the logic:

- Hand-rolling queries and formatting output for humans.
- Bolting on external orchestration to feed it live data.
- Writing glue to expose it as a service.
- Testing it with whatever ad-hoc harness you could improvise, because a first-class test story was never part of the deal.
- And, now, awkwardly trying to connect it to an LLM that could do 80% of the fact-gathering if only the language knew how to ask.

The reasoning core is timeless. The world it was built for is gone.

## Tln: A Modern Reimagining

[Tln](https://tln-lang.org/) is a contemporary reimagining of Prolog for today's computing landscape. It keeps the part that was always right -- declarative rules over facts, deterministic resolution, full explainability -- and rebuilds everything Prolog was missing.

The design goals are deliberately unfashionable in an era of "just ask the model":

- **Deterministic.** Same facts, same rules, same decision -- every time, on any machine, at any hour. A decision cannot rest on sampling.
- **Testable.** A first-class testing framework (`.tln.test`, with a `given` / `when` / `expect` structure) is part of the language, not an afterthought.
- **Data-native.** Rules evaluate against live data through a native Go engine, not a hand-loaded knowledge base.
- **Readable by experts.** Rules read close to natural English, so the people who actually own the domain -- underwriters, procurement leads, senior engineers -- can read and sign off on them.
- **Built for the AI era.** Native Model Context Protocol (MCP) tool orchestration, reactive workflows, and explainable ML primitives (`predict`, `forecast`, `classify`, `cluster`, similarity matching) are in the box.
- **Real language ergonomics.** Compile-time metaprogramming with Elixir-style macros, and a package/bundling system via `mod.tln` manifests.

Where Prolog assumed a lone program, Tln assumes a pipeline: an LLM turns messy natural language into structured facts, and Tln makes the deterministic, auditable call on top of them. The model does what it is good at -- language. Tln does what *it* is good at -- deciding.

## The Same Idea, Modernized

Compare the two directly. Here is the classic Prolog approach -- expressive, but you are hand-rolling the query and orchestrating everything else yourself:

```prolog
active_vehicle(E, Id) :- record(E, Id, item, 'Vehicles', active, _).
```

And here is the same intent in Tln -- you declare the *outcome*, and the engine produces a deterministic query plan that is testable and ready to escalate over MCP:

```tln
define "active_vehicle" {
  type == "item" and status == "active" and category == "Vehicles"
}

detect "Service overdue" {
  for records where is "active_vehicle"
    and attr "km" > attr "last_service_km" + 20000
  flag matching items
  label "{item.name}: {attr.km} km since last service"
}
```

You are no longer scripting *how* to walk the data. You are stating *what* counts as overdue service, and letting the engine plan the rest. That shift -- from implementation to outcome -- is exactly what made Prolog compelling in the first place. Tln just carries it forward into a world with live fleets, MCP tools, and an LLM ready to fill in the facts.

## A Small End-to-End Taste

Facts, named conditions, and rules -- the whole model fits in your head. Here is a compact example: a fleet of service vehicles where an LLM has already parsed the technician's free-text notes into facts, and Tln makes the call.

```tln
fact "forklift_A12" {
  category       = "Vehicles"
  status         = "active"
  km             = 84200
  last_service_km = 61000
  last_service_days = 190
  open_faults    = ["hydraulic_leak_minor"]
}

define "active_vehicle" {
  status == "active" and category == "Vehicles"
}

define "service_overdue" {
  attr "km" > attr "last_service_km" + 20000
    or attr "last_service_days" > 180
}

define "has_safety_fault" {
  attr "open_faults" not_empty
}

rule "Schedule routine service" {
  when is "active_vehicle"
    and is "service_overdue"
    and not is "has_safety_fault"
  do schedule_service "item.id" within "14d"
  do explain "{item.name} is {attr.km} km / {attr.last_service_days} days since last service"
}

rule "Escalate vehicles with open safety faults" {
  when is "active_vehicle"
    and is "has_safety_fault"
  do hold_vehicle "item.id"
  do require "review.fleet_safety"
  do explain "{item.name} has open faults: {attr.open_faults}"
}
```

The LLM never decides whether the forklift keeps running. It only reads the technician's note and produces the `fact` block. The decision -- service, hold, or escalate -- belongs to rules a fleet safety lead can read, test, and defend. That is the division of labor Tln is built for.

And because the testing story is first-class, you can pin that behavior down before it ever touches production:

```tln
test "overdue vehicle with a safety fault is escalated, not just serviced" {
  given fact "forklift_A12"
  when evaluate
  expect fired "Escalate vehicles with open safety faults"
  expect not fired "Schedule routine service"
}
```

## Why Now

For twenty years there was no strong reason to reinvent Prolog. The world of curated knowledge bases and hand-written queries was slow, but it was stable. LLMs changed that overnight. Suddenly there is an abundant, cheap source of *facts* -- a model that can read an invoice, a service note, a support ticket, or a contract and turn it into structure. What was missing was never the ability to reason over facts. Prolog nailed that in the seventies. What was missing was a reasoning language built to *stand next to* that model: to accept its facts, refuse its bad ones, decide deterministically, and explain itself to an auditor.

That is the gap Tln fills. Not a rejection of Prolog -- an inheritance of it, dragged forward fifty years into a world of LLMs, live data, and MCP.

If you want to go deeper, the language, its grammar, and the tooling live at [tln-lang.org](https://tln-lang.org/).
