# Trexis comparator quote API — developer questionnaire

**Purpose:** Close the technical decisions about the contract, determinism, and environments so the test suite and the OpenAPI spec stay in step.

**From:** QA / Test Engineering · **To:** API / Backend Developer · **How your answers will be used:** folded into the decision log (§16 of `API Test Plan v0.5.md`) and the schema/contract test level.

## Context

We've adopted the captured responses as the source of truth for the quote contract (ADR-0001), which means `ComparatorSchema.json` now needs to be brought in line with reality. These are the technical items; a separate questionnaire covers the client/business questions.

## How to answer

No hard deadline. Partial answers and "I don't know" are both useful. Write your answer on the line beneath each question.

## Contract and spec

### Q10 — Are `producerInfo` and `location` ever returned?

_Why this matters: the spec declares both, but no capture contains either — so we can't tell whether they're optional, conditionally returned, or dead in the contract._

>

### Will `ComparatorSchema.json` be regenerated to match the captured responses?

_Why this matters: until it is, schema validation (TC-04) cannot pass. Known gaps: `payment` is declared an object but returned as an array; `msgStatus`, `options`, `quoteID`, `parentQuote`, and `errorresponse` are undeclared; only a `200` response is declared._

>

## Determinism

### Q8 — For identical input, is the rating deterministic?

_Why this matters: two Confie captures differ for a look-alike insured/vehicle/state. The answer decides whether idempotency (TC-30) is a valid test, and whether each capture should be treated as input-specific._

>

## Authentication

### Q3 — Confirm the flow to test is OAuth client-credentials, not the spec's `authorizationCode`.

_Why this matters: the spec's `authorizationCode` flow points at Guidewire-hosted Okta (`guidewire-hub.oktapreview.com`), while our provisioned collection uses client-credentials. We've standardised on client-credentials (Q3 resolved) and want developer sign-off._

>

## Environments and credentials

### Q11 — Can client credentials be provisioned for PERF / SIT?

_Why this matters: performance/load testing is blocked — credentials exist only in QA and DEV._

>

## Coverage mapping (Confie → GW)

### Q17 — How do GW coverage codes surface in downstream reporting?

_Why this matters: the Confie requirement depends on the mapped codes being consumed by reporting, but the report shape and mechanism are undefined. Gates TC-60/TC-61._

>

### Q18 — When is `RREIM` (`TPARNTCov`) enabled?

_Why this matters: `RREIM` is release-gated, so TC-43/TC-55 can't run until the code is live._

>

## Anything else?

_Anything we didn't ask that we should know?_

>
