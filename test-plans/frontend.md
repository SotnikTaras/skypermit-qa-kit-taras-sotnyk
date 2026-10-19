# Test Plan: Frontend (Operator Portal & Reviewer Queue)

**Source:** `../skypermit/docs/04-frontend-spec.md` (with `03-api-contract.md` auth section, `05-open-questions.md`, `07-clarifications.md`) · **Date:** 2026-10-19

## Scope

**In scope**
- Operator auth: register (combined credentials + profile, starts at `L1`, auto-login), login, logout, session-driven header elements (04 §Operator portal; 03 §Auth)
- Pilot and aircraft registry pages: create forms and lists showing only the operator's own records
- Request creation form: zone picker from `GET /zones`, pilot/aircraft pickers scoped to the operator, window, altitude, purpose
- "Check before submitting": per-gate result, no state change (04; 03 endpoint 13)
- "My requests" list with state filter; detail view with rejection reason and `rule_reference`
- Reviewer queue: `X-Reviewer-Id` entry, `UNDER_REVIEW`-only table sorted by submission time, detail with gate results, approve and reject (note required)
- Role switcher between operator and reviewer
- Contract rules: every interactive element has a stable `data-testid`, `operator-id-input` is gone; pickers do **not** pre-filter by business rules (04 §Two contracts)

**Out of scope**
- Gate logic and thresholds: covered by unit tests and the business-rules plan
- Visual design and polish: the spec calls it intentionally minimal
- UI for `Operator.status = SUSPENDED`: response undefined (open question 4)
- Raising `authorisation_level`: out-of-band admin action, no UI

## Risks

| # | Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- | --- |
| R1 | Operator sees or picks another operator's pilots, aircraft or requests (lists, pickers, detail by URL) | M | H | E2E with two operators: lists and pickers contain only own records, foreign detail URL shows an error |
| R2 | "Check before submitting" changes state or shows a result different from what `submit` then returns | M | H | E2E: run validate repeatedly, assert state stays `DRAFT`, then submit and compare the gate outcome |
| R3 | `data-testid` drift: element renamed, missing, or `operator-id-input` reintroduced; automation breaks silently | M | H | Inventory test asserting every interactive element in the spec has its testid; review renames in PRs |
| R4 | Pickers get client-side eligibility filtering, hiding gate failures from operators | L | M | E2E: select a combination that cannot pass (e.g. `R_BASIC` pilot + `RESTRICTED` zone), assert it can be chosen and validate shows the failure |
| R5 | Reviewer detail must show "all four gate results", but the pipeline stops at the first failure: what to show after a failure is unclear | M | M | Clarify with doc owner; manual check until then, no automated expected result for gates after the first failure |
| R6 | Reject submitted without a note, or reviewer actions work with no `X-Reviewer-Id` entered | M | M | E2E: empty note keeps request `UNDER_REVIEW` with an error; no reviewer ID shows `reviewer-queue-no-reviewer` |
| R7 | Rejection reason or `rule_reference` missing or wrong on the operator detail view | M | M | E2E: one rejection per gate, assert detail shows the matching `rule_reference` |

## Strategy

| Level | Tool | Focus | Existing coverage |
| --- | --- | --- | --- |
| Unit | Vitest (`../skypermit/tests`) | Form input validation helpers | `lib/validation.test.ts` covers UUID, date, positive-integer helpers. No component tests |
| API | Playwright (`request`) | Seed data for UI tests (register, pilots, aircraft, requests in a given state); `401`/`403` behind the pages | `routes/aircraft`, `routes/submit` only; no auth or list scoping tests |
| E2E UI | Playwright (`getByTestId`) | Register → create pilot/aircraft → create request → validate → submit; reviewer approve/reject; R1–R4, R6, R7; testid inventory | none found |
| Manual | Exploratory | Reviewer gate-result display (R5), role switching mid-flow, session expiry and logout, error message clarity | — |

## Definition of Done

- [ ] Every `data-testid` named in 04 and every interactive element on the operator and reviewer pages is covered by the inventory test, which passes
- [ ] Operator happy path and reviewer approve/reject paths pass end to end via `getByTestId` only
- [ ] Two-operator isolation tests (R1) pass for lists, pickers and detail views
- [ ] Validate tests (R2) confirm no state change across repeated clicks
- [ ] R5 has a recorded decision from the doc owner, or stays documented as manual-only
- [ ] No open P1/P2 UI defects
