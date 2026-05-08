---
name: whatsapp-demo-prototype
description: Build a single-file HTML prototype that simulates a WhatsApp Business API customer journey for an 8x8 client or prospect. Use whenever the user wants to scaffold a WhatsApp demo, Meta matchmaking demo, POC walkthrough, presales prototype, or "demo orchestrator" for a named customer (e.g. "build me a demo for DBS retail loyalty", "WhatsApp prototype for Singtel", "CIMB onboarding walkthrough"). Also trigger on "WABA demo", "WhatsApp Flow demo", "broadcast demo", "campaign demo", "credit card demo", "onboarding demo", "CRM bulk-send demo", "agent inbox demo", or any request to mock up a WhatsApp journey — 1-to-1 conversation, 1-to-many broadcast, or ops-facing agent console. Output is a self-contained HTML file with WhatsApp chrome, left-side step orchestrator, right-side phone preview (or embedded Moobidesk inbox), parameterized for brand, markets/personas, vertical, and integration hooks (Converse handoff, Converse embedded view, MockPass/Singpass, 8x8 Verify OTP, custom URLs; Converse formerly Moobidesk).
---

# WhatsApp Demo Prototype Builder

## What this produces

A single self-contained `index.html` (no build step, no dependencies) that walks a customer through a scripted WhatsApp Business API journey. Fork `assets/starter-template.html` and re-skin per client — the rest of this skill is the playbook.

## Modes

Pick one (or combine):

- **1-to-1 journey** — single persona, scripted conversation. Left pane = step orchestrator, right pane = phone preview. Default. Best for journey design, conversational depth, Flows, interactive CTAs.
- **1-to-many broadcast** — campaign fires to a recipient list, right pane = delivery dashboard. See `references/broadcast.md`. Best for scale, deliverability, CRM bulk sends, OTP volumes.
- **Embedded 8x8 Converse inbox** (formerly Moobidesk) — viewport swaps into a faithful 3-column agent console. See `references/converse-inbox.md`. Best when the audience is the ops/CX manager — agent productivity, reply handling, bot-to-human handoff.

Common combinations: broadcast → embedded Converse (CRM bulk + reply ops); 1-to-1 → Converse handoff (click-out or embedded). Decide combination at intake.

## When to invoke

Trigger eagerly on any of:

- Named bank/telco/retailer/insurer + WhatsApp/demo/prototype/POC/Meta/matchmaking/showcase
- Channel phrases: "WABA demo", "Business API demo", "WhatsApp Flow demo", "template demo", "interactive message demo"
- Use-case phrases: "onboarding demo", "credit card demo", "loyalty demo", "claims demo", "appointment reminder demo", "OTP demo", "broadcast demo", "campaign demo", "CRM bulk-send demo", "reply inbox demo", "agent console demo"
- Generic "build me a demo for [Customer]" with CPaaS context

When in doubt, **ask** rather than invent — the skill works best when brand details are real.

## Workflow

### 1. Intake interview

Use AskUserQuestion (group into 2–4 questions max — don't grill the user one field at a time). Capture:

1. **Client identity** — name, vertical, two or three brand colors (or logo URL to eyedrop), tagline.
2. **Use case + mode** — journey in one sentence (e.g. "credit card acquisition", "policy renewal", "loan reminder bulk send"); choose mode (1-to-1 / broadcast / embedded Converse / combination).
3. **Markets / personas** — single or multi-market tabs (SG/ID/MY/TH). Capture customer name, phone format, currency, locale, market-specific differentiator (e.g. SG = Singpass, ID = SMS OTP).
4. **Integration hooks** — MockPass/Singpass, external Converse handoff, embedded Converse, 8x8 Verify OTP, custom URLs. List each click-out and what it opens.

If the user already gave most of this, skip to gaps.

### 2. Decide step sequence

Pick the closest vertical from `references/verticals.md`. Adapt — don't write from scratch unless genuinely novel.

Target **6–10 steps** (1-to-1) or **4–6 stages** (broadcast / Converse). Each step maps to a real business event. Surface the WhatsApp template category visibly: `MARKETING` (promotional, opt-in, ~24h restrictions), `UTILITY` (transactional updates), `AUTHENTICATION` (OTP/verification), `SERVICE`/`REPLY` (freeform within the 24h customer service window).

### 3. Write demo data

Read `references/architecture.md` first — `DEMO_DATA` shape and step types (`template`, `customer_reply`, `sms_otp`, `system_event`) are specific. Matching the architecture exactly is what makes the orchestrator JS work without modification.

Multi-market: keep `id` and `title` aligned across markets, vary only substantive content (currency, persona, regulatory differentiator).

For broadcast / embedded-Converse modes, see the respective references for their data shapes.

### 4. Re-skin starter template

Fork `assets/starter-template.html`. In order:

1. CSS `:root` brand vars — replace `--brand-primary/secondary/accent`. Keep WA green tokens (`--wa-green`, `--wa-light`, `--wa-bg`, `--wa-dark`) unchanged. If embedded Converse, also define Converse tokens (`--cv-*`) from `references/converse-inbox.md`.
2. Header copy — control panel title, phone preview business name, avatar initials.
3. `DEMO_DATA` from step 3.
4. `FLOW_SCREENS` if Flow used; else strip overlay HTML/JS.
5. Companion pages — `verify.html`, `reschedule.html`, etc. as separate files in same folder. Use `assets/companion-page-template.html`.
6. `.moobidesk-view` (class name preserved for backwards compat) + `switchStage()` swap if embedded Converse — see `references/converse-inbox.md`.

Keep WhatsApp chrome pixel-authentic — full spec in `references/whatsapp-chrome.md`. Do not regress.

### 5. Wire button handlers

Five action types: quick-reply (built-in if `step.message` matches button label), CTA URL, click-out handoff (transforms main send button into "Open [Tool] Console"), embedded handoff (`switchStage('moobidesk')`), in-app Flow (`step.triggersFlow: true`). Snippets in `references/integrations.md`.

**Patron's voice rule** — every label inside a WhatsApp template reads like something the customer would tap: "Speak to Agent", "Pay Now", "RSVP", "View Order". Operator-side language ("Open Console", "Switch View", "Open Inbox") belongs on the orchestrator's main send button or left control panel — never inside the phone preview. Patrons never see those words on their actual phones, so they shouldn't see them in your demo either. (See `references/guardrails.md` for the full posture set.)

### 6. Verify before delivery

Run `references/qa-checks.md` mandatorily. Two layers: (a) mechanical — script extraction + `node --check`, tag balance, placeholder sweep, function presence, optional Playwright runtime assertion; (b) visual walk-through.

**Do not ship a demo that fails the script-extraction check.** Orphaned function tails are silent killers — file loads, chrome renders, but every JS-driven element fails.

### 7. Save and present

Save to `<workspace>/demos/<client-slug>/index.html`. Share via `computer://` link. If multiple files, share the entry point. If the user maintains a `demos/` index, append a card.

## Customization knobs

| Knob | Where | Reference |
|---|---|---|
| Brand colors / logo | CSS `:root` + `.wa-avatar` | starter-template.html |
| Markets / personas | `DEMO_DATA` keys + `.market-tabs` | architecture.md |
| Vertical journey | `DEMO_DATA[market].steps` | verticals.md |
| Mode toggle | Top-level view + preview pane | broadcast.md / converse-inbox.md |
| Click-outs | `handleButtonClick()` branches | integrations.md |
| Embedded 8x8 Converse | `.moobidesk-view` + `switchStage()` | converse-inbox.md |
| WhatsApp Flow | `FLOW_SCREENS` + overlay | architecture.md |
| Companion pages | Separate HTML files | integrations.md |

## References

- `references/architecture.md` — `DEMO_DATA` shape, step types, state machine, button handler patterns
- `references/verticals.md` — proven step sequences for banking, retail, insurance, telco, healthcare, logistics
- `references/broadcast.md` — 1-to-many campaign mode (recipient list + delivery dashboard)
- `references/converse-inbox.md` — embedded 8x8 Converse agent console (3-column inbox, reply scenarios, smart-reply chips, composer). Formerly Moobidesk.
- `references/integrations.md` — handoff snippets, MockPass/Singpass, 8x8 Verify OTP, custom URLs
- `references/whatsapp-chrome.md` — pixel-authentic WA chrome spec (mandatory before re-skin)
- `references/qa-checks.md` — mandatory mechanical verification scripts + common failure modes
- `references/guardrails.md` — full posture set, what-NOT-to-do list, regulatory carve-outs
- `references/asset-mirrors.md` — GitHub mirror URLs + fallback ladder when local assets missing

## Assets

- `assets/starter-template.html` — fork point for new demos (working out-of-the-box)
- `assets/companion-page-template.html` — base for verify.html / reschedule.html / etc.
- `assets/moobidesk-inbox-2026-04.png` — visual ground truth for embedded 8x8 Converse (filename retained from pre-rebrand; view this BEFORE writing Converse markup; details in `references/guardrails.md`)
