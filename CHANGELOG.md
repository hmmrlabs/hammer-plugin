# Changelog

This plugin ships an MCP declaration and one skill. It ships no corpus and no binary, so
the world it points at changes independently of the version numbers below. Where a release
matters mainly because the world moved, this file says so.

## 0.4.1, 2026-09-15

**The skill stops carrying a count that tracks a world.**

`SKILL.md` said `resources/list` returns **seven** on the hosted coverage world. It now
returns eight, because that world gained a skill.

That sentence had already been corrected once, on 2026-08-30, for having gone false the
same way. Twice in two weeks. So this correction removes the number instead of updating
it: the text now says one per skill and tells the reader to ask the world, which answers in
one call.

The count was never a fact about this plugin. It ships no corpus and no per-world
documentation by design, so any number in it that tracks a world goes stale the next time
any world is minted, and nobody is looking.

Nothing else changed. Same MCP declaration, same URL, same skill.

### What did change is on the other side of the URL

`coverage-intelligence` moved from `2026-09-03+3` to `2026-09-13+1`, and if you use that
world this is the part that affects you:

| | before | after |
|---|---|---|
| billing codes | 12 | **115** |
| payers | 7 | **11** |
| hospital filers | 0 | **7** |
| filed-charge rows | 0 | **3.5 million** |

Hospital price-transparency filings under 45 CFR 180 are new, with the methodology field
that says whether a figure is a case rate, a fee schedule or a per diem. Two new tools
read them: `filed_charge` and `filer_inventory`.

Tool descriptions also got much shorter. Each is now a stub naming its refusal count, with
the full text served by `hello_world` on request. On the previous build prose was about 70
percent of the tool listing and every caller paid for all of it on connect, before asking
anything.

**Nothing to do to get it.** The plugin is a pointer. Reconnect the MCP server if your
client caches its tool list.

## 0.4.0

**Five tools for the caller whose input is a range rather than a number.**

The skill gained the simulation family the runtime mounts into every world: `simulate`,
`sensitivity`, `interval_check`, `decision_navigate` and `network_infer`, with the four
things that otherwise cost a call to learn.

Everything they serve is an assumption, however the underlying estimator answered, because
arithmetic over your own declarations is not an observation. A cohort is its recipe rather
than a saved object, which is what makes "these two figures are about the same trials" a
fact rather than a label. A pinned input carries no index, because held-fixed and
varied-but-inert would both print as zero. And an edge is a factorisation rather than a
cause, so the interventional names are refused by name.

## 0.3.0

**A route for a caller holding a goal, and skills fetched from the build that answers you.**

Three additions, each from a measured failure rather than an imagined one.

A goal is not a question, and every checker was right about that on the way to a dead end.
Measured 2026-08-30: somebody said a clinical guideline would let hospitals make more
money. `claim_check` returned `undefined`, because the sentence names no declared figure.
`judgement_check` returned `forbidden`, correctly, because that corpus holds no evidence a
guideline changes revenue. Every answer was right and their sum told the person only what
they may not write. The skill now names that tell, and routes to `frontier`.

Skills are served by the world from the same image that answers your tool calls, rather
than shipped here, so routing advice and answers cannot drift apart.

## Earlier

Releases before 0.3.0 predate this file. Their history is in the repository log.
