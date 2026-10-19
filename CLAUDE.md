# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

QA kit for **SkyPermit**, a system for registering drone operators, pilots and aircraft, and for requesting and reviewing authorisation to fly in airspace zones. The application under test lives in the sibling repository `../skypermit` (Next.js 16, React 19, Supabase, Vitest).

Main work in this repository:

1. **Test cases and test plans** derived from the documentation in `../skypermit/docs/` (primary work).
2. **Playwright automated tests** (occasionally).

## `../skypermit` is read-only

You may read anything in `../skypermit` (docs, source code, existing tests) to understand how the system behaves. **You must not:**

- modify, create, move or delete any file there;
- run git operations there that change anything (`commit`, `add`, `push`, `checkout`, `reset`, `stash`, etc.);
- run commands there that change the working tree (`npm install`, formatters, code generation).

All artifacts (test cases, plans, automated tests, scripts) are created only in this repository. If something in SkyPermit needs to change, describe it to the user instead of changing it yourself.

## Requirement sources

The docs in `../skypermit/docs/` are the source of truth for test design:

| File | What to use it for |
| --- | --- |
| `01-data-model.md` | entities, fields, enums, constraints: validation and boundary values |
| `02-state-machine.md` | flight request states and legal transitions: positive and negative transition tests |
| `03-api-contract.md` | endpoints, status codes, the `422` error body: API checks |
| `04-frontend-spec.md` | the two portals, pages, the `data-testid` contract: UI scenarios and Playwright locators |
| `06-business-rules.md` | logic of the four automated gates, thresholds, zone catalogue: business rule checks |
| `05-open-questions.md`, `07-clarifications.md` | open and resolved questions, clarifications: these take precedence over earlier docs |
| `08-uml-diagrams.md` | flow diagrams |

Rules:

- Do not invent behaviour that is not in the docs. If a requirement is ambiguous or touches an open question from `05-open-questions.md`, mark it in the test case (as an assumption or a blocker) rather than presenting a guess as the expected result.
- Every test case references its requirement (docs file and section) for traceability.
- Before writing test cases, check the existing unit tests in `../skypermit/tests/` (`business-rules/`, `lib/gates/`, `routes/`) to see what is already covered.

## Planned structure

```text
test-cases/      # test cases grouped by area (entity, state machine, API, UI, gates)
test-plans/      # test plans (scope, approach, risks, entry/exit criteria)
.claude/skills/  # Claude Code skills for recurring tasks (generating test cases, plans, etc.)
```

The Playwright test directory and toolchain (`package.json`, Playwright config) do not exist yet. When they are added, document here the commands to run the full suite and a single test, and where the tests live.

## Playwright

- Locate elements by the `data-testid` values from `04-frontend-spec.md` (`getByTestId`), not by CSS or text.
- The app is started from `../skypermit` with `npm run dev` (`localhost:3000`) and needs a `.env.local` with Supabase keys. Start the server only when the user asks, and never in a way that changes files in `../skypermit`.

## Repository

- Remote: `git@github.com:SotnikTaras/skypermit-qa-kit-taras-sotnyk.git`
- Default branch: `main`
