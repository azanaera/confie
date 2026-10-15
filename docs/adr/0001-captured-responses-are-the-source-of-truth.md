# Captured responses are the source of truth for the quote contract

The three recorded response bodies (`itc`, `itc 2`, `PLRater`) contradict the OpenAPI spec
`ComparatorSchema.json`: the spec declares `payment` as an object while the API returns an array,
and the spec omits fields the API actually returns (`msgStatus`, `options`, `quoteID`,
`parentQuote`, `errorresponse`), while declaring some the API never returns (`producerInfo`,
`location`). We decided the **captured responses are authoritative** and the spec must be
regenerated to match reality — not the reverse — because the API is live and returning these shapes
in production-promising traffic, so the artifact that is provably wrong is the spec.

## Status

accepted

## Considered Options

- **Treat the spec as the contract** and log the responses as non-conformant — rejected: it would
  fail every schema-level assertion (TC-04) against a spec that is demonstrably out of date.
- **Negotiate a new contract** and treat both as provisional — rejected: the responses are already
  the working truth for the existing test cases (TC-02/TC-03 assert on fields the spec lacks).

## Consequences

- Schema/contract testing moves from *Planned* to *Now* — a contract exists, but it must be
  reconciled before assertions can pass.
- TC-04 (response validates against the schema) cannot pass until `ComparatorSchema.json` is regenerated.
- The spec's OAuth `authorizationCode` flow is treated as stale in the same way; testing standardises
  on the client-credentials flow the Postman collection actually uses.
