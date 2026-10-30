# How do you know your agent is wrong?

In my [last post](https://br0kensmi1e.github.io/system-prompt/) I replaced an elaborate agent workflow with a minimal system prompt. That fixed procedure-following. It left a harder problem open: the agent commits to an approach and rarely questions whether the approach is worth pursuing.

Since then I ran two more agent-research projects. One worked. One failed. This post is about the difference — and the failure taught me more, because of *how* it failed.

## The win that wasn't

The failed project was a tensor-compiler research loop. A coding agent iteratively improved a scheduler for quantum-chemistry tensor expressions, with measured wall time as feedback. After some weeks it reported a result: its selector beat gristmill, the state-of-the-art baseline, by 1.42× on CCSD. Verified schedules. Interleaved timings. The claim made it into the contribution report.

Then one evening I looked at the comparison and typed:

> i don't understand. how does the result perform compared to gristmill baseline? and i said DO NOT compare to naive, it is meaningless.

The agent's reply came fast:

> You're right, and the fix is clear: every presented number needs a gristmill baseline, or it doesn't get presented.

It rebuilt the comparison properly — same equations fed to both sides — and the 1.42× win evaporated. The baseline had been handed a weaker version of the problem. Geomean across six problems: 0.90×. We were *behind*.

Nothing in the loop caught this. The loop had trials logs, held-out problem sets, exact symbolic verification of every candidate. It had the *form* of a careful setup. But the one wrongness that mattered — a baseline too weak to mean anything — produced no signal anywhere. The only error detector in the system was me, reading numbers at night.

## Why did the other project work?

The project that worked was an STM image analyzer, built by the same kind of loop. It ended with eighteen named, mechanism-verified failure modes, a sealed exam it passed honestly (including documented failures), and calibrated uncertainties that generalized to unseen data.

Same agent. Same loop idea. Opposite outcomes.

For a while I thought the difference was the domain: STM fixture evaluation is deterministic and cheap; wall-time benchmarking is noisy and slow. That's true, but it's not the interesting part. The interesting part showed up when I read the two agents' *thinking* side by side.

Here is the STM agent, mid-diagnosis:

> noise/2000: residual 14.3° even after outlier rejection — […] but why is grid residual 14°? Because true ring pairs themselves were evicted as "outliers" before the noise spikes (if noise spikes dominate the fit offset).

And here is the compiler agent, also mid-diagnosis:

> ADC2 selected appears almost naive, why no biclique? Perhaps terms split into defs >20 and skip, or initial has only 8 terms output etc. Should inspect problem ADC2 definitions terms. Use Python summarize.

And later, still diagnosing:

> Old candidate perhaps generated at sizes100,1000 not16,64! […] The `select_heldout` perhaps commands not stored.

See the difference in *shape* before you see it in content. The STM agent's thinking starts from a localized fact — which fixture, which quantity, how far off — because its evaluator emits per-fixture residuals against known ground truth. The failure arrives with the "why" attached. The compiler agent's thinking starts from a bare artifact and a scatter of "perhaps." Its environment emitted one scalar per run. Before it could diagnose anything, it had to *build* inspection tooling, and then guess which of three stories was true.

One agent thinks in observations. The other thinks in hypotheses. That is the whole difference, and it has nothing to do with how smart the agent is.

## Numbers don't carry their own validity

Once I saw this, I found it everywhere in the compiler project's records.

A schedule got timed and showed a win. The timing run had no idea the schedule was numerically wrong — correctness lived in a *different command* nobody ran at the time. The win looked exactly like a real win.

The same baseline was measured twice, in two honest, protocol-compliant sessions: 58.7 ms and 67.9 ms. Fifteen percent apart. Nothing flagged the discrepancy, and two downstream conclusions silently depended on which number was right. There was no way to adjudicate, because there is no ground truth for wall time — only measurements of it.

A headline geomean rested on measurements with too-few iterations and a 3.4× spread between repeats of the *same candidate* — visible right there in the file, for weeks. Nobody was assigned to look.

In the STM project, a wrong answer came with its location and size. In the compiler project, wrongness lived at *intersections* — timing ∩ correctness, session ∩ session, claim ∩ fine print — and intersections only get checked when someone with judgment decides to look. For the failed project, that someone was me.

## What would make wrongness visible?

Debugging is my mental model here. Programming environments make wrongness visible in four ways, and agent research environments usually have none of them:

**Detectable.** A failing build produces a signal. A silently-wrong number does not. Most agent loops emit rewards, not alarms.

**Diagnosable.** A stack trace says *where*. A bare "test failed" does not. This is the difference between a per-fixture residual and a single wall-time scalar — between getting the "why" with the failure and manufacturing it yourself, one "perhaps" at a time.

**Expensive to ignore.** A compiler warning gets ignored; a compiler error blocks the build. This is the property people underestimate. The failed project *disclosed* its weak baseline, in a paragraph, and then kept going — disclosure without enforcement is a warning, and warnings get ignored. In the STM project, by contrast, the scoring priced a confident wrong answer at ten times a refusal. As that agent put it: *a refusal is a delay; a lie is fake science.*

**Remembered.** A bug fixed without a postmortem recurs. An agent session that ends forgets. The STM project froze every failure into a permanent regression test; the compiler project re-derived the same lessons across context resets.

So the fix was not a better prompt, or a smarter agent, or a fancier loop. It was building these four properties into the environment.

## The constitution

For the new compiler project, the workflow moved out of the prompt and into six small, versioned files in the repo: a protocol (how work happens), a spec (what counts as success, with the exact commands that measure it), a failure registry (mechanism verified *before* the fix, fixture pinned first), an idea ledger (every research idea carries its own kill condition, written before the experiment), an append-only trial log, and a compressed research summary.

The system prompt shrank to six lines: read these files, work autonomously, stop only at milestones or blockers. If the prompt and the files disagree, the files win.

Two pieces deserve special mention because they came straight from the failures above.

The **protected exam**: a benchmark suite that runs once per milestone, reported as-is, never iterated against. And the **critic**: at each milestone boundary, a fresh-context agent — one that did no work and owes the work nothing — reads the ledgers and must file challenges. The milestone cannot close until every challenge is answered. The critic's brief ends with: *do not suggest improvements; identify where the evidence does not support the claims.*

In the failed project, that critic role existed. It was me, at night, with the "DO NOT compare to naive" message. The new design institutionalizes exactly that intervention.

## The first test

The first milestone under the new regime was not a success story. That is why I trust it.

The exam ran and stopped immediately at the first gate: a previously shipped schedule passed exact symbolic verification but failed the numerical oracle — worst relative error 1.985. The agent's report:

> The protected exam stopped immediately at Axis 0: `mp3_selected` passes `truthmill::verify` but fails the numerical oracle at `(3,5)` with worst relative error 1.985. No exam timings were run, so there are no new performance claims.

No timings, no claims. The gate was more important than the result.

Then the agent retired a research idea — a lookahead search — on what looked like clean evidence, and wrote the reasoning down. The critic checked the controls and found the main CCSD definition had been silently skipped: the "clean evidence" compiled in 0.11 seconds precisely because it never touched the hard part. The retirement was invalid. The agent's response:

> The critic's F-010 challenge is valid: the CCSD "unsplit" runs skipped the 63-term doubles definition. I'm reopening I-005 rather than defending the retirement.

This is the sentence the whole design was for. In my last post, the open problem was: how do you get an agent to question its chosen approach, not just polish it? Here is one answer: you don't wait for it to feel doubt. You install a critic whose challenges block progress, and you watch the questioning happen on schedule.

Over the milestone, the failure registry grew from 7 entries to 25. Three preliminary claims were retracted before I ever read them — including a "win" that had been timed on the oracle-failing schedule above. And the exam falsified an assumption I had written into the spec myself: I had claimed FLOP quality was guaranteed "by construction." It wasn't; the gate failed on all six problems. The workflow caught its own constitution being wrong.

And the honest costs. The mistakes still *happened* — invalid controls, tests run on uncommitted code, an exam run at the wrong sizes. The machinery caught them; it did not prevent them. Much of the milestone's effort went to evidence repair rather than compiler progress. The compiler still loses to its baseline. And the critic is launched by the agent it criticizes — teeth, yes; independence, not yet.

## What I do now

The last post ended with a complaint: the research was still my job. This project taught me what my job actually became.

I did not direct the work. I amended the rules. Every intervention I made was constitutional: "don't pin durations, use events." "The baseline is a baseline, not a component of our identity." "Each baseline keeps its own backend." "Failures generate ideas — link them." Those four sentences did more for the project than any line of code in it.

I think that's the real answer to "how do you get past *make this idea work*?" You stop trying to make the agent curious and start making wrongness expensive — then let the agent's competence, which was never the bottleneck, do the rest.

Still open: whether a critic launched by the criticized can be trusted with negative results, not just positive ones. Whether the machinery atrophies into ceremony once there's little left to catch. And whether a ledger full of honest failures converts, eventually, into a compiler that wins. The next milestone is the test.

*Written with AI assistance from the original conversations and saved session records. The projects described are real; the compiler one is unfinished. Quotes are verbatim from session logs, with […] marking omissions.*
