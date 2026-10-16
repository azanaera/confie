# Quote API — Coverage Mapping Test Case Catalog

Test cases for the **new requirement**: support **Confie** as an additional comp rater and support the
**downstream reporting** requirement that depends on a coverage-code mapping between Confie and GW.

**Columns:** ID · Section · Test Case · Expected Result · Source · Priority · Status (same conventions as
[`test-cases.md`](./test-cases.md): `Source` = `Existing script` / `Suggested` / `Coverage mapping (new)`;
`Status` = `Not Run` until executed). IDs use a `CM-` prefix so they don't collide with the `TC-` series;
renumber to `TC-34+` if folded into the main catalog.

> **Truth set.** Expected canonical names come from the three captures (ADR-0001) — `itc response body.json`,
> `itc 2 response body.json` (`parentQuote: "Confie"`) and `PLRater response body.json`
> (`parentQuote: "PLRating"`). Names marked **inferred** below are not observed in any capture yet.

## 1. Requirement

- Confie is added as an upstream comp rater; its quotes must appear with `quoteInfo.parentQuote: "Confie"`.
- Confie sends coverages using **its own short codes**; the platform must map them to the **GW codes** used
  for rating and downstream reporting.
- The mapping must hold for every code, for valid combinations, and must fail cleanly on unknown codes.

## 2. The code map

The pairs below are the requirement as supplied. "Canonical name" is what the response actually returns in
`coverage[].name`.

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

**Codes present in the sample requests but absent from the supplied map (must be confirmed):**

| Confie code | Observed canonical name | Where it appears | Open |
| --- | --- | --- | --- |
| `AD` | Accidental Death | policy-level `coverages[]`, all `requests/*.json`; top-level `coverage[]` in `itc` / `PLRater` | GW code unknown (CM-22) |
| `UDB` | — | vehicle-level in `requests/ezlynx.json`, `requests/testcomp-EZLynx.json` | code and GW mapping unknown (CM-22) |
| `UIM` | — | vehicle-level in `requests/testcomp-EZLynx.json` | overlaps `UIMBI`/`UMBI`; disambiguation needed (CM-22) |

## 3. How coverage travels

- **Request — policy level:** `policy.coverages[]` (plural), e.g. `{"name": "AD", "limit": "5000"}`.
- **Request — vehicle level:** `vehicles[].coverage[]` (singular), e.g. `{"name": "COMP", "deductible": "250"}`.
- **Item shape** (`CoverageDTO`): `name` plus value as one of `limit`, `deductible`, or `value` (spec allows all three).
- **Response:** top-level `coverage[]` (policy-level) and `vehicles[].coverage[]` (vehicle-level). Names are
  **normalized to canonical display names** — the Confie short code and the GW code are not returned to the caller.

## 4. Test data templates

Requests reuse the shapes in `requests/` (`ezlynx.json`, `testcomp-EZLynx.json` for many vehicles/drivers;
`ITC.json` for a single vehicle/driver). The coverage blocks to vary:

**Single code (vehicle level):**
```json
"coverage": [ { "name": "COLL", "deductible": "1000" } ]
```

**All live codes on one vehicle (CM-16):**
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

**Policy-level code (CM-14):**
```json
"coverages": [ { "name": "AD", "limit": "5000" } ]
```

## 5. Test cases

### A. Single-code mapping (one case per code)

Each case sends a request whose only coverage is the code named, then asserts the response.

| ID | Test Case | Expected Result | Source | Priority | Status |
| --- | --- | --- | --- | --- | --- |
| CM-01 | Send `BI` only | Response `vehicles[0].coverage` contains `Bodily Injury` with the `limit` echoed; request accepted (`msgStatus: "quoted"`) | Coverage mapping (new) | High | Not Run |
| CM-02 | Send `PD` only | `Property Damage`, `limit` echoed | Coverage mapping (new) | High | Not Run |
| CM-03 | Send `MEDPM` only | `Medical Payments`, `limit` echoed | Coverage mapping (new) | High | Not Run |
| CM-04 | Send `COMP` only | `Comprehensive` carries `deductible` (not `limit`) | Coverage mapping (new) | High | Not Run |
| CM-05 | Send `COLL` only | `Collision` carries `deductible` (not `limit`) | Coverage mapping (new) | High | Not Run |
| CM-06 | Send `UMPD` only | `Uninsured Motorist Property Damage`, `limit` echoed | Coverage mapping (new) | High | Not Run |
| CM-07 | Send `UIMBI` only | `Underinsured Motorist Bodily Injury`, `limit` echoed | Coverage mapping (new) | High | Not Run |
| CM-08 | Send `UMBI` only | Maps to `Uninsured Motorist Bodily Injury` (name unconfirmed — see CM-22/open items) | Coverage mapping (new) | High | Not Run |
| CM-09 | Send `TL` only | Maps to `Towing & Labor` (name unconfirmed) | Coverage mapping (new) | High | Not Run |
| CM-10 | Send `RREIM` (after enablement) | `Rental Reimbursement`, `limit` echoed — **blocked** until the code is released | Coverage mapping (new) | Medium | Not Run |

### B. Value & field handling

| ID | Test Case | Expected Result | Source | Priority | Status |
| --- | --- | --- | --- | --- | --- |
| CM-11 | Limit-based codes (`BI`, `PD`, `MEDPM`, `UMPD`, `UIMBI`, `UMBI`, `TL`, `RREIM`) | Each renders `limit`; no `deductible` key present | Coverage mapping (new) | High | Not Run |
| CM-12 | Deductible-based codes (`COMP`, `COLL`) | Each renders `deductible`; no `limit` key present | Coverage mapping (new) | High | Not Run |
| CM-13 | Split-limit / combined formats (`"25/50"`, `"100/300"`, `"40/1200"`, `"25"`) | Value echoed unchanged, not reformatted or truncated | Coverage mapping (new) | Medium | Not Run |
| CM-14 | `AD` at policy level (`policy.coverages[]`) | Response top-level `coverage[]` contains `Accidental Death` with `limit "5,000"` (as captured) | Coverage mapping (new) | Medium | Not Run |
| CM-15 | `value` used instead of `limit`/`deductible` | Either accepted and echoed, or rejected with a clear error — behaviour undefined in captures | Suggested | Low | Not Run |

### C. Combinations

| ID | Test Case | Expected Result | Source | Priority | Status |
| --- | --- | --- | --- | --- | --- |
| CM-16 | All nine live codes on one vehicle | All nine canonical names returned, each with its correct `limit`/`deductible`; no drops or extras | Coverage mapping (new) | High | Not Run |
| CM-17 | Codes split across multiple vehicles (e.g. 3 codes on V1, 4 on V2, 2 on V3) | Each vehicle echoes **only its own** set — no bleed between vehicles | Coverage mapping (new) | High | Not Run |
| CM-18 | Multiple vehicles + multiple drivers in one request (see `requests/ezlynx.json`) | `vehicles[]` and `drivers[]` counts match the request; coverage mapping holds per vehicle | Coverage mapping (new) | High | Not Run |
| CM-19 | Same set sent in shuffled order | Same canonical set returned regardless of order | Coverage mapping (new) | Medium | Not Run |
| CM-20 | Same code duplicated on one vehicle | Deduped to a single entry, or rejected — behaviour undefined; assert one rule consistently | Suggested | Medium | Not Run |
| CM-21 | Policy-level and vehicle-level codes together (`AD` + vehicle codes) | Both surfaces populated; policy-level code not duplicated onto vehicles | Coverage mapping (new) | Medium | Not Run |

### D. Negative & edge cases

| ID | Test Case | Expected Result | Source | Priority | Status |
| --- | --- | --- | --- | --- | --- |
| CM-22 | Legacy codes `AD`, `UDB`, `UIM` (used in samples but not in the supplied Confie map) | Each maps to a defined canonical name, or is explicitly rejected — no silent drop | Suggested | High | Not Run |
| CM-23 | Code that is not in the map at all (e.g. `XYZ`) | Rejected with a populated `errorresponse`, or ignored per a defined rule | Suggested | High | Not Run |
| CM-24 | `RREIM` sent before enablement | Handled cleanly (rejected/ignored) — must not break the quote | Suggested | Medium | Not Run |
| CM-25 | Lower/mixed-case code (`bi`, `Coll`) | Case-insensitive mapping, or a clear validation error — one rule, asserted consistently | Suggested | Medium | Not Run |
| CM-26 | Coverage with no `limit`, `deductible`, or `value` | Validation error; no quote produced (or the documented default) | Suggested | Medium | Not Run |
| CM-27 | Empty `coverage` / `coverages` arrays | Accepted; response coverage list is empty (`[]`), matching `itc 2`'s policy-level `coverage: []` | Suggested | Medium | Not Run |

### E. Rater identity & downstream reporting

| ID | Test Case | Expected Result | Source | Priority | Status |
| --- | --- | --- | --- | --- | --- |
| CM-28 | Confie as a supported comp rater | Response `quoteInfo.parentQuote` equals `"Confie"`; `msgStatus: "quoted"` | Coverage mapping (new) | High | Not Run |
| CM-29 | Downstream reporting emits the **GW code** (`TPABICov`, …) for each mapped coverage | Report/extract carries the GW code, not the Confie short code — **blocked**: reporting spec/mechanism undefined | Coverage mapping (new) | High | Not Run |
| CM-30 | Reporting round-trip for every mapped code | Every Confie code that maps is traceable end-to-end Confie code → GW code → report row | Coverage mapping (new) | Medium | Not Run |
| CM-31 | Cross-rater consistency (Confie vs PLRating) | Identical coverage sets produce identical canonical names in `coverage[]` | Suggested | Medium | Not Run |
| CM-32 | Regression — existing PLRating / ITC flows | Existing mappings unchanged; no new key or rename leaked into older rater responses | Suggested | High | Not Run |

## 6. Summary

| Metric | Value |
| --- | --- |
| Total cases | 32 |
| Priority — High | 19 |
| Priority — Medium | 12 |
| Priority — Low | 1 |
| Source — Coverage mapping (new) | 22 |
| Source — Suggested | 10 |
| Status — Not Run | 32 |

## 7. Open items (fold into §16 of `API Test Plan v0.5.md`)

- **Q14 — Confie codes `AD`, `UDB`, `UIM`.** The supplied Confie map omits them, yet they appear in the
  sample requests (`AD` maps to `Accidental Death` in captures; `UDB`/`UIM` are unmapped). Are they Confie
  codes, and what are their GW codes? _Owner: Trexis / Client._
- **Q15 — Canonical names for `UMBI`, `TL`.** No capture shows `Uninsured Motorist Bodily Injury` (uninsured)
  or `Towing & Labor`; confirm the exact response names. _Owner: Client._
- **Q16 — Unknown-code behaviour.** Reject or ignore? What does `errorresponse` contain (ties to Q4)? _Owner: Client._
- **Q17 — Downstream reporting contract.** Where do GW coverage codes surface (report/extract), and what is
  the acceptance criterion that proves the mapping? Gates CM-29/CM-30. _Owner: Client + Dev._
- **Q18 — `RREIM` release.** Confirm the enablement date so CM-10/CM-24 can be executed. _Owner: Dev._
