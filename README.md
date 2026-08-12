# Space Quotes

Short, sourced dispatches ("tidbits") from real FCC, ITU and FAA space filings. Each tidbit is
built around a verbatim pull-quote taken from the filing itself, verified against the primary
document before it is published.

**Live site:** https://spacequotes.org

## How it works

A zero-dependency Node (ESM) pipeline. Filing data comes from the
[Orbit Sentinel](https://viventine.com) API; the drafting and verification steps are driven by
the `/tidbit` skill in `.claude/skills/tidbit/SKILL.md`. The generated HTML is committed to the
repo and served by GitHub Pages.

```
content/tidbits/*.md   approved tidbits (the source of truth for the site)
out/<slug>/            per-tidbit review area (gitignored): factsheet, source text, draft
data/                  docket allowlist + published ledger (dedupe)
tools/                 the pipeline
```

## Tools

All four run from the repo root with no install step.

| Tool | What it does |
| --- | --- |
| `tools/find-candidate.mjs` | Finds a publishable filing. Enforces a hard gate (primary PDF + a hook) and returns a ranked shortlist. Modes: no args (rotates the `data/space-dockets.json` allowlist), `--docket 25-201`, `--q "deorbit"`, `--id <uuid>`. |
| `tools/build-factsheet.mjs <filing_id> <slug>` | Deterministic, no LLM. Writes `out/<slug>/factsheet.json` — structured fields as trusted facts, a complete docket tally, and the enriched summary quarantined as untrusted. Downloads the primary PDF and extracts `out/<slug>/source.txt`. |
| `tools/style-check.mjs [file ...]` | Voice linter for our editorial prose (not the verbatim quote): no em-dashes, no "Why it matters", no banned AI phrasings, no date-led openings. Exits nonzero on violations. Defaults to `content/tidbits/*.md`. |
| `tools/build-site.mjs` | Renders the whole static site from `content/tidbits/*.md`: homepage, tidbit permalinks, topic/docket hubs, about/terms, OG cards, RSS and sitemap. No network, no LLM. |
| `tools/publish.mjs <slug>` | Promotes a reviewed `out/<slug>/draft.md` to `content/tidbits/`, records it in the dedupe ledger, and rebuilds the site. Runs the style linter first and aborts if it fails. |

## From candidate to published

1. `node tools/find-candidate.mjs` — pick a filing off the shortlist.
2. `node tools/build-factsheet.mjs <filing_id> <slug>` — fact sheet + primary source text.
3. Draft `out/<slug>/draft.md` using only trusted facts and claims confirmed verbatim in
   `source.txt`. The centerpiece is a real line from the filing, quoted exactly and attributed.
4. `node tools/style-check.mjs out/<slug>/draft.md` — fix anything it flags.
5. Independent adversarial verification (a fresh agent that sees only the fact sheet, the source
   text and the draft) must return PASS. This step is mandatory.
6. `node tools/publish.mjs <slug>` — publish and rebuild, then review the generated files and
   commit.

## Requirements

- Node 18+ (uses the built-in `fetch`). No npm dependencies.
- `.env` at the repo root (gitignored), for the tools that hit the Orbit Sentinel API:
  ```
  MCP_API_URL=https://orbit-sentinel.viventine.com
  MCP_API_KEY=<key minted at https://console.viventine.com>
  ```
- ImageMagick 7 (`magick`) on `PATH` — renders the OG share cards. The build aborts if a card
  fails to render.
- `pdftotext` (poppler) on `PATH` — extracts primary-document text in `build-factsheet.mjs`.
- A bold, a regular and an italic TrueType font. `build-site.mjs` resolves macOS
  (Arial/Georgia) then common Linux paths (DejaVu, Liberation), and fails with the paths it
  tried if none are present.

## Local preview

`node tools/build-site.mjs`, then serve the repo root:

```bash
python3 -m http.server 8000
```

## License

MIT
