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

Once I saw this, I found its consequences everywhere in the compiler project's first milestone.

A schedule had been timed and showed a win. The timing run had no idea the schedule was numerically wrong — correctness lived in a *different command*, and the intersection was checked by nobody. The win looked exactly like a real win until the project's protected exam ran the two checks in one place and stopped itself:

> The protected exam stopped immediately at Axis 0: `mp3_selected` passes `truthmill::verify` but fails the numerical oracle at `(3,5)` with worst relative error 1.985. No exam timings were run, so there are no new performance claims.

The same baseline was measured in two honest, protocol-compliant sessions: 58.7 ms and 67.9 ms. Fifteen percent apart, nothing flagged it, and two downstream conclusions silently depended on which number was right. There was no way to adjudicate — there is no ground truth for wall time, only measurements of it.

And a headline comparison rested on measurements with too-few iterations and a 3.4× spread between repeats of the *same candidate* — visible right there in the file, for weeks. Nobody was assigned to look.

In the STM project, a wrong answer came with its location and size. In the compiler project, wrongness lived at *intersections* — timing ∩ correctness, session ∩ session, claim ∩ fine print — and intersections only get checked when someone with judgment decides to look.

## What would make wrongness visible?

Debugging is my mental model. Programming environments make wrongness visible in four ways; agent research environments usually ship with none:

**Detectable.** A failing build produces a signal. A silently-wrong number does not. Most agent loops emit rewards, not alarms.

**Diagnosable.** A stack trace says *where*. A bare "test failed" does not. This is the difference between a per-fixture residual and a single wall-time scalar — between getting the "why" with the failure and manufacturing it yourself, one "perhaps" at a time.

**Expensive to ignore.** A compiler warning gets ignored; a compiler error blocks the build. This is the property people underestimate. Disclosure without enforcement is a warning, and warnings get ignored. In the STM project, the scoring priced a confident wrong answer at ten times a refusal. As that agent put it: *a refusal is a delay; a lie is fake science.*

**Remembered.** A bug fixed without a postmortem recurs. An agent session that ends forgets. The STM project froze every failure into a permanent regression test; the compiler project re-derived the same lessons across context resets.

The STM project got all four from its environment, for free, because ground truth was available. The compiler project had no ground truth. So for its new regime, we built the four properties by hand.

## The constitution

The workflow lives in six small, versioned files in the repo: a protocol (how work happens), a spec (what counts as success, with the exact commands that measure it), a failure registry (mechanism verified *before* the fix, fixture pinned first), an idea ledger (every research idea carries its own kill condition, written before the experiment), an append-only trial log, and a compressed research summary. The system prompt shrank to six lines: read these files, work autonomously, stop only at milestones or blockers. If the prompt and the files disagree, the files win.

Two pieces came straight from the comparison above.

The **protected exam**: a benchmark suite that runs once per milestone, reported as-is, never iterated against — the thing that stopped itself at the correctness gate instead of reporting timings.

And the **critic**: at each milestone boundary, a fresh-context agent — one that did no work and owes the work nothing — reads the ledgers and must file challenges. The milestone cannot close until every challenge is answered. Its brief ends with: *do not suggest improvements; identify where the evidence does not support the claims.*

## The first test

The compiler project's first milestone under this regime was not a success story. That is why I trust it.

Mid-milestone, the agent retired a research idea — a lookahead search — on what looked like clean evidence. The critic checked the controls and found the main CCSD definition had been silently skipped: the "clean evidence" compiled in 0.11 seconds precisely because it never touched the hard part. The retirement was invalid. The agent's response:

> The critic's F-010 challenge is valid: the CCSD "unsplit" runs skipped the 63-term doubles definition. I'm reopening I-005 rather than defending the retirement.

In my [last post](https://br0kensmi1e.github.io/system-prompt/), the open problem was: how do you get an agent to question its chosen approach, not just polish it? Here is one answer. You don't wait for it to feel doubt. You install a critic whose challenges block progress, and you watch the questioning happen on schedule.

Over one milestone, the failure registry grew from 7 entries to 25. Three preliminary claims were retracted before I ever read them. The exam falsified an assumption I had written into the spec myself — I had declared FLOP quality guaranteed "by construction"; the gate failed on all six problems. The workflow caught its own constitution being wrong.

And the honest costs. The mistakes still *happened* — invalid controls, tests run on uncommitted code, an exam first run at the wrong sizes. The machinery caught them; it did not prevent them. Much of the milestone's effort went to evidence repair rather than compiler progress. The compiler still loses to its baseline. And the critic is launched by the agent it criticizes — teeth, yes; independence, not yet.

## What I do now

The last post ended with a complaint: the research was still my job. This project taught me what my job actually became.

I did not direct the work. I amended the rules. Every intervention I made was constitutional: "don't pin durations, use events." "The baseline is a baseline, not a component of our identity." "Each baseline keeps its own backend." "Failures generate ideas — link them." Those four sentences did more for the project than any line of code in it.

I think that's the real answer to "how do you get past *make this idea work*?" You stop trying to make the agent curious and start making wrongness expensive — then let the agent's competence, which was never the bottleneck, do the rest.

Still open: whether a critic launched by the criticized can be trusted with negative results, not just positive ones. Whether the machinery atrophies into ceremony once there's little left to catch. And whether a ledger full of honest failures converts, eventually, into a compiler that wins. The next milestone is the test.

You are not your agent's reviewer. You are its error-handling runtime. Build accordingly.

*Written with AI assistance from the original conversations and saved session records. Quotes are verbatim from session logs; `[…]` marks omissions. The compiler project is unfinished and currently loses to its baseline. The raw sessions, credentials, and private repositories are not published.*
