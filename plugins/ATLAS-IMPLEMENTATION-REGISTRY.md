# ATLAS Ecosystem Implementation Registry

**Audit snapshot:** 2026-11-19  
**Scope:** The complete ATLAS-related GitHub ecosystem in the Chaotic Minds Collective organization, including platform applications and game adapters.

This registry records repository ownership, implementation maturity, overlap boundaries, integration blockers, and the current cleanup state. It is an implementation-status document, not a promise that every planned feature is production-ready.

## Sources of truth

1. The current default branch of each canonical GitHub repository is the implementation source of truth.
2. Shared contracts and public APIs are authoritative at their versioned/pinned GitHub revisions.
3. The uploaded Drive SDK ZIPs and plugin-suite ZIPs are historical design material only. The inspected `AtlasApi_1.71.zip` is incomplete and must not replace the pinned GitHub API dependency.
4. Architecture documents describe intended ownership; this registry distinguishes that intent from verified implementation.

## Current consolidation snapshot

- **34** repositories exist in the organization.
- **22** belong to the ATLAS ecosystem or its direct platform/game integration layer; the other 12 are separate CMG projects and were left untouched.
- All 22 in-scope repositories now have only their default branch; the three remediation branches were automatically removed after merge.
- The organization-wide open pull-request check returned **zero open PRs** across the 22 in-scope repositories.
- **17 obsolete or superseded branches** have been explicitly deleted across the cleanup passes. The final additional deletion was `fix/quest-progress-completion-integrity` in Core, verified identical to `main` with no open PR.
- **10 PRs have been merged** across the cleanup passes: the seven recorded in the prior snapshot plus Core #29, Glitch Bot #25, and native adapter #20.
- **No repositories were deleted.** The reviewed repositories have distinct responsibilities; apparent overlaps are integration boundaries, not confirmed duplicate repositories.

### Latest validated changes — 2026-11-20

- [Core PR #29](https://github.com/Chaotic-Minds-Collective/cmg-atlas-core-plugin/pull/29) removed a duplicate `STEAM_OPENID_RETURN_URL` entry while retaining the required callback. Core tests, migration checks, validation, and static analysis passed.
- [Glitch Bot PR #25](https://github.com/Chaotic-Minds-Collective/cmg-discord-glitch-bot/pull/25) added bounded multi-message server-status delivery, retry-safe partial progress, a conflicting-catalog recovery guard, and single-flight monitor cycles. Node tests, CodeQL, and post-merge checks passed.
- [Native adapter PR #20](https://github.com/Chaotic-Minds-Collective/cmg-atlas-plugin/pull/20) pinned the adapter to hardened API commit `a757d0c256e6d221742fb131d46d8049c12815a6`, corrected the MSVC API-version diagnostic, and updated compatibility documentation. The API build and adapter Windows x64 build passed; live ATLAS server compatibility remains unverified.
- Core quest progress now rejects initial progress jumps and excessive subsequent increments; the corresponding regression tests are on the default branch.

### Security snapshot

- The current Dependabot inventory returned 29 historical alerts for `cmg-admin-console-plugin`, all in the `fixed` state, and one historical alert for `cmg-discord-glitch-bot`, also `fixed`. The current open-alert query returned none for either repository.
- Code-scanning output must be interpreted carefully: an empty alert list is not proof that CodeQL has run where the workflow is absent.
- Core issues #7 and #8 are closed as `not_planned`; their Aikido report text points to historical `chaotic-gaming-core` paths, not the current `cmg-atlas-core-plugin` implementation. Closure is not proof that any historical credential was rotated.
- A separate Drive audit found three Discord webhook URLs in the legacy Atlas Command spreadsheet. Those credentials were not rotated as part of this GitHub-only cleanup; rotate them if still active and restrict spreadsheet access.

## Canonical repository ownership

Status meanings:

- **Foundation:** real shared infrastructure exists; dependent workflows or production proof remain.
- **Partial:** useful implementation exists, but the full feature is not complete.
- **Scaffold:** repository/contracts/planning exist, but the intended feature is not implemented end-to-end.
- **Reference:** source material or compatibility dependency; not a feature plugin.
- **UI/adapter:** a client or game-specific integration that consumes shared contracts.

| Repository | Canonical responsibility | Current maturity | Boundary / remaining work |
|---|---|---|---|
| `cmg-atlas-core-plugin` | Shared identity, authorization, PostgreSQL persistence, migrations, event outbox/leases, commerce and reward foundations | Foundation | Core owns identity, RBAC, shared persistence, append-only wallet ledger, authenticated wallet balance/ledger reads, atomic wallet-funded order payment, color-token application records, and shared delivery infrastructure. Manual adjustments/transfers and running native reward delivery remain incomplete. |
| `cmg-sdk` | Shared TypeScript contracts and package definitions | Foundation | Keep contracts versioned and consistent with Core. CI and consumer contract compatibility must remain green. |
| `cmg-atlas-api` | Pinned native ATLAS Server API source/dependency | Reference | The adapter pins hardened commit `a757d0c256e6d221742fb131d46d8049c12815a6`. Not a feature plugin; validate the exact pinned revision against the target server build and ABI before runtime claims. |
| `cmg-atlas-plugin` | Native Windows/C++ ATLAS adapter and loader integration | Partial | Pins `cmg-atlas-api` at `a757d0c256e6d221742fb131d46d8049c12815a6`; Windows x64 compilation passes with loader path/dependency and hook hardening. Player hooks remain opt-in/disabled; isolated-server ABI and load proof are still required. |
| `cmg-atlas-admin-plugin` | Game/server-side ATLAS administration backend and operations | Partial | Keep game-specific server operations here. Use Core identity/authorization rather than a second independent permission authority. |
| `cmg-admin-console-plugin` | Operator-facing administration console | Partial | Owns the UI and operator workflows. Consume Core APIs; do not duplicate Core authorization, audit, or canonical status logic. |
| `cmg-atlas-connect-plugin` | Cross-grid message envelopes, relay policy, announcements and Discord relay contracts | Partial | Contracts and retry/scope policy exist; actual Discord delivery remains incomplete. |
| `cmg-discord-glitch-bot` | Discord-native onboarding, verification, tickets, moderation and community operations | Partial | Server-status delivery now preserves full catalogs, resumes partial sends safely, and prevents overlapping monitor cycles. Ticket lifecycle persistence/transcripts remain incomplete. Use it as a transport/integration point for Connect, not as a second chat or identity authority. |
| `cmg-atlas-commerce-plugin` | Shop, checkout, products, entitlements and commerce client | Partial | Core owns wallet/ledger authority; Commerce exposes authenticated wallet balance/ledger reads and wallet-funded payment for eligible orders. Actual game delivery, refund/reconciliation, and full legacy shop parity remain incomplete. |
| `cmg-atlas-player-plugin` | Player utilities, recovery policy, snapshots and progression-facing operations | Partial | Snapshot contracts, encryption and notifications exist; native capture and full restore workflows remain incomplete. |
| `cmg-atlas-adventures-plugin` | Quest definitions, objective validation, progression and completion | Partial | Durable progress persists through Core with optimistic-version conflict retries, outbox leases/renewals, and lease-fenced acknowledgements. Current objectives cover only `player.joined` and `player.left`; complete event coverage and actual game reward delivery remain. Do not add an independent queue. |
| `cmg-atlas-tribes-plugin` | Tribe membership and group-domain reconciliation | Partial | Own tribe-domain reconciliation and group objectives; consume normalized Core events. |
| `cmg-atlas-world-plugin` | World policies, safe-zone rules and environment configuration | Partial | Current policies are observe-only/dry-run. Live enforcement, NPCs, weather and scheduled world-event controls remain future work. |
| `cmg-atlas-sentinel-plugin` | Server health classification, monitoring and alert processing | Partial | Monitoring lifecycle/classification exists; durable alert storage, notification delivery and recovery automation remain incomplete. |
| `cmg-atlas-gridforge-plugin` | Grid topology/configuration authoring and validation | Scaffold | Share canonical grid/map data with Resource Map; do not rebuild the existing map UI. Add CI and actual authoring/validation behavior. |
| `cmg-atlas-resource-map-plugin` | Existing ATLAS map and resource display | UI | Preserve the existing map interface/data. Provide the shared map/grid contract consumed by GridForge. |
| `cmg-atlas-restart-overlay-plugin` | Restart countdown and status display | UI | Consume authoritative status/restart data; do not create a separate status source or use Git polling as the long-term transport. |
| `cmg-atlas-status-host-plugin` | Planned status-hosting component | Scaffold | Currently a placeholder. Keep only as a planned distinct host component; Sentinel owns monitoring/alert classification. |
| `cmg-gaming-site` | Public website, player portal, account and game catalog UI | UI | Integrate through authenticated Core APIs; never connect the browser directly to PostgreSQL. |
| `cmg-7dtd-plugin` | 7 Days to Die game-specific adapter | UI/adapter | Keep game-specific translation in the adapter; share Core contracts without importing ATLAS-only assumptions. |
| `cmg-ark-plugin` | ARK: Survival Evolved game-specific adapter | UI/adapter | Keep as a separate game adapter and later rollout; do not treat ARK item blueprints/config as ATLAS data. |
| `cmg-docs` | Canonical platform architecture, plugin catalog, security standards and implementation roadmap | Reference | Keep repository ownership, implementation status and integration decisions synchronized with live code. |

## Original six-plugin architecture mapping

| Original module | Current owner(s) | Consolidation decision |
|---|---|---|
| AtlasCore | `cmg-atlas-core-plugin` | Canonical shared identity, RBAC, persistence, outbox/leases, append-only wallet/ledger, wallet-funded eligible orders, color-token application records, and API foundation. |
| AtlasAdmin | `cmg-atlas-admin-plugin` + `cmg-admin-console-plugin` | Keep server-side game operations separate from the operator UI. Consolidate authorization and audit through Core. |
| AtlasChat | `cmg-atlas-connect-plugin` + `cmg-discord-glitch-bot` | Connect owns relay contracts/policy; Glitch Bot provides Discord-native transport and community workflows. |
| AtlasEconomy | Core + `cmg-atlas-commerce-plugin` | Core owns wallet/ledger; Commerce owns catalog, checkout and entitlements. Player/native code performs game actions. |
| AtlasTames | `cmg-atlas-player-plugin` | Preserve recovery/snapshot ownership. Cryo serialization and full tame restoration remain incomplete. |
| AtlasFleet | No dedicated repository | Missing module. Preserve as a distinct future module for ship recovery, spawning, repair, construction and combat timers; do not bury it in Player or World. |
| AtlasWorld | `cmg-atlas-world-plugin` + `cmg-atlas-adventures-plugin` | Keep world policies separate from quest definitions/progression. They are complementary, not duplicates. |

## Ownership rules and overlap resolution

1. **Identity and authorization:** Core is authoritative. Website, bot, console and game modules use scoped Core identities/permissions; Discord roles do not implicitly grant game-administration permissions.
2. **Status and monitoring:** Sentinel owns monitoring and alert classification. Core exposes authoritative operational records. Admin Console, Status Host and Restart Overlay consume that data rather than implementing separate status systems.
3. **Chat:** Connect owns message envelopes, cross-grid scope and relay policy. Glitch Bot owns Discord-native transport and community workflows.
4. **Wallet and color tokens:** Core owns the wallet/append-only ledger and color-token application records. Commerce owns purchases/entitlements. The native adapter must validate and apply the selected color, then acknowledge success before a token is consumed. No independent SQLite/MySQL currency ledger.
5. **Events:** Core owns shared durable outbox/lease infrastructure. Each module owns its domain processing and must be idempotent. Do not create independent queues per plugin.
6. **Quests and world policy:** Adventures owns quest progression; World owns environment and policy definitions. Do not combine their domain models.
7. **Maps and topology:** Resource Map owns the existing map experience/data; GridForge owns topology/configuration authoring and validation through a shared schema.
8. **Admin:** AtlasAdmin owns server-side ATLAS operations; Admin Console owns the operator interface. Avoid two authorization systems or duplicate canonical status stores.

## Confirmed gaps and integration blockers

| Priority | Gap | Owner(s) | Required outcome |
|---|---|---|---|
| P0 | Native API/loader runtime proof | `cmg-atlas-api`, `cmg-atlas-plugin` | Verify the pinned API against an isolated target ATLAS server build; validate dependency checks, safe paths, game-thread dispatch and shutdown. |
| P0 | Native reward and item delivery | Core + Commerce + native adapter | Core wallet APIs and reward claim/ack contracts exist. Implement a running idempotent delivery worker and verified native ATLAS item/creature/ship grants; acknowledge only after the game confirms success. |
| P0 | Quest objective and reward completion | Core + Adventures + native adapter | Durable progress, optimistic conflict retries, lease renewal/fencing and durable acknowledgements exist. Add remaining event types, connect completion to Core reward jobs, and deliver rewards on the game thread before acknowledging game delivery. |
| P1 | Discord relay delivery | Connect + Glitch Bot | Implement actual delivery, retry/idempotency and safe reconnect behavior against the existing relay contracts. |
| P1 | Player capture/restore | Player + native adapter | Complete a verified native snapshot capture/restore pipeline with ownership, encryption and rollback tests. |
| P1 | Admin boundary cleanup | AtlasAdmin + Admin Console + Core | Route identity, authorization and audit through Core; keep server operations separate from the UI. |
| P1 | Sentinel persistence/notifications | Sentinel + Core | Persist alert state and deliver notifications/recovery signals idempotently. |
| P2 | Grid authoring | GridForge + Resource Map | Implement topology editing/validation using the existing map schema and add CI. |
| P2 | Original Fleet capabilities | Future AtlasFleet module | Design a separate ship-domain module and its native adapter contract. |
| P2 | World events/environment | World + native adapter | Add scheduled events, NPC/weather controls and safe live enforcement only after observe-only validation. |

### Legacy capabilities that must remain visible

- Cryo storage and complete tame serialization/restoration.
- Ship spawning, repair, construction, recovery and combat timers.
- Player-selected creature, ship and armor color application.
- Full legacy shop/trading parity and native color-token application. Wallet balance/ledger reads and eligible wallet-funded order payment are implemented; actual in-game delivery remains separate.
- Scheduled world events, NPCs and weather control.
- Verified native game hooks for planned gameplay operations.

These are **not complete** merely because a related repository, API contract, or issue exists.

## Data, lifecycle and integration constraints

- Use the Core-owned PostgreSQL/Neon persistence path for shared player, wallet, entitlement, quest and cluster records. MySQL and SQLite drafts are not competing sources of truth.
- Keep game-specific data isolated by game/domain and use versioned Core contracts; adapters must not depend on Core's internal database schema or credentials.
- Use Core's durable outbox/lease infrastructure for cross-module work. Consumers must be idempotent, retry safely and expose delivery failures.
- Native plugin initialization and cleanup use the verified `Plugin_Init` / `Plugin_Unload` lifecycle. Do not perform complex initialization in `DllMain`; do not leave detached workers running after unload.
- Treat API names and offsets as build-specific. A successful compile is not proof of ABI/runtime compatibility.
- Never trust client-submitted currency, inventory, permissions or reward values. Privileged operations require server-side authorization and audit records.
- Keep development/test/production isolated. No production server changes, live payments, paid infrastructure or deployments without a separately approved change.

## Consolidation order

1. Keep SDK/Core contracts and migrations green; complete isolated ATLAS server validation for the pinned API/adapter.
2. Implement the Core-backed native reward-delivery worker and verified game grants, including color-token application.
3. Expand Adventures objectives and connect quest completion to Core reward jobs without adding a second queue.
4. Implement Connect's actual Discord delivery through Glitch Bot.
5. Complete Player native snapshot/restore and acknowledge-after-success workflows.
6. Consolidate AtlasAdmin/Admin Console around Core authorization and audit.
7. Implement Sentinel durable alert/notification workflows.
8. Build GridForge on Resource Map's shared schema.
9. Design AtlasFleet as a distinct module.
10. Expand World enforcement, NPCs, weather and scheduled events only after observe-only validation.

## Cleanup policy

- Do not delete a repository solely because it is inactive, a placeholder, or adjacent to another module.
- Delete a repository only when its complete history and unique functionality are preserved in a verified canonical destination and its dependencies have been redirected.
- Delete a branch only after its commits are merged, its tree is identical/empty, or every unique change has been deliberately ported.
- Close a PR only after its changes are merged elsewhere, superseded, or explicitly rejected with the reason recorded.
- Keep this registry aligned with live GitHub state; update statuses only after code and validation evidence support the change.
