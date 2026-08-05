# Connectors

This plugin requires no MCP connector. `/lead-intake-brief` drafts from intake notes, an email thread, or uploaded documents — typed into chat, or attached as a workspace folder in Claude Desktop / Cowork — no separate authorization step, no credentials.

## How the plugin reads your files

Cowork's filesystem access is attach-only: the plugin can only see files inside a folder you've explicitly attached to the conversation. It does not browse your computer, does not search beyond that folder, and does not retain access after the conversation ends. It never connects to your practice-management system, your CRM, a conflicts database, or any other outside service — there is nothing for this plugin to authorize.

To use `/lead-intake-brief`, attach a folder containing (or paste directly into chat):

- **Intake notes, an email thread, or uploaded documents** describing the prospective client's situation — whatever your firm captured at first contact. Anything not included is flagged with a `[NEEDS: ...]` placeholder in the brief rather than guessed.
- Nothing else is required — the skill needs no template and no firm-specific configuration to draft a brief.

The plugin drafts a one-page triage brief for your review. It never runs a conflict check, never accepts, declines, routes, or assigns the lead, and never contacts the lead itself — you review, decide, and act yourself.

## Privacy note

The plugin processes intake material and any attached files within your Claude Desktop / Cowork conversation under your Claude plan's data handling terms. No intake fact or brief is transmitted to Protomated or any third party.

Before attaching a real intake file — a prospective client's name, contact details, and case narrative — confirm you are on Claude for Work, Claude Team, or Claude Enterprise, or using the Claude API under a signed Data Processing Agreement (DPA). See the main README for plan requirements.
