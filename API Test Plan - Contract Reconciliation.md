# API Test Plan — Addendum: Contract Reconciliation

**Companion to:** `API Test Plan.md` v0.3 (2 September 2026)
**Status:** Draft proposal — this file changes nothing in v0.3; it is the delta to be merged in.
**Purpose:** replace the single open bullet in §12 with a field-level contract reconciliation, grounded in the three artifact classes now on disk.

Artifacts referenced:

- `ComparatorSchema.json` — OpenAPI 3.0 spec (`POST /trexis/comparator/quote`, `operationId: compositeQuoteAPI`)
- `ComparatorResponseFormat.json` — documented response shape
- Captures: `itc response body.json`, `itc 2 response body.json` (both `parentQuote: "Confie"`), `PLRater response body.json` (`parentQuote: "PLRating"`)

---

## 0. Proposed changes to v0.3 (delta only)

| v0.3 section | Current wording | Proposed change |
| --- | --- | --- |
| §2 Scope | "The single POST create-quote endpoint" | Name method + exact path + base URL: `POST /trexis/comparator/quote` |
| §3 Test Levels | Schema / contract = **Planned** ("an agreed contract" not in place) | Move to **Now**. An OpenAPI contract exists; the real constraint is that it is *wrong*, not absent |
| §4 Environments | Environment = **QA** | Reconcile the label with the captured `uat.trexis.com` URLs (QA vs UAT) |
| §5 Test Data | Expected values "not yet defined" | Point at the captures as the de-facto truth set, with the caveats in §A |
| §6 Exit Criteria | "Contract tests pass with zero breaking changes" | Reword — unachievable today until the spec is reconciled (§A) |
| §11 Deliverables | "Automated test suite, versioned alongside the API codebase" | There is no codebase/git repo here; name the suite's actual home |
| §12 Open Items | One bullet | Replaced by the decision log in §C |
| Contents / TOC | Anchors `#purpose`, `#scope`… | Don't match the `## 1Purpose`-style headings; fix or drop |

---

## A. The three artifact classes, and how they disagree

The plan assumes the response contract is unknown. It is not — it is *contradicted*. Summary:

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

- **D1 — `payment` type.** Spec/format declare `payment` as an object; the API returns an **array**. Every schema-level assertion fails against `ComparatorSchema.json` as written.
- **D2 — Missing fields.** `msgStatus`, `options`, `quoteID`, `parentQuote`, `errorresponse` exist in reality but not in the spec — yet TC-02 and TC-03 assert on two of them. The test cases were written against the captures, not the spec.
- **D3 — Auth flow.** Spec = OAuth **`authorizationCode`** (Okta `tokenUrl`); plan = **client-credentials**. One is wrong.
- **D4 — Environment.** Plan says **QA**; captured URLs are `uat.trexis.com`.
- **D5 — Value relationships.** `payment[].planPremium`, `options[].totalPremium`, `fee`, and top-level `premium` do not reconcile under any single rule. See §C Q5–Q7.
- **D6 — Determinism.** The two Confie captures differ for the same-looking insured/vehicle/state. Different inputs, or rating variance? This decides whether idempotency (TC-30) is meaningful.
- **D7 — Conditional fields.** Top-level `premium` appears in one capture and not another; `coverage` is non-empty in one and `[]` in another. Required vs optional is undefined.

---

## B. Field inventory (observed truth)

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

---

## C. Open questions / decision log (replaces §12)

Recorded as decisions to obtain, each with the evidence and a proposed default.

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

### New test cases implied by this reconciliation

- **TC-31 — `planPremium` reconciliation**: for every plan, `planPremium` (parsed to number) equals the sum of `additionalQuotedScenario[].cost[].actualAmountBilling`.
- **TC-32 — shape guard**: `payment` is an array and `options` is a non-empty array (fails loudly if the spec-shaped object reappears).
- **TC-33 — conditional fields**: assert `premium` presence matches whichever rule Q7 settles.
