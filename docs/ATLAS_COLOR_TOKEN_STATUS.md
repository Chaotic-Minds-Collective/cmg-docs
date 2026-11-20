# ATLAS Color-Token Application Status

**Updated:** 2026-11-20

## Merged Core and SDK foundation

| Repository | Change | State |
|---|---|---|
| Core | Server-scoped color-token application records, claim and acknowledgement workflow | Merged |
| Core | Active lease renewal with current-worker identity and fencing-token checks | Merged in [Core PR #33](https://github.com/Chaotic-Minds-Collective/cmg-atlas-core-plugin/pull/33) |
| SDK | Typed claim, renew, and acknowledge client methods | Merged in [SDK PR #8](https://github.com/Chaotic-Minds-Collective/cmg-sdk/pull/8) |
| Native adapter | Verified color application on the ATLAS game thread | Not implemented |

## Lease protocol

- Claim: `POST /integrations/servers/{serverId}/atlas/color-token-applications/claim`
- Renew: `POST /integrations/servers/{serverId}/atlas/color-token-applications/{applicationId}/renew`
- Acknowledge: `POST /integrations/servers/{serverId}/atlas/color-token-applications/{applicationId}/ack`

Renewal requires the same server-scoped worker principal and active lease token. The lease must not already be expired. The requested duration is bounded to 15–300 seconds; successful renewals are audited. A stale or expired lease cannot be revived.

The SDK exposes matching typed methods for claim, renewal, and acknowledgement. These methods provide transport contracts only; they do not perform the native color change.

## Native adapter work still required

1. Resolve the player and target asset at execution time; do not retain actor pointers between callbacks.
2. Validate Steam ownership, server/grid scope, token kind, asset type, and selected color against server-owned policy.
3. Perform the color application only on the ATLAS game thread using an API/ABI verified against the exact target server build.
4. Persist a durable idempotency receipt before acknowledging Core. Define and test the ambiguous crash window so an uncertain application is not blindly repeated.
5. Renew the Core lease while waiting for game-thread work. Stop processing if renewal fails or the lease token is no longer current.
6. Test duplicate replay, stale lease, player disconnect, target despawn, partial application, lost acknowledgement, plugin shutdown, and persistence across save/restart on an isolated ATLAS server.

## Release boundary

No native color application is enabled by the Core/SDK changes. Do not configure or enable a native handler until the exact server build, API version, ABI, persistence behavior, and idempotency protocol have passed isolated-server tests. No production database migration or live color change was performed by these changes.
