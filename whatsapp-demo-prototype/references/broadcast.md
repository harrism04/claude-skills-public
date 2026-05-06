# 1-to-Many (Broadcast / Campaign) Mode

The default 1-to-1 demo walks one persona through one journey. Broadcast mode is different in shape: the audience needs to see a campaign going out to *many* recipients, with per-recipient delivery + read + reply states animating in real time. This is the right fit for marketing campaigns, payment-due waves, OTP volumes, retail promos, polling, CRM bulk reminders, and any demo where the question is "show me throughput, not just one happy path."

## When to use broadcast vs 1-to-1

Pick **1-to-1** when the pitch is about journey design, conversational depth, or interactive features (Flows, CTAs, agent handoff). The audience is being asked to imagine themselves as the customer.

Pick **broadcast** when the pitch is about scale, deliverability, or campaign orchestration. The audience is being asked to imagine themselves as the marketer or ops manager. Think: "We're sending 50,000 promo messages — watch them deliver and watch the click-through happen."

Some demos benefit from both. Add a top-level mode toggle to the control header (`◯ 1-to-1  ◯ Broadcast`) and swap the preview pane based on the selection. Keep `DEMO_DATA` separate (`DEMO_DATA_1TO1` and `DEMO_DATA_BROADCAST`) so the data shapes can diverge.

## Broadcast layout

The right-pane preview becomes a **delivery dashboard** instead of a single chat. Recommended split:

- **Top quarter** — campaign summary card: campaign name, template used, total recipients, headline counters (Sent / Delivered / Read / Replied / Failed) updating in real time as the broadcast fires.
- **Middle half** — recipient table: row per recipient with name, phone, locale, status pill (Queued → Sent → Delivered → Read → Replied), and last-event timestamp. New rows light up as they progress.
- **Bottom quarter** — preview of one selected recipient's WhatsApp conversation, so the audience can drill in. Click any row to swap the chat view.

The left control panel changes too:
- Header: "Send Campaign" instead of "Send Next Message"
- Step list becomes: 1) Compose template, 2) Upload audience, 3) Schedule, 4) Send (the big button). Optional 5) Reply handling, 6) Export results.

## Recipient list

Generate a believable list of 8–20 recipients. Don't go higher — table fatigue sets in and the per-row animation gets lost.

```javascript
const RECIPIENTS = [
  { id: 1, name: "Sarah Tan",      phone: "+65 9123 4567", locale: "en_SG", segment: "Premier" },
  { id: 2, name: "Dewi Anggraini", phone: "+62 812 3456 7890", locale: "id_ID", segment: "Preferred" },
  { id: 3, name: "Ahmad Razak",    phone: "+60 12 345 6789", locale: "ms_MY", segment: "Standard" },
  // ... 5–17 more
];
```

Include realistic locale + segment mix — the audience reads the table for diversity signals. For multi-country campaigns, weight the list toward the markets the customer actually operates in (don't put 18 SG numbers in an Indonesia-only broadcast).

## State progression

Each recipient transitions through states with realistic-feeling timing:

```
QUEUED  →  SENT     (50–200ms after campaign fires)
SENT    →  DELIVERED (300–1500ms — varies per recipient)
DELIVERED → READ    (1–8s — most recipients, ~70% will hit READ in demo)
READ    →  REPLIED  (3–15s — small fraction, ~20%, only for journeys with CTA)
SENT    →  FAILED   (rare — 1–3% — to show error handling)
```

Implement with `setTimeout` and randomized delays per recipient. The demo doesn't need to hit perfect proportions — what matters is that the animation feels organic, not lock-step.

```javascript
function fireCampaign() {
  RECIPIENTS.forEach((r, i) => {
    const sentAt    = 50  + Math.random() * 200;
    const deliveredAt = sentAt + 300 + Math.random() * 1200;
    const readAt    = deliveredAt + 1000 + Math.random() * 7000;
    const repliedAt = readAt + 3000 + Math.random() * 12000;

    setTimeout(() => updateStatus(r.id, 'SENT'), sentAt);
    setTimeout(() => updateStatus(r.id, 'DELIVERED'), deliveredAt);
    if (Math.random() < 0.7) setTimeout(() => updateStatus(r.id, 'READ'), readAt);
    if (Math.random() < 0.2) setTimeout(() => updateStatus(r.id, 'REPLIED'), repliedAt);
    if (Math.random() < 0.02) setTimeout(() => updateStatus(r.id, 'FAILED'), sentAt + 500);
  });
}
```

Update the headline counters in the same `updateStatus` function.

## Status pill colors

Match the WABA delivery model the audience knows:

| Status | Color | Background |
|---|---|---|
| QUEUED | `#888` | `#f0f0f0` |
| SENT | `#1565c0` | `#e3f2fd` |
| DELIVERED | `#2e7d32` (single tick) | `#e8f5e9` |
| READ | `#0288d1` (double blue tick) | `#e1f5fe` |
| REPLIED | `#6a1b9a` | `#f3e5f9` |
| FAILED | `#c62828` | `#ffebee` |

Use the WhatsApp tick glyphs (✓ for sent, ✓✓ in grey for delivered, ✓✓ in blue for read) inside the pill — it telegraphs "this is the same delivery model you know from your own WhatsApp."

## Reply handling

If the campaign template has buttons, simulate replies coming back. Pick 2–4 recipients and animate them into the REPLIED state. When the user clicks a REPLIED row, the bottom chat preview should show the inbound reply, and offer two next-actions on the left:

- "Auto-reply with template X" — fires another template back to that recipient
- "Route to agent" — transforms into a Moobidesk handoff, like the 1-to-1 pattern

This lets the demo cross from broadcast into conversational follow-up, which is often the real value prop ("we don't just send — we listen and route").

## Schedule + audience steps

Even though the "send" is the climax, the prep steps matter for the pitch. Keep them:

1. **Compose template** — show the template body and parameters; let the SE click "Edit" to open a small modal with template name + body + variable list.
2. **Upload audience** — show a file-drop area (mocked) that "uploads" `recipients_q2_promo.csv`, then displays "18 recipients · 3 locales · estimated cost: $0.42 SGD".
3. **Schedule** — "Send now" or "Schedule for [date]" with a date picker. Default to "Send now" for live demos.
4. **Send** — the big button.

## Cost estimation

A nice touch for marketing-side audiences: surface a per-message cost estimate in the schedule step, broken down by template category (MARKETING vs UTILITY vs AUTHENTICATION) and destination market. WABA pricing varies by both, so showing the breakdown demonstrates the SE understands their economics.

```
Estimated cost (USD)
  18 messages × $0.0233 (SG MARKETING) = $0.42
  Includes Meta conversation pricing + 8x8 carrier fees
```

Use illustrative pricing — don't quote real WABA rates unless the user has confirmed them, since they change.

## What NOT to do in broadcast mode

- Don't show 1000 recipients. Animation chokes, and the audience can't follow what's happening.
- Don't use real phone numbers. Generate plausible local-format numbers per market.
- Don't claim 100% delivery. Show 1–3% FAILED to make the demo credible. Failure is normal in real WABA traffic.
- Don't skip the cost estimate. Marketing audiences ask "how much?" within 30 seconds. Have the answer on screen.
