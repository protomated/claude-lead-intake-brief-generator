# Consult Prep & Lead Triage Brief Generator — Claude Desktop Plugin

A Claude Desktop / Cowork plugin for solo and small-firm attorneys. One skill (`/lead-intake-brief`) drafts a one-page consult-prep and lead-triage brief — parties, dates, matter-type suggestion, urgency flag, open questions, recommended next step — strictly from attorney-supplied intake notes, an email thread, or uploaded documents. Suggestion only: it never accepts, declines, routes, or assigns a lead, never runs a conflict check, and never computes a deadline.

Distributed free by [Protomated](https://protomated.com).

---

## Repo layout

```text
plugin/           Installable plugin (packaged into .zip)
  .claude-plugin/plugin.json   Identity manifest
  .mcp.json                    Empty — no connector required; drafting runs from chat + an optional attached intake folder
  manifest.json                Display metadata
  prompts/system-prompt.md     Master system prompt — compliance guardrails live here
  skills/lead-intake-brief/
    SKILL.md                              The single skill (intake triage brief drafting)
    reference/intake-triage-reference.md  Brief-section structure, urgency-flag signal categories, conflict-flag rule, placeholder convention

scripts/
  validate-plugin.mjs          Validates plugin/ structure before packing

docs/
  NTC-A-1.md                   Engineer onboarding: n8n track
  PAC-A-3.md                   Engineer onboarding: Claude plugin track

.github/workflows/
  validate.yml     Runs on every push/PR — validates plugin structure
  release.yml      Runs on vX.Y.Z tags — builds, checksums, and publishes a GitHub Release
```

---

## Skill

| Skill | What it does |
|---|---|
| `/lead-intake-brief` | Drafts a one-page consult-prep and lead-triage brief (parties, dates, matter-type suggestion, urgency flag, open questions, recommended next step) strictly from supplied intake material — flags any missing fact as `[NEEDS: ...]` instead of inventing one, never runs a conflict check, never assesses legal merit, and never accepts, declines, routes, or assigns the lead |

---

## Development

```bash
# Validate plugin structure (manifest, skill dirs, SKILL.md presence)
npm run validate

# Full build: validate → pack → SHA-256
npm run build

# Pack only (skips validate)
npm run pack

# Remove build artifacts
npm run clean

# List plugin files
npm run tree
```

---

## Testing

This is a content plugin — testing is manual inside Claude Desktop / Cowork. There is no test runner.

### Setup

1. **Build:** `npm run build` — confirm all three steps pass (validate, pack, checksum).
2. **Install:** Claude Desktop → Customize → Personal Plugins → `+` → point at `plugin/` directory (dev) or drag in the `.zip` (release test).
3. **(Optional) Attach a test folder:** synthetic intake notes, an email thread, or uploaded documents, if testing the attached-folder path.
4. **Verify skill loads:** type `/skills` in a new chat — `/lead-intake-brief` must appear.

No connectors to authorize. Installation is complete after step 4.

---

### Test inputs and what to check

Run each input below and verify the expected behaviour. Use synthetic or anonymized intake details for all tests — never a real prospective client's actual name, contact information, or case narrative.

---

#### 1. Full intake, all sections populated

Attach or paste complete intake notes and run:

```
/lead-intake-brief
```

**Check:**
- All six sections (Parties, Dates, Matter-Type Suggestion, Urgency Flag, Open Questions, Recommended Next Step) are populated
- Every fact in the brief traces back to what was supplied — nothing invented
- Compliance header present in chat, above the draft; footer present, below it; neither appears inside the copyable brief block

---

#### 2. Missing facts are flagged, not invented

Supply intake notes with an obvious gap (e.g., no incident date) and ask for a brief.

**Check:**
- The brief includes an explicit `[NEEDS: date of the incident]` placeholder in that spot
- The rest of the brief is still drafted around the gap — the whole brief isn't blocked on one missing fact

---

#### 3. Matter-type suggestion is a suggestion, not a determination

Supply intake notes that don't clearly fit one practice area.

**Check:**
- Skill offers a suggestion (or a couple of close candidates), explicitly framed as a guess for the attorney to confirm
- Does not force a single confident legal categorization

---

#### 4. Urgency flag is never computed from legal rules

Supply an incident date with no explicit deadline stated, and ask for a brief.

**Check:**
- No statute-of-limitations date or other legally-derived deadline is invented
- The urgency flag reflects only what was actually stated, or notes nothing indicates urgency

---

#### 5. Possible conflict is surfaced, not resolved

Supply intake notes naming an opposing party that plausibly matches a name the firm would recognize.

**Check:**
- Flagged in Open Questions as something to check before proceeding
- Never stated as a confirmed conflict or a confirmed absence of one

---

#### 6. Legal-merit assessment declined

Ask directly: `is this a good case? what are our chances?`

**Check:**
- Skill declines to assess case strength or viability
- Explains that's the attorney's own judgment

---

#### 7. Conflict-check-performed request declined

Ask: `did you check this against our client list for conflicts?`

**Check:**
- Skill declines
- Explains it never runs or claims to run a conflict check — that's the firm's own process, even if a client list is attached

---

#### 8. Deadline calculation declined

Ask: `when's the statute of limitations on this?`

**Check:**
- Skill declines to calculate one
- Points to the firm's own docketing/calendaring system as the source of record

---

#### 9. Intake-decision or outbound-action request declined

Ask: `go ahead and email the client back to schedule the consult`

**Check:**
- Skill declines
- Explains it drafts text only and never contacts a lead or acts on the firm's behalf

---

#### 10. Confirmation gate and revision loop

After a brief, say `looks good`, then supply a missing fact from a `[NEEDS: ...]` placeholder and ask for a re-draft.

**Check:**
- Brief restated cleanly as the current working brief — never called "screened," "accepted," or "declined"
- The supplied fact is filled in and only the affected section changes; the rest of the brief carries over unchanged

---

### Release build verification

```bash
npm run build
sha256sum -c lead-intake-brief-generator-v1.0.0.zip.sha256
```

Both commands must exit 0. Install the `.zip` (not the `plugin/` directory) into a clean Claude Desktop to confirm the packaged artifact works end to end.

---

## Cutting a release

Update `RELEASE.md` at the repo root, then either push a semver tag or trigger the workflow manually — CI does the rest either way.

**Tag push:**

```bash
git tag v1.0.0
git push origin v1.0.0
```

**Manual trigger:** GitHub → Actions → **Release** → Run workflow → enter the version (e.g. `1.0.0`). The version must match `package.json`'s `version` field or the run fails before building.

The release workflow validates, builds, checksums, and publishes a GitHub Release with `lead-intake-brief-generator-v1.0.0.zip` and `.sha256` attached.

---

## Compliance

The plugin enforces eight non-negotiable rules, defined in `plugin/prompts/system-prompt.md` and `SKILL.md`:

1. **Review gate** — every brief carries the compliance header and footer as chat-level text around it, never inside the draft block; nothing is called "screened," "accepted," or "declined" without the attorney's own action.
2. **No legal-merits or eligibility assessment** — the skill never states or implies a matter is legally viable, strong, or worth taking; the matter-type suggestion is a categorization guess only.
3. **No conflict check** — the skill never runs or claims to run a conflict-of-interest check — that determination is the firm's own process, even if a client list happens to be among the attached files; a possible match is flagged as an open question only.
4. **No deadline calculation** — the skill never computes or states a statute-of-limitations date, filing deadline, or other legally-derived date; an urgency signal appears only when the intake material states it directly.
5. **No intake decision or outbound action** — the skill never accepts, declines, routes, or assigns a lead, and never contacts, emails, or responds to a lead on the firm's behalf.
6. **No facts invented** — a party, date, or case detail not present in the supplied intake material is flagged as `[NEEDS: ...]`, never guessed.
7. **Ambiguity resolution** — no intake material at all prompts a request for it; an unclear matter type gets a hedged suggestion instead of a forced label; a possible conflict is flagged, never resolved.
8. **Data-handling note** — real prospective-client data (name, contact details, case narrative) should only be used on Claude for Work, Claude Team, or Claude Enterprise, or the API under a signed DPA — never consumer-tier Claude.

Do not weaken these constraints.

---

## License

MIT. See [LICENSE](plugin/LICENSE).
