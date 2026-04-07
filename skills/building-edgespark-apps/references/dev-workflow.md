# Dev Workflow

This is the first-class operational workflow for the parts users touch most often: auth config, storage schema, database schema, vars, secrets, generated types, and deploy.

Use `edgespark ... --help` for exact flags and arguments. This document is for workflow, sequencing, and non-obvious guardrails.

## Canonical Paths

Default to these paths unless the user explicitly asks for a lower-level or debugging flow:

- Browser auth: `@edgespark/web`
- Managed browser auth UI: `client.authUI.mount()`
- Custom browser auth: `client.auth`
- App API calls from the browser: `client.api.fetch()`
- Server auth in routes: `auth` from `edgespark/http`
- Database schema changes: defs -> `edgespark db generate` -> `edgespark db migrate`
- Client uploads: server-generated presigned PUT URL flow
- Secrets: `edgespark secret set` -> human browser handoff -> rerun the blocked command

## Contents

- [Database Schema](#database-schema)
- [Storage Schema](#storage-schema)
- [Auth Config](#auth-config)
- [Vars](#vars)
- [Secrets](#secrets)
- [Generated Types](#generated-types)
- [Deploy Preflight](#deploy-preflight)
- [Deploy](#deploy)
- [Recommended Order For Common Tasks](#recommended-order-for-common-tasks)

## Database Schema

Source of truth:

- `src/defs/db_schema.ts`
- `src/defs/db_relations.ts` when app-level relations are needed

Workflow:

1. Edit `src/defs/db_schema.ts`
2. Edit `src/defs/db_relations.ts` if relations changed
3. Run `edgespark db generate`
4. Review generated migration SQL
5. Run `edgespark db migrate`
6. Deploy after migrations are applied

Rules:

- use additive, forward-only changes by default
- do not use DDL through `edgespark db sql`
- use `db sql` for queries or DML, not schema evolution
- destructive migration SQL requires explicit confirmation and review

## Storage Schema

Source of truth:

- `src/defs/storage_schema.ts`

Workflow:

1. Edit `src/defs/storage_schema.ts`
2. Run `edgespark storage apply`
3. Inspect current synced buckets with `edgespark storage bucket list` when needed

Rules:

- bucket declarations live in code, not in ad hoc CLI-only state
- removing buckets is dangerous and requires explicit confirmation

## Auth Config

Source of truth:

- `configs/auth-config.yaml`

Workflow:

1. Pull current config with `edgespark auth pull` when you want a local working copy
2. Or inspect current config with `edgespark auth get`
3. Edit `configs/auth-config.yaml`
4. Run `edgespark auth apply`
5. Deploy so the active deployment picks up the new auth config

Rules:

- `auth get` is for inspection
- `auth pull` writes the current server config to local YAML
- `auth apply` validates and stores the config on the active deployment record
- changes take effect on the next deploy
- deploy does not silently auto-apply local auth config; `edgespark auth apply` remains explicit

## Vars

Source of truth for allowed keys in code:

- `src/defs/runtime.ts`

Workflow:

1. Add the key to `VarKey` in `src/defs/runtime.ts`
2. Run `edgespark var set KEY=VALUE`
3. Confirm when useful with `edgespark var list`
4. Read it in code with `vars.get("KEY")`

Rules:

- vars are plain remote environment-specific values
- `vars.get()` returns `string | null`
- do not use undeclared keys in code

## Secrets

Source of truth for allowed keys in code:

- `src/defs/runtime.ts`

Workflow:

1. Add the key to `SecretKey` in `src/defs/runtime.ts`
2. Run `edgespark secret set KEY`
3. Show the returned secure URL to the user
4. The human user enters the secret value in the browser
5. Rerun the command that was blocked on the missing secret
6. Read it in code with `secret.get("KEY")`

Rules:

- `edgespark secret set` prints a secure URL — the human must open it in a browser and enter the value there. This is a hand-off: show the URL to the user, explain that the CLI/LLM never handles the secret value, and tell them which command to rerun afterward.
- secret values never pass through CLI output, agent context, or LLM API calls. This is by design — no secret value should ever appear in a conversation, log, or prompt.
- `secret.get()` returns `string | null`
- do not hardcode secrets in code or commit them

## Generated Types

Source of truth after pull:

- `src/__generated__/edgespark.d.ts`
- `src/__generated__/server-types.d.ts`

Workflow:

1. Run `edgespark pull types` when generated types are missing, stale, or still placeholders
2. Read `edgespark.d.ts` to confirm available imports
3. Read `server-types.d.ts` for the actual method contracts

Rules:

- do not edit generated files
- do not trust scaffold placeholders as the final SDK contract
- the CLI has staleness checks for SDK types; if types are stale, refresh them before coding or deploying

## Deploy Preflight

Deploy in this repo is not just "build and upload". It has careful preflight behavior.

Before the actual deployment, deploy checks for things like:

- project structure resolution from `edgespark.toml`
- required local dependencies
- generated/barrel file consistency
- auth config drift between local config and server state
- runtime key consistency
- migration readiness and schema divergence
- storage schema drift
- stale generated types

This is why deploy workflow matters and why the sequence before deploy should stay explicit.

Important:

- preflight catches many classes of drift, but it is not a substitute for following the workflow correctly
- do not rely on deploy to silently fix auth config, runtime keys, or schema workflow mistakes for you

## Deploy

Workflow:

1. Make code/config/schema changes
2. Run migrations and storage/auth apply steps first when relevant
3. Run `edgespark deploy`
4. Use `edgespark deploy --dry-run` when you need validation only
5. Use `edgespark deploy promote` only when production promotion is intended

Rules:

- deploy includes preflight validation, build, upload, and deployment execution
- `deploy --dry-run` still exercises validation paths without performing the deploy
- do not promote unless the user intends production rollout

## Recommended Order For Common Tasks

### Add a new database-backed feature

1. Update `db_schema.ts`
2. Generate and apply migrations
3. Pull types if needed
4. Implement server routes
5. Implement web usage with `@edgespark/web`
6. Deploy

### Add a new file-upload feature

1. Update `storage_schema.ts` if a new bucket is needed
2. Run `edgespark storage apply`
3. Implement presigned upload flow on the server
4. Implement browser upload using `requiredHeaders`
5. Deploy

### Add a new external integration

1. Add keys to `VarKey` and/or `SecretKey`
2. Set vars and register secrets
3. Implement server code using `vars.get()` and `secret.get()`
4. Deploy
