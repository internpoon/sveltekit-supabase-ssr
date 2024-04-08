# MVP for "SSalSA" stack

SvelteKit, Supabase, and lovely SSR Auth

## Code Showcase

- Email sign-up/sign-in.
- GitHub sign-in. Can easily be changed to other oauth providers.
- Requires a session to access all pages under the `authenticated` layout group.

> All sign-up and sign-ins happen server-side.

## Install

> Tweak your Sveltekit creation as desired.

```
npm create svelte@latest sveltekit-supabase-ssr
  > Skeleton project
  > Yes, using Typescript syntax
  > none

cd sveltekit-supabase-ssr
npm install
```

## Setup

1. Supabase types
    ```
    supabase init
    supabase link --project-ref <your-project-id>
    supabase gen types typescript --linked > src/lib/database.d.ts
    ```

2. Environment variables.
    
    Create a `src/env.local` file.
    ```
    PUBLIC_SUPABASE_ANON_KEY=<your-project-anon-key>
    PUBLIC_SUPABASE_URL=https://<your-project-id>.supabase.co
    ```

3. Change email templates, per [official docs](https://supabase.com/docs/guides/auth/server-side/email-based-auth-with-pkce-flow-for-ssr?framework=sveltekit#update-email-templates-with-url-for-api-endpoint)

## Run!

```
npm run dev
```

Open a browser to http://localhost:5173

## Security

Within the `(authenticated)` layout group, we have a `+page.server.ts` file for each route. This ensures that even during client navigation we can verify there's still a session before rendering the page.

We do two things to verify the session on the server:

1. Check for a session object using `getSession()`. This should not be the only thing we rely on to verify the request is authenticated, because sessions are stored in a cookie sent from a client. And the client could be an attacker with just enough information to get past the session check and possibly render data about a victim user. e.g. `{ access_token: 'got', refresh_token: 'ya', expires_at: 2000000000, user: { id: 'valid-victim-user-id' } }`. And, in this example, if our code uses `id` to render sensitive information, or we're using `getSession()` as an authentication guard, this could lead to a security breach or unexpected behavior for valid users (see [here](https://github.com/supabase/auth-helpers/pull/722) for more details.)
2. We verify that the `access_token` within the session was signed by our Supabase project. This avoids the vulnerability above, because we now know the session and associated data is from a known source.
