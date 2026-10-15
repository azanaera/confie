Quality Engineering · Test Planning

# API Test Plan

REST API — high-level plan, scope and approach

- **Target system:** Quote creation endpoint — `POST /trexis/comparator/quote` (`operationId: compositeQuoteAPI`)
- **Version:** 0.4 — single-endpoint focus, contract reconciliation merged in
- **Date:** 15 October 2026
- **Owner:** QA / Test Engineering
- **Environment:** UAT (`uat.trexis.com`) — label previously recorded as "QA", see §4
- **Auth:** OAuth (client credentials, via Postman collection)
- **Tooling:** Postman
- **Supersedes:** v0.3 (2 September 2026) and the standalone addendum `API Test Plan - Contract Reconciliation.md`

> **Note — contract reconciliation merged in (v0.4):** v0.3 assumed the response contract was *unknown*. It is not — it is *contradicted*. The OpenAPI spec (`ComparatorSchema.json`) and documented format (`ComparatorResponseFormat.json`) disagree with the captured responses on several fields. The divergences (§12), the observed field inventory (§13) and the decision log (§14) are now part of this document; the addendum is no longer maintained separately.

Draft — pending answers to the decision log in §14.

## What changed in v0.4

| v0.3 section | Was | Now |
| --- | --- | --- |
| §2 Scope | "The single POST create-quote endpoint" | Names method + exact path + base URL: `POST /trexis/comparator/quote` |
| §3 Test Levels | Schema / contract = **Planned** ("an agreed contract" not in place) | Moved to **Now** — an OpenAPI contract exists; the real constraint is that it is *wrong*, not absent |
| §4 Environments | Environment = **QA** | Reconciled with the captured `uat.trexis.com` URLs |
| §5 Test Data | Expected values "not yet defined" | Points at the captures as the de-facto truth set, with the caveats in §13 |
| §6 Exit Criteria | "Contract tests pass with zero breaking changes" | Reworded — unachievable today until the spec is reconciled (§12) |
| §11 Deliverables | "versioned alongside the API codebase" | Names the suite's actual home (no API codebase/git repo exists here) |
| §12 Open Items | A single blocker bullet | Replaced by the decision log in §14 |
| New sections | — | §12 Divergences, §13 Field Inventory, §14 Decision Log, §15 New Test Cases |
| Contents | Anchors (`#purpose`, `#scope`…) did not match `## 1Purpose`-style headings | Headings numbered and anchors fixed |
| Header | Claimed a companion addendum file | Addendum folded in; this document is now self-contained |

## Contents

- [1. Purpose & Objectives](#1-purpose--objectives)
- [2. Scope](#2-scope)
- [3. Test Levels & Types](#3-test-levels--types)
- [4. Environments & Tooling](#4-environments--tooling)
- [5. Test Data Strategy](#5-test-data-strategy)
- [6. Entry & Exit Criteria](#6-entry--exit-criteria)
- [7. Roles & Responsibilities](#7-roles--responsibilities)
- [8. Defect Management](#8-defect-management)
- [9. Risks & Mitigations](#9-risks--mitigations)
- [10. Schedule & Milestones](#10-schedule--milestones)
- [11. Deliverables](#11-deliverables)
- [12. Contract Divergences](#12-contract-divergences)
- [13. Field Inventory (Observed Truth)](#13-field-inventory-observed-truth)
- [14. Open Questions / Decision Log](#14-open-questions--decision-log)
- [15. New Test Cases Implied by the Reconciliation](#15-new-test-cases-implied-by-the-reconciliation)

## 1. Purpose & Objectives

This plan defines the approach for verifying that the API behaves correctly, securely, and reliably before and after release — covering what gets tested, at what level, with what tooling, and who signs off.

- Confirm each endpoint meets its functional contract: correct status codes, response shapes, and business logic for valid and invalid input.
- Catch breaking changes to request/response contracts before they reach consumers.
- Verify authentication, authorization, and data-access boundaries hold under both expected and adversarial use.
- Surface performance and reliability issues (latency, error rate under load) ahead of production.

## 2. Scope

### In scope (this phase)

- The single create-quote endpoint: **`POST /trexis/comparator/quote`** (`operationId: compositeQuoteAPI`), against the base URL for the target environment (see §4)
- Smoke check: endpoint reachable, authenticates, returns a response
- Functional testing against the correct/expected response, once the divergences in §12 are settled
- Negative & edge-case handling (bad input, missing fields, wrong types)
- OAuth client-credentials flow, including token refresh (see §4)

### Out of scope (this phase)

- Any endpoint other than the single create-quote POST
- Automated contract/regression tooling in CI (deferred — see §3)

## 3. Test Levels & Types

Coverage is layered so cheap, fast checks run constantly and expensive ones run where they matter most.

| Level | Description | Tooling | Status |
| --- | --- | --- | --- |
| Smoke | Create-quote endpoint reachable, authenticates via OAuth, returns a response (allowing for its few-minute response time) | Postman | Now |
| Functional / endpoint | Create-quote request/response works end-to-end; asserting against the *correct* values once the divergences in §12 are confirmed | Postman | Now |
| Negative & edge case | Malformed quote payloads, boundary values, missing/extra fields, wrong content-types | Postman | Now |
| Schema / contract | Responses validated against an OpenAPI spec; consumer contracts checked for breaking changes | Pact, Dredd, JSON Schema | **Now** — an OpenAPI contract exists (`ComparatorSchema.json`); it is contradicted by the live responses (§12) and must be reconciled before assertions can pass. Not yet in a pipeline |
| Regression | Full automated suite re-run before each release to catch unintended breakage | CI pipeline | Planned — not yet in a pipeline |

**Now** = active this phase, run manually in Postman. The schema/contract level moved from Planned to Now because a contract now exists; the blocker is that the contract is wrong, not that it is missing.

## 4. Environments & Tooling

A single environment is provisioned for this phase, tested manually through a Postman collection that already includes the create-quote request and OAuth credentials.

> **Label reconciliation (D4):** v0.3 called this environment **QA**, but every captured response URL is `uat.trexis.com`. Until Q9 is answered, treat the target as **UAT** and align the label with the URLs.

### OAuth in Postman

- Client ID, client secret, and the account/id are provided in the collection — import once, credentials are ready to use.
- Tokens expire every 2 hours; the token must be refreshed manually or via a pre-request script before it lapses mid-session.
- **Auth-flow discrepancy (D3):** the OpenAPI spec declares an OAuth **`authorizationCode`** flow (Okta `tokenUrl`), while this plan and the provisioned Postman collection use **client credentials**. One is wrong — see Q3.

### Environment & tooling map

| Stage | Purpose | Test types run | Client credentials | Notes |
| --- | --- | --- | --- | --- |
| DEV | Developer validation | — | **Yes** | Credentials already provisioned |
| QA | Manual + Postman-driven validation | Smoke, functional, negative | **Yes** | Credentials already provisioned |
| UAT | Client-facing validation, incl. quote-link → PORTAL visibility (§4.1) | Smoke, functional, negative | TBC | Captured URLs are `uat.trexis.com`; label vs URLs unresolved (D4, Q9) |
| PERF / SIT | Isolated load testing | — | **Not yet — requested** | Not yet provisioned; client-credentials must be requested (see §4.1) |

### 4.1 Outstanding provisioning & dependencies

- **PERF / SIT client credentials.** Request client credentials for the **PERF / SIT** environment — today credentials exist only for **QA** and **DEV**. Load testing can't be scheduled until they're provisioned.

  | Field | Value |
  | --- | --- |
  | Environment needing credentials | PERF / SIT |
  | Environments that have them today | QA, DEV |
  | Status | Not yet requested / not provisioned |
  | Owner | Dev + Client |

- **Producer code per state (Confie).** A producer-code mapping for Confie is required **per state**. The actual codes are **to be determined by Trexis**. This feeds the `producerCode.id` field in `policyInfo` (see §13, TC-16) and gates quote requests that depend on a state-specific producer code.

  | Field | Value |
  | --- | --- |
  | Scope | Confie, per state |
  | Values | **To be determined by Trexis** |
  | Affects | `policyInfo.producerCode.id` (TC-16) |
  | Owner | Trexis / Client |

- **Quote-link → PORTAL visibility (UAT).** **To be determined:** after a successful API response, Trexis wants to see the quote in **PORTAL** by following the returned quote link (`quoteInfo.summaryURL` / `bridgeURL`) in the **UAT** environment. The exact mechanism and acceptance criteria are not yet defined; a meeting with **Anetra** is being arranged.

  | Field | Value |
  | --- | --- |
  | Requirement | Quote visible in PORTAL via the returned quote link |
  | Environment | UAT |
  | Status | **To be determined** — meeting with Anetra being arranged |
  | Acceptance criteria | Not yet defined |
  | Owner | You + Trexis |

## 5. Test Data Strategy

- **Request payloads:** reuse the sample quote request already provided and working in Postman; vary fields deliberately for negative/edge cases.
- **Expected values:** the de-facto truth set is the three captured response bodies on disk — `itc response body.json`, `itc 2 response body.json` (both `parentQuote: "Confie"`) and `PLRater response body.json` (`parentQuote: "PLRating"`). These supersede the OpenAPI spec where the two disagree (§12), subject to the caveats and open questions in §13–§14.
- **Response latency:** the endpoint can take a few minutes to respond — set Postman request timeouts accordingly so slow-but-valid responses aren't mistaken for failures.
- **Determinism:** the two Confie captures differ for the same-looking insured/vehicle/state. Confirm whether that is different inputs or rating variance — this decides whether idempotency testing (TC-30) is meaningful (D6, Q8).
- **Sensitive data:** no production data in UAT/QA; use anonymized/synthetic quote details only.

## 6. Entry & Exit Criteria

### Entry criteria

- The authoritative response shape is agreed — i.e. Q2 and Q3 in §14 are answered *(not yet met — current blocker)*
- Target environment (UAT/QA) is deployed and reachable
- OAuth credentials imported into Postman, with a refresh step in place before the 2-hour expiry
- Build under test is tagged and stable

### Exit criteria

- 100% of critical-path endpoints pass functional tests
- No open critical or high-severity defects
- Any observed response deviation from the contract is either fixed in the spec or explicitly approved as an accepted change — *note: "contract tests pass with zero breaking changes" is unachievable as written until the spec is reconciled (§12)*
- Performance results within agreed thresholds

## 7. Roles & Responsibilities

| Role | Responsibility |
| --- | --- |
| QA / Test Engineer | Owns test plan, writes and maintains automated suites, triages failures |
| API / Backend Developer | Fixes defects, maintains OpenAPI spec, supports test data needs |
| Client | Confirms correct/expected response values; owns the answers tagged **Client** in §14 |

## 8. Defect Management

Defects are logged with request/response evidence (method, endpoint, payload, actual vs. expected) and triaged by severity:

| Severity | Definition | Response |
| --- | --- | --- |
| Critical | Data loss, security breach, endpoint fully unavailable | Blocks release, fix immediately |
| High | Core flow broken, no workaround | Blocks release |
| Medium | Incorrect behavior with a workaround | Fix before next release |
| Low | Cosmetic or minor inconsistency | Backlog |

## 9. Risks & Mitigations

| Risk | Mitigation |
| --- | --- |
| Response contract is contradicted by the spec, blocking schema-level assertions | Reconcile via the decision log (§14) before relying on the spec; treat captures as truth until then |
| Correct/expected quote response not yet confirmed | Drive the decision log to closure; meanwhile limit functional checks to status code and response structure, not exact values |
| OAuth token expires every 2 hours, mid-test-session | Refresh the token before each test session; consider a Postman pre-request script to auto-refresh |
| Endpoint response can take a few minutes | Set generous Postman timeouts; don't treat slow responses as failures |
| Single environment means no isolation from other testing/dev activity | Coordinate usage windows with the team; flag data collisions early |
| QA vs UAT label ambiguity misdirects test runs | Resolve Q9 before the first formal run |

## 10. Schedule & Milestones

Kept intentionally short while scope stays at one endpoint:

1. Resolve the decision log (§14) — at minimum Q2, Q3, Q9
2. Client confirms correct/expected quote response
3. Functional test cases finalized in Postman against confirmed values
4. Smoke, functional, and negative suite run in the target environment
5. Sign-off on phase 1; phase 2 (§3 Planned / deferred items) scoped separately when it's picked up

## 11. Deliverables

- Test suite (Postman collection + any scripts), versioned **in this repository** — there is no separate API codebase or git repo; this plan and its artifacts are the home
- Test coverage report mapped to endpoints and scenarios
- Defect log with severity breakdown
- This test plan, kept current as scope firms up
- The decision log (§14), tracked to closure with owners and status

## 12. Contract Divergences

Three artifact classes are on disk and they disagree. The plan assumed the response contract was unknown; it is not — it is *contradicted*.

**Artifacts referenced:**

- `ComparatorSchema.json` — OpenAPI 3.0 spec (`POST /trexis/comparator/quote`, `operationId: compositeQuoteAPI`)
- `ComparatorResponseFormat.json` — documented response shape
- Captures: `itc response body.json`, `itc 2 response body.json` (both `parentQuote: "Confie"`), `PLRater response body.json` (`parentQuote: "PLRating"`)

| Aspect | `ComparatorSchema.json` | `ComparatorResponseFormat.json` | Captures (`itc`, `itc 2`, `PLRater`) |
| --- | --- | --- | --- |
| `payment` | **object** | **object** | **array** of plans |
| `options[]` | absent | absent | present |
| `msgStatus` | absent | absent | present (`"quoted"`) |
| top-level `premium` | absent | absent | present in `itc 2` only |
| `quoteInfo.quoteID` | absent | absent | present |
| `quoteInfo.parentQuote` / `.status` | absent | absent | present |
| `quoteInfo.quotePreparedDT` | present | present | absent |
| `errorresponse` | absent | absent | present (`{}`) |
| `producerInfo`, `location` | present | present | absent |

### Divergences to reconcile (these are the requirements)

- **D1 — `payment` type.** Spec/format declare `payment` as an object (`PaymentDTO`); the API returns an **array**. Every schema-level assertion fails against `ComparatorSchema.json` as written.
- **D2 — Missing fields.** `msgStatus`, `options`, `quoteID`, `parentQuote`, `errorresponse` exist in reality but not in the spec — yet TC-02 and TC-03 assert on two of them. The test cases were written against the captures, not the spec.
- **D3 — Auth flow.** Spec = OAuth **`authorizationCode`** (Okta `tokenUrl`); plan = **client-credentials**. One is wrong.
- **D4 — Environment.** Plan says **QA**; captured URLs are `uat.trexis.com`.
- **D5 — Value relationships.** `payment[].planPremium`, `options[].totalPremium`, `fee`, and top-level `premium` do not reconcile under any single rule. See §14 Q5–Q7.
- **D6 — Determinism.** The two Confie captures differ for the same-looking insured/vehicle/state. Different inputs, or rating variance? This decides whether idempotency (TC-30) is meaningful.
- **D7 — Conditional fields.** Top-level `premium` appears in one capture and not another; `coverage` is non-empty in one and `[]` in another. Required vs optional is undefined.

## 13. Field Inventory (Observed Truth)

Types are as observed in the captures. "Schema?" = whether the OpenAPI spec also declares it.

### Top level

| Field | Type (observed) | Schema? | Example | Notes |
| --- | --- | --- | --- | --- |
| `msgStatus` | string (enum?) | no | `"quoted"` | enum values unknown — only `quoted` seen (Q1) |
| `policyInfo` | object | yes (differs) | — | see below |
| `payment` | **array** | yes (as object) | 13–16 plans | D1 |
| `options` | array | **no** | 13–16 options | D2 |
| `premium` | string | no | `"4538.00 usd"` | composition unknown (Q7); present in `itc 2` only |
| `coverage` | array | yes | `[]` or `[{name,limit}]` | sometimes empty |
| `drivers` | array | yes | — | |
| `vehicles` | array | yes | — | |
| `quoteInfo` | object | yes (differs) | — | see below |
| `errorresponse` | object | no | `{}` | failure shape unknown (Q4) |
| `producerInfo` / `location` | — | yes | — | **not seen** in any capture |

### `policyInfo`

| Field | Type | Example | Notes |
| --- | --- | --- | --- |
| `insured` | object | `{firstName, lastName, primaryAddress, emailAddress1}` | `validUntil` returns the **string** `"null"` (TC-18) |
| `producerCode.id` | string | `"139999"` | capture-only; not in spec's `PolicyInfoDTO` (TC-16) |
| `baseState` | string | `"IN"` | |
| `jobEffectiveDate` | string (ISO) | `"2026-07-25T05:01:00Z"` | |
| `jobExpirationDate` | string (ISO) | `"2027-07-25T05:01:00Z"` | |
| `termType` | string | `"Annual"` | enum? |
| `policyType` | string | `"PersonalAuto"` | enum? |
| `effectiveDt` | string (ISO) | `"2026-07-25T05:01:00Z"` | TC-14 expects day delta 0 or −1 |
| `expirationDt` | string (ISO) | `"2027-07-25T05:01:00Z"` | TC-15 |
| `product` | string | `"Personal Auto"` | |
| `uwCompany` | string | `"Trexis Insurance Corp"` | |

### `payment[]` (one entry per payment plan)

| Field | Type | Example | Notes |
| --- | --- | --- | --- |
| `paymentPlanCd` | string | `"1-Month Down - EFT"` | joined key: plan + method (EFT / RCC / Direct Bill) |
| `planPremium` | string | `"844.00 usd"` | number **and** currency in one string (TC-19) |
| `additionalQuotedScenario[]` | array | `[{cost:[{actualAmountBilling, chargePattern, displayName}], coverable}]` | `planPremium` ≈ Σ `actualAmountBilling` — see Q6 |

### `options[]` (one entry per payment option)

| Field | Type | Example | Notes |
| --- | --- | --- | --- |
| `paymentPlanName` | string | `"1-Month Down"` | plan name **without** method suffix |
| `paymentType` | string | `"PIF"` / `"EFT"` / `"RCC"` / `"Direct"` | enum |
| `totalPremium` | number | `1010.00` | differs from `planPremium` (Q5) |
| `installment` | number | `88.81` | |
| `noOfInstallments` | number | `10` | |
| `fee` | number | `6.00` | |
| `downPayment` | number | `118.81` | |
| `firstInstallmentDate` | string | `"2026-10-05"` | absent for PIF |
| `installmentData[]` | array | `{statementDate, dueDate, amount (string), currency}` | TC-08 sums `amount` vs `totalPremium` |

### `quoteInfo`

| Field | Type | Example | Notes |
| --- | --- | --- | --- |
| `summaryURL` | string | `…?UrlType=summary&QuoteID=<uuid>` | must contain `quoteID` (TC-07) |
| `bridgeURL` | string | `…?UrlType=bridge&QuoteID=<uuid>` | must contain `quoteID` (TC-07) |
| `quoteID` | string (UUID) | `"b4e159ce-ef63-4366-a492-1c2e81e6d199"` | capture-only |
| `parentQuote` | string | `"Confie"` / `"PLRating"` | identifies the upstream rater |
| `status` | string | `"quoted"` | duplicates `msgStatus`? |
| `quotePreparedDT` | string | — | **spec-only**, absent in captures |

## 14. Open Questions / Decision Log

Replaces v0.3's single §12 blocker. Each row is a decision to obtain, with its evidence and a proposed default.

| ID | Question | Evidence | Proposed default | Owner | Status |
| --- | --- | --- | --- | --- | --- |
| Q1 | Full enum for `msgStatus` (and `quoteInfo.status`)? | only `"quoted"` observed | Enumerate all values incl. failure states | Client | Open |
| Q2 | Which shape is authoritative — the spec or the captures? | spec `payment` object vs captures `payment` array | Captures are truth; spec must be regenerated | Client + Dev | Open |
| Q3 | Auth flow: `authorizationCode` (spec) or client-credentials (plan)? | D3 | Client-credentials (matches the provisioned Postman collection) | Dev | Open |
| Q4 | Shape of `errorresponse` on failure (4xx/5xx)? | only `{}` on success; TC-21/22/25 expect a populated one | Provide a failing example | Client | Open |
| Q5 | Relationship of `options[].totalPremium` to `payment[].planPremium`? | PIF: 894 vs 844, and 1010 vs 960 — both +50, while `fee` = 30 | Define the formula explicitly | Client | Open |
| Q6 | Is `planPremium` = Σ `additionalQuotedScenario[].cost[].actualAmountBilling`? | PIF 844 = 275+569; PIF 960 = 275+569+56+60 | Confirm, then add a financial-integrity test | Client | Open |
| Q7 | What composes top-level `premium` (`"4538.00 usd"`)? | present in `itc 2` only; matches neither plan nor option totals | Define, or mark non-testable | Client | Open |
| Q8 | Is rating deterministic for identical input? | two Confie captures differ | State answer; keep/drop TC-30 (idempotency) | Dev | Open |
| Q9 | Which environment is authoritative — QA or UAT? | plan says QA; URLs say `uat.trexis.com` | Align label and URLs | You | Open |
| Q10 | Are `producerInfo` and `location` ever returned? | in spec, absent in all captures | Confirm optional / remove from contract | Dev | Open |
| Q11 | Can client credentials be provisioned for **PERF / SIT**? | credentials exist only in QA and DEV (§4.1) | Request provisioning before load testing | Dev + Client | Open |
| Q12 | Producer code per state for Confie — what are the codes? | only `producerCode.id: "139999"` observed (§13, TC-16) | Trexis to supply the per-state mapping | Trexis / Client | Open |
| Q13 | How is a quote surfaced in **PORTAL** from the returned quote link, in UAT? | Trexis requirement; no mechanism defined (§4.1) | Define mechanism + acceptance criteria; meeting with Anetra being arranged | You + Trexis | Open |

**Resolved so far:** auth model (OAuth, credentials in the Postman collection), tooling (Postman), and endpoint scope (single POST create-quote). Q1–Q13 remain open.

## 15. New Test Cases Implied by the Reconciliation

- **TC-31 — `planPremium` reconciliation:** for every plan, `planPremium` (parsed to number) equals the sum of `additionalQuotedScenario[].cost[].actualAmountBilling`.
- **TC-32 — shape guard:** `payment` is an array and `options` is a non-empty array (fails loudly if the spec-shaped object reappears).
- **TC-33 — conditional fields:** assert `premium` presence matches whichever rule Q7 settles.
