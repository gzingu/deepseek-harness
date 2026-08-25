# Agent Note: dark mode retinted to the gruvbox dark hard palette

Status: implemented

English | [中文](2026-08-25-gruvbox-dark-hard-palette.zh.md)

## Problem

The GUI's dark mode used a neutral dark-blue scale — near-black surfaces (`#151517` base), near-white text (`#f9fafb`) — declared as a verbatim copy of the light static token block under `body[data-ds-dark-theme]`. The request was to replace the dark palette with the gruvbox dark hard palette while leaving light mode untouched.

## Decision

The `--dsw-static-*` scale under `body[data-ds-dark-theme]` in `packages/client/ui-theme/src/styles/design-platform.css` remaps to the gruvbox dark hard palette (morhetz/gruvbox): `bg0_h`/`bg0`/`bg0_s` surface steps, `fg0`–`fg4`/`gray` text steps, and the dark/bright accent pairs (`#458588`/`#83a598` blue, `#98971a`/`#b8bb26` green, `#d79921`/`#fabd2f` yellow, `#d65d0e` orange, `#cc241d`/`#fb4934` red, `#d3869b` purple). Every token name stays declared; the alias layer and the light block are untouched, so components keep consuming semantic aliases as before. Intermediate ramp steps are linear RGB blends between the gruvbox anchors that preserve the old dark ramp's ordering (base darkest, layers progressively lighter, text steps dimming from `fg0` to `gray`/`bg4`).

Companion dark overrides move with the palette: `shiki.css` syntax tokens, `gradient-shadow-text.css` thinking gradient, `JsonTree.module.css`, and the boot splash in `packages/client/web/src/boot-page.module.css`. Literal values inside the dark alias block retint too (`bg-mask-drop`, tool-bar fills, `interactive-bg-hover-danger`). Fixed surfaces that are theme-independent stay fixed: the both-themes hover card (`#2C2C2E`), data-URI SVG colors that cannot resolve CSS variables, the katex error red, and the ANSI terminal palette.

## Testing

- `apps/web/tests/settings-chrome.e2e.ts` boot-page background pin updated from `rgb(21, 21, 23)` to `rgb(29, 32, 33)`.
- `apps/web/tests/snapshots/sidebar-scrollbar/geometry.expected.md` updated to the gruvbox-resolved scrollbar tokens (`rgb(124, 111, 100)` thumb, `rgb(146, 131, 116)` hover — the dark l2 pair now resolves through `neutral-600`/`neutral-550`).
- `pnpm run test:gui` (284 files) and the full `pnpm run build` pass locally; every other color assertion is token-relative (light/dark comparison, token probe vs computed value, `theme-color` equals the computed background). The web e2e replay lane is Linux/macOS-owned (the seed fixture substitutes `{{cwd}}` raw into JSON, which a Windows backslash temp path breaks), so the golden update is derived from the token chain and verified by CI replay.

## Alternatives considered

- **Solarized dark** — the original request; replaced by gruvbox dark hard at the requester's direction before implementation.
- **Registering a selectable gruvbox theme alongside the built-in palettes** — rejected: the request was a direct replacement of dark mode, and a third built-in theme would extend the Host settings schema and the Appearance row without a consumer.
- **Retinting the both-themes surfaces (HoverCard `#2C2C2E`, workspace row hover cards)** — rejected: figma fixes those surfaces identically in light and dark, so changing them alters light mode, which stays byte-identical.

## Consequences

- Every dark-mode surface renders gruvbox: base `rgb(29, 32, 33)`, primary text `fg0`, accents per the mapping; the `theme-color` metadata follows the computed background automatically.
- Light mode renders byte-identically to before.
- Data-URI SVG colors and the ANSI terminal palette remain off-palette by design; the code comments already document that limitation.
- The palette lives only in the compiled CSS bundle: no config keys, wire fields, or settings schema changed.
