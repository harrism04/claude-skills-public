# Embedded 8x8 Converse Inbox Mode

The embedded 8x8 Converse view (formerly Moobidesk) is a full recreation of the 8x8 Converse agent console, rendered inside the same HTML file as the rest of the prototype. Use this when the agent's reply experience is the point of the pitch — ops-team demos, CRM bulk-send follow-up, bot-to-human handoff, anywhere the audience is the CX manager rather than the end customer.

This reference covers layout, styling tokens, data shape, navigation pattern, and the most common wiring bugs.

> **Naming note.** The product was rebranded from Moobidesk to 8x8 Converse in early 2026. JS identifiers in the prototype (`switchStage('moobidesk')`, `.moobidesk-view`, `renderMoobiConvList`, `openMdConversation`, `sendMdReply`) are preserved — they are internal stage names, not product names, and renaming them would break compatibility with `assets/starter-template.html`. CSS tokens have been migrated `--md-*` → `--cv-*` in this doc; existing forked demos using `--md-*` tokens still work, and a mapping table is included at the bottom.

## Visual reference — read first

**Start by reading `assets/moobidesk-inbox-2026-04.png`.** It is the ground-truth screenshot of the 8x8 Converse UI this skill targets. The text below describes the *why* (semantics of each element, when to use which colour, which interactions matter for the pitch); the screenshot describes the *what* (pixel-level layout, spacing, icon choices, font weights). You need both.

When the Converse UI changes meaningfully, re-capture against a clean 8x8 trial tenant (no customer PII — names and phone numbers must be test data or anonymised) and replace the asset. Bump the date in the filename so staleness is obvious — e.g. `converse-inbox-2027-01.png`. Update this document at the same time if the layout semantics shift.

## When to use (vs click-out)

Use the embedded view when:
- The audience uses Converse daily and will scrutinise the recreation for authenticity
- The presenter can't log into a real Converse tenant during the demo (no tenant, SSO pain, demo environment)
- The pitch is about **auto-categorisation**, **smart-reply productivity**, **bulk-reply handling** — things the click-out version can't show because they happen inside the inbox
- The demo flows from broadcast → reply handling (CRM bulk-send pattern)

Use the click-out pattern (Pattern A in `integrations.md`) when:
- The demo is a 1-to-1 customer journey that ends in a simple handoff
- The presenter has a real Converse tenant and wants to show it live
- The audience doesn't need to see the agent UX in detail

## Top-level view swap

Add a second top-level view to the prototype. The body now has two children:

```html
<div class="workflow-view">
  <!-- existing control panel + preview pane (1-to-1 or broadcast) -->
</div>
<div class="moobidesk-view" style="display:none">
  <!-- 8x8 Converse recreation: topbar + 3-column layout -->
</div>
```

The class name `.moobidesk-view` stays — it's an internal stage identifier, not a product label. A `switchStage(stage)` function toggles `display` between them. When entering the Converse view, render the conversations list and auto-select the first unhandled conversation. When leaving, keep handled/unhandled state so returning to the view doesn't feel like a reset.

```javascript
let currentStage = 'workflow';  // 'workflow' | 'moobidesk'

function switchStage(stage) {
  currentStage = stage;
  document.querySelector('.workflow-view').style.display = (stage === 'workflow') ? 'flex' : 'none';
  document.querySelector('.moobidesk-view').style.display = (stage === 'moobidesk') ? 'flex' : 'none';
  if (stage === 'moobidesk') {
    renderMoobiConvList();
    openMdConversation(firstUnhandledCustomerId());
  }
}
```

## 8x8 Converse top bar

Full-width, ~64–72px tall, **dark navy/near-black background** with **white text**. This is the single biggest visual flip from the old Moobidesk chrome — the top bar used to be white. Left-to-right:

- Hamburger menu icon (white)
- **8x8 logo + Converse wordmark** — the orange `#FF7A00` "8x8" lozenge (rounded rectangle, white "8x8" text) immediately followed by the white "Converse" wordmark in a clean sans-serif. NOT the old "M**oo**bidesk" wordmark.
- Account selector — phone number string in white centred text (e.g. `+251559393`). The legacy "8x8 Trial Account" dropdown copy is gone; the new chrome surfaces the connected number directly.
- Right-side cluster:
  - Green pill button with a phone icon (call FAB / Call Notes shortcut — varies by tenant)
  - User profile: "Available" status (small green dot) + circular avatar + user name. For the demo, use the presenter's own name so it feels like *their* console.
  - Optional small action icons (settings, notifications) at the far right
- "← Back to Ops Console" button — calls `switchStage('workflow')`. This button is **not** part of the real Converse UI; place it subtly (top-left as a small pill, white-on-dark) so it doesn't visually compete with the real chrome.

The back button is the one navigational affordance that pulls the presenter out of the Converse view. Make it visible but not loud — the pitch wants the audience to believe they're inside Converse, not looking at a toggle.

## 3-column layout

Below the top bar, flex row with three columns:

1. **Folder sidebar** (~240–260px) — white background. Top section is a "My Reminders" entry with a calendar icon and count (e.g. `0`). Below that, a section header "Conversations" (bold, small caps), then a nested tree:
   - `> Unassigned` (count)
   - `> Assigned` (count)
   - `∨ Assigned to me` (expanded by default, **header row highlighted with a light blue tint background `#E3F2FD` and bold text** — NOT the old dark navy inversion. Count on the right.)
     - Sub-items listed below with counts (channel/topic buckets like `General Enquiry - Wh...`, `Outbound - WhatsApp`, etc.)
     - Sub-item names truncated with `...` when long — let CSS `text-overflow: ellipsis` handle this
   - `> Open`, `> Closed`, `> Spam`

   Use caret icons (`>` collapsed, `∨` expanded) not folder icons — the UI stays minimal. Sub-items indent one level with no bullets.

2. **Conversations list** (~440–500px, middle column) — column header bar at top reads `Conversations` with a filter/sort icon and search icon on the right. A thin controls row with a radio dot (select-all) and sort/filter chevrons sits below.

   Each conversation card has this structure:
   - **Top row**: customer avatar (circular, small) + customer name (bold, 15–16px) + status text (`Responded` / `Read` / `Queued`) + **WhatsApp green channel badge** (small circle with white phone glyph). Elapsed timer right-aligned in **red** when over SLA (format `4h:11m:58s/15m`).
   - **Snippet row**: one-line preview of the most recent message, muted grey, truncated.
   - **Tag row**: tag chips left-aligned. Auto-category chips (`Marketing`, `Renewable`, `TaxSeason`, `Will Pay`, `Defer Request`, `Acknowledged`, `Dispute`, `Speak to Agent`) appear as small coloured pills — match real Converse tag colours where the tenant has custom labels.
   - **Footer row**: conversation # + topic in muted small text, right-aligned `🎧 [Agent Name]` (headset icon + agent name).
   - Cards are separated by a thin grey divider line.

   **Active card styling (NEW in 8x8 Converse — major change from old Moobidesk):** The active card no longer inverts to dark navy. Instead:
   - Card background stays white (or a very subtle `#F8FAFC` tint)
   - **Vertical blue accent bar on the left edge of the card** (`#1976D2`, ~3px wide, full card height)
   - Text colours stay normal — name/phone/snippet remain dark on white
   - The accent bar + slight bg tint is enough visual signal that the card is selected; no further inversion needed

   This is the single most important fidelity update from the previous styling. Get it right and the recreation feels like the current product; get it wrong (revert to charcoal-everything-flips) and the demo looks 12 months old.

3. **Active conversation** (flex: 1, largest column) — Customer header + chat area + composer.

   **Header**: thin row. Left: small grey circular avatar (silhouette if no image) + customer name (bold, 16px) + sub-text (e.g. `Dunder Mifflin · Purchasing` or company/department) stacked vertically. Right: 6–7 action icons in a horizontal row (search, priority/flag, edit/pencil, assign/transfer, info `i`, plus a green call pill and a contact-add icon — match the PNG). Thin border-bottom.

   **Chat area**: large scrollable region with a **near-white background `#FAFAFA`** (lighter than the old `#f7f9fb`). Shows:
   - **Outbound (agent) messages** right-aligned in **dark saturated green bubbles** (~`#1F7A4D`) with **white text**. Agent avatar (initials circle) to the right of the bubble. Below the bubble: timestamp + WhatsApp green phone icon + business number + double-check read receipt. **Critical change**: outbound is no longer the old charcoal `#2c3e50` — it's now in the WA palette family, which makes the channel identity more obvious.
   - **Centred conversation marker** between messages for new threads: bold title (`Conversation #189 - General WA Call`) with a subtitle line (`Opened 12:49PM 17/04/2026`). Muted text, no bubble chrome.
   - **Inbound (customer) messages** left-aligned in **pale grey bubbles** (~`#F0F0F0`) with black text. Small silhouette avatar to the left. Below bubble: timestamp + WhatsApp icon + customer number.
   - **Rich content cards** (when present) render inline with white card backgrounds, rounded corners, optional image, header text, body text, and **action chips** below (e.g. `Track Order`, `Approve Quote`, `Cancel Order` — light-tinted backgrounds, hover states). These represent interactive WhatsApp messages or list/button replies.
   - Multiple back-and-forth messages stack chronologically with ~12px gaps.

   **Smart-reply chips row** (above composer, shown when a customer reply is selected but not yet responded to):
   - 3–5 chips with suggested responses based on the reply's auto-category
   - Tapping a chip pre-fills the composer; presenter can edit before sending
   - Chips should feel specific ("Thank you, we'll note your payment is coming by the 30th") not generic ("Thanks!")
   - Smart-reply chips are not part of the real Converse UI today — if you include them, render them with a subtle sparkle/AI icon so the audience understands this is an *added* AI productivity layer (part of the 8x8 AI Studio pitch), not a retrofitted Converse feature.

   **Composer** (bottom row, ~120–140px tall):
   - Left block: "Send Via:" label with a `WhatsApp` (or `Default WhatsApp`) dropdown underneath (small down-caret)
   - Right of the dropdown: a large text input area placeholder "Type a message"
   - Above the text input, flush-right: row of media tool icons (voice/microphone, image, video, audio-note, document, ellipsis "more"). Spacing is generous; icons are muted grey until hover.
   - Far-right: **black square send button** (`#1A1A1A`) with a white paper-plane send icon. **Critical change**: this used to be the Material blue `#2196F3` square — it's now black, matching the rebranded Converse action style.
   - Top of the composer has a thin border separating it from the chat area

   **Floating green call FAB** (bottom-right, absolute positioned, ~56px diameter) — solid green circle with a white phone-handset icon. Present in the real Converse UI as the voice-call affordance. For the demo, wire it to `window.open()` on the 8x8 trial Converse conversation URL so the presenter can pivot to the live tool mid-demo if they want.

## CSS tokens for 8x8 Converse

Add these to `:root` alongside the brand + WhatsApp tokens. Don't override them per-client — they should match Converse, not the customer's brand.

```css
:root {
  /* ... existing brand and WhatsApp tokens ... */

  /* 8x8 Converse tokens (post-2026 rebrand) */
  --cv-topbar-bg:     #1A2332;   /* dark navy top bar */
  --cv-topbar-text:   #FFFFFF;
  --cv-8x8-orange:    #FF7A00;   /* 8x8 logo lozenge */
  --cv-active-accent: #1976D2;   /* selected conversation card left bar */
  --cv-active-bg:     #F8FAFC;   /* selected card subtle tint (optional) */
  --cv-folder-active: #E3F2FD;   /* "Assigned to me" sidebar highlight */
  --cv-agent-bubble:  #1F7A4D;   /* outbound (agent) message bubble */
  --cv-customer-bubble: #F0F0F0; /* inbound (customer) message bubble */
  --cv-chat-bg:       #FAFAFA;   /* chat area background */
  --cv-send-bg:       #1A1A1A;   /* send button (now black, was blue) */
  --cv-text:          #1A1A1A;
  --cv-muted:         #888;
  --cv-border:        #E8E8E8;
  --cv-hover:         #F5F7FA;
  --cv-sla-red:       #D32F2F;   /* elapsed timer when over SLA */
  --cv-responded:     #6A1B9A;   /* status dot: responded */
  --cv-read:          #0288D1;   /* status dot: read */
  --cv-queued:        #888;
  --cv-wa-green:      #25D366;   /* WhatsApp channel badge */
}
```

### Legacy `--md-*` mapping (for demos forked before the rebrand)

| Legacy token | Legacy value | New token | New value | Notes |
|---|---|---|---|---|
| `--md-blue` | `#2196F3` | `--cv-active-accent` (selection) / `--cv-send-bg` (send btn) | `#1976D2` / `#1A1A1A` | Was used for both. Now split — selection bar stays blue, send button goes black. |
| `--md-charcoal` | `#2c3e50` | `--cv-agent-bubble` (bubbles) / removed (selection) | `#1F7A4D` / — | Was used for both outbound bubbles AND active card inversion. Both flipped. |
| `--md-text` | `#1a1a1a` | `--cv-text` | `#1A1A1A` | Unchanged. |
| `--md-muted` | `#888` | `--cv-muted` | `#888` | Unchanged. |
| `--md-border` | `#e8e8e8` | `--cv-border` | `#E8E8E8` | Unchanged. |
| `--md-hover` | `#f5f7fa` | `--cv-hover` | `#F5F7FA` | Unchanged. |
| `--md-sla-red` | `#d32f2f` | `--cv-sla-red` | `#D32F2F` | Unchanged. |
| `--md-responded` | `#6a1b9a` | `--cv-responded` | `#6A1B9A` | Unchanged. |
| `--md-read` | `#0288d1` | `--cv-read` | `#0288D1` | Unchanged. |
| `--md-queued` | `#888` | `--cv-queued` | `#888` | Unchanged. |
| `--md-wa-green` | `#25D366` | `--cv-wa-green` | `#25D366` | Unchanged. |

**Note on `assets/starter-template.html`:** the starter template ships only with brand + WhatsApp tokens (`--brand-*`, `--wa-*`) — it does NOT pre-define any Converse/Moobidesk tokens. The Converse `--cv-*` tokens above are added to `:root` only when the demo includes an embedded Converse stage. Existing forked demos that hand-coded `--md-*` tokens still work with the legacy mapping below; new demos should add `--cv-*` directly.

## Data shape

Two objects drive the inbox.

### `TODAYS_BATCH` (recipient list from stage 1/2)

Same shape as the broadcast recipient list — see `broadcast.md`. Each recipient is a potential conversation in the inbox.

```javascript
const TODAYS_BATCH = [
  { id: 1, name: "Sarah Tan", phone: "+65 9123 4567", loanRef: "PL-2026-00742",
    amount: 847.50, dueDate: "2026-04-22", initials: "ST" },
  { id: 2, name: "Dewi Anggraini", ...},
  // ... 8–15 more
];
```

### `REPLY_SCENARIOS` (what each customer replied with)

One entry per customer that has an inbound reply. Auto-category is the pitch — surface it prominently on the conversation card and use it to drive the smart-reply chip set.

```javascript
const REPLY_SCENARIOS = {
  1: {
    tag: "Will Pay", tagColor: "willpay",
    reply: "Thanks for the reminder, I'll pay on the 25th via PayNow.",
    time: "09:14", elapsed: "2h:37m:46s/15m", status: "Responded",
    convId: 4521, agent: "Priya",
    smartReplies: [
      "Thank you, Sarah — payment noted for 25 Apr. We'll confirm once received.",
      "Appreciate the update. Can you share a screenshot of the transfer after it's done?",
      "Noted, thank you. Please reply STOP if you'd like to opt out of future reminders."
    ]
  },
  2: {
    tag: "Defer Request", tagColor: "defer",
    reply: "I've lost my job, can I defer for 30 days?",
    // ... smartReplies tuned to deferral scenario
  },
  3: { tag: "Acknowledged", ... },
  4: { tag: "Dispute", ... },
  5: { tag: "Speak to Agent", ... }
};
```

Use 5 distinct categories in the demo set (Will Pay, Defer, Acknowledged, Dispute, Speak to Agent) — it's enough variety to sell the auto-categorisation story without drowning the audience in cards.

### `CONVERSATIONS` (state tracker)

Tracks which conversations the presenter has handled during the demo. Simple dict:

```javascript
const CONVERSATIONS = {};  // customerId -> { handled: boolean, agentReply: string }
```

## Wiring

Minimum set of functions the Converse view needs:

- `renderMoobiConvList()` — renders the middle column from `TODAYS_BATCH` + `REPLY_SCENARIOS` + `CONVERSATIONS`. Active card gets the new blue-accent-bar styling (NOT charcoal inversion).
- `openMdConversation(customerId)` — selects a customer; re-renders the list (to update active state) and the active-conversation column.
- `renderMdActiveConversation(customerId)` — builds the chat area + smart-reply chips + composer for the selected customer.
- `useSmartReply(text)` — pre-fills the composer with a chip's text.
- `sendMdReply()` — sends the composer contents, adds an agent bubble (now `--cv-agent-bubble` green) to the chat area, marks the conversation as handled, re-renders the list, auto-advances to the next unhandled conversation.

Function names start with `Md`/`Moobi` for backwards compatibility with the starter template — these are internal identifiers, treat them like any other variable name. Don't rename them just because the product was rebranded; existing demos depend on them.

Keep the state mutations consistent with the existing orchestrator style — small, direct, no router. The Converse view is a parallel state machine; don't try to share state with the workflow view beyond the recipient list.

## Navigation hooks

From the workflow side, `switchStage('moobidesk')` is the entry point (the string `'moobidesk'` stays — it's a stage identifier). There are three legitimate trigger patterns — pick based on what the journey looks like:

- **Patron-realistic CTA in a template** — e.g. tapping "Speak to Agent" inside a service-hub template fires `switchStage('moobidesk')` from the `handleButtonClick` branch. The button label stays in the patron's voice; only the underlying handler is operator-aware. Best when the journey naturally surfaces a "speak to a human" moment.
- **Orchestrator send-button transform after the journey ends** — when the final journey step is a freeform host reply with no patron CTA, transform the orchestrator's main send button into "Switch to [Agent]'s Converse Console →" with `btn.classList.add('handoff')` and `btn.onclick = () => switchStage('moobidesk')`. Cleanest pattern for 1-to-1 demos that end on the host's reply, because the segue button is unmistakably an orchestrator action — it lives in the left control panel, not inside the phone preview. See `references/integrations.md` Pattern B for the snippet + CSS.
- **Stage button in a multi-stage broadcast demo** — explicit "Open Reply Inbox" stage button on the orchestrator side (control panel, not phone preview). A REPLIED recipient row in broadcast mode also makes sense as an entry point — tapping it enters the inbox on that customer's conversation.

**Anti-pattern to avoid**: a CTA button labeled "Open Host Console", "Open Inbox", or any operator-side language inside a WhatsApp template bubble. Patrons don't see those labels on their real phones, so they shouldn't see them in the demo. If the segue needs a button, put it in the patron's voice ("Speak to Agent") or move the trigger to the orchestrator side.

From the Converse side, the "← Back to Ops Console" button is the only way out. Keep it one click — presenters pivot back and forth during the meeting, and a multi-step return breaks flow.

## Talking points to surface in the UI

The embedded Converse stage is the presenter's chance to surface the messages that matter most for CX leaders:
- **Auto-categorisation cuts triage time** — the tag on every card proves the system read the reply
- **Smart-reply chips cut handle time** — chips that are specific to the category prove the system understood the intent
- **SLA timer creates urgency** — the red `2h:37m:46s/15m` timer shows the cost of slow response
- **Channel icon + conversation # + agent avatar** show the single-pane-of-glass omnichannel story
- **Rich content cards** (interactive product/order cards with action chips) show that Converse handles WhatsApp's full message-type spectrum, not just text

Don't add narration banners to the Converse view itself — the fidelity fights the pitch. Put talking points in the presenter's notes, not on the canvas.

## Things that often break

- **Active card styling regressed to charcoal inversion.** If you're working from an older fork of the prototype or older skill instructions, you may default to the old "everything inverts to `#2c3e50`" pattern. **The new pattern is a blue left-edge accent bar (`--cv-active-accent`) on a white card.** Check the DOM after selecting — text should stay dark on white, only the bar should be coloured.
- **Send button still blue.** Old demos used `#2196F3`. The new send button is **black** (`--cv-send-bg`, `#1A1A1A`). Easy to miss because the button still looks like a button — but a presenter who uses Converse daily will spot it.
- **Top bar still white.** The pre-rebrand Converse top bar was white with a bottom border. The post-rebrand top bar is dark navy with white text. If the demo has a white top bar, it looks 12 months old.
- **Wordmark says "Moobidesk".** Update the logo block to `8x8` orange lozenge + `Converse` wordmark.
- **Stage swap leaves stale scroll position** — the chat area and conversations list both scroll. Reset both on `switchStage('moobidesk')`.
- **Smart-reply chips lag behind selection** — if the chips array doesn't re-render when `openMdConversation()` changes the customer, you end up with chips for the previous reply. Always rebuild chips inside `renderMdActiveConversation`.
- **FAB overlaps composer on narrow viewports** — absolute-positioned FAB + composer can collide below 1280px width. Constrain the Converse view to min-width 1280px or move the FAB up.
- **Sending a reply doesn't advance to the next unhandled** — `sendMdReply` needs to call `openMdConversation(firstUnhandledCustomerId())` at the end. Without it, the presenter has to tap every conversation manually, which kills pacing.
