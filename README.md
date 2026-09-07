# com-etzhayyim-app-open-ossekai

Open Ossekai — L1/L2/L3 intelligence + Well-Becoming kyu/dan coaching actor.
OSINT briefs, arbitrage proposals, and consent-gated jocho scoring. It is the
Cloudflare Worker serving `open-ossekai.etzhayyim.com` and
`oss3k41x.etzhayyim.com`.

## Frontend migrated to ClojureScript (2026-09-07)

The `worker/svelte/` directory (SvelteKit) is gone. The frontend is now
ClojureScript — reagent + re-frame + `jp-go-dds` (デジタル庁デザインシステム) —
at [`worker/cljs/`](worker/cljs). This was a **frontend-only** migration; the
backend Worker/XRPC logic was moved, not rewritten:

| Then | Now |
|---|---|
| `worker/svelte/src/routes/+page.svelte` (the status page) | [`worker/cljs/src/open_ossekai/app.cljs`](worker/cljs/src/open_ossekai/app.cljs) — same fields, faithfully ported |
| `worker/svelte/src/routes/xrpc/[...path]/+server.ts` (the file that actually deployed, per `wrangler.jsonc`'s old `main`) | [`worker/src/xrpc-dispatcher.ts`](worker/src/xrpc-dispatcher.ts) — moved byte-for-byte, only a provenance header comment added |
| `worker/wrangler.jsonc` `main: svelte/.svelte-kit/cloudflare/_worker.js` | `main` dropped entirely |
| `worker/wrangler.jsonc` `assets.directory: ./svelte/.svelte-kit/cloudflare/client` | `assets.directory: ./cljs/public` |

Neither `worker/src/app.ts` nor the moved `worker/src/xrpc-dispatcher.ts`
calls `env.ASSETS.fetch`, so `main` was dropped rather than repointed at
either — putting either Worker in front of the static assets with no
`env.ASSETS.fetch` call would mean nothing serves the frontend. Both backend
files are orphaned source, not currently wired to any deploy target. See
`worker/wrangler.jsonc`'s header comment and `worker/src/xrpc-dispatcher.ts`'s
header comment for the full reasoning.

**This is unverified**: `wrangler deploy` / `wrangler dev` were not run
against this change.

Three fields in `worker/cljs/src/open_ossekai/app.cljs`'s `default-db` were
also corrected, not merely ported, against what the old Svelte constant
held — see that namespace's docstring for detail:

- `:app/route-count` / `:app/routes` now report the two route patterns
  `worker/wrangler.jsonc` actually declares, not the stale `0` / `[]` the
  Svelte constant carried.
- `:app/relative-path` now names the new cljs file, not the deleted Svelte
  one.
- `:app/xrpc?` is now `false` (the old constant held literal `true`),
  because `main` no longer deploys the XRPC handler (it is preserved,
  unwired, at `worker/src/xrpc-dispatcher.ts`).

## Build and test

```bash
cd worker/cljs
npm install
npm run build   # shadow-cljs compile app -> public/js/app.js
npm test        # shadow-cljs compile test && node out/tests.js
```
