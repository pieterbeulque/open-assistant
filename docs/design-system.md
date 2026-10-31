# Design system

The web UI (`src/ui/static/`) uses a small design system built from plain CSS custom properties. The goal is an interface that feels light, calm and approachable. It is monochrome by default and easy to re-brand.

A live reference of every token and component is served at **`/static/design-system.html`** on any running instance. It includes a light/dark switch and a brand-colour playground.

## Principles

- **Monochrome first.** A soft, slightly warm neutral scale carries the whole interface. The only hues are the three semantic families (green = success, orange = warning, red = danger). They always appear as soft tints with a darker, readable foreground, never as loud solid fills. "Info" is neutral.
- **One accent, swappable.** Everything that signals "primary" or "active" reads from a single `--color-accent` token. By default that is the ink colour itself.
- **Accessible by default.** Text meets WCAG AA in both themes. Input and toggle edges meet 3:1 non-text contrast. Every control has a visible `:focus-visible` ring, touch targets are 44px on phones, and `prefers-reduced-motion` is respected.
- **Whitespace over chrome.** Content sits directly on the canvas and is grouped in white cards with a hairline border. Shadows are soft and used sparingly.
- **Inter, self-hosted.** Inter (variable, SIL OFL) ships in `static/fonts/`, so there are no third-party font requests and the PWA works offline. Stylistic sets `cv05` (tailed *l*) and `cv11` (single-storey *a*) are enabled for legibility and a friendlier texture.

## Files and load order

Every page loads, in this order:

| File | Purpose |
| --- | --- |
| `js/theme.js` | Resolves light/dark/system before first paint and sets `<html data-theme>`. |
| `css/tokens.css` | All design tokens: colour (light and dark), type, spacing, radius, elevation, motion. Also holds legacy aliases. |
| `css/base.css` | `@font-face`, reset, document and typography defaults, focus, scrollbars, keyframes. |
| `css/components.css` | Reusable, page-agnostic components (see below). |
| `css/common.css` | App patterns: app shell and navigation, conversation sidebar, Settings building blocks. |
| `css/mobile.css` | Touch targets, safe areas, mobile menu, bottom-sheet modals. |
| `css/pages/*.css` | Optional page-specific styles (`chat.css`, `monitoring.css`, `artifacts.css`). |

`artifact-unlock.html` is served to anonymous visitors and must stay self-contained. It mirrors the few tokens it needs inline and embeds the Inter latin subset as a data URI.

## Tokens

Only use **semantic** tokens in component and page CSS. Never use raw hex values or the `--gray-*` primitives.

| Group | Tokens |
| --- | --- |
| Surfaces | `--color-bg` (canvas), `--color-surface` (cards), `--color-surface-raised` (dialogs), `--color-sunken` (wells, code) |
| Text | `--color-text`, `--color-text-secondary`, `--color-text-tertiary`, `--color-text-disabled` |
| Borders | `--color-border-subtle`, `--color-border`, `--color-border-strong`, `--color-border-control` (3:1 for inputs) |
| Interaction | `--color-fill-hover`, `--color-fill-active`, `--color-focus-ring` |
| Accent | `--color-accent`, `--color-on-accent`, `--color-accent-hover`, `--color-accent-subtle`, `--color-accent-muted` |
| Semantic | `--color-{success,warning,danger,neutral}-{fg,bg,border,solid}` |
| Type | `--font-sans`, `--font-mono`, `--text-xs` … `--text-2xl`, `--leading-*`, `--weight-*` |
| Space | `--space-1` (4px) … `--space-16` (64px) |
| Shape | `--radius-xs` … `--radius-xl`, `--radius-full`, `--control-h-{sm,md,lg}` |
| Elevation | `--shadow-xs`, `--shadow-sm`, `--shadow-md`, `--shadow-lg` |

Dark mode overrides colour tokens only, under `:root[data-theme="dark"]`. When you add a colour token, define it in both blocks.

## Adding a brand accent

The baseline is intentionally monochrome. To give an installation its own accent, define two variables after `tokens.css` loads:

```css
:root {
    --brand: #3b5bdb;          /* the accent */
    --brand-contrast: #ffffff; /* readable text on top of it */
}
```

Primary buttons, toggles, checkboxes, the active navigation icon, the logo tile, focus rings and accent tints all derive from these two variables through `color-mix()`. No other change is needed. Pick a brand colour that has at least 4.5:1 contrast with `--brand-contrast` and 3:1 against the page background.

## Components

All components live in `css/components.css`. Existing class names used by the JavaScript are mapped onto them, so `settings.js` and friends did not need rewriting.

| Component | Markup |
| --- | --- |
| Button | `.btn` + `.btn-primary` / `.btn-secondary` / `.btn-ghost` / `.btn-danger` / `.btn-success`, sizes `.btn-sm` / `.btn-lg`, `.btn-block`; icon-only `.btn-icon` |
| Form | `.form-group` > `.form-label` + `.form-input` / `.form-select` / `.form-textarea` + `.form-hint`; `.input-group`; `.form-control-sm` for compact filters |
| Toggle | `label.toggle-label` > `input.toggle-input` + `span.toggle-slider` (+ `span.toggle-text`) |
| Checkbox / radio | Native inputs (tinted via `accent-color`), `.checkbox-label`, `.radio-label`, `.radio-group` |
| Card | `.card` (+ `.card-header`, `.card-footer`, `.card-flush` for edge-to-edge tables), `.stat-card` |
| Badge | `.badge` + `.badge-{success,warning,danger,neutral,accent,outline}`, optional `.status-dot` inside |
| Alert | `.alert` + `.alert-{info,success,warning,danger}` |
| Tabs | `.tabs` > `button.tab(.active)`, a segmented control |
| Table | `.table` (or legacy `.jobs-table`); add `.table-stack` + `data-label` on cells to stack into cards on phones |
| Modal | `.modal` > `.modal-content` > `.modal-header` / `.modal-body` / `.modal-footer`; `ModalManager` in `common.js` produces the same look |
| Toast | `toast.success()` / `.error()` / `.warning()` / `.info()` from `common.js` |
| Feedback | `.spinner`, `.skeleton`, `.empty-state` |

Icons come from the inline SVG sprite in `js/components.js`. Use them as `<svg class="icon"><use href="#icon-name"></use></svg>`. Prefer these over emoji for interface controls. Service and agent emoji are displayed desaturated inside neutral icon tiles.

## Writing UI copy

Use sentence case for headings, buttons and tabs ("Add agent", not "Add Agent"). Keep labels short and friendly, and say what will happen ("Yes, stop it" rather than "OK").
