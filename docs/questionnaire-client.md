# Trexis comparator quote API — client questionnaire

**Purpose:** Close the response-contract questions that only the business/contract owner can answer, so functional assertions run against confirmed values rather than assumptions.

**From:** QA / Test Engineering · **To:** Trexis (contract owner) · **How your answers will be used:** folded into the decision log (§16 of `API Test Plan v0.5.md`) and the Postman suite.

## Context

We test `POST /trexis/comparator/quote` in UAT. We hold three captured responses plus an OpenAPI spec that disagree with each other, and we've adopted the captures as authoritative. Most of the contract is now settled; these are the remaining business questions. A companion questionnaire goes to the developers for the technical items.

## How to answer

No hard deadline. Partial answers and "I don't know" are both useful — flag anything you're unsure of rather than skipping it. Write your answer on the line beneath each question.

## Status and failure behaviour

### Q1 — What are the full allowed values of `msgStatus` (and `quoteInfo.status`)?

_Why this matters: we have only ever seen `quoted`; we need the failure states named to test them._

>

### Q4 — On a failed request (4xx/5xx), what does `errorresponse` contain?

_Why this matters: we only have an empty `{}` on success, so our negative cases have no shape to assert._

>

## Premium relationships

### Q5 — How does `options[].totalPremium` relate to `payment[].planPremium`?

_Why this matters: they never match. For Paid-in-Full the difference is +50 in both captures, but for installment plans it varies by plan and method, so we cannot infer the rule. Example: PIF 1010 vs 960._

>

### Q7 — What makes up the top-level `premium` (e.g. `"4538.00 usd"`)?

_Why this matters: it appeared in only one capture and matches no plan or option total, so we can't tell whether it should be tested at all._

>

## Configuration

### Q12 — What producer code should be used for Confie, per state?

_Why this matters: a quote request needs a state-specific producer code; we have only seen `139999` (IN)._

>

### What base URL should we use for each environment (DEV, QA, UAT)?

_Why this matters: we currently only know UAT (`uat.trexis.com`)._

>

## Portal visibility

### Q13 — How should a quote appear in PORTAL via its quote link, and what counts as success?

_Why this matters: Trexis expects the quote to be visible in PORTAL after a response; we need acceptance criteria to test it. (A meeting with Anetra is being arranged.)_

>

## Coverage mapping (Confie → GW)

### Q14 — Confie codes `AD`, `UDB`, `UIM`.

_Why this matters: these appear in the legacy sample requests but not in the Confie code list. We've treated them as outdated other-vendor codes and deferred them. Confirm they are out of scope for Confie._

>

### Q15 — Canonical names for `UMBI` and `TL`.

_Why this matters: no capture shows `Uninsured Motorist Bodily Injury` (uninsured) or `Towing & Labor`; we need the exact response names before TC-41/TC-42 can pass._

>

### Q16 — Unknown/unmapped coverage code behaviour.

_Why this matters: we don't know whether an unknown code is rejected or silently ignored, nor what `errorresponse` contains (ties to Q4). Drives TC-54._

>

### Q17 — Downstream reporting contract.

_Why this matters: the requirement says the GW coverage codes feed downstream reporting, but we have no report shape or acceptance criterion. Gates TC-60/TC-61._

>

## Anything else?

_Anything we didn't ask that we should know?_

>
