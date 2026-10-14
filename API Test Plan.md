Quality Engineering · Test Planning

# API Test Plan

REST API — high-level plan, scope and approach

**Target system:** Quote creation endpoint (single `POST`) **Version:** 0.3 — single-endpoint focus **Date:** 2 September 2026 **Owner:** QA / Test Engineering **Environment:** QA **Auth:** OAuth (client credentials, via Postman collection) **Tooling:** Postman

Draft — pending details

> **Contract divergences (added 2026-10-14):** The response contract is contradicted by `ComparatorSchema.json` and the captured responses. See `API Test Plan - Contract Reconciliation.md` (§A divergences, §C decision log).

Note

Scope for now is one endpoint: `POST` create-quote, tested in QA via Postman with OAuth credentials already provisioned in the collection. Requests and responses work end-to-end; what's still open is the client confirming the *correct/expected* response values — see [§12 Open Items](#pending). Everything beyond smoke/functional/negative (§3, marked Planned) is intentionally set aside for now.

Contents

[1. Purpose & Objectives](#purpose) [2. Scope](#scope) [3. Test Levels & Types](#levels) [4. Environments & Tooling](#environments) [5. Test Data Strategy](#data) [6. Entry & Exit Criteria](#criteria) [7. Roles & Responsibilities](#roles) [8. Defect Management](#defects) [9. Risks & Mitigations](#risks) [10. Schedule & Milestones](#schedule) [11. Deliverables](#deliverables) [12. Open Items](#pending)

## 1Purpose & Objectives

This plan defines the approach for verifying that the API behaves correctly, securely, and reliably before and after release — covering what gets tested, at what level, with what tooling, and who signs off.

- Confirm each endpoint meets its functional contract: correct status codes, response shapes, and business logic for valid and invalid input.
- Catch breaking changes to request/response contracts before they reach consumers.
- Verify authentication, authorization, and data-access boundaries hold under both expected and adversarial use.
- Surface performance and reliability issues (latency, error rate under load) ahead of production.

## 2Scope

### In scope (this phase)

- The single POST create-quote endpoint, in QA
- Smoke check: endpoint reachable, authenticates, returns a response
- Functional testing against the correct/expected response, once confirmed by the client
- Negative & edge-case handling (bad input, missing fields, wrong types)
- OAuth client-credentials flow, including token refresh (see §4)

## 3Test Levels & Types

Coverage is layered so cheap, fast checks run constantly and expensive ones run where they matter most.

Now Smoke Create-quote endpoint reachable, authenticates via OAuth, returns a response (allowing for its few-minute response time). Postman

Now Functional / endpoint Create-quote request/response works end-to-end today; asserting against the *correct* values once the client confirms them. Postman

Now Negative & edge case Malformed quote payloads, boundary values, missing/extra fields, wrong content-types. Postman

Planned Schema / contract Responses validated against an OpenAPI spec; consumer contracts checked for breaking changes. Pact, Dredd, JSON Schema — not yet in pipeline

Planned Regression Full automated suite re-run before each release to catch unintended breakage. CI pipeline — not yet in pipeline

Now = active this phase, run manually in Postman. Planned = deferred until tooling beyond Postman and an agreed contract are in place.

## 4Environments & Tooling

Confirmed for this phase: a single QA environment, tested manually through a Postman collection that already includes the create-quote request and OAuth credentials.

### OAuth in Postman

- Client ID, client secret, and the account/id are provided in the collection — import once, credentials are ready to use.
- Tokens expire every 2 hours; the token must be refreshed manually or via a pre-request script before it lapses mid-session.

Environment & tooling map

| Stage | Purpose | Test types run | Notes |
| --- | --- | --- | --- |
| QA | Manual + Postman-driven validation | Smoke, functional, negative | Only environment provisioned so far |
| PERF / SIT | Isolated load testing | — | Not yet provisioned |

## 5Test Data Strategy

- **Request payloads:** reuse the sample quote request already provided and working in Postman; vary fields deliberately for negative/edge cases.
- **Response latency:** the endpoint can take a few minutes to respond — set Postman request timeouts accordingly so slow-but-valid responses aren't mistaken for failures.
- **Sensitive data:** no production data in QA; use anonymized/synthetic quote details only.
- **Expected values:** not yet defined by the client — tracked as the open blocker in §12.

## 6Entry & Exit Criteria

### Entry criteria

- Correct/expected response confirmed by the client *(not yet met — current blocker, see §12)*
- QA environment is deployed and reachable
- OAuth credentials imported into Postman, with a refresh step in place before the 2-hour expiry
- Build under test is tagged and stable

### Exit criteria

- 100% of critical-path endpoints pass functional tests
- No open critical or high-severity defects
- Contract tests pass with zero breaking changes (or changes are approved)
- Performance results within agreed thresholds

## 7Roles & Responsibilities

| Role | Responsibility |
| --- | --- |
| QA / Test Engineer | Owns test plan, writes and maintains automated suites, triages failures |
| API / Backend Developer | Fixes defects, maintains OpenAPI spec, supports test data needs |

## 8Defect Management

Defects are logged with request/response evidence (method, endpoint, payload, actual vs. expected) and triaged by severity:

| Severity | Definition | Response |
| --- | --- | --- |
| Critical | Data loss, security breach, endpoint fully unavailable | Blocks release, fix immediately |
| High | Core flow broken, no workaround | Blocks release |
| Medium | Incorrect behavior with a workaround | Fix before next release |
| Low | Cosmetic or minor inconsistency | Backlog |

## 9Risks & Mitigations

| Risk | Mitigation |
| --- | --- |
| Correct/expected quote response not yet defined, blocking real assertions | Coordinate with the client now to confirm expected values; in the meantime, limit functional checks to status code and response structure, not exact values |
| OAuth token expires every 2 hours, mid-test-session | Refresh the token before each test session; consider a Postman pre-request script to auto-refresh |
| Endpoint response can take a few minutes | Set generous Postman timeouts; don't treat slow responses as failures |
| Single QA environment means no isolation from other testing/dev activity | Coordinate QA env usage windows with the team; flag data collisions early |

## 10Schedule & Milestones

Kept intentionally short while scope stays at one endpoint:

1. Client confirms correct/expected quote response
2. Functional test cases finalized in Postman against confirmed values
3. Smoke, functional, and negative suite run in QA
4. Sign-off on phase 1; phase 2 (§3 Planned items) scoped separately when it's picked up

## 11Deliverables

- Automated test suite, versioned alongside the API codebase
- Test coverage report mapped to endpoints and scenarios
- Defect log with severity breakdown
- This test plan, kept current as scope firms up

## 12Open Items — needed from you

Resolved: auth model (OAuth, credentials in the Postman collection), environment (QA), tooling (Postman), and endpoint scope (single POST create-quote). One item remains, and it's the one blocking real functional assertions:

- **Correct/expected quote response:** the request/response works today, but the client hasn't yet confirmed what the *correct* response should contain. Until that lands, functional tests can check status codes and response shape but not exact values.