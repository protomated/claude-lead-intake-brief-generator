# Consult Prep & Lead Triage Brief Generator v1.0.0

Initial release.

## What's included

### `/lead-intake-brief` — Consult Prep & Lead Triage Brief Generator

A one-page consult-prep and lead-triage brief drafting assistant for solo and small-firm attorneys:

- **Six-section brief:** Parties, Dates, Matter-Type Suggestion, Urgency Flag, Open Questions, Recommended Next Step — built strictly from intake notes, an email thread, or uploaded documents you supply.
- **No facts invented:** any missing party, date, or detail is flagged as an explicit `[NEEDS: ...]` placeholder, never guessed.
- **No legal-merits assessment:** the matter-type suggestion is a categorization guess for you to confirm — never an opinion on whether the case is viable or worth taking.
- **No conflict check:** a party that might match an existing client is surfaced as an open question, never resolved as a confirmed conflict or a confirmed absence of one.
- **No deadline calculation:** the urgency flag reflects only what the intake material states directly — no statute-of-limitations or other legally-derived date is ever computed.
- **No intake decision or outbound action:** the skill never accepts, declines, routes, or assigns a lead, and never contacts the lead itself — every recommended next step is a suggestion for you to act on.

Handles: the consult-prep and lead-triage step every inbound inquiry needs, for firms with no dedicated triage staff and no time to lose before a lead goes cold.

## Setup

Install time: about 3 minutes. Download the zip, drag it into Claude Desktop's Extensions panel. No connectors to authorize. Open a new chat, type `/skills`, and verify `/lead-intake-brief` appears. Optionally attach a workspace folder with intake notes, an email thread, or uploaded documents before running it.

## Compliance

Requires Claude for Work, Claude Team, or Claude Enterprise (or the Claude API under a signed DPA) before attaching a real intake file. Every brief carries a "SUGGESTED INTAKE TRIAGE BRIEF — ATTORNEY REVIEW REQUIRED BEFORE ACTING" header and footer as chat text around the draft, never inside the copyable brief block. The skill never assesses legal merit, never runs a conflict check, never computes a deadline, never invents a fact, and never accepts, declines, routes, assigns, or contacts a lead itself.
