---
name: test-plan
description: Generate a one-page test plan (scope, risks, test strategy split into manual vs automated, definition of done) from a SkyPermit documentation file or directory, and save it to test-plans/<topic>.md. Use this whenever the user asks for a test plan, test strategy, QA plan, "what should we test", or test approach for any part of SkyPermit — a single doc like ../skypermit/docs/06-business-rules.md, a feature area like the state machine or gates, or the whole docs/ directory — including requests in Ukrainian or Russian ("тест-план", "план тестування", "стратегія тестування"). Not for writing individual test cases or Playwright code.
---

# Test plan from docs

Turn SkyPermit requirement docs into a **one-page** test plan that a QA engineer can read in two minutes and act on. The value of the plan is in its judgment — which areas carry the most risk, what is worth automating, what "done" means — not in restating the docs. Keep it short enough to fit on one page (roughly 60 lines / 600 words of Markdown); if you have more to say, you are listing test cases, which belong in `test-cases/`, not here.

## Inputs

The user passes a file or a directory, usually under `../skypermit/docs/`. If they name a topic instead of a path ("the gates", "the reviewer queue"), find the docs that cover it.

`../skypermit` is read-only: read anything there, never create, edit or delete files in it.

## Workflow

1. **Read the input in full.** For a directory, read every Markdown file in it.
2. **Read the overrides even if not passed.** `../skypermit/docs/05-open-questions.md` and `07-clarifications.md` hold the current status of unresolved and resolved questions and take precedence over earlier docs. A plan that tests against a superseded rule, or states an expected result for something still open, is worse than no plan.
3. **Check existing coverage.** Skim the file names and `describe`/`it` titles in `../skypermit/tests/` (`business-rules/`, `lib/gates/`, `routes/`) to see what Vitest already covers at unit level, so the strategy can build on it instead of duplicating it.
4. **Write the plan** using the template below and save it to `test-plans/<topic>.md`, where `<topic>` is a short kebab-case name for the input (`business-rules`, `state-machine`, `full-system` for the whole docs directory). If that file already exists, ask before overwriting.
5. **Reply** with the file path and a 2–3 line summary (top risks, anything blocked). Do not paste the whole plan back into chat.

## Template

Write the plan in English and use exactly these four sections; there is no separate section for open questions or traceability — those fold into scope and risks as described below.

```markdown
# Test Plan: <Topic>

**Source:** <input path(s)> · **Date:** <YYYY-MM-DD>

## Scope

**In scope**
- <area> — <what is verified> (<doc> §<section>)

**Out of scope**
- <area> — <why: covered elsewhere / not in this input / blocked by open question N>

## Risks

| # | Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- | --- |
| R1 | <what could go wrong for users or the business> | H/M/L | H/M/L | <which test or activity addresses it> |

## Strategy

| Level | Tool | Focus | Existing coverage |
| --- | --- | --- | --- |
| Unit | Vitest (`../skypermit/tests`) | <logic best checked in isolation> | <what is already there / gaps> |
| API | Playwright (`request`) | <endpoints, status codes, `422` body> | — |
| E2E UI | Playwright (`getByTestId`) | <critical user journeys> | — |
| Manual | Exploratory | <UX, edge cases hard to automate, areas tied to open questions> | — |

## Definition of Done

- [ ] <measurable exit criterion>
```

## Writing each section well

**Scope.** Name concrete areas and what about them is verified (e.g. "`AIRCRAFT` gate — weight band thresholds and `RETIRED` status"), not generic phrases like "functional testing". Add a short source reference in parentheses so a reader can find the requirement. Anything blocked by an open question goes under *Out of scope* with the question number, so nobody writes a test with a guessed expected result.

**Risks.** 4–7 rows, ordered by severity. Ground each in something specific in the docs: a boundary value, a state transition that must be rejected, an auth header, an ambiguity. An unresolved open question is itself a risk (the behaviour may change or be undefined) — list it here with mitigation such as "exclude from automation; manual check once resolved". Rate likelihood by how error-prone the rule looks (many thresholds, ordering, time zones → higher), impact by user or business consequence (approving an illegal flight → H).

**Strategy.** Assign each area to the lowest level that can check it reliably: pure rule logic to unit, contract and status codes to API, user journeys and the `data-testid` contract from `04-frontend-spec.md` to E2E, and judgment-heavy or undefined behaviour to manual. The point of the *Existing coverage* column is to stop the team re-testing what Vitest already proves; say "none found" if you looked and found nothing. Omit a row if the input genuinely has nothing for that level (e.g. a pure data-model doc has no UI journeys).

**Definition of Done.** 4–6 checklist items that someone can tick objectively — e.g. "every `rule_reference` value in 06 §6 has at least one passing API test", "all H-impact risks have a passing mitigation test", "no open P1 defects". Avoid unmeasurable items like "quality is acceptable".

## Things to avoid

- Expected results for behaviour the docs mark as open or undefined.
- Listing individual test cases or steps — that is a different deliverable.
- Inventing features, endpoints or thresholds not in the docs; if the input is thin, say so in the plan rather than padding it.
