# 1-to-Many Broadcast Mode

The broadcast view replaces the right-hand phone preview with a campaign delivery dashboard. Use this when the pitch is about **scale, deliverability, or campaign orchestration** — CRM bulk sends, payment reminders, OTP volumes, promo waves, transit/aviation disruption blasts. The audience is the marketer, the ops manager, or the CRM lead — not the end customer.

This reference covers when to use the mode (vs 1-to-1), the dashboard layout, the data shape, the per-recipient state machine, the animation pacing, and the wiring to chain into an embedded 8x8 Converse reply stage (formerly Moobidesk).

## When to use (vs 1-to-1)

Use broadcast mode when:
- The customer's volume is the story (e.g. "we send 2.4M payment reminders a month")
- Deliverability, throughput, or per-segment opt-in handling is part of the pitch
- The journey starts with a CRM trigger (bulk export → bulk send) rather than a single customer event
- The demo flows from broadcast → reply handling (the HLF-style ops console combo, see "Bridging to 8x8 Converse" below)

Use 1-to-1 mode when the pitch is about journey *design* — Flows, conversational depth, interactive CTAs. A single customer journey with rich content beats a 12-row delivery table for that conversation.

A demo can start in broadcast and end in 1-to-1 — e.g. a campaign fires, one recipient replies, the right pane swaps from the dashboard to a phone preview to walk through that single conversation. Don't try to show both panes at once; swap.

## Layout

The broadcast preview pane keeps the same outer chrome as 1-to-1 mode (control panel on the left, preview on the right) but replaces the WhatsApp phone shell with three stacked sections in the right pane:

1. **Campaign summary card** (top, ~140px tall) — campaign name, template name + category, sender business profile, scheduled time, total recipients, live counters for `Queued / Sent / Delivered / Read / Replied / Failed`. Counters animate as the stages play.
2. **Recipient table** (middle, scrollable, flex: 1) — one row per recipient. Per-recipient state, timestamp, channel cost (optional), latest message snippet. State pills animate from `Queued` → `Sent` → `Delivered` → `Read` → optional `Replied` / `Failed`.
3. **Drill-in preview** (bottom, collapsible) — when a row is clicked, the bottom expands into a compact WhatsApp bubble preview showing what *that* recipient received (parameter substitution made real). Closes back to the table when another row is clicked or the close caret is tapped.

The control panel on the left keeps the existing structure — campaign metadata where the customer profile would normally sit, stages list where the steps list would normally sit, send button at the bottom.

Don't add a third top-level pane. The drill-in is part of the preview pane, not a sibling — putting it in a third column makes the dashboard feel like a CRM, not a comms console.

## Stages

A broadcast demo has **4–6 stages** (vs 6–10 steps for 1-to-1). Each stage maps to a real campaign phase, and the dashboard animates through it before the next stage triggers. Canonical sequence:

1. **Campaign queued** — recipients flip from blank to `Queued` in a single fast pass. Trigger label: `CRM segment exported · 2,847 recipients`.
2. **Send fires** — recipients flip `Queued` → `Sent` in a staggered cascade (~30–60ms per row for the first 8 rows visible; the rest tick over silently). Counters update live. Trigger: `8x8 CPaaS Business Messaging API · template approved`.
3. **Delivery callbacks land** — recipients flip `Sent` → `Delivered` with the same staggered animation, slightly slower (~60–100ms). A small percentage (~3%) flip to `Failed` instead. Trigger: `Meta delivery callbacks received`.
4. **Read receipts arrive** — most `Delivered` rows flip to `Read`, again staggered. A handful stay `Delivered` (recipients with read receipts disabled). Trigger: `Meta read callbacks received`.
5. **Replies start coming in** — 5–8 selected rows flip to `Replied` and surface an auto-category tag inline. This is the seam where a Converse handoff makes sense — see "Bridging to 8x8 Converse". Trigger: `Inbound replies routed to 8x8 Converse`.
6. **Optional: campaign summary / settlement** — final stage that swaps the campaign card to a "Campaign complete" view with delivery rate, read rate, reply rate, opt-out count. Useful for finance/CFO-leaning audiences. Trigger: `Reporting batch finalised`.

Authenticity tip: the staged animation is the pitch. A dashboard that snaps from empty to full in one frame *looks* faked. The cascade is what sells "this is a real distributed system processing callbacks one at a time".

## CSS tokens for broadcast

Add these to `:root` alongside the brand and WhatsApp tokens. Use semantic names; the colours should not change per client.

```css
:root {
  /* ... existing brand and WhatsApp tokens ... */

  /* Broadcast / delivery state tokens */
  --bc-queued:    #90a4ae;   /* slate grey: queued, neutral */
  --bc-sent:      #1976d2;   /* blue: sent to upstream */
  --bc-delivered: #388e3c;   /* green: confirmed delivered */
  --bc-read:      #00695c;   /* teal-dark: read receipt */
  --bc-replied:   #6a1b9a;   /* purple: customer replied (matches Converse Responded dot) */
  --bc-failed:    #c62828;   /* red: failed delivery */
  --bc-row-hover: #f5f7fa;
  --bc-row-active: #eef3f9;
  --bc-table-border: #e8eaed;
}
```

The `--bc-replied` colour intentionally matches Converse's `--cv-responded`. When the dashboard transitions into the embedded Converse view, the same purple dot follows the customer across panes — small detail, big "this is one platform" payoff.

## Data shape

Two objects drive the broadcast view. Keep them parallel to `DEMO_DATA` in shape so the orchestrator code style stays consistent.

### `CAMPAIGN` (campaign metadata)

```javascript
const CAMPAIGN = {
  id: "CAMP-2026-04-1183",
  name: "April Loan Payment Reminders",
  template: "loan_reminder_v2",
  category: "UTILITY",                       // MARKETING | UTILITY | AUTHENTICATION | SERVICE
  scheduledAt: "2026-04-22 09:00 SGT",
  sender: { businessName: "Acme Bank", verified: true, avatarText: "AB" },
  totalRecipients: 2847,                     // shown in the summary; bigger than visible row count
  visibleRecipients: 12,                     // rows actually rendered in the table (the rest tick silently)
  countCounters: {
    queued: 2847, sent: 0, delivered: 0, read: 0, replied: 0, failed: 0
  }
};
```

`visibleRecipients` is the cosmetic knob that keeps the demo readable. The animation cascade plays across the visible rows (so the audience sees individual states change); behind the scenes the counters tick up to `totalRecipients` over the same duration, giving the impression of a real high-volume send.

### `RECIPIENTS` (the visible row list)

Same shape as `TODAYS_BATCH` in `references/converse-inbox.md` — this is deliberate. If the demo combines broadcast and Converse, both views read from the same recipient list, so the customer the marketer watches reply in the dashboard is the same customer the agent picks up in the inbox.

```javascript
const RECIPIENTS = [
  { id: 1, name: "Sarah Tan", phone: "+65 9123 4567", initials: "ST",
    loanRef: "PL-2026-00742", amount: 847.50, dueDate: "2026-04-22",
    state: "queued", lastEvent: null, willReply: true },
  { id: 2, name: "Dewi Anggraini", phone: "+62 812-3456-7890", initials: "DA",
    loanRef: "PL-2026-00891", amount: 1240.00, dueDate: "2026-04-22",
    state: "queued", lastEvent: null, willReply: true },
  { id: 3, name: "Ahmad Rashid", phone: "+60 12-345 6789", initials: "AR",
    loanRef: "PL-2026-00903", amount: 562.30, dueDate: "2026-04-23",
    state: "queued", lastEvent: null, willReply: false, willFail: true },
  // ... 8–12 more
];
```

`willReply` and `willFail` are scenario flags consumed by stage 3 and stage 5. Do not pick them at random — choose deliberately so the dashboard tells a story (e.g. row 3 fails because the number is invalid, row 7 replies "Will Pay", row 9 replies "Defer Request"). When the demo combines broadcast + Converse, the replied rows are the ones the agent triages in the inbox — make sure the categories cover the same five auto-categories listed in `converse-inbox.md` (`Will Pay`, `Defer Request`, `Acknowledged`, `Dispute`, `Speak to Agent`).

### State machine

Per-recipient state transitions are linear and one-way:

```
queued → sent → delivered → read → replied
            ↓
          failed   (terminal)
```

A recipient can branch to `failed` from `sent` or `delivered`. Once `failed`, no further transitions. `replied` is terminal too — replies are handled in Converse, not back in the dashboard.

## Animation pacing

The cascade is what sells the demo. Get it wrong and the dashboard either feels frozen (too slow) or fake (too fast). Tested defaults:

| Stage | Per-row stagger | Notes |
|---|---|---|
| `Queued` cascade | 20–30ms | Fast — this is the cheap step |
| `Sent` cascade | 30–60ms | Slightly slower; sells the upstream API call |
| `Delivered` cascade | 60–100ms | Slowest visible cascade; sells round-trip to Meta |
| `Read` cascade | 50–80ms | Medium |
| `Replied` cascade | 200–400ms | Deliberately slower — replies are individual events worth lingering on |

Counters update on every row transition, not on a separate timer. This keeps the numbers in lockstep with the rows.

If pacing feels off in your demo, tune in 10ms increments. Don't drop below 20ms per row — the eye blurs the cascade together and it just looks like a snap-fill.

## Wiring

The broadcast view is a parallel state machine to the 1-to-1 view. Don't try to share the `currentStep` global; introduce a stage-aware control flow:

```javascript
let currentStage = 'workflow';        // 'workflow' | 'moobidesk'
let currentBroadcastStage = 0;        // index into BROADCAST_STAGES

function playBroadcastStage(stageIdx) {
  const stage = BROADCAST_STAGES[stageIdx];
  // 1. surface the trigger label in the control panel
  setActiveStageCard(stageIdx);
  // 2. run the cascade across visible recipients
  cascadeRecipientStates(stage.fromState, stage.toState, stage.staggerMs, stage.targetIds);
  // 3. tick the campaign counters
  animateCounters(stage.counterDeltas, stage.durationMs);
}
```

Minimum function set:

- `renderCampaignCard()` — re-renders the top summary from `CAMPAIGN`. Called on init and after each stage.
- `renderRecipientTable()` — initial render from `RECIPIENTS`. After init, prefer per-row updates (`updateRecipientRow(id, newState)`) over full re-renders to keep the cascade smooth.
- `updateRecipientRow(id, newState)` — flips a single row's state pill, snippet text, and timestamp. Adds the `.row-replied` highlight if `newState === 'replied'`.
- `cascadeRecipientStates(from, to, staggerMs, targetIds)` — orchestrates the staggered animation. Takes optional `targetIds` so stages can target subsets (e.g. only `willFail` rows in the failure pass).
- `animateCounters(deltas, durationMs)` — eases the numeric counters from current to current+delta over the duration. Use a simple 60fps tween, not CSS transitions on text content.
- `openRecipientDrillIn(id)` — expands the bottom drill-in pane with the WhatsApp bubble preview for that recipient.
- `closeRecipientDrillIn()` — collapses the drill-in.
- `playNextBroadcastStage()` — main control-panel send button. Increments `currentBroadcastStage`, calls `playBroadcastStage()`.

The control panel send button label updates per stage: `Queue Campaign` → `Send to 2,847 recipients` → `Receive delivery callbacks` → `Receive read receipts` → `Surface 7 replies` → `Open Reply Inbox →` (this last label transforms the button into a `switchStage('moobidesk')` trigger; see below).

## Bridging to 8x8 Converse (the HLF-style combo)

The most common combination demo: broadcast fires → replies surface → presenter clicks into the embedded Converse inbox to triage them. This is the canonical "8x8 single-platform CX ops" pitch (CPaaS broadcast + Converse agent console, formerly Moobidesk).

Three legitimate triggers for the segue (mirror `references/converse-inbox.md` "Navigation hooks"):

1. **Send-button transform after stage 5** — the cleanest. After the `Replied` cascade finishes, transform the orchestrator send button into "Open Reply Inbox →" with `btn.classList.add('handoff')` and `btn.onclick = () => switchStage('moobidesk')`. The button is unmistakably an orchestrator action and it lives in the control panel, not in any phone preview, so there is no "operator language inside a patron bubble" violation.
2. **Click-through on a `Replied` row** — tapping a replied row in the dashboard fires `switchStage('moobidesk')` and `openMdConversation(row.id)`. Useful when the presenter wants to drill into a specific reply category. Requires that the recipient's `id` matches the customer id in `TODAYS_BATCH`/`REPLY_SCENARIOS`.
3. **Explicit "Open Reply Inbox" stage** — a sixth stage card in the control panel that does nothing but call `switchStage('moobidesk')`. Use when the presenter wants a clean discrete moment to introduce the inbox to a non-technical audience ("now let's see what the agent sees").

When entering the Converse view, `RECIPIENTS` rows that are `Replied` should already be present in `TODAYS_BATCH` with matching ids. The unhandled count badge in the inbox sidebar should reflect those rows. If the counts don't reconcile, the seam is broken — the presenter loses the "this is one platform" beat.

## Talking points to surface in the UI

The broadcast stage is where a CX or marketing leader sees throughput, deliverability, and segment intelligence. Surface:

- **Template category visible per row** — proves the WABA pricing model is understood (UTILITY here, not MARKETING)
- **Per-recipient state granularity** — proves callbacks are ingested at the message level, not just bulk acknowledgement
- **Failure breakdown on hover** — `failed` rows tooltip the reason (`Number not registered on WhatsApp`, `Recipient blocked sender`, `Throughput limit · retry queued`). Proves the platform handles the unhappy path, which is what marketers actually worry about.
- **Reply auto-category** — the inline tag on a `Replied` row proves the system reads the inbound, sets up the Converse segue
- **Counters reach `totalRecipients`, not `visibleRecipients`** — proves the cascade is cosmetic, the volume is real

Keep narration banners off the dashboard surface. The pitch is the dashboard. Talking points belong in the presenter's notes, not on the canvas.

## Things that often break

- **Counter and row state out of sync.** Easy to introduce when one stage updates rows but forgets to tick the campaign card. The card is the audience's anchor — fix it first if a number looks off.
- **Cascade animation jitters on Safari.** `setTimeout` chains drift on background tabs in Safari. Use `requestAnimationFrame` for the cascade timing if jank shows up; the stagger maths stays the same.
- **Replies surface for rows that don't exist in `REPLY_SCENARIOS`.** Stage 5 picks `willReply: true` rows from `RECIPIENTS`, but if `REPLY_SCENARIOS` (in converse-inbox.md's data shape) doesn't have entries for those ids, the inbox handoff lands on an empty conversation. Keep the two lists referentially aligned — same ids, same names.
- **Drill-in WhatsApp bubble doesn't substitute parameters.** The summary card shows the template body with `{{1}}`, `{{2}}` placeholders; the drill-in must substitute with that recipient's actual values (`recipient.name`, `recipient.amount`). If the drill-in still shows raw placeholders, the demo regresses to "this is just a CSV uploader" instead of "this is real per-recipient personalisation".
- **Counters tween past `totalRecipients`.** Floating-point easing can overshoot. Clamp the final value with `Math.min(target, current + delta)` on each frame.
- **Send-button transform fires before cascade finishes.** If you transform the button into "Open Reply Inbox →" inside the stage 5 trigger but the cascade is still running asynchronously, the audience sees the segue button before the replies have surfaced. Wait for the cascade to complete (callback or `await`) before flipping the button.
- **Stage-card highlights don't reset on `resetDemo()`.** A presenter resetting between meetings expects a clean slate. Call `setActiveStageCard(0)` and reset every recipient's state to `queued` plus zero all counters except the `queued` total.
