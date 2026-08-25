---
name: hammer-worlds
description: Use when somebody says "using Hammer Worlds" or asks what hammer worlds are mounted, when routing a question through the mounted worlds and deciding which parts no world can answer, when writing up any answer built on a hammer world, choosing between claim_check and judgement_check, or installing, verifying, listing or serving a world, wiring one into an MCP client, debugging a hammer MCP server that will not start, or reading a refusal. Carries how to find out which worlds are mounted and how to choose among them, that two worlds' answers never compose into a third claim, the required claim report every such answer must close with, why claim_check is preferred and why neither check falls back to the other, plus where a world runs and what crosses the wire, that a hosted world serves its own routing skill from the build that answers you, user scope versus project scope, what hammer verify did not check, and the build bump that cannot be installed over its own vintage.
---

# hammer-worlds

## Using Hammer Worlds

"Using Hammer Worlds, investigate X" is a request to route X through whatever
worlds this client has mounted, and to refuse the parts none of them can answer.
Three steps, in order, and the third is the one that gets skipped.

**1. Find out what is mounted. There is no list in a file to read.** A world is
mounted when the client has an MCP server for it, so the mounted set is whatever
servers are connected right now. Inspect the client's MCP connections or read
the tool names you were given. In Claude Code, `/mcp` opens that view. In
Cowork, open the installed plugin's Connectors page. One server per world and
the server's name is the world's name, so a tool
arriving as `mcp__noise__noise_audit` is the noise world and one arriving as
`mcp__fda__claim_check` is the fda world. A server declared by a plugin is
namespaced by the plugin too, so the same tools arrive as
`mcp__plugin_<plugin>_<world>__<tool>`. Either way the segment before the final
`__` is the world.

The plugin's `.mcp.json` is the list of worlds it **offers**, which is not the
same set. A static file cannot enumerate what a client has connected. An entry
whose server did not start, did not reach its host or was never authenticated is
not mounted.

**Do not answer from memory about which worlds exist.** The set differs per
machine and per install, and a world you remember having and do not have is
exactly the case that ends in hand arithmetic presented as a finding.

**2. Call `hello_world` on each world you are about to use, before spending a
call on anything else.** Every world serves it, no world declares it, and it
takes no argument. One call hands back that world's identity and what a receipt
is redeemed against, which question goes to which of its tools with each tool's
closed vocabularies, what it refuses and why, which `knowledge_navigate` anchors
actually resolve in that build, and the claim discipline. All of it is read from
that world's own declarations at the moment you call, so it cannot disagree with
the tools listed beside it.

That last property is why this step is here rather than "read the world's skill".
A skill is written once and goes stale: one shipped here named fewer than half
its world's tools for a month and told callers a question was unserved while the
tool answering it was already listed. `hello_world` cannot do that, because there
is nothing in it to forget to update.

Every world's `cannot_answer` list also arrives in the MCP `initialize` response
as the server's instructions, and every tool carries its own `refusals` array. So
the question can be matched against the worlds before any tool runs.

**3. Split the question, and say which parts have no world.** A world that
refuses the question is information, not a dead end: it names what it cannot
answer and often what would settle it. The honest answer to a five-part question
is frequently that two parts have a world and three do not, and **that is a
result, not a failure to deliver one.** Name the three.

Do not fill the three from your own knowledge inside an answer whose credibility
comes from the worlds. A reader cannot tell the two halves apart and will read the
whole thing as world backed. If nothing mounted bears on the question at all, say
so and say what is mounted. "No world here answers this" is shorter than the
answer built by guessing, and it is the true one.

## Two worlds' answers are two answers

**Worlds do not compose.** Nothing licenses combining two worlds' figures into a
third claim, and nothing in this harness will catch you doing it.

The mechanism is the receipt. Each world issues its own receipts against its own
payloads, and `claim_check` and `judgement_check` are per server: a claim goes to
the world whose receipt it names and is checked against that payload alone. A
claim resting on a figure from noise and a figure from fda names two receipts,
belongs to neither world, and **has no receipt that covers it.** There is no
cross-world check, no cross-world payload, and no verdict that could come back
`entailed`, because there is nothing there to entail it.

So a ratio, a difference, a correlation or a "therefore" spanning two worlds is
your own arithmetic. It may well be worth writing. It goes under `asserted
without a check` in the claim report, in words, naming both worlds and both
receipts, and it is never presented as something a world said.

This is the obvious next mistake as soon as more than one world is mounted, and
it is not hypothetical: two payloads on one screen look like one dataset.

**Each mounted world's answer gets its own claim report**, or its own block
inside one report, carrying its own world id, its own session and its own
receipts. One merged report across two worlds loses what the report is for, which
is that a reader holding a log can find which answers came from which world.

## What a world is

A directory. `world.toml` carries identity, vintage, build and a published
`cannot_answer` list. `tools/tools.json` declares each tool's name, the
estimator that answers it, a description, a JSON Schema and a plain-language
`refusals` array.

**A world contains no executable.** Data, knowledge, models, rules, prompts,
skills and tools are all declarations. The estimators they name are compiled
into `hammerd`, so correctness is not per-world and two worlds cannot disagree
for reasons nobody can trace.

A world is identified as `name:vintage`, for example `noise:2026-08-14`. The
build number is not part of the id. Hold onto that, it causes the defect below.

## Where a world runs, and what crosses the wire

A world reaches a client over MCP by one of two transports, and which one you
have changes what can go wrong rather than what the world will say.

**Hosted.** The server is a URL and the world runs on whoever operates it. The
entry is a URL and a type, and there is nothing to build, install or keep in
step:

```json
{
  "mcpServers": {
    "coverage-intelligence": {
      "type": "http",
      "url": "https://worlds.hammer.ai/mcp"
    }
  }
}
```

**Local.** The server is a process on your machine, launched from a package on
your disk. That is the rest of this skill: `hammer install`, `hammer serve`, the
store, the build-bump defect and the resolver. None of it applies to a hosted
world, and a hosted world has none of those failure modes to debug.

Three things about the hosted transport that are worth knowing before the first
call.

**Authentication is OAuth and there is no token to paste.** The first call opens
a browser. Measured 2026-08-24 against `https://worlds.hammer.ai/mcp`: an
unauthenticated POST answers `401` carrying
`www-authenticate: Bearer resource_metadata="https://worlds.hammer.ai/.well-known/oauth-protected-resource"`,
and that document names the authorisation server. **If anything asks you to put
a key in a file, you are not talking to this service.**

**Authentication is not entitlement.** Signing in says who is calling. It does
not open every compartment to every caller, and a world may answer some tools
and decline others for the same authenticated caller. Where a world runs an
entitlement flow it declares tools for it, `request_access` and `redeem_code` on
the coverage world, and a refusal for want of entitlement is a refusal with a
remedy rather than an outage.

**A hosted world serves its own skills, and that is the point.** `resources/list`
on the server returns one resource per skill in the world's `skills/`
compartment, at `hammer://<package>/skills/<skill>`, and `resources/read` returns
that `SKILL.md` out of the deployed image. So the routing instructions for a
world come from the same build that answers its tools. **Read them from the
server rather than from anywhere else.** A copy of a world's skill kept beside
the world is a second thing to keep in step, and the copy a reader meets first is
the one that went stale.

**What the client sends is the tool call and nothing else.** The arguments are
what you put in them. No file is read off your disk and uploaded, because a
declaration of a URL has no code that could read one. What the operator keeps is
on their side: every answer is a receipt and every claim checked against one is
logged with its verdict, which is what makes answers taken and never checked
countable rather than assumed.

## Running a world yourself: the commands

These are for a world you run on your own machine. A hosted world needs none of
them.

```
hammer install <dir>       install a world package into the local store
hammer list                list installed worlds
hammer verify <world>      recompute digests against install time
hammer tools <world>       list the tools a world declares
hammer serve <world>       speak MCP over stdin and stdout
```

`<world>` is `name:vintage` **or a path to a package directory**. That second
form is not a convenience, it is the workaround for the defect below.

The store is `$HAMMER_HOME`, default `~/.hammer`, laid out as
`worlds/<name>/<vintage>/`.

## Install the plugin and mount the MCP server

This plugin declares its servers in its own `.mcp.json`, so installing the
plugin through Cowork, Claude Code, or Codex wires them up. Do not add a second
copy of the same server by hand.

**One server per world, and the server's name is the world's name.** A world is
mounted by adding an entry, keyed by the world's name: a URL for a hosted world,
or the `bin/hammer-world-serve` launcher with the world's name as its one
argument for a local one. MCP namespaces tools by
server, which is why this shape was chosen: two worlds arrive as
`mcp__noise__claim_check` and `mcp__fda__claim_check` with no collision, so **no
world-scoped tool naming scheme exists anywhere.** It also keeps each world's
audit log its own file, and stops one process from being the single point of
failure for every world mounted.

**The scope is the whole point.** Use a user-wide install when the client offers
one. A project-scoped install makes the server available only in that project.

This is not a preference and not tidiness. **The first live test failed exactly
this way.** A noise world added while inside the hammer repo was invisible from
`/tmp/nn`. The agent working in `/tmp/nn` did not report a missing tool. It did
the arithmetic by hand and presented the result, and a hand-rolled spread has
none of the machinery that makes this world worth having: no reporting floor,
no refusal when the design is not fully crossed, no `not_measured` field, no
`cannot_answer` in the handshake. A silently absent tool is worse than a broken
one, because the answer still arrives and still looks like an answer.

**Cowork:** Open **Cowork > Customize > Plugins**. In **Personal plugins**,
choose **+**, then **Add marketplace**. Add `hmmrlabs/hammer-plugin`, install
**Hammer Worlds**, and complete connector sign-in when prompted.

**Claude Code:**

```
/plugin marketplace add hmmrlabs/hammer-plugin
/plugin install hammer-worlds@hammer   choose user scope when prompted
```

Check it with `claude plugin details hammer-worlds`. Then run `/mcp`, complete
browser OAuth, and confirm the world is connected.

**Codex:**

```
codex plugin marketplace add hmmrlabs/hammer-plugin
codex plugin add hammer-worlds@hammer
```

Approve browser OAuth during installation. If it was deferred, open the plugin
or MCP connection in Codex and authenticate there.

**Other Agent Skills clients:**

```
npx skills add hmmrlabs/hammer-plugin --skill hammer-worlds -a <profile> --yes
```

This route installs the skill. If the client does not consume the MCP dependency
in `agents/openai.yaml`, add `https://worlds.hammer.ai/mcp` as a remote HTTP MCP
server through that client's settings and complete browser OAuth.

In Claude Code and Codex, verify the server appears from a directory that is not
a Hammer checkout. In Cowork, verify it on the installed plugin's Connectors
page. Then call its no-argument `hello_world` tool. A connected tool and a
successful `hello_world` response prove more than a manifest on disk.

## Known limitation: a build bump cannot be installed over its own vintage

The store keys a world on **name and vintage only**. `WorldId` is
`name:vintage`, the install path is `worlds/<name>/<vintage>/`, and installing
over an existing vintage is refused: *"is already installed; vintages are
immutable"*.

The build number does not appear in that key. So `noise:2026-08-14` build 2
**cannot replace** build 1 in the store. `hammer install` refuses, and `hammer
list` keeps showing a world id that looks current while serving stale bytes.

The immutability rule is right and is not the bug. A decision made against a
vintage has to be reproducible against that vintage later, so silently
replacing bytes under a name would make the pin a promise rather than a fact.
The bug is that the key is too coarse to distinguish a build from a vintage.

**Workaround: serve from a path.**

```
hammer serve /path/to/worlds/noise
```

`hammer serve` accepts a package directory as well as an id, and a path has no
immutability rule to trip over, so it always gets the current build. This
plugin's `bin/hammer-world-serve` prefers a path over the store for exactly
this reason. Verified on 2026-08-15: serving the plugin's copy of the world
returned `serverInfo {"name": "hammer/noise", "version": "2026-08-14+2"}` and
both tools.

Until the store keys on build as well as vintage, treat `hammer install` as a
way to pin a vintage you are done with, and a path as the way to run the one
you are working on.

## Reading `hammer verify`

```
noise:2026-08-14: 2 files match their install digests
  checked:     install integrity, the bytes are those that were installed
  NOT checked: that a rebuild from data/ would answer identically
```

**Read both lines.** The second is not boilerplate and deleting it would be a
downgrade. There are three distinct guarantees and this command holds only the
first:

1. **Install integrity.** These bytes are the bytes that were installed. This
   is what runs today, and it is a SHA-256 per file recomputed against install
   time.
2. **Content equivalence.** A rebuild answers every query identically. This is
   the one the product claim needs, because it is what makes a decision made in
   March checkable in November. Not built.
3. **Byte-identical rebuild.** A rebuild produces the same bytes. The strongest
   and the least necessary. Measured separately in `spike-replay-determinism`.

The wording exists because "verified" alone is the failure mode: a reader takes
the strongest available meaning, so an install-integrity check that reads like
a reproducibility proof retires the question it did not answer. When you relay
a verify result, relay what it did not check too. `hammer pull` and
`hammer build`, and verify's placebo suite, do not exist yet.

## Reading a refusal

**A refusal is a value, not an error.** It crosses MCP as a normal result
carrying `kind: "refusal"`, never as a JSON-RPC error, because a client that
sees an error retries an honest refusal.

```json
{"kind":"refusal","detail":"UNKNOWN (rule accuracy-ceiling/no-repeats): ... What would settle it: a second independent reading of the same cases by the same judges, blind to the first, carrying a different occasion number ..."}
```

Three kinds, and the distinction is load bearing:

- **`Unknown`** is epistemic. It names the open question that would settle it.
  Read the "What would settle it" clause, because it is the next experiment.
- **`NotPermitted`** is not epistemic at all. It cites the standard that forbids
  the question, so an integrity finding is never reported as ignorance.
- **`BelowReportingFloor`** is a house rule about presentation. The estimate
  could be computed and would read as a finding when it is not one.

**Do not retry a refusal, and do not work around one by hand.** The refusal is
the product. Change the design it named, or report the refusal.

Refusals are published before you ask, not only after. `cannot_answer` from
`world.toml` comes back in the MCP `initialize` response as the server's
instructions, and each tool carries its own `refusals` array, so an agent can
see what a world declines before spending a call.

Undeclared MCP capabilities answer with **empty lists, not `-32601`**. Verified
on 2026-08-15: `resources/list` returns `{"resources": []}`. Clients probe
methods a server never advertised and, on an error, conclude the whole server
is unavailable while its tools were reachable the entire time.

## Which check to call, and there is no fallback

Every world serves two checks and they decide by different means.

**`claim_check` decides by entailment** over the numeric figures the payload
declares. It reads no sentence at all. A claim is an expression naming figures
by their dotted path in the payload:

```json
{"op":"<=","left":"report.level_noise_sd","right":31,"scope":"marginal"}
```

`op` is one of `<`, `<=`, `>`, `>=`, `==`, `!=`, `in`. Arithmetic is
`{"op":"/","left":...,"right":...}` with `+`, `-`, `*` and `/`. Only `entailed`
licenses writing the claim down.

**`judgement_check` decides partly from the English of the sentence**: a phrase
list, a content-word overlap, a digit scan. It has been walked past four times,
and a claim written in German cannot pass it at all.

**Prefer `claim_check`.** Its verdict comes from arithmetic over declared
figures, so it is the same verdict in every language and no wording buys a pass.
Reach for `judgement_check` only when the claim cannot be written as an
expression over the payload's numeric figures: a claim about a text field, about
a named party, about something existing, or about a shape the grammar has no
operator for.

**Choose once, before the call, and do not fall back.** A claim `claim_check`
will not read comes back `undefined`, `out_of_scope`, `vacuous`,
`indeterminate` or `unreadable`, and each of those is a refusal carrying a
remedy. **Do not resubmit it to `judgement_check`.** A checker that falls back to
a weaker checker is a fall-through approval with extra steps, and this repository
has settled that absence of a failed check is never evidence. Carry the refusal
into the claim report instead.

Four things about `claim_check` that move verdicts, all measured against
`noise:2026-08-14+5` on 2026-08-16 with the fixture B grid:

- **A payload-derived world is marginal only.** A payload declares no scopes, so
  `scope` must be `"marginal"`. `"per_case"` or a segment comes back
  `out_of_scope` naming `supported=["marginal"]`. `scope` is required on every
  claim, because a defaulted scope is the overclaim the check exists to refuse.
- **A float is read as quoted and an integer as exact**, decided by the JSON
  encoding and nothing else. So `==` against a float does not pass: a quoted
  figure is widened by one ulp either side and the equality is not settled
  inside that region. Measured: `report.grand_mean == 2040` came back
  `not_entailed`, while `>= 2039.5` and `<= 2040.5` were both `entailed`, and
  `report.n_judges == 5` was `entailed` because a count is an integer. **To write
  a rounded figure, claim the interval you are writing rather than the point**,
  as two claims at half a unit in the last written place.
- **A misspelled or absent figure is `undefined`, not a near miss.** Measured:
  `report.judge_alice_effect` came back `undefined` with the remedy *"this world
  declares no figure of that name, so either the claim misspelled it or this is
  the wrong vintage"*. That is the honest answer to a per-judge claim, because
  there is no such field.
- **Entailment is arithmetic, and it does not lift a `cannot_answer`.** The noise
  payload echoes the input grid back under `report.by_case`, so those echoed
  values are declared figures. Measured: `report.by_case[0].judged[0].value >
  report.by_case[0].case_mean` came back **`entailed`**, which would appear to
  license "alice priced this case above its mean". It does not. That figure is
  the caller's own input handed back, and `claim_check` has no `restates_input`
  verdict because a payload cannot tell it an observation from an estimate.
  `judgement_check` does have one. **An `entailed` verdict on a `by_case` path is
  a restatement of what you typed in, and the world's published `cannot_answer`
  still forbids the conclusion.** Treat entailment as necessary and never as
  sufficient.

A payload-derived world also carries no validity, no relations and no declared
incomparability. The reply names all four gaps under `world.cannot_express`, and
every field it could not declare, which is the text and the nulls, under
`world.not_declared`, so a figure that vanished between the answer and the world
is named rather than silently missing.

**A boolean field is declared, not skipped.** It arrives under
`world.declared.flags` at the value the payload states, and because a payload
that states a flag has decided it, `==` against that value is `entailed` and
against the other is `refuted`. Claim one with a category on the right,
`{"cat": "true"}`, and expect no ordering: `<` and `>=` against a flag are
`unreadable` rather than answered. (**Correction, 2026-08-17.** This paragraph
used to say the reply named *"every non-numeric field under
`world.not_declared`"*. That stopped being true at commit `5c37c45`, and a reader
who believed it would leave the most important qualifier on an answer unchecked
because they had been told it was undeclarable.)

## The claim report, which is part of the answer

**Any answer built on a hammer world ends with a claim report.** It is not a
courtesy and not a debugging aid. An answer that arrives without one is
incomplete, and whoever receives it should read it as incomplete.

It exists because of the one thing no check can do: **a check cannot see a claim
the caller does not submit.** Measured on 2026-08-15: given a correct refusal, a
model read it, relayed it accurately, then computed the refused quantity by hand
and delivered it anyway. Nothing in the protocol can stop that. What the report
changes is that skipping the submission now shows up in the deliverable rather
than nowhere.

**The list comes first and the prose second.** Before writing a sentence,
enumerate every claim the answer intends to make, one per line, in the claim
grammar rather than in English. Then check them. Then write only what came back
`entailed`. The report at the bottom is that same list with the verdicts filled
in, which is why it costs almost nothing to produce. Assembling it afterwards
from finished prose is how a claim goes missing.

**When you know who is going to read it, call `say_world` with that reader.**
Every world serves it beside `hello_world`, and the audience is a closed set:
`executive`, `statistician`, `clinical`. It names the misreading that reader is
most likely to make and points at the refusal of that world's own that forbids
it, which is the part you cannot get from the payload. It changes vocabulary,
emphasis and ordering only: every audience carries every one of that world's
refusals in the world's own words, reports the same basis, and closes with the
same claim report. If a rendering has lost a qualifier, it is not shorter, it is
false, and `say_world` is built so it cannot hand you one.

The shape, and it is fixed:

```
## Claim report

world     noise:2026-08-14+5
checked   claim_check
session   s18cca98cae2d24d0-2425-0, in ~/.hammer/audit/noise-<time>-<pid>.jsonl
receipts  r1 noise_audit, r2 noise_accuracy_ceiling (refused)

entailed
  r1  {"op":"<=","left":"report.level_noise_sd","right":31,"scope":"marginal"}
  r1  {"op":">","left":"report.pattern_noise_sd","right":"report.level_noise_sd","scope":"marginal"}
  r1  {"op":">=","left":"report.grand_mean","right":2039.5,"scope":"marginal"}
  r1  {"op":"<=","left":"report.grand_mean","right":2040.5,"scope":"marginal"}

refused
  r1  not_entailed  {"op":"==","left":"report.grand_mean","right":2040,"scope":"marginal"}
      grand_mean is a float, read as quoted, so the point is not settled. Reclaimed as
      the pair above and the sentence says 2040.
  r1  undefined  {"op":"<=","left":"report.judge_alice_effect","right":100,"scope":"marginal"}
      no figure of that name. There is no per-judge field, so the sentence was dropped.
  r2  answer_was_refused  whole receipt, no claim reaches a figure
      r2 is a refusal and declares nothing. It forbids a reliability coefficient, a
      bound computed from one, or any statement about an individual judge, by any
      method including arithmetic done by hand. Reported the refusal.

asserted without a check
  none
```

Five rules about it.

**Name the session and the receipts.** Every row the server writes carries the
session id, so a reader holding the log can find exactly which answers you were
handed, exactly which claims you submitted, and any receipt you took and never
redeemed. The session id goes to the server's stderr at startup and into every
log row, so read it from the log file rather than guessing. If no log was
written, write `session not recorded` and say why. Do not drop the line.

**A refusal is a value, so it goes in the report carrying its remedy.** Every
refused verdict has one: `undefined` carries the namespace's repair,
`out_of_scope` names the scopes that exist, `not_entailed` may carry a note, and
a refused receipt carries the world's own forbidden-conclusion sentence. Write
the remedy down. It is often the most useful line in the report, because it names
the rephrasing or the experiment that would settle what you could not say.

**Check against a refusal too.** Measured on 2026-08-15: given an answer, a model
checked its claims and rewrote the rejected ones; given a **refusal**, the same
model never checked at all, then computed the refused quantity by hand and
delivered it. A refusal is when checking matters most, because it is exactly the
case where nothing supports what you were about to write. A refusal receipt you
submitted against is in the log as redeemed; one you took and walked away from is
in the log as unredeemed.

**`asserted without a check` is the point of the section and is never optional.**
Anything your answer asserts that no check licensed goes here, in words, with
what it rests on. Arithmetic you performed yourself goes here. A figure read off
the input grid rather than out of a payload goes here. A restatement of your own
input goes here even when `claim_check` said `entailed`. `none` is itself a claim
about your own answer, and it is only true if you re-read the prose against the
list. **A report listing only the claims that passed is the opt-out wearing a
uniform**, and that is worse than no report, because it looks like compliance.

**Do not hedge a refused claim into the prose.** `DO NOT WRITE` means do not
write it, not "write it with a caveat". The measured failure this exists to stop
is a correct caveat sitting in the same paragraph as the claim it forbids.

## When the server will not connect: the hosted transport

Open the client's MCP connection view and read the state before changing
anything, because three different situations all look like the tools are
missing. In Claude Code, that view is `/mcp`. In Cowork, open the installed
plugin's Connectors page.

**Not connected at all.** The plugin is not enabled, or it is enabled at project
scope and you are somewhere else. Check the scope first; see above for why that
is the failure worth ruling out before any other.

**Connected but not authenticated.** Choose the server in the client's MCP view
and authenticate. Until that is done the service answers `401`, which is the
correct answer to an anonymous caller and not a fault.

**Connected, authenticated, and the world still declines.** Then it is a refusal
and the refusal is the product. Read it, report it, and do not compute around it.

Two things this transport does not have. There is no per-world path to guess at:
measured 2026-08-24, `https://worlds.hammer.ai/mcp` answers and paths under it
named for other worlds answer `404`, so a world that is not on the surface is not
reachable by inventing a URL. And there is no local audit file, so take the
session id and the receipt ids out of the tool results you were handed. If a
result carried neither, write `session not recorded` in the claim report and say
so. Do not drop the line and do not invent one.

## When the server will not start: a world you run yourself

Everything below is about the local transport. The launcher it describes travels
with a hammer checkout and with a plugin that ships a world package; it is not in
a plugin that only points at a hosted URL, and none of this applies to one.

Run the resolver's own explain mode, which prints what it would use and exits
without serving. It takes the world's name, the same name the server has in
`.mcp.json`:

```
<plugin root>/bin/hammer-world-serve noise --explain
```

Nothing in that script is world specific. Named with no world at all it refuses
rather than picking one, because a launcher that guesses can serve the wrong
world and the handshake would succeed anyway.

It resolves three things and reports all of them.

**The binary**, in order: `$HAMMER_BIN`, `hammer` on `PATH`,
`$HAMMER_REPO/result/bin/hammer` then `target/release` then `target/debug`,
then the plugin's cache directory. Nothing in that chain builds anything. That
is deliberate: a Rust build takes minutes, and minutes spent inside an MCP
`initialize` handshake is a hang from the client's side, not a slow start.
`/hammer:noise-setup` does the build, once, and puts the result where the
resolver looks.

**The world**, for a world called `<name>`, in order: `$HAMMER_WORLD_<NAME>`,
`$HAMMER_WORLD`, `$HAMMER_REPO/worlds/<name>`, the copy shipped inside the
plugin, then the newest vintage of `<name>` in the store. Paths beat the store
for the build-bump reason above. Set `HAMMER_REPO` if you are developing hammer
and want your working tree served instead of the shipped copy. A world named in
`.mcp.json` and absent everywhere is refused by name, which is the ordinary state
of a world that has no estimator yet.

**The audit log, one per world.** `$HAMMER_AUDIT_LOG_<NAME>` if it is set, then
`$HAMMER_AUDIT_LOG`, otherwise `$HAMMER_HOME/audit/<name>-<UTC time>-<pid>.jsonl`,
defaulting to `~/.hammer/audit`. One row per answer the world hands out and one
row per claim submitted against it, each stamped with a session id so two runs can
be told apart. A bare `hammer serve` writes nothing unless given `--audit-log`
itself; only the plugin's launcher switches it on. If the file cannot be opened,
hammer says so on stderr and serves unrecorded, because losing the recording is
better than losing the world.

The scoped forms exist because every server a client launches reads one
environment. An unscoped `HAMMER_AUDIT_LOG` would pour every mounted world into
one file and an unscoped `HAMMER_WORLD` would hand every mounted world the same
package. Both are right with one world mounted and wrong with two. The default
path carries the world's name for the same reason: the count the ledger exists for
is receipts issued and never redeemed, which is a number about a world, and
merging two worlds turns it into a number about the client.

**The stdout rule.** Once the server is running, stdout is a JSON-RPC stream.
One stray byte written to it corrupts the stream, and the client reports the
server as broken rather than as misconfigured. Every diagnostic in the resolver
goes to stderr for this reason. If you wrap or replace that script, keep the
rule.

**A world shipped inside the plugin is a copy** of `worlds/<name>` in the
repo, taken at plugin release time. The repo copy is the source of truth. If a
served `world_version` disagrees with the checkout you are reading, that is
drift, and `/hammer:noise-setup` diffs the two and says so.

## Related

- A world's own routing skill, served by the world itself at
  `hammer://<package>/skills/<skill>`. Fetch it from the server rather than
  looking for a copy: it comes from the build that is answering you.
- The tool descriptions in the MCP surface, which are the authority on what
  exists, what each call takes and what each one refuses.
