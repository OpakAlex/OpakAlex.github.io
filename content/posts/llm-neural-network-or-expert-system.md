---
title: "LLM, Neural Network, or Expert System?"
date: 2026-05-27
draft: false
tags: ["AI", "expert systems", "Tln", "LLM", "neural networks"]
summary: "Each kind of system makes a different kind of decision. LLMs generate language, neural networks find patterns, expert systems make the call. A look at why the boundary matters -- and how the real world runs on it."
ShowToc: true
---

LLMs, artificial neural networks, or expert systems? An excellent question with no single answer. Each of us has, at some point, felt like an all-knowing "god" by opening a paid LLM tool -- ChatGPT, Claude, Cursor, Gemini, take your pick -- and asking about our health, software architecture, or what to wear on a date. A familiar scene? The catch is that each of these systems can give radically different "advice."

If we take LLMs as the baseline, their job is to generate coherent, logical text. Finding patterns and recurring regularities, analyzing images, and so on -- these are tasks of artificial neural networks. So what is left for expert systems?

In my view, expert systems should make decisions: based on the knowledge supplied by LLMs and the patterns surfaced by neural networks. Making a decision is part of the final work of any system. Of course, one could argue that LLMs can also make decisions -- and that's partly true. But would we trust an LLM to fly the plane during our vacation? I think the answer is unequivocally no. And yet 90% of flights are conducted on autopilot. Who is flying? Who is deciding? The obvious answer is math, calculations, physics -- but that's only 50% of the truth. The other 50% is expert knowledge, i.e., rules. How does a typical expert system work? It's a set of rules and decisions applied at a specific moment, based on specific data.

## Who Actually Makes the Decision?

Let me return to the author's definition of what expert systems should do. Of course, anyone with 10+ LLM chats open at once will tell you that *they* are the one making the decisions. True. But what kind of decision? The decision of what to wear on a date? The decision of which pill to take at night? The decision of what to deploy to production?

These are radically different decisions. And as my experience tells me, decisions on critical matters -- or questions that touch the business -- are not made by a human with LLM suggestions. They are made by experts in the field. Otherwise, the business would incur enormous losses.

At this stage of computer engineering and the LLM boom, people have forgotten that many systems have lived hand in hand with us for decades. They help us get to work, pay our bills, prevent credit card fraud. These systems exist. They are niche, less visible, you can't hold a conversation with them the way you can with an LLM, they don't bestow "divine knowledge" on their creators -- but without them there would be chaos.

The deeper I go into the world of expert systems, the more I understand the weight of responsibility for decision-making -- and the philosophical pragmatism of delegating that responsibility to a machine.

## Why the Decision Was Made

A major benefit of an expert system is the ability to understand *why* a particular decision was made. It is not an LLM hallucinating about whether a mushroom is edible or not, with the answer shifting on the next retry.

![AI confidently telling a human a poisonous mushroom is edible, then apologizing after the human is dead](/images/ai-mushroom-meme.jpeg)

An expert system, by contrast, knows exactly which facts it is reasoning from. It does not improvise. It looks the mushroom up.

![Expert system explaining the natural toxins in a mushroom -- muscarine, ibotenic acid, muscimol -- and the cooking conditions that neutralize them](/images/expert-system-mushroom-facts.png)

Those facts live in the system as data, not as vibes:

```tln
fact "amanita_muscaria" {
  common_name = "Fly Agaric"
  edible = "conditional"
  toxins = ["muscarine", "ibotenic_acid", "muscimol"]
  heat_sensitive = true
  safe_when = "boiled or sautéed for 15+ minutes"
  warning = "Do not eat raw or undercooked"
}

rule "Refuse to recommend without identification" {
  when "mushroom.species" == "unknown"
  do block "consumption"
  do explain "Species must be 100% identified before consumption"
}

rule "Block unsafe preparation" {
  when "mushroom.toxins" not_empty
    and "preparation.heat_treated" == false
  do block "consumption"
  do explain "mushroom.warning"
}
```

An expert system can always justify its choice -- it can point to the exact rule that fired, the exact facts that triggered it, the exact thresholds that were crossed.

And that is what decision-making really is. A decision cannot rest on randomness. It must be backed by facts.

## A Concrete Example: Warehouse Replenishment

Let's look at how an expert system behaves in a concrete decision: replenishing a warehouse. The example below uses the [Tln language](https://github.com/opentalon/tln-language).

Imagine an online retailer with thousands of SKUs flowing through a distribution center. The system watches stock levels, incoming orders, supplier availability, and seasonal patterns. When inventory dips below a threshold, the system decides on its own whether to place an order, with which supplier, for what quantity -- and when to pull a human into the loop.

```tln
define "low_stock" {
  "item.quantity_on_hand" < "item.reorder_threshold"
}

define "critical_stock" {
  "item.quantity_on_hand" < "item.safety_stock"
}

define "demand_spike" {
  "item.orders_last_7_days" > "item.weekly_average" * 1.5
}

define "supplier_available" {
  "supplier.preferred.in_stock" == true
    and "supplier.preferred.lead_time_days" <= "item.max_lead_time"
}

define "perishable" {
  "item.shelf_life_days" < 30
}

define "seasonal_peak" {
  "calendar.month" in ["11", "12"]
    and "item.category" == "holiday"
}

// ── Routine reorder ─────────────────────────────────────

rule "Reorder when stock falls below threshold" {
  when is "low_stock"
    and not is "critical_stock"
    and is "supplier_available"
  do calculate "order.quantity" "item.economic_order_quantity"
  do create_order "supplier.preferred" "item.sku" "order.quantity"
  do notify "team.procurement" "order.id"
}

// ── Emergency reorder ───────────────────────────────────

rule "Emergency reorder when stock hits safety level" {
  when is "critical_stock"
  do calculate "order.quantity" "item.economic_order_quantity" * 2
  do create_order "supplier.fastest" "item.sku" "order.quantity"
  do notify "team.warehouse_manager" "item.sku"
  do comment "order" "Stock below safety threshold — expedited supplier selected"
}

// ── Demand-driven adjustments ───────────────────────────

rule "Scale order up on demand spike" {
  when is "demand_spike"
    and is "low_stock"
  do calculate "order.quantity" "item.economic_order_quantity" * 1.5
  do create_order "supplier.preferred" "item.sku" "order.quantity"
  do comment "order" "Demand exceeded weekly average by 50%+"
}

// ── Supplier selection ──────────────────────────────────

rule "Switch supplier when preferred is unavailable" {
  when is "low_stock"
    and "supplier.preferred.in_stock" == false
  do select_supplier "item.sku" by "lowest_price_within_lead_time"
  do create_order "supplier.selected" "item.sku" "order.quantity"
  do comment "order" "Preferred supplier out of stock — switched to alternate"
}

// ── Perishables — no overstock ──────────────────────────

rule "Limit perishable orders to forecast window" {
  when is "low_stock"
    and is "perishable"
  do calculate "order.quantity" "item.forecasted_demand_14_days"
  do create_order "supplier.preferred" "item.sku" "order.quantity"
}

// ── Seasonal stocking ───────────────────────────────────

rule "Pre-stock seasonal items before peak" {
  when is "seasonal_peak"
    and "item.quantity_on_hand" < "item.seasonal_target"
  do calculate "order.quantity" "item.seasonal_target" - "item.quantity_on_hand"
  do create_order "supplier.preferred" "item.sku" "order.quantity"
  do notify "team.procurement" "order.id"
}

// ── Human gates ─────────────────────────────────────────

rule "Require approval for unusually large orders" {
  when "order.total_value" > 50000
  do hold_order "order.id"
  do require "approval.warehouse_director"
  do comment "order" "Order value exceeds $50,000 — director approval required"
}

rule "Flag suspicious demand patterns" {
  when "item.orders_last_24_hours" > "item.daily_average" * 5
  do hold_order "item.sku"
  do require "review.fraud_team"
  do comment "item" "Unusual order velocity — possible fraud or pricing error"
}
```

The LLM cannot run this. A neural network alone cannot run this. What runs the warehouse is a set of explicit, auditable rules written by people who understand procurement, lead times, and cash flow.

The LLM may help draft those rules, summarize a supplier contract, or explain a demand spike in plain language. The neural network may forecast next month's demand based on historical patterns. But the *decision* -- whether to place a $20,000 order at 2 AM on a Sunday -- belongs to the expert system. And that is precisely where it belongs.

## The Boundary

LLMs are probabilistic. They predict the most likely next token, not the correct answer. Neural networks find patterns in data, but they don't know what those patterns mean for the business. Expert systems are deterministic: given the same inputs and rules, they produce the same output, every time, at any hour, under any load.

The mistake of the current LLM boom is treating language models as if they could replace the layer underneath -- the layer of rules, constraints, and accountability. They can't. They were never built to. The right architecture is to let each tool do what it's actually good at: language to the LLM, patterns to the neural network, and decisions to the expert system.
