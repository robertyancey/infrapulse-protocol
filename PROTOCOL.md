# InfraPulse Protocol Specification

**Specification:** `infrapulse-control-plane/1.0`  
**Status:** Production-oriented public specification  
**Canonical origin:** `https://infrapulse.ai`

## 1. Scope

This document defines the InfraPulse-specific control-plane lifecycle around discovery, durable identity/connectivity, prepaid authorization, execution, settlement, and signed receipt verification.

It does not redefine HTTP, OpenAPI, MCP, A2A, Stripe, OAuth, or JOSE. Where InfraPulse uses those technologies, their own specifications remain authoritative.

The key words **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, and **MAY** are used in the usual normative sense for this InfraPulse specification.

## 2. Canonical lifecycle

A production client integrates through the following logical sequence:

`DISCOVER → CONNECT → QUOTE → AUTHORIZE → EXECUTE → VERIFY`

A client MAY omit a step only when the live capability/OpenAPI contract explicitly defines an equivalent safe path.

## 3. Discovery

### 3.1 Production sources

The canonical service MUST publish machine-readable production metadata over HTTPS.

Primary discovery resources are:

- `GET /.well-known/capabilities`
- `GET /.well-known/capability-states.json`
- `GET /.well-known/pricing`
- `GET /.well-known/agent.json`
- `GET /.well-known/agent-card.json`
- `GET /.well-known/jwks.json`
- `GET /openapi.json`
- `GET /ai.txt`

The live response is authoritative over examples embedded in documentation.

### 3.2 Production-state rule

The existence of source code, a schema, a historical endpoint, or a sandbox implementation MUST NOT by itself imply production capability.

Production capability is determined by the service's enforced production capability/route contract. Unknown historical or unvalidated routes MAY be returned as not found; sandbox/mock/demo/simulation routes MUST NOT execute as production work.

### 3.3 Cacheability

Clients SHOULD cache public discovery according to the response cache headers and SHOULD NOT call discovery endpoints in tight execution loops.

## 4. Identity and governed connectivity

InfraPulse supports durable machine identity and governed connections.

A durable connection MAY contain:

- actor identity;
- requested and approved scopes;
- protocol selection and fallbacks;
- human-approval policy;
- prepaid spending limits;
- callback/heartbeat state;
- expiry;
- token/key rotation state;
- suspension or revocation state.

Production mutations MUST authenticate the caller according to the live OpenAPI/security contract and MUST fail closed when required authorization or signing prerequisites are absent.

## 5. Quote

A quote describes the proposed billable action before authorization.

For an operation that requires quoting:

- obtaining the quote MUST NOT itself execute the paid work;
- the quote MUST identify the action sufficiently for later authorization;
- the quoted cost or cost basis MUST be expressed in the service's canonical billing units;
- the live quote MUST take precedence over stale examples or historical fixed-price tables.

The canonical quote entry point is advertised by the live OpenAPI and pricing metadata.

## 6. Authorization

### 6.1 Funds-before-work

InfraPulse production paid execution is prepaid.

A paid operation MUST NOT begin billable execution unless sufficient prepaid balance/budget and a valid authorization exist.

When funding or authorization is insufficient, the service SHOULD return deterministic payment-required behavior before billable provider work begins.

### 6.2 Human policy

If the governing policy requires human approval, the service MUST NOT treat a machine request alone as final approval. The operation remains pending or is refused until the approval requirement is satisfied.

### 6.3 Idempotency

Billable mutation entry points SHOULD require or honor an `Idempotency-Key`. Repeating the same accepted idempotency key MUST NOT create a second economic effect for the same logical operation.

## 7. Funding

### 7.1 Credits

The canonical pricing resource declares:

- currency;
- credits per USD;
- minimum top-up;
- checkout entry point;
- other current economic metadata.

Current numerical values in this document are informational. `/.well-known/pricing` is authoritative.

### 7.2 Stripe checkout

For card funding, InfraPulse creates Stripe Checkout Sessions dynamically. Generic fixed-denomination Stripe Payment Links are not part of the production control-plane contract.

The service MUST NOT credit a payment merely because a client claims that payment occurred.

Production Stripe credit fulfillment MUST be based on verified Stripe ingress and MUST be idempotent.

Card data remains on Stripe-hosted payment surfaces rather than being collected by InfraPulse application fields.

## 8. Execution

A production execution request MUST resolve to an allowlisted production capability and an approved native handler or configured live provider.

InfraPulse MUST NOT silently substitute a mock, sandbox, demonstration, placeholder, or unrelated provider when no valid production execution route exists.

If the selected production path cannot safely execute, the service MUST fail before representing the operation as completed.

Execution MAY be asynchronous. When asynchronous, a durable operation identifier SHOULD allow the client to retrieve state without resubmitting the economic operation.

## 9. Settlement

Authorization and final settlement are distinct concepts.

When the implementation reserves budget before execution, it SHOULD settle actual allowed cost and release unused authorization according to the live product contract.

Failure handling MUST avoid converting a failed or unexecuted operation into an unrecoverable successful charge unless the product contract explicitly defines a non-refundable consumed resource.

## 10. Receipts

### 10.1 Requirement

Canonical successful production execution SHOULD emit a verifiable receipt for the economic/control-plane event.

### 10.2 Signature

InfraPulse canonical receipts use Ed25519 signatures. Public verification material is exposed through JWKS at:

`GET /.well-known/jwks.json`

Clients MUST NOT need the signing private key to verify a receipt.

### 10.3 Verification

The canonical receipt verification interface is advertised by OpenAPI and discovery metadata. A verifier SHOULD validate the signature/key identifier and the receipt's integrity before treating it as InfraPulse-authenticated evidence.

A receipt proves only the fields that are cryptographically bound to it. It MUST NOT be interpreted as a guarantee that every external-world fact or provider-generated answer is correct.

## 11. Protocol interfaces

### 11.1 HTTP/OpenAPI

`/openapi.json` is the canonical machine-readable HTTP interface description.

### 11.2 A2A

InfraPulse publishes an A2A Agent Card at the current well-known Agent Card path and exposes its supported A2A interfaces as described by live discovery/OpenAPI.

InfraPulse-specific billing and receipt semantics are extensions of the service behavior; they MUST NOT be represented as requirements imposed by the A2A specification itself.

### 11.3 MCP

InfraPulse exposes an MCP HTTP interface and MCP discovery metadata. MCP protocol semantics remain governed by the MCP specification; InfraPulse-specific prepaid authorization and signed receipt behavior remain service policy around MCP-mediated actions.

## 12. Security invariants

Production deployments MUST:

- keep signing/payment credentials out of public metadata;
- verify Stripe webhook signatures before economic mutation;
- preserve idempotency/deduplication across durable payment processing;
- enforce production route and intent boundaries;
- fail closed when required production configuration/evidence is absent;
- avoid exposing internal-only application hops as public interfaces;
- publish only public verification keys, never receipt signing private keys;
- prevent mock/sandbox/demo execution from being mislabeled as production.

## 13. Non-goals

This specification does not define:

- model output correctness;
- provider rankings or endorsements;
- unrestricted autonomous spending;
- bank, wallet, escrow-custody, or money-transmission semantics;
- a replacement for provider-specific service terms;
- a claim that all AI systems expose identical capabilities.

## 14. Compatibility and evolution

Clients SHOULD prefer feature/capability discovery over assumptions derived from a document version.

InfraPulse MAY add optional fields or endpoints without changing this lifecycle. A breaking change to the meaning of a required lifecycle primitive SHOULD receive a new InfraPulse protocol version.

Deprecated historical surfaces SHOULD identify the canonical replacement when practical.

## 15. Canonical machine references

- Capabilities: `https://infrapulse.ai/.well-known/capabilities`
- Capability states: `https://infrapulse.ai/.well-known/capability-states.json`
- Pricing: `https://infrapulse.ai/.well-known/pricing`
- InfraPulse manifest: `https://infrapulse.ai/.well-known/agent.json`
- A2A Agent Card: `https://infrapulse.ai/.well-known/agent-card.json`
- JWKS: `https://infrapulse.ai/.well-known/jwks.json`
- OpenAPI: `https://infrapulse.ai/openapi.json`
- AI-readable overview: `https://infrapulse.ai/ai.txt`
