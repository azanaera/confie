# Quote API — Test Case Catalog

Transcribed from `quote_api_test_cases.xlsx` (sheets **Test Cases** and **Summary**). The workbook
remains the artifact of record; this file exists so the TC IDs cited by the test plans resolve
in-repo.

**Columns:** ID · Section · Test Case · Expected Result · Source · Priority · Status

**Conventions carried from the workbook:**

- **Source** — `Existing script` (already in the Postman collection) or `Suggested` (proposed, not yet implemented).
- **Priority** — the workbook's suggested defaults (assumptions), not yet ratified.
- **Status** — all `Not Run` as of transcription.
- `Actual Result`, `Defect Link`, and `Notes` columns exist in the workbook but are empty; omitted here.

## Summary (from the workbook's Summary sheet)

| Metric | Value |
| --- | --- |
| Total cases | 30 |
| Priority — High | 16 |
| Priority — Medium | 12 |
| Priority — Low | 2 |
| Source — Existing script | 20 |
| Source — Suggested | 10 |
| Status — Not Run | 30 |
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

**Note (from the workbook):** priority values are the author's suggested defaults (assumptions),
editable in the sheet. Suggested-addition rows are not yet implemented in the Postman script.
