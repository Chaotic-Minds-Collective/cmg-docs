# Repository Map

This is the canonical map of current Chaotic Minds Gaming repositories and their ownership boundaries. For implementation maturity, integration blockers and the cleanup snapshot, see the [ATLAS Ecosystem Implementation Registry](../plugins/ATLAS-IMPLEMENTATION-REGISTRY.md).

## Platform and ATLAS repositories

| Repository | Responsibility |
|---|---|
| `cmg-atlas-core-plugin` | Shared Core API, identity, scoped authorization, PostgreSQL migrations, event outbox/leases, wallet and reward foundations |
| `cmg-sdk` | Shared TypeScript contracts and package definitions |
| `cmg-atlas-api` | Pinned native ATLAS Server API source/dependency; not a feature plugin |
| `cmg-atlas-plugin` | Native Windows/C++ ATLAS adapter and loader integration |
| `cmg-atlas-admin-plugin` | Game/server-side ATLAS administration operations |
| `cmg-admin-console-plugin` | Operator-facing administration console and server operations dashboard |
| `cmg-atlas-connect-plugin` | Cross-grid message contracts, relay policy and Discord relay integration |
| `cmg-discord-glitch-bot` | Discord-native verification, onboarding, tickets, moderation and community operations |
| `cmg-atlas-commerce-plugin` | Products, shop, checkout, entitlements and commerce client |
| `cmg-atlas-player-plugin` | Player utilities, recovery policy, snapshots and progression-facing operations |
| `cmg-atlas-adventures-plugin` | Quest definitions, objective validation and progression |
| `cmg-atlas-tribes-plugin` | Tribe membership and group-domain reconciliation |
| `cmg-atlas-world-plugin` | World policies, safe zones and environment configuration |
| `cmg-atlas-sentinel-plugin` | Server health classification, monitoring and alerts |
| `cmg-atlas-gridforge-plugin` | Grid topology/configuration authoring and validation |
| `cmg-atlas-resource-map-plugin` | Existing ATLAS map and resource display |
| `cmg-atlas-restart-overlay-plugin` | Restart countdown and status display |
| `cmg-atlas-status-host-plugin` | Planned status-hosting component; currently a placeholder |
| `cmg-gaming-site` | Public website, player portal and game catalog |
| `cmg-7dtd-plugin` | 7 Days to Die game-specific adapter |
| `cmg-ark-plugin` | ARK: Survival Evolved game-specific adapter |
| `cmg-docs` | Canonical architecture, plugin catalog, security standards and implementation roadmap |

## Ownership boundaries

- Core is authoritative for identity, authorization, shared persistence, wallet/ledger and durable event delivery.
- SDK defines shared contracts; adapters and applications consume versioned contracts rather than Core's internal schema.
- AtlasAdmin owns game-specific server operations; Admin Console owns the operator UI.
- Connect owns relay contracts/policy; Glitch Bot owns Discord-native transport and community workflows.
- Sentinel owns monitoring/alert classification; status interfaces consume authoritative status data.
- Adventures owns quest progression; World owns environment/policy definitions.
- Resource Map owns the existing map UI/data; GridForge owns topology/configuration authoring through a shared schema.
- ATLAS, 7DTD and ARK remain separate game adapters over shared Core contracts.

## Historical repository names

The following table preserves previously documented names for continuity. These are historical names, not additional repositories.

| Canonical repository | Previously documented name |
|---|---|
| `cmg-gaming-site` | `chaotic-gaming` |
| `cmg-discord-glitch-bot` | `glitch-bot` |
| `cmg-atlas-core-plugin` | `chaotic-gaming-core` |
| `cmg-sdk` | `chaotic-gaming-sdk` |
| `cmg-atlas-plugin` | `chaotic-gaming-atlas` |
| `cmg-7dtd-plugin` | `chaotic-gaming-7dtd` |
| `cmg-ark-plugin` | `chaotic-gaming-ark` |
| `cmg-docs` | `chaotic-gaming-docs` |

Repository visibility was not changed by the consolidation work. Existing repositories were retained rather than deleted or duplicated.

## Dependency direction

Websites, Discord applications and game adapters depend on versioned Core API contracts and the SDK. They must not share database credentials or depend on Core's internal database schema. Game-specific events and operations are translated by their adapters into stable Core contracts.
