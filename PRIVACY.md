# Privacy

**Hammer Worlds**, operated by hmmr labs. Last updated 1 September 2026.

This describes what actually happens when you use a hosted Hammer world. It is
written from the behaviour of the software rather than from a template, and where
something is retained or identified it says so plainly rather than in the
negative.

## What reaches us

**Only your tool calls, and only what you put in their arguments.**

A Hammer world is a remote MCP server. Your client sends a tool name and a set of
arguments; the server answers. There is no code in it that reads your filesystem,
your repository, your other tools, or the rest of your conversation, and it never
receives them. A declaration of a URL has nothing that could read a file.

So what we see is the question you asked: a billing code, a payer name, a
population figure, the distributions you declared. If you put something in an
argument, it reaches us. If you do not, it does not.

## What is written down, and where

Two separate records exist and they hold different things. The distinction is
load-bearing, so it is set out rather than summarised.

### The session ledger, which does contain your questions and our answers

Every answered call issues a receipt, and the receipt is written to an object in
cloud storage. That object holds, for each receipt:

- the tool that produced it
- **the arguments you supplied**
- **the answer payload, exactly as it crossed the wire**
- whether the world refused
- any claims you later submitted for checking, and the verdicts they came back
  with
- counts of what was issued, checked and left unchecked at each `deliver` call

This is the audit trail. It exists so that a figure you quote three months from
now can be traced to the call that produced it, which is the entire premise of
the product. It cannot do that without holding the question and the answer.

**Retention: these objects are kept indefinitely.** There is no automatic
deletion and no lifecycle rule that removes them. If you want a session's ledger
object deleted, write to the address below and it will be deleted.

### The operational stream, which contains no content at all

Separately, the service emits structured log lines so it can be operated and
counted. That stream carries **no tool arguments, no answer payloads, no claim
text and no refusal message body**. What it carries is:

- which tool was called, and whether it answered or refused
- when a refusal fired, its **slug** only, from a closed vocabulary. A slug says
  a code was unknown; it never says which code
- how long a call took
- counts of receipts issued, checked and unchecked
- the size of the tool listing on connect
- a **subject** identifier

The subject is `oauth:<subject-id>` or a token fingerprint. It exists so that a
busy month is distinguishable from one person reconnecting forty times. **It is
an identifier and not content**: it says two sessions were the same caller
without saying what either of them asked. A raw bearer token never appears in
this stream.

## Identity and sign-in

Access is by OAuth. We receive the subject identifier your authorization server
issues, and we use it to key your session and to authorise your calls. We do not
receive your password. We do not ask for, and have no use for, your name, your
employer or your email beyond what your authorization server chooses to assert.

## What the world holds, which is not about you

The corpus is public data: published payer rate files, Medicare fee schedules,
NCCI edit files, Medicare Coverage Database articles, hospital price-transparency
filings, and published clinical literature. None of it is patient data, none of
it is your organisation's data, and nothing you send is added to it. Asking about
a billing code does not put that code, or you, into the corpus.

## What we do not do

We do not sell anything to anyone. We do not share your ledger with other users
of the service. Your questions are not used to train a model.

## Cookies and tracking

The MCP endpoint sets no cookies and runs no analytics in your client. It is an
API that answers tool calls.

## Your requests

Write to **privacy@hammer.ai** to ask what is held for you, to have a session's
ledger object deleted, or to raise a concern. Say which sessions if you know
them; the session id appears in every answer the world returns and in `deliver`.

## Changes

This page is versioned in the repository that distributes the plugin, so its
history is public and a change to it is a commit rather than a silent edit. The
date at the top moves when the text does.
