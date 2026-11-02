# Specialist lens: data isolation and lifecycle

Use when the code stores multi-tenant or access-controlled data, derives caches, indexes, or analytics, issues object links, exports or restores records, migrates schemas, or promises deletion, revocation, or retention. Follow one data item through every copy and state change.

## Discipline

- A tenant or owner column is not isolation. Find the query, key, path, or policy that enforces it on each read and write path.
- Trace derived copies. Safe primary data can become unsafe in search, cache, analytics, exports, previews, logs, replicas, and backups with different rules.
- Deletion and revocation are contracts. Check current, historical, cached, indexed, exported, restored, and queued copies within what the product actually promises.
- A retention preference is not automatically a vulnerability. Require an explicit boundary or guarantee, plus an unauthorized reader or later operation.
- External storage policy, replica lag, and backup access outside the repo make the claim `needs_validation`.

## Tenant and object isolation

- **Missing enforcement:** read, update, delete, list, count, or bulk query not bound to the authenticated tenant, or trusting a body field. Compare direct, nested, background, admin, import, and legacy paths.
- **Key and namespace collision:** cache keys, object paths, unique constraints, search IDs, temp files, or dedupe keys that omit tenant or environment, so two principals share a logical key. Include prompt and prefix caches.
- **Policy and query disagreement:** row-level policy, default scopes, and raw or service clients applying different predicates. Check joins, views, aggregates, `SECURITY DEFINER` functions, and what happens when the session context is missing.
- **Signed references:** object keys, shared links, or signed URLs allowing more than the issuer could, or staying valid after access changes. Bind operation, exact object, audience, expiry, and tenant.

## Derived data and disclosure

- **ACL drift:** a record's access or lifecycle changes without invalidating its cached, indexed, embedded, or preview copy.
- **Alternate readers:** private content or credentials flowing into logs, traces, analytics, or dashboards with broader access or longer retention. Confirm the data class and a realistic reader.
- **Oracles:** counts, filters, ordering, errors, unique constraints, or timing revealing protected state. Require a concrete confidential fact.

## Export, restore, and migration

- **Export scope:** exports or reports including other tenants, inaccessible fields, soft-deleted data, or secrets. Check per-item authorization after selection and authorization to download the result.
- **Import authority:** import or restore bypassing owner, schema, or validation rules, or overwriting others' records. Treat archive contents as untrusted.
- **Migration defaults:** old records missing tenant or lifecycle fields; old and new readers applying different defaults during rollout; backfills, rollback, and resumed migrations. Ask what a half-applied migration allows.
- **Backup boundary:** backups, replicas, or support snapshots with broader identity or scope than primary data. Only in-repo policy can be confirmed from source.

## Deletion and revocation

- **Soft-delete bypass:** a lookup, search, relation, or background job that ignores the lifecycle predicate. Can a deleted identifier be re-registered to reclaim something, such as a trial?
- **Stale authority:** membership removal, role downgrade, or secret revocation not reaching sessions, caches, subscriptions, or queued jobs.
- **Queued-work overrun:** deletion completing while retries, exports, or analytics recreate the data.
- **Restore reintroduces state:** undo, undelete, or recovery bringing back data or permissions that current policy forbids.

## Moves

Pick one protected record and draw its write, query, cache, index, event, export, backup, delete, and restore paths, marking the principal and tenant on every edge. Start from bypass clients, background jobs, migrations, global uniqueness, and cache keys, since those commonly miss request-scoped identity.
