# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

**CP4** — Consult Prep & Lead Triage Brief Generator. A Claude Desktop / Cowork plugin for solo and small-firm attorneys. One skill (`/lead-intake-brief`) drafts a one-page brief — parties, dates, matter-type suggestion, urgency flag, open questions, recommended next step — strictly from intake notes, an email thread, or uploaded documents the attorney supplies or attaches. It never assesses legal merit or case strength, never runs or claims to run a conflict-of-interest check, never accepts, declines, routes, or assigns a lead, and never contacts the lead itself — every output is a suggestion for the attorney to review and act on. There is no runtime code, no MCP server, no connector, and no backend. The product is entirely content: a markdown skill file and a reference doc.

The leak it plugs: firms that respond to an inbound lead within an hour convert significantly more of them than firms that take a day or two, but solo and small firms have no dedicated triage staff, so qualification happens inconsistently, squeezed between everything else on an attorney's desk. This skill turns raw intake material into a scannable brief in the time it takes to read it, so the attorney walks into the consult already oriented. It complements — without depending on or naming at runtime — the catalog's n8n-track AI Intake Qualifier workflow: that's the always-on, automated routing pipeline; this is the on-demand, single-lead, Claude-native version, and it stays a suggestion tool by design — the automated triage-and-route version, with a conflict check and CRM write-back, is explicitly the paid upsell behind this free skill.

Landing page: `protomated.com/templates/lead-intake-brief-generator/` (WordPress — managed outside this repo).

## Working a new ticket (this repo is a per-ticket template)

This repo is not owned by any single ticket — it's reused for every plugin in the PAC catalog. Each new build **replaces the current plugin's content in place**, same repo, same git history, no "repo is now for ticket X" migration. **Never write the word "replace" (or otherwise describe the swap) in a commit message, README, RELEASE.md, or code comment.** Every build should read as if it were authored fresh for its own ticket, not as a diff against whatever was here before. There is also an invocable `/work-pac-ticket` skill (`.claude/skills/work-pac-ticket/`) that runs this recipe end to end for a given ticket number.

**Starting a ticket:**

1. Look up the ticket in Nifty — project niceId `PAC` (project id `FivxoeVG9E`). Fetch the ticket by `niceId`, then its parent epic (`parentTaskId`, normally PAC-61) and any linked/dependency tasks — a single ticket's description is often incomplete without the epic's framing.
2. Read the three fixed reference docs before drafting anything; they don't change per ticket: `docs/NTC-A-1.md` (n8n track onboarding — shared pillar/leak-test framing), `docs/PAC-A-3.md` (Claude plugin track onboarding — the "assisted draft, always reviewed" rule this whole catalog runs on), and `docs/how-to-add-a-new-lead-magnet-template.md` (how the landing page gets published afterward, and why the build format decides the compliance note).
3. Confirm the ticket's plugin name, skill slug, and CP number (from the ticket title/custom fields) with the user before rewriting if any of them are ambiguous — don't guess.
4. If the ticket description or comments point at another catalog plugin for shared design rationale, resolve it — however it's referenced: an explicit `PAC-N`; a CP-number shorthand ("CP7," "C7" — every ticket title in this catalog follows `CP<N>: <Display Name>`, searchable via Nifty full-text search scoped to this project when only the shorthand or name is given); or just the plugin's name with no number at all. Once resolved to a PAC-N, read its `Plugin Repo/Marketplace URL` custom field, or ask the user for that ticket's repo URL/path if the field is empty or the search is ambiguous. Judge incidental mentions (a doc citation, a "replaces X" note) separately — those don't need resolving. Treat whatever you do pull in as read-only research — never a dependency, submodule, or copied file in this repo.

**What gets rewritten vs. left alone:**

Rewritten per ticket: the ticket-specific sections of this file (`What this repo is` above, and the skill/compliance sections below — keep sections like this one and "Commit style"), root `README.md`, `RELEASE.md`, `package.json`, `plugin/.claude-plugin/plugin.json`, `plugin/manifest.json`, `plugin/CONNECTORS.md`, `plugin/README.md`, `plugin/prompts/system-prompt.md`, `plugin/skills/<old-slug>/` → `plugin/skills/<new-slug>/` (delete the old directory rather than leaving both), `tests/skills/<old-slug>.md` + its fixture folder → `tests/skills/<new-slug>.md` + fixtures, `.github/workflows/release.yml` (zip filename + release title).

Left alone: `plugin/LICENSE`, root `LICENSE`, `docs/NTC-A-1.md`, `docs/PAC-A-3.md`, `docs/how-to-add-a-new-lead-magnet-template.md`, `.github/workflows/validate.yml`, `.claude/skills/`.

**Before committing:** run `npm run clean && npm run build` — it should validate, pack, and checksum cleanly — then grep the tree for the previous plugin's name/slug/skill terminology to catch anything left behind. Once the user confirms the build is ready, commit as a single `Build PAC-<N> CP<M>: <Display Name> Skill plugin` commit per "Commit style" below — same standing rule as everywhere else in this project: don't commit unless the user has asked for it. Do not push, tag, or run `npm run release` without the user's explicit go-ahead either.

## Repo layout

```
.claude/skills/work-pac-ticket/SKILL.md   Runs "Working a new ticket" above end to end — stable, not rewritten per ticket
plugin/           The installable plugin (packaged into .zip bundle)
  .claude-plugin/plugin.json   Manifest validated by scripts/validate-plugin.mjs
  .mcp.json                    Empty — filesystem access is Cowork's implicit attached-folder model, not a connector
  manifest.json                Plugin display metadata
  prompts/system-prompt.md     Master system prompt — compliance guardrails live here
  skills/lead-intake-brief/
    SKILL.md                              The single skill; YAML frontmatter + markdown body
    reference/intake-triage-reference.md  Brief-section structure, urgency-flag signal categories, conflict-flag rule, and the `[NEEDS: ...]` placeholder convention
scripts/
  validate-plugin.mjs          Validates plugin/ structure before packing
docs/
  NTC-A-1.md, PAC-A-3.md      Engineer onboarding reference docs
```

## Commands

All commands run from the repo root.

```bash
# Validate plugin structure (manifest, skill dirs, SKILL.md presence)
npm run validate

# Full build: validate → pack → SHA-256 → artifact
npm run build

# Pack only (skips validate)
npm run pack

# Cut a GitHub release (runs build first; requires RELEASE.md at repo root)
npm run release

# Remove build artifacts
npm run clean

# List plugin files (excludes node_modules)
npm run tree
```

## Plugin format

The bundle format is `.zip`. It uses the **plugin variant** (not standalone) — no bundled MCP server, no connectors. Plugin name: `lead-intake-brief-generator`, current version: `1.0.0`.

Two manifests serve different purposes:
- `plugin/.claude-plugin/plugin.json` — the identity manifest the validator and Claude Desktop read (`name` must be kebab-case)
- `plugin/manifest.json` — display metadata only (no `server` block — this is a plugin variant, not standalone)

The validator (`scripts/validate-plugin.mjs`) checks:
- `.claude-plugin/plugin.json` is valid JSON with a kebab-case `name`
- Each `skills/*/` subdirectory contains a `SKILL.md`
- `agents/`, `commands/`, `hooks/` (if present) contain files with the expected extension

## Skill: /lead-intake-brief

The single skill drafts one kind of output — a one-page consult-prep and lead-triage brief — always from what the attorney supplies: intake notes, an email thread, or uploaded documents, typed into chat or attached as a workspace folder:

1. Asks for the intake material if nothing has been supplied yet — a brief needs something to summarize.
2. Drafts six sections: **Parties**, **Dates**, **Matter-Type Suggestion**, **Urgency Flag**, **Open Questions**, **Recommended Next Step** — see `reference/intake-triage-reference.md` for what each structurally contains.
3. Never invents a party, date, or case detail not present in the attached intake material or chat input; a missing fact is left as an explicit `[NEEDS: ...]` placeholder, and the rest of the brief is still drafted around it.
4. Never assesses legal merit, case strength, or eligibility — the matter-type suggestion is a categorization guess for the attorney to confirm, never a legal opinion on whether the case is viable or worth taking.
5. Never runs or claims to run a conflict-of-interest check — a party that might match an existing client is surfaced in Open Questions as something to check, never resolved as a confirmed conflict or a confirmed absence of one.
6. Never calculates or states a statute-of-limitations date, filing deadline, or other legally-derived date; the urgency flag reflects only what the intake material states directly, otherwise it's flagged for the attorney or the firm's docketing system to confirm.
7. Never accepts, declines, routes, or assigns a lead, and never contacts, emails, or responds to the lead itself — the recommended next step is always a suggestion; the attorney or firm staff makes and carries out the actual decision.
8. Presents every brief with the compliance header and footer as chat-level text around it — never embedded inside the copyable draft block, since that block is what the attorney may paste into a CRM or matter-management note.
9. Iterates on corrections and `[NEEDS: ...]` placeholder fill-ins as many times as needed; never marks a brief "screened," "qualified," "accepted," or "declined" — that's the attorney's own action.

Each `SKILL.md` has YAML frontmatter:
```yaml
---
name: skill-name
description: shown to attorney in /skills list
argument-hint: "[hint shown in Claude Desktop]"
---
```

## Compliance constraints — non-negotiable

These rules are enforced in `prompts/system-prompt.md` and `SKILL.md`. Do not weaken them:

1. **Review gate**: Claude must present every brief with the compliance header and footer as chat-level text around it, never inside the draft block, and never call a brief "screened," "qualified," "accepted," or "declined" without the attorney's own action.
2. **No legal-merits or eligibility assessment**: the skill never states or implies that a matter is legally viable, strong, or worth taking. The matter-type suggestion is a categorization guess for the attorney to confirm, never a legal opinion.
3. **No conflict check**: the skill never runs or claims to run a conflict-of-interest check — that determination is the firm's own process, even if a client list happens to be among the attached files. If a named party might match an existing client, it's flagged in Open Questions as something to check — never resolved as a confirmed conflict or a confirmed absence of one.
4. **No deadline calculation**: the skill never computes or states a statute-of-limitations date, filing deadline, or other legally-derived date. An urgency signal appears in a brief only when the intake material states it directly (an explicit deadline, the lead's own expressed urgency, a mentioned upcoming date); otherwise it's flagged (`[NEEDS: confirm applicable deadline — attorney/docketing system]`) for the attorney or the firm's docketing system to confirm.
5. **No intake decision or outbound action**: the skill never accepts, declines, routes, or assigns a lead, and never contacts, emails, or responds to the lead on the firm's behalf. Every recommended next step is a suggestion; the attorney or firm staff makes and carries out the actual decision.
6. **No facts invented**: a party, date, or case detail not present in the attached intake material or chat input is flagged as `[NEEDS: ...]`, never guessed or filled in with a "typical" detail.
7. **Ambiguity resolution**: no intake material at all prompts a request for it before drafting anything; a matter type that isn't clear from the intake material gets a hedged suggestion (or a couple of close candidates) instead of a forced single label; a possible conflict is flagged, never resolved.
8. **Data-handling note**: The system prompt must warn that real prospective-client data — name, contact details, case narrative — should only be used on Claude for Work, Claude Team, or Claude Enterprise, or the Claude API under a signed Data Processing Agreement (DPA), never consumer-tier Claude (claude.ai Personal / Pro).

## Internal QA fixtures — tests/skills/

`tests/skills/<skill-name>.md` is the internal QA testing guide for a skill — a standing convention for every plugin built in this repo, alongside (not replacing) the end-user testing guide in `plugin/README.md`. The difference:

- `plugin/README.md` — ships inside the plugin zip, short scenarios with pasted one-liners, aimed at an attorney verifying the install.
- `tests/skills/<skill-name>.md` — internal only, not packaged, uses real attached-folder fixtures under `tests/skills/<skill-name>/` for the intake-material path, since this skill's primary input is intake notes, an email thread, or documents attached as a case folder, not a single file to read cold. Deeper checks (e.g., compliance-wrapper placement, the `[NEEDS: ...]` placeholder rule, legal-merits and conflict-check refusal, deadline-calculation refusal) belong here even when they overlap with `plugin/README.md`'s scenarios.

All fixture data must be clearly synthetic — fictional firms, clients, and matter details. Never use real client or matter data, even anonymized real data, without checking with Dele first. Intake material can include a prospective client's name, contact details, and case narrative, so fixture hygiene matters here the same way it does across the rest of this catalog.

## Commit style

Do not include `Co-Authored-By` attribution lines in commit messages.

## Canonical plugin description

Used in `plugin/.claude-plugin/plugin.json` and any marketing copy — keep consistent:

> A consult-prep and lead-triage assistant that turns attorney-supplied intake notes, an email thread, or uploaded documents into a one-page brief — parties, dates, matter-type suggestion, urgency flag, open questions, recommended next step. Suggestion only: never accepts, declines, routes, or assigns a lead, never runs a conflict check, and never computes a deadline; you review and decide before acting on any lead.

## Testing

Testing is manual inside Claude Desktop / Cowork — there is no test runner. The `plugin/README.md` is the canonical testing guide. It contains:
- Setup steps (build → install → optionally attach a test intake folder → verify skill loads)
- 10 specific test inputs with exact text to paste and what to check for each

Key scenarios that must pass: a full brief drafted from complete intake material (all six sections populated, everything traceable to what was supplied), missing facts flagged as `[NEEDS: ...]` rather than invented, an ambiguous matter type hedged rather than forced into one label, a possible-conflict flag surfaced as an open question rather than resolved, a deadline never computed, a legal-merits-assessment request declined, a conflict-check-performed request declined, an intake-decision-or-outbound-action request declined, confirmation gate (nothing marked screened/accepted/declined until the attorney says so), and a revision loop that fills a placeholder without disturbing the rest of the brief.

## Notes

- `plugin/.mcp.json` is `{}` — this plugin requires no connector. Drafting runs in chat, with optional intake material (notes, an email thread, or documents) attached via Cowork's implicit attached-workspace-folder model, which needs no separate config.
- `plugin/manifest.json` has no `server` block — the plugin variant does not require one. Do not add one.
- `plugin/README.md` and `plugin/CONNECTORS.md` are end-user documentation included in the ZIP bundle; they are not internal developer docs.
- The root `.mcp.json` is gitignored — it holds workspace-level Claude Code MCP credentials and is not part of the plugin artifact.
- `npm run release` passes `--notes-file RELEASE.md` to `gh release create` — create/update `RELEASE.md` at repo root before running it.
- Package scripts use `$npm_package_name` and `$npm_package_version` — keep the `name` field in `package.json` in sync with the plugin slug.
