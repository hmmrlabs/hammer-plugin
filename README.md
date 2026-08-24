# hammer worlds, as a Claude Code plugin marketplace

A **hammer world** is a corpus that publishes what it cannot answer. It arrives
as an MCP server, it hands out figures with receipts attached, it checks the
claims you write against those receipts, and where a question is outside what its
sources can settle it refuses and says why. The refusals are not a limitation
that came with it. They are the product.

This repository is the marketplace. Adding it gives you one plugin, which mounts
the hosted **coverage-intelligence** world and carries the one skill that is
about the discipline rather than about any particular corpus.

## Add it

```
/plugin marketplace add hmmrlabs/hammer-plugin
/plugin install hammer-worlds@hammer
```

Choose **user scope** when prompted, so the world is reachable from every
directory rather than from one repository. The skill explains at length why that
is not a matter of taste; the short version is that a silently absent tool is
worse than a broken one, because the answer still arrives and still looks like an
answer.

Then run `/mcp` and authenticate. A browser opens. There is no key to paste, no
token in this repository and nothing to put in your settings.

## What it costs

Nothing to run. The world is hosted at `https://worlds.hammer.ai/mcp` and it is
the operator's own service, so there is no build, no download and no corpus on
your disk. What a given caller is entitled to ask it is a separate question from
being able to reach it: signing in says who you are, and
the world decides per compartment what it will serve you. Where you are not
entitled to something it says so and names the way to ask, through its
`request_access` and `redeem_code` tools. A refusal for want of entitlement is a
refusal with a remedy, not an outage.

## What it refuses

This is the part worth reading before you install anything. Three of the
questions people most often bring to a rates corpus, and why this world will not
answer them:

- **It cannot tell you a denial rate, or a probability that a claim gets paid.**
  Not by any method, including arithmetic over counts it will happily serve you.
  Machine-readable files hold negotiated rates, not adjudications. A rate being
  present does not record a payment and a rate being absent does not record a
  denial, so there is no denominator anywhere in this corpus for that fraction.
- **It cannot tell you whether a payer covers a code.** A published negotiated
  rate says what a payer pays when it pays, not whether it will. "Payer X covers
  code Y" is not a sentence these sources can settle in either direction, and the
  absence of a rate row is not evidence of non-coverage: payers omit, fragment
  and misfile their disclosures market-wide. The supportable sentence is that no
  rate was recorded in this month's disclosure.
- **It cannot turn relative value units into dollars.** A code priced nationally
  on the Medicare Physician Fee Schedule carries relative value units, and
  converting those to money needs a conversion factor and a locality adjustment
  this world does not hold. So it returns the units and no ratio, rather than a
  dollar figure that would be a guess wearing a decimal point.

It also declines, among others, to average across payers without weights you
declare, to predict or rank an individual provider, to make any clinical claim
about any device or test, to state what was in force after its own vintage, and
to add a cost that arrives later to a cost that never arrives.

**That is a summary, and the world publishes the real list itself.** Its full
`cannot_answer` list arrives in the MCP `initialize` handshake as the server's
instructions, and every tool carries its own `refusals` array, so a client can
see what the world declines before spending a call. Where this README and the
handshake disagree, the handshake is right: it comes from the build that is
answering you and this file does not.

## Why this plugin is so small

One skill and one URL. That is deliberate, and it is the plugin's best argument.

**A world ships its own instructions and serves them from its own deployed
build.** Each world carries a `skills/` compartment inside its package, and the
server exposes it over MCP: `resources/list` returns one resource per skill at
`hammer://<package>/skills/<skill>`, and `resources/read` returns that `SKILL.md`
out of the running image. The coverage world's routing skill,
`hammer-coverage-navigate`, comes to you that way.

So the routing instructions for a world and the tools that world answers with are
the same revision by construction. They cannot describe a version that no longer
exists, because there is only one of each and they are deployed together. If you
have ever installed a plugin whose documentation described a release from six
months ago, that is the failure this arrangement removes rather than manages.

What is left over for this repository is the part that belongs to no world: how
to route a question through whatever worlds a client has mounted, why two worlds'
answers never compose into a third claim, how to read a refusal, and the claim
report that any answer built on a world has to close with. That is the
`hammer-worlds` skill, and it is the only skill here.

## What this plugin sends where

The dialog that adds a marketplace warns that Anthropic cannot verify third-party
plugins. That warning is correct and this section is what it is owed. Everything
in it can be checked against the files in this repository before you install
anything, which is the only kind of assurance worth offering.

- **Read it first; it is small enough to read.** One skill, which is prose, and
  three JSON files. Nothing here executes.
- **No hooks, no commands, no agents, no binaries, no corpus.** Nothing in this
  plugin runs on your machine on a trigger you did not press, and nothing in it
  can read a file, because there is no code in it that could.
- **One network destination, declared in one file.**
  `hammer-worlds/.mcp.json` names `https://worlds.hammer.ai/mcp` and nothing
  else. What crosses the wire is the tool call your client made and the arguments
  you put in it.
- **The service is the operator's own**, running on Google Cloud. Its replies
  carry a `server: Google Frontend` header, and its authorisation is OAuth
  against an authorisation server named in its published protected-resource
  metadata, which you can fetch yourself before installing anything:

  ```
  curl https://worlds.hammer.ai/.well-known/oauth-protected-resource
  ```

  Measured 2026-08-24: an unauthenticated POST to the MCP endpoint answers `401`
  with a `www-authenticate: Bearer` header naming that document. That is the
  correct answer to an anonymous caller.
- **The operator keeps a ledger.** Every answer the world hands out is a receipt,
  and every claim submitted against one is logged with its verdict, on the
  operator's side. That exists so that answers taken and never checked are
  countable rather than assumed. If that is not a trade you want to make, do not
  install this.

## What is deliberately not here

**No corpus travels in this repository, and none ever will.** The data the
coverage world reads sits behind licence conditions that permit acquisition and
not redistribution: some of it is cleared for non-commercial use with a source
linkback, and some of it is cleared to hold and not to pass on. A public
repository shipping any of it would breach that. The plugin ships a pointer and
the discipline for reading what comes back, and the world serves the data under
its own terms or declines to.

**The `noise` world is not here either**, and it is a world an evaluator would
enjoy: it measures how much of a set of judgements is the case and how much is
whoever picked it up, and it refuses more than it answers. It is absent for a
plain reason rather than a policy one. Measured 2026-08-24, the hosted service
answers on one endpoint and paths under it named for other worlds return `404`,
so there is nothing for a second entry in `.mcp.json` to point at. Shipping the
noise skills without a noise server would hand an agent a procedure for reading a
payload it has no way to obtain, and the measured failure that procedure exists
to prevent is precisely an agent doing the arithmetic by hand and presenting the
result. When the world is hosted, it becomes one more entry in the same file.

## Licence

Apache License 2.0, in `LICENSE`, with `NOTICE` stating what it covers.

The skill here is prose rather than code, and a content licence such as CC BY 4.0
would fit prose more exactly. Apache-2.0 is used anyway, for one reason that
outweighs the fit: this text moves between this repository and the private one
that holds the hammer runtime, which is Apache-2.0, and the same text under two
licences is a worse problem than a licence that is slightly the wrong shape for
its subject. Attribution is required either way, and the patent grant and the
explicit contribution terms are useful for the JSON manifests that sit beside the
prose.

The licence covers what is in this repository. It says nothing about the corpora
the hosted world reads, which are not distributed here and travel under their own
terms.
