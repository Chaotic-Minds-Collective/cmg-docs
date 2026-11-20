# ATLAS Core-Backed Reward Delivery Status

**Updated:** 2026-11-20

## Merged implementation

| Repository | Change | State |
|---|---|---|
| Core | Configurable reward claims and active lease renewal | Merged |
| Core | `manual_review` queue state and audited admin reconciliation | Merged |
| Core | New idempotency key on an administrator-approved retry only | Merged |
| SDK | Core renewal client and renewable reward worker | Merged |
| SDK | Require lease renewal support before claiming | Merged |
| Native adapter | Background claim/renew/ack worker, game-thread dispatch, allowlist, journal, manual-review fallback | Merged; disabled by default |

## Delivery protocol

1. Core claims a server-scoped reward and returns a lease token and expiry.
2. The adapter's background worker renews the lease while waiting for game-thread work.
3. A timer callback dispatches the native grant on the game thread.
4. The adapter validates the player Steam ID, item key, quantity, and server-owned Blueprint allowlist.
5. Before granting, it writes and flushes a `granting` journal record. After the native call it adds and verifies a custom item-data marker, then writes and flushes `granted`.
6. Core is acknowledged only after the journal and grant receipt are confirmed.
7. If a worker restart or ambiguous result leaves a `granting` journal record, the adapter does not grant again. It parks the reward in Core's `manual_review` state.
8. A platform commerce administrator must verify the player's inventory and resolve the reward as `delivered` or `retry`. An approved retry rotates the idempotency key so it is distinguishable from the uncertain prior attempt.

This is an at-least-once queue with a durable game-side journal and manual reconciliation for the non-atomic game-grant/Core-ack boundary. It does not claim transactional exactly-once delivery.

## Native enablement gates

The native worker is off unless all of these are explicitly configured:

- `CHAOTIC_ATLAS_ENABLE_REWARD_DELIVERY=1`
- `CHAOTIC_ATLAS_GIVE_ITEM_ABI_VERIFIED=1`
- `CHAOTIC_ATLAS_REWARD_VERIFIED_SERVER_BUILD` exactly matches runtime `ATLAS_SERVER_BUILD`
- `CHAOTIC_ATLAS_REWARD_VERIFIED_API_VERSION` exactly matches runtime `ATLAS_SERVER_API_VERSION` and the pinned adapter API version
- `CHAOTIC_ATLAS_REWARD_ITEM_MAP_JSON` contains reviewed item-key-to-Blueprint mappings
- `CHAOTIC_ATLAS_REWARD_JOURNAL_PATH` is an absolute, writable, durable path

**Important:** `AShooterPlayerController.GiveItem` is described in the uploaded plugin build guide but is not declared in the pinned public API headers. The adapter checks for the runtime function address and requires explicit operator attestation, but that is not a substitute for ABI proof. No item Blueprint paths are invented or configured in this change.

## Required isolated-server acceptance tests

- Verify the native function address and exact call ABI against the target server build.
- Grant a single allowlisted item to an online test player and verify item IDs, quantity, and custom-data marker.
- Confirm the marker survives inventory save, player logout/login, and server restart.
- Replay the same reward and confirm no duplicate item is created.
- Simulate a lost Core acknowledgement after a successful grant; confirm the journal causes a delivered acknowledgement without a second grant.
- Simulate a crash after inventory mutation but before the journal changes from `granting`; confirm the reward enters manual review rather than being granted again.
- Verify an offline player produces a retryable failure without an item grant.
- Verify lease-renewal failure prevents acknowledgement and does not allow stale-token acknowledgement.
- Verify a partial/ambiguous native grant enters manual review.
- Verify an administrator-approved retry rotates the idempotency key and proceeds only after the administrator confirms no item was granted.
- Verify invalid item keys and unlisted Blueprint paths cannot be granted.
- Verify plugin shutdown stops the worker promptly and does not perform game API calls from the background thread.

## Deployment boundary

The Core migrations and code are merged, but no production database migration was applied. The native adapter code is merged but not enabled on a live server. No live item grants, production server configuration changes, or credential changes were made.
