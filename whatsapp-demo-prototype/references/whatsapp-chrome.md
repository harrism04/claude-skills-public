# WhatsApp Chrome — Authenticity Spec

The starter template implements all of these. Do not regress when re-skinning. This is the visual ground truth — when the prototype feels off, it's almost always one of these regressing silently.

## Phone shell (required for 1-to-1 mode)

- Dark phone shell, ~44px corner border-radius, subtle border, box-shadow for depth.
- Dynamic Island / notch as a dark pill at top-center, z-index above status bar.
- Home indicator bar at bottom (thin pill, ~130px wide, rgba white, centered in a dark strip).

## Status bar

- Background `#025144` (the teal dark edge, not the main teal).
- Left: live clock (update every 10s via `setInterval`). Right: signal bars + WiFi + battery icons in SVG.
- Height 52px with content aligned to bottom-edge (simulates notch safe-area).

## WA navigation bar (topbar)

- Back chevron `‹` + circular business avatar (initials, white bg, brand-primary text) + business name + verified badge + subtitle "Business Account · Verified" + three right-side icons (video, phone, vertical-dots) all in SVG.
- Verified badge: small `#53bdeb` teal circle with white checkmark SVG inside — NOT a Unicode `✓`.
- Background: `#008069` (WA teal, slightly lighter than header edge).

## Chat area / wallpaper

- `background-color: #efeae2` (the real WA beige).
- `background-image`: SVG tile of the authentic leaf/petal pattern at ~35% opacity in `#c8bfb0`. Do NOT use a generic diamond grid.
- Custom scrollbar: 4px wide, transparent track, semi-transparent dark thumb.

## Message bubbles

- **Incoming (business)**: white `#ffffff` bg, `border-top-left-radius: 2px`, CSS triangle tail top-left using `::before` (`border-top: 8px solid #fff; border-left: 8px solid transparent`).
- **Outgoing (customer)**: WA green `#d9fdd3` bg, `border-top-right-radius: 2px`, CSS triangle tail top-right.
- Bottom padding `22px` on both to make room for the absolute-positioned meta row.
- **Meta row**: `position:absolute; bottom:5px; right:9px` — time string + tick glyphs.
- **Ticks**: `✓✓` characters styled via CSS. Grey `#667781` for delivered, blue `#53bdeb` for read. Do NOT use emoji or images.
- **Sender name row** inside business bubble: brand-primary colour, bold, with verified badge icon.
- `box-shadow: 0 1px 1px rgba(0,0,0,0.1)` on all bubbles.
- Message animation: `opacity 0 → 1`, `translateY(6px → 0)`, 0.18s ease-out.
- Max bubble width: leave `padding-right: 48px` (in-wrap) / `padding-left: 48px` (out-wrap) to match WA proportions.

## CTA buttons

- **Rendered as a SEPARATE block** (`wa-cta-block`) appended below the bubble, NOT inside it. This matches WA Business template layout exactly.
- White background (same as incoming bubble), `border-radius: 0 0 8px 8px`.
- Each button: `border-top: 1px solid #e0e0e0`, link-blue text `#027eb5`, centered, 13px bold.
- URL-type buttons get an external-link SVG icon; reply buttons get no icon.
- **Patron's voice only.** "Speak to Agent" ✓, "Pay Now" ✓, "View Order" ✓. "Open Console" ✗, "Switch View" ✗, "Open Inbox" ✗. Rule: would the customer tap this label without confusion? If no, move it to the orchestrator's main send button. Full posture in `guardrails.md`.

## Typing indicator

- Persistent DOM node, shown/hidden via `.show` class — not created/destroyed per step.
- Three bouncing dots (7px circles, `#b0bec5`) with staggered vertical animation.
- Housed inside a white bubble (incoming shape) with the tail.
- Positioned via `wa-typing-wrap.in` wrapper with matching padding.

## WhatsApp Flow overlay

- Full-pane scrim `rgba(0,0,0,0.55)`, bottom-sheet slides up with `cubic-bezier(0.32,0.72,0,1)`.
- Sheet has: drag handle pill at top, header (brand icon + title + close `×`), segmented progress bars (not dots — WA Flows uses bars), scrollable body, sticky footer with Back/Continue/Submit.
- Progress bars: active `#008069`, done `#4caf50`, inactive `#e0e0e0`.

## Input bar

- Decorative only (not functional). `background: #f0f2f5`. Emoji icon left, rounded pill with "Message" placeholder + attach icon inside, circular teal send FAB right.
- Makes the phone feel inhabited, not hollow.

## SMS OTP bubble

- Yellow-tinted `#fffde7`, left border `3px solid #f9a825`, `border-radius: 0 8px 8px 8px` (no tail).
- Header label "📱 SMS — separate channel · 8x8 Verify API" in amber bold.
- OTP body in monospace.
- This differentiation is part of the pitch — it signals multi-channel orchestration, not a WA message.
