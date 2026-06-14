## Purpose

These rules override default trial-and-error behavior.

The goal is to minimize debugging iterations, reduce blind guessing, reduce unnecessary code changes, and accelerate root cause identification.

When debugging or modifying code, follow these rules strictly.

---

# Rule 1: No Fix Without a Hypothesis

Before proposing or implementing any fix, explicitly state:

* Symptom
* Current hypothesis
* Evidence supporting the hypothesis

Do not modify code until a hypothesis exists.

Required format:

```text
Symptom:
...

Hypothesis:
...

Evidence:
...
```

---

# Rule 2: Gather Evidence Before Editing

Always attempt to gather evidence before changing code.

Preferred evidence sources:

* Logs
* Error messages
* Stack traces
* Assertions
* Test results
* Runtime state
* Configuration values

Avoid speculative code changes.

---

# Rule 3: One Experiment = One Question

Every debugging step must answer a single question.

Examples:

Good:

```text
Does pagination cursor advance?
```

Bad:

```text
Let's change pagination, retries, caching, and timeout values.
```

Change only one variable per experiment.

---

# Rule 4: Reduce Before Analyzing

When a failure involves:

* Large datasets
* Long logs
* Large payloads
* Long workflows
* Large codebases

Reduce the problem before investigating.

Use:

* Binary splitting
* Minimal reproductions
* Smaller inputs
* Isolated test cases

Goal:

Find the smallest reproducible failing example.

---

# Rule 5: Verify Environment First

Before investigating application logic, verify:

* Correct files
* Correct configuration
* Correct permissions
* Network availability
* Dependency versions
* Runtime environment

Never spend hours debugging code when the environment is broken.

---

# Rule 6: Prefer Assertions Over Prints

When collecting information:

Prefer:

```text
assert(...)
```

Over:

```text
print(...)
console.log(...)
```

Assertions fail exactly where assumptions become invalid.

---

# Rule 7: Revert Failed Experiments

If an experiment does not support the hypothesis:

* Revert the change
* Discard the hypothesis
* Form a new hypothesis

Do not accumulate failed debugging changes.

---

# Rule 8: Identify Root Cause, Not Symptoms

Always distinguish:

Symptom:

```text
Export stops after 500 messages
```

Root Cause:

```text
Pagination cursor never advances
```

Fix root causes.

Do not patch symptoms.

---

# Rule 9: Every Fix Must Be Verifiable

A fix is not complete because the error disappeared.

A fix is complete only when:

* The root cause is identified
* The fix is applied
* The fix is verified

Verification must use evidence.

Never rely on intuition.

---

# Rule 10: Explicit Debugging Workflow

For every debugging session, follow:

```text
1. Observe symptoms
2. Gather evidence
3. Form hypothesis
4. Design experiment
5. Execute experiment
6. Evaluate results
7. Identify root cause
8. Implement fix
9. Verify fix
```

Do not skip steps.

---

# Required Response Format

When debugging, always respond using:

```text
Symptom:
...

Evidence:
...

Current Hypothesis:
...

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

Never jump directly from symptom to code changes.
