# Server Patterns

Do not use this file as a substitute for `src/__generated__/server-types.d.ts`.

Read `edgespark.d.ts` and `server-types.d.ts` directly for the exact interface. Use this reference only for the non-obvious patterns that agents tend to get wrong.

## Canonical Server Paths

Default to these paths unless the user is explicitly debugging lower-level platform behavior:

- Read generated types first: `src/__generated__/edgespark.d.ts` and `src/__generated__/server-types.d.ts`
- Server auth: `auth` from `edgespark/http`
- Runtime config: declare in `src/defs/runtime.ts`, then read with `vars.get()` / `secret.get()`
- Client uploads: presigned PUT URL flow
- Client downloads: presigned GET URL flow
- File persistence: store `s3://...` URIs in the database, not client URLs

## Contents

- [Auth Path Semantics](#auth-path-semantics)
- [Vars and Secrets](#vars-and-secrets)
- [Storage: Smart Usage](#storage-smart-usage)
- [Database: Smart Usage](#database-smart-usage)
- [Request Context](#request-context)

## Auth Path Semantics

Path conventions matter:

- `/api/*` -> authenticated user is guaranteed
- `/api/public/*` -> user may be present or `null`
- `/api/webhooks/*` -> auth user is always `null`

Useful narrowing pattern:

```ts
if (auth.isAuthenticated()) {
  auth.user.id;
}
```

## Vars and Secrets

This reference does not restate the API surface. The important part is workflow:

- declare keys in `src/defs/runtime.ts` before using them
- use vars for plain remote config
- use secrets for sensitive values
- always handle missing values explicitly

## Storage: Smart Usage

The important part is not memorizing the methods. It is using them in the right pattern.

Rules:

- persist `S3Uri` in the database, not raw client URLs
- return presigned GET URLs to clients
- for large client uploads, create a presigned PUT URL and forward `requiredHeaders`
- check `null` after `get()` and `head()`
- `storage.parseS3Uri(row.column)` accepts `string` directly — no type guard needed
- avoid naming bucket exports the same as table exports (e.g. bucket `home_photos` not `application_photos` if a table already uses that name) — the barrel namespaces them as `buckets.x`, but identical names cause confusion in code

### Presigned Upload Flow

For client-originated uploads, generate a presigned URL on the server, return `requiredHeaders`, let the browser upload directly, then optionally confirm and store the reference.

```ts
app.post("/api/upload/presign", async (c) => {
  const { filename, contentType } = await c.req.json();
  const path = `uploads/${auth.user!.id}/${Date.now()}-${filename}`;

  const { uploadUrl, requiredHeaders } = await storage
    .from(buckets.uploads)
    .createPresignedPutUrl(path, 3600, { contentType });

  return c.json({ uploadUrl, requiredHeaders, path });
});
```

### File Reference Pattern

Store the S3 URI in the database, then convert it back to a presigned URL when returning data to the client.

```ts
app.post("/api/users/:id/avatar", async (c) => {
  const userId = parseInt(c.req.param("id"), 10);
  const data = await c.req.arrayBuffer();
  const path = `avatars/${userId}.jpg`;

  await storage.from(buckets.avatars).put(path, data, {
    contentType: "image/jpeg",
  });

  const s3Uri = storage.createS3Uri(buckets.avatars, path);
  await db.update(users)
    .set({ avatar_s3_uri: s3Uri })
    .where(eq(users.id, userId));

  const { downloadUrl } = await storage
    .from(buckets.avatars)
    .createPresignedGetUrl(path, 3600);

  return c.json({ avatarUrl: downloadUrl });
});

app.get("/api/users/:id", async (c) => {
  const userId = parseInt(c.req.param("id"), 10);
  const [user] = await db.select().from(users).where(eq(users.id, userId));
  if (!user) return c.json({ error: "Not found" }, 404);

  let avatarUrl: string | null = null;
  if (user.avatar_s3_uri) {
    const { bucket, path } = storage.parseS3Uri(user.avatar_s3_uri);
    const signed = await storage.from(bucket).createPresignedGetUrl(path, 3600);
    avatarUrl = signed.downloadUrl;
  }

  return c.json({ user: { ...user, avatarUrl } });
});
```

## Database: Smart Usage

The type signatures are already in the generated files. What agents usually need help with is the safe usage pattern.

Rules:

- use `.returning()` when you need inserted rows or IDs
- use `db.batch()` for atomic grouped work
- do not use `db.transaction()`
- use migration workflow for schema changes
- for SQL expression defaults (timestamps, etc.), use `` default(sql`(current_timestamp)`) `` not `.default("(datetime('now'))")` — the latter stores the literal string

### `db.batch()` Pattern

```ts
const [inserted, , allRows] = await db.batch([
  db.insert(posts).values({ title: "New", user_id: auth.user!.id }).returning(),
  db.update(posts).set({ title: "Renamed" }).where(eq(posts.id, 1)),
  db.select().from(posts),
]);
```

## Request Context

The main thing worth preserving here is usage, not interface listing:

```ts
ctx.runInBackground(sendAnalytics());
```

Use `ctx.runInBackground()`, not raw Worker plumbing in app code.
