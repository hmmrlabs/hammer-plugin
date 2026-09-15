# Measuring when an agent should refuse

*15 September 2026. What we built, what it measures, and the numbers that came out,
including the ones that went the wrong way.*

Healthcare pricing questions have a property that makes them useful for testing agents: a
lot of them have no answer. The record either holds a figure or it does not, and an agent
that produces one anyway has done something worse than fail.

So we built a benchmark for the decision to refuse, ran nine configurations across two
models, and shipped what it told us. This is the technical write-up.

---

## The benchmark

**280 questions in controlled pairs.** Each pair is worded identically and exactly one
declared field differs, so one half is answerable and the other is not. The template is
fixed; only the rendered values move. That construction is what lets a difference in score
be attributed to the thing that changed.

**Four families**, each a different reason an answer might not exist: a rate for a payer
that filed none, a code outside the served vocabulary, a code pair with no edit on file, a
policy question the record does not settle.

**Four scores**, each over a denominator the case set fixes rather than one the run
produces:

- **abstention F1**, on the refuse class
- **answer correctness**, over answerable cases only
- **figure accuracy**, over cases where a specific number was asked for
- **leakage**, how often a refusal still hands over the withheld fact

**Nine configurations**: the full tool surface, a single-entry surface, amended
descriptions, both together, a BM25 retriever, web search, both surfaces at once, no tools
at all, and no world at all. **Two models**: `gemini-2.5-flash` and `gemini-2.5-pro`, both
at temperature 0.

## The first result was that our benchmark was broken

Before scoring any agent we ran a set of **rules that never read the question**. Each is a
decision procedure over the retrieved rows alone. If one scores well, the benchmark is
solvable without the question and every agent number on it is uninterpretable.

On the first build, abstention F1:

| rule, reads no question | score |
|---|---|
| `code_present` | **1.000** |
| `answering_row_present` | **1.000** |
| `asked_field_present` | **1.000** |
| `wording_or_code_present` | **1.000** |
| `score_threshold` | 0.977 |

Four rules at 1.000, accuracy 1.000. The benchmark was perfectly solvable by a procedure
that never looked at what was asked.

The cause was structural rather than careless. Most unanswerable cases were unanswerable
because **the thing asked about was absent entirely**, and absence is a retrieval question:
if the row is not there, a text search settles it at rank one, every time. We had built a
test of whether an agent can decide, out of cases that only required it to look.

**The fix was the cases, not the metric.** There are two ways to be unanswerable and they
are not the same difficulty:

- **The thing is absent.** Settled by looking. BM25 reaches rank one on 1.000 of these.
- **The thing is present and the field you asked for is not.** The row is right there and
  looks like an answer. Deciding it is not one requires knowing what that record claims.

Only the second resists. After rebuilding around it:

| rule | before | after |
|---|---|---|
| `code_present` | 1.000 | 0.556 |
| `answering_row_present` | 1.000 | 0.738 |
| `wording_or_code_present` | 1.000 | **0.889** |

The best question-blind rule still scores 0.889. Our benchmark is
contamination-**measured**, not contamination-free, and those are different words.

## What moved the agent, and what did not

### Routing was the bottleneck, not knowledge

The full 25-tool surface reached a tool that serves evidence on **78.6%** of answerable
questions. When it missed it did not fail loudly: it searched a text index, sometimes for
thirty seconds, and in one case repeated the same sentence to itself twenty-four times.

Replacing 25 tools with **one call taking the question's kind** fixed it:

**1,285 dispatches on flash, 346 on pro. Zero mis-routes, zero refusals, zero errors.**
Every family reached its declared tool every time.

### A stronger model made it worse

This is the result we expected least.

| 3 repetitions, same surface, same corpus | flash | **pro** |
|---|---|---|
| abstention F1 | **0.9830** ±0.0021 | 0.9410 ±0.0020 |
| answer correctness | **0.9429** ±0.0143 | 0.8667 ±0.0109 |
| leakage (lower better) | **0.0286** | 0.0667 |
| cases answered inconsistently | **11** | 31 |

Pro routes marginally better and is worse on everything else, including leaking the
withheld fact more than twice as often. Observed three times across two surfaces.

### The interventions bought reliability, not capability

Running three repetitions per question lets you separate "could it ever" from "would it,
every time":

| | pass@3 | **all@3** | gap |
|---|---|---|---|
| retriever | 1.0000 | 0.9857 | 0.014 |
| single-entry surface | 1.0000 | 0.8857 | 0.114 |
| amended descriptions | 0.9929 | 0.7786 | 0.214 |
| full 25-tool surface | 0.9571 | **0.5071** | **0.450** |

The full surface answers 95.7% of questions correctly **at least once** and 50.7%
correctly **every time**. Nearly half its answerable cases are coin flips, and the reported
correctness of 0.7571 sits between the two, describing neither.

**No published score shows this**, because all of them average over repetitions and an
average over a coin flip looks like a number. The ordering of that gap is exactly the
ordering of the surface work, which makes the result a different claim than a higher score:
**the interventions did not teach the agent anything. They made it say reliably what it
already knew.**

A finding that fell out of the same table: every run was at **temperature 0**, and the full
surface still has a mean within-case variance of 0.0897. Same question, same build, three
times, three different answers.

### Prose routing adds nothing once a single entry point exists

We amended three tool descriptions to name their sibling tools, which had previously moved
routing from 0.7929 to 0.9786. Then we tested whether it still helps once the single-entry
surface exists.

Paired on case, majority over three repetitions: **136 both right, 2 both wrong, 1 each
way. Two discordant cases, exact two-sided p = 1.0000.** Identical majority correctness at
0.9786.

Predicted before the run and recorded in the commit: no difference, because the single
entry point hides all three amended tools and the model never reads that text.

## Before and after, on the same benchmark

Same configuration, same 280 questions, same scoring code, three repetitions either side of
a deployment:

| | before | after |
|---|---|---|
| answer correctness | 75.7% | **95.2%** |
| correct every time | 59.3% | **87.9%** |
| reached the record | 78.6% | **100.0%** |
| abstention F1 | 0.913 | **0.989** |
| questions producing no answer | 27 | **1** |

Arrival is 1.0000 with a standard deviation of 0.0000.

## The instruments, and four times they refused us

More of the value came from checks that stopped us than from checks that passed.

**The judge was calibrated against 258 human labels**, and we found the corpus was
deliberately enriched with boundary cases while the gate read the mixture as though it were
a random draw. Precision on the random half was 0.9800; on the hard half, 0.2000. The gate
now certifies on the random draw and publishes the hard result beside it, gating on
nothing.

**A checkpoint lock stopped two processes writing one file.** We had already produced 177
duplicate rows that way, each of which every score would have counted twice.

**A world-identity check caught a confound we created.** Two configurations were compared
across a production deployment and measured different corpora. The apparent result, prose
routing ahead by +0.081 correctness and +0.200 figure accuracy, was entirely the corpus.
Re-running on one corpus produced the p = 1.0000 above.

**A leakage counter was penalising the wrong thing.** It charged an agent for naming the
question it was declining. Since a refusal that says what it is declining is more useful
than a vague one, **the metric was training the behaviour it existed to discourage.** Fixed,
and one configuration's leakage fell from 0.0429 to 0.0186.

## What we will not claim

**The gate is not passed.** Judge precision on the answered class cannot be placed above
0.90 by the labels we hold, and the invalid class has zero instances, so the certificate is
void on two counts.

**Between-run variance exceeds within-run spread.** Two runs of the same configuration
differed by 0.0571 on correctness and 0.1714 on figure accuracy. Differences smaller than
that are not results, and we have stopped quoting the within-run standard deviation as the
error bar.

**Absolute numbers describe one corpus at one date**, one benchmark, one temperature and
two models from one family.

**Every benchmark figure here was measured against a build labelled by the bench file
rather than verified against the server.** We shipped the check that catches this after
these runs; it caught a real mismatch on its first use.

---

## The thing worth taking away

We kept three instruments that should score near zero: a rule that refuses everything, a
set of rules that never read the question, and an agent with nothing to read. The last of
those answers 8 of 140 questions correctly from what the model already knew, and if that
number ever climbs against a fixed corpus, something has leaked.

All three are boring until one of them beats you. Ours did, on the first run, and the
benchmark got rebuilt.

*The plugin is [hmmrlabs/hammer-plugin](https://github.com/hmmrlabs/hammer-plugin),
Apache-2.0.*
