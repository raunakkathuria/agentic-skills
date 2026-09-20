---
name: jev-fit-review
description: >-
  Audit existing software for closed-set semantic decisions that TypeSafe's Jev
  model could replace or harden. Use when the user asks where Jev, TypeSafe, or
  System One fits, or asks to inspect prompt-and-parse flows, LLM judges or
  classifiers, semantic routing, manual triage, or unmeasured heuristic
  thresholds. Do not use for general AI ideation unless the user is asking about
  decision automation in an existing system. Produce a plain-language report
  with evidence, actionable steps, explicit non-fits, and checked request bodies.
license: MIT
compatibility: Requires internet access to read docs.typesafe.ai
metadata:
  author: raunakkathuria
  version: "1.1"
---

# Jev fit review

Find the decisions in a project that a System One model should make, prove why,
and say plainly where it should not be used.

## What Jev is

Jev is TypeSafe's System One model. It reads text and returns typed answers with
probabilities. It does not write text, code, or explanations. It reads text only:
strings, JSON objects, arrays.

You send a `state`, which is what the model should look at, and a set of
`questions` you name yourself. Each question is one of three types:

| Question type | Asks | You get back |
| --- | --- | --- |
| **Noul** | Is this true? | The probability of yes. There is no separate confidence. The number is the answer. |
| **Choice** | Which one of my options? | The option it picked, a probability for every option, and a `confidence`. |
| **Score** | How much, on my scale? | A number that can sit between two levels, and a `confidence`. |

The endpoint is `POST https://api.typesafe.ai/v1/systemone` with
`"model": "jev-latest"`.

**Read the live docs before you write any request body.** Start at
https://docs.typesafe.ai/llms.txt. Read at least
https://docs.typesafe.ai/api.md and https://docs.typesafe.ai/confidence.md.
Do not invent fields. If the docs cannot be reached, say so in the report and
mark every request body as unchecked.

## Process

Work in this order. Do not start writing the report until step 6 is done.

### Step 1. Understand the project

Read the README, the entry points, the config, and the CI workflows. Write two
or three sentences for yourself about what the system does and what it produces.
If the project is large, scope the review to the subsystem named by the user or
the subsystem closest to the request, and state that scope. Ask only when two
plausible scopes would materially change the result. Do not scan everything.

If a document disagrees with the code, trust the code. Say so in the report.

### Step 2. Find every decision point

A decision point is any place where the system chooses, judges, sorts, accepts,
rejects, or routes something. Look for these signs.

**Strong signs. Look here first.**

- A prompt that asks a model to reply in JSON, followed by parsing, fence
  stripping, a `try/catch`, or a retry. This is usually the best candidate in
  the whole project.
- A model used as a judge, grader, reviewer, or classifier, where only the
  verdict is used and the words are thrown away.
- Keyword lists, regular expressions, or `if` chains on strings that try to
  guess meaning.
- A queue where a person reads items and adds a label by hand.
- A limit or weight that somebody picked by hand and never measured.

**Medium signs.**

- Search or list results sorted by a formula that guesses at relevance.
- A check that can see the shape of something but not its meaning. For example,
  a test that names a requirement but never tests its behaviour.
- A fallback branch that exists because the model output was unreliable.

**Weak signs. Mention only if nothing better exists.**

- Work a person does today because nobody automated it yet.

For each decision point, write down where it lives, what it decides, what it
costs today, and what happens when it is wrong.

Look for measured evidence in tests, issue history, incidents, logs, and git
history. A parser or retry loop proves complexity, not failure frequency. If a
number is unknown, say what should be measured instead of estimating it.

### Step 3. Apply the fit test

A decision point **fits** when all of these are true:

- The answer is one of a set you can write down first, or a yes or no, or a
  position on a scale.
- The input is text or JSON you can assemble before the call.
- Meaning matters. Plain code cannot decide it with a rule.
- The answer is used by code, not shown to a person as writing.

It **does not fit** when any of these is true:

- The output must be written text, code, a document, or an explanation. Jev
  cannot write. Keep the LLM.
- The input is an image, audio, or video. Jev reads text only.
- A rule, a lookup, or a calculation already gives the exact answer. Keep the
  code. Never replace a rule that is always right with a probability.
- The set of possible answers is not known and cannot be built by code first.
- The decision cannot be undone and there is no safe path for uncertain cases.

Write down the rejected items too. A report that only says yes is not useful.

### Step 4. Choose the question type and write the question

| The answer is | Use | Note |
| --- | --- | --- |
| One of a set you define | Choice | Add a "none of these" option if nothing may fit. |
| Whether something is true | Noul | Use one Noul per label when several labels can apply at once. |
| A position on a scale | Score | Describe each level with a real situation. |

Rules:

- Ask one narrow thing per question. Split separate ideas into separate
  questions.
- Put the judgment in `instructions`. Put the possible answers in `criteria`.
- The question name is for your code only. It is never sent to the model, so
  the full meaning must sit inside the question.
- Give the model enough state: the source text, the policy, the facts it needs.
  Use named JSON fields when the state has several parts.
- Questions about the same state go in one request. They run at the same time
  and cannot see each other's answers.

Write a real request body for each opportunity, using the shape from the live
docs.

### Step 5. Rank by value, not by ease

Rank on two things and sort by the first.

- **Value.** How much does this cut cost, delay, wrong answers, or human effort?
  A step that runs on every item beats a step that runs once a week.
- **Effort.** How much code has to change? Prefer places where the workflow
  around them stays the same.

Put the highest value first, even when it is not the easiest.

### Step 6. Write the steps for each opportunity

Every opportunity gets three to six numbered steps the reader can follow.

**Step 1 of every list is the backtest.** Collect past cases where the correct
answer is already known. Run them through the proposed questions. Compare the
answers. Use the same data to choose the limits. This is never moved later in
the list. A reader who starts at step 2 is putting an unchecked judgment into
production.

**The last step names what gets deleted:** the parser, the retry loop, the
validator. Deleted code is the clearest proof that the change worked.

## Confidence and safety

Put this in every report.

- A high `confidence` means probability is concentrated on one Choice option or
  Score level. It does not mean the answer is correct, and it is not permission
  to act.
- A Noul near 0.5 means yes and no are about equally likely. It does not mean
  "medium amount".
- A typed answer guarantees the shape of the answer, not the truth of it.
- Set two limits for each decision. Above the first, the code acts alone. Below
  the second, the case goes to a person or to a slower model. Never guess the
  limits. They come from the backtest in step 6.
- A decision that can be undone and a decision that is final need different
  limits. The final one needs a higher limit.
- Keep API keys on the server. Never put one in client code.

## Write and verify the report

After completing the analysis above, read
[the report guide](references/report-guide.md) before drafting the answer. It
contains the default structure, required opportunity fields, worked example,
plain-language rules, and delivery checklist.

Use the full structure for a codebase-wide review. For a narrow question or a
user-specified output format, keep the same evidence, request validation,
non-fit analysis, backtest-first steps, and safety rules, but answer at the
requested depth.
