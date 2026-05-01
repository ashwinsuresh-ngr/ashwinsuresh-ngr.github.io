# UI Redesign — Design Spec

**Date:** 2026-05-01  
**Status:** Approved

---

## Goal

Improve the overall polish and modernity of the Pelican blog. The current theme uses a vivid electric-blue background (#2563eb) that feels heavy and unrefined. The redesign adopts a Dark Tech aesthetic with a card-based article grid and a compact sidebar.

---

## Design Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Color direction | Dark Tech (navy/slate) | Agreed in brainstorming — richer than current blue |
| Homepage layout | 2-column card grid | Scannable, modern, confirmed by user |
| Sidebar style | Compact, centered | Avatar-focused, icon-only social links |
| Avatar size | Large (110px) with glow ring | User requested; makes the sidebar feel anchored |
| Implementation approach | CSS + template edits (Approach B) | Best tradeoff: real card grid requires template change, but no new build tools needed |

---

## Color Palette

| Token | Value | Usage |
|---|---|---|
| `bg-base` | `#0f172a` | Page background (main area) |
| `bg-sidebar` | `#080f1e` | Sidebar background |
| `bg-card` | `#1e293b` | Article cards, code blocks |
| `border` | `#334155` | Card borders, dividers |
| `border-hover` | `#3b82f6` | Card hover state |
| `text-primary` | `#f1f5f9` | Headings, card titles |
| `text-secondary` | `#94a3b8` | Article body text |
| `text-muted` | `#64748b` | Card summaries, dates |
| `text-subtle` | `#475569` | Meta labels, tags |
| `accent` | `#3b82f6` | Category labels, links, "Read more" |
| `accent-hover` | `#60a5fa` | Hovered links |
| `code-text` | `#7dd3fc` | Inline code |

---

## Sidebar

**File:** `theme/templates/partial/sidebar.html`

- Width: 220px
- Avatar: 110px circular, 3px solid `#3b82f6` border, double-ring glow (`box-shadow: 0 0 0 6px #0f172a, 0 0 0 8px #1e3a5f`)
- Site name: 16px, bold, `#f1f5f9`
- Subtitle (SITESUBTITLE): 11px, `#475569`
- Navigation (pages): text links, `#94a3b8`, hover → `#f1f5f9` on `#1e293b` background, 6px border-radius
- Social icons: 32×32px rounded squares (`#1e293b`), hover → `#2563eb` background with white icon
- Social icon labels: use Font Awesome icon classes (already in theme) — no text labels

---

## Homepage — Article Card Grid

**File:** `theme/templates/index.html`

Replace the current `<article>` loop with a `<div class="card-grid">` wrapping `<div class="post-card">` elements.

### Card structure (per article):
```
[Category · Date]
[Title — bold, 14px]
[Summary — 12px, muted]
[Read more →]
```

### Card styles:
- Background: `#1e293b`, border: 1px `#334155`, border-radius: 10px, padding: 20px
- Hover: border-color → `#3b82f6`, translateY(-2px)
- Grid: `grid-template-columns: 1fr 1fr`, gap: 16px
- Category: 10px, bold, `#3b82f6`, uppercase
- Title: 14px, 700 weight, `#f1f5f9`
- Summary: 12px, `#64748b`, line-height 1.6 (uses `article.summary`)
- "Read more →": 11px, `#3b82f6`, bold

---

## Article Page

**File:** `theme/templates/article.html`

- Max content width: 640px
- Title: 22px, 800 weight, `#f8fafc`
- Body text: 14px, `#94a3b8`, line-height 1.8
- Inline `<code>`: background `#0f172a`, border `#334155`, color `#7dd3fc`, monospace font
- Tags: pill chips — `#0f172a` background, `#334155` border, `#64748b` text, 999px border-radius
- Meta line (date + category): 11px, category in `#3b82f6`, date in `#475569`

---

## CSS Changes

**Files:** `theme/static/stylesheet/dark-theme.min.css`, `theme/static/stylesheet/style.min.css`

### dark-theme.min.css — full replacement of body/sidebar/main colors:
- `body`: background `#0f172a`, color `#e2e8f0`
- `aside`: background `#080f1e`, border-right `1px solid #1e293b`
- `hr`: `#1e293b`
- `main nav`: border-bottom `#1e293b`
- Links: `#3b82f6`, hover `#60a5fa`

### style.min.css — structural additions:
- `.card-grid`: CSS grid, 2 columns, 16px gap
- `.post-card`: card styles with hover transition
- `.card-meta`, `.card-cat`, `.card-date`, `.card-title`, `.card-summary`, `.card-read`: typography classes
- Avatar: width/height 110px, box-shadow ring
- `.tag`: pill chip styles

---

## Out of Scope

- Dark/light theme toggle (keeping auto-detect as configured)
- Search UI
- Pagination styling (minor cleanup only)
- Mobile responsive changes (current breakpoints remain)

---

## Files to Change

| File | Change |
|---|---|
| `theme/static/stylesheet/dark-theme.min.css` | Replace color palette |
| `theme/static/stylesheet/style.min.css` | Add card grid + avatar styles |
| `theme/templates/index.html` | Replace article loop with card grid |
| `theme/templates/partial/sidebar.html` | Large avatar, compact layout, icon social links |
| `theme/templates/article.html` | Cleaner header, tag pills, constrained width |
