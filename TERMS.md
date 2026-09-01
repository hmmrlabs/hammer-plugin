# Terms of service

**Hammer Worlds**, operated by hmmr labs. Last updated 1 September 2026.

These terms cover the hosted Hammer world reached at `https://worlds.hammer.ai/mcp`
and the plugin that mounts it. The software in this repository is separately
licensed under Apache-2.0; a licence governs the code, and these terms govern the
service.

## What the service is

A corpus that answers questions about published healthcare pricing and coverage
data, over MCP, and that refuses the questions its sources cannot settle.

## What an answer is, and what it is not

This is the part worth reading, because it is the part that decides whether the
service is useful to you.

**Every figure carries its basis.** `measured` means this world read it out of a
register it holds. `cited` means it is published, with a source. `assumption`
means you declared it. A figure computed from several is never stronger than its
weakest input, and the answer says which input that was.

**A refusal is an answer.** Where a question is outside what the sources settle,
the world declines and names what would settle it. That is the service working,
not failing.

**Nothing here is advice.** Not medical advice, not legal advice, not
reimbursement advice, not investment advice. The world serves what payers
published and what registers record. Deciding what to do about it is yours, and
so is the responsibility for it.

**Published rates are not coverage, and not a guarantee of payment.** A rate in a
payer's machine-readable file says what that file said on the day it was read. It
does not say the payer will pay it, that a service is covered, that a code may be
billed in your circumstances, or that a contract exists. Those are questions the
world declines by name.

**The data has a vintage and the vintage is on every answer.** Sources move.
Payers republish monthly and some retain only the current month. An answer is
about the bytes that were read, on the date stated, and is not a claim about
today.

**Counts are what they are stated to be.** Where an answer reports an `n`, that
`n` counts what the answer says it counts, which for rate data is contract rows
in an extract rather than transactions, providers, or patients.

## Your responsibilities

Use the service for lawful purposes. Do not attempt to overwhelm it, work around
its authorisation, or present its answers as something other than what they say
they are.

**If you quote a figure, quote what came with it.** The intervals, the counts and
the basis are part of the answer rather than decoration on it. Presenting a
median without the evidence attached to it is the failure this whole service is
built to prevent, and doing so is on you rather than on us.

## Availability

The service is provided as it is. It may be unavailable, and it may change: tools
are added, corpora are re-read, and a world's build number moves when either
happens. A receipt names the build that issued it, so an answer stays traceable
across those changes.

We do not promise a service level. If that matters to your use, write to us
before you depend on it.

## Liability

To the fullest extent the law allows, hmmr labs is not liable for indirect or
consequential loss arising from use of the service, and total liability for any
claim is limited to what you paid for the service in the twelve months before it
arose.

Nothing here limits liability that cannot lawfully be limited.

## Suspension

We may suspend access that is abusing the service or that puts it at risk. Where
we do, we will say why.

## Changes

These terms are versioned in the public repository that distributes the plugin,
so a change is a commit with a history rather than a silent edit. Continued use
after a change accepts it. Material changes will be noted in the release notes.

## Contact

**hammer@multiversal.ventures** reaches us: for the service, for data
questions, and to report a vulnerability.

One address rather than three. It is a group rather than a person, so it
survives any one of us being away, and it accepts mail from anyone rather than
only from people we already know. Three aliases forwarding to one inbox is a
directory of aliases rather than a support channel.
