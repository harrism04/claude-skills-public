# QA Checks — Mandatory Before Delivery

Two layers. Mechanical first (catches code-level bugs that pass review but fail at runtime), then visual (catches code that parses but feels wrong).

## 1. Mechanical checks (run every time)

### Script syntax check

Extract every `<script>` block and run `node --check` on each. This catches the #1 silent killer: orphaned function tails left by an imperfect `str_replace`. The file still loads, chrome still renders, but every JS-driven element (steps list, profile card, button handlers) silently fails — you only notice the empty step list visually, after the user has opened the file.

```bash
python3 -c "
import re, sys
html = open('index.html').read()
scripts = re.findall(r'<script[^>]*>(.*?)</script>', html, re.DOTALL)
for i, s in enumerate(scripts):
    open(f'/tmp/s{i}.js','w').write(s)
print(f'Wrote {len(scripts)} script blocks')
" && for f in /tmp/s*.js; do node --check "$f" || { echo "FAIL: $f"; exit 1; }; done && echo "All script blocks OK"
```

If a check fails, find the orphaned code (usually a duplicated function tail or unmatched brace), delete it, re-run. **Do not ship a demo that fails this check.**

### Other one-liners (also mandatory)

```bash
# 1. No template placeholders left over (every {{...}} should be replaced)
grep -n '{{' index.html | head -20 || echo "OK: no placeholders"

# 2. Tag balance (mismatch usually means a bad str_replace cut off a closing tag)
echo "<div: $(grep -o '<div' index.html | wc -l), </div>: $(grep -o '</div>' index.html | wc -l)"
echo "<script: $(grep -o '<script' index.html | wc -l), </script>: $(grep -o '</script>' index.html | wc -l)"
echo "<style: $(grep -o '<style' index.html | wc -l), </style>: $(grep -o '</style>' index.html | wc -l)"

# 3. Critical functions still defined
grep -E '^function (sendNext|renderSteps|renderProfile|switchMarket|handleButtonClick)' index.html
```

### Playwright runtime assertion (if available)

Catches a third class of bug `node --check` misses: code that parses but throws at runtime (e.g. referencing an undefined variable from a half-completed rename).

```python
from playwright.sync_api import sync_playwright
errors = []
with sync_playwright() as p:
    page = p.chromium.launch().new_page()
    page.on('pageerror', lambda e: errors.append(str(e)))
    page.goto(f'file://{path}')
    page.wait_for_timeout(800)
    assert page.evaluate("document.querySelectorAll('.step-card').length") > 0, "step cards empty"
    assert page.evaluate("!!document.querySelector('.cp-name')"), "profile card missing"
    assert not errors, f"page errors: {errors}"
```

## 2. Common script-surgery failure modes

Each of these has bitten this skill in the past. When using `str_replace` on function bodies or step blocks, one of these is what happens when it goes wrong:

- **Orphaned function tail.** Replacement string omits trailing lines that were in the original — those lines now sit outside any function and break parsing. Fix: replace the *entire function body* (from `function name(` through the matching `}`), not a snippet inside it.
- **Duplicated step in `DEMO_DATA`.** A `str_replace` that adds a step but matches an existing one too loosely will duplicate it. Fix: include enough surrounding context (the `id:` line plus the preceding step's closing `},`) so the match is unambiguous.
- **Stale `currentMarket` after journey change.** Switching the market key in `DEMO_DATA` (e.g. `default` → `sg`) requires updating `let currentMarket = 'default'` too. Easy to miss; result is a blank profile card.
- **Smart-reply chip JSON escape.** Chip text inside an `onclick=` attribute must be JSON-stringified *and* HTML-escaped. The starter pattern uses `JSON.stringify(r).replace(/"/g, '&quot;')` — preserve it.
- **Send button transform overwritten by `renderSteps`.** Custom transforms (e.g. "Speak to Agent" → "Switch to Console") need a `setTimeout` longer than the typing-indicator delay, otherwise the post-step `renderSteps()` resets the button before the transform runs.

## 3. Visual walk-through (after mechanical checks pass)

Open the file mentally and walk through every step:

- Does each step's `trigger` label read like a real backend system event (not "user clicks button")? Triggers are part of the pitch — they communicate "automated, deterministic, eventable".
- Does the customer profile card at the top tell the persona's story in one glance?
- Is every CTA button wired? An unhandled button click is the most common bug.
- Multi-market: does switching tabs reset the chat and re-render the profile?
- Broadcast mode: do per-recipient delivery states animate convincingly?
- Embedded 8x8 Converse: is the active-conversation card highlight faithful (blue left-edge accent bar, NOT the legacy charcoal everything-inverts), do smart-reply chips populate the composer, does the back-button return to the workflow view cleanly?
