---
title: "Enterprise AI Orchestration"
date: 2026-05-28
draft: false
tags: ["AI", "expert systems", "Tln", "OpenTalon", "LLM", "EITL", "enterprise"]
summary: "Managing AI agents in the enterprise means solving for security, knowledge access, and orchestration at the same time. A look at OpenTalon -- an EITL-first framework that combines LLMs with expert rules for corporate AI architecture."
ShowToc: true
---

Managing AI agents today is a reality -- and for large companies, a necessary one. Keeping documentation current, onboarding new employees or product customers, opening access to corporate knowledge and experience: all of this can be successfully addressed with AI agents.

But how do you protect the company from leaks of that information? How do you make AI an assistant rather than a hallucination generator that does more harm than good? These are the new questions and challenges of the moment.

Information security has become a goal that can easily put the entire employee enthusiasm for AI adoption on pause. Self-hosted LLMs make it possible to isolate information, but self-hosted -- and even open-source -- LLMs will always lag behind the market leaders.

I see corporate AI agents developing first and foremost within a paradigm of digital security, and also of knowledge about the company. You cannot stuff an entire corporate documentation into an LLM prompt -- that is not realistic today. RAG, of course, simplifies the task, but it adds complexity to orchestration.

There are now quite a few AI agent projects that let you easily customize them for your own use, with Human-in-the-Loop decision-making -- but most are aimed at a single end user. The best known of those is OpenClaw. The problem with these systems is their security: the access agents have to various plugins and skills, and the way that access is extended.

These problems can only be solved by a fundamental approach to building the AI architecture of a company -- something that can be used and extended without compromising security. And such a solution exists: the [OpenTalon framework](https://github.com/opentalon/opentalon).

## What OpenTalon Brings

OpenTalon is a framework for managing AI agents. Through plugins you can extend the tasks AI handles and add new functionality. OpenTalon puts security first -- every request to the LLM passes through a security check first -- and it lets you use any LLM model, both for security and for building business processes inside the company.

Channel-level authentication lets you identify Microsoft Teams, Slack, or other end users within your MCP service. Built-in RAG plugins let you populate knowledge and provide access in any language -- so if you are an international company, you do not need a human to translate that knowledge into other languages.

## Built on the EITL Principle

OpenTalon was built for corporate needs: using the terminology and knowledge of the company, opening access to information, and creating business processes. But the main feature of the framework is its commitment to the [Expert-in-the-Loop (EITL)](https://opakalex.github.io/posts/expert-in-the-loop/) principle. The combination of LLMs and expert rules is what makes OpenTalon a unique solution on the market.

Let's walk through an example using the [Tln language](https://tln-lang.org/). Imagine a logistics warehouse with a fleet of electric forklifts. Each one accumulates operating hours, gets serviced on intervals, has parts replaced over time. Technicians write up their work in plain text in whatever language they prefer. The question OpenTalon answers is: when should the next service happen, what parts need to be ordered, and which exceptions need a human eye?

```tln
// ── Facts: forklift baseline + service history ──────────

fact "asset.forklift_F12" {
  model = "Toyota 8FBE15U"
  commissioned = "2022-04-10"
  hours_operated = 4820
  last_oil_service_hours = 4500
  hydraulic_fluid_replaced_hours = 3000
  battery_installed_date = "2024-09-01"
  battery_type = "lead_acid_24V"
}

fact "service_baseline.electric_forklift" {
  oil_change_interval_hours = 500
  hydraulic_service_interval_hours = 2000
  battery_replacement_months = 36
  brake_inspection_interval_hours = 1000
}

// ── Conditions ──────────────────────────────────────────

define "oil_service_due" {
  "asset.hours_operated" - "asset.last_oil_service_hours"
    >= "service_baseline.electric_forklift.oil_change_interval_hours"
}

define "hydraulic_service_due" {
  "asset.hours_operated" - "asset.hydraulic_fluid_replaced_hours"
    >= "service_baseline.electric_forklift.hydraulic_service_interval_hours"
}

define "battery_end_of_life" {
  months_since("asset.battery_installed_date")
    >= "service_baseline.electric_forklift.battery_replacement_months"
}

// ── LLM as information provider ─────────────────────────

rule "Parse technician notes into structured events" {
  when "service_note.text" exists
  do llm_extract "service_note.text" into "event.parts_replaced"
  do llm_extract "service_note.text" into "event.symptoms"
  do llm_extract "service_note.text" into "event.duration_minutes"
  do append "asset.service_log" "event"
}

// ── Predictive maintenance — expert system decides ──────

rule "Schedule oil service before threshold" {
  when is "oil_service_due"
  do create_work_order "asset.id" "oil_change"
  do book_slot "maintenance.next_available"
  do notify "team.fleet_ops" "work_order.id"
}

rule "Schedule hydraulic service and reserve parts" {
  when is "hydraulic_service_due"
  do create_work_order "asset.id" "hydraulic_fluid_replacement"
  do reserve_parts ["hydraulic_fluid_5L", "filter_kit"]
}

rule "Replace battery before failure" {
  when is "battery_end_of_life"
  do create_work_order "asset.id" "battery_replacement"
  do order_part "asset.battery_type"
  do notify "team.warehouse_manager" "Forklift {asset.id} battery at end of expected life"
}

// ── Anomaly detection — escalate to a human ─────────────

rule "Unusual service frequency" {
  when count("asset.service_log", last_days = 90) > 4
  do require "review.fleet_supervisor"
  do explain "Asset has had {count} service events in the past 90 days — investigate root cause"
}

// ── LLM drafts, expert system verifies, then sends ──────

rule "Draft service summary, verify facts, then send" {
  when "work_order.status" == "completed"
  do llm_draft "summary" from "work_order.events" tone "concise"
  do verify "summary" against "work_order.facts"
  do send "summary" to "team.warehouse_manager" only_if "verify.passed" == true
}
```

Notice what each side does. The LLM does what LLMs are actually good at: reading messy technician notes in any language, pulling out which parts were replaced and which symptoms were reported, drafting a clear summary for the warehouse manager. The expert system does what *it* is good at: checking thresholds against facts, deciding when to schedule, when to order parts, when to escalate to a human, and refusing to send any message that contradicts the recorded service history.

![Expert-in-the-Loop (EITL) architecture: the user request flows through an LLM information provider, then an expert system decision-maker that calls external workflows via MCP tools, with a feedback-and-learning loop closing the cycle](/images/eitl-architecture.png)

The EITL principle is the key to a sensible use of knowledge and action, and to the effective allocation of company resources.
