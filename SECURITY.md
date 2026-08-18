# InfraPulse Security and Trust Model

**Status:** Public security model  
**Canonical service:** https://infrapulse.ai

## 1. Trust boundary

InfraPulse is a control-plane service. Its security objective is to make authorization, metering, routing, payment acknowledgement, and receipt evidence explicit and auditable without requiring clients to treat every service or model provider as the same trust domain.

InfraPulse does not claim that external model output is inherently correct. It provides evidence about the InfraPulse-controlled lifecycle around that output.

## 2. Primary security properties

### 2.1 Funds before paid work

A billable production action must have sufficient prepaid balance/budget and an applicable authorization before billable provider work begins.

### 2.2 Human policy remains authoritative

When a user's policy requires explicit human approval, a machine request does not silently override that requirement.

### 2.3 Production capability is explicit

Production execution is allowlisted. Historical code, schemas, tests, sandbox routes, mock routes, demonstrations, and simulations are not automatically production capabilities.

### 2.4 Payment fulfillment is verified

Stripe payment state is accepted through verified webhook ingress. Payment-credit mutation is idempotent. Client-submitted claims that a card payment succeeded are not sufficient evidence to credit an account.

### 2.5 Receipts are independently verifiable

Canonical receipts use Ed25519. Public keys are published through JWKS. The private signing key is not part of public discovery.

### 2.6 Retries do not create duplicate economic effects

Billable mutations use idempotency/deduplication so network retries can be made without intentionally multiplying the same logical charge or authorization.

### 2.7 Internal hops are not public APIs

Internal process-to-process listeners are kept on loopback or otherwise isolated. The documented public ingress is the supported client boundary.

## 3. Threats and controls

| Threat | Primary control |
|---|---|
| Agent discovers a stale/historical route and treats it as production | strict production route/capability allowlist |
| Unfunded request triggers paid upstream work | funds-before-work authorization boundary |
| Duplicate retry creates duplicate charge | durable idempotency and webhook deduplication |
| Forged Stripe event credits an account | Stripe signature verification, fail-closed webhook handling |
| Client forges a success claim | server-side verified payment ingress is authoritative |
| Receipt is modified after issuance | Ed25519 signature verification against JWKS |
| Receipt private key leaks through discovery | only public verification material is exposed |
| Model/provider switch destroys control relationship | durable identity, connection, ledger, and receipt state |
| Internal service port becomes externally reachable | loopback-only production internal listeners |
| Mock/demo route is mistaken for real execution | mock/sandbox/demo/simulation default denial in production |
| User policy is bypassed by autonomous client | scoped authorization, approval policy, spending limits |
| Secret is placed in machine-readable public metadata | public discovery contains no private credentials by design |

## 4. Receipt semantics

A signed receipt is cryptographic evidence that InfraPulse attested to the fields included in that receipt. Depending on the operation, those fields can represent identifiers, authorization references, metering facts, status, timestamps, hashes, and other lifecycle metadata.

A valid receipt does **not** prove that:

- every factual statement produced by an external model is true;
- an external provider had no outage or defect;
- a legal/compliance requirement outside the receipt's scope was satisfied;
- an off-platform event occurred unless that event is represented by evidence InfraPulse actually verified.

Consumers should interpret receipts narrowly according to their signed fields.

## 5. Payment model

InfraPulse's canonical card funding flow uses server-created Stripe Checkout and verified Stripe webhook fulfillment. InfraPulse application forms do not collect raw card data.

The live `/.well-known/pricing` document is authoritative for current credit conversion and top-up limits.

InfraPulse describes itself as non-custodial in the sense that it is not a general-purpose bank or crypto wallet. External payment rails perform the monetary transfer; InfraPulse maintains the application credit/authorization ledger used for its services.

## 6. Authentication and secrets

Production secrets have no safe source-code defaults. Required signing, payment, database, administrative, and provider credentials must come from protected deployment configuration.

Clients must not send passwords, seed phrases, signing private keys, or unrelated provider secrets to public discovery endpoints.

Where bearer/API credentials are issued, they should be scoped, rotatable, revocable, and treated as secrets by the client.

## 7. Availability and durability

Durable identity, credits, receipts, repository state, and operation journals are designed to survive ordinary process restarts and deployments through persistent storage. This is not a guarantee against every catastrophic event; production operations should maintain tested backups and restore procedures.

Release readiness therefore includes database recovery evidence in addition to application health checks.

## 8. Release integrity

InfraPulse distinguishes configuration from evidence. A deployment is not considered proven merely because environment variables are present.

Production release validation includes, as applicable:

- exact source/deployment identity;
- production build and startup;
- machine discovery and OpenAPI contract checks;
- payment and billing contract checks;
- receipt/JWKS verification;
- provider execution evidence;
- database backup/restore evidence;
- security and source-integrity checks;
- load/concurrency checks.

## 9. Reporting and verification

Canonical machine-readable security-relevant resources include:

- `https://infrapulse.ai/.well-known/jwks.json`
- `https://infrapulse.ai/.well-known/capabilities`
- `https://infrapulse.ai/.well-known/capability-states.json`
- `https://infrapulse.ai/openapi.json`
- `https://infrapulse.ai/.well-known/security.txt`

Security reports should be sent through the contact published by `/.well-known/security.txt`.
