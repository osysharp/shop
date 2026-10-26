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

## Examples

Taken from this package's own tests, which compile and run on every release.

```osy
using Osysharp.Identity;
using Osysharp.Shop;
using Osysharp.Shop.Agents;
using Osysharp.Llm;
using Osysharp.Agents;
using Osysharp.Agents.Testing;

[Test(Shop)]
void a_draft_the_model_could_not_write_says_what_to_do_in_plain_words_and_offers_nothing() {
  // The model answers in words and never hands a description back.
  ScriptedAnswer.Stub((model, request) => Says.Words("I would rather not."));
  runas (Anna) {
    TurnProductWriterOn();
    var coat = Product.Single(p => p.Slug == "harbour-coat");
    Assert.Equal("The description writer could not write a draft this time. Try again, or write it yourself.",
                 DraftDescription(coat).Problem);
    Assert.Null(OfferedDescriptionOf(coat));
  }
}
```

From [`tests/descriptions.test.osy`](tests/descriptions.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.Shop;
using Osysharp.Shop.Agents;
using Osysharp.Llm;
using Osysharp.Agents;
using Osysharp.Agents.Testing;

[Test(Shop)]
void a_draft_is_held_to_five_cents_whatever_the_apps_own_budget() {
  // The first answer reads 60,000 tokens at $1 a million — 6 cents — and only the second would hand a draft back. The
  // app's own budget (50 cents) would let it; the writer's 5 cents stops it before the second answer is asked for.
  ScriptedAnswer.Stub((model, request) => AgentScript.Of(
      Says.Words("Let me think about the coat.", input: 60000),
      Says.HandBack(new WrittenDescription { Description = CoatDescription() })).Answer(model, request));
  runas (Anna) {
    TurnProductWriterOn();
    var coat = Product.Single(p => p.Slug == "harbour-coat");
    Assert.Equal("The description writer could not write a draft this time. Try again, or write it yourself.",
                 DraftDescription(coat).Problem);
    Assert.Null(OfferedDescriptionOf(coat));
    Assert.NotEqual(AgentTaskStatus.Done, AgentTask.Single().Status);
  }
}
```

From [`tests/descriptions.test.osy`](tests/descriptions.test.osy).

```osy
using Osysharp.Identity;
using Osysharp.Shop;
using Osysharp.Shop.Agents;
using Osysharp.Llm;
using Osysharp.Agents;
using Osysharp.Agents.Testing;

[Test(Shop)]
void only_staff_ask_for_a_draft_or_switch_the_writer_on() {
  ScriptTheWriter();
  runas (Maja) {
    Assert.Throws(() => TurnProductWriterOn());
    Assert.Throws(() => DraftDescription(Product.Single(p => p.Slug == "harbour-coat")));
  }
  runas (Anna) { Assert.False(ProductWriterIsOn()); }
}
```

From [`tests/descriptions.test.osy`](tests/descriptions.test.osy).

## Learn more

- [The Osysharp.Shop.Agents reference](https://osysharp.com/reference/agent/kit-shop-agents/)
- [Osysharp.Shop.Agents in the package catalogue](https://osysharp.com/packages/#osysharp-shop-agents)
- [The Osy# reference](https://osysharp.com/reference/) — the language, the runtime and every kit
- [`llms-full.txt`](https://osysharp.com/llms-full.txt) — the whole reference and specification as one Markdown file, for a coding agent

## Licence

MIT — see [`LICENSE`](LICENSE).
