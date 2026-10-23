# Osysharp.Shop.Agents — the shop's agents

## Use case

Writing a good description for every product is the job a shop owner puts off. With this package, "Draft a
description" on a product in the desk asks an agent for one, written from what the product already says: its name,
brand, category, sizes, colours, spec sheet and its current words. The draft is offered in the product editor. You
read it, take it into the description, change what you like and save — or discard it. Nothing it writes is used by
itself.

## What it ships

| | |
|---|---|
| **The description writer** | `DescriptionWriter`, an agent with one skill (`WriteDescription`). It holds no tools and no role: it is told the product's details, read with the asking staff member's own access |
| **Offered, never used** | its draft is the shop's `OfferedDescription`, shown under "Prepared by Description writer" with "Use it" and "Discard" |
| **Off until switched on** | the desk's Settings switch it on and off; whoever switches it on answers for it, and the settings show the last 30 days of drafts and what they cost |
| **A capped cost** | one draft may spend at most 5 cents; every draft leaves a receipt with its cost at the model's list price |

## Install and use

It arrives by itself in an app that uses `Osysharp.Shop` and `Osysharp.Agents` (`osy lock` says so). The app lists
the writer, gives it a model, and says who uses and supervises agents:

```osy
app.Agents = new AgentsSetup { Agents = [new DescriptionWriter()] };
policy UsesAgents       => IsShopStaff;
policy SupervisesAgents => IsShopStaff;
```

The model is the app's `app.Llm`; declare its `Price` so each receipt says what a draft cost.

## Source and tests

`model/DescriptionWriter.osy`; the tests in `tests/` script the model, so no model is ever called.
