---
name: whatsapp-demo-prototype
description: Build a single-file HTML prototype that simulates a WhatsApp Business API customer journey for an 8x8 client or prospect. Use whenever the user wants to scaffold a WhatsApp demo, Meta matchmaking demo, POC walkthrough, presales prototype, or "demo orchestrator" for a named customer (e.g. "build me a demo for DBS retail loyalty", "WhatsApp prototype for Singtel", "CIMB onboarding walkthrough"). Also trigger on "WABA demo", "WhatsApp Flow demo", "broadcast demo", "campaign demo", "credit card demo", "onboarding demo", "CRM bulk-send demo", "agent inbox demo", or any request to mock up a WhatsApp journey — 1-to-1 conversation, 1-to-many broadcast, or ops-facing agent console. Output is a self-contained HTML file with WhatsApp chrome, left-side step orchestrator, right-side phone preview (or embedded Moobidesk inbox), parameterized for brand, markets/personas, vertical, and integration hooks (Moobidesk handoff, Moobidesk embedded view, MockPass/Singpass, 8x8 Verify OTP, custom URLs).
---

# WhatsApp Demo Prototype Builder

## What this skill produces

A single self-contained `index.html` file (no build step, no dependencies) that an SE can open in a browser to walk a customer through a scripted WhatsApp Business API journey. The file is the pitch — every detail is tuned to feel like the customer's own product and brand.

The canonical reference is `assets/starter-template.html` — a working, brand-neutral orchestrator with placeholder tokens and a generic step journey. Fork it and re-skin per client; the rest of this skill is the playbook for how.

## Prototype modes

The skill supports three top-level modes. Pick the one that matches the pitch:

- **1-to-1 journey** — single persona walked through a scripted conversation. Left pane is the step orchestrator; right pane is a WhatsApp phone preview. Default mode. Best when the pitch is about journey design, conversational depth, Flows, or interactive CTAs.
- **1-to-many broadcast** — campaign fires to a recipient list; right pane is a delivery dashboard with per-recipient status animation. See `references/broadcast.md`. Best when the pitch is about scale, deliverability, or campaign orchestration (CRM bulk sends, promo waves, payment reminders, OTP volumes).
- **Embedded Moobidesk inbox** — one or more stages swap the entire viewport into a faithful 3-column Moobidesk agent console recreation (folder sidebar, conversations list, active conversation with composer). See `references/moobidesk-inbox.md`. Best when the pitch centres on the agent's reply experience — ops-team productivity, reply handling after a bulk send, bot-to-human handoff, or anywhere the audience is the ops/CX manager rather than the end customer.

A single demo can combine modes. Common combinations:
- Broadcast stage → Embedded Moobidesk stage (CRM bulk send + reply handling — the HLF-style ops console demo).
- 1-to-1 journey → Moobidesk click-out handoff (simple conversational journey that transfers to a real Moobidesk tenant at the end — see `references/integrations.md`).
- 1-to-1 journey → Embedded Moobidesk stage (conversational journey that ends inside an in-prototype agent console, so the audience sees the whole pipeline without leaving the file).

Decide the combination during intake.

## When to invoke

Trigger eagerly on requests that mention any of:
- A named bank, telco, retailer, insurer, or other 8x8 prospect plus "WhatsApp", "demo", "prototype", "POC", "Meta session", "matchmaking", "showcase"
- Channel-specific phrases: "WABA demo", "Business API demo", "WhatsApp Flow demo", "template demo", "interactive message demo"
- Use-case phrases: "customer onboarding demo", "credit card demo", "loyalty demo", "claims demo", "appointment reminder demo", "OTP demo", "broadcast demo", "campaign demo", "CRM bulk-send demo", "reply inbox demo", "agent console demo"
- Generic "build me a demo for [Customer]" where the prior context is CPaaS / messaging

When in doubt, **ask** rather than invent — the skill works best when the brand details are real.

## Workflow

Follow these steps in order. Don't skip the intake — the demo is only as good as the brief.

### 1. Intake interview

Use AskUserQuestion to capture (group into 2-4 questions max — don't grill the user one field at a time):

1. **Client identity** — name, industry vertical, two or three brand colors (or a logo URL to eyedrop), tagline.
2. **Use case + mode**
   - The customer journey in one sentence (e.g. "credit card acquisition", "policy renewal", "delivery notifications + reschedule", "loan reminder bulk send").
   - **Mode**: 1-to-1 conversation, 1-to-many broadcast, embedded Moobidesk inbox, or a combination (e.g. broadcast + inbox for CRM reply-handling demos). See `references/broadcast.md` and `references/moobidesk-inbox.md` when in doubt.
3. **Markets / personas** — single market or multi-market tabs (e.g. SG/ID/MY/TH). Capture customer name, phone format, currency, locale, and any market-specific differentiator (e.g. SG uses Singpass, ID uses SMS OTP).
4. **Integration hooks** — does the demo need to open MockPass/Singpass, hand off to an external Moobidesk tenant, embed a Moobidesk inbox view in-prototype, trigger 8x8 Verify OTP, or open any other live URL? List each click-out and what it should open.

If the user has already given you most of this in the prompt, skip to the gaps.

### 2. Decide the step sequence

Use `references/verticals.md` for proven step sequences by industry. Pick the closest vertical and adapt — don't write from scratch unless the use case is genuinely novel.

A good demo has **6–10 steps** (1-to-1 mode) or **4–6 stages** (broadcast / Moobidesk modes). Fewer feels thin; more loses the audience. Each step should map to a real business event, not just "another message". Mark each step with its WhatsApp template category:

- `MARKETING` — promotional / acquisition (must be opt-in, ~24h restrictions)
- `UTILITY` — transactional updates (order confirmations, OTPs, reminders)
- `AUTHENTICATION` — OTP / verification (different pricing tier in some markets)
- `SERVICE` / `REPLY` — freeform inside the 24-hour customer service window

Surfacing the category visibly in the left panel is part of the pitch — it shows the SE knows the WABA pricing model.

### 3. Write the demo data

Read `references/architecture.md` first if you're unfamiliar with the `DEMO_DATA` shape and step types (`template`, `customer_reply`, `sms_otp`, `system_event`). The architecture is small but specific; matching it exactly is what makes the orchestrator JS work without modification.

For multi-market demos, keep the step `id` and `title` aligned across markets so the user can compare side-by-side. Vary only the substantive content (currency, persona name, regulatory differentiator).

For broadcast mode, write a recipient list and campaign definition instead — see `references/broadcast.md`.

For embedded-Moobidesk stages, write reply scenarios and conversation cards — see `references/moobidesk-inbox.md`.

### 4. Re-skin the starter template

Start from `assets/starter-template.html`. It is a complete, working orchestrator with placeholder client tokens and a generic 3-step journey. Re-skin in this order:

1. **CSS variables** in `:root` — replace `--brand-primary`, `--brand-secondary`, `--brand-accent` with the client's colors. Keep WhatsApp green tokens (`--wa-green`, `--wa-light`, `--wa-bg`, `--wa-dark`) unchanged. If the demo includes an embedded Moobidesk stage, also define the Moobidesk tokens listed in `references/moobidesk-inbox.md`.
2. **Header copy** — control panel title, phone preview business name, verified-account avatar initials.
3. **`DEMO_DATA`** — replace with the step sequence you wrote in step 3.
4. **`FLOW_SCREENS`** — only if the demo includes a WhatsApp Flow (the in-app form). Skip the overlay HTML and JS if not needed.
5. **Companion pages** — if the demo opens `verify.html`, `reschedule.html`, etc., create those as separate small HTML files in the same folder. Use `assets/companion-page-template.html` as the base.
6. **Moobidesk view (optional)** — if the demo includes an embedded Moobidesk stage, add the `.moobidesk-view` section and `switchStage()` swap logic as described in `references/moobidesk-inbox.md`.

### 5. Add interactive button handlers

Buttons inside WhatsApp messages can do four things:
- **Quick reply** — advances the demo if the next step is a matching `customer_reply`. Built into the template; no extra code needed if you set `step.message` to match the button label.
- **CTA** — opens an external URL in a new tab (Singpass, MockPass, custom portal). Add a branch in `handleButtonClick()` keyed on the button text.
- **Handoff (click-out)** — transforms the main send button into "Open [Tool] Console" (e.g. external Moobidesk URL). Pattern is in the starter template's `Speak to Agent` handler.
- **Handoff (embedded)** — switches the view from 1-to-1/broadcast to the embedded Moobidesk stage. Pattern is in `references/moobidesk-inbox.md`.
- **In-app flow** — opens the WhatsApp Flow overlay. Set `step.triggersFlow: true` on the preceding step.

See `references/integrations.md` for ready-to-paste handler snippets for the common click-outs.

### 6. Sanity check before delivery

Open the file mentally and walk through every step:
- Does each step's `trigger` label read like a real backend system event (not "user clicks button")? Triggers are part of the pitch — they communicate "this is automated, deterministic, eventable".
- Does the customer profile card at the top tell the persona's story in one glance?
- Is every CTA button wired? An unhandled button click is the most common bug.
- For multi-market: does switching tabs reset the chat and re-render the profile?
- For broadcast mode: do the per-recipient delivery states animate convincingly?
- For embedded Moobidesk: is the active-conversation card highlight faithful, do smart-reply chips populate the composer, does the back-button return to the workflow view cleanly?

### 7. Save and present

Save the final HTML to the user's selected folder (typically `<workspace>/demos/<client-slug>/index.html`) and share via a `computer://` link. If multiple files (companion pages), share the entry point.

If the user already maintains a `demos/` index page, append a card linking to the new demo.

## Customization knobs (summary)

| Knob | Where it lives | Reference |
|---|---|---|
| Brand colors / logo | CSS `:root` vars + `.wa-avatar` text | starter-template.html |
| Markets / personas | `DEMO_DATA` keys + `.market-tabs` | architecture.md |
| Vertical journey | `DEMO_DATA[market].steps` array | verticals.md |
| Mode (1-to-1 / broadcast / Moobidesk) | Top-level view toggle, swaps preview pane | broadcast.md / moobidesk-inbox.md |
| Integration click-outs | `handleButtonClick()` branches | integrations.md |
| Embedded Moobidesk inbox | `.moobidesk-view` + `switchStage()` | moobidesk-inbox.md |
| WhatsApp Flow form | `FLOW_SCREENS` + overlay HTML | architecture.md |
| Companion pages | Separate HTML files in same folder | integrations.md |

## Style and posture

The demo is a **sales asset**, not engineering documentation. That means:

- **Show the channel mechanics, not the implementation.** Audiences care that "this is a Utility template that fires when the credit decisioning engine returns an approved status" — they don't care about the API endpoint. Lead with business events.
- **Use real-looking data.** Believable reference IDs (`CC-2026-04158`, `PL-2026-00742`), last-4-only card numbers, local currency formatting, locale-appropriate dates. Lazy placeholders (`[Customer Name]`, `XXX-XXX`) kill the spell.
- **Keep WhatsApp chrome pixel-authentic** — see the mandatory list below.
- **Keep Moobidesk chrome authentic too.** If you include an embedded Moobidesk stage, match the real UI — folder tree hierarchy, active-card charcoal background, status dots, red elapsed timers, WhatsApp channel badge, headset agent-assignment icon. Reference a real Moobidesk screenshot before building. See `references/moobidesk-inbox.md`.
- **Don't overdesign the left panel.** It's a control surface, not a brochure. Step cards with status pills, category tags, trigger labels. Resist adding charts or decorative elements that compete with the chat preview.

## WhatsApp chrome — mandatory authenticity checklist

The starter template implements all of these. Do not regress them when re-skinning.

### Phone shell (required for 1-to-1 mode)
- Dark phone shell with correct border-radius (~44px corners), subtle border, box-shadow giving depth.
- Dynamic Island / notch rendered as a dark pill at top-center, z-index above status bar.
- Home indicator bar at bottom (thin pill, ~130px wide, rgba white, centered in a dark strip).

### Status bar
- Background matches the WA header colour (`#025144` — the teal dark edge, not the main teal).
- Left: live clock (update every 10s via `setInterval`). Right: signal bars + WiFi + battery icons in SVG.
- Height 52px with content aligned to bottom-edge (simulates notch safe-area).

### WA navigation bar (topbar)
- Back chevron (`‹`) + circular business avatar (initials, white bg, brand-primary text) + business name + verified badge + subtitle "Business Account · Verified" + three right-side icons (video, phone, vertical-dots) all in SVG.
- Verified badge: small `#53bdeb` teal circle with white checkmark SVG inside — NOT a Unicode ✓.
- Background: `#008069` (WA teal, slightly lighter than header edge).

### Chat area / wallpaper
- `background-color: #efeae2` (the real WA beige).
- `background-image`: SVG tile of the authentic leaf/petal pattern at ~35% opacity in `#c8bfb0`. Do not use a generic diamond grid.
- Custom scrollbar: 4px wide, transparent track, semi-transparent dark thumb.

### Message bubbles
- **Incoming (business)**: white (`#ffffff`) bg, `border-top-left-radius: 2px`, CSS triangle tail top-left using `::before` (`border-top: 8px solid #fff; border-left: 8px solid transparent`).
- **Outgoing (customer)**: WA green (`#d9fdd3`) bg, `border-top-right-radius: 2px`, CSS triangle tail top-right.
- Bottom padding `22px` on both to make room for the absolute-positioned meta row.
- **Meta row**: `position:absolute; bottom:5px; right:9px` — time string + tick glyphs.
- **Ticks**: use `✓✓` characters styled via CSS. Grey (`#667781`) for delivered, blue (`#53bdeb`) for read. Do NOT use emoji or images.
- **Sender name row** inside business bubble: brand-primary colour, bold, with verified badge icon.
- `box-shadow: 0 1px 1px rgba(0,0,0,0.1)` on all bubbles.
- Message animation: `opacity 0 → 1`, `translateY(6px → 0)`, 0.18s ease-out.
- Max bubble width: leave `padding-right: 48px` (in-wrap) / `padding-left: 48px` (out-wrap) to match WA proportions.

### CTA buttons
- **Rendered as a SEPARATE block** (`wa-cta-block`) appended below the bubble, NOT inside it. This matches WA Business template layout exactly.
- White background (same as incoming bubble), `border-radius: 0 0 8px 8px`.
- Each button: `border-top: 1px solid #e0e0e0`, link-blue text (`#027eb5`), centered, 13px bold.
- URL-type buttons get an external-link SVG icon; reply buttons get no icon.

### Typing indicator
- Sits in DOM as a persistent node, shown/hidden via `.show` class — not created/destroyed on each step.
- Three bouncing dots (7px circles, `#b0bec5`) with staggered vertical animation.
- Housed inside a white bubble (incoming shape) with the tail.
- Positioned via `wa-typing-wrap.in` wrapper with matching padding.

### WhatsApp Flow overlay
- Full-pane scrim (`rgba(0,0,0,0.55)`), bottom-sheet slides up with `cubic-bezier(0.32,0.72,0,1)`.
- Sheet has: drag handle pill at top, header with brand icon + title + close `×`, segmented progress bars (not dots — WA Flows uses bars), scrollable body, sticky footer with Back/Continue/Submit.
- Progress bars: active = `#008069`, done = `#4caf50`, inactive = `#e0e0e0`.

### Input bar
- Decorative only (not functional). `background: #f0f2f5`. Emoji icon on left, rounded pill with "Message" placeholder + attach icon inside, circular teal send FAB on right.
- This makes the phone feel inhabited, not hollow.

### SMS OTP bubble
- Yellow-tinted (`#fffde7`), left border `3px solid #f9a825`, `border-radius: 0 8px 8px 8px` (no tail).
- Header label "📱 SMS — separate channel · 8x8 Verify API" in amber bold.
- OTP body in monospace.
- This differentiation is part of the pitch — it signals multi-channel orchestration, not a WA message.

## What NOT to do

- Don't use external CSS frameworks, build tools, or npm packages. Single file, browser-openable, period.
- Don't render CTA buttons inside the message bubble div. They must be a separate sibling block — this is the correct WA Business template structure.
- Don't use Unicode ticks (✓) as plain text styled with `color`. Use the glyph inside a `.tick` / `.tick.read` span so grey/blue state can be toggled.
- Don't skip the phone shell. A raw chat pane on a white background looks like a web app, not WhatsApp. The shell is non-negotiable for 1-to-1 mode.
- Don't use a generic grid or diamond tile for the wallpaper. Use the SVG leaf/petal pattern from the starter template.
- Don't fake the WhatsApp green styling for SMS OTPs. They get the yellow `#fffde7` treatment.
- Don't invent regulatory claims. If the SG market shows Singpass, that's because MAS banned SMS OTP for banking.
- Don't write CSS or JS in separate files. The demo must travel as one file.
- Don't half-recreate Moobidesk. If you embed it, go faithful.

## References

Read these before scaffolding a new demo:
- `references/architecture.md` — `DEMO_DATA` shape, step types, state machine, button handler patterns
- `references/verticals.md` — proven step sequences for banking, retail, insurance, telco, healthcare, logistics
- `references/broadcast.md` — 1-to-many campaign mode (recipient list + delivery dashboard)
- `references/moobidesk-inbox.md` — embedded Moobidesk agent console (3-column inbox recreation, reply scenarios, smart-reply chips, composer)
- `references/integrations.md` — Moobidesk click-out handoff, MockPass/Singpass, 8x8 Verify OTP, custom URL patterns

Assets:
- `assets/starter-template.html` — fork point for new demos (working out-of-the-box)
- `assets/companion-page-template.html` — base for verify.html / reschedule.html / etc.
