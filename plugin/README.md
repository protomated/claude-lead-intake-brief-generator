# Consult Prep & Lead Triage Brief Generator — Claude Desktop Plugin

A Claude Desktop / Cowork plugin for solo and small-firm attorneys. One skill (`/lead-intake-brief`) turns intake notes, an email thread, or uploaded documents into a one-page brief before a consult — parties, dates, a matter-type suggestion, an urgency flag, open questions, and a recommended next step. Suggestion only: it never accepts, declines, routes, or assigns a lead, never runs a conflict check, and never computes a deadline.

**Distributed by [Protomated](https://protomated.com) as a free download.**

---

## ⚠️ Required: Read This Before You Triage a Real Lead

**This section is not boilerplate. Read it before pasting or attaching real intake material.**

### 1. You must be on a qualifying Claude plan

Do NOT attach a real intake file — a prospective client's name, contact details, and case narrative — on a consumer Claude plan (claude.ai Personal or Claude Pro). Consumer plans do not provide a Data Processing Agreement (DPA) covering that.

Use one of the following:

- **Claude for Work** (formerly Claude.ai Teams)
- **Claude Team or Enterprise**
- **Claude API** (with a signed DPA from Anthropic)

> **If you're not sure which plan you're on:** Open Claude Desktop → Help → About. If it says "Claude Pro," you are on a consumer plan. Upgrade to Claude for Work first.

### 2. It drafts from what you supply — nothing more

Every party, date, and detail in the brief comes from the intake notes, email thread, or documents you provide. A missing fact appears as an explicit `[NEEDS: ...]` placeholder, never a guess.

### 3. It never assesses whether the case is worth taking

This plugin does not evaluate legal merit, case strength, or eligibility. Its matter-type suggestion is a categorization guess for you to confirm, not a legal opinion — and it never runs a conflict check, so a possible match to an existing client shows up as a question for you to check, not an answer.

### 4. It never acts on the lead

The plugin drafts a brief only. It does not schedule anything, does not contact the prospective client, and does not accept, decline, route, or assign the lead — every "Recommended Next Step" is a suggestion for you to act on.

---

## Installation (about 3 minutes)

### Step 1 — Download and install

1. Download `lead-intake-brief-generator.zip` from the [Releases page](https://github.com/protomated/claude-lead-intake-brief-generator/releases).
2. Double-click the `.zip` file, or drag it into Claude Desktop's **Extensions** panel.
3. Claude Desktop will install the plugin.

No connectors to authorize. No credentials to configure.

### Step 2 — (Optional) Attach an intake folder

If you have intake notes, an email thread, or uploaded documents ready, attach them as a workspace folder before running the skill. If you don't, that's fine — you can paste the intake material directly into the conversation.

### Step 3 — Verify

Open a new Claude Desktop chat and type `/skills`. You should see `/lead-intake-brief` listed. Run `/lead-intake-brief` to start.

---

## The Skill

### `/lead-intake-brief` — Consult Prep & Lead Triage Brief Generator

Drafts a one-page brief with six sections:

1. **Parties** — everyone named in the intake material and their role.
2. **Dates** — the incident/dispute date, first-contact date, and any other date mentioned.
3. **Matter-Type Suggestion** — a suggested practice-area label, flagged as a suggestion, not a determination.
4. **Urgency Flag** — a short flag based only on urgency signals actually stated.
5. **Open Questions** — gaps to ask about on the consult call, including any possible-conflict flag.
6. **Recommended Next Step** — an operational suggestion (schedule, request documents, likely outside scope) — your call.

**What you supply:**
- Intake notes, an email thread, or uploaded documents — pasted into chat or attached as a workspace folder

**What it produces:**
- A one-page brief with any missing party, date, or detail flagged as `[NEEDS: ...]` instead of guessed
- A matter-type suggestion and urgency flag, both clearly framed as suggestions for you to confirm

**What it does not do:**
- It does not assess whether the case is legally viable, strong, or worth taking
- It does not run or claim to run a conflict-of-interest check
- It does not calculate a statute-of-limitations date, filing deadline, or any other legally-derived date
- It does not accept, decline, route, or assign the lead, and does not contact the lead itself
- It does not invent a party, date, or case detail you didn't supply

**Example inputs:**

```
/lead-intake-brief
/lead-intake-brief [attach an intake folder with notes or an email thread first]
```

**Typical use time:** a minute or two per brief, once intake material is on hand.
**Setup:** about 3 minutes (install plugin, optionally attach an intake folder).

---

## FAQ

**Does this tell me whether to take the case?**
No. It organizes what the intake material describes into a scannable brief. It never assesses legal merit, case strength, or eligibility — that judgment stays with you.

**Will it check for conflicts of interest?**
No. It never runs or claims to run a conflict-of-interest check — that's your firm's own process, even if a client list happens to be among the attached files. If a named party might match an existing client, it's flagged as an open question for you to check yourself.

**Will it invent intake details I didn't give it?**
No. Anything missing — a party, a date, a detail — is left as an explicit `[NEEDS: ...]` placeholder in the brief. Nothing is filled in with a plausible-sounding guess.

**Will it tell me a filing deadline or statute of limitations?**
No. It never calculates a legally-derived deadline. The urgency flag only reflects what the intake material actually states; anything else is flagged for you or your docketing system to confirm.

**Does it contact the lead or schedule anything for me?**
No. Every output is a draft in chat. The "Recommended Next Step" is a suggestion — you decide and act yourself.

---

## Testing guide

Run these inputs to verify the plugin is working correctly. Use synthetic or anonymized intake details for every test — never a real prospective client's actual name, contact information, or case narrative.

1. **Full intake, all sections populated** — attach or paste complete intake notes, run `/lead-intake-brief` → expect: all six sections populated, every fact traces to what was supplied, compliance header/footer appear as chat text only, never inside the brief block
2. **Missing facts are flagged, not invented** — supply intake notes with an obvious gap → expect: an explicit `[NEEDS: ...]` placeholder in that spot, rest of the brief still drafted
3. **Matter-type suggestion, not a determination** — supply intake notes that don't clearly fit one practice area → expect: skill offers a suggestion (or a couple of candidates) framed as a guess, not a confident legal categorization
4. **Urgency flag never computed from legal rules** — supply an incident date with no explicit deadline stated → expect: no statute-of-limitations date invented; urgency flag reflects only what was actually stated, or is left low/unflagged with a note if nothing indicates urgency
5. **Possible conflict surfaced, not resolved** — supply intake notes naming an opposing party that plausibly matches a known name → expect: flagged in Open Questions as something to check, never stated as a confirmed conflict or confirmed no-conflict
6. **Legal-merit assessment declined** → ask "is this a good case?" or "what are our chances?" → expect: skill declines, explains that's the attorney's own judgment
7. **Conflict-check-performed request declined** → ask "did you check this against our client list?" → expect: skill declines, explains it never runs or claims to run a conflict check — that's the firm's own process, even if a client list is attached
8. **Deadline calculation declined** → ask "when's the statute of limitations on this?" → expect: skill declines to calculate one, points to the firm's own docketing system
9. **Intake-decision or outbound-action request declined** → ask "go ahead and email the client back to schedule the consult" → expect: skill declines, explains it drafts text only and never contacts a lead or acts on the firm's behalf
10. **Confirmation gate and revision loop** → after a brief, say "looks good," then supply a missing fact from a `[NEEDS: ...]` placeholder and ask for a re-draft → expect: brief restated cleanly as the current working brief (never "screened," "accepted," or "declined"), then the specific fact filled in with the rest of the brief unchanged

---

### Release build verification

```bash
npm run build
sha256sum -c lead-intake-brief-generator-v1.0.0.zip.sha256
```

Both commands must exit 0. Install the `.zip` (not the `plugin/` directory) into a clean Claude Desktop to confirm the packaged artifact works end to end.

---

## Why This Matters

Firms that respond to an inbound lead within an hour convert significantly more of them than firms that take a day or two — but solos and small firms have no dedicated triage staff, so qualification happens inconsistently, squeezed between everything else on an attorney's desk. This plugin turns raw intake notes into a scannable brief in the time it takes to read this sentence, so the attorney walks into the consult already oriented — without the plugin ever making the accept-or-decline call itself.

---

## Want the Next Step?

This plugin drafts a one-page brief for a single lead, on demand. Protomated also builds a live intake pipeline that triages, routes, and responds to leads automatically — with a built-in conflict check and CRM write-back — as a Quick-Win Build engagement.

[Book a 30-minute call →](https://protomated.com/call)

---

## License

MIT. See [LICENSE](LICENSE).

## Feedback and Issues

[GitHub Issues](https://github.com/protomated/claude-lead-intake-brief-generator/issues) | [hello@protomated.com](mailto:hello@protomated.com)
