# Plan: Mallard block and accessibility fixes

Created: 2026-09-12

Found while comparing the Skeleton theme with Carmine and Harvest. Carmine is the reference for block templates unless an item says otherwise.

**Useful commands**

```bash
# Compare a Mallard file with Carmine (run from the Themes folder)
diff -w -B Mallard/src/<path> Carmine/src/<path>

# Show a Carmine commit referenced below
git -C Carmine show <hash>
```

## Theme-specific fixes

- [x] **`full-width.twig`** — `<main>` has no `id="main"`, so the skip link has no target on this layout. Add it. Every layout (one-column, narrow, full-width, both two-column layouts, calendar agenda/list) now has exactly one `#main`.
- [x] **Skip link CSS** — the skip link's `z-index: 100` equals the sticky header's default, so the focused link can be painted under the header. Use Harvest's approach: `z-index: calc(var(--Header-sticky-z-index, 100) + 1)` (Harvest `src/css/components/header/skip-to-main.css`). Applied to `.Header-skipToMain` in `css/components/header/header.css`.
- [x] **`blocks/image-grid.twig`** — `{% set width = 800 %}` and the `width:` values in `count2`–`count6` overwrite the block's Width field, so the Width setting never takes effect. Remove them (Carmine commit `1745710`). Mallard's existing `blockWidth(width)` call (without `true`) was kept as-is.
- [x] **`blocks/image-row.twig`** — `href="{image.url}"` uses single braces, so the link is broken. Change to `href="{{ image.url }}"`, and drop `target="_blank"` or add a visually hidden "(opens in a new window)". Kept `target="_blank"`, added `rel="noopener"` and the visually hidden text.
- [x] **Margin/width support** — add `macros.blockMargin(margin)`/`macros.blockWidth(width, true)` to grid-2…6-columns, heading, html-code, and columned-content (Carmine commits `265e272`, `81c6c37`, `660fa93`). Confirm the CMS block definitions have Margin and Width fields. Grids, heading, and html-code copied from Skeleton `df443e1` (includes the `macros` import on the grids). columned-content follows Carmine `660fa93` (`blockWidth(width)` without `true`) plus the missing `macros` import. The block definitions live in the CMS, not in this repo, so the Margin/Width fields still need checking there. `featured-content-wrapper.twig` only exists in the untracked `_export/` folder (old content-builder export), not in `src/templates/blocks`, so it wasn't changed.
- [x] **`blocks/google-ratings-bar.twig`** — each star is its own labelled image, so screen readers repeat "Star rating". The visible text already states the rating, so wrap the stars and number in `aria-hidden="true"` and use `iconAriaHidden` instead of `iconImg` (see Skeleton's `blocks/google-ratings-bar.twig`). Copied Skeleton's file (Mallard's differed only by these changes).
- [x] **Reviews link setting** — there are two "URL to view reviews" fields. Use the Settings one and remove the Styles one (Skeleton has this change):
    - [x] `blocks/google-ratings-bar.twig` — change `_core.theme.settings.googleRatingsBarReviewsLink` to `_core.theme.settings.customerRatingsBarReviewsLink` (3 places)
    - [x] `config/theme-styles.json` — in the "Blocks - Google Ratings Bar" group, remove the `googleRatingsBarReviewsLink` field and its "Review link" subgroup (the first one, which holds only that field). Keep the second "Review link" subgroup (the link typography).
    - Existing sites that set the link under Styles will need it re-entered under Settings → Customer Reviews & Ratings.
- [x] **`js/sticky-header.js`** — `hide()` moves the header offscreen even when keyboard focus is inside it (WCAG 2.4.11). Skip hiding when `header.contains(document.activeElement)`. Also added a `focusin` listener on the header that calls `show()`.

## Accessibility fixes shared by all themes

Carmine isn't a good reference for these — each needs a new fix. Items marked *(verify)* were found in Skeleton, Carmine, and Harvest but haven't been checked in Mallard yet.

- [x] **Accordion isn't keyboard-operable** — the heading is `<div class="Accordion-heading js-accordionHeading">` with only a click listener. Use a `<button>` with `aria-expanded` and `aria-controls` (`blocks/accordion.twig`, `js/accordion.js`). *High* Applied Skeleton `31a8df1` + `da5e0f9` (includes the `width: 100%` and `visibility: hidden` CSS in `accordion.css`).
- [x] **Modals** — the close button `<button class="Modal-close" data-micromodal-close></button>` has no accessible name. Also check for a missing `aria-labelledby`, `disableFocus: true` on the popup, and the notification icon's missing `aria-hidden` *(verify)* (`widgets/collections/popups.twig`, `notifications.twig`). All four confirmed in Mallard and fixed per Skeleton `62a1047` and `d0fa498` (`MicroModal.init()` in `main.js`). Mallard's `Modal-slide` class was kept.
- [x] **Pagination** — wrap in `<nav aria-label="Pagination">`, add `aria-current="page"` to the current page, and change the chevron icons from `role="img"` to `aria-hidden="true"` (`snippets/pagination.twig`).
- [x] **`iconImg` macro** outputs `<svg role="img" alt="…">` — `alt` isn't valid on `<svg>`. Use `aria-label` or a `<title>` (`macros/macros.twig`). Removed `alt`; the existing `<title>` + `aria-labelledby` name it.
- [x] **`rel="noopenner"` typo** — should be `noopener`. Fixed on the TopBar social links in `snippets/header.twig`; added `rel="noopener"` to the two footer credit links.
- [x] **Form errors** — the form error container has no `role="alert"`/`aria-live`. Also check that `form.js` sets `aria-invalid`/`aria-describedby` *(verify)* (`macros/form-macros.twig`, `js/form.js`). It didn't; applied Skeleton `7dfb4b6` and `faf6d57`.
- [x] **No `prefers-reduced-motion` CSS** — transitions, slider autoplay, and modal animations ignore it. Added Skeleton's block to `css/base/base.css`.
- [x] **Mobile submenus hidden from screen readers** — submenus render with `aria-hidden="true"` and a tap doesn't change it; `aria-expanded` sits on the `<ul>` instead of the toggle *(verify)* (`navigation/main.twig`, `js/navigation/small-screen.js`). *High* Confirmed; applied Skeleton `9898fed`.
- [x] **Mobile menu** — no Escape to close, no focus trap, no focus return *(verify)*. Escape now closes the menu and returns focus to the menu button (Skeleton `9898fed`). As in Skeleton, no focus trap was added because the menu is disclosure-style. The menu button's `aria-label="Navigation menu"` was removed so the visible "Menu" text is its name.
    - [ ] Rewriting the menu from `role="menubar"`/`menuitem` to a disclosure pattern — Skipped — same as Skeleton.
- [x] **Main `<nav>`** — add `aria-label="Main"` *(verify)*. Added to `<nav class="Header-items">` in `snippets/header.twig`. That nav also holds the logo and menu button.
- [x] **`title` as the only label** on social and logo links; social links open in a new window with no warning *(verify)*. Logo links already get their name from the logo alt text (defaults to the company name) or the text logo. TopBar social links now have visually hidden "(opens in a new window)" text, and the icon's label uses the `socialNames` map instead of `social.name`, which could be empty. `title` was kept, as in Skeleton.
- [x] **Required marker** — add `aria-hidden="true"` to the `*` in labels *(verify)*. Confirmed and fixed (Skeleton `58182bf`).
- [x] **Upload previews** use `alt="Image"` *(verify)* (`macros/form-macros.twig`). Confirmed and fixed (Skeleton `8a328ec`).
- [ ] **Video/audio** — no `<track>` captions or transcript option *(verify)*. Skipped — same as Skeleton. This is confirmed in `blocks/video.twig` and `blocks/audio.twig`; the fix needs a captions field in the CMS.
- [x] **Landmark labels** — footer navs and sidebar `<aside>` elements have no `aria-label` *(verify)*. The footer nav is wrapped in `<nav aria-label="Footer">` in `navigation/footer.twig`. The two-column sidebars are labelled "Section navigation"/"Sidebar", and the calendar agenda/list `<aside>` is labelled "Calendar sidebar".
- [ ] **`lang="en"` is hardcoded** in `snippets/header.twig` *(verify)*. Skipped — same as Skeleton. This is confirmed; replacing it needs a site language setting from the CMS.
- [ ] **Slider pause/play control** — Skipped — same as Skeleton.

## Verification

- [ ] `npm run build` completes and `npm run stylelint` shows no new warnings. Stylelint is clean. `npm run build` wasn't run. `npm run jslint` reports one error, which was already there: `no-useless-assignment` in `js/navigation/accessibility.js:96`, a file these changes didn't touch.
- [ ] Every changed block renders in the CMS, including the Margin and Width options
- [ ] Keyboard check: the skip link appears above the sticky header and works on the full-width layout; focus inside the sticky header keeps it visible; accordion headings open with Enter/Space

## Follow-up fixes (2026-09-13)

- [x] **`blocks/columned-content.twig` width override** — removed the `width:` values from `count2`–`count6` and every `{% set width = countN.width %}` line. They overwrote the block's Width field, so the Width setting never took effect (same fix as image-grid, Carmine commit `1745710`).
- [x] **`blocks/image-grid.twig`** — changed `blockWidth(width)` to `blockWidth(width, true)` so it matches the other themes. The file is now identical to Carmine's.
- [x] **`blog/post.twig`** — the comment author's website link opens in a new window; added `rel="noopener"` and a visually hidden "(opens in a new window)" (same as Carmine).
