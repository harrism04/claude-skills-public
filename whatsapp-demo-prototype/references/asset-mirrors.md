# Asset Mirrors & Fallback Ladder

Files referenced throughout the skill (`assets/...`, `references/...`) ship locally alongside SKILL.md. Local files are canonical; mirrors are insurance against missing or corrupted local assets.

## Mirror URLs

| Local path | Mirror URL |
|---|---|
| `assets/starter-template.html` | `https://raw.githubusercontent.com/harrism04/claude-skills-public/main/whatsapp-demo-prototype/assets/starter-template.html` |
| `assets/companion-page-template.html` | `https://raw.githubusercontent.com/harrism04/claude-skills-public/main/whatsapp-demo-prototype/assets/companion-page-template.html` |
| `assets/moobidesk-inbox-2026-04.png` | `https://raw.githubusercontent.com/harrism04/claude-skills-public/main/whatsapp-demo-prototype/assets/moobidesk-inbox-2026-04.png` |
| `references/architecture.md` | `https://raw.githubusercontent.com/harrism04/claude-skills-public/main/whatsapp-demo-prototype/references/architecture.md` |
| `references/broadcast.md` | `https://raw.githubusercontent.com/harrism04/claude-skills-public/main/whatsapp-demo-prototype/references/broadcast.md` |
| `references/integrations.md` | `https://raw.githubusercontent.com/harrism04/claude-skills-public/main/whatsapp-demo-prototype/references/integrations.md` |
| `references/converse-inbox.md` | `https://raw.githubusercontent.com/harrism04/claude-skills-public/main/whatsapp-demo-prototype/references/converse-inbox.md` |
| `references/verticals.md` | `https://raw.githubusercontent.com/harrism04/claude-skills-public/main/whatsapp-demo-prototype/references/verticals.md` |
| `references/qa-checks.md` | `https://raw.githubusercontent.com/harrism04/claude-skills-public/main/whatsapp-demo-prototype/references/qa-checks.md` |
| `references/whatsapp-chrome.md` | `https://raw.githubusercontent.com/harrism04/claude-skills-public/main/whatsapp-demo-prototype/references/whatsapp-chrome.md` |
| `references/guardrails.md` | `https://raw.githubusercontent.com/harrism04/claude-skills-public/main/whatsapp-demo-prototype/references/guardrails.md` |
| `references/asset-mirrors.md` | `https://raw.githubusercontent.com/harrism04/claude-skills-public/main/whatsapp-demo-prototype/references/asset-mirrors.md` |

PNG is binary — if local `view` fails, `view` against the raw URL works the same way (URL serves `Content-Type: image/png`). Markdown and HTML files can be read via `web_fetch`.

## Fallback ladder when local assets are missing

The bash environment in some 8x8 projects has a strict egress allowlist that blocks `raw.githubusercontent.com`. A `curl` against the mirror returns `Host not in allowlist` (a 21-byte stub), not a 404. **Check for this — a small file is suspicious before assuming the file is small.** If `curl -sL <url> | wc -c` returns under 200 bytes for what should be a multi-KB file, egress is blocked.

If blocked, walk the ladder:

1. **`web_fetch` from the conversational tool path.** Different network egress than `bash_tool`; usually reaches GitHub. Works fine for `.md` references. Less ideal for HTML — `web_fetch` returns markdown-extracted text, which corrupts the starter template's HTML/CSS/JS structure. **Use `web_fetch` only for the markdown references, never for `starter-template.html`.**
2. **Project-bundled assets.** Ask the user to drop the seven files into the project's file mount (typically `/mnt/project/`). They become readable via `view` with no network at all. This is the most reliable path; recommend it as the primary fix when the user is running multiple demo builds.
3. **Inline reconstruction.** Last resort. The skill prompt + user's pasted-in content contains enough information to reconstruct the structure — but slow, error-prone, almost guaranteed to regress WhatsApp chrome (wallpaper SVG, CSS triangle tails, message animations). Avoid unless the user explicitly accepts the trade-off.

Mirror is maintained by the skill owner. Treat local files as canonical; only fall back when local access fails.

## Mirror sync history

- **2026-05 refactor + Converse rebrand**: pushed slimmed `SKILL.md`, renamed `moobidesk-inbox.md` → `converse-inbox.md`, added 4 new references (`qa-checks.md`, `whatsapp-chrome.md`, `guardrails.md`, `asset-mirrors.md`), patched `starter-template.html` comment, updated cross-references in 4 patched references.
