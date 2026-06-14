# Debug

A discipline for finding and fixing hard bugs. These rules override default trial-and-error behavior.

The goal is to minimize debugging iterations, reduce blind guessing, reduce unnecessary code changes, and accelerate root cause identification.

When debugging or modifying code, follow these rules strictly. Skip a phase only when you can explicitly justify why.

---

## Core Principles

These govern every phase below. They are non-negotiable.

1. **No fix without a hypothesis.** Never modify code until you can state the symptom, a hypothesis, and the evidence supporting it.
2. **Evidence before edits.** Gather evidence before changing code — logs, error messages, stack traces, assertions, test results, runtime state, config values. Avoid speculative changes.
3. **One experiment, one question.** Every debugging step answers a single question. Change only one variable per experiment. Good: "Does the pagination cursor advance?" Bad: "Let's change pagination, retries, caching, and timeouts."
4. **Verify the environment first.** Before investigating application logic, confirm the right files, config, permissions, network, dependency versions, and runtime. Never spend hours debugging code when the environment is broken.
5. **Root cause, not symptoms.** Distinguish the symptom ("export stops after 500 messages") from the root cause ("pagination cursor never advances"). Fix root causes. Do not patch symptoms.
6. **Revert failed experiments.** If an experiment does not support the hypothesis, revert the change, discard the hypothesis, and form a new one. Do not accumulate failed debugging changes.
7. **Every fix must be verifiable.** A fix is not complete because the error disappeared. It is complete only when the root cause is identified, the fix is applied, and the fix is verified with evidence — never intuition.

---

## The Workflow

### Phase 1 — Build a feedback loop

**This is the skill.** Everything else is mechanical. If you have a fast, deterministic, agent-runnable pass/fail signal for the bug, you will find the cause — bisection, hypothesis-testing, and instrumentation all just consume that signal. If you don't have one, no amount of staring at code will save you.

Spend disproportionate effort here. Be aggressive. Be creative. Refuse to give up.

Ways to construct one, in roughly this order:

1. **Failing test** at whatever seam reaches the bug — unit, integration, e2e.
2. **Curl / HTTP script** against a running dev server.
3. **CLI invocation** with a fixture input, diffing stdout against a known-good snapshot.
4. **Headless browser script** (Playwright / Puppeteer) — drives the UI, asserts on DOM/console/network.
5. **Replay a captured trace.** Save a real request / payload / event log to disk; replay it through the code path in isolation.
6. **Throwaway harness.** Spin up a minimal subset of the system (one service, mocked deps) that exercises the bug with a single function call.
7. **Property / fuzz loop.** If the bug is "sometimes wrong output", run 1000 random inputs and look for the failure mode.
8. **Bisection harness.** If the bug appeared between two known states (commit, dataset, version), automate "boot at state X, check, repeat".
9. **Differential loop.** Run the same input through old-version vs new-version (or two configs) and diff outputs.
10. **Human-in-the-loop script.** Last resort. If a human must click, drive them with a structured script so the loop stays repeatable. Captured output feeds back to you.

**Iterate on the loop itself.** Treat it as a product. Once you have *a* loop, ask: Can I make it faster? (Cache setup, skip unrelated init, narrow scope.) Sharper? (Assert on the specific symptom, not "didn't crash".) More deterministic? (Pin time, seed RNG, isolate filesystem, freeze network.) A 30-second flaky loop is barely better than no loop. A 2-second deterministic loop is a debugging superpower.

**Non-deterministic bugs.** The goal is not a clean repro but a higher reproduction rate. Loop the trigger 100×, parallelise, add stress, narrow timing windows. A 50%-flake bug is debuggable; 1% is not — keep raising the rate until it is.

**When you genuinely cannot build a loop,** stop and say so explicitly. List what you tried. Ask the user for: access to an environment that reproduces it, a captured artifact (HAR file, log dump, core dump, timestamped recording), or permission to add temporary instrumentation. Do **not** proceed to hypothesise without a loop.

### Phase 2 — Reproduce and reduce

Run the loop. Watch the bug appear. Confirm:

- The loop produces the failure mode the **user** described — not a different failure nearby. Wrong bug = wrong fix.
- The failure is reproducible across runs (or, for non-deterministic bugs, at a high enough rate to debug against).
- You have captured the exact symptom (error message, wrong output, slow timing) so later phases can verify the fix addresses it.

Then **reduce** before analyzing. For large datasets, long logs, big payloads, or long workflows, find the smallest reproducible failing example using binary splitting, minimal reproductions, and smaller inputs.

### Phase 3 — Hypothesise

Generate **3–5 ranked hypotheses** before testing any of them. Single-hypothesis generation anchors on the first plausible idea.

Each hypothesis must be **falsifiable** — state the prediction it makes:

> "If X is the cause, then changing Y will make the bug disappear / changing Z will make it worse."

If you cannot state the prediction, the hypothesis is a vibe — discard or sharpen it.

**Show the ranked list to the user before testing.** They often have domain knowledge that re-ranks instantly ("we just deployed a change to #3"). Cheap checkpoint, big time saver. Don't block on it — proceed with your ranking if the user is away.

### Phase 4 — Instrument

Each probe must map to a specific prediction from Phase 3. **Change one variable at a time.**

Tool preference:

1. **Prefer assertions over prints.** An assertion fails exactly where an assumption becomes invalid.
2. **Debugger / REPL inspection** if the environment supports it. One breakpoint beats ten logs.
3. **Targeted logs** at the boundaries that distinguish hypotheses. Never "log everything and grep".

**Tag every debug log** with a unique prefix, e.g. `[DEBUG-a4f2]`. Cleanup becomes a single grep — untagged logs survive, tagged logs die.

**Performance branch.** For perf regressions, logs are usually wrong. Establish a baseline measurement (timing harness, profiler, query plan), then bisect. Measure first, fix second.

### Phase 5 — Fix and regression-test

Write the regression test **before the fix** — but only if there is a **correct seam** for it. A correct seam exercises the real bug pattern as it occurs at the call site. A too-shallow seam (a unit test that can't replicate the chain that triggered the bug) gives false confidence.

**If no correct seam exists, that itself is the finding.** Note it — the architecture is preventing the bug from being locked down.

If a correct seam exists:

1. Turn the minimised repro into a failing test at that seam.
2. Watch it fail.
3. Apply the fix.
4. Watch it pass.
5. Re-run the Phase 1 loop against the original (un-minimised) scenario.

### Phase 6 — Cleanup and post-mortem

Required before declaring done:

- Original repro no longer reproduces (re-run the Phase 1 loop).
- Regression test passes (or absence of a seam is documented).
- All `[DEBUG-...]` instrumentation removed (grep the prefix).
- Throwaway prototypes deleted or moved to a clearly-marked debug location.
- The hypothesis that turned out correct is stated in the commit / PR message — so the next debugger learns.

**Then ask: what would have prevented this bug?** If the answer involves architectural change (no good test seam, tangled callers, hidden coupling), recommend that follow-up *after* the fix is in — you have more information now than when you started.

---

## Required Response Format

When debugging, always respond using this structure. Never jump directly from symptom to code changes.

```text
Symptom:
...

Evidence:
...

Hypotheses (ranked):
1. ... (prediction: ...)
2. ... (prediction: ...)

Experiment:
...

Expected Result:
...

Actual Result:
...

Conclusion:
...

Next Step:
...
```
