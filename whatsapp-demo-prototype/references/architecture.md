# Architecture Reference

This is the deep dive on how the demo orchestrator works internally. Read this before scaffolding a new demo so the JS keeps working without modification.

## File layout

```
demos/<client-slug>/
├── index.html              ← the main orchestrator (always)
├── verify.html             ← optional: identity verification companion page
├── reschedule.html         ← optional: any other CTA target
└── ...
```

Everything ships as inline HTML/CSS/JS. No build, no bundler, no node_modules. The only "imports" are favicon-style data URIs.

## Two-pane layout (1-to-1 and broadcast modes)

The body is a flex container with two children:

- `.control-panel` (width 420px, fixed) — the SE-facing surface. Header with client name, market tabs, customer profile card, scrollable steps list, send button, reset button.
- `.preview-panel` (flex: 1) — the audience-facing surface. In 1-to-1 mode: WhatsApp business header, scrolling chat area, optional WhatsApp Flow overlay. In broadcast mode: campaign summary + recipient table + drill-in chat.

Don't change these widths casually. 420px is wide enough for step cards with category tags + trigger labels but narrow enough that the chat preview gets the visual weight.

## Full-viewport swap (embedded Moobidesk mode)

For demos that include an embedded Moobidesk stage, wrap the two-pane layout in `.workflow-view` and add a sibling `.moobidesk-view` that occupies the full viewport. A `switchStage(stage)` function toggles `display: none` between them. See `references/moobidesk-inbox.md` for layout and styling.

## CSS token system

All branding lives in `:root` CSS variables. Re-skinning is a 4-line edit:

```css
:root {
  --brand-primary:   #003DA5;  /* client primary */
  --brand-secondary: #D71920;  /* client accent */
  --brand-text-on:   #ffffff;  /* text color on primary background */
  /* WhatsApp tokens — keep unchanged */
  --wa-green: #075E54;
  --wa-light: #DCF8C6;
  --wa-bg:    #E5DDD5;
  --wa-dark:  #128C7E;
}
```

Use the WhatsApp tokens for chat chrome (header, message bubbles, send button) and brand tokens for the control panel header, step card accents, customer avatar.

If the demo includes an embedded Moobidesk stage, add the Moobidesk tokens too — see `references/moobidesk-inbox.md`.

## DEMO_DATA shape

The orchestrator reads everything from a single global `DEMO_DATA` object. For a multi-market demo, the top-level keys are market codes; for single-market, use one key (e.g. `default`).

```javascript
const DEMO_DATA = {
  sg: {
    name: "Singapore",
    customerName: "Sarah",
    phone: "+65 9123 4567",
    segment: "Premier Banking",
    cardOffer: "Platinum Card",           // shown as a profile tag
    steps: [ /* see step types below */ ]
  },
  id: { /* same shape, different content */ }
};
```

The profile fields (`segment`, `cardOffer`) are rendered as colored tags on the customer card. Add or remove tag fields by editing `renderProfile()`.

## Step types

Every entry in `steps` has a `type`. Five types are supported out of the box.

### `template` — outbound WhatsApp Business template

The default. Renders as an incoming WhatsApp bubble with header, body, footer, and optional CTA buttons.

```javascript
{
  id: 1,
  title: "Pre-Approval Offer",
  template: "cc_preapproval",           // template name shown as a tag
  category: "MARKETING",                // MARKETING | UTILITY | AUTHENTICATION | SERVICE
  type: "template",
  trigger: "CRM pre-approval engine flags eligible customer",
  params: { header: ["Sarah"], body: ["Platinum Card", "8%"] },
  preview: "You're Pre-Approved, Sarah!\n\n...",   // what the audience sees
  footer: "{{BRAND_NAME}} — {{BRAND_TAGLINE}}",
  // Buttons — ALWAYS object form, never plain strings
  buttons: [
    { type: "reply", label: "Apply Now" },   // quick reply — no icon
    { type: "url",   label: "Learn More" }   // CTA URL — renders with external-link icon
  ]
}
```

`params` is purely cosmetic — it isn't substituted into `preview`. The author writes the final string in `preview` directly. `params` exists so the audience sees that the template *has* parameters (some SEs surface these in a tooltip).

**CTA block layout**: buttons render as a SEPARATE `wa-cta-block` div appended **below** the bubble div, not inside it. This matches the real WA Business template structure — body/footer and buttons are distinct visual containers separated by a hairline. Never nest buttons inside the bubble. The `buildTemplateBubble()` function in the starter template handles this correctly; preserve this pattern when building custom demos.

### `customer_reply` — simulated user tap

```javascript
{
  id: 2,
  title: "Customer Replies",
  type: "customer_reply",
  message: "Apply Now",                          // text shown in outgoing bubble
  trigger: "Customer taps quick-reply button",
  triggersFlow: true,                            // optional: opens WhatsApp Flow overlay
  category: "REPLY"
}
```

If `message` matches a button label on the previous step, clicking that button auto-advances. This gives the SE two paths: click the in-chat button OR press the main "Simulate" button.

`triggersFlow: true` triggers the WhatsApp Flow overlay 400ms after the bubble lands.

### `sms_otp` — out-of-band SMS

Renders as a yellow-tinted bubble with a "📱 SMS (separate channel)" header. Use when the demo crosses channels (e.g. an Indonesia journey uses SMS OTP because Singapore's MAS guidance bans SMS OTP for banking).

```javascript
{
  id: 4,
  title: "SMS OTP Sent",
  type: "sms_otp",
  category: "AUTHENTICATION",
  trigger: "KYC check requires identity verification via OTP",
  preview: "A 6-digit verification code has been sent to Dewi's phone.\n\n[SMS] Your verification code is: 847293..."
}
```

The bubble strips the first paragraph (the descriptive sentence) and shows only the SMS body in monospace-ish styling. Adjust by editing the `sms_otp` branch in `sendNext()`.

### `system_event` — centered status label

Renders as a centered green pill (like "Today" date dividers, but green for success). Use for events that aren't messages — KYC validated, payment processed, callback received.

```javascript
{
  id: 5,
  title: "Identity Verified",
  type: "system_event",
  category: "SYSTEM",
  trigger: "OTP validated by 8x8 Verify API",
  preview: "Identity verification successful. Resuming WhatsApp conversation..."
}
```

### `agent_message` — outbound from a live agent (optional)

Use after a Moobidesk handoff. Renders as an incoming bubble but with an agent avatar and "Agent · <name>" header instead of the business name.

```javascript
{
  id: 10,
  title: "Agent Greeting",
  type: "agent_message",
  agentName: "Priya",
  trigger: "Conversation routed to Tier-1 agent in Moobidesk",
  preview: "Hi Sarah, this is Priya from Card Services..."
}
```

Add this branch to `sendNext()` if the demo needs it — it's not in the starter template by default. If the demo uses the embedded Moobidesk mode, agent replies live inside the Moobidesk view instead — see `references/moobidesk-inbox.md`.

## State machine

Three globals drive everything:

```javascript
let currentMarket = 'sg';     // active market key in DEMO_DATA
let currentStep = 0;          // index into DEMO_DATA[currentMarket].steps
let sentSteps = new Set();    // indices of steps already played
```

Three functions are the entire control flow:

- `renderProfile()` — re-renders the customer card from `DEMO_DATA[currentMarket]`.
- `renderSteps()` — re-renders the left-side step list and the send button label.
- `sendNext()` — plays the current step, increments `currentStep`, calls `renderSteps()`.

`switchMarket(market)` resets everything and re-renders.

`resetDemo()` clears the chat, resets state, calls `renderSteps()`.

This is small on purpose. Don't add a router or state library.

For demos with stage-based navigation (broadcast or embedded Moobidesk), add a `currentStage` global and a `switchStage(stage)` function that toggles view visibility — see `references/moobidesk-inbox.md` for the pattern.

## Button click flow

`handleButtonClick(buttonText)` is the catch-all for in-chat button taps. The default behavior:

1. If the next step is a `customer_reply` whose `message` equals `buttonText`, advance the demo (calls `sendNext()`).
2. Else, render an outgoing bubble with the button text and run any custom branches (CTA opens, handoff transforms).

Add custom branches by name, not by index. Example:

```javascript
if (buttonText.startsWith('Verify Identity')) {
  window.open('verify.html?ref=' + reference, '_blank');
  return;
}
if (buttonText === 'Speak to Agent') {
  // either: transform send button into "Open Moobidesk Console" (click-out)
  // or:     switchStage('moobidesk') to enter the embedded inbox
  ...
}
```

## WhatsApp Flow overlay

A bottom-sheet overlay that slides up over the chat to simulate a multi-screen WhatsApp Flow form. Driven by a separate `FLOW_SCREENS` constant:

```javascript
const FLOW_SCREENS = {
  sg: [
    { title: "Personal Details", fields: [
      { label: "Full Name", type: "input", value: "Sarah Tan Wei Ling" },
      { label: "NRIC", type: "input", value: "S8912345A" },
      ...
    ]},
    { title: "Employment Details", fields: [...] },
    { title: "Review & Submit", review: true }   // last screen renders the review summary
  ]
};
```

The overlay is opened by `showWhatsAppFlow()` (called when a step has `triggersFlow: true`). Navigation is Back / Next / Submit. Submitting closes the overlay and auto-advances the demo by one step (so the next step in `DEMO_DATA` should be the "Application Received" confirmation).

If your demo doesn't use a Flow, **delete the `wa-flow-overlay` div, the `FLOW_SCREENS` constant, and all `flow*` functions**. They're cosmetically irrelevant when unused but they bloat the file.

## Typing indicator

Templates and SMS messages get an 800ms typing indicator before they appear. Customer replies and system events skip it (they should feel instant). This is hardcoded as `delay = 800` in `sendNext()`. If pacing feels off in your demo, tune to 600–1000ms but don't go below 400ms — the indicator becomes invisible.

## Things that often break

- **Missing `category` on a step** — the cat-* CSS class won't apply, the tag renders unstyled. Always set one of `MARKETING | UTILITY | AUTHENTICATION | SERVICE | SYSTEM | REPLY`.
- **Button label with apostrophes or double quotes** — escaped wrong in `onclick` attribute. The starter template uses `b.replace(/'/g, "\\'")` but if you add new escaping, double-check.
- **Multi-market step arrays of different lengths** — `currentStep` becomes invalid after a market switch. The template resets it to 0 on switch, so step counts can differ, but visually it's confusing. Keep step counts equal across markets when possible.
- **`triggersFlow: true` but `FLOW_SCREENS[market]` is undefined** — overlay opens empty. Either define screens for both markets or remove the trigger.
- **Stage swap leaves stale state** — when `switchStage()` enters the Moobidesk view, reset or refresh the conversations list (`renderMoobiConvList()`); don't assume the user hasn't already clicked around.
