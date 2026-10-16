Quality Engineering · Test Planning

# API Test Plan

REST API — high-level plan, scope and approach

> **Superseded by [`API Test Plan v1.0.md`](./API%20Test%20Plan%20v1.0.md) (16 October 2026).** Retained for history.

- **Target system:** Quote creation endpoint — `POST /trexis/comparator/quote` (`operationId: compositeQuoteAPI`)
- **Version:** 0.5 — single-endpoint focus; contract reconciled, decisions closed
- **Date:** 15 October 2026
- **Owner:** QA / Test Engineering
- **Environment:** UAT (`uat.trexis.com`)
- **Auth:** OAuth (client-credentials, via the Postman collection)
- **Tooling:** Postman
- **Supersedes:** v0.4 (15 October 2026), v0.3 (2 September 2026), and the retired addendum `API Test Plan - Contract Reconciliation.md`
- **Related:** [`GLOSSARY.md`](./GLOSSARY.md), [`docs/adr/0001-captured-responses-are-the-source-of-truth.md`](./docs/adr/0001-captured-responses-are-the-source-of-truth.md), [`docs/test-cases.md`](./docs/test-cases.md)

> **v0.5 — contract reconciled, decisions closed.** The response contract is not unknown; it is *contradicted* by `ComparatorSchema.json`. We have adopted the captured responses as authoritative and the spec as downstream (ADR-0001). Several previously open questions are now closed by evidence or decision; the rest are routed to the client and Dev questionnaires (§16). Performance/load testing is explicitly **out of scope** for this phase.

Draft — pending the answers tracked in §16.

## What changed in v0.5

| Area | v0.4 | v0.5 |
| --- | --- | --- |
| Source of truth | Stated in prose | Recorded as ADR-0001: captures authoritative, spec regenerated |
| Environment (Q9) | "Reconcile QA vs UAT" | **UAT** is authoritative; relabelled |
| Auth (Q3) | "One is wrong — see Q3" | **Client-credentials**; spec's `authorizationCode` (Guidewire Okta preview) marked stale |
| `planPremium` (Q6) | "needs confirmation" | **Resolved by evidence**: equals Σ `additionalQuotedScenario[].cost[].actualAmountBilling` |
| Counts | "13–16 plans/options" | Corrected: **16 `payment[]` and 16 `options[]`** in every capture |
| New sections | — | §12 Source Artifacts, §15 Traceability Matrix, request-side inventory in §14 |
| Performance | Thresholds "agreed" (undefined) | **Out of scope** this phase; PERF/SIT awaits credentials |
| Addendum file | Companion, still present | Deleted; this document is self-contained |
| Domain language | — | `GLOSSARY.md`, ADR-0001, and `docs/test-cases.md` added |

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
- [12. Source Artifacts](#12-source-artifacts)
- [13. Contract Divergences](#13-contract-divergences)
- [14. Field Inventory (Observed Truth)](#14-field-inventory-observed-truth)
- [15. Traceability Matrix](#15-traceability-matrix)
- [16. Open Questions / Decision Log](#16-open-questions--decision-log)
- [17. New Test Cases Implied by the Reconciliation](#17-new-test-cases-implied-by-the-reconciliation)

## 1. Purpose & Objectives

This plan defines the approach for verifying that the API behaves correctly, securely, and reliably before and after release — covering what gets tested, at what level, with what tooling, and who signs off.

- Confirm each endpoint meets its functional contract: correct status codes, response shapes, and business logic for valid and invalid input.
- Catch breaking changes to request/response contracts before they reach consumers.
- Verify authentication, authorization, and data-access boundaries hold under both expected and adversarial use.
- Surface defects in the returned financial data (premium reconciliation) before they reach production.

## 2. Scope

### In scope (this phase)

- The single create-quote endpoint: **`POST /trexis/comparator/quote`** (`operationId: compositeQuoteAPI`), against the UAT base URL (see §4)
- Smoke check: endpoint reachable, authenticates, returns a response
- Functional testing against the **captured responses as the de-facto contract** (§14), pending the client decisions in §16
- Negative & edge-case handling (bad input, missing fields, wrong types)
- OAuth client-credentials flow, including token refresh (see §4)

### Out of scope (this phase)

- **Performance / load testing** — deferred; PERF/SIT is not provisioned and has no credentials (§4.1)
- Any endpoint other than the single create-quote POST
- Automated contract/regression tooling in CI (deferred — see §3)

## 3. Test Levels & Types

Coverage is layered so cheap, fast checks run constantly and expensive ones run where they matter most.

| Level | Description | Tooling | Status |
| --- | --- | --- | --- |
| Smoke | Endpoint reachable, authenticates via OAuth, returns a response (allowing for its few-minute response time) | Postman | Now |
| Functional / endpoint | Request/response works end-to-end; asserting against the captured truth set (§14) | Postman | Now |
| Negative & edge case | Malformed payloads, boundary values, missing/extra fields, wrong content-types | Postman | Now |
| Schema / contract | Responses validated against an OpenAPI spec; consumer contracts checked for breaking changes | Pact, Dredd, JSON Schema | **Now** — the spec (`ComparatorSchema.json`) is contradicted by the live responses (§13) and must be regenerated before assertions pass (ADR-0001). Not yet in a pipeline |
| Regression | Full automated suite re-run before each release | CI pipeline | Planned — not yet in a pipeline |
| Performance / load | Latency and error rate under load | — | **Out of scope** this phase (PERF/SIT unprovisioned) |

**Now** = active this phase, run manually in Postman. The schema/contract level moved from Planned to Now because a contract now exists; the blocker is that the contract is wrong, not that it is missing.

## 4. Environments & Tooling

Tested manually through a Postman collection that already includes the create-quote request and OAuth credentials.

> **Environment (Q9 — resolved):** the authoritative environment is **UAT** (`uat.trexis.com`), matching every captured response URL. v0.3's "QA" label was stale and has been dropped.

### OAuth in Postman

- Client ID, client secret, and the account/id are provided in the collection — import once, credentials are ready to use.
- Tokens expire every 2 hours; refresh manually or via a pre-request script before it lapses mid-session.
- **Auth flow (Q3 — resolved):** testing uses OAuth **client-credentials**, matching the provisioned collection. The spec's `authorizationCode` flow (`guidewire-hub.oktapreview.com`) is treated as stale.

### Environment & tooling map

| Stage | Purpose | Test types run | Client credentials | Notes |
| --- | --- | --- | --- | --- |
| DEV | Developer validation | — | **Yes** | Credentials already provisioned |
| QA | Manual + Postman-driven validation | Smoke, functional, negative | **Yes** | Credentials already provisioned |
| UAT | **Authoritative** — client-facing validation, incl. quote-link → PORTAL visibility | Smoke, functional, negative | Yes | Captured URLs are `uat.trexis.com` |
| PERF / SIT | Isolated load testing | — | **Not yet — requested** | Not provisioned; client-credentials requested (§4.1) |

### 4.1 Outstanding provisioning & dependencies

- **PERF / SIT client credentials.** Request client credentials for **PERF / SIT** — today credentials exist only for **QA** and **DEV**. Load testing can't be scheduled until they're provisioned. _(Open item Q11.)_

  | Field | Value |
  | --- | --- |
  | Environment needing credentials | PERF / SIT |
  | Environments that have them today | QA, DEV |
  | Status | Not yet requested / not provisioned |
  | Owner | Dev + Client |

- **Producer code per state (Confie).** A producer-code mapping for Confie is required **per state**; the values are **to be determined by Trexis**. Feeds `policyInfo.producerCode.id` (§14, TC-16). _(Open item Q12.)_

  | Field | Value |
  | --- | --- |
  | Scope | Confie, per state |
  | Values | **To be determined by Trexis** |
  | Affects | `policyInfo.producerCode.id` (TC-16) |
  | Owner | Trexis / Client |

- **Quote-link → PORTAL visibility (UAT).** After a successful response, Trexis wants to see the quote in **PORTAL** by following the returned quote link (`quoteInfo.summaryURL` / `bridgeURL`) in UAT. The mechanism and acceptance criteria are undefined; a meeting with **Anetra** is being arranged. _(Open item Q13.)_

  | Field | Value |
  | --- | --- |
  | Requirement | Quote visible in PORTAL via the returned quote link |
  | Environment | UAT |
  | Status | **To be determined** — meeting with Anetra being arranged |
  | Acceptance criteria | Not yet defined |
  | Owner | QA + Trexis |

## 5. Test Data Strategy

- **Request payloads:** reuse the sample quote request already working in Postman; vary fields deliberately for negative/edge cases. The request schema is inventoried in §14.
- **Expected values:** the de-facto truth set is the three captures — `itc response body.json`, `itc 2 response body.json` (both `parentQuote: "Confie"`) and `PLRater response body.json` (`parentQuote: "PLRating"`). They supersede the OpenAPI spec where the two disagree (§13, ADR-0001). Each returns **16 `payment[]` and 16 `options[]` entries**.
- **Response latency:** the endpoint can take a few minutes — set Postman timeouts accordingly so slow-but-valid responses aren't mistaken for failures.
- **Determinism:** the two Confie captures differ for the same-looking insured/vehicle/state. Whether that reflects different inputs or rating variance is unresolved (Q8); until then, treat every capture as input-specific and hold TC-30 (idempotency) provisionally.
- **Sensitive data:** no production data; use anonymized/synthetic quote details only.

## 6. Entry & Exit Criteria

### Entry criteria

- The authoritative response shape is settled — **met**: captures are authoritative (ADR-0001, Q2) and the auth flow is fixed (Q3)
- Target environment (UAT) is deployed and reachable
- OAuth credentials imported into Postman, with a refresh step in place before the 2-hour expiry
- Build under test is tagged and stable

### Exit criteria

- 100% of critical-path cases pass functional tests against the captured contract
- No open critical or high-severity defects
- Any observed deviation from the captured contract is either fixed in the spec or explicitly approved as an accepted change — *"contract tests pass with zero breaking changes" is unachievable as written until `ComparatorSchema.json` is regenerated (§13)*
- Financial-integrity cases (TC-08, TC-10, TC-31) pass — currently a known failure (TC-20)
- Performance exit criteria are deferred with the performance level (§3)

## 7. Roles & Responsibilities

| Role | Responsibility |
| --- | --- |
| QA / Test Engineer | Owns the plan, writes and maintains the suite, triages failures |
| API / Backend Developer | Fixes defects, maintains the OpenAPI spec, supplies test data |
| Client (Trexis) | Confirms expected values; owns the answers routed "Client" in §16 |

## 8. Defect Management

Defects are logged with request/response evidence (method, endpoint, payload, actual vs. expected) and triaged by severity:

| Severity | Definition | Response |
| --- | --- | --- |
| Critical | Data loss, security breach, endpoint fully unavailable | Blocks release, fix immediately |
| High | Core flow broken, no workaround | Blocks release |
| Medium | Incorrect behavior with a workaround | Fix before next release |
| Low | Cosmetic or minor inconsistency | Backlog |

Known defect already captured by the case set: **TC-20** — EFT/RCC installments equal Direct's minus the fee difference.

## 9. Risks & Mitigations

| Risk | Mitigation |
| --- | --- |
| Response contract contradicted by the spec, blocking schema assertions | Captures are authoritative (ADR-0001); regenerate the spec before relying on it |
| Correct/expected values for some fields still unconfirmed | Drive §16 to closure via the questionnaires; assert on structure and the captured set meanwhile |
| OAuth token expires every 2 hours, mid-session | Refresh before each session; consider a Postman pre-request script |
| Endpoint response can take a few minutes | Set generous Postman timeouts; don't treat slow responses as failures |
| Known financial-integrity failure (TC-20) | Tracked as a defect; TC-10 fails by design until fixed, then delete TC-20 |
| Producer codes unknown per state (Q12) | Confirm with Trexis before state-specific quote requests |

## 10. Schedule & Milestones

1. Close the §16 decision log — at minimum Q5, Q8, Q12
2. Client confirms the field values routed "Client"
3. Functional cases finalized in Postman against the captured set
4. Smoke, functional, and negative suites run in UAT
5. Sign-off on phase 1; performance and CI automation (§3) scoped separately when provisioned

## 11. Deliverables

- Test suite (Postman collection + scripts), versioned **in this repository**
- Test-case catalog: [`docs/test-cases.md`](./docs/test-cases.md)
- Domain docs: [`GLOSSARY.md`](./GLOSSARY.md) and [`docs/adr/`](./docs/adr/)
- Decision log (§16) tracked to closure, with owners
- Two questionnaires: client and Dev (see §16 routing)
- Coverage report mapped to endpoints and scenarios; defect log with severity breakdown
- This test plan, kept current as scope firms up

## 12. Source Artifacts

| Artifact | What it is |
| --- | --- |
| `ComparatorSchema.json` | OpenAPI 3.0.0 spec — `POST /trexis/comparator/quote`, `operationId: compositeQuoteAPI`, server `/api`. Declares only a `200` response and **no enums** |
| `ComparatorResponseFormat.json` | Documented response shape (matches the spec, not the live API) |
| `quote_api_test_cases.xlsx` | 30 test cases across 6 sections (sheets *Test Cases*, *Summary*); transcribed to `docs/test-cases.md` |
| `itc response body.json`, `itc 2 response body.json` | Captures, `parentQuote: "Confie"` |
| `PLRater response body.json` | Capture, `parentQuote: "PLRating"` |
| `GLOSSARY.md`, `docs/adr/0001-…md` | Domain vocabulary and the source-of-truth decision |

## 13. Contract Divergences

Three artifact classes are on disk and they disagree. The response contract is not unknown; it is *contradicted*.

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

- **D1 — `payment` type.** Spec/format declare `payment` as an object (`PaymentDTO`); the API returns an **array**. Every schema-level assertion fails against the spec as written.
- **D2 — Missing fields.** `msgStatus`, `options`, `quoteID`, `parentQuote`, `errorresponse` exist in reality but not in the spec — yet TC-02/TC-03 assert on them. The cases were written against the captures, not the spec.
- **D3 — Auth flow.** Spec = OAuth `authorizationCode` (Guidewire Okta preview); this plan = client-credentials. **Resolved (Q3):** client-credentials.
- **D4 — Environment.** v0.3 said QA; captured URLs are `uat.trexis.com`. **Resolved (Q9):** UAT.
- **D5 — Value relationships.** `planPremium`, `totalPremium`, `fee`, and top-level `premium` do not reconcile under a single rule. See §16 Q5–Q7.
- **D6 — Determinism.** The two Confie captures differ for the same-looking input. Different inputs, or rating variance? Decides whether TC-30 is meaningful (Q8).
- **D7 — Conditional fields.** Top-level `premium` appears in one capture and not another; `coverage` is non-empty in one and `[]` in another. Required vs optional is undefined.

## 14. Field Inventory (Observed Truth)

Types are as observed in the captures unless marked as request-side. "Schema?" = whether the OpenAPI spec declares it.

### Response — top level

| Field | Type (observed) | Schema? | Example | Notes |
| --- | --- | --- | --- | --- |
| `msgStatus` | string (enum?) | no | `"quoted"` | only `quoted` observed (Q1) |
| `policyInfo` | object | yes (differs) | — | see below |
| `payment` | **array** (16) | yes (as object) | — | D1 |
| `options` | array (16) | **no** | — | D2 |
| `premium` | string | no | `"4538.00 usd"` | present in `itc 2` only; composition unknown (Q7) |
| `coverage` | array | yes | `[]` or `[{name, limit}]` | empty only in `itc 2` |
| `drivers` | array | yes | — | |
| `vehicles` | array | yes | — | |
| `quoteInfo` | object | yes (differs) | — | see below |
| `errorresponse` | object | no | `{}` | failure shape unknown (Q4) |
| `producerInfo` / `location` | — | yes | — | **not seen** in any capture (Q10) |

### Response — `policyInfo`

| Field | Type | Example | Notes |
| --- | --- | --- | --- |
| `insured` | object | `{firstName, lastName, primaryAddress, emailAddress1}` | `validUntil` returns the **string** `"null"` (TC-18) |
| `producerCode.id` | string | `"139999"` | capture-only; not in the spec's `PolicyInfoDTO` (TC-16) |
| `baseState` | string | `"IN"` | |
| `jobEffectiveDate` / `jobExpirationDate` | string (ISO) | `"2026-07-25T05:01:00Z"` / `"2027-07-25T05:01:00Z"` | |
| `termType` | string | `"Annual"` | enum? |
| `policyType` | string | `"PersonalAuto"` | enum? |
| `effectiveDt` | string (ISO) | `"2026-07-25T05:01:00Z"` | TC-14 expects day delta 0 or −1 |
| `expirationDt` | string (ISO) | `"2027-07-25T05:01:00Z"` | TC-15 |
| `product` | string | `"Personal Auto"` | |
| `uwCompany` | string | `"Trexis Insurance Corp"` | |

### Response — `payment[]` (16 entries; one per payment plan)

| Field | Type | Example | Notes |
| --- | --- | --- | --- |
| `paymentPlanCd` | string | `"1-Month Down - EFT"` | plan name joined with method (EFT / RCC / Direct Bill) |
| `planPremium` | string | `"844.00 usd"` | number **and** currency in one string (TC-19). Equals Σ `actualAmountBilling` (Q6 resolved) |
| `additionalQuotedScenario[]` | array | `[{cost:[{actualAmountBilling, chargePattern, displayName}], coverable}]` | 2–32 `cost[]` entries per plan depending on capture |

### Response — `options[]` (16 entries; one per payment option)

| Field | Type | Example | Notes |
| --- | --- | --- | --- |
| `paymentPlanName` | string | `"1-Month Down"` | plan name **without** method suffix |
| `paymentType` | string | `"PIF"` / `"EFT"` / `"RCC"` / `"Direct"` | enum |
| `totalPremium` | number | `1010.00` | differs from `planPremium`; delta not constant (Q5) |
| `installment` | number | `88.81` | |
| `noOfInstallments` | number | `10` | |
| `fee` | number | `6.00` | |
| `downPayment` | number | `118.81` | |
| `firstInstallmentDate` | string | `"2026-10-05"` | absent for PIF |
| `installmentData[]` | array | `{statementDate, dueDate, amount (string), currency}` | TC-08 sums `amount` vs `totalPremium` |

### Response — `quoteInfo`

| Field | Type | Example | Notes |
| --- | --- | --- | --- |
| `summaryURL` / `bridgeURL` | string | `…?UrlType=summary&QuoteID=<uuid>` | must contain `quoteID` (TC-07) |
| `quoteID` | string (UUID) | `"b4e159ce-ef63-4366-a492-1c2e81e6d199"` | capture-only |
| `parentQuote` | string | `"Confie"` / `"PLRating"` | identifies the upstream rater |
| `status` | string | `"quoted"` | duplicates `msgStatus`? |
| `quotePreparedDT` | string | — | **spec-only**, absent in captures |

### Request — `QuoteRequestDTO` (from the spec)

| Field | Type | Notes |
| --- | --- | --- |
| `policy` | `PolicyDTO` | carries `insured`, `producerCode`, `baseState`, `jobEffectiveDate`/`jobExpirationDate`, `termType`, `policyType`, `coverages[]` |
| `drivers` | `DriverDTO[]` | `contact`, `licenseNumber`, `licenseState`, `violations[]`, `modifiers[]` |
| `vehicles` | `VehicleDTO[]` | `vin`, `make`, `model`, `year` (int32), `serialNumber` (int32), `coverage[]`, `garageLocation` |

The full nested field lists are declared in `ComparatorSchema.json`; the response-vs-request consistency cases (TC-11–TC-17) assert the request values survive into the response.

## 15. Traceability Matrix

Each divergence maps to the fields it touches and the case(s) that would prove it resolved.

| Divergence | Field(s) | Proving case(s) |
| --- | --- | --- |
| D1 — `payment` is an array | `payment[]` | TC-05, TC-32 |
| D2 — spec-absent fields | `msgStatus`, `options`, `quoteID`, `parentQuote`, `errorresponse` | TC-02, TC-03, TC-06, TC-07, TC-32 |
| D3 — auth flow | OAuth scheme | TC-25 (auth negative); smoke auth |
| D4 — environment | base URL | TC-01 (reachability) |
| D5 — value relationships | `planPremium`, `totalPremium`, `fee`, `premium` | TC-08, TC-10, TC-31 |
| D6 — determinism | `quoteID`, premium | TC-30 |
| D7 — conditional fields | `premium`, `coverage` | TC-33 |

## 16. Open Questions / Decision Log

Each row is a decision with its evidence, a proposed default, an owner, and its route. **Resolved** rows are retained for the record; **Open** rows are being sent to the named owner (client or Dev questionnaire).

| ID | Question | Evidence | Proposed default | Owner | Status |
| --- | --- | --- | --- | --- | --- |
| Q1 | Full enum for `msgStatus` (and `quoteInfo.status`)? | only `"quoted"` observed; spec declares no enums | Enumerate all values incl. failure states | Client | Open |
| Q2 | Which shape is authoritative — spec or captures? | spec `payment` object vs captures `payment` array | **Captures are truth** (ADR-0001) | Client + Dev | **Resolved** |
| Q3 | Auth flow: `authorizationCode` or client-credentials? | spec uses Guidewire-hosted Okta | **Client-credentials** | Dev | **Resolved** |
| Q4 | Shape of `errorresponse` on failure (4xx/5xx)? | only `{}` on success; spec declares no error response; TC-21/22/25 need one | Provide a failing example | Client | Open |
| Q5 | Relationship of `options[].totalPremium` to `payment[].planPremium`? | PIF = +50 in both captures, but installment deltas vary by plan/method (e.g. +110, +116, +150) — no single rule | Define the formula explicitly | Client | Open |
| Q6 | Is `planPremium` = Σ `additionalQuotedScenario[].cost[].actualAmountBilling`? | confirmed in every plan of all three captures (844=275+569; 10734=Σ of 32) | **Confirmed** | — | **Resolved** |
| Q7 | What composes top-level `premium` (`"4538.00 usd"`)? | present in `itc 2` only; matches neither plan nor option totals | Define, or mark non-testable | Client | Open |
| Q8 | Is rating deterministic for identical input? | two Confie captures differ | State answer; keep/drop TC-30 | Dev | Open |
| Q9 | Which environment is authoritative — QA or UAT? | plan said QA; URLs say `uat.trexis.com` | **UAT** | QA | **Resolved** |
| Q10 | Are `producerInfo` and `location` ever returned? | in spec, absent in all captures | Confirm optional / remove from contract | Dev | Open |
| Q11 | Can client credentials be provisioned for PERF / SIT? | credentials exist only in QA and DEV (§4.1) | Request provisioning before load testing | Dev + Client | Open |
| Q12 | Producer code per state for Confie — what are the codes? | only `producerCode.id: "139999"` observed (§14, TC-16) | Trexis to supply the per-state mapping | Trexis / Client | Open |
| Q13 | How is a quote surfaced in PORTAL from the quote link, in UAT? | Trexis requirement; no mechanism defined (§4.1) | Define mechanism + acceptance criteria; meeting with Anetra being arranged | QA + Trexis | Open |

**Resolved:** Q2 (captures authoritative), Q3 (client-credentials), Q6 (`planPremium` = Σ `actualAmountBilling`), Q9 (UAT).
**Open:** Q1, Q4, Q5, Q7, Q8, Q10, Q11, Q12, Q13 — routed to the client questionnaire (Q1, Q4, Q5, Q7, Q12, Q13) or the Dev questionnaire (Q8, Q10, Q11).

## 17. New Test Cases Implied by the Reconciliation

- **TC-31 — `planPremium` reconciliation:** for every plan, `planPremium` (parsed to number) equals the sum of `additionalQuotedScenario[].cost[].actualAmountBilling`.
- **TC-32 — shape guard:** `payment` is an array and `options` is a non-empty array (fails loudly if the spec-shaped object reappears).
- **TC-33 — conditional fields:** assert `premium` presence matches whichever rule Q7 settles.
