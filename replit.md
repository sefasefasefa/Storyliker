# Instagram API Explorer

A developer tool for security researchers and engineers to inspect and test Instagram's undocumented web API endpoints in real time. Features a dark terminal-inspired UI with request history, session management, and interactive JSON response inspection.

## Run & Operate

Four services run as managed Replit workflows (started automatically). Note: these artifacts were imported with pre-existing `artifact.toml` files, so their directory names carry a `-src` suffix (a byproduct of the import/registration process) even though the package names and workflow titles are clean:

- `artifacts/instagram-explorer-src: web` — frontend, port 18900, served at `/` (package `@workspace/instagram-explorer`)
- `artifacts/api-server-backup-src: API Server` — Express API proxy, port 8080, served at `/api` (package `@workspace/api-server`)
- `artifacts/ig-automation-ui-src: web` — a second, separate frontend "Instagram Otomasyon Paneli" (automation panel), served at `/ig-automation-ui/` (package `@workspace/ig-automation-ui`)
- `artifacts/mockup-sandbox: Component Preview Server` — canvas component preview, port 8081, served at `/__mockup`

To restart a service manually, use the Replit workflow restart tool with the exact workflow name above (do not run the dev commands directly — the managed workflows inject required `PORT`/`BASE_PATH` env vars).

## Other content in this import

- `İnstagram Follewer Puller/` — an unrelated, standalone Python toolkit (not part of the pnpm workspace) with Instagram follower-scraping/analysis scripts (`instagramFollowerPuller.py`, `yenipuller.py`) and a `buildozer.spec` suggesting it was once packaged as a mobile app. Left as-is; not wired into any workflow.
- `main.py` at the project root — leftover generic Replit placeholder ("Hello from repl-nix-workspace!"), unused by the pnpm workspace apps.

- `pnpm run typecheck` — full typecheck across all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- API: Express 5 (proxy server for Instagram endpoints)
- Frontend: React + Vite + Tailwind v4 + shadcn/ui
- State: In-memory session + request history (single-user tool)
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)

## Where things live

- `lib/api-spec/openapi.yaml` — OpenAPI spec (source of truth for all contracts)
- `artifacts/api-server-backup-src/src/lib/instagram.ts` — Instagram API proxy client (REST + GraphQL)
- `artifacts/api-server-backup-src/src/lib/session.ts` — In-memory session store + header builder
- `artifacts/api-server-backup-src/src/lib/history.ts` — In-memory request history (last 100 entries)
- `artifacts/api-server-backup-src/src/routes/instagram.ts` — Proxy route handlers
- `artifacts/instagram-explorer-src/src/pages/` — Frontend pages (dashboard, profile, post, graphql, hashtag, stories, session)
- `artifacts/ig-automation-ui-src/` — separate "Instagram Otomasyon Paneli" automation frontend, served at `/ig-automation-ui/`

## Architecture decisions

- Single-user tool: session and history are global in-memory state (intentional for personal dev tool)
- CORS locked to localhost + *.replit.dev to prevent cross-origin abuse of shared session state
- Upstream HTTP status codes are propagated from Instagram to client (not always 200)
- Dark mode is forced globally via `document.documentElement.classList.add('dark')` in main.tsx (Tailwind v4 doesn't support `@apply dark`)
- GraphQL endpoints use Instagram's doc_id pattern (pre-compiled server-side queries), not freeform GraphQL

## Product

Pages:
- `/` — Dashboard: session status, endpoint modules, network request log
- `/profile` — Profile explorer: search by username, view follower/following stats, bio, user ID
- `/post` — Post inspector: fetch by shortcode, view media URLs, comments, likes
- `/graphql` — GraphQL builder: select known doc_ids or enter custom, edit variables JSON, inspect raw response
- `/hashtag` — Hashtag explorer: paginated posts by tag
- `/stories` — Stories tray (requires authenticated session)
- `/session` — Session manager: set sessionId + csrfToken, view security headers reference

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Gotchas

- Tailwind v4: cannot use `@apply dark` — add the `dark` class to `document.documentElement` via JS instead
- OpenAPI codegen TS2308 collision: endpoints with BOTH path params AND query params generate `*Params` types in both `api.ts` and `types/` — workaround: move path params to query params for paginated endpoints
- Instagram endpoints require specific headers (X-IG-App-ID, X-ASBD-ID etc.) even for public endpoints — see `lib/session.ts:buildInstagramHeaders()`
- The PolarisPostRootQuery doc_id changed to `27128499623469141` in June 2026 (old: `8845758582119845`)

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
