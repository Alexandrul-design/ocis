# Kiteworks theme — change handoff

Purpose: bring the ownCloud/oCIS web UI in line with the Kiteworks design system
(colors from the `kiteworks-atoms` semantic token set). This documents every change
made in the working copy so it can be reviewed and re-implemented cleanly.

**Scope:** the light **"Light Theme"** only (dark / high-contrast variants untouched).
**Deploy:** all of these are baked into the `owncloud/ocis:dev` image — rebuild + restart to see them:
```
cd ~/ocis && make -C ocis dev-docker
cd ~/ocis/deployments/examples/ocis_full && docker compose down && OCIS_DOCKER_TAG=dev docker compose up -d
```

> ⚠️ **Two of these live in built artifacts (a JS bundle patch and CSS on served assets), not in source.**
> They survive an oCIS image rebuild but are **lost if the web frontend (`owncloud/web`) is rebuilt.**
> Items marked **[reimplement in source]** should be ported into `owncloud/web` proper.

---

## 1. Color tokens — `services/web/assets/themes/owncloud/theme.json`

Edit the `colorPalette` of the **"Light Theme"** block(s) under
`clients.web.themes[]`. Note: the file currently contains **two** blocks named
"Light Theme" (a pre-existing duplicate); both were edited identically. Consider
de-duplicating.

| token | before | after | role |
|---|---|---|---|
| `swatch-brand-default` | `#FFFFFF` | `#0F152A` | brand / ink navy |
| `swatch-brand-hover` | `#324595` | `#283878` | blue 700 |
| `swatch-brand-contrast` | `#585757` | `#FFFFFF` | text on brand |
| `swatch-primary-hover` | `#425CC7` | `#283878` | primary hover = blue 700 (darken-on-hover) |
| `swatch-primary-muted-hover` | `#425CC7` | `#283878` | |
| `swatch-primary-gradient-hover` | `#425CC7` | `#283878` | |
| `swatch-danger-default` | `#AC3B33` | `#BD413A` | red 500 |
| `swatch-danger-hover` | `#8E2F27` | `#A5413B` | red 700 |
| `swatch-danger-muted` | `#D69D99` | `#F8D9D7` | red 200 (light bg) |
| `swatch-success-default` | `#3C6D17` | `#649B3B` | green 500 |
| `swatch-success-hover` | `#2F5412` | `#3A6E14` | green 700 |
| `swatch-success-muted` | `#9DB68B` | `#EFF5EB` | green 100 (light bg) |
| `swatch-warning-default` | `rgb(183,134,26)` | `#C2880A` | yellow 700 (darkened; = hover, for contrast) |
| `swatch-warning-hover` | `#7A5912` | `#C2880A` | yellow 700 |
| `swatch-warning-muted` | `rgba(183,134,26,.5)` | `#FAEFD5` | yellow 100 (light bg) |
| `text-default` | `oklch(13% …)` | `#000000` | body text |
| `text-muted` | `#585D65` | `#585757` | grey 800 |
| `input-text-default` | `#111732` | `#000000` | |
| `input-text-muted` | `#585D65` | `#757575` | grey 700 (placeholder) |
| `input-border` | `#A3A7AB` | `#838382` | grey 600 |
| `border` | `#E6E7E8` | `#F0F0EF` | grey 200 |
| `color-components-apptopbar-border` | `#E6E7E8` | `#F0F0EF` | |
| `background-muted` | `#ffffff` | `#F5F5F4` | grey 100 |
| `background-accentuate` | `rgba(255,255,5,.1)` | `#FAEFD5` | yellow tint (no alpha trick) |
| `background-sidebar` | `#283878` | `#0F152A` | **left nav = inverse navy** |
| `swatch-passive-muted` | `#E1E0DE` | `#F0F0EF` | |
| `icon-document` | `#3b44a6` | `#425CC7` | blue 500 |
| `icon-archive` | `#fbbe54` | `#C2880A` | yellow 700 |
| `icon-pdf` | `#ec0d47` | `#BD413A` | red 500 |
| `icon-spreadsheet` | `#15c286` | `#649B3B` | green 500 |

Convention applied to status ramps: `default = 500`, `hover = 700` (darker),
`muted = 100/200` (light background tint), `contrast = white` (warning contrast stays black).
**Exception:** **warning** `default` was darkened to the 700 value (`#C2880A`, = its hover) so
its text passes contrast — so warning has no separate 500 and no darken-on-hover. **Danger** and
**success** follow the 500→700 pattern (success `#649B3B` → `#3A6E14`).

Icons without a close KW hue were left as-is (`icon-image`, `icon-video`,
`icon-audio`, `icon-presentation`, `icon-medical`).

---

## 2. CSS overrides — `services/web/assets/themes/owncloud/assets/kiteworks-overrides.css`

Loaded via web config (`web.config.styles`) in
`deployments/examples/ocis_full/config/ocis/web.yaml`:
```yaml
web:
  config:
    styles:
      - href: "themes/owncloud/assets/kiteworks-overrides.css"
```
These handle things the color tokens can't express. **[reimplement in source]** where a
proper component/theme mechanism exists.

1. **Left nav: white labels + icons** on the dark sidebar (no "sidebar text" token exists;
   `text-default` is shared with the white body). Scoped to `#web-nav-sidebar`, light themes only.
2. **Selected nav item = blue 700 fill + white, bold content.** `.oc-sidebar-nav-item-link.active`
   gets `background-color:#283878` and its label is bolded (`font-weight:700`). (Active item renders
   as a transparent button, so it has no fill token to set.) Hover (non-active) already resolves to
   `swatch-primary-hover` (#283878).
3. **Right side panel stays white.** `.sidebar-panel` shares `--oc-color-background-sidebar`
   with the left nav (now dark); force it back to `background-default`.
4. **Main content flush + square corners.** Remove shell frame padding on left/right/bottom
   (`body:has(#web-content)`, top gap kept); `border-radius:0` on `.app-container`, the app-bars
   (`#files-app-bar`, `#admin-settings-app-bar`), and `#web-nav-sidebar`.
5. **Secondary (outlined) button:** keep the **default** text/icon color on hover/focus
   (was flipping to white). Hover fill softened from blue to `background-hover` so the
   default-grey text stays legible. Selector: `.oc-button-passive-outline:hover/:focus`.
6. **Top-right collapse (toggle-sidebar) button:** white icon; **no hover/pressed background**
   (transparent — revised; was `#283878`). Selector: `.toggle-sidebar-button`.

7. **Left-nav bottom text (`.versions`) → light grey** `#838382` (grey 600, ~4.8:1 on the navy),
   readable on the dark sidebar. Light themes only.
8. **Tertiary / raw buttons keep DEFAULT text + icon colour on hover/focus** (instead of turning
   blue via `swatch-passive-hover` / `swatch-brand-hover`). Covers the file-selection batch-action
   bar, right-panel actions, list-item share / copy-link, the sort-column arrow. Selectors:
   `.oc-button-passive-raw`, `.oc-button-brand-raw`, `.oc-button-sort`.
9. **Corner radii normalised (sharp-corner policy).** Already-rounded controls → 4px
   (`.oc-button-group`, password field, search field); modals → 12px (`.oc-modal` + title +
   body-actions). Elements that were square (radius 0) are **left square on purpose** — no rounding
   added where there was none (`.oc-text-input`, `.oc-textarea` untouched; `.oc-button`,
   `.vs__dropdown-toggle` already 4px natively).
10. **Input fields = white fill.** Core tinted the password field & select control light-blue and
   the textarea grey; forced white via `background-color` (shorthand avoided so the dropdown arrow
   image survives). `.oc-text-input`, `.oc-input`, `.oc-textarea`, `.oc-search-input`, password
   wrapper, `.oc-select .vs__dropdown-toggle`. (Also whitens a disabled select — flagged.)
11. **Login page: white background** — `.oc-login` inline dark photo removed
   (`background-image:none`); the colored/dark Kiteworks logo reads on white.
12. **Action / context-menu (3-dot) items keep DEFAULT label colour on hover/focus** — core
   recolored the label to `swatch-brand-hover` (navy); `color:inherit` restores it (hover background
   kept). `.oc-menu-item-hover a:hover span`, tippy switch/button variants.
13. **Left-nav items on focus show ONLY the white focus outline** — no background change on
   focus-without-hover; mouse hover still gets the `#283878` fill; active item unaffected.
14. **Dropdown-menu highlight radius = 4px** across the 3-dot "more" menu, app-switcher rows, and
   v-select option rows (were 5px / 8px / inherited).
15. **App-switcher icon tiles (`.oc-application-icon`)** — corners 4px. Actual current behaviour:
   - **Rest** (non-current app): grey tile `#F0F0EF` + dark-grey glyph `#585757`.
   - **Hover / focus** (non-current app): tile turns light-blue `#ECEFFA`; the **glyph stays grey**
     (the rule's intended darker-blue glyph does not take effect in practice).
   - **Current / selected app:** tile is a **darker grey** `#E1E0DE` + dark-ink glyph `#0F152A`
     — set by item **19c**, which supersedes the original light-blue-for-current styling here.
16. **Selected item = light-grey chip + LEFT checkmark** in role/permission dropdowns and the app
   switcher (was periwinkle "primary gradient" with the check on the right, or no check at all).
   Explicit light-grey hex `#F0F0EF` (not a token) so the chip stays light grey on both the **dark**
   right-hand share panel and the **light** app-switcher menu (`background-muted` resolves dark in
   the dark panel). Selectors: `.files-recipient-role-drop-btn.selected` (reorders its trailing
   check to `order:-1`) and `.applications-list .oc-button-primary` / `.router-link-active`
   (injects a masked-SVG check via `::before`).
17. **Font → SF Pro.** `--oc-font-family` repointed to the SF Pro system stack
   (`-apple-system, BlinkMacSystemFont, "SF Pro Text", "SF Pro Display", …`), forced on the root
   elements (core shipped Inter as a webfont).
18. **Filter toggles keep WHITE text when selected** (resting + hover/focus) — inline filters
   (`Shares` / `Hidden Shares`) and filter chips (`Include disabled`, Share Type, Shared By).
19. **Dropdown refinements** (role/permission dropdown + app switcher): (a) check→content gap
   halved; (b) non-selected rows get an invisible leading spacer so their icon+text align with the
   selected row; (c) app-switcher selected tile = darker grey than the selection + dark (ink) glyph.
20. **Right side-panel "Actions" list keeps its resting colour on hover/focus** — id-scoped to
   `#oc-files-actions-sidebar` (reinforces item 8).
21. **Public-link permission dropdown (`LinkRoleDropdown`)** gets the same light-grey / left-check
   treatment — `.role-dropdown-list`, options `[id^="files-role-"]` (separate component from the
   recipient dropdown; recipient / edit-collaborator / Spaces-member dropdowns already covered by
   items 16 & 19).

---

## 3. Logo — `services/web/assets/themes/owncloud/assets/`

- New asset: **`kiteworks-white.svg`** — the white wordmark (blue `#425cc7` kite-mark accent).
  Derived from the existing white artwork but with the viewBox expanded to reproduce the
  padding of `kiteworks-coloured.png` (~12% top / 15% bottom / 2% sides) so it renders at the
  same size/position as the previous logo.
- `theme.json` repointed: `common.logo` and `clients.web.defaults.logo.topbar` → `kiteworks-white.svg`.
- **Left as colored PNG:** `logo.login` (sits on `loginBackground.jpg` — confirm that bg is dark
  before switching) and the `favicon`.

---

## 4. Nav icons: outline when unselected — **[reimplement in source]**

**Requirement:** left-nav items that are **not** selected show **outlined** icons; the selected
item shows the **filled** icon.

**Current working-copy state:** patched directly in the built bundle
`services/web/assets/core/js/index.html-D8uIZSRL.mjs` (and its `.gz`) — the sidebar nav-item
icon render was changed from:
```js
{ name: t.icon, "fill-type": t.fillType,          variation: "inherit" }
```
to:
```js
{ name: t.icon, "fill-type": t.active ? "fill" : "line", variation: "inherit" }
```

**Proper fix:** in `owncloud/web`, the sidebar nav-item component (renders
`.oc-sidebar-nav-item-link`) should bind the icon `fill-type` to the item's active state
instead of the static `fillType` prop — `fill-type = active ? 'fill' : 'line'`.

**Caveat:** assumes every nav glyph has a `-line` (outline) variant. Standard Remix icons do;
verify any **custom** glyph (e.g. the Spaces mark) has a line variant, else that item's icon
renders empty when unselected — special-case those to stay filled.

**Personal item fix (this round):** the **Personal** nav item uses the custom icon
`resource-type-folder`, which only shipped a `-fill` variant. When unselected the render
asks for `resource-type-folder-line`, which 404s, so `OcIcon` fell back to its default
`information` glyph — the item showed an **info circle** instead of an outlined folder.
Fixed by adding the missing asset (no bundle patch):
`services/web/assets/core/icons/resource-type-folder-line.svg` — an outlined folder
(FontAwesome 5 `far fa-folder`, same viewBox `0 0 520 550` as the fill variant so it sits
identically). Selected still shows the filled folder; unselected now shows the outline.

---

## 5. Open items (not yet implemented)

- **(c) Bottom text in left nav → light grey.** ✅ **Done** — shipped as override **item 7**
  (`.versions` → `#838382`).
- **(e) Tertiary/raw buttons: keep default text/icon color on hover.** ✅ **General rule done** —
  override **item 8** keeps default text/icon colour on all `oc-button-passive-raw` /
  `oc-button-brand-raw` (extended for the sidebar Actions in **item 20**). ⚠️ **Still open:** the
  intended carve-out where the **rename** button and list-item **share** / **copy-link** actions
  should *keep* the blue hover — item 8 currently greys them like the rest. Needs the specific
  selectors before a carve-out rule can be written.

---

## 6. Accessibility notes (WCAG contrast, computed)

Pairs that are **below AA (4.5:1) for normal text** — flag for design sign-off:

- White text on `swatch-success-default` `#649B3B` = **3.34:1** (AA-large only).
- `swatch-warning-default` is now `#C2880A` (yellow 700, = hover) with **black** text
  (`swatch-warning-contrast:#000000`) — comfortable contrast. (Was `#F4B223`, which failed at
  1.87:1 with white text; darkening to 700 + black text fixed it — hence warning's default==hover.)
- `default-border` `#F0F0EF` on white = 1.14:1 (decorative border, exempt) — fine.
- Focus ring inner white core is paired with a `swatch-passive-default` halo (`#585757`, 7.2:1) — OK.

Passing comfortably: body text (21:1), white on primary `#425CC7` (5.85:1), white on
primary-hover / sidebar / selected-nav `#283878`/`#0F152A` (10.9 / 18:1).

---

## 7. Change chronology (which round did what)

The complete override list is §2 (items 1–21); full inline docs are in the CSS file. Rough order:

- **Initial rounds:** items 1–15 (nav colours, sharp corners/radii, input fills, login page,
  hover/focus exceptions, app-switcher tiles) + colour tokens (§1), logo (§3), nav outline
  icons (§4).
- **Later rounds (this handoff's most recent work):**
  - Selected-item light-grey chip + left checkmark — role/permission dropdowns & app switcher (item 16).
  - Font → SF Pro (item 17).
  - Filter toggles keep white text when selected (item 18).
  - Dropdown spacing/alignment + darker selected app tile (item 19).
  - Right-panel Actions keep resting colour on hover (item 20).
  - Public-link permission dropdown brought in line (item 21).
  - **Revisions:** selected left-nav label bolded (item 2); collapse-button hover background
    removed (item 6).
  - **New asset:** `core/icons/resource-type-folder-line.svg` (outlined folder) so the **Personal**
    nav item shows an outline when unselected instead of the `information` fallback glyph (see §4).
