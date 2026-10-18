# PROJECT.md — Verifiable Interaction Engine (VIE)

**Status:** Greenfield, M0 architecture research. **Primary language:** TypeScript.  
**Reference application:** `community.rbgtk.dev`  
**SSI infrastructure:** `id.rbgtk.dev` (walt.id reference integration)

## Purpose

Build a reusable, framework-independent engine for verifiable user interactions. Its first client is a Reddit-like community application with DID/wallet-based onboarding, verifiable credentials for scoped permissions, and user-signed posts, comments and moderation events. Build working vertical slices; avoid speculative infrastructure.

## MVP identity policy — approved scope change (2026-10-18)

**Identity continuity across key rotation or wallet migration is NOT an MVP requirement.** The engine must not automatically associate a new key or DID with a previous identity, transfer roles or credentials, or claim old content under the new identity. Users are responsible for managing, demonstrating, or arranging continuity themselves outside the MVP. A new DID or signing key is treated as a separate identity/authority unless an explicit, independently verified authorization mechanism is later designed and approved.

No automatic account recovery, key rotation linking, wallet migration linking, historical DID-state reconstruction service, or continuity proof protocol is required for MVP. These are possible post-MVP extensions, not acceptance criteria or M0 blockers. Do not build an account-linking shortcut based on matching names, emails, self-assertions, or possession of a new key.

**Still required:** User private keys remain user-controlled; optional custodial wallet is separate and not a dependency. Verify the signature and applicable key authority for each action, check credential issuer trust/status/scope and authorization independently, and retain the evidence available for the platform's acceptance decision. Exported signed events must be independently verifiable **when sufficient key evidence accompanies them**. When historical evidence is unavailable, return an explicit `indeterminate` result rather than falsely asserting authorship. The platform must not forge user signatures.

## Architectural layers

1. **SSI infrastructure:** walt.id issuer/verifier, OID4VCI/OID4VP, optional wallet; externally compatible wallets wherever practical. Verify actual protocol versions and endpoints before implementation.
2. **Verifiable Interaction Engine:** TypeScript packages for event schema, signature validation, key evidence, credential trust, scoped authorization, acceptance decisions and export. No direct dependency on NestJS, Nuxt, PostgreSQL or walt.id.
3. **Community application:** community ownership, subscriptions, posts, comments, moderation, profiles and feeds. Keep community domain rules outside the generic engine.

## Preferred stack

TypeScript strict mode; pnpm monorepo; NestJS API; Nuxt/Vue frontend; PostgreSQL and Drizzle; Vitest and Playwright; OpenAPI; Docker Compose for development. Use established JOSE/VC implementations and versioned deterministic signed-event formats. Research and record standards choices in ADRs.

## Credentials and signed actions

Initial credentials: `UserCredential`, community-scoped `OwnerCredential`, `SubscriberCredential`, `AdminCredential`. Validate issuer, holder binding, expiration, status, scope, delegation and policy as appropriate. Credentials prove authority, not content authorship.

Initial signed events: community creation/updates, subscriptions, moderator appointments/revocations, post/comment creation/edit/deletion, and moderation actions. Signed event envelopes should contain schema version, signing domain, event type/id, actor/key reference, resource, timestamp, relevant parent/previous references and event-specific payload. Define exact signed bytes, replay controls and signature verification. Preserve original signed events and distinguish user intent from server acceptance decisions.

## Milestones

- **M0 — Research and architecture:** Verify walt.id APIs, OID4VCI/OID4VP compatibility, external wallet arbitrary-content signing, DID-method verification limits, signing envelope, authorization and threat model. Produce ADR proposals, compatibility matrix, risk register and first vertical-slice backlog. **Do not block M0 on key-rotation or cross-key continuity research.** Stop for human approval.
- **M1 — Authentication:** BYOD registration, credential issuance, OID4VP presentation, session establishment and holder-binding checks for at least one documented compatible wallet.
- **M2 — Signed content:** Client-controlled signing, versioned event verification and persistence, tampering/replay tests, export and independent verification with explicit key-evidence limitations.
- **M3 — Credential authorization:** Community-scoped subscriber/owner/admin permissions, revocation, delegation and decision evidence. No implicit permission transfer to new keys/DIDs.
- **M4 — Community MVP:** Communities, subscriptions, posts, comments, edits/deletes and moderation through the engine.
- **M5 — Hardening and interoperability:** External-wallet tests, negative/security tests, rate limiting, reverse-proxy allowlists, privacy/retention documentation and reproducible deployment.

Key rotation, account linking, wallet migration continuity, DIDComm messaging, federation, blockchain, tokenomics, decentralized storage and native mobile applications are outside MVP scope unless explicitly approved.

## Planned endpoints

`https://id.rbgtk.dev/issuer-api`, `https://id.rbgtk.dev/verifier-api`, `https://id.rbgtk.dev/wallet`, `https://id.rbgtk.dev/wallet-api`, `https://community.rbgtk.dev`. These are proposed public paths, not guaranteed upstream routes. Use default-deny ingress with necessary public protocol endpoints, and protect privileged APIs in the application as well as the reverse proxy.

## Agent governance

Read `AGENTS.md` and role-specific `.agents/*.md`. Project Manager plans and coordinates; Developer implements; Tester independently verifies; Security Reviewer assesses risks. Human Product Owner approves significant architecture, security exceptions and milestone transitions. No fabricated approvals or unexecuted test results. Do not deploy or modify production without explicit permission.

## First vertical slice

External DID/wallet → credential issuance → OID4VP login → user-controlled signing of one comment → signature and authorization validation → published comment → export and independent verification with stated evidence limits. No key rotation or identity-linking flow is needed.
