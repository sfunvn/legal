# Instructions for AI agents: adding an app's legal pages

This repo hosts the Support page, Privacy Policy and Terms of Service for Silver AI's
iOS apps. It is plain static HTML + CSS: no build step, no JavaScript, no framework.
GitHub Pages publishes the `main` branch root to **https://sfunvn.github.io/legal/**.

Your usual task: **create the three pages for a new app, following the existing
structure exactly.** Use `wardra/` as the reference. It is the newest and most complete
app (it has its own icon, favicon and web font).

## How this repo is used

You are most likely running inside **a developer's app repo, not this one**. The
developer has asked you to read these instructions and publish legal pages for their
app. There are two repos involved:

| Repo | Role |
| --- | --- |
| **App repo** (your current working directory) | Source of truth. Scan it for the facts in step 1. **Read only**: don't modify it for this task. |
| **Legal repo** (`sfunvn/legal`, this one) | Where you create the files, commit and push. |

### Step 0: Get a fresh copy of the legal repo

Work in a separate clone. Never nest it inside the app repo.

```sh
LEGAL=~/.cache/silverai-legal        # or any path outside the app repo
if [ -d "$LEGAL/.git" ]; then
  git -C "$LEGAL" checkout main && git -C "$LEGAL" pull --ff-only
else
  git clone https://github.com/sfunvn/legal.git "$LEGAL"
fi
```

Before you create anything, check that the app doesn't already have a folder
(`ls "$LEGAL"`). If it does, you are **updating** the existing pages (see the last
section), not creating new ones. All paths below are relative to `$LEGAL`.

---

## 1. Collect the facts from the app repo. Never invent them.

A privacy policy is a legal statement about what the app actually does. **Scan the
app repo first**, then ask the developer only about what the code can't tell you, such
as backend retention or the effective date. If a claim can't be confirmed, **ask. Do
not guess or copy another app's claims.**

Places to look in an iOS app repo:

- `Info.plist` / build settings: `CFBundleDisplayName` (app name), the
  `NS*UsageDescription` keys (permissions and why)
- `PrivacyInfo.xcprivacy`: declared collected data types, tracking, required-reason APIs
- Networking code (`URLSession`, API clients, base URLs): every request that leaves the
  device, its payload, and which identifiers go with it
- Package / Pod dependencies: Firebase, RevenueCat, ad SDKs, analytics SDKs
- Analytics event calls: which parameters are logged
- StoreKit / paywall code and product IDs: free limits, what Pro unlocks
- `docs/DESIGN.md`, `tokens.json`, or colour assets: brand colours and font
- `Assets.xcassets/AppIcon.appiconset`: the 1024 px icon

Before writing any pages, **show the developer a short summary of the data flows you
found** and get their confirmation.

| Fact | Used in |
| --- | --- |
| App display name (e.g. `Wardra`) and folder slug (lowercase, no spaces, e.g. `wardra`) | everywhere |
| One-sentence description of what the app does | Support lead |
| **Everything that leaves the device**: which data, to which service, whether it's stored, what identifiers go with it | Privacy §“What is sent” |
| What stays on the device only | Privacy §“What stays local” |
| Accounts / sign-in? Sync / cloud backup? | Privacy, Terms, FAQ |
| Analytics / crash SDKs (e.g. Firebase) and what events carry | Privacy §Analytics |
| Ads, tracking (ATT), advertising identifier | Privacy |
| iOS permissions requested (camera, photos, mic, location, notifications, …) and why each one | Privacy §Permissions, FAQ |
| Monetization: free tier limits, subscription name, what Pro unlocks | Terms, FAQ |
| Features that need a disclaimer (AI output, health/body data, third-party links, …) | Terms |
| Brand accent colour(s), font, icon | CSS + assets |
| Effective date (default: today, written as `25 August 2026`) | Privacy + Terms pill |

---

## 2. Files to create

```
<slug>/
  index.html     Support page
  privacy.html   Privacy Policy
  terms.html     Terms of Service
  icon.png       App icon (square, used as brand mark + apple-touch-icon)
  favicon.png    Small favicon
assets/<slug>.css   App stylesheet
```

Start by copying the reference, then rewrite every piece of app-specific text:

```sh
cp -r wardra <slug>
cp assets/wardra.css assets/<slug>.css
```

Then remove every leftover Wardra reference:

```sh
grep -rn -i "wardra\|Thêm từ link\|Archivo" <slug>/ assets/<slug>.css
```

That command must print nothing, unless you deliberately kept the Archivo font.

Make the icons from the app repo's 1024 px AppIcon (on macOS, `sips` is built in):

```sh
sips -Z 180 /path/to/AppIcon-1024.png --out <slug>/icon.png
sips -Z 64  /path/to/AppIcon-1024.png --out <slug>/favicon.png
```

If the app has no icon, follow `workcam/` or `rolliecam/` instead. They use a text glyph
mark (`<span class="mark">◎</span>`) and no favicon links.

---

## 3. Page skeletons (keep this markup and these class names)

The CSS depends on these class names. Don't rename them, and don't add inline styles
beyond the ones already in the reference.

### Shared `<head>`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{App} — Support</title>            <!-- or “Privacy Policy” / “Terms of Service” -->
  <meta name="description" content="…">    <!-- page-specific, one sentence -->
  <link rel="icon" type="image/png" href="favicon.png">
  <link rel="apple-touch-icon" href="icon.png">
  <!-- optional Google Font: preconnect ×2 + stylesheet, as in wardra/ -->
  <link rel="stylesheet" href="../assets/{slug}.css">
</head>
```

Use relative paths only (`../assets/…`, `privacy.html`). The site is served from the
`/legal/` sub-path, so a root-absolute `/assets/…` link will break.

### Masthead (all three pages)

```html
<div class="wrap">
  <header class="masthead">
    <div class="brand"><img class="mark" src="icon.png" width="26" height="26" alt=""> Silver AI · {App}</div>
    <h1>{Page title}</h1>
    <!-- privacy/terms only: -->
    <span class="pill"><span class="dot"></span> Effective {D Month YYYY}</span>
    <p class="lead">…</p>
    <!-- privacy/terms only: -->
    <nav class="toc"><a href="#id">Label</a> …</nav>
  </header>
```

### `index.html`: Support

1. Lead: `<strong>We're here to help.</strong>`, then one sentence on what the app is,
   then an invitation to write in.
2. `.btn` link to `mailto:support@silverai.com?subject={App}%20Support`. Keep the
   envelope SVG.
3. `<main class="card faq">` with one `<section><h2>Question?</h2><p>Answer</p></section>`
   per FAQ. Always include:
   - How do I send feedback or report a problem? (`?subject={App}%20Feedback`)
   - Where is my data stored? / Is anything sent anywhere?
   - Why does the app ask for {permission}?
   - If the app has subscriptions: What does {App} Pro give me? **and** How do I
     manage or cancel my subscription? (Apple bills; iOS Settings → your name →
     Subscriptions; Restore purchases)
   - App-specific troubleshooting and disclaimers
4. `<h2 style="margin:8px 0 4px">Legal</h2>` + `<nav class="docs">` with the two
   `.doc` cards (🔒 Privacy Policy, 📄 Terms of Service). Copy them verbatim.
5. Footer: `© {year} Silver AI · support@silverai.com`.

### `privacy.html`: Privacy Policy

The lead states **in one bold sentence what leaves the device**, then what stays local.
Number the sections with `<h2><span class="num">N</span> Title</h2>` and give each
`<section>` an `id` that the `.toc` links to. Standard order (add, drop or merge
sections to match the app):

1. Who we are: `{App} (“the app”) is developed by Silver AI (“we”, “us”)`
2. What is sent off your device: each payload, where it goes, whether it's kept, which identifiers are **not** sent
3. *(app-specific flows, e.g. in-app browser, sharing)*
4. What stays on your device
5. No accounts, no profile of you: no ads, no cross-app tracking, no data brokers
6. Analytics and crash reports: name the SDK, list what is and isn't in events
7. Purchases: Apple handles payment; we never see payment details
8. Device permissions we request: one bullet per permission, plus what happens if the user refuses; list what is *not* requested
9. Your control over your data
10. Children: general audiences, not directed at under-13s
11. Changes to this policy
12. Contact

Footer: `← Support · Terms of Service · © {year} Silver AI`

### `terms.html`: Terms of Service

Lead: `These Terms govern your use of the {App} app. By downloading or using the app,
you agree to them. <strong>If you do not agree, do not use the app.</strong>`

Standard order:

1. License: personal, non-exclusive, non-transferable, revocable; subject to Apple App Store Terms
2. *(app-specific disclaimers: not medical advice, AI output can be wrong, third-party sites, …)*
3. Your content: user keeps rights; stored locally; link to `privacy.html`
4. Subscription (if any): free limit, what Pro unlocks, Apple auto-renew wording (charged to Apple ID at confirmation, renews unless cancelled ≥ 24 h before period end, manage in Settings, refunds via Apple)
5. No cloud backup: keep your own copies (if applicable)
6. Acceptable use
7. Disclaimer of warranties: “as is” and “as available”
8. Limitation of liability
9. Changes
10. Contact

Footer: `← Support · Privacy Policy · © {year} Silver AI`

---

## 4. Stylesheet: same layout, the app's look

Every app shares **one layout** (the markup and class names in section 3, first defined
in `assets/legal.css`). Each app gets **its own stylesheet** that re-skins that layout
so the web pages look like the app. The look comes from the **app's own design
system**, not from the other apps here:

| Sheet | Design direction |
| --- | --- |
| `legal.css` (Task Cam) | Neutral default: system font, plain cards |
| `rolliecam.css` | Analog film: warm film-stock palette, sprocket decorations on the brand mark, monospace labels |
| `moodraw.css` | “Sunlit Ceramic”: warm cream surfaces, moss accent, colour kept for the gradient header |
| `wardra.css` | “Modernist”: red accent, content areas darker than the page, no shadows or gradients, two corner radii only, Archivo font |

How to build `assets/<slug>.css`:

1. **Read the app repo's design source**, in this order: `docs/DESIGN.md` + `tokens.json`,
   then colour assets in `Assets.xcassets`, then a theme/colour file in code
   (e.g. `Theme.swift`, `Color+Brand.swift`). Note the palette (light **and** dark),
   the font, the corner radii, and any strong rules (e.g. “no shadows”).
2. **Start from the closest existing sheet**, or `legal.css` if none is close. Keep
   every selector; the pages depend on them.
3. **Set the `:root` tokens** (`--accent`, `--accent-deep`, `--accent-soft`, `--bg`,
   `--surface`, `--card`, `--text`, `--text-2`, `--text-3`, `--border`, `--radius`) and
   the matching `@media (prefers-color-scheme: dark)` block from the app's palette.
   Pages must support both light and dark mode.
4. **Font:** use the system font stack unless the app has its own font. If it does,
   load it from Google Fonts (preconnect + stylesheet `<link>`s on all three pages, as in
   `wardra/`) and put it first in the `body` font stack. No other external resources.
5. **Optional motifs:** small touches taken from the app are welcome (like RollieCam's
   sprockets or Moodraw's gradient). Keep them in CSS, no images or JS, and keep the
   text readable. Don't copy another app's motif.
6. **Accessibility:** text colours must pass WCAG AA (4.5:1) in both modes. If the brand
   accent fails at text size, add a darker `--accent-deep` for text, as `wardra.css` does.
7. **Header comment:** name the app, the design direction, and any rules a later
   editor must not “fix”, as `wardra.css` does.

If the app repo has no design system, ask the developer for the brand colour. If they
have no preference, use `legal.css` unchanged: link to it and don't create a new sheet.

---

## 5. Writing style

- Plain, specific and honest. Say exactly what is sent and what isn't. Avoid vague
  words like “may collect certain information”.
- Use `<strong>` for the key promise in each paragraph, and don't overuse it.
- Use curly quotes (“ ” ’) and em dashes (—), and write dates as `D Month YYYY`.
- Contact is always **support@silverai.com**. Encode spaces in `mailto:` subjects
  as `%20`.
- Keep app-UI labels in the app's own language when quoting them (e.g. Wardra's
  Vietnamese “Thêm từ link”).
- The company is **Silver AI**.

---

## 6. Verify before finishing

```sh
python3 -m http.server 8000     # open http://localhost:8000/<slug>/
```

- [ ] All three pages render in light **and** dark mode, and at phone width
- [ ] Every `.toc` link jumps to a matching `section id`
- [ ] Footer and `.docs` links work between the three pages
- [ ] Icon and favicon load (no 404s in devtools)
- [ ] Same effective date on privacy and terms
- [ ] `grep` for the reference app's name returns nothing
- [ ] Every data-flow claim was confirmed by the user or the code

## 7. Commit, push and publish

Committing and pushing to `main` is part of this task, and the developer has asked for
it by pointing you here. Other developers push to this repo too, so:

1. Add your app to **both** tables in `README.md` (**Apps** and **Publishing**).
2. Stage **only your own files**: `<slug>/`, `assets/<slug>.css`, `README.md`. Never
   edit or stage another app's folder or stylesheet, and never `git add -A`.
3. Commit using the message format from the history:

   ```sh
   git add <slug>/ assets/<slug>.css README.md   # omit the .css if you reused legal.css
   git commit -m "Add {App} support page, privacy policy, and terms of service"
   ```

4. Sync and push. Never force-push.

   ```sh
   git pull --rebase origin main
   git push origin main
   ```

   If the rebase conflicts, it will almost always be in `README.md`: keep both sides'
   rows. For any other conflict, or if the push is rejected for permissions, stop and
   tell the developer.
5. GitHub Pages rebuilds in about a minute. Check that the pages are live:

   ```sh
   curl -s -o /dev/null -w "%{http_code}\n" https://sfunvn.github.io/legal/<slug>/privacy.html   # expect 200
   ```

6. Report back to the developer with the URLs for App Store Connect:
   - Support URL: `https://sfunvn.github.io/legal/<slug>/`
   - Privacy Policy URL: `https://sfunvn.github.io/legal/<slug>/privacy.html`
   - Terms (EULA) URL: `https://sfunvn.github.io/legal/<slug>/terms.html`

## Updating an existing app's policy

Use the same flow: fresh clone (step 0), re-scan the app repo for what changed
(new SDK, new permission, new network call), and edit only that app's pages. Change the
**Effective** date on both `privacy.html` and `terms.html` if either one changes
materially, and update the date in the README table. Then commit
(`Update {App} privacy policy`, or similar) and push as in step 7.
