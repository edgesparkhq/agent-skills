# Project Structure

EdgeSpark projects are scaffolded, then refined by pulled schema and generated SDK types.

## Contents

- [Layouts](#layouts)
- [`edgespark.toml`](#edgesparktoml)
- [Generated Files](#generated-files)
- [Files To Read Before Coding](#files-to-read-before-coding)
- [Import Boundary](#import-boundary)
- [`src/defs/index.ts` Barrel Convention](#srcdefsindexts-barrel-convention)
- [Scaffolded Agent Files](#scaffolded-agent-files)

## Layouts

Server-only projects usually look like:

```text
.
├── <agent-file>.md
├── configs/
├── edgespark.toml
├── package.json
└── src/
    ├── index.ts
    ├── defs/
    └── __generated__/
```

Full-stack projects usually look like:

```text
.
├── <agent-file>.md
├── configs/
├── edgespark.toml
├── server/
└── web/
```

`edgespark init` writes this layout. The default template in this repo is full-stack.

## `edgespark.toml`

Read this file first. It tells you which directories are authoritative for server and web code.

Typical full-stack shape:

```toml
project_id = "proj-abc123"

[server]
path = "server"

[web]
path = "web"
output_path = "web/dist"
```

## Generated Files

`src/__generated__/` is read-only.

Most important files:

- `edgespark.d.ts`: what may be imported from `edgespark` and `edgespark/http`
- `server-types.d.ts`: the actual SDK type contract and method signatures
- `sys_schema.ts` and `sys_relations.ts`: platform-managed schema references

Important: freshly scaffolded templates may contain placeholder generated files. Do not infer the real SDK from stubs. Pull types first when needed.

Treat the generated type files as the canonical contract before prose docs or old examples.

## Files To Read Before Coding

For server work:

1. `src/__generated__/edgespark.d.ts`
2. `src/__generated__/server-types.d.ts`
3. `src/defs/db_schema.ts`
4. `src/defs/db_relations.ts`
5. `src/defs/storage_schema.ts`
6. `src/defs/runtime.ts`
7. `src/defs/index.ts`
8. `src/index.ts`

For web work:

1. the scaffolded web agent file (for example `web/AGENTS.md`, `web/CLAUDE.md`, or `web/GEMINI.md`) or `templates/web/AGENTS.md`
2. `web/src/lib/edgespark.ts`
3. `node_modules/@edgespark/web/dist/index.d.ts` when installed

## Import Boundary

Application code may import runtime values from:

```ts
import { db, storage, vars, secret, ctx } from "edgespark";
import { auth } from "edgespark/http";
```

Definitions code under `src/defs/**` may import types from `@sdk/server-types`, but it must not import runtime SDK values from `edgespark` or `edgespark/http`.

## `src/defs/index.ts` Barrel Convention

`src/defs/index.ts` is the required barrel file for the server-side definitions layer.

- do not casually delete it
- do not remove required exports casually
- when changing schema, storage, or runtime defs, check whether the barrel still exports the right symbols

## Scaffolded Agent Files

The CLI can scaffold different top-level agent instruction files. In this repo the mapping is:

- `claude` -> `CLAUDE.md`
- `codex` -> `AGENTS.md`
- `gemini` -> `GEMINI.md`

Any other value falls back to `AGENTS.md`.
