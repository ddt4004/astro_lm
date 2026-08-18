# LinksMaxing — Deploy & Update Guide

Quick reference for making changes to linksmaxing.com and getting them live.

**Project location:** `~/astro/linksmaxing/`
**Live URL:** [astro-lm.pages.dev](https://astro-lm.pages.dev) *(custom domain wiring pending)*
**GitHub repo:** [ddt4004/astro_lm](https://github.com/ddt4004/astro_lm)
**Hosting:** Cloudflare Pages (auto-deploys on every push to `main`)

---

## The daily loop

The standard flow for any change — copy edit, style tweak, new page, anything.

```bash
cd ~/astro/linksmaxing
npm run dev
```

Opens dev server at `http://localhost:4321/`. Every file save triggers instant browser refresh. Do all your work here first — nothing goes live until you push.

When done editing, kill the dev server with **Ctrl+C**, then:

```bash
git add -A
git commit -m "Short description of what changed"
git push
```

That's it. Cloudflare sees the push within seconds, rebuilds the site (~90 seconds), and the new version replaces the old at `astro-lm.pages.dev`. No manual deploy step, no dashboard clicking.

**Watch the deploy** at [Cloudflare Pages dashboard](https://dash.cloudflare.com/) → Compute → Workers & Pages → `astro-lm` → Deployments. New build shows up automatically with a spinner, turns green ✓ when live.

---

## Common tasks

### Edit copy on an existing page

Find the right file:

| Page | File |
|---|---|
| Homepage (manifesto) | `src/pages/index.astro` |
| Scan / visibility check | `src/pages/scan.astro` |
| Services | `src/pages/services.astro` |
| Method | `src/pages/method.astro` |
| Results index | `src/pages/results.astro` |
| Individual case study | `src/pages/results/case-study-01.astro` |
| Learn hub | `src/pages/learn.astro` |
| About | `src/pages/about.astro` |
| Pricing | `src/pages/pricing.astro` |
| Contact | `src/pages/contact.astro` |
| Harper founder page | `src/pages/team/harper.astro` |

Edit in your editor of choice, save, watch localhost update, push when ready.

### Change styling site-wide

`src/styles/global.css` — one file, ~990 lines, drives design system for every page. Design tokens (colors, fonts, spacing) live at the top in `:root`.

### Change nav or footer (affects every page)

- Nav: `src/components/Nav.astro`
- Footer: `src/components/Footer.astro`
- Layout shell (fonts, head, motion script): `src/layouts/Layout.astro`

Editing any of these updates every page in one go.

### Add a new page

Create `src/pages/YOUR-PAGE-NAME.astro` — it'll be live at `/your-page-name`. Copy the structure from any existing interior page (e.g., `about.astro`) as a starting template.

For a nested URL like `/blog/first-post`, create `src/pages/blog/first-post.astro`.

### Swap Harper's photo

Replace `public/harper.png` with a new file at the same path and filename. Push. Done.

### Add a new image

Drop it in `public/`. Reference in your page as `/filename.png` (no `public/` prefix in the URL path).

---

## First-time setup

If you or Harper are on a fresh machine and need to work on the project:

```bash
# 1. Clone the repo
git clone https://github.com/ddt4004/astro_lm.git ~/astro/linksmaxing
cd ~/astro/linksmaxing

# 2. Install dependencies
npm install

# 3. Set git identity (once per machine)
git config --global user.name "Your Name"
git config --global user.email "your-github-noreply@users.noreply.github.com"

# 4. Set credential helper so pushes don't repeatedly prompt (once per machine)
git config --global credential.helper osxkeychain

# 5. Start dev server
npm run dev
```

First push after setup will prompt for GitHub credentials. Use your GitHub username and a Personal Access Token (create at [github.com/settings/tokens?type=beta](https://github.com/settings/tokens?type=beta) with Contents: Read/write on the `astro_lm` repo).

**Finding your GitHub noreply email:** [github.com/settings/emails](https://github.com/settings/emails) — look for the `<numbers>+<username>@users.noreply.github.com` address listed under "Keep my email addresses private".

---

## Troubleshooting

### `npm run dev` errors out immediately

Check Node version:
```bash
node --version
```
Needs to be 18.17.1+ or 20.3.0+ or 22+. Fix with nvm if outdated.

If Node is fine, try:
```bash
rm -rf node_modules package-lock.json
npm install
```

### `git push` prompts for password and rejects it

GitHub blocks password auth. You need a Personal Access Token. Create one at [github.com/settings/tokens?type=beta](https://github.com/settings/tokens?type=beta) — grant Contents: Read/write on `astro_lm`, and paste that token when prompted for password.

Once it works, the token gets saved to macOS Keychain and won't prompt again (assuming `credential.helper osxkeychain` is set).

### Push succeeds but Cloudflare doesn't rebuild

Check that Cloudflare Pages is still connected to the repo:
- Dashboard → Compute → Workers & Pages → `astro-lm` → Settings → Build → Source
- Should show `ddt4004/astro_lm`, `main` branch

If the connection was broken, reconnect via Settings.

### Cloudflare build fails with a red X

Click **Details** on the failed deploy. The build log will show what broke. Most common:
- Syntax error in an `.astro` or `.css` file → fix, commit, push again
- Missing dependency → run `npm install` locally, commit `package-lock.json`, push
- Node version mismatch → set `NODE_VERSION` env var in Cloudflare project settings

### Site works locally but breaks on Cloudflare

Almost always a case-sensitivity issue. macOS filesystems are case-insensitive; Cloudflare's build server is case-sensitive. If you import `./Layout.astro` but the file is actually named `layout.astro`, local works and Cloudflare fails. Match filenames exactly.

### Favicon doesn't update after deploy

Browser cache. Hard-refresh with Cmd+Shift+R, or close and reopen the tab. If still stubborn, wait a few minutes — browsers cache favicons aggressively.

---

## The Python patch pattern

For style-system changes or systematic edits across multiple files, use a Python heredoc from the project root — safer than sed for multi-line replacements. Pattern:

```bash
cd ~/astro/linksmaxing
python3 << 'PYEOF'
with open('src/styles/global.css') as f:
    content = f.read()

old = '''<paste exact old text here>'''
new = '''<paste new text here>'''

if old in content:
    with open('src/styles/global.css', 'w') as f:
        f.write(content.replace(old, new))
    print('OK: updated')
else:
    print('FAIL: old text not found - was it already changed?')
PYEOF
```

Use this pattern when Claude gives you a "here's the change to apply" instruction that spans multiple lines or needs to preserve exact formatting.

---

## Key URLs bookmark

- **Live site:** https://astro-lm.pages.dev
- **GitHub repo:** https://github.com/ddt4004/astro_lm
- **Cloudflare dashboard:** https://dash.cloudflare.com/ → Compute → Workers & Pages → astro-lm
- **GitHub email settings:** https://github.com/settings/emails
- **Create a PAT:** https://github.com/settings/tokens?type=beta

---

## What's *not* automated

Domain wiring for `linksmaxing.com` is the outstanding infrastructure work. Currently the site is only at `astro-lm.pages.dev`. When ready to point the custom domain:

- **Path 1** — keep GoDaddy as registrar, update DNS records to point at Cloudflare Pages
- **Path 2** — transfer domain to Cloudflare Registrar (5-7 days, cleaner long-term)

Recommend doing this as its own focused session with about 45 minutes uninterrupted.

---

*Last updated: August 18, 2026*
