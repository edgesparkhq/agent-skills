# Auth Patterns

Auth is a managed service. The platform handles sessions, login flows, and user storage. Your app reads the current user through `auth` and configures providers declaratively via YAML.

Auth config lives in `configs/auth-config.yaml`. The schema is at the top of the file (`yaml-language-server` directive) — always check the schema for the latest supported fields.

Auth endpoints live under `/api/_es/auth/` — this is a platform-managed prefix used for callbacks, managed flows, and debugging. Normal browser app code should use `@edgespark/web`, not manual auth endpoint calls.

## Contents

- [Auth Config File](#auth-config-file)
- [Adding an OAuth Provider](#adding-an-oauth-provider)
- [OAuth Provider Reference](#oauth-provider-reference)
- [Callback URL](#callback-url)
- [Server-Side Auth](#server-side-auth)
- [Web-Side Auth](#web-side-auth)

## Auth Config File

Source of truth: `configs/auth-config.yaml`

Workflow:

1. `edgespark auth pull` to get current server config locally (or `edgespark auth get` to inspect)
2. Edit `configs/auth-config.yaml`
3. `edgespark auth apply` to validate and push
4. `edgespark deploy` to activate

Rules:

- `auth apply` validates against the schema — invalid configs are rejected with specific field errors
- deploy does not auto-apply local auth config; `edgespark auth apply` is always explicit
- changes take effect on the next deploy

## Adding an OAuth Provider

Every OAuth provider follows the same pattern. The steps must happen in this order:

1. **Create the OAuth app** with the external provider (GitHub, Google, etc.)
2. **Set the client ID as a var**:
   ```bash
   edgespark var set <PROVIDER>_CLIENT_ID=<value>
   ```
3. **Set the client secret as a secret**:
   ```bash
   edgespark secret set <PROVIDER>_CLIENT_SECRET
   ```
   This prints a URL — the user must open it in a browser and enter the secret value there. Secret values never pass through the CLI.
4. **Add the key names to `src/defs/runtime.ts`**:
   ```ts
   export type VarKey =
     | "<PROVIDER>_CLIENT_ID";

   export type SecretKey =
     | "<PROVIDER>_CLIENT_SECRET";
   ```
5. **Enable the provider in `configs/auth-config.yaml`** with `clientIdVarRef` and `clientSecretRef`:
   ```yaml
   provider<Name>:
     config:
       clientIdVarRef: <PROVIDER>_CLIENT_ID
       clientSecretRef: <PROVIDER>_CLIENT_SECRET
     enabled: true
   ```
6. **Apply and deploy**:
   ```bash
   edgespark auth apply
   edgespark deploy
   ```

Trap: `config: {}` with `enabled: true` will fail validation. You must provide `clientIdVarRef` and `clientSecretRef` when a provider is enabled.

## OAuth Provider Reference

Key names are fixed per provider — you cannot choose arbitrary names.

| Provider | YAML key | clientIdVarRef | clientSecretRef | Extra fields |
|----------|----------|----------------|-----------------|--------------|
| Google | `providerGoogle` | `GOOGLE_CLIENT_ID` | `GOOGLE_CLIENT_SECRET` | — |
| GitHub | `providerGithub` | `GITHUB_CLIENT_ID` | `GITHUB_CLIENT_SECRET` | — |
| Apple | `providerApple` | `APPLE_CLIENT_ID` | `APPLE_CLIENT_SECRET` | `appBundleIdentifier` |
| Microsoft | `providerMicrosoft` | `MICROSOFT_CLIENT_ID` | `MICROSOFT_CLIENT_SECRET` | `tenantId` |
| GitLab | `providerGitlab` | `GITLAB_CLIENT_ID` | `GITLAB_CLIENT_SECRET` | — |
| Discord | `providerDiscord` | `DISCORD_CLIENT_ID` | `DISCORD_CLIENT_SECRET` | — |

## Callback URL

When creating the OAuth app with the external provider, set the authorization callback URL to:

```
https://<your-domain>/api/_es/auth/callback/<provider>
```

Where `<provider>` is the lowercase provider name: `google`, `github`, `apple`, `microsoft`, `gitlab`, `discord`.

## Server-Side Auth

Path conventions control auth enforcement — no middleware needed:

| Path | `auth.user` | Use for |
|------|-------------|---------|
| `/api/*` | guaranteed non-null | protected routes |
| `/api/public/*` | user or `null` | optional auth |
| `/api/webhooks/*` | always `null` | external callbacks |

```ts
import { auth } from "edgespark/http";

// Protected — user guaranteed
app.get("/api/me", (c) => c.json({ id: auth.user!.id }));

// Public — check if logged in
app.get("/api/public/feed", (c) => {
  if (auth.isAuthenticated()) {
    // personalize for logged-in user
  }
});
```

## Web-Side Auth

Use `@edgespark/web` for all browser auth flows.

- Prefer `authUI.mount()` for managed login UI.
- For custom or headless flows, use `client.auth` directly. Its surface is Better Auth-compatible with EdgeSpark-specific additions like `requireSession()` and `onSessionChange()`.
- Do not implement app auth by calling `/api/_es/auth/*` endpoints manually.
- Raw `/api/_es/auth/*` endpoints are platform-managed and useful for debugging or verification, not normal browser app code.
- Treat auth route wiring details as implementation internals unless you are debugging the platform itself.

`authUI.mount()` automatically shows enabled providers (email/password + OAuth buttons).

```tsx
import { client } from "@/lib/edgespark";

const ui = client.authUI.mount(container, {
  redirectTo: "/dashboard",
});
// cleanup: ui.destroy()
```

For programmatic OAuth sign-in:

```ts
await client.auth.signIn.social({
  provider: "github",
  callbackURL: "/dashboard",
});
```

The managed auth UI picks up enabled providers automatically — no frontend changes needed when you add a new OAuth provider via the YAML config. Enable a provider in the YAML, apply, deploy, and the login page shows the new OAuth button. This is the declarative model: configure in YAML, not in code.

Important: `edgespark secret set` prints a secure URL. The human must open it in a browser to enter the secret value. Secret values never pass through CLI, agent context, or LLM API calls. Always show the URL to the user and tell them to complete the step in the browser.
