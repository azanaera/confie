Quality Engineering · Test Planning

# API Test Plan — High-Level Draft

REST API — executive summary for stakeholders

| Document ID | Version | Date | Owner | Status |
| --- | --- | --- | --- | --- |
| QE-ATP-001-HL | 1.0-HL | 16 October 2026 | QA / Test Engineering | Draft |

**Detail and reference:** [`API Test Plan v1.0.md`](./API%20Test%20Plan%20v1.0.md) · **Cases:** [`docs/test-cases.md`](./docs/test-cases.md)

This is a one-page summary of the full plan. It covers testing the Trexis comparator quote API
(`POST /trexis/comparator/quote`) in UAT, including the new requirement to add **Confie** as a comp rater
and map its coverage codes to the **GW codes** used for downstream reporting.

## Purpose

Verify the quote API behaves correctly, securely, and reliably before release: the right status codes and
response shapes, correct data, clean rejection of bad input, and correct coverage-code mapping for Confie.

## Scope at a glance

**In scope**

- The create-quote endpoint in UAT (smoke, functional, negative/edge)
- **Confie support and the Confie → GW coverage-code mapping** (new, top priority)
- Authentication (OAuth client-credentials, including token refresh)
- Response fidelity against the captured truth set (ADR-0001)

**Out of scope (this phase)**

- Performance / load testing (PERF/SIT not provisioned)
- Endpoints other than create-quote; CI automation
- Legacy coverage codes `AD`, `UDB`, `UIM` (outdated other-vendor codes — deferred)
- Genuine quote-link → PORTAL visibility until the mechanism is defined (Q13)

## Requirements at a glance

| ID | Requirement |
| --- | --- |
| R-01 | A valid request returns a well-formed quote |
| R-02 | The response conforms to the captured truth set, not the spec (ADR-0001) |
| R-03 | OAuth client-credentials authenticate; unauthenticated requests are rejected |
| R-04 | Malformed / incomplete input is rejected with a populated `errorresponse` |
| R-05 | Financial data reconciles (installments vs `totalPremium`; `planPremium` vs billed amounts) |
| **R-06** | **Confie is supported as an additional comp rater** |
| **R-07** | **Confie coverage codes map to GW codes** — every code, valid combinations, clean failure when unknown |
| **R-08** | **Downstream reporting** consumes the GW codes (not the Confie short codes) |
| R-09 | A returned quote link surfaces the quote in PORTAL (UAT) — pending |

**R-06–R-08 are the newly added, top-priority requirement.**

## Test approach at a glance

| Level | Status |
| --- | --- |
| Smoke | Now |
| Functional | Now |
| Coverage mapping (new) | Now |
| Negative & edge | Now |
| Schema / contract | Now — blocked until the OpenAPI spec is regenerated |
| Regression | Planned |
| Performance / load | Out of scope this phase |

## Environment & tooling

UAT is the authoritative environment; DEV and QA have credentials; PERF/SIT credentials are pending. Testing
runs manually through the Postman collection (OAuth client-credentials; tokens expire every 2 hours).

## Key risks

| Risk | Mitigation |
| --- | --- |
| Coverage codes not in the map / unknown-code behaviour undefined | Confirm scope and behaviour with the client (Q14, Q16) |
| Canonical response names for `UMBI`, `TL` unconfirmed | Confirm names (Q15) |
| Downstream reporting contract undefined (blocks R-08) | Define report shape + acceptance criterion (Q17) |
| `RREIM` not yet released | Release-gated; defer its cases until enabled (Q18) |
| Known financial-integrity defect (TC-20) | Tracked; financial-integrity cases fail by design until fixed |

## Milestones

1. Close the decision log (at minimum Q5, Q8, Q12, Q16, Q17)
2. Client confirms the field values routed "Client"
3. Functional and coverage-mapping cases finalized in Postman
4. Smoke, functional, coverage-mapping, and negative suites run in UAT
5. Sign-off on phase 1; performance and CI automation scoped separately

## Exit criteria (summary)

- 100% of critical-path cases pass against the captured contract
- No open critical or high-severity defects
- Financial-integrity cases pass; **coverage-mapping high-priority cases (R-06/R-07) pass**, with R-08
  verified once the reporting contract (Q17) is defined

## Open decisions blocking progress

| ID | Decision | Owner |
| --- | --- | --- |
| Q12 | Producer code per state for Confie | Trexis / Client |
| Q13 | Quote-link → PORTAL visibility mechanism (UAT) | QA + Trexis |
| Q16 | Unknown/unmapped coverage-code behaviour | Client |
| Q17 | Downstream reporting contract | Client + Dev |

Full decision log (Q1–Q18) is in Appendix D of the [full plan](./API%20Test%20Plan%20v1.0.md).

## Approvals & Sign-off

| Role | Name | Date | Signature |
| --- | --- | --- | --- |
| QA Lead / Test Engineer | | | |
| API / Backend Developer | | | |
| Client (Trexis) | | | |
| Project Manager | | | |
