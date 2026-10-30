# How do you know your agent is wrong?

I recently ran two agent-research projects side by side. In both, a coding agent iteratively improved a program against measurable feedback, accumulating what it learned into files. In one, the agent analyzed STM images of kagome materials. In the other, it optimized tensor-contraction schedules for quantum chemistry.

Both agents were competent. Both worked hard. What differed — and what this post is about — showed up when I read their *thinking* side by side.

## One agent thinks in observations

Here is the STM agent, mid-diagnosis, working on why its lattice-fitting failed on a noisy fixture:

> noise/2000: residual 14.3° even after outlier rejection — […] but why is grid residual 14°? Because true ring pairs themselves were evicted as "outliers" before the noise spikes (if noise spikes dominate the fit offset).

Look at what the failure *arrived with*: a fixture name (`noise/2000`), a quantity, a magnitude (`residual 14.3°`). The agent's reasoning starts from a localized fact, and one step later it has the mechanism. This was the pattern all through that project: the evaluator compared every output against known ground truth, per fixture, per quantity, so every failure came with its own coordinates.

That project ended with eighteen named, mechanism-verified failure modes, each frozen into a permanent regression test, and a sealed exam the analyzer passed honestly — including the failures, documented, with named fixes.

## The other thinks in hypotheses

Here is the compiler agent, also mid-diagnosis, working on why its scheduler lost to the baseline on one benchmark:

> ADC2 selected appears almost naive, why no biclique? Perhaps terms split into defs >20 and skip, or initial has only 8 terms output etc. Should inspect problem ADC2 definitions terms. Use Python summarize.

And later, still on the same question:

> Old candidate perhaps generated at sizes100,1000 not16,64! […] The `select_heldout` perhaps commands not stored.

The difference is visible before you read the content. The STM agent's thinking quotes numbers the environment emitted. The compiler agent's thinking is a scatter of "perhaps" — because its environment emitted one scalar per run, and its records couldn't answer even basic questions about themselves. Which command produced this schedule? Nobody stored it. The first act of every diagnosis in this project was *building* an inspection tool, then guessing which of three stories was true.

One agent's failures arrive with the "why" attached. The other's "why" must be manufactured by the reasoner, at full price, every time.

## Numbers don't carry their own validity

Once I saw this, I found its consequences everywhere in the compiler project's records.

A schedule had been timed and showed a win. The timing run had no idea the schedule was numerically wrong — correctness lived in a *different command*, and the intersection was checked by nobody. The win looked exactly like a real win.

The same baseline was measured in two honest, protocol-compliant sessions: 58.7 ms and 67.9 ms. Fifteen percent apart, nothing flagged it, and two downstream conclusions silently depended on which number was right. There was no way to adjudicate — there is no ground truth for wall time, only measurements of it.

And a headline comparison rested on measurements with too-few iterations and a 3.4× spread between repeats of the *same candidate* — visible right there in the file, for weeks. Nobody was assigned to look.

In the STM project, a wrong answer came with its location and size. In the compiler project, wrongness lived at *intersections* — timing ∩ correctness, session ∩ session, claim ∩ fine print — and intersections only get checked when someone with judgment decides to look.

## What would make wrongness visible?

Debugging is my mental model. Programming environments make wrongness visible in four ways; agent research environments usually ship with none:

**Detectable.** A failing build produces a signal. A silently-wrong number does not. Most agent loops emit rewards, not alarms.

**Diagnosable.** A stack trace says *where*. A bare "test failed" does not. This is the difference between a per-fixture residual and a single wall-time scalar — between getting the "why" with the failure and manufacturing it yourself, one "perhaps" at a time.

**Expensive to ignore.** A compiler warning gets ignored; a compiler error blocks the build. This is the property people underestimate. Disclosure without enforcement is a warning, and warnings get ignored. In the STM project, the scoring priced a confident wrong answer at ten times a refusal. As that agent put it: *a refusal is a delay; a lie is fake science.*

**Remembered.** A bug fixed without a postmortem recurs. An agent session that ends forgets. The STM project froze every failure into a permanent regression test; the compiler project re-derived the same lessons across context resets.

The STM project got all four from its environment, for free, because ground truth was available. The compiler project has no ground truth. So the question becomes: can these four properties be built by hand, into an environment that doesn't provide them?

I don't know. We're trying — the current attempt moves the workflow into versioned files, installs a run-once exam and a critic whose challenges block progress, and prices confident wrong answers above honest refusals. It's too early to say whether it works. What I can say is that the attempt has already changed what I watch: less the agent's outputs, more whether its failures arrive with their coordinates attached.

My last post ended by asking how to get an agent to question its chosen approach. I suspect this is the same question in disguise. An agent questions its conclusions exactly when wrongness is detectable, diagnosable, expensive, and remembered — and not before. Making that true, in domains without ground truth, is the problem we're working on now.

*Written with AI assistance from the original conversations and saved session records. Quotes are verbatim from session logs; `[…]` marks omissions. The compiler project is unfinished. The raw sessions, credentials, and private repositories are not published.*
