# cloning-landing-pages

A [Claude Code](https://claude.com/claude-code) skill for cloning a public web
page's layout into a Next.js codebase you own, then rebranding it.

It wraps [**ditto**](https://github.com/ion-design/ditto.site) by
[ion-design](https://github.com/ion-design) (MIT) — ditto does the actual
capture and code generation. This repo is just the operating manual: the things
that break in practice, which the tool's own docs don't tell you.

## What's in here

`SKILL.md` deliberately does **not** restate ditto's MCP tool schemas. Those are
self-describing and stay current when ditto ships; a copy in a markdown file
would not. What it documents instead is the hard-won part:

- The MCP config shape that actually authenticates, and what a 401 really means
- The bundle download path — the read tools never return file contents
- `tar --force-local`, without which extraction fails outright on Windows
- Why `mode: "single"` leaves every nav link 404ing, which reads as a broken clone
- A fidelity check that works, and why comparing total page height doesn't
- A known unresolved rendering failure, with the dead ends already ruled out

## Install

Clone it straight into your skills directory:

```bash
git clone https://github.com/NanaWhan/cloning-landing-pages.git \
  ~/.claude/skills/cloning-landing-pages
```

Claude Code discovers it automatically. Ask it to clone a landing page and the
skill loads itself.

## Prerequisites

- Node 20+
- A ditto API key from https://www.ditto.site/api-key
- The ditto MCP server configured — see the Setup section in `SKILL.md`

## A note on what this is for

Cloning a *layout* to rebrand it is how design has always worked. Republishing
someone else's brand, logo, or copy is not, and the skill refuses that. Be
especially careful with personal sites: the text on them is somebody's actual
biography, not placeholder filler.

## Author

**Cherish Banini Seinty** — [cherishseinty.dev](https://www.cherishseinty.dev/)
· [@NanaWhan](https://github.com/NanaWhan)

Everything in `SKILL.md` came out of actually running this end to end — the
Windows tar failure, the 404ing nav, the unresolved paint bug. It's a field
report, not a summary of the docs.

## License

MIT — see [LICENSE](LICENSE). ditto itself is separately MIT-licensed by
ion-design.
