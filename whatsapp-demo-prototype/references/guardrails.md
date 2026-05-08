# Style, Posture & Guardrails

The demo is a **sales asset**, not engineering documentation. Every choice should reinforce that.

## Posture rules

- **Show channel mechanics, not implementation.** Audiences care that "this is a Utility template that fires when the credit decisioning engine returns an approved status" — they don't care about the API endpoint. Lead with business events.

- **Use real-looking data.** Believable reference IDs (`CC-2026-04158`, `PL-2026-00742`), last-4-only card numbers, local currency formatting, locale-appropriate dates. Lazy placeholders (`[Customer Name]`, `XXX-XXX`) kill the spell.

- **Patron's voice rule.** Every CTA inside a WhatsApp template reads like something the customer would tap — "Speak to Agent", "Pay Now", "RSVP", "View Order". Operator-side language ("Open Console", "Switch to Inbox", "Open Host View") inside a template bubble breaks the spell — patrons never see those words on their phones. Operator affordances belong on the orchestrator's main send button or the left control panel, not the phone preview.

- **Keep WhatsApp chrome pixel-authentic.** Full spec in `whatsapp-chrome.md`. The starter template implements everything; do not regress when re-skinning.

- **Keep 8x8 Converse chrome authentic too.** If the demo includes an embedded Converse stage (formerly Moobidesk), the visual ground truth is the PNG asset under `/mnt/skills/.../assets/` (typically `moobidesk-inbox-2026-04.png` — filename retained from pre-rebrand) — **view this PNG before writing any Converse markup**. The screenshot encodes pixel-level layout, spacing, icon choices, and font weights that no prose description can fully capture. `converse-inbox.md` covers the semantics (when to use which colour, which interactions matter for the pitch); the PNG tells you how it actually looks. You need both.

- **When the PNG and the markdown disagree, the PNG wins.** The product UI evolves faster than the prose docs. As of the latest skill update, `converse-inbox.md` is in sync with the post-rebrand PNG (orange "8x8" lozenge + black "Converse" wordmark, dark navy top bar, dark green outbound bubbles, blue active-card left accent bar instead of charcoal inversion, black send button). When the next UI shift happens, expect the PNG to update before the prose. If the asset and the doc diverge, match the PNG, build the demo, and leave a note in the response so the skill owner can refresh the prose.

- **Don't overdesign the left panel.** It's a control surface, not a brochure. Step cards with status pills, category tags, trigger labels. Resist adding charts or decorative elements that compete with the chat preview.

- **Don't volunteer regulatory claims unless the customer is in scope.** The reference docs say things like "MAS guidance bans SMS OTP for banking" or "for SG banking demos, always use Singpass." That's true *for banks*. It is not true for SPH Media, an insurance broker, a retail loyalty programme, or anyone else who isn't an MAS-regulated financial institution. When tempted to weave a regulatory rationale into the demo ("we picked Singpass because MAS bans SMS OTP for…"), pause and confirm the customer is actually in-scope for that regulation. If they're not, drop the framing — it's irrelevant at best, and at worst it'll trip up a sharp prospect who knows their own regulatory perimeter better than you do. Same pattern applies to PDPA, GDPR, MCMC, OJK, and other locale-specific regulators.

## What NOT to do

- Don't use external CSS frameworks, build tools, or npm packages. Single file, browser-openable, period.
- Don't render CTA buttons inside the message bubble div. They must be a separate sibling block — that's the correct WA Business template structure.
- Don't use Unicode ticks (`✓`) as plain text styled with `color`. Use the glyph inside a `.tick` / `.tick.read` span so grey/blue state can be toggled.
- Don't skip the phone shell. A raw chat pane on a white background looks like a web app, not WhatsApp. The shell is non-negotiable for 1-to-1 mode.
- Don't use a generic grid or diamond tile for the wallpaper. Use the SVG leaf/petal pattern from the starter template.
- Don't fake the WhatsApp green styling for SMS OTPs. They get the yellow `#fffde7` treatment.
- Don't invent regulatory claims. SG banking → Singpass because MAS banned SMS OTP for banking; that's the only regulator-specific carve-out the skill ships with.
- Don't write CSS or JS in separate files. The demo must travel as one file.
- Don't half-recreate the Converse console. If you embed it, go faithful.
- Don't put operator-side controls inside WhatsApp template CTAs. "Open Inbox", "Switch View", "Open [Tool] Console" — these are orchestrator-side actions and belong on the main send button or left control panel, not in the phone preview. Patrons never see those labels on their actual phones, so they shouldn't see them in your demo either.
