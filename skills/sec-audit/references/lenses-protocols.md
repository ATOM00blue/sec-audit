# Specialist lens: webhooks, queues, and internal messages

Use when the code uses webhooks, queues, pub/sub, Durable Object messages, streaming, RPC, or service bindings. Covers peer identity, how a message is interpreted, routing, replay, ordering, and delivery.

## Discipline

- "Internal" is not authentication. Name the peer identity at each hop and show how it becomes the principal used for authorization.
- Schema validation proves shape, not origin, authority, order, or safe values. Follow decoded fields to policy and side effects.
- Write down retry, ordering, acknowledgement, and dedupe behavior before judging state changes. Delivery guarantees of the broker and the application differ.
- A parser disagreement needs two concrete consumers and one security-relevant value that differs.
- Broker ACLs, topic attachment, and platform delivery facts outside the repo make the claim `needs_validation`.

## Interpretation

- **Boundary disagreement:** components differ on length, duplicate fields, unknown fields, encoding, numeric width, or envelope versus body precedence.
- **Defaults:** unknown variants, missing discriminators, zero values, or compatibility mappings reaching code that assumes a validated case. Look at `default:` branches and how old consumers read new fields.
- **Envelope and payload mismatch:** authorization using routing metadata while the handler acts on a different tenant or account in the body.

## Identity and authorization

- **Path inconsistency:** an auth layer covering unary calls but not streams, health, reflection, gateway-transcoded paths, or compatibility services.
- **Peer versus principal:** a bound or mutually authenticated channel, where a caller-controlled field then chooses the user or tenant.
- **Per-item gaps:** a stream, subscription, or batch authorized once, then later items naming other resources or continuing after revocation.
- **Correlation confusion:** predictable or reused correlation IDs letting a response, webhook, or cancel satisfy another caller's pending operation.
- **Producer as control plane:** a message body declaring itself an admin event, provider callback, or migration instruction without an authenticated producer.

## Webhooks

Verify the signature over the raw body, before parsing. Check the timestamp window, replay protection, idempotency key scope, event-type allowlist, and tenant binding. Then ask whether a forged, replayed, or reordered event can change money, ownership, or access. Check for a shared-secret comparison that isn't constant time, a missing-secret fallback, and a test mode reachable in production.

## Delivery

- **Duplicates:** retries repeating a side effect because dedupe is absent, runs after the mutation, or uses a key that collides across tenants. Money and grants need this most.
- **Stale messages:** older state, revoked membership, or cancelled work arriving after newer state and overwriting it.
- **Ack and commit order:** acknowledging before commit loses security work; committing before an unreliable ack duplicates it.
- **Partial transitions:** several consumers implementing one transition, with retries leaving only some committed.
- **Dead letters:** failed messages with secrets or cross-tenant content reaching lower-trust consumers, logs, or operator views.
- **Tenant scope:** a publisher or subscriber choosing another tenant's topic or queue. Tenant text in a payload isn't isolation.

## Moves

Draw producer → transport → gateway → consumer → storage for each message family, recording at each hop the authenticated peer, authoritative tenant fields, validation, and side effect. Compare normal, retry, dead-letter, replay, and migration routes.
