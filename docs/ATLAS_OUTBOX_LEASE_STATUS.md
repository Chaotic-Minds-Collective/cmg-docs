# Atlas Core Outbox Consumer Readiness

**Updated:** 2026-11-20

| Consumer | Current lease safety | Remaining boundary |
|---|---|---|
| Adventures | One event per poll; active lease is renewed while processing; immediately polls again after nonempty work. | Event coverage remains limited to `player.joined` and `player.left`; native reward delivery remains separately gated. |
| Connect | One event per poll; filters to `chat.global` and `chat.tribe`; exposes fenced renewal. | Caller must schedule renewal and abort work on lease loss. No Discord delivery worker, durable receipt store, or canonical chat payload mapping. |
| Tribes | One event per poll; consumer and processor reject multi-event claims. | Processing must remain bounded within the active lease until a worker heartbeat is added. |
| Sentinel | One event per poll; consumer and processor reject multi-event claims. | Processing must remain bounded within the active lease until a worker heartbeat is added. |
| Native ATLAS host | One event per poll; rejects multi-event claim responses before invoking handlers. | Processing must remain bounded within the active lease until host-level renewal is implemented. |

## Safety invariant

Do not increase a consumer's claim limit above one unless it renews every outstanding lease while events wait and process. Do not acknowledge an event after lease loss. Handlers must be idempotent because Core outbox delivery is at-least-once.

## Non-goals of these changes

These lease-safety changes do not implement Discord delivery, native game mutations, live RCON, production configuration, or production database changes.
