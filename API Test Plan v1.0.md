Quality Engineering · Test Planning

# API Test Plan

REST API — formal test plan (structure per ISO/IEC/IEEE 29119-3)

| Document ID | Version | Date | Owner | Status |
| --- | --- | --- | --- | --- |
| QE-ATP-001 | 1.0 | 16 October 2026 | QA / Test Engineering | Draft — for approval (§6) |

**Target system:** Quote creation endpoint — `POST /trexis/comparator/createquote` (`operationId: compositeQuoteAPI`)
**Environment:** UAT (`uat.trexis.com`) — authoritative
**Auth:** OAuth client-credentials (via the Postman collection)
**Tooling:** Postman

This plan supersedes `API Test Plan v0.5.md` (kept for history, marked superseded). It folds in the new
requirement — **Confie as an additional comp rater and the Confie → GW coverage-code mapping for downstream
reporting** — reconciles the test-case catalog, and adds the formal control sections a plan of record needs.

## Revision history

| Version | Date | Author | Summary |
| --- | --- | --- | --- |
| 0.3 | 2 Sep 2026 | QA / Test Engineering | Initial single-endpoint plan |
| 0.4 | 15 Oct 2026 | QA / Test Engineering | Environments/auth/premium reconciliation; captures vs spec |
| 0.5 | 15 Oct 2026 | QA / Test Engineering | Contract reconciled, decisions closed; UAT authoritative; ADR-0001 |
| 1.0 | 16 Oct 2026 | QA / Test Engineering | Formalized to ISO 29119-3: identity/revision/approval, requirements R-01–R-09, test techniques, suspension/resumption criteria, requirement→case traceability, coverage mapping folded in |

## Contents

- [1. Introduction](#1-introduction)
- [2. Test Context](#2-test-context)
- [3. Test Management](#3-test-management)
- [4. Test Processes](#4-test-processes)
- [5. Traceability Matrix](#5-traceability-matrix)
- [6. Approvals & Sign-off](#6-approvals--sign-off)
- [Appendix A — Source Artifacts](#appendix-a--source-artifacts)
- [Appendix B — Contract Divergences](#appendix-b--contract-divergences)
- [Appendix C — Field Inventory (Observed Truth)](#appendix-c--field-inventory-observed-truth)
- [Appendix D — Decision Log / Open Questions](#appendix-d--decision-log--open-questions)

## 1. Introduction

### 1.1 Purpose

Define the approach for verifying that the comparator quote API behaves correctly, securely, and reliably
before release: what is tested, at what level, with what tooling, against which criteria, and who signs off.

### 1.2 Scope

**In scope (this phase):**

- The single endpoint `POST /trexis/comparator/createquote` (`compositeQuoteAPI`) in UAT
- Smoke: reachable, authenticates, returns a response
- Functional testing against the **captured responses as the de-facto contract** (Appendix C), pending the
  decisions in Appendix D
- **Confie support and the Confie → GW coverage-code mapping** — single codes, valid combinations, and
  negative/edge handling (the new, top-priority requirement)
- Negative & edge-case handling (bad input, missing fields, wrong types)
- OAuth client-credentials flow, including token refresh

**Out of scope (this phase):**

- **Performance / load testing** — deferred; PERF/SIT is unprovisioned and has no credentials
- Any endpoint other than the single create-quote POST
- Automated contract/regression tooling in CI (deferred)
- Genuine **quote-link → PORTAL visibility** acceptance (mechanism undefined — Q13; see R-09)
- The **legacy coverage codes `AD`, `UDB`, `UIM`** — outdated other-vendor codes, not part of the Confie
  mapping (Q14)

### 1.3 Audience

QA / Test Engineering (owner), API / Backend Development, the client (Trexis), and the project manager.

### 1.4 References

| Reference | What it is |
| --- | --- |
| [`GLOSSARY.md`](./GLOSSARY.md) | Domain vocabulary (`Confie`, `parentQuote`, `planPremium`, `GW code`, …) |
| [`docs/adr/0001-captured-responses-are-the-source-of-truth.md`](./docs/adr/0001-captured-responses-are-the-source-of-truth.md) | ADR-0001: captures are authoritative, spec regenerated |
| [`docs/test-cases.md`](./docs/test-cases.md) | Test case catalog (TC-01–TC-63) — the cases this plan cites |
| [`docs/questionnaire-client.md`](./docs/questionnaire-client.md), [`docs/questionnaire-dev.md`](./docs/questionnaire-dev.md) | Decision-log routes (Q1–Q18) |
| `ComparatorSchema.json` | OpenAPI 3.0.0 spec (`POST /trexis/comparator/createquote`); contradicted by captures |
| `ComparatorResponseFormat.json` | Documented response shape (matches the spec, not the live API) |
| `itc response body.json`, `itc 2 response body.json`, `PLRater response body.json` | Captures — the truth set |
| `requests/ITC.json`, `requests/ezlynx.json`, `requests/testcomp-EZLynx.json` | Sample requests (formatting for coverages, vehicles, drivers) |
| `quote_api_test_cases.xlsx` | Original 30-case workbook (artifact of record for TC-01–TC-30) |

### 1.5 Definitions & abbreviations

Terms are defined in `GLOSSARY.md`. Key ones: **Confie**, **PLRating**, **comp rater**, **parentQuote**,
**coverage code mapping**, **GW code**, **planPremium**, **totalPremium**, **premium**, **payment plan**,
**payment option**, **producerCode**, **msgStatus**, **quote link**, **PORTAL**, **Capture**. Abbreviations:
**TPA** = the `TPA*` GW coverage-code family; **UAT** = user acceptance testing.

## 2. Test Context

### 2.1 Test items

- The comparator quote endpoint: `POST /trexis/comparator/createquote` (`compositeQuoteAPI`).
- The coverage-code mapping component that translates Confie coverage codes to GW codes.

### 2.2 Requirements

| ID | Requirement | Primary cases |
| --- | --- | --- |
| R-01 | A valid request returns a well-formed quote (`msgStatus: "quoted"`, non-empty `payment[]`/`options[]`, resolvable `quoteInfo`) | TC-01–TC-07, TC-11–TC-17, TC-26, TC-30 |
| R-02 | The response conforms to the captured truth set, not the spec (ADR-0001) | TC-04–TC-07, TC-18, TC-19, TC-31, TC-32, TC-33 |
| R-03 | OAuth client-credentials authenticate; unauthenticated requests are rejected | TC-25; smoke |
| R-04 | Malformed/incomplete input is rejected with a populated `errorresponse`; invalid data produces no quote | TC-21–TC-24, TC-28 |
| R-05 | Financial data reconciles: installments vs `totalPremium`, `planPremium` vs `actualAmountBilling` | TC-08–TC-10, TC-19, TC-20, TC-27, TC-31 |
| R-06 | **Confie is supported as an additional comp rater** | TC-59, TC-63 |
| R-07 | **Confie coverage codes map to GW codes** for every code, in valid combinations, and fail cleanly when unknown | TC-34–TC-58, TC-62 |
| R-08 | **Downstream reporting** consumes the GW codes (not the Confie short codes) | TC-60, TC-61 |
| R-09 | A returned quote link surfaces the quote in PORTAL (UAT) | Pending — mechanism undefined (Q13) |

### 2.3 Features to be tested / not to be tested

- **To be tested:** quote creation contract; Confie support; coverage-code mapping (single codes,
  combinations, negatives) and its downstream reporting code; auth; negative/edge input; contract fidelity
  against captures.
- **Not to be tested (this phase):** performance/load; endpoints other than create-quote; CI automation;
  legacy `AD`/`UDB`/`UIM` codes; genuine PORTAL visibility until Q13 defines it.

### 2.4 Test levels & types

| Level | Description | Tooling | Status |
| --- | --- | --- | --- |
| Smoke | Endpoint reachable, authenticates via OAuth, returns a response | Postman | Now |
| Functional / endpoint | Request/response works end-to-end, asserted against the captured truth set | Postman | Now |
| **Coverage mapping** | **Confie code → GW code → canonical response name; single codes, combinations, negatives** | Postman | **Now** |
| Negative & edge case | Malformed payloads, boundary values, missing/extra fields, wrong content-types | Postman | Now |
| Schema / contract | Responses validated against the OpenAPI spec | Pact, Dredd, JSON Schema | Now — blocked until `ComparatorSchema.json` is regenerated (ADR-0001) |
| Regression | Full automated suite re-run before each release | CI pipeline | Planned |
| Performance / load | Latency and error rate under load | — | Out of scope this phase |

### 2.5 Test techniques

- **Equivalence partitioning** — each coverage code is a partition; known vs unknown; limit-based vs
  deductible-based.
- **Decision table** — coverage item value field (`limit` / `deductible` / `value`) and the mapping outcome.
- **Boundary value analysis** — limit formats (`25`, `25/50`, `40/1200`), installment sums vs `totalPremium`
  (within ±$1.00).
- **Combination / pairwise** — coverage sets across multiple vehicles and drivers.
- **State transition** — OAuth token issue/expiry/refresh.

### 2.6 Test data strategy

- **Request payloads:** reuse the shapes in `requests/`; vary coverages, vehicles, and drivers deliberately.
  Coverage blocks follow `docs/test-cases.md` §8.3 (Confie codes only).
- **Expected values:** the de-facto truth set is the three captures — `itc`, `itc 2`
  (both `parentQuote: "Confie"`) and `PLRater` (`parentQuote: "PLRating"`); each returns 16 `payment[]` and
  16 `options[]` entries.
- **Coverage mapping data:** the code map in `docs/test-cases.md` §8.1; `RREIM` is release-gated (Q18).
- **Latency:** the endpoint can take a few minutes — set generous Postman timeouts.
- **Determinism:** the two Confie captures differ for look-alike input (Q8); treat every capture as
  input-specific and hold TC-30 provisionally.
- **Sensitive data:** synthetic/anonymized quote details only.

### 2.7 Test environment & tooling

- UAT is authoritative; DEV and QA have credentials; PERF/SIT credentials not yet provisioned (Q11).
- Tested manually through the Postman collection that already carries the create-quote request and OAuth
  credentials. Tokens expire every 2 hours (refresh before expiry).

| Stage | Purpose | Test types | Credentials | Notes |
| --- | --- | --- | --- | --- |
| DEV | Developer validation | — | Yes | Provisioned |
| QA | Manual + Postman validation | Smoke, functional, negative | Yes | Provisioned |
| UAT | **Authoritative** | Smoke, functional, negative, coverage mapping | Yes | Captured URLs are `uat.trexis.com` |
| PERF / SIT | Load testing | — | **Not yet** | Unprovisioned (§1.2) |

### 2.8 Test deliverables

- Postman collection + scripts, versioned in this repository
- Test case catalog: `docs/test-cases.md` (TC-01–TC-63)
- Coverage report mapped to requirements R-01–R-09; defect log with severity breakdown
- Domain docs: `GLOSSARY.md`, `docs/adr/`
- Decision log (Appendix D) tracked to closure; the two questionnaires
- This plan, kept current as scope firms up

## 3. Test Management

### 3.1 Organization & roles

| Role | Responsibility |
| --- | --- |
| QA / Test Engineer | Owns the plan, writes and maintains the suite, triages failures |
| API / Backend Developer | Fixes defects, maintains the OpenAPI spec, supplies test data |
| Client (Trexis) | Confirms expected values; owns the answers routed "Client" in Appendix D |
| Project Manager | Tracks milestone, risks, and sign-off |

### 3.2 Schedule & milestones

1. Close the Appendix D decision log — at minimum Q5, Q8, Q12, Q16, Q17
2. Client confirms the field values routed "Client"
3. Functional and coverage-mapping cases finalized in Postman against the captured set
4. Smoke, functional, coverage-mapping, and negative suites run in UAT
5. Sign-off on phase 1; performance and CI automation scoped separately when provisioned

### 3.3 Risks & mitigations

| Risk | Mitigation |
| --- | --- |
| Response contract contradicted by the spec, blocking schema assertions | Captures are authoritative (ADR-0001); regenerate the spec before relying on it |
| **Coverage codes not in the map (unknown / `AD`/`UDB`/`UIM`)** | Confirm scope (Q14) and unknown-code behaviour (Q16); assert a single rule consistently |
| **Canonical names for `UMBI`/`TL` unconfirmed** | Confirm before TC-41/TC-42 can pass (Q15) |
| **Downstream reporting contract undefined** | Define report shape + acceptance criterion (Q17); TC-60/TC-61 blocked until then |
| `RREIM` not yet released | Release-gated; defer TC-43/TC-55 until enabled (Q18) |
| Correct/expected values for some fields unconfirmed | Drive Appendix D to closure; assert structure meanwhile |
| OAuth token expires every 2 hours, mid-session | Refresh before each session; consider a pre-request script |
| Endpoint response can take a few minutes | Generous Postman timeouts; don't treat slow responses as failures |
| Known financial-integrity failure (TC-20) | Tracked as a defect; TC-10 fails by design until fixed |
| Producer codes unknown per state (Q12) | Confirm with Trexis before state-specific requests |

### 3.4 Defect management

Defects are logged with request/response evidence (method, endpoint, payload, actual vs. expected) and triaged:

| Severity | Definition | Response |
| --- | --- | --- |
| Critical | Data loss, security breach, endpoint fully unavailable | Blocks release, fix immediately |
| High | Core flow broken, no workaround | Blocks release |
| Medium | Incorrect behavior with a workaround | Fix before next release |
| Low | Cosmetic or minor inconsistency | Backlog |

### 3.5 Monitoring, control & status reporting

- **Metrics:** cases executed / passed / failed by level; coverage-mapping pass rate (R-07); defect count by
  severity; blocked cases (Q13/Q17/Q18).
- **Cadence:** status update to the PM at each milestone in §3.2; ad-hoc for new critical/high defects.
- **Control:** scope changes re-enter this plan via revision history; new cases are added to
  `docs/test-cases.md` and traced in §5.

## 4. Test Processes

### 4.1 Entry criteria

- The authoritative response shape is settled — **met** (captures authoritative, ADR-0001)
- Target environment (UAT) deployed and reachable
- OAuth credentials imported into Postman, refresh step in place
- The coverage-code map (§2.2 R-07) supplied and the build tagged

### 4.2 Exit criteria

- 100% of critical-path cases pass against the captured contract
- No open critical or high-severity defects
- Any deviation from the captured contract is fixed in the spec or approved as an accepted change — *the
  written "zero breaking changes" test is unachievable until `ComparatorSchema.json` is regenerated*
- Financial-integrity cases (TC-08, TC-10, TC-31) pass — currently a known failure (TC-20)
- **Coverage mapping: 100% of high-priority R-06/R-07 cases pass**, with R-08 verified once Q17 defines the
  reporting contract
- Performance exit criteria deferred (§1.2)

### 4.3 Suspension & resumption criteria

**Suspend testing when:**

- UAT is unreachable, or OAuth credentials/refresh are unavailable
- A defect blocks more than 25% of the planned cases in a level
- A new deployment changes the captured contract such that the truth set no longer holds

**Resume when:** the environment/auth is restored, the blocking defect is fixed or worked around, or the
contract is re-baselined (a new capture is added to Appendix A/C). Suspended cases are marked accordingly
and re-run on resumption.

## 5. Traceability Matrix

Each requirement maps to the fields/artifacts it exercises and the cases that prove it (full catalog in
`docs/test-cases.md`).

| Requirement | Field(s) / artifact | Proving cases |
| --- | --- | --- |
| R-01 Core quote | `msgStatus`, `payment[]`, `options[]`, `quoteInfo`, request echoes | TC-01–TC-07, TC-11–TC-17, TC-26, TC-30 |
| R-02 Contract fidelity | `payment[]`/`options[]` shapes, conditional `premium` | TC-04–TC-07, TC-18, TC-19, TC-31, TC-32, TC-33 |
| R-03 Authentication | OAuth scheme | TC-25; smoke |
| R-04 Negative & edge | `errorresponse`, validation | TC-21–TC-24, TC-28 |
| R-05 Financial integrity | `planPremium`, `totalPremium`, `fee`, `premium`, `installmentData[]` | TC-08–TC-10, TC-19, TC-20, TC-27, TC-31 |
| R-06 Confie comp rater | `quoteInfo.parentQuote` | TC-59, TC-63 |
| R-07 Coverage mapping | `vehicles[].coverage[]`, code map | TC-34–TC-58, TC-62 |
| R-08 Downstream reporting | GW code in report/extract | TC-60, TC-61 |
| R-09 Quote-link → PORTAL | `quoteInfo.summaryURL`/`bridgeURL` | Pending (Q13) |
| **D1** `payment` is an array | `payment[]` | TC-05, TC-31, TC-32 |
| **D2** spec-absent fields | `msgStatus`, `options`, `quoteID`, `parentQuote`, `errorresponse` | TC-02, TC-03, TC-06, TC-07, TC-32 |
| **D3** auth flow | OAuth scheme | TC-25; smoke |
| **D4** environment | base URL | TC-01 |
| **D5** value relationships | `planPremium`, `totalPremium`, `fee`, `premium` | TC-08, TC-10, TC-31 |
| **D6** determinism | `quoteID`, premium | TC-30 |
| **D7** conditional fields | `premium`, top-level `coverage` | TC-33 |
| **D8** coverage-name normalization | `coverage[].name` | TC-34–TC-58 |

TC-29 (response time under load) is deferred with the performance level (§1.2) and is not mapped to an
in-scope requirement this phase.

## 6. Approvals & Sign-off

| Role | Name | Date | Signature |
| --- | --- | --- | --- |
| QA Lead / Test Engineer | | | |
| API / Backend Developer | | | |
| Client (Trexis) | | | |
| Project Manager | | | |

## Appendix A — Source Artifacts

| Artifact | What it is |
| --- | --- |
| `ComparatorSchema.json` | OpenAPI 3.0.0 spec — `POST /trexis/comparator/createquote`; declares only a `200` response and no enums |
| `ComparatorResponseFormat.json` | Documented response shape (matches the spec, not the live API) |
| `quote_api_test_cases.xlsx` | 30 test cases (sheets *Test Cases*, *Summary*); transcribed to `docs/test-cases.md` |
| `itc response body.json`, `itc 2 response body.json` | Captures, `parentQuote: "Confie"` |
| `PLRater response body.json` | Capture, `parentQuote: "PLRating"` |
| `requests/*.json` | Sample requests (coverage/vehicle/driver formatting) |
| `GLOSSARY.md`, `docs/adr/0001-…md` | Domain vocabulary and the source-of-truth decision |

## Appendix B — Contract Divergences

| Aspect | `ComparatorSchema.json` | `ComparatorResponseFormat.json` | Captures |
| --- | --- | --- | --- |
| `payment` | object | object | **array** of plans |
| `options[]` | absent | absent | present |
| `msgStatus` | absent | absent | present (`"quoted"`) |
| top-level `premium` | absent | absent | present in `itc 2` only |
| `quoteInfo.quoteID` | absent | absent | present |
| `quoteInfo.parentQuote` / `.status` | absent | absent | present |
| `quoteInfo.quotePreparedDT` | present | present | absent |
| `errorresponse` | absent | absent | present (`{}`) |
| `producerInfo`, `location` | present | present | absent |

- **D1 — `payment` type.** Spec declares an object; the API returns an array.
- **D2 — Missing fields.** `msgStatus`, `options`, `quoteID`, `parentQuote`, `errorresponse` exist in reality
  but not in the spec.
- **D3 — Auth flow.** Spec = `authorizationCode`; resolved: client-credentials.
- **D4 — Environment.** Resolved: UAT.
- **D5 — Value relationships.** `planPremium`, `totalPremium`, `fee`, `premium` do not reconcile under one rule.
- **D6 — Determinism.** The two Confie captures differ for look-alike input.
- **D7 — Conditional fields.** `premium` and top-level `coverage` presence is undefined.
- **D8 — Coverage-name normalization (new).** Confie sends short codes at `vehicles[].coverage[]`; the
  response returns canonical display names only, and the GW code is not surfaced to the caller — mapping
  correctness must therefore be proven at the display-name and downstream-report boundaries.

## Appendix C — Field Inventory (Observed Truth)

### Response — top level

| Field | Type (observed) | Schema? | Example | Notes |
| --- | --- | --- | --- | --- |
| `msgStatus` | string (enum?) | no | `"quoted"` | only `quoted` observed (Q1) |
| `policyInfo` | object | yes (differs) | — | see below |
| `payment` | **array** (16) | yes (as object) | — | D1 |
| `options` | array (16) | **no** | — | D2 |
| `premium` | string | no | `"4538.00 usd"` | present in `itc 2` only; composition unknown (Q7) |
| `coverage` | array | yes | `[]` or `[{name, limit}]` | policy-level; empty in `itc 2` |
| `drivers` | array | yes | — | |
| `vehicles` | array | yes | — | `vehicles[].coverage[]` carries the mapped codes |
| `quoteInfo` | object | yes (differs) | — | |
| `errorresponse` | object | no | `{}` | failure shape unknown (Q4) |
| `producerInfo` / `location` | — | yes | — | not seen in any capture (Q10) |

### Response — `policyInfo`

| Field | Type | Example | Notes |
| --- | --- | --- | --- |
| `insured` | object | `{firstName, lastName, primaryAddress, emailAddress1}` | `validUntil` returns the string `"null"` (TC-18) |
| `producerCode.id` | string | `"139999"` | capture-only; per-state mapping (Q12, TC-16) |
| `baseState` | string | `"IN"` | |
| `jobEffectiveDate` / `jobExpirationDate` | string (ISO) | `"2026-07-25T05:01:00Z"` / `"2027-07-25T05:01:00Z"` | |
| `termType` | string | `"Annual"` | enum? |
| `policyType` | string | `"PersonalAuto"` | enum? |
| `effectiveDt` | string (ISO) | `"2026-07-25T05:01:00Z"` | TC-14 expects day delta 0 or −1 |
| `expirationDt` | string (ISO) | `"2027-07-25T05:01:00Z"` | TC-15 |
| `product` | string | `"Personal Auto"` | |
| `uwCompany` | string | `"Trexis Insurance Corp"` | |

### Response — `payment[]` (16 entries)

| Field | Type | Example | Notes |
| --- | --- | --- | --- |
| `paymentPlanCd` | string | `"1-Month Down - EFT"` | plan name joined with method |
| `planPremium` | string | `"844.00 usd"` | equals Σ `actualAmountBilling` (Q6, TC-19) |
| `additionalQuotedScenario[]` | array | `[{cost:[{actualAmountBilling, chargePattern, displayName}], coverable}]` | 2–32 `cost[]` per plan |

### Response — `options[]` (16 entries)

| Field | Type | Example | Notes |
| --- | --- | --- | --- |
| `paymentPlanName` | string | `"1-Month Down"` | without method suffix |
| `paymentType` | string | `"PIF"` / `"EFT"` / `"RCC"` / `"Direct"` | enum |
| `totalPremium` | number | `1010.00` | differs from `planPremium` (Q5) |
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
| `quoteID` | string (UUID) | `"b4e159ce-…"` | capture-only |
| `parentQuote` | string | `"Confie"` / `"PLRating"` | identifies the upstream comp rater (TC-59) |
| `status` | string | `"quoted"` | duplicates `msgStatus`? |
| `quotePreparedDT` | string | — | spec-only, absent in captures |

### Request — coverage items

| Surface | Field | Notes |
| --- | --- | --- |
| `vehicles[].coverage[]` (singular) | `name` + (`limit` \| `deductible` \| `value`) | **In scope** — where Confie sends coverage codes |
| `policy.coverages[]` (plural) | same shape | Out of scope for Confie; not used in the mapping |

## Appendix D — Decision Log / Open Questions

| ID | Question | Evidence | Proposed default | Owner | Status |
| --- | --- | --- | --- | --- | --- |
| Q1 | Full enum for `msgStatus` (and `quoteInfo.status`)? | only `"quoted"` observed; no enums in spec | Enumerate all values incl. failure states | Client | Open |
| Q2 | Which shape is authoritative — spec or captures? | spec `payment` object vs captures array | **Captures are truth** (ADR-0001) | Client + Dev | **Resolved** |
| Q3 | Auth flow: `authorizationCode` or client-credentials? | spec uses Guidewire Okta | **Client-credentials** | Dev | **Resolved** |
| Q4 | Shape of `errorresponse` on failure (4xx/5xx)? | only `{}` on success; no error response in spec | Provide a failing example | Client | Open |
| Q5 | Relationship of `options[].totalPremium` to `payment[].planPremium`? | no single rule | Define the formula explicitly | Client | Open |
| Q6 | Is `planPremium` = Σ `actualAmountBilling`? | confirmed in all captures | **Confirmed** | — | **Resolved** |
| Q7 | What composes top-level `premium`? | present in `itc 2` only | Define, or mark non-testable | Client | Open |
| Q8 | Is rating deterministic for identical input? | two Confie captures differ | State answer; keep/drop TC-30 | Dev | Open |
| Q9 | Which environment is authoritative — QA or UAT? | URLs say `uat.trexis.com` | **UAT** | QA | **Resolved** |
| Q10 | Are `producerInfo` and `location` ever returned? | in spec, absent in all captures | Confirm optional / remove from contract | Dev | Open |
| Q11 | Can client credentials be provisioned for PERF / SIT? | credentials exist only in QA and DEV | Request provisioning before load testing | Dev + Client | Open |
| Q12 | Producer code per state for Confie? | only `139999` (IN) observed | Trexis to supply the per-state mapping | Trexis / Client | Open |
| Q13 | How is a quote surfaced in PORTAL from the quote link, in UAT? | Trexis requirement; no mechanism defined | Define mechanism + acceptance criteria | QA + Trexis | Open |
| Q14 | Confie codes `AD`, `UDB`, `UIM`? | appear in legacy sample requests; not in the Confie list | **Deferred / out of scope** — outdated other-vendor codes; revisit only if Confie sends them | Client | Deferred |
| Q15 | Canonical names for `UMBI` and `TL`? | no capture shows them | Confirm exact response names | Client | Open |
| Q16 | Unknown/unmapped coverage code behaviour? | undefined; `errorresponse` shape unknown (Q4) | Reject or ignore — assert one rule | Client | Open |
| Q17 | Downstream reporting contract (where GW codes surface)? | requirement names reporting; no shape | Define report shape + acceptance criterion | Client + Dev | Open |
| Q18 | When is `RREIM` (`TPARNTCov`) enabled? | code is release-gated | Confirm release date | Dev | Open |

**Resolved:** Q2, Q3, Q6, Q9. **Deferred:** Q14. **Open:** Q1, Q4, Q5, Q7, Q8, Q10–Q13, Q15–Q18 — routed
to the client questionnaire (Q1, Q4, Q5, Q7, Q12–Q17) or the Dev questionnaire (Q8, Q10, Q11, Q17, Q18).
