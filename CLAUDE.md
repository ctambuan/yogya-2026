# yogya-2026 Project Notes

## Architecture
Single `index.html` — no build step. All logic is vanilla JS inside one `<script>` block.
The app renders by setting `document.getElementById('root').innerHTML = html`.

## Before Marking Any Task Done — Checklist

1. **Validate JS syntax**: extract the `<script>` block and run `node --check`
   ```bash
   python3 -c "
   with open('index.html') as f: c = f.read()
   s = c[c.find('<script>')+8:c.find('</script>')]
   open('/tmp/t.js','w').write(s)
   " && node --check /tmp/t.js && echo OK
   ```
2. **Check the live site** after every push: fetch via API (bypasses CDN cache) and re-run the syntax check.
3. **Never leave `#root` blank** — if the page header shows but content is empty, it's a JS crash in `render()`.

## Known Pitfalls in This Codebase

### onclick quote escaping
When building HTML strings with inline `onclick`, always escape single quotes with `\'`:
```js
// WRONG — adjacent string literals = syntax error
html += '<button onclick="openCat(''+key+'')">

// RIGHT
html += '<button onclick="openCat(\''+key+'\')">'
```

### Mobile keyboard closes on every keystroke
`render()` wipes `#root` innerHTML, destroying the focused input and dismissing the keyboard.
**Rule**: never call `render()` from input handlers (`oninput`, `onfocus`).
Instead, update only the relevant DOM node in-place (see `renderAC()` pattern).

### XSS
All user-controlled values (`S.name`, `S.notes`, `S.search`) must go through `esc()` before innerHTML insertion.
`esc()` is defined at the top of the script block.

## Git Push
`git push` via the local proxy (`127.0.0.1:...`) returns **403** — the session proxy has read-only access.

**Workaround**: push via GitHub API using a fine-grained PAT with Contents: Write on this repo.
```python
import json, urllib.request, base64

PAT = "..."  # fine-grained PAT, revoke immediately after use

# 1. Get current main SHA + tree
# 2. Create blob from file content (base64 encoded)
# 3. Create tree (base_tree + new blob)
# 4. Create commit (tree + parent SHA)
# 5. PATCH refs/heads/main (or branch) to new commit SHA
```
See session history for the full reusable snippet.
**Always revoke the PAT immediately after use.**

## Participants (20 people)
Monang Panjaitan IR, Rohana Tambunan S.PD, Lusiana, Nhaomy Roma Lestary Panjaitan,
Rany Yamemia, Leandro Petrus Ratu, Arlo Hernand Bumi Ratu, Alora Malea Langit Ratu,
Gerard Sahat Pardomuan, Diana Pardede, Christine Tambunan, Agustianto Batubara,
Alexander Peter Batubara, Adolf M.K. Tambunan, Intan A.K., Agustinus Tambunan,
Linda Napitupulu, Ferdiana Sondang Veranica, Ronald Daniel, Ivana Elliora Veda

## Deployment
- Live at: **https://ctambuan.github.io/yogya-2026**
- GitHub Pages serves from `main` branch root
- After pushing to `main`, Pages redeploys automatically (~1 min)
- CDN cache on `raw.githubusercontent.com` can lag — verify via API instead
