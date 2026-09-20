# Report guide

Read this after completing the fit analysis and before drafting the answer.

## Default report format

Use these sections, in this order, for a codebase-wide review.

```text
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
- What is new: the API key, the network call, and the outside company.
- What happens when the service is slow or down. Say whether the step should
  fail open or fail closed, and why.
```

For a narrow question or a user-specified format, preserve the evidence,
request validation, non-fit analysis, backtest-first steps, and safety rules.
Do not force the full structure when it would obscure the answer.

Each opportunity in section 3 has seven parts, in this order:

1. **Where it is now.** The file and the lines.
2. **What happens today.** The current method, in two or three sentences.
3. **The problem.** What goes wrong, or what it costs. Use measured numbers.
4. **The Jev version.** A real request body in a code block.
5. **What gets better.** Two to four bullet points.
6. **Steps to try this.** The numbered steps from the fit-review process.
7. **Limits.** When the code acts alone. When a person decides.

In cost and risk, verify current pricing from a first-party source before giving
a monetary estimate. Otherwise mark financial cost as unknown. State whether a
Jev call was executed; checking the request shape does not prove model quality.

## Worked example

This shows the required tone and level of detail. Copy the sentence length and
word choice, not the facts or numbers.

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
>
> - A fourth team is not possible. You wrote the list of options.
> - The parsing code and the retry loop can be deleted.
> - You get a confidence number. The old prompt gave you nothing to measure.
>
> **Steps to try this:**
>
> 1. Take past tickets that people already routed by hand. Run them through the
>    question above. Compare the answers with the reviewed ground truth.
> 2. Look at the cases it got wrong. Check whether the option descriptions were
>    unclear, or whether the ticket was genuinely hard.
> 3. Using the same tickets, find the confidence level above which the model is
>    right often enough for the cost of a mistake.
> 4. Add the call behind a flag. Run it in shadow mode before it controls work.
> 5. Turn on automatic routing only for the backtested confidence range.
> 6. Delete the fence stripping, parser, and retry loop in
>    `src/support/route.ts`.
>
> **Limits:** Send the ticket automatically above the level found in step 3.
> Send it to a human queue below the lower backtested level.

Use the same real example throughout one report. Do not invent a new example
for each section.

## Plain-language rules

The reader may not speak English as a first language. Check these rules before
delivery.

### The six that matter most

**1. Explain a term before using it.** Do not name an idea and then rely on it
in the next sentence. Explain what it means, then use the name.

**2. Run one example through the report.** Pick one real decision from the
project. Use it in the summary, detailed section, and request body. Keep names
and numbers consistent.

**3. Use the concrete word.** Write "checking code", not "validation logic".
Write "a rule that is always right", not "a deterministic check".

**4. Keep facts and instructions apart.** Do not mix what happens today with
what the reader should do in the same paragraph, list item, or table cell.

**5. Say the idea directly.** Do not make the reader decode a metaphor.

- Weak: "You are asking a writing machine to pretend it is a function."
- Better: "The model was built to write. You needed it to choose."

**6. If one visual is useful, put it at the end.** The reader should understand
the idea before seeing its visual summary. Never use more than one.

### Sentences

- One idea per sentence.
- Aim for 15 words. Do not exceed 25 without a concrete reason.
- Use active voice. Write "the code sends the ticket", not "the ticket is sent".
- Use the same word for the same thing throughout the report.

### Words

- Use common words. Write "use" not "leverage" and "start" not "initiate".
- Avoid idioms such as "low-hanging fruit" or "moving the needle".
- Use one verb where possible. Write "reduce", not "cut down on".
- Explain each technical term once when it first appears.
- Do not use contractions.
- Do not double a noun already contained in an acronym. Write "LLMs", not
  "LLM models".

### Numbers and claims

- Give measured numbers where available.
- When a number is unknown, write `unknown` and say what to measure.
- Do not turn a vendor claim into a project fact. Name the first-party source
  for any vendor price, speed, or benchmark.
- Separate code evidence, historical evidence, and inference.

### Layout

- Keep paragraphs to three sentences or fewer.
- Use tables only when they make comparisons easier to scan.
- Put code in code blocks.
- Use numbered lists for ordered steps and bullets for unordered items.

If the user asks for another language, write in that language and keep the
same evidence and safety requirements.

## Before delivery

- [ ] Every request body matches the current API docs.
- [ ] The report says whether a Jev call was executed.
- [ ] The non-fit section names at least one rejected part.
- [ ] Every opportunity has numbered steps.
- [ ] The backtest is step 1 of every opportunity.
- [ ] One real example stays consistent throughout the report.
- [ ] No exact rule was replaced with a probability.
- [ ] Confidence is described as concentration on one outcome, not correctness.
- [ ] Monetary estimates cite current first-party pricing or remain unknown.
- [ ] The answer follows the plain-language rules.
