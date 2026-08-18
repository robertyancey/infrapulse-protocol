# InfraPulse: Provider-Neutral Control and Verifiable Receipts for AI Agents

**Status:** Public technical paper  
**Version:** 1.0  
**Canonical service:** https://infrapulse.ai  
**Production contract:** machine-readable discovery and OpenAPI published by the canonical service

## Abstract

InfraPulse is a provider-neutral control plane for machine-to-machine AI services. It gives an agent a consistent way to discover a service, establish durable identity and governed connectivity, obtain a quote, authorize prepaid spend, execute work through an approved provider or native handler, and verify the outcome with an Ed25519-signed receipt.

The core sequence is:

**Discover → Connect → Quote → Authorize → Execute → Verify**

InfraPulse is not a foundation-model provider, bank, payment custodian, or ranking authority. It is an authorization, metering, routing, and evidence layer designed to reduce the amount of provider-specific trust, billing, identity, and audit logic that every agent integration otherwise has to rebuild.

## 1. The integration problem

Agents increasingly call APIs, models, tools, data services, and other agents. The useful work may be similar across providers, but the surrounding control plane is fragmented. Each integration can require different discovery conventions, credentials, billing rules, spending controls, retries, receipts, and recovery logic.

This fragmentation creates several practical problems:

- an agent may know that a service exists but not how to discover its callable production surface;
- a user may authorize a task without a clear, enforceable spending boundary;
- a provider may execute work before payment authorization is durable;
- a client may receive a success response without portable evidence of what was authorized and completed;
- identity, balances, receipts, and connection state may be lost when an agent process or model provider changes;
- integrations may confuse historical, sandbox, demo, or schema-only routes with production capability.

InfraPulse addresses these problems by separating the control plane from the underlying model or service provider.

## 2. Design goal

The design goal is not to make every provider identical. It is to give agents a small set of stable control-plane primitives around heterogeneous providers.

A client should be able to answer six questions before trusting a paid operation:

1. **What can this service do in production?**
2. **Who or what is acting?**
3. **What will this operation cost?**
4. **Was that spend authorized under the user's policy?**
5. **What actually happened?**
6. **Can the result be verified independently?**

InfraPulse maps those questions to discovery, identity/connectivity, quote, authorization, execution, and signed verification.

## 3. Production workflow

### 3.1 Discover

Agents begin with public, cacheable service metadata. Canonical discovery includes capabilities, capability states, pricing, agent metadata, OpenAPI, public verification keys, and protocol-specific cards.

Discovery is intentionally separate from paid execution. Public discovery must not create a billable operation.

### 3.2 Connect

InfraPulse supports durable identity and governed connections. A connection can bind the actor, requested scopes, approved scopes, protocol selection, spending limits, approval policy, callback health, expiration, rotation, suspension, and revocation.

The purpose is continuity: a process restart or provider change should not require the user to reconstruct the control relationship from an unstructured chat history.

### 3.3 Quote

A quote is the authoritative pre-execution cost statement for a supported billable operation. Clients should treat the quote—not marketing copy or an old fixed-price table—as the source of truth for the proposed operation.

### 3.4 Authorize

Paid work requires prepaid balance and a durable authorization. Policy can require explicit human approval. Authorization and execution are distinct so that a client can inspect cost and policy before work begins.

### 3.5 Execute

Execution is routed to an approved live provider or a native production handler. If InfraPulse cannot identify a valid production execution path, it fails before settlement rather than silently substituting a mock, sandbox, placeholder, or unrelated provider.

### 3.6 Verify

Successful production execution produces a signed receipt. Receipts use Ed25519 signatures and public JWKS discovery so a verifier can check authenticity independently of the process that performed the work.

The receipt layer is evidence, not a claim that an external provider is infallible. It records the control-plane facts InfraPulse can attest to: authorization, metering, execution identity, status, and related hashes or references.

## 4. Economic model

InfraPulse uses prepaid credits for its canonical paid control-plane flow.

Current public funding facts are machine-readable from `/.well-known/pricing`. At publication time, the production service uses 100 credits per USD and a minimum top-up of USD 5. The live pricing manifest remains authoritative if these values change.

Card checkout is created dynamically on the server and fulfilled only after verified Stripe payment ingress. Fixed generic Stripe Payment Links are not part of the production contract.

InfraPulse's application-level use of HTTP 402 expresses a deterministic payment-required boundary: an unfunded paid operation is refused before running billable work.

## 5. Provider neutrality

Provider neutrality means that the public InfraPulse contract does not require one model vendor, cloud, or payment rail to define the protocol.

It does **not** mean that every provider is interchangeable, that every provider supports every capability, or that InfraPulse certifies provider quality. The selected provider still determines its own models, limits, service-level behavior, and upstream economics.

InfraPulse's role is to make the surrounding control-plane lifecycle more consistent and auditable.

## 6. Interoperability surfaces

InfraPulse publishes a conventional HTTP/OpenAPI surface and agent-native interfaces.

- **OpenAPI:** the HTTP API description is published at `/openapi.json`.
- **A2A:** the production Agent Card is published at `/.well-known/agent-card.json`, with A2A interfaces exposed by the service.
- **MCP:** InfraPulse publishes MCP discovery metadata and an HTTP JSON-RPC MCP surface.
- **JWKS / Ed25519:** public verification material is published at `/.well-known/jwks.json`.

These interfaces are intended to complement one another. InfraPulse-specific economic and receipt semantics remain explicitly documented rather than being presented as requirements of MCP, A2A, or OpenAPI themselves.

## 7. Security model

The security model is fail-closed around privileged and paid behavior.

Core invariants include:

- secrets are not accepted through public discovery metadata;
- production routes are allowlisted from explicit production capability state;
- sandbox, mock, demo, simulation, schema-only, and unvalidated historical surfaces are not callable as production execution;
- paid work requires durable authorization and sufficient prepaid balance;
- Stripe webhook ingress is signature verified and processed idempotently;
- public receipt verification does not require possession of the signing private key;
- internal service hops are not intended as public interfaces;
- idempotency keys protect retried billable mutations from duplicate effects;
- a release is not considered production-ready solely because configuration values exist; deployment and recovery evidence are independently checked.

A fuller threat model is maintained in `SECURITY.md`.

## 8. Non-goals

InfraPulse does not claim to:

- replace the underlying model, API, cloud, or tool provider;
- custody customer funds as a bank or wallet;
- guarantee the correctness of an external provider's output;
- rank or certify providers as objectively "best";
- make unrestricted autonomous spending safe;
- turn an unvalidated route into production capability merely because code or a schema exists;
- define MCP, A2A, OpenAPI, Stripe, or HTTP standards.

## 9. Why signed receipts matter

Most API integrations end with a transient response and application logs. For autonomous or semi-autonomous systems, that is often insufficient. A user, provider, auditor, or later agent instance may need to know what was authorized and whether the result corresponds to the same control-plane event.

Signed receipts create a portable verification boundary. They do not eliminate the need to trust all external facts, but they allow InfraPulse-controlled facts to be checked independently and after the original process has ended.

This makes receipts useful for recovery, accounting, dispute investigation, policy audit, and multi-agent workflows.

## 10. Public sources of truth

For machines, the live service is authoritative:

- `https://infrapulse.ai/.well-known/capabilities`
- `https://infrapulse.ai/.well-known/capability-states.json`
- `https://infrapulse.ai/.well-known/pricing`
- `https://infrapulse.ai/.well-known/agent.json`
- `https://infrapulse.ai/.well-known/agent-card.json`
- `https://infrapulse.ai/.well-known/jwks.json`
- `https://infrapulse.ai/openapi.json`
- `https://infrapulse.ai/ai.txt`

For implementers, `PROTOCOL.md` defines the stable InfraPulse-specific lifecycle and `protocol.json` provides a compact machine-readable description.

## 11. Positioning in one sentence

**InfraPulse is a provider-neutral control plane that lets AI agents discover capabilities, establish governed connectivity, authorize prepaid work, execute through approved providers, and verify outcomes with signed receipts.**
