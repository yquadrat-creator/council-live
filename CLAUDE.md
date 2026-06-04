# CLAUDE.md

Guidance for AI assistants (Claude Code) working in this repository.

## What this repo actually is

`council-live` is a **tiny static-hosting repo**, not the application itself. It
contains two hand-written HTML files and nothing else — no build system, no
package manager, no dependencies, no CI.

```
index.html   # redirect stub
stats.html   # redirect stub (near-identical to index.html)
```

Both files do the same thing: immediately redirect the browser to the **live
dashboard** hosted on Railway:

```
https://hermes-trading-production-1c86.up.railway.app/stats
```

The redirect uses three belt-and-suspenders mechanisms so it fires as fast as
possible on mobile and never serves a stale cache:

1. `<script>location.replace(...)</script>` — fastest, runs before paint.
2. `<meta http-equiv="refresh" content="0;url=...">` — fallback if JS is off.
3. `<meta http-equiv="Cache-Control" content="no-cache, no-store, must-revalidate">`
   — prevents the stub from being cached.

The visible `<body>` ("opening the dashboard…") is only shown for the split
second before the redirect lands.

## The most important thing to understand

**The real application — "The Council and the Brain" trading dashboard — does
NOT live in this repo.** It is a separate service deployed on Railway
(`hermes-trading-production`). This repo is intentionally a thin redirect layer.

This is a deliberate design decision. See commit history:

- `stats.html: always mirror the live Railway dashboard (no more drift/stale copies)`
- `stats: redirect straight to live Railway dashboard (drop iframe — faster on mobile)`
- `remove 3D characters/effects site — redirect to the single live dashboard`

Earlier the repo contained a full **self-contained single-file HTML app** (a 3D
"council amphitheater" built on a `<canvas>`, plus a neon mobile stats page —
all CSS/JS inlined into one `index.html` of ~900 lines). That was deleted in
favor of redirecting to the single canonical live dashboard, so the hosted
pages can never drift out of sync with what the trading service actually serves.

### Do NOT, unless the user explicitly asks

- Do not re-add a local copy of the dashboard UI to this repo. The whole point
  of the current setup is to avoid a second, drifting copy.
- Do not add a build step, framework, or `node_modules`. There is no toolchain
  here and adding one contradicts the repo's purpose.
- Do not try to edit dashboard features (charts, trade cards, gauges, the 3D
  council, etc.) here — that code lives in the Railway service, not in this repo.

If the user wants dashboard changes, clarify that those belong in the
`hermes-trading` service and aren't editable from this repository.

## Working on the redirect stubs

The only legitimate edits in this repo are to the two redirect stubs. Things you
might reasonably be asked to change:

- The **redirect target URL** (if the Railway deployment URL changes). Update it
  in **both** files, in **both** the `<script>` and the `<meta refresh>` lines.
- The loading message, fonts, or background color of the brief loading screen.
- Adding another redirect stub for a new route.

`index.html` and `stats.html` are identical except for the loading text
("opening the dashboard…" vs "opening the live dashboard…"). Keep them in sync
when changing redirect behavior.

### Conventions for these HTML files

- **Single self-contained file.** Everything inline — no external CSS/JS/asset
  references. Keep it that way.
- **Mobile-first.** The dashboard is consumed primarily on phones; the meta
  viewport tag and small loading screen reflect that. Prefer changes that keep
  first paint fast on mobile.
- **Dark neon aesthetic.** Background `#05060d`, muted blue text `#8aa0c8`,
  system font stack. Match this if you add UI.
- **No cache.** Preserve the no-cache meta tag on any stub so redirects always
  resolve to current content.

## Verifying changes

There is no test suite or linter. To verify a change:

- Open the edited `.html` file in a browser (or read it) and confirm the
  redirect URL is correct and identical in the `<script>` and `<meta refresh>`.
- Optionally check the live target is reachable:
  `curl -sI https://hermes-trading-production-1c86.up.railway.app/stats`

## Git workflow

- Active development branch for AI-assisted work: `claude/claude-md-docs-kp2p8`
  (create locally if missing). Do not push to `main` without explicit
  permission.
- Push with `git push -u origin <branch-name>`.
- Commit messages in this repo are **lowercase, concise, and outcome-focused**,
  often using a `area: change` prefix. Match that style. Examples from history:
  - `stats: redirect straight to live Railway dashboard (drop iframe — faster on mobile)`
  - `dashboard: margin-level gauge`
  - `rebrand display text: Hermes Institute -> The Council and the Brain`
- Do **not** open a pull request unless the user explicitly asks for one.

## Naming / branding notes

- The product is branded **"The Council and the Brain"** (page `<title>`).
- It was previously called **"Hermes Institute"** — the Railway service name
  (`hermes-trading`) still reflects the old name. Don't be confused by the two
  names referring to the same system.
