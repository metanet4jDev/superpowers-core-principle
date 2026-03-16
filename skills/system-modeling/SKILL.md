---
name: system-modeling
description: Use when brainstorming, clarifying requirements, modeling domains, analyzing business rules, or evaluating behavior changes when entities, relationships, attributes, state transitions, or functional boundaries are unclear or easy to skip.
---

# System Modeling

## Overview

Build core understanding before proposing solutions.

**Core principle:** Keep the model short, precise, and verifiable. Understand the system in a fixed order so time pressure does not push you straight into ideas, implementation, or conclusions.

**Alias note:** This is the English-search alias of `core-principle`. Use the same 7-step cognition order.

## When to Use

**Use when:**
- brainstorming system architecture, features, components, or behavior changes
- clarifying requirements before design or implementation
- modeling entities, business rules, state transitions, or domain boundaries
- a request sounds simple but the system rules are easy to misread

**Do not use when:**
- looking up a command, API, or syntax reference
- doing a purely mechanical change with no system understanding cost

## Core Pattern

Always model the system in this order. Do not skip or reorder steps.

| Order | Dimension | Question to answer |
| --- | --- | --- |
| 1 | Structure | How are the system, subsystems, modules, and components organized? |
| 2 | Classification | What are the key entities and categories? |
| 3 | Relationships | How do those entities depend on or constrain each other? |
| 4 | Attributes | What key fields matter: type, uniqueness, mutability, requiredness, length, precision? |
| 5 | States | What are the initial, intermediate, and final states? How do transitions work? |
| 6 | Functions | What capabilities exist and what purpose does each serve? |
| 7 | Boundaries | What changes to attributes/states does a function cause, and what conditions make a function available or unavailable? |

If one step is unclear, stop and clarify it before moving on.

## Quick Reference

Use this minimal scaffold before offering design advice:

```text
Structure:
Classification:
Relationships:
Attributes:
States:
Functions:
Boundaries:
```

Boundaries must cover both:
- what a function changes
- what state or attribute conditions limit that function

## Implementation

- Prefer extracting understanding from code, naming, directory structure, and existing state flows
- Do not create extra documents if the code already expresses the truth
- Keep the output short, exact, purpose-driven, and independently checkable

### Example

```markdown
Structure: Billing manages payment state, Orders manage shipping eligibility, Fulfillment only executes released shipments.
Classification: Order, Payment, Shipment Instruction.
Relationships: Payment state affects order state; order state decides whether shipment can be created.
Attributes: Payment has amountDue, amountPaid, status; Order has shippingEligibility.
States: unpaid, partially_paid, paid, refunding, refunded.
Functions: confirm payment, evaluate shipping, process refund callbacks.
Boundaries: partial payment updates paid amount but must not enable shipping; only paid orders can ship; refunding or refunded orders block shipping.
```

## Common Mistakes

| Mistake | Fix |
| --- | --- |
| Jumping straight to solutions | Write the 7-dimension model first |
| Listing features without entities or states | Fill in classification, attributes, states, and boundaries |
| Treating boundaries as vague risks | State what changes and what blocks availability |
| Writing extra docs before reading code | Extract the model from code first |
| Skipping modeling because time is tight | Time pressure is exactly when fixed order matters |

## Red Flags

- "Let's give the solution first and clarify later"
- "Keep it concise, so we can skip states and boundaries"
- "I basically understand the system already"
- "Let's list functions first and fill the model later"

These mean you are skipping core understanding. Return to the 7-step order.
