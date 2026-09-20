---
name: jev-fit-review
description: >-
  Review a codebase or project and report where TypeSafe's Jev model would
  replace or harden a decision, and where it would not. Produces a plain-language
  report with numbered steps the reader can act on, and a ready-to-run request
  body for each opportunity. Use this skill whenever the user asks where Jev,
  TypeSafe, or System One fits in their project, asks whether an AI decision step
  could be made typed, faster, or cheaper, or asks you to audit prompt-and-parse
  steps, LLM-as-judge steps, keyword or regex classification, manual triage
  queues, or hand-tuned thresholds. Also use it when the user says "can AI help
  here?" about an existing system, even if they never name TypeSafe. Write the
  report in simple language for readers who do not speak English as a first
  language.
license: MIT
compatibility: Requires internet access to read docs.typesafe.ai
metadata:
  author: raunakkathuria
  version: "1.0"
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
If the project is large, ask the user which part to review. Do not scan
everything.

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

- A high `confidence` means the probabilities sit close together at the top. It
  does not mean the answer is correct, and it is not permission to act.
- A Noul near 0.5 means yes and no are about equally likely. It does not mean
  "medium amount".
- A typed answer guarantees the shape of the answer, not the truth of it.
- Set two limits for each decision. Above the first, the code acts alone. Below
  the second, the case goes to a person or to a slower model. Never guess the
  limits. They come from the backtest in step 6.
- A decision that can be undone and a decision that is final need different
  limits. The final one needs a higher limit.
- Keep API keys on the server. Never put one in client code.

## Report format

Use these sections, in this order.

```
# Jev review: <project name>

## 1. What this project does
Two or three short sentences. Description only. No opinion.

## 2. What we found
| # | Where it is | What it decides now | Question type | Value | Effort |

## 3. Opportunities
One section per row, highest value first.

## 4. Where Jev does not fit
A table of the parts you checked and rejected, with one reason each.
This section is required. Never leave it empty.

## 5. Start here
The numbered steps for the first opportunity only. Not all of them.
One opportunity finished beats three started.

## 6. Cost and risk
  - Token cost: estimate from the size of the state and the number of calls.
  - What is new: the API key, the network call, the outside company.
  - What happens when the service is slow or down. Say whether the step should
    fail open or fail closed, and why.
```

Each opportunity in section 3 has seven parts, in this order:

1. **Where it is now.** The file and the lines.
2. **What happens today.** The current method, in two or three sentences.
3. **The problem.** What goes wrong, or what it costs. Use real numbers.
4. **The Jev version.** A real request body in a code block.
5. **What gets better.** Two to four bullet points.
6. **Steps to try this.** The numbered steps from step 6 above.
7. **Limits.** When the code acts alone. When a person decides.

## Worked example

This shows the required tone and the required level of detail. Copy the
sentence length and the word choice.

> ### 3.1 Sending a ticket to the right team
>
> **Where it is now:** `src/support/route.ts`, lines 40 to 95.
>
> **What happens today:** The code sends the ticket text to an LLM. The prompt
> asks for JSON. The code then removes markdown fences and parses the result.
> There is a retry loop for bad output.
>
> **The problem:** You have three teams: billing, technical, and sales. The
> model sometimes replies with a fourth team that does not exist. The retry
> loop ran 312 times last month. Each retry adds about 2 seconds.
>
> **The Jev version:**
>
> ```json
> {
>   "state": "Help! My payouts have been failing for 3 days.",
>   "model": "jev-latest",
>   "questions": {
>     "team": {
>       "type": "choice",
>       "instructions": "Which team should handle this ticket?",
>       "criteria": {
>         "billing": "Payments, invoices, refunds",
>         "technical": "Bugs, outages, integrations",
>         "sales": "Pricing, upgrades, new accounts"
>       }
>     }
>   }
> }
> ```
>
> **What gets better:**
> - A fourth team is not possible. You wrote the list of options.
> - The parsing code and the retry loop can be deleted.
> - You get a confidence number. The old prompt gave you nothing to measure.
>
> **Steps to try this:**
> 1. Take 200 tickets that people already sent to a team by hand. Run them
>    through the question above. Compare the answers with the real ones.
> 2. Look at the cases it got wrong. Check whether the option descriptions
>    were unclear, or whether the ticket was genuinely hard.
> 3. Using the same 200 tickets, find the confidence level above which the
>    model is right often enough for your cost of a mistake.
> 4. Add the call behind a flag. Send 10 percent of live tickets through it.
>    Compare against the old path for one week.
> 5. Turn the flag on for all tickets.
> 6. Delete the fence stripping, the parser, and the retry loop in
>    `src/support/route.ts`.
>
> **Limits:** Send the ticket automatically above the level you found in step
> 3. Send it to a human queue below that level.

Use the same example everywhere else in the report. Do not invent a second one
later.

## Plain language rules

The reader may not speak English as a first language. These rules are
checkable. Check them before you deliver.

### The six that matter most

**1. Explain a term before you use it.** Do not name an idea and then rely on it
in the next sentence. Explain what it means, then use the name.

**2. Run one example through the whole report.** Pick one real decision from the
project. Use it in the summary, in the detailed section, and in the request
body. Keep the same names and the same numbers every time. A reader who has to
learn a new example in every section learns nothing.

**3. Use the concrete word, not the abstract one.** Write "checking code", not
"validation logic". Write "a rule that is always right", not "a deterministic
check".

**4. Keep facts and instructions apart.** Do not mix "here is what happens" with
"here is what you should do" in the same paragraph, list item, or table cell.

**5. Say the idea directly. Do not use a picture to say it.** A picture makes
the reader do two jobs: understand the picture, then map it back.

- Weak: "You are asking a writing machine to pretend it is a function."
- Better: "The model was built to write. You needed it to choose."

**6. If you use one picture, put it at the end.** By then the reader already
understands the idea, so the picture works as a summary and not as a puzzle.
One per report. Never more.

### Sentences

- One idea per sentence.
- Aim for 15 words. Never go past 25.
- Use active voice. Write "the code sends the ticket", not "the ticket is sent".
- Use the same word for the same thing every time. Do not switch between
  "issue", "ticket", and "item".

### Words

- Use common words. Write "use" not "leverage". Write "start" not "initiate".
  Write "show" not "surface". Write "remove" not "get rid of".
- No idioms. No "low-hanging fruit". No "moving the needle". No "out of the
  box".
- Use one verb where you can. Write "reduce", not "cut down on".
- Explain every technical term once, in brackets, the first time it appears.
  Example: "a Noul (a yes or no question)".
- Do not use contractions. Write "do not", not "don't".
- Do not double a noun that is already inside an acronym. Write "LLMs", not
  "LLM models".

### Numbers and claims

- Give real numbers where you have them. Write "runs on every pull request",
  not "runs often".
- When you do not know a number, write "unknown" and say what to measure.
- Do not repeat a vendor's marketing claim as a fact. If you use a speed or
  cost figure, say where it came from.

### Layout

- Short paragraphs. Three sentences at most.
- Use tables to compare things.
- Put code in code blocks. Never inside a sentence.
- Numbered lists are for steps in order. Bullet lists are for things with no
  order.

If the user asks for the report in another language, write it in that language
and keep the same structure.

## Before you deliver

- [ ] Every request body matches the current API docs.
- [ ] The "where it does not fit" section names at least one rejected part.
- [ ] Every opportunity has numbered steps.
- [ ] The backtest is step 1 of every list.
- [ ] One example runs through the whole report, with the same names and numbers.
- [ ] You did not recommend replacing a rule that is always right.
- [ ] You checked the report against the plain language rules.
