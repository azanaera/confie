# Quote API — Test Case Catalog

Transcribed from `quote_api_test_cases.xlsx` (sheets **Test Cases** and **Summary**), extended with the
cases implied by the contract reconciliation and the Confie coverage-mapping requirement. The workbook
remains the artifact of record for the original 30 cases; this file is the catalog the test plans cite.

**Columns:** ID · Section · Test Case · Expected Result · Source · Priority · Status

**Conventions:**

- **Source** — `Existing script` (already in the Postman collection), `Suggested` (proposed, not yet
  implemented), or `Coverage mapping (new)` (added for the Confie → GW coverage-code requirement).
- **Priority** — the workbook's suggested defaults (assumptions), not yet ratified.
- **Status** — all `Not Run` as of transcription.
- `Actual Result`, `Defect Link`, and `Notes` columns exist in the workbook but are empty; omitted here.

## Summary

| Metric | Value |
| --- | --- |
| Total cases | 63 |
| Priority — High | 36 |
| Priority — Medium | 24 |
| Priority — Low | 3 |
| Source — Existing script | 20 |
| Source — Suggested | 22 |
| Source — Coverage mapping (new) | 21 |
| Status — Not Run | 63 |
| Pass rate (of executed) | 0 (nothing executed) |

## 1. Basic response

| ID | Test Case | Expected Result | Source | Priority | Status |
| --- | --- | --- | --- | --- | --- |
| TC-01 | Submit a valid quote request | HTTP 200 | Existing script | High | Not Run |
| TC-02 | Check msgStatus | Equals `'quoted'` | Existing script | High | Not Run |
| TC-03 | Check errorresponse | Empty object `{}` | Existing script | High | Not Run |

## 2. Schema and structure

| ID | Test Case | Expected Result | Source | Priority | Status |
| --- | --- | --- | --- | --- | --- |
| TC-04 | Validate the response against the quote response schema | Passes with no schema errors | Existing script | High | Not Run |
| TC-05 | Inspect the payment array | Non-empty. Each plan has paymentPlanCd (string), planPremium (string), additionalQuotedScenario (array) | Existing script | High | Not Run |
| TC-06 | Inspect the options array | Non-empty. Each option has numeric totalPremium, installment, fee, downPayment, plus a non-empty installmentData array | Existing script | High | Not Run |
| TC-07 | Inspect quoteInfo | quoteID is a UUID (36 chars). summaryURL and bridgeURL both contain the quoteID | Existing script | Medium | Not Run |

## 3. Financial integrity

| ID | Test Case | Expected Result | Source | Priority | Status |
| --- | --- | --- | --- | --- | --- |
| TC-08 | For every option, sum installmentData[].amount and compare to totalPremium | Difference within +/-$1.00 for ALL plans (all mismatches reported together) | Existing script | High | Not Run |
| TC-09 | Compare Direct Bill vs EFT/RCC options with the same plan name | Installment count is identical | Existing script | Medium | Not Run |
| TC-10 | Check each EFT/RCC installment against Direct Bill | Reconciles to the option's own totalPremium _(currently fails, see TC-20)_ | Existing script | High | Not Run |

## 4. Request-response consistency

| ID | Test Case | Expected Result | Source | Priority | Status |
| --- | --- | --- | --- | --- | --- |
| TC-11 | Compare vehicles[0].vin to the request VIN | Exact match | Existing script | High | Not Run |
| TC-12 | Compare driver licenseNumber and licenseState | Exact match | Existing script | High | Not Run |
| TC-13 | Compare insured firstName, lastName and emailAddress1 | Exact match | Existing script | High | Not Run |
| TC-14 | Compare effectiveDt to the requested date (date only) | Day delta is 0 or -1 | Existing script | Medium | Not Run |
| TC-15 | Compare expirationDt to the requested date (date only) | Day delta is 0 or -1 | Existing script | Medium | Not Run |
| TC-16 | Compare producerCode.id | Exact match | Existing script | Medium | Not Run |
| TC-17 | Compare vehicle make/model to the request | May differ (normalization, e.g. TOYOTA -> TYTA). Logged as informational, never fails | Existing script | Low | Not Run |

## 5. Known issues

| ID | Test Case | Expected Result | Source | Priority | Status |
| --- | --- | --- | --- | --- | --- |
| TC-18 | validUntil type | Currently returns the string `"null"`, not JSON null. Flip to assert null once fixed | Existing script | Medium | Not Run |
| TC-19 | planPremium format | Currently a string like `"844.00 usd"`. Flip to numeric assertion plus separate currency field once fixed | Existing script | Medium | Not Run |
| TC-20 | EFT/RCC installment calculation | Currently installments equal Direct's minus the fee difference (`'1-Month Down'` first installment is fee-exempt). Delete test once fixed; TC-08/TC-10 then cover correct behavior | Existing script | High | Not Run |

## 6. Suggested additions

| ID | Test Case | Expected Result | Source | Priority | Status |
| --- | --- | --- | --- | --- | --- |
| TC-21 | Missing required field (e.g., no VIN) | 4xx status and a populated errorresponse | Suggested | High | Not Run |
| TC-22 | Invalid VIN format or invalid license state | Validation error, no quote produced | Suggested | High | Not Run |
| TC-23 | Expiration date before effective date | Rejected with a clear error | Suggested | Medium | Not Run |
| TC-24 | Invalid or unknown producer code | Rejected, or the quote is not issued | Suggested | Medium | Not Run |
| TC-25 | Missing/invalid auth token | 401/403 | Suggested | High | Not Run |
| TC-26 | Multiple vehicles or drivers in one request | vehicles[] and drivers[] counts match the request, each item verified, not just index 0 | Suggested | High | Not Run |
| TC-27 | `'Paid in Full'` plan | Single installment equal to totalPremium, no Direct counterpart required | Suggested | Medium | Not Run |
| TC-28 | Schema env var not set | Schema test is skipped with a warning and the rest of the run continues | Suggested | Low | Not Run |
| TC-29 | Response time under load or SLA threshold | Responds within the agreed limit (e.g., 3s) | Suggested | Medium | Not Run |
| TC-30 | Idempotency: same request twice | Different quoteID, same premium | Suggested | Medium | Not Run |

## 7. Cases implied by the contract reconciliation

Moved out of `API Test Plan v0.5.md` §17 into the catalog so the plan can cite them.

| ID | Test Case | Expected Result | Source | Priority | Status |
| --- | --- | --- | --- | --- | --- |
| TC-31 | `planPremium` reconciliation: for every plan, parsed `planPremium` equals the sum of `additionalQuotedScenario[].cost[].actualAmountBilling` | Equal for all 16 plans | Suggested | High | Not Run |
| TC-32 | Shape guard: `payment` is an array and `options` is a non-empty array | Fails loudly if the spec-shaped object reappears | Suggested | High | Not Run |
| TC-33 | Conditional fields: presence of top-level `premium` matches whichever rule Q7 settles | Presence matches the Q7 rule | Suggested | Medium | Not Run |

## 8. Coverage mapping (Confie → GW)

Cases for the new requirement: Confie is added as a comp rater, and its coverage codes must map to the GW
codes used for rating and downstream reporting.

### 8.1 The code map

"The canonical name" is what the response actually returns in `coverage[].name`.

| Confie code | GW code (required) | Canonical name (response) | Value field | Evidence | Status |
| --- | --- | --- | --- | --- | --- |
| `BI` | `TPABICov` | Bodily Injury | `limit` | observed in `itc` / `itc 2` / `PLRater` | Live |
| `PD` | `TPAPDCov` | Property Damage | `limit` | observed in all captures | Live |
| `MEDPM` | `TPAMPCov` | Medical Payments | `limit` | observed in `itc` | Live |
| `COMP` | `TPACOMPCov` | Comprehensive | `deductible` | observed in `itc` | Live |
| `COLL` | `TPACOLCov` | Collision | `deductible` | observed in `itc` | Live |
| `UMPD` | `TPAUMPDCov` | Uninsured Motorist Property Damage | `limit` | observed in `itc` | Live |
| `UIMBI` | `TPAUIMBICov` | Underinsured Motorist Bodily Injury | `limit` | observed in `itc` | Live |
| `UMBI` | `TPAUMBICov` | Uninsured Motorist Bodily Injury | `limit` | name **inferred** — not observed | Live |
| `TL` | `TPATOWCov` | Towing & Labor | `limit` | name **inferred** — not observed | Live |
| `RREIM` | `TPARNTCov` | Rental Reimbursement | `limit` | name observed; code **release-gated** | Soon |

> **Out of scope (deferred).** `AD`, `UDB`, and `UIM` appear in the sample requests but are **outdated codes
> from other vendors**, not part of the Confie mapping. This requirement prioritizes the codes above, so
> these are skipped for now (Q14). They must not be used as Confie test data.

### 8.2 How coverage travels

- **Request — vehicle level (in scope):** `vehicles[].coverage[]` (singular), e.g.
  `{"name": "COMP", "deductible": "250"}`. This is the surface Confie uses.
- **Request — policy level (out of scope):** `policy.coverages[]` (plural) exists in the contract but is not
  part of the Confie mapping.
- **Item shape** (`CoverageDTO`): `name` plus value as one of `limit`, `deductible`, or `value`.
- **Response:** `vehicles[].coverage[]` carries the mapped coverage, normalized to canonical display names —
  neither the Confie short code nor the GW code is returned to the caller.

### 8.3 Test data templates

Reuse the request shapes in `requests/` (`ezlynx.json`, `testcomp-EZLynx.json` for many vehicles/drivers;
`ITC.json` for a single vehicle/driver). Replace their coverage blocks with the ones below (do **not** carry
over the legacy `AD` / `UDB` / `UIM` entries).

**Single code (vehicle level):**
```json
"coverage": [ { "name": "COLL", "deductible": "1000" } ]
```

**All nine live codes on one vehicle (TC-48):**
```json
"coverage": [
  { "name": "BI",    "limit": "25/50" },
  { "name": "PD",    "limit": "25" },
  { "name": "MEDPM", "limit": "1000" },
  { "name": "COMP",  "deductible": "250" },
  { "name": "COLL",  "deductible": "1000" },
  { "name": "UMPD",  "limit": "25" },
  { "name": "UIMBI", "limit": "100/300" },
  { "name": "UMBI",  "limit": "50/100" },
  { "name": "TL",    "limit": "50" }
]
```

### 8.4 Single-code mapping (one case per code)

| ID | Test Case | Expected Result | Source | Priority | Status |
| --- | --- | --- | --- | --- | --- |
| TC-34 | Send `BI` only | `vehicles[0].coverage` contains `Bodily Injury` with the `limit` echoed; `msgStatus: "quoted"` | Coverage mapping (new) | High | Not Run |
| TC-35 | Send `PD` only | `Property Damage`, `limit` echoed | Coverage mapping (new) | High | Not Run |
| TC-36 | Send `MEDPM` only | `Medical Payments`, `limit` echoed | Coverage mapping (new) | High | Not Run |
| TC-37 | Send `COMP` only | `Comprehensive` carries `deductible` (not `limit`) | Coverage mapping (new) | High | Not Run |
| TC-38 | Send `COLL` only | `Collision` carries `deductible` (not `limit`) | Coverage mapping (new) | High | Not Run |
| TC-39 | Send `UMPD` only | `Uninsured Motorist Property Damage`, `limit` echoed | Coverage mapping (new) | High | Not Run |
| TC-40 | Send `UIMBI` only | `Underinsured Motorist Bodily Injury`, `limit` echoed | Coverage mapping (new) | High | Not Run |
| TC-41 | Send `UMBI` only | Maps to `Uninsured Motorist Bodily Injury` (name unconfirmed — see Q15) | Coverage mapping (new) | High | Not Run |
| TC-42 | Send `TL` only | Maps to `Towing & Labor` (name unconfirmed — see Q15) | Coverage mapping (new) | High | Not Run |
| TC-43 | Send `RREIM` (after enablement) | `Rental Reimbursement`, `limit` echoed — **blocked** until the code is released | Coverage mapping (new) | Medium | Not Run |

### 8.5 Value & field handling

| ID | Test Case | Expected Result | Source | Priority | Status |
| --- | --- | --- | --- | --- | --- |
| TC-44 | Limit-based codes (`BI`, `PD`, `MEDPM`, `UMPD`, `UIMBI`, `UMBI`, `TL`, `RREIM`) | Each renders `limit`; no `deductible` key present | Coverage mapping (new) | High | Not Run |
| TC-45 | Deductible-based codes (`COMP`, `COLL`) | Each renders `deductible`; no `limit` key present | Coverage mapping (new) | High | Not Run |
| TC-46 | Split-limit / combined formats (`"25/50"`, `"100/300"`, `"40/1200"`, `"25"`) | Value echoed unchanged, not reformatted or truncated | Coverage mapping (new) | Medium | Not Run |
| TC-47 | `value` used instead of `limit`/`deductible` | Either accepted and echoed, or rejected with a clear error — behaviour undefined in captures | Suggested | Low | Not Run |

### 8.6 Combinations

| ID | Test Case | Expected Result | Source | Priority | Status |
| --- | --- | --- | --- | --- | --- |
| TC-48 | All nine live codes on one vehicle | All nine canonical names returned, each with its correct `limit`/`deductible`; no drops or extras | Coverage mapping (new) | High | Not Run |
| TC-49 | Codes split across multiple vehicles (e.g. 3 codes on V1, 4 on V2, 2 on V3) | Each vehicle echoes **only its own** set — no bleed between vehicles | Coverage mapping (new) | High | Not Run |
| TC-50 | Multiple vehicles + multiple drivers in one request (see `requests/ezlynx.json`) | `vehicles[]` and `drivers[]` counts match the request; coverage mapping holds per vehicle | Coverage mapping (new) | High | Not Run |
| TC-51 | Same set sent in shuffled order | Same canonical set returned regardless of order | Coverage mapping (new) | Medium | Not Run |
| TC-52 | Same code duplicated on one vehicle | Deduped to a single entry, or rejected — behaviour undefined; assert one rule consistently | Suggested | Medium | Not Run |
| TC-53 | Vehicle-level codes only (no policy-level `coverages[]`) | Accepted; `vehicles[].coverage[]` populated with the mapped codes | Coverage mapping (new) | Medium | Not Run |

### 8.7 Negative & edge cases

| ID | Test Case | Expected Result | Source | Priority | Status |
| --- | --- | --- | --- | --- | --- |
| TC-54 | Code that is not in the map at all (e.g. `XYZ`) | Rejected with a populated `errorresponse`, or ignored per a defined rule (see Q16) | Suggested | High | Not Run |
| TC-55 | `RREIM` sent before enablement | Handled cleanly (rejected/ignored) — must not break the quote | Suggested | Medium | Not Run |
| TC-56 | Lower/mixed-case code (`bi`, `Coll`) | Case-insensitive mapping, or a clear validation error — one rule, asserted consistently | Suggested | Medium | Not Run |
| TC-57 | Coverage with no `limit`, `deductible`, or `value` | Validation error; no quote produced (or the documented default) | Suggested | Medium | Not Run |
| TC-58 | Empty `coverage[]` array | Accepted; `vehicles[].coverage[]` empty in the response | Suggested | Medium | Not Run |

### 8.8 Rater identity & downstream reporting

| ID | Test Case | Expected Result | Source | Priority | Status |
| --- | --- | --- | --- | --- | --- |
| TC-59 | Confie as a supported comp rater | Response `quoteInfo.parentQuote` equals `"Confie"`; `msgStatus: "quoted"` | Coverage mapping (new) | High | Not Run |
| TC-60 | Downstream reporting emits the **GW code** (`TPABICov`, …) for each mapped coverage | Report/extract carries the GW code, not the Confie short code — **blocked**: reporting spec/mechanism undefined (Q17) | Coverage mapping (new) | High | Not Run |
| TC-61 | Reporting round-trip for every mapped code | Every Confie code that maps is traceable end-to-end Confie code → GW code → report row | Coverage mapping (new) | Medium | Not Run |
| TC-62 | Cross-rater consistency (Confie vs PLRating) | Identical coverage sets produce identical canonical names in `coverage[]` | Suggested | Medium | Not Run |
| TC-63 | Regression — existing PLRating / ITC flows | Existing mappings unchanged; no new key or rename leaked into older rater responses | Suggested | High | Not Run |

**Note (from the workbook):** priority values are the author's suggested defaults (assumptions), editable in
the sheet. Suggested-addition rows are not yet implemented in the Postman script.
