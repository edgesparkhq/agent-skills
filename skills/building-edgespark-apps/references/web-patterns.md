# Frontend Patterns

Do not use this file as a substitute for `node_modules/@edgespark/web/dist/index.d.ts`.

Read the installed package types and source directly for the exact interface. Use this reference only for small frontend patterns and traps.

## Canonical Browser Paths

Use these default paths unless the user explicitly asks for a lower-level debugging flow:

- Browser auth: `@edgespark/web`
- Managed auth UI: `client.authUI.mount()`
- Custom auth flows: `client.auth`
- Browser-to-app API calls: `client.api.fetch()`

Treat `/api/_es/auth/*` routes and lower-level provider wiring as platform internals for debugging, not normal app code.

## Contents

- [`src/lib/edgespark.ts` Singleton](#srclibedgesparkts-singleton)
- [API Calling Pattern](#api-calling-pattern)
- [Auth Page Pattern](#auth-page-pattern)
- [Route / Session Guard Pattern](#route--session-guard-pattern)
- [Upload Pattern](#upload-pattern)

## `src/lib/edgespark.ts` Singleton

Keep one app-level client module and import from it everywhere.

```ts
import { createEdgeSpark } from "@edgespark/web";
import "@edgespark/web/styles.css";

export const client = createEdgeSpark();
```

Then in app code:

```ts
import { client } from "@/lib/edgespark";
```

This avoids recreating the client in every component.

## API Calling Pattern

Use `client.api.fetch()` for app API requests.

```ts
import { client } from "@/lib/edgespark";

const res = await client.api.fetch("/api/posts");
const posts = await res.json();
```

Trap:

- do not replace this with plain `fetch("/api/...")` for normal app requests
- do not build browser auth by manually calling `/api/_es/auth/*`

## Auth Page Pattern

Use managed auth UI unless the user explicitly asks for custom forms.

```tsx
import { useEffect, useRef } from "react";
import { client } from "@/lib/edgespark";

export function LoginPage() {
  const ref = useRef<HTMLDivElement | null>(null);

  useEffect(() => {
    if (!ref.current) return;

    const ui = client.authUI.mount(ref.current, {
      redirectTo: "/dashboard",
    });

    return () => ui.destroy();
  }, []);

  return <div ref={ref} />;
}
```

Traps:

- `authUI.mount()` requires either `redirectTo` or `onSuccess`
- destroy the mounted UI on unmount
- use `client.auth` for custom forms instead of manually wiring platform auth endpoints

## Route / Session Guard Pattern

Use `onSessionChange()` for app-level auth state and `requireSession()` when a flow should hard-fail if unauthenticated.

```tsx
import { useEffect, useState } from "react";
import { client } from "@/lib/edgespark";
import type { AuthSession } from "@edgespark/web";

export function useAuth() {
  const [session, setSession] = useState<AuthSession | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const unsubscribe = client.auth.onSessionChange((nextSession) => {
      setSession(nextSession);
      setLoading(false);
    });

    return unsubscribe;
  }, []);

  return {
    user: session?.user ?? null,
    session,
    loading,
    isAuthenticated: !!session,
    signOut: () => client.auth.signOut(),
  };
}
```

For a hard guard:

```ts
await client.auth.requireSession();
```

Trap:

- build around `onSessionChange()` instead of inventing your own session polling

## Upload Pattern

Use a server-generated presigned URL flow and forward `requiredHeaders` exactly.

```ts
import { client } from "@/lib/edgespark";

const res = await client.api.fetch("/api/upload/presign", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    filename: file.name,
    contentType: file.type,
  }),
});

const { uploadUrl, requiredHeaders, path } = await res.json();

await fetch(uploadUrl, {
  method: "PUT",
  body: file,
  headers: {
    ...requiredHeaders,
    "Content-Type": file.type,
  },
});

await client.api.fetch("/api/upload/confirm", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ path }),
});
```

Trap:

- do not drop `requiredHeaders`; this is one of the easiest upload mistakes to make
