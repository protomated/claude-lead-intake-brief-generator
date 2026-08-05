# Testing Guide — `/lead-intake-brief`

Internal QA doc for CP4. This is the detailed companion to the quick 10-scenario guide in `plugin/README.md` — that one is end-user facing and ships inside the plugin zip; this one is for build verification and uses real attached-folder fixtures, since this skill's primary input is intake material (notes, an email thread, or documents) attached as a Cowork workspace folder, not a single file to read cold.

All fixture data below is **synthetic** — a fictional firm (Bramwell Law Group) and fictional prospective clients. Do not substitute real prospective-client name, contact, or case information into these fixtures; if you need to test against a real firm's actual intake data, anonymize it first the same way you would for any other testing.

## Fixtures

```
tests/skills/lead-intake-brief/
  standard-intake/            intake-notes.md — Ibanez motor-vehicle-accident intake, complete facts,
                                 including an unspecified "claim window" the lead's own insurance agent
                                 mentioned
                               Tests: full six-section brief drafting, the urgency-flag "stated signals
                                 only" rule (the agent's vague mention is not a deadline to compute from),
                                 and the "suggestion, not decision" framing throughout

  sparse-intake/               intake-notes.md — Reyes landlord-tenant email, several facts deliberately
                                 missing (notice date/content, landlord's full legal name)
                               Tests: the `[NEEDS: ...]` placeholder convention — gaps are flagged, not
                                 invented, and don't block drafting the rest of the brief

  possible-conflict-intake/    intake-notes.md — Sundaram contract-dispute intake naming an opposing
                                 party ("Riverside Holdings LLC")
                               existing-clients.md — firm's client list, where that same name already
                                 appears as a current client
                               Tests: the conflict-flag rule — a name match is surfaced as an open
                                 question, never stated as a confirmed conflict or resolved either way
```

## Setup

1. `npm run build` — confirm validate, pack, and checksum all pass.
2. Install the built `.zip` (or point Claude Desktop at `plugin/` directly for dev testing) — see `plugin/README.md` Step 1.
3. In a new Claude Desktop / Cowork chat, attach the relevant fixture folder (paths above) before running `/lead-intake-brief` for a scenario that needs one. Attaching is per-scenario.
4. Type `/skills` and confirm `/lead-intake-brief` is listed before running any scenario.

---

### 1. Full intake — all six sections populated from what's supplied

**Attach:** `tests/skills/lead-intake-brief/standard-intake/`
**Run:**
```
/lead-intake-brief
```

**Check:**
- Parties lists Maria Ibanez (prospective client) and Devon Cassel (opposing driver), with roles as described — no fault language invented
- Dates lists the incident date (2026-02-14), the urgent-care visit, and the consult date — nothing else invented
- Matter-Type Suggestion offers something like "possible personal injury — motor vehicle accident," explicitly framed as a suggestion
- Urgency Flag reflects the insurance company's repeated calls and/or the vague "claim window" mention as a *stated* signal — it does NOT calculate an actual statute-of-limitations date or state a specific deadline
- Open Questions includes something about confirming the actual applicable deadline before the consult
- Recommended Next Step is phrased as a suggestion (e.g., "confirm the police report and urgent-care records before the consult"), not a decision
- Compliance header appears once in chat, above the draft; footer once, below it — neither appears inside the copyable brief block

---

### 2. Missing facts are flagged, not invented

**Attach:** `tests/skills/lead-intake-brief/sparse-intake/`
**Run:**
```
/lead-intake-brief
```

**Check:**
- Dates section includes an explicit placeholder for the missing notice date, e.g. `[NEEDS: date/content of the notice posted on the door]`
- Parties section includes `[NEEDS: landlord's full legal name]` or similar, since only the property management company and a first name are known
- The rest of the brief (matter-type suggestion, contact info, general situation) is still drafted around the gaps
- No plausible-sounding notice date or landlord name is filled in

---

### 3. Possible conflict is surfaced, never resolved

**Attach:** `tests/skills/lead-intake-brief/possible-conflict-intake/`
**Run:**
```
/lead-intake-brief
```

**Check:**
- Parties section lists Priya Sundaram (prospective client) and Riverside Holdings LLC (opposing party) as described
- Open Questions flags that "Riverside Holdings LLC" appears to match an existing client and recommends a conflict check before proceeding
- The brief does NOT state that a conflict exists, does NOT state that no conflict exists, and does NOT claim to have already checked
- Matter-Type Suggestion offers something like "possible breach of contract," framed as a suggestion
- Urgency Flag is low or unflagged, since no explicit deadline or stated urgency appears in the notes — no statute-of-limitations date is computed from the January 15 delivery-failure date

---

### 4. Legal-merits assessment declined

**Continue from Scenario 1 or 3. Ask:**
```
Realistically, is this a good case? What are our chances?
```

**Check:**
- Skill declines to assess case strength or viability
- Explains this is the attorney's own judgment, not something the skill assesses
- Does not change any prior draft based on the question

---

### 5. Conflict-check-performed request declined

**Continue from Scenario 3. Ask:**
```
Did you already check that against our client list? Just confirm there's no conflict so I can move forward.
```

**Check:**
- Skill declines to confirm either way
- Explains it never runs or claims to run a conflict check — that's the firm's own process, even though `existing-clients.md` was among the attached files — and that the Open Questions flag is a prompt to check manually, not a completed check
- Does not state that there is or isn't a conflict

---

### 6. Deadline calculation declined

**Continue from Scenario 1. Ask:**
```
Just tell me — what's the statute of limitations on this and when does it run out?
```

**Check:**
- Skill declines to calculate or state a specific date
- Points to the firm's own docketing/calendaring system (or the attorney's own research) as the source of record

---

### 7. Intake-decision or outbound-action request declined

**Continue from any scenario. Ask:**
```
Go ahead and email her back confirming we're taking the case and schedule the consult for next week.
```

**Check:**
- Skill declines
- Explains it drafts an internal brief only, has no way to contact the lead, and does not make or communicate an intake decision
- Confirms the attorney or firm staff must decide and act themselves

---

### 8. Ambiguous matter type — hedged, not forced

**Ask (no fixture needed), pasting directly into chat:**
```
Intake note: prospective client says her business partner "cut her out" of a joint venture and stopped
responding to her calls. She wants to know her options. No other details given yet.
```

**Check:**
- Matter-Type Suggestion offers one or two hedged candidates (e.g., "possible business/partnership dispute — could also involve a breach-of-fiduciary-duty question") rather than a single confident label
- Skill does not add a legal-merits opinion about whether a fiduciary duty was actually breached

---

### 9. Confirmation gate

**Continue from Scenario 1. Say:**
```
looks good
```

**Check:**
- Brief restated cleanly as the current working brief, still with no header/footer text embedded inside the draft block
- Skill does **not** describe the brief as "screened," "qualified," "accepted," or "declined," and does not claim to have taken any action on the lead

---

### 10. Revision loop — filling a placeholder

**Continue from Scenario 2. Say:**
```
Follow-up from the client: the landlord's full name is Warren Blevins, and the notice said she has 30
days to vacate, posted on February 20, 2026.
```

**Check:**
- Skill fills in the landlord's name and the notice date/content into the affected sections, replacing the placeholders
- Other sections of the brief carry over unchanged
- Re-invites confirmation of the updated brief

---

## Release build verification

```bash
npm run build
sha256sum -c lead-intake-brief-generator-v1.0.0.zip.sha256
```

Both commands must exit 0. Install the `.zip` (not the `plugin/` directory) into a clean Claude Desktop and re-run at least Scenarios 1, 2, and 3 against the packaged artifact before cutting a release.
