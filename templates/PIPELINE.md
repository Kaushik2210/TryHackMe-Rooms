# Room Processing Pipeline

Reference for the single-room invocation prompt. Full spec lives in the project's master prompt
(kept by the user); this is the condensed version for repeated runs.

## Inputs (read in order)
1. `_data/completed-rooms.md` — authoritative room list. Do not process a room not listed here.
2. `_incoming/<room-slug>/notes.md` — raw session notes.
3. `_incoming/<room-slug>/*.png|jpg` — raw screenshots.
4. `_data/profile.md` — author context for tone/depth.

## Phases
1. **Reconnaissance** — read notes/screenshots, map what the room teaches vs. what evidence exists. Mark gaps, don't invent them.
2. **Concept research** — explain the underlying mechanism for each technique (why the vuln exists, what the defensive control is). Cite docs/CVEs/advisories.
3. **Asset processing** — rename screenshots `NN-short-description.png`, write caption + alt text, redact usernames/non-RFC1918 IPs/tokens/flags, log every redaction in `assets/manifest.json`. Missing screenshot + notes with terminal output → render via `templates/terminal-card.html`, labeled as a rendered transcript.
4. **Theming** — derive `theme.json` (palette + typography) from the room's own visual identity. WCAG AA contrast on light and dark GitHub themes.
5. **Authoring** — write `rooms/<difficulty>/<slug>/README.md` against `templates/article.md`.
6. **Publish** — branch `room/<slug>`, regenerate root `README.md` index + `_data/manifest.json`, open PR titled `Add write-up: <Room Name> (<difficulty>)`. No merge/force-push without asking. Never commit `_incoming/`.

## Hard constraints
- Never fabricate evidence or results — ask, or write `> **Evidence gap:** no capture available for this step`.
- Redact flags (`THM{[redacted]}`) and any identifying credential/hostname/token by default.
- No verbatim THM room/task text — paraphrase and attribute.
- If the room is one where write-ups are discouraged, publish concept + defensive sections only, skip the answer key, and flag before writing.
- Ask before any destructive git operation.

## Quality gate (before opening a PR)
- [ ] Every screenshot has caption, alt text, manifest entry
- [ ] Every redaction logged
- [ ] Every command's flags explained on first appearance
- [ ] Every finding has remediation + detection opportunity
- [ ] Every MITRE technique ID resolves to a real technique
- [ ] Every external link resolves
- [ ] No flag values/credentials/personal identifiers in the diff
- [ ] Theme colours pass AA contrast, light and dark
- [ ] Markdown renders correctly on GitHub
- [ ] Root README index includes the new room
