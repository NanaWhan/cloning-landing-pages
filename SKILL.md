---
name: cloning-landing-pages
description: Use when the user wants to clone, copy, replicate, or "steal" the layout/design of an existing public web page into their own Next.js codebase to rebrand it — e.g. "make my site look like that", "clone this landing page", "turn this website into code I own". Uses the ditto.site MCP server.
---

# Cloning Landing Pages with ditto

## Overview

ditto (https://github.com/ion-design/ditto.site, MIT — by ion-design) captures
what a browser actually rendered on a public page and writes it back out as a
runnable Next.js codebase. It is not an AI guessing at a screenshot.

**This file does not restate the ditto tool schemas** — they are self-describing
and stay current when ditto ships; this file would not. Only what the schemas
cannot tell you is written here.

**Core principle:** clone the *layout*, then rebrand. Shipping someone else's
brand, logo, or copy back out is not cloning a layout — refuse that.

## When to Use

- User points at a public page and wants "mine to look like that"
- User wants a landing page's layout as editable Next.js code

**Hard limits:**
- Redeploying someone else's brand/logo/copy as-is → refuse
- Non-public or gated pages the user has no right to → refuse
- Backend behavior is never captured: forms, logins, payments, app logic

## Setup

Add ditto to the top-level `mcpServers` of `~/.claude.json` (on Windows,
`C:\Users\<you>\.claude.json`), with the key inline:

```json
"ditto": {
  "type": "http",
  "url": "https://api.ditto.site/mcp",
  "headers": { "Authorization": "Bearer dtto_live_..." }
}
```

`"type": "http"` is required. Restart Claude Code after editing — MCP servers
are read at startup — then confirm with `/mcp`.

*Alternative:* `"Bearer ${DITTO_API_KEY}"` reading from the environment. Cleaner,
but env expansion inside MCP `headers` is client-dependent; if it silently does
not expand you get a **401**, which is the first thing to check when auth fails.
The inline form above is the one verified working.

**Key hygiene.** Plaintext in a local uncommitted file is fine. Never paste it
into chat — transcripts persist to `~/.claude/projects/<project>/<session>.jsonl`
and stay readable long after the key is rotated. Rotate leaks at
https://www.ditto.site/api-key.

## Workflow

Tool order — read each tool's own schema for arguments:

```
clone_website  → jobId          (mode/framework/styling in options)
get_clone_status → poll until "succeeded"
get_clone_result → check capture.blocked and capture.pollution before trusting it
get_clone_bundle → { url, bytes, sha256 }
```

The read tools never return file contents, by design. The only way to get a
project on disk is the bundle:

```bash
curl -sS -L -o clone.tgz "<presigned url from get_clone_bundle>"
sha256sum clone.tgz     # must equal the sha256 the tool returned

# inspect before extracting
tar --force-local -tzf clone.tgz | grep -E '^/|\.\.' || echo "safe"

mkdir -p ./my-site
tar --force-local -xzf clone.tgz -C ./my-site
cd my-site && npm install && npm run dev
```

Three things the schema will not tell you:

- **`--force-local` is mandatory on Windows.** Without it GNU tar reads the `C:`
  in a path as a remote host and fails with `Cannot connect to C: resolve failed`.
- **The presigned URL needs no auth header** and expires in one hour.
- **The archive is flat at its root** — no top-level directory — so always
  extract into a directory you created, or it detonates into the current one.

## Rebranding — the actual point

The clone ships an `AGENTS.md` naming its own safe-to-edit areas; trust that
file over any list here. What that file won't tell you is that **a fresh clone
is still wearing the source's identity in four places people forget**, all of
which ship silently:

- **`src/app/layout.tsx` metadata.** Carries the source's `title`, its
  `openGraph.title`, its favicon, and — easiest to miss — `openGraph.siteName`
  set to the source's own domain.
- **Stray strings inside SVG `<title>` elements.** A real clone had
  `mediaval-fantasy-paper-scroll` sitting in `sections/navbar.tsx`, the source
  author's internal name for an icon. It never appears in the visible page, so
  reading the rendered site will not catch it — but it is in the markup, and it
  surfaces as a title in some contexts. Grep the source's brand across `src/`
  rather than trusting your eyes.
- **`public/assets/cloned/`.** The source's real image files. On a personal site
  that includes photographs of a person. Replace them; do not ship them.
- **`robots.ts`, `sitemap.ts`, `llms.txt/route.ts`.** Generated from the
  source's SEO identity and pointing at its routes.

Edit freely in `content.ts`, `sections/`, `components/`, and `svgs/`. Treat
`ditto.css` as load-bearing: small tweaks are fine, broad rewrites break the
fidelity you paid for. Leave `_cids.ts` and `_styles.ts` alone — they are anchor
plumbing, not content. `ditto-meta.ts` and `src/app/ditto/` are emitted only for
captures that need runtime interaction wiring; when present, don't touch those
either.

The rule that matters: **rebranding is not a find-and-replace on a company
name.** If the source is a personal site, the copy is a biography — someone's
family, their faith, their job history. Replacing that is a rewrite, not a
substitution, and it is the difference between using a layout and stealing an
identity.

## Choosing mode

`mode: "single"` captures only `/`. **Every nav link in the result then 404s** —
the markup keeps the original's hrefs, but no other route exists. That is easy
to mistake for a broken clone. Use `"multi"` when the user wants a navigable
site rather than one page.

## Verifying the clone

Do not compare total page height. Source pages animate and lazy-load; one real
page measured 5104px and 5783px on two consecutive loads, so a height mismatch
proves nothing on its own.

Measure the **distance between two landmarks** in both pages instead — pick two
pieces of text far apart, get `getBoundingClientRect().top + scrollY` for each,
and compare the deltas. If the source spans 958px between them and the clone
spans 960px, the layout is faithful regardless of what the totals say. A
constant offset between the two pages is normal; drift that *grows* with scroll
depth is the real signal of broken spacing.

## Known failure modes

**Content present in the DOM but not painted.** Seen on a real clone: below the
fold, large regions rendered as flat background colour while the source showed
text and illustrations. The DOM insisted all was well — `opacity: 1`, visible,
no transform/filter/clip, correct position, dark text on light, ancestors clean.
Ruled out and *not* the cause: occlusion by the absolute overlay image, stale
screenshots, service workers, and **a clean console** — no errors, no warnings,
a single React DevTools notice. **Unresolved.** Those five checks are spent;
start somewhere else.

**Debug order:** console → network → section source → DOM. The DOM is the most
expensive instrument and the last to reach for. Note that a clean console does
not clear the clone — it only means the failure is in paint, not in script.

## Common Mistakes

- **Rewriting a working MCP config** to match the env-var example above. If
  `/mcp` shows ditto connected, leave it alone.
- **Expecting file contents from the read tools.** Use the bundle.
- **Trusting `list_clones` as private.** It returns jobs that are not the user's
  (stripe.com, linear.app and similar have shown up). Never present that list as
  the user's own history.
- **Shipping the original brand.** Rebrand before anything public — and note the
  copy on a personal site is someone's actual biography, not filler.
