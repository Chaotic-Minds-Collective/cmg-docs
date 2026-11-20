# Implementation Roadmap

**Status snapshot:** 2026-11-20. Core identity/authorization, wallet and ledger APIs, eligible wallet-funded orders, payment sandbox/webhook verification, durable quest progress, reward lease renewal/manual-review APIs, color-token application lease renewal, typed SDK color-token methods, and a disabled native reward-worker scaffold are implemented. Reviewed Core outbox consumers in Adventures, Connect, Tribes, Sentinel, and the native host now enforce single-event claims; Adventures renews its active lease and Connect exposes renewal. Actual native ATLAS item/color application, Discord delivery, and isolated ATLAS/7DTD runtime validation remain release blockers.

## Phase 1 — Platform foundation
- [x] Create dedicated repositories.
- [x] Establish Core TypeScript/Fastify scaffold.
- [x] Add initial PostgreSQL identity, access, audit, outbox and idempotency migration.
- [x] Add game registry and isolated game schemas.
- [x] Add service-token authentication middleware and provisioning CLI.
- [x] Define shared SDK contracts.
- [x] Pass Core CI build and migration checks.
- [x] Review and merge foundational PRs.

## Phase 2 — Identity and API
- [ ] Player registration and secure login/recovery.
- [x] Discord and Steam account linking with proof of ownership.
- [x] Scoped permission enforcement for user and service identities.
- [ ] API versioning, request validation, rate limiting and audit coverage.
- [ ] API contract tests and OpenAPI documentation.

## Phase 3 — Existing applications
- [ ] Integrate website with Core account and game catalog APIs.
- [x] Integrate Glitch Bot using a dedicated service identity.
- [x] Preserve bot-native features during Core outages.
- [ ] Add integration health and reconnect behavior.

## Phase 4 — Game integrations
- [ ] ATLAS compatibility proof-of-concept on an isolated test server.
- [ ] 7DTD compatibility proof-of-concept on an isolated test server.
- [ ] Normalize player identity, sessions, server health and game events.
- [ ] Build admin tools and safe configuration workflows.
- [ ] ARK adapter when first two game integrations are stable.

## Phase 5 — Community and commerce
- [ ] Community tickets, moderation cases, announcements and events.
- [x] Product catalog and game-specific product mappings.
- [x] Payment sandbox and signed webhook verification.
- [x] Entitlements, append-only wallet ledger and reward queue.
- [ ] Reconciliation, refund handling and abuse controls.

## Reward delivery implementation status

- [x] Core reward claim/ack contract with configurable 15–300 second leases.
- [x] Fenced renewal by the current service identity holding the active server-scoped lease.
- [x] Manual-review state for ambiguous external game outcomes; manual-review rewards are excluded from automatic claim retries.
- [x] Audited platform-admin reconciliation. An approved retry receives a new idempotency key; ordinary retries retain the existing key.
- [x] SDK worker requires lease renewal before claiming and stops acknowledgement after lease loss.
- [x] Native adapter worker scaffold with background HTTP, game-thread timer dispatch, item-key allowlist, durable append-only journal, and manual-review fallback.
- [ ] Verify the native `AShooterPlayerController.GiveItem` ABI against the exact isolated target server build. The method is documented in the uploaded build guide but is not declared in the pinned public API headers.
- [ ] Verify that item custom-data idempotency markers persist across inventory save, logout, and server restart.
- [ ] Run end-to-end tests for duplicate claims, lost acknowledgements, worker restart between grant and journal commit, partial grants, player disconnect, lease loss, and admin reconciliation.
- [ ] Configure reviewed item Blueprint mappings and an absolute durable journal path only after isolated validation.
- [ ] Enable live reward delivery only after the exact server build/API version and ABI are attested and the isolated test suite passes.

The native worker remains disabled by default. No production server configuration, database migration, or live item grant has been performed.

## Core outbox lease-safety status

- [x] Adventures: one-event claims, active lease renewal during processing, and immediate polling after nonempty work.
- [x] Connect: one-event claims filtered to chat.global and chat.tribe; exposes fenced renewal for its caller-owned heartbeat. No Discord delivery worker or durable delivery receipts yet.
- [x] Tribes: one-event consumer and processor guard; multi-event arrays fail closed.
- [x] Sentinel: one-event consumer and processor guard; multi-event arrays fail closed.
- [x] Native ATLAS host: one-event claims and multi-event response rejection before plugin callbacks.
- [ ] Add active-event heartbeats to Tribes, Sentinel, and the native host before allowing long-running handlers or increasing claim limits.
- [ ] Implement Connect's durable delivery worker only after canonical chat payload mapping, destination routing, per-destination idempotency receipts, and terminal-failure handling are defined.

## Release gates
No production deployment, live payment processing, game-server modification, live Discord changes or paid infrastructure until explicitly approved. All game adapters must pass compatibility tests on isolated servers first.
