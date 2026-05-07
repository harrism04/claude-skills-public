# Integration Hooks

Real demos almost always need at least one click-out or cross-view — to MockPass, to a Moobidesk console, to a custom portal. These integrations are the "this is real, not a mockup" moments. Wire them carefully.

This reference has ready-to-paste handler snippets for the integrations 8x8 presenters use most often.

---

## Moobidesk Agent Console — two patterns

Moobidesk can appear in the prototype in two distinct ways. Pick based on what the pitch needs.

### Pattern A: Click-out handoff (opens external Moobidesk tenant in a new tab)

The lightest-weight option. Used when the demo wants to show the conversation transferring from automated WhatsApp into a live agent's queue, and the presenter has a Moobidesk tenant they can log into during the meeting. The audience sees the demo hand off and then sees the real tool open in another tab — the live system is the payoff.

**Trigger button**: usually "Speak to Agent" in the final service-hub step.

**Behavior**: when clicked, send an in-chat acknowledgement ("We're transferring you to an agent..."), then transform the main send button into "Open Moobidesk Agent Console" pointing at the live console URL.

**Snippet** — add this branch to `handleButtonClick()`:

```javascript
if (buttonText === 'Speak to Agent') {
  setTimeout(() => {
    const ack = document.createElement('div');
    ack.className = 'wa-message incoming';
    const ackTime = new Date().toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit' });
    ack.innerHTML = `
      <div class="msg-header">${BRAND_NAME} ✓</div>
      <div style="white-space: pre-line">We're transferring you to a customer service agent who can help.\n\nPlease hold on — an agent will be with you shortly.</div>
      <div class="msg-footer">${BRAND_NAME}</div>
      <div class="msg-time">${ackTime}</div>
    `;
    document.getElementById('waChat').appendChild(ack);

    // Transform send button
    const btn = document.getElementById('sendBtn');
    const btnText = document.getElementById('sendBtnText');
    btn.disabled = false;
    btn.classList.remove('complete');
    btn.style.background = '#1565c0';
    btnText.textContent = 'Open Moobidesk Agent Console';
    btn.onclick = () => window.open('https://e18.moobidesk.com/barfi-e19/8x8_trial/conversation', '_blank');
  }, 600);
}
```

**URL**: the canonical 8x8 trial Moobidesk URL is `https://e18.moobidesk.com/barfi-e19/8x8_trial/conversation`. For a customer-specific Moobidesk tenant, substitute the customer's URL.

**Gotcha**: customer-specific Moobidesk tenants often sit behind SSO. If the presenter isn't logged in during the dry run, the click-out 404s. Always log in before the meeting.

### Pattern B: Embedded Moobidesk view (in-prototype full recreation)

Used when the agent's reply experience is the pitch — ops-team demos, CRM bulk-send demos, or any time the audience is the CX manager rather than the end customer. Instead of opening an external URL, the prototype swaps the entire viewport into a faithful 3-column Moobidesk recreation (folder sidebar, conversations list, active conversation with composer).

This pattern is more work (you're rebuilding the inbox UI) but vastly higher-fidelity. The audience gets to watch replies land, tap auto-categorised conversations, fire smart-reply chips, and feel the full ops workflow without the presenter juggling tabs.

See `references/moobidesk-inbox.md` for the full pattern — layout, styling tokens, data shape, reply scenarios, and wiring.

**When to choose A vs B:**

| Factor | Pattern A (click-out) | Pattern B (embedded) |
|---|---|---|
| Audience | End-customer journey | Ops / CX team |
| Presenter has real Moobidesk tenant to demo? | Yes, use it | No, or audience can't see it |
| Audience cares about agent UX fidelity? | Low — it's a handoff moment | High — it's the centrepiece |
| Build cost | ~20 lines of JS | Full stage (see moobidesk-inbox.md) |
| Risk | SSO / popup blocker | Moobidesk UI changes → fidelity drift |

When in doubt for CRM / bulk-send / reply-handling use cases, choose B. When the demo is a customer acquisition journey that ends in a handoff, A is usually enough.

**Trigger patterns for Pattern B.** The button that fires `switchStage('moobidesk')` must always read in the patron's voice (or come from the orchestrator side) — never as an operator-labelled CTA inside a WhatsApp template bubble. Two clean patterns:

#### B1 — Patron-realistic CTA fires `switchStage`

When the journey naturally surfaces a "speak to a human" moment, a patron-voice CTA inside the template handles the handoff. The audience sees a believable customer-facing button; the handler routes operator-side.

```javascript
if (buttonText === 'Speak to Agent') {
  // Optional: emit a brief in-chat ack so the audience sees the handoff register
  setTimeout(() => switchStage('moobidesk'), 300);
  return;
}
```

#### B2 — Orchestrator send button transforms after the final step

When the final journey step is a freeform host reply with no patron CTA, the orchestrator's main "Send Next" button becomes the segue. This is the cleanest pattern for 1-to-1 demos that end on the host's personal acknowledgement — the transform makes it visually distinct from anything inside the phone preview, so the audience reads the segue as "we're now switching to the agent's view," not as a magical button on the customer's phone.

```javascript
// In renderSteps(), the existing tail block:
if (currentStep >= steps.length) {
  btn.classList.add('handoff');
  btn.onclick = () => switchStage('moobidesk');
  btnTxt.textContent = "Switch to Sarah's Moobidesk Console →";
}
```

Add the matching CSS so the transform is unmistakable (charcoal Moobidesk colour with a pulsing blue ring telegraphs "different kind of action"):

```css
.send-btn.handoff {
  background: linear-gradient(135deg, #2c3e50, #1f2c3a);
  color: #fff;
  box-shadow: 0 0 0 2px rgba(33,150,243,0.35), 0 4px 14px rgba(44,62,80,0.4);
  animation: handoffPulse 2.4s ease-in-out infinite;
}
@keyframes handoffPulse {
  0%, 100% { box-shadow: 0 0 0 2px rgba(33,150,243,0.35), 0 4px 14px rgba(44,62,80,0.4); }
  50%      { box-shadow: 0 0 0 6px rgba(33,150,243,0.10), 0 6px 20px rgba(44,62,80,0.5); }
}
```

**Anti-pattern to avoid.** A CTA button labeled "Open Host Console" or "Open Inbox" inside a WhatsApp template bubble. Operator-side language in a patron-facing template breaks the spell — patrons don't see those words on their real phones. If you need the segue to be button-driven from the chat side, use a patron-voice label ("Speak to Agent", "Speak to Concierge"). Otherwise route it through the orchestrator's main button (B2).

---

## Singpass / MockPass identity verification (Singapore)

For SG banking/insurance demos. Singpass is Singapore's national digital identity. The public test environment is **MockPass**, which is what you should link to in demos.

**Trigger button**: usually "Verify Identity ↗" in the KYC step.

**Behavior**: opens a companion `verify.html` page (in a new tab) with the application reference. That page has a "Log in with Singpass" button pointing at MockPass.

**Snippet** — add to `handleButtonClick()`:

```javascript
if (buttonText.startsWith('Verify Identity')) {
  const ref = REFERENCE_BY_MARKET[currentMarket];   // e.g. 'CC-2026-04158'
  window.open('verify.html?ref=' + encodeURIComponent(ref), '_blank');
  return;
}
```

**Companion `verify.html`** — create using `assets/companion-page-template.html` as the base. The MockPass URL it should redirect to:

```
https://test.api.myinfo.gov.sg/mockpass-sp/authorize
  ?scope=openid
  &response_type=code
  &redirect_uri=https%3A%2F%2Ftest.api.myinfo.gov.sg%2Fserviceauth%2Fmyinfo-com%2Fv1%2Fauthenticate
  &client_id=NDI-AUTHCOM
```

(Read the full URL on a single line — it's split here for readability.)

After the audience completes the MockPass flow, they tab back to the demo and the presenter clicks "Simulate verification complete" (a `customer_reply` step) to advance.

**Important**: SMS OTP for banking is banned in Singapore as of MAS guidance. If the demo is a Singapore bank, **always** use Singpass for verification — using SMS OTP misrepresents what the bank can actually do.

---

## 8x8 Verify API SMS OTP (Indonesia / non-Singapore markets)

For markets where SMS OTP is still appropriate. Renders in-chat as the `sms_otp` step type (yellow bubble) — see architecture.md.

**No companion page needed** — the OTP is shown inside the chat preview as if it landed via SMS. The "trigger" label should reference the 8x8 Verify API to telegraph that 8x8 is the orchestrator behind it:

```javascript
{
  id: 4, title: "SMS OTP Sent", type: "sms_otp",
  category: "AUTHENTICATION",
  trigger: "KYC check requires identity verification via 8x8 Verify API",
  preview: "A 6-digit verification code has been sent to ${name}'s phone via SMS.\n\n[SMS] Your ${BRAND} verification code is: 847293. Do not share this code. Expires in 10 minutes."
}
```

If the presenter wants to fire a *real* OTP during the demo (instead of a simulated one), they can call the 8x8 CPaaS MCP `initiate_verification` tool out-of-band before the meeting — the demo just shows what landed.

---

## 8x8 ChatApps API (live WhatsApp send during demo)

Some presenters want to fire a real WhatsApp template during the demo (instead of just the simulated chat). Pattern:

1. Use the `8x8-cpaas-se-toolkit:business-messaging` skill to send the template to the presenter's own phone before/during the meeting.
2. Project the presenter's phone screen alongside the demo HTML so the audience sees the message land for real.
3. The HTML demo continues to walk through all steps — the live send is a "look, this is real" moment, not a replacement for the orchestrator.

Don't try to fire live sends from the HTML itself. Browser CORS will block direct API calls, and embedding API keys in client-side HTML is a security gun pointed at the presenter's foot.

---

## Custom portal click-outs

For demos where the customer has their own portal (mobile banking app, insurance dashboard, retail order-tracking page) and you want the demo to "open" that portal:

**Snippet**:

```javascript
if (buttonText === 'View Order') {
  window.open('https://customer-portal-url.example/orders/' + orderRef, '_blank');
  return;
}
```

Confirm with the customer first whether the portal URL should be:
- their **production** URL (best for showing the round-trip feel — customer sees their real product)
- a **staging/sandbox** URL (avoids surprise data loads)
- a **mock page** the demo ships (full control, but loses the "real" punch)

Default to production unless the customer flags a concern.

---

## WhatsApp Pay / payment links

Most APAC banks support WhatsApp Pay or hand off to a payment portal. Pattern is the same as portal click-outs — the button opens a payment URL. If the demo is a generic "Pay Now" with no specific portal, link to a placeholder:

```javascript
if (buttonText === 'Pay Now ↗') {
  window.open('https://pay.example.com/' + invoiceRef, '_blank');
  return;
}
```

For a more polished demo, build a `pay.html` companion page that mimics the customer's payment UI (using their brand tokens) and shows a 3-step pay flow (amount → method → confirm).

---

## Companion page conventions

Companion pages (`verify.html`, `reschedule.html`, `pay.html`, etc.) are mini-pages that simulate the click-out destinations. Conventions:

- **Single file each**, same folder as `index.html`.
- **Read the application reference from the URL query string** (`?ref=CC-2026-04158`) so the demo passes context.
- **Match the client brand** — use the same `:root` CSS tokens.
- **Keep them under 100 lines** — they exist to look real, not to be functional.
- **End with a "complete" button** that closes the tab or shows a "Return to WhatsApp" message. The actual round-trip is the presenter clicking "Simulate verification complete" back in the orchestrator.

Use `assets/companion-page-template.html` as the starting point.

---

## Things that often break

- **CTA button text doesn't match the `handleButtonClick()` branch** — silent fail, button just renders the outgoing bubble. Always test every button.
- **Companion page doesn't open** — popup blocker. Tell the presenter to allow popups for the demo URL during their dry-run.
- **MockPass URL changes** — the URL is occasionally rotated by GovTech. If MockPass returns 404, check `https://api.singpass.gov.sg` for the current test endpoint.
- **Moobidesk URL behind SSO** (Pattern A) — the trial URL is open, but customer-specific Moobidesk tenants may require SSO. If the presenter isn't logged in, the click-out 404s. Ask them to log in to Moobidesk in the same browser before the demo.
- **Embedded Moobidesk stage fidelity drift** (Pattern B) — Moobidesk's real UI evolves; an old screenshot-based recreation can start looking dated. Re-screenshot before every demo cycle to a new customer.
