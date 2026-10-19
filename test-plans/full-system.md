# Test Plan: SkyPermit Full System

**Source:** `../skypermit/docs/` (01–08; 05 and 07 take precedence) · **Date:** 2026-10-19

## Scope

**In scope**
- Auth and isolation: register at `L1`, login/logout, `401` without session, `403` on another operator's resource, reviewer access via `X-Reviewer-Id` (03 §Auth)
- Registry: pilots and aircraft, derived currency and weight band, `mtom_grams` > 30,000 g refused (01; 06 §3)
- Gate pipeline: order `PILOT → AIRCRAFT → ZONE → TIME_WINDOW`, first failure only, `422` body with exact `rule_reference` (03 §Error semantics; 06 §1, §6)
- `validate` = `submit` result without state change (03 endpoint 13)
- State machine: every legal transition, `409` for every illegal pair, 2 h cancellation deadline (02; 06 §5)
- Reviewer: approve/reject, required note, `ReviewAction` and `AuditLog` rows (02; 03 endpoints 15–16)
- Frontend: operator portal, reviewer queue, `data-testid` contract, no picker pre-filtering (04; detail in `test-plans/frontend.md`)

**Out of scope**
- `Operator.status = SUSPENDED` behaviour: undefined (open question 4)
- `ZC-2026.1 §4` as authoritative citation for inactive zone: unconfirmed (open question 5); test the rejection only
- `APPROVED → COMPLETED`: no `complete` endpoint in the API contract (03)
- Zone create/update: reference data, no API (03 §Reference data)
- `CR-1` and `OH-1` v1.4 values: not in force (06)

## Risks

| # | Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- | --- |
| R1 | Illegal flight approved: gate boundary or curfew error (midnight crossing, UTC↔Kyiv/DST, `A-7` override for KBP/ODS) | H | H | API tests at each threshold ±1 and per curfew zone in summer and winter offsets |
| R2 | Operator reads or changes another operator's data | M | H | API `401`/`403` matrix over all operator-scoped endpoints; two-operator UI isolation tests |
| R3 | Illegal transition accepted, or concurrent approve/reject both succeed | M | H | API grid of all 7 states × submit/approve/reject/cancel; parallel-request tests |
| R4 | Docs contradict each other or the code: aircraft over 30,000 g returns `400` in tests vs `422` in 06; cancel from `SUBMITTED` legal in 02 but refused in code; 08 shows submit returning `200/201` vs `422` in 03 | H (already observed) | M | Raise with doc owner; automate each only after a decision is recorded |
| R5 | Overlap reported inconsistently: `409` scheduling conflict (03) vs `TIME_WINDOW` `422` (06 §1.4) | M | M | Clarify; manual check until decided |
| R6 | `validate` changes state or disagrees with `submit` | L | H | API: repeated validate keeps `DRAFT`, then submit gives the same gate result |
| R7 | State change without `AuditLog` / `ReviewAction` row | M | M | API tests read audit rows after every transition |

## Strategy

| Level | Tool | Focus | Existing coverage |
| --- | --- | --- | --- |
| Unit | Vitest (`../skypermit/tests`) | Gate logic, thresholds, pipeline order, curfew, state-machine guards, validators | Strong: `lib/gates/*`, `curfew`, `derived`, `state-machine`, `validation`, `zone-catalogue`. Gaps: DST switch, `ODS-CTR-5` curfew, overlap only with mocked query |
| API | Playwright (`request`) | **Automate first:** auth/isolation (R2), state grid (R3), `422` body per `rule_reference` (R1), validate vs submit (R6), audit rows (R7) | `routes/aircraft`, `routes/submit` (`409`/`404`/`400` only) |
| E2E UI | Playwright (`getByTestId`) | Operator journey register → request → validate → submit; reviewer approve/reject; rejection shown with `rule_reference` | none found |
| Manual | Exploratory | Doc contradictions (R4, R5), reviewer "all four gate results" display, session expiry, message wording | — |

## Definition of Done

- [ ] Every row of 06 §6 (except the unconfirmed zone-inactive citation) has a passing API test asserting `failing_gate` and `rule_reference`
- [ ] Every `(state, action)` pair from 02 has a passing API test: legal → new state, illegal → `409`
- [ ] `401`/`403` tests pass for every operator-scoped endpoint
- [ ] Operator and reviewer E2E journeys pass using `data-testid` selectors only
- [ ] Each R4/R5 contradiction has a recorded decision and a matching test, or is listed as a known issue
- [ ] No open P1/P2 defects
