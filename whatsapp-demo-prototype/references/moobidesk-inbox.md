# Embedded Moobidesk Inbox Mode

The embedded Moobidesk view is a full recreation of the 8x8 Moobidesk agent console, rendered inside the same HTML file as the rest of the prototype. Use this when the agent's reply experience is the point of the pitch — ops-team demos, CRM bulk-send follow-up, bot-to-human handoff, anywhere the audience is the CX manager rather than the end customer.

This reference covers layout, styling tokens, data shape, navigation pattern, and the most common wiring bugs.

## Visual reference — read first

**Start by reading `assets/moobidesk-inbox-2026-04.png`.** It is the ground-truth screenshot of the Moobidesk UI this skill targets. The text below describes the *why* (semantics of each element, when to use which colour, which interactions matter for the pitch); the screenshot describes the *what* (pixel-level layout, spacing, icon choices, font weights). You need both.

When Moobidesk's UI changes meaningfully, re-capture against a clean 8x8 trial tenant (no customer PII — names and phone numbers must be test data or anonymised) and replace the asset. Bump the date in the filename so staleness is obvious — e.g. `moobidesk-inbox-2027-01.png`. Update this document at the same time if the layout semantics shift.

## When to use (vs click-out)

Use the embedded view when:
- The audience uses Moobidesk daily and will scrutinise the recreation for authenticity
- The presenter can't log into a real Moobidesk tenant during the demo (no tenant, SSO pain, demo environment)
- The pitch is about **auto-categorisation**, **smart-reply productivity**, **bulk-reply handling** — things the click-out version can't show because they happen inside the inbox
- The demo flows from broadcast → reply handling (CRM bulk-send pattern)

Use the click-out pattern (Pattern A in `integrations.md`) when:
- The demo is a 1-to-1 customer journey that ends in a simple handoff
- The presenter has a real Moobidesk tenant and wants to show it live
- The audience doesn't need to see the agent UX in detail

## Top-level view swap

Add a second top-level view to the prototype. The body now has two children:

```html
<div class="workflow-view">
  <!-- existing control panel + preview pane (1-to-1 or broadcast) -->
</div>
<div class="moobidesk-view" style="display:none">
  <!-- Moobidesk recreation: topbar + 3-column layout -->
</div>
```

A `switchStage(stage)` function toggles `display` between them. When entering the Moobidesk view, render the conversations list and auto-select the first unhandled conversation. When leaving, keep handled/unhandled state so returning to the view doesn't feel like a reset.

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

## Moobidesk top bar

Full-width, ~72px tall, white background with subtle bottom border. Left-to-right:
- Hamburger menu icon (dark grey)
- Moobidesk logo + word mark (black "M**oo**bidesk" with the two O's as the signature eyes/logo marks)
- Version string in small muted text below the wordmark (e.g. `v20260225001` — Moobidesk uses a date-based build number, not semver)
- Account selector (large dropdown showing "8x8 Trial Account" — centred in the top bar region)
- User profile: circular avatar + user name + "Available" status (small green dot). Real-world shows `Harris Malik / Available`; for the demo use the presenter's own name so it feels like *their* console.
- Unread chat counter on the far right — small speech-bubble icon with a count in `X/Y` format (e.g. `8/50`) representing current load against max concurrent chats.
- "← Back to Ops Console" button — calls `switchStage('workflow')`. This button is **not** part of the real Moobidesk UI; place it subtly (e.g. top-left as a small pill) so it doesn't visually compete with the real chrome.

The back button is the one navigational affordance that pulls the presenter out of the Moobidesk view. Make it visible but not loud — the pitch wants the audience to believe they're inside Moobidesk, not looking at a toggle.

## 3-column layout

Below the top bar, flex row with three columns:

1. **Folder sidebar** (~260px) — white background. Top section is a "My Reminders" entry with a calendar icon and count (e.g. `0`). Below that, a section header "Conversations" (bold, small caps), then a nested tree:
   - `> Unassigned` (count)
   - `> Assigned` (count — real tenant shows 53)
   - `∨ Assigned to me` (expanded by default, **header row highlighted in dark navy `#2c3e50`-ish with blue accent text**, count on right — real tenant shows 8)
     - Sub-items listed below with counts (channel/topic buckets like `General Enquiry - Wh...`, `General Enquiry Bot -...`, `General Enquiry Sand...`, `General WA Call`, `Outbound - WhatsApp`, `Outbound Sandbox 8...`)
     - Sub-item names are truncated with `...` when long — let CSS `text-overflow: ellipsis` handle this
   - `> Closed` (count — real tenant shows 162)
   - `> Spam`

   Use caret icons (`>` collapsed, `∨` expanded) not folder icons — the real UI is minimal. Sub-items indent one level with no bullets.

2. **Conversations list** (~500px, middle column) — column header bar at top reads `Conversations` with a filter/sort icon and search icon on the right. A thin controls row with a radio dot (select-all) and sort/filter chevrons sits below.

   Each conversation card has this structure:
   - **Top row**: empty radio dot (far left), customer name (bold, 15-16px), status dot (purple `#6a1b9a` = `Responded`, blue `#0288d1` = `Read`, grey = `Queued`), status label text (`Responded` / `Read`), elapsed timer on the far right in **red** when over SLA (format `4h:11m:58s/15m` — elapsed vs SLA target).
   - **Phone row**: phone number in large semi-bold text (17-18px, dark `#333`), left-aligned.
   - **Snippet row**: one-line preview of the most recent message, truncated. Small WhatsApp green circle icon (white phone glyph inside) right-aligned on this row.
   - **Tag row**: tag chip left-aligned — auto-category (`NO LABEL` in grey is the unstyled default; use `Will Pay`, `Defer Request`, `Acknowledged`, `Dispute`, `Speak to Agent` for the demo; match real Moobidesk tag colours if your tenant has custom labels).
   - **Footer row**: conversation # + topic (`#189 General WA Call`) in muted small text, and right-aligned `🎧 Harris Malik` (headset icon + agent name).
   - Cards are separated by a thin grey divider line.

   **Active card styling**: the entire card inverts to dark navy `#2c3e50` background with white text. Everything flips — name, phone, snippet, `#189 General WA Call`, agent name. The only things that keep their colour are the status dot (stays purple/blue), the elapsed timer (stays red), the WhatsApp icon (stays green), and the `NO LABEL` chip (stays grey/white). This inversion is the single most important fidelity detail — get it wrong and the whole recreation feels off.

3. **Active conversation** (flex: 1, largest column) — Customer header + chat area + composer.

   **Header**: thin row. Left: small grey circular avatar (silhouette if the customer has no image) + customer name (bold, 16px) + phone number (smaller, muted) stacked vertically. Right: 5 action icons (magnifier-search, exclamation-mark priority, pencil-edit, arrow-box assign/transfer, info `i`) in a horizontal row. Thin border-bottom.

   **Chat area**: large scrollable region with an off-white/light background (~`#f7f9fb`). Shows:
   - Outbound messages from the agent/business right-aligned in **dark navy `#2c3e50`** bubbles, white text. Agent avatar (e.g. cyan-ish initials circle like `HM`) to the right of the bubble. Below the bubble: timestamp + WhatsApp green phone icon + business number + double-check read receipt.
   - A **centred conversation marker** between messages for new threads: bold title (`Conversation #189 - General WA Call`) with a subtitle line (`Opened 12:49PM 17/04/2026`). Muted text, no bubble chrome.
   - Inbound customer messages left-aligned in **light grey-blue `#eaeef4`** bubbles with black text. Small silhouette avatar to the left. Below bubble: timestamp + WhatsApp icon + customer number.
   - Multiple back-and-forth messages stack chronologically with ~12px gaps.

   **Smart-reply chips row** (above composer, shown when a customer reply is selected but not yet responded to):
   - 3-5 chips with suggested responses based on the reply's auto-category
   - Tapping a chip pre-fills the composer; presenter can edit before sending
   - Chips should feel specific ("Thank you, we'll note your payment is coming by the 30th") not generic ("Thanks!")
   - Chips are not part of the real Moobidesk UI today — if you include them, render them with a subtle sparkle/AI icon so the audience understands this is an *added* AI productivity layer (part of the 8x8 AI Studio pitch), not a retrofitted Moobidesk feature.

   **Composer** (bottom row, ~120-140px tall):
   - Left block: "Send Via:" label with a `WhatsApp` dropdown underneath (small down-caret)
   - Right of the dropdown: a large text input area placeholder "Type a message"
   - Above the text input, flush-right: row of media tool icons (voice/microphone, image, video, audio-note, document, ellipsis "more"). Spacing is generous; icons are muted grey until hover.
   - Far-right: large blue `#2196F3` square button with a white paper-plane send icon
   - Top of the composer has a thin border separating it from the chat area

   **Floating green call FAB** (bottom-right, absolute positioned, ~56px diameter) — solid green circle with a white phone-handset icon. Present in the real Moobidesk UI as the voice-call affordance. For the demo, wire it to `window.open()` on the 8x8 trial Moobidesk conversation URL so the presenter can pivot to the live tool mid-demo if they want.

## CSS tokens for Moobidesk

Add these to `:root` alongside the brand + WhatsApp tokens. Don't override them per-client — they should match Moobidesk, not the customer's brand.

```css
:root {
  /* ... existing brand and WhatsApp tokens ... */

  /* Moobidesk tokens */
  --md-blue:      #2196F3;   /* Moobidesk primary action */
  --md-charcoal:  #2c3e50;   /* selected conversation + agent bubble */
  --md-text:      #1a1a1a;
  --md-muted:     #888;
  --md-border:    #e8e8e8;
  --md-hover:     #f5f7fa;
  --md-sla-red:   #d32f2f;   /* elapsed timer when over SLA */
  --md-responded: #6a1b9a;   /* status dot: responded */
  --md-read:      #0288d1;   /* status dot: read */
  --md-queued:    #888;
  --md-wa-green:  #25D366;   /* WhatsApp channel badge */
}
```

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

Minimum set of functions the Moobidesk view needs:

- `renderMoobiConvList()` — renders the middle column from `TODAYS_BATCH` + `REPLY_SCENARIOS` + `CONVERSATIONS`. Active card gets the charcoal styling.
- `openMdConversation(customerId)` — selects a customer; re-renders the list (to update active state) and the active-conversation column.
- `renderMdActiveConversation(customerId)` — builds the chat area + smart-reply chips + composer for the selected customer.
- `useSmartReply(text)` — pre-fills the composer with a chip's text.
- `sendMdReply()` — sends the composer contents, adds an agent bubble to the chat area, marks the conversation as handled, re-renders the list, auto-advances to the next unhandled conversation.

Keep the state mutations consistent with the existing orchestrator style — small, direct, no router. The Moobidesk view is a parallel state machine; don't try to share state with the workflow view beyond the recipient list.

## Navigation hooks

From the workflow side, `switchStage('moobidesk')` is the entry point. There are three legitimate trigger patterns — pick based on what the journey looks like:

- **Patron-realistic CTA in a template** — e.g. tapping "Speak to Agent" inside a service-hub template fires `switchStage('moobidesk')` from the `handleButtonClick` branch. The button label stays in the patron's voice; only the underlying handler is operator-aware. Best when the journey naturally surfaces a "speak to a human" moment.
- **Orchestrator send-button transform after the journey ends** — when the final journey step is a freeform host reply with no patron CTA, transform the orchestrator's main send button into "Switch to [Agent]'s Moobidesk Console →" with `btn.classList.add('handoff')` and `btn.onclick = () => switchStage('moobidesk')`. Cleanest pattern for 1-to-1 demos that end on the host's reply, because the segue button is unmistakably an orchestrator action — it lives in the left control panel, not inside the phone preview. See `references/integrations.md` Pattern B for the snippet + CSS.
- **Stage button in a multi-stage broadcast demo** — explicit "Open Reply Inbox" stage button on the orchestrator side (control panel, not phone preview). A REPLIED recipient row in broadcast mode also makes sense as an entry point — tapping it enters the inbox on that customer's conversation.

**Anti-pattern to avoid**: a CTA button labeled "Open Host Console", "Open Inbox", or any operator-side language inside a WhatsApp template bubble. Patrons don't see those labels on their real phones, so they shouldn't see them in the demo. If the segue needs a button, put it in the patron's voice ("Speak to Agent") or move the trigger to the orchestrator side.

From the Moobidesk side, the "← Back to Ops Console" button is the only way out. Keep it one click — presenters pivot back and forth during the meeting, and a multi-step return breaks flow.

## Talking points to surface in the UI

The embedded Moobidesk stage is the presenter's chance to surface the messages that matter most for CX leaders:
- **Auto-categorisation cuts triage time** — the tag on every card proves the system read the reply
- **Smart-reply chips cut handle time** — chips that are specific to the category prove the system understood the intent
- **SLA timer creates urgency** — the red `2h:37m:46s/15m` timer shows the cost of slow response
- **Channel icon + conversation # + agent avatar** show the single-pane-of-glass omnichannel story

Don't add narration banners to the Moobidesk view itself — the fidelity fights the pitch. Put talking points in the presenter's notes, not on the canvas.

## Things that often break

- **Active card styling inconsistent** — if the dark charcoal active card doesn't invert every text element (name, phone, snippet), the card looks broken. Check the DOM after selecting — every text child should be white.
- **Stage swap leaves stale scroll position** — the chat area and conversations list both scroll. Reset both on `switchStage('moobidesk')`.
- **Smart-reply chips lag behind selection** — if the chips array doesn't re-render when `openMdConversation()` changes the customer, you end up with chips for the previous reply. Always rebuild chips inside `renderMdActiveConversation`.
- **FAB overlaps composer on narrow viewports** — absolute-positioned FAB + composer can collide below 1280px width. Constrain the Moobidesk view to min-width 1280px or move the FAB up.
- **Sending a reply doesn't advance to the next unhandled** — `sendMdReply` needs to call `openMdConversation(firstUnhandledCustomerId())` at the end. Without it, the presenter has to tap every conversation manually, which kills pacing.
