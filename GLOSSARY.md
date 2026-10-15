# Quote Comparator Testing

Testing the Trexis comparator quote API and the artifacts that pin down its response contract.

## Language

**Trexis**:
The insurance carrier whose comparator quote API is under test.
_Avoid_: the vendor, the client

**Comparator API**:
The Trexis service under test — `POST /trexis/comparator/quote` (`operationId: compositeQuoteAPI`) — which returns a single quote carrying multiple payment plans and options.
_Avoid_: quote service, rating engine

**Confie**:
An upstream rater that submits quote requests to the comparator; identified in a response by `parentQuote: "Confie"`.
_Avoid_: client, agency

**PLRating**:
An upstream rater, distinct from Confie; identified in a response by `parentQuote: "PLRating"`.

**parentQuote**:
The response field naming the upstream rater that originated the quote (`Confie`, `PLRating`).
_Avoid_: source, origin

**planPremium**:
A payment plan's premium, returned as a string such as `"844.00 usd"`. Equals the sum of that plan's `additionalQuotedScenario[].cost[].actualAmountBilling`.
_Avoid_: plan price, plan cost, premium

**totalPremium**:
An `options[]` entry's premium, a bare number. Never equal to `planPremium`; the difference between them is not a constant.
_Avoid_: total, option premium

**premium**:
The occasional top-level response string such as `"4538.00 usd"`. Its composition is undefined; do not conflate it with `planPremium` or `totalPremium`.
_Avoid_: grand total, overall premium

**Payment plan**:
An entry in `payment[]`, one of 16, keyed by `paymentPlanCd` (a plan name joined with a payment method such as EFT, RCC, or Direct Bill).
_Avoid_: payment option

**Payment option**:
An entry in `options[]`, one of 16, keyed by `paymentPlanName` plus `paymentType`.
_Avoid_: payment plan

**producerCode**:
The producer/agency code carried on the request and echoed at `policyInfo.producerCode.id`; required per state.
_Avoid_: producer, agency code

**msgStatus**:
The top-level quote status string; only `quoted` has been observed.

**quote link**:
The returned `quoteInfo.summaryURL` / `bridgeURL`, both of which embed the `quoteID`.

**PORTAL**:
Trexis's user-facing portal, where a quote is expected to be visible via its quote link.
_Avoid_: dashboard, UI

**Capture**:
A recorded response body checked into the repo (`itc`, `itc 2`, `PLRater`) used as the de-facto truth set for the contract.
_Avoid_: fixture, mock, sample
