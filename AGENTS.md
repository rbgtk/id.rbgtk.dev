# AGENTS.md — Verifiable Interaction Engine

## Mission and principles

Build a reusable, application-independent TypeScript engine for cryptographically verifiable interactions using SSI, DIDs and verifiable credentials. `community.rbgtk.dev` is the first reference application; `id.rbgtk.dev` hosts the walt.id-based SSI infrastructure.

Identity belongs to the user. User private signing keys remain under user control; the engine must never require custody of them. An optional custodial wallet must remain separate and its custody model explicit. Credentials establish authority; signatures establish cryptographic authorship; application policies determine authorization. Signed content must be independently verifiable. MVP identity continuity is limited to the identity and signing keys presented for an action. The engine does not link old and new identities across key rotation or wallet migration; users manage any continuity claims themselves. A new DID/key is not automatically treated as the old identity.

## Roles and authority

The human Product Owner is the final authority on product scope, architectural decisions, security exceptions and releases.

| Agent | Owns | Must not |
|---|---|---|
| Project Manager | Roadmap, issues, dependencies, coordination, acceptance criteria, progress | Implement code, override failed tests or security findings, approve own work |
| Developer | Architecture proposals, code, migrations, unit tests, technical documentation | Self-approve security-sensitive changes or weaken verification to pass tests |
| Tester | Test plans, integration and regression tests, defect reports, interoperability validation | Quietly alter production code or claim unexecuted tests passed |
| Security Reviewer | Threat model, cryptographic and authorization review, findings, remediation requirements | Weaken security requirements to unblock delivery |

The Project Manager coordinates the three specialist agents. They may collaborate directly but must report changes in scope, priority or architecture. No agent may impersonate another agent or fabricate independent approval. If agent isolation is unavailable, disclose it.

## Issue workflow

1. PM defines the issue's objective, scope, exclusions, acceptance criteria, dependencies, deliverables, test requirements and security considerations.
2. Security Reviewer identifies relevant threats and pre-implementation requirements.
3. Developer implements on an isolated branch or worktree and writes relevant unit tests.
4. Tester independently executes functional, integration, negative and regression tests.
5. Security Reviewer evaluates the completed change and documents findings.
6. PM verifies evidence and marks the change ready for required approval.
7. Human Product Owner approves reserved decisions and milestone completion.

Failed tests or blocking security findings return the issue to implementation. Never claim an issue is done without executed evidence, completed required reviews and approvals.

## Definition of Done

- All acceptance criteria satisfied.
- Relevant automated tests executed and passed; exact commands and revision reported.
- Type checking and linting passed.
- Contracts, schemas, migrations and documentation updated as applicable.
- Required security review completed and no unresolved blocking findings.
- Changes merged through the approved review process.

## Architecture boundaries

**SSI infrastructure:** walt.id is the initial reference provider for credential issuance, verification, OID4VCI, OID4VP and supported wallet/DID functions. Confirm actual versioned API contracts and metadata requirements; do not assume compatibility between wallet or API generations.

**Verifiable Interaction Engine:** Framework-independent TypeScript for signed-event verification, signing-key authority, credential-based authorization, delegation, acceptance evidence, event verification and export within the evidence available for each event. It must not directly depend on NestJS, Nuxt, PostgreSQL or walt.id. Use integration interfaces where justified by actual requirements, not speculative abstractions.

**Community application:** Owns communities, subscriptions, threads, comments, profiles, feeds, moderation UI and other Reddit-like application concepts. These concepts must not be baked into the reusable engine.

Preferred stack: TypeScript strict mode, Node.js, pnpm workspaces, NestJS, Nuxt/Vue, PostgreSQL, Drizzle, Vitest, Playwright, OpenAPI and Docker Compose initially. Verify dependency versions and compatibility before adoption. Avoid creating empty packages or unnecessary scaffolding.

Provisional layout:

```text
vie/
├── AGENTS.md
├── PROJECT.md
├── .agents/
│   ├── project-manager.md
│   ├── developer.md
│   ├── tester.md
│   └── security-reviewer.md
├── apps/{community,api,verifier-cli}/
├── packages/{engine,identity,authorization,credentials,signing,schemas}/
├── infrastructure/
├── docs/{architecture,decisions,security}/
└── tests/
```

## Cryptographic and authorization invariants

- Never possess or require users' private signing keys in the engine.
- Signed actions must use a documented versioned format, deterministic signed bytes, algorithm allowlists, domain separation, replay protection and reliable key identification.
- Verify signatures and key authority; a valid signature alone does not prove real-world identity or permission.
- Evaluate current credential validity, status, scope, delegation and application policy separately from authorship.
- Preserve the verification inputs and authorization decision evidence available at acceptance time. Do not assert historical signing authority from a current DID document alone.
- Do not require key rotation, wallet migration, account linking, or continuity proofs in the MVP. Verify historical signatures only when sufficient trustworthy key evidence is available; otherwise report verification as indeterminate, not valid. Never automatically link distinct DIDs or keys.
- Use established standards and cryptographic libraries. Do not invent primitives or silently bypass checks.
- Preserve signed events without silently modifying their signed contents. Distinguish signed user intent from server acceptance decisions.
- Independently verifiable exports must not rely on trusting the community server's assertion of authorship.

## Security and privacy

Planned URLs: `community.rbgtk.dev`, `id.rbgtk.dev/issuer-api`, `id.rbgtk.dev/verifier-api`, `id.rbgtk.dev/wallet`, `id.rbgtk.dev/wallet-api`. These are public URL intentions, not confirmed upstream paths.

Use HTTPS, default-deny ingress, explicit public endpoint allowlists, private administrative interfaces, service isolation, authenticated privileged operations, rate limits and request-size limits. Do not use Nginx as the sole authorization boundary. Confirm wallet interoperability metadata and protocol endpoint requirements before path rewriting or filtering.

Minimize personal data, avoid unnecessary credential disclosure, never log private keys or secrets, protect credential data, and document retention/deletion semantics. Cryptographic signatures do not guarantee availability, privacy, or legal ownership. Security review is not a substitute for external audit.

## Git and evidence

Use focused issue branches and pull requests; use separate worktrees for concurrently working agents. Never concurrently edit the same working tree. Do not force-push shared branches without approval, commit secrets, or merge failing required checks. GitHub Issues and PRs are the intended durable task/review records; if unavailable, keep equivalent repository records and state that limitation.

Every agent report identifies the issue, revision, work performed, commands actually executed, results, unresolved risks and next steps. PM reports milestone progress, blockers and decisions needed. Never invent test results, issues, PRs or approvals.

## Architecture decisions and escalation

Document material decisions in `docs/decisions/NNNN-short-title.md` with context, alternatives, decision, rationale, consequences, security implications and status. Cryptographic formats, wallet compatibility, trust, authorization and portability choices require Product Owner approval. Cross-key identity continuity is explicitly deferred from the MVP.

Agents may research, propose, implement approved tasks, run local tests, document and prepare PRs. Explicit human approval is required before production deployment, infrastructure changes, package publication, releases, licensing changes, paid services, destructive migrations, custodial key management, fundamental architectural changes, or accepting unresolved critical/high findings. Do not invent approval.

When agents disagree, document positions and evidence; PM proposes a resolution and escalates unresolved architectural or security disputes. No agent can unilaterally dismiss another agent's blocking security finding.

## Initial milestone: M0 — research and architecture

**Do not begin production application implementation until M0 is approved.**

- **Developer:** Research walt.id Community Stack API contracts, OID4VCI/OID4VP, credential formats, external wallet signing, DID-method signing and verification capabilities (not cross-key continuity), signed-event standards and TypeScript libraries.
- **Tester:** Build wallet compatibility matrix and interoperability, credential lifecycle, tampering, replay, unsupported-key/identity transitions and independent-verification test plans.
- **Security Reviewer:** Produce threat model, trust boundaries, custody analysis, authorization and delegation risks, unsupported historical-key-evidence risks, replay/impersonation scenarios and initial controls.
- **PM:** Consolidate architecture proposal, ADRs, risk register, prioritized backlog, dependencies and smallest vertical slice.

M0 exit criteria: documented architecture; major assumptions and limitations identified; proposed signing and per-event key-evidence approach; threat model reviewed; first slice specified; Product Owner approval. **Stop at the M0 review gate. Do not automatically proceed to M1.**

## Final operating rule

Build small, working, verifiable vertical slices rather than mountains of scaffolding. Never trade user key control, authorization integrity or evidence quality for speed. Escalate uncertainty rather than silently making security-critical assumptions.
