# Agent Note: dark mode retinted to the solarized dark palette

Status: implemented

English | [中文](2026-09-02-solarized-dark-palette.zh.md)

## Problem

The dark mode palette ships as a verbatim copy of the light static token block under `body[data-ds-dark-theme]` in `design-platform.css`: neutral dark-blue surfaces, near-white text. An earlier retint to the gruvbox dark hard palette (implemented on the `feature/gruvbox-dark-hard` branch) read as too high-contrast in daily use: `fg0`-bright text on `bg0_h` plus saturated warm accents. The request was a second retint with lower contrast: Ethan Schoonover's solarized dark palette, where the muted `base1`/`base0` text steps and desaturated accents are designed for exactly that.

## Decision

The `--dsw-static-*` scale under `body[data-ds-dark-theme]` remaps to the solarized dark palette: `base03`/`base02` surface steps (`rgb(0, 43, 54)` base, `rgb(7, 54, 66)` elevated), `base1`/`base0`/`base00`/`base01` text steps (primary `rgb(147, 161, 161)`, secondary `rgb(101, 123, 131)`, caption `rgb(88, 110, 117)`), and the solarized accents (blue `#268bd2`, cyan `#2aa198`, green `#859900`, yellow `#b58900`, orange `#cb4b16`, red `#dc322f`, magenta `#d33682`). Every token name stays declared; alias tokens and the light block are untouched. Intermediate ramp steps are linear RGB blends between the solarized anchors preserving the old ramp's ordering (base darkest, layers progressively lighter, text steps dimming from `base1`).

Companion dark overrides move with the palette: `shiki.css` syntax tokens (string/constant cyan, keyword green, function/link blue, comment `base01`), `gradient-shadow-text.css` thinking gradient, `JsonTree.module.css` (property blue, string green, number cyan, keyword magenta), and the boot splash in `packages/client/web/src/boot-page.module.css`. Literal values inside the dark alias block retint too (`bg-mask-drop`, tool-bar fills, `interactive-bg-hover-danger`). Theme-independent fixed surfaces stay fixed: the both-themes hover card (`#2C2C2E`), data-URI SVG colors that cannot resolve CSS variables, the katex error red, and the ANSI terminal palette.

## Testing

- `apps/web/tests/settings-chrome.e2e.ts` boot-page background pin updated from `rgb(21, 21, 23)` to `rgb(0, 43, 54)`.
- `snapshots/web/sidebar-scrollbar/geometry.expected.md` dark lines updated to the solarized-resolved l2 pair (`rgb(95, 117, 124)` thumb through `neutral-600`, `rgb(101, 123, 131)` hover through `neutral-550`).
- `pnpm run test:gui` and the full `pnpm run build` pass; the resolution was confirmed in real chromium (base background `rgb(0, 43, 54)`, primary text `rgb(147, 161, 161)`, scrollbar pair matches the golden). The web e2e replay lane is Linux/macOS-owned (the seed fixture substitutes `{{cwd}}` raw into JSON, which a Windows backslash temp path breaks), so the golden update is derived from the token chain and verified by CI replay.

## Alternatives considered

- **Gruvbox dark hard** — implemented first and used daily; rejected for this iteration because its `fg0` text on `bg0_h` and saturated accents still read high-contrast. Its implementation lives on the `feature/gruvbox-dark-hard` branch.
- **Registering a selectable theme alongside the built-in palettes** — rejected: the request is a direct replacement of the dark palette, and a third built-in theme would extend the Host settings schema and the Appearance row without a consumer.
- **Retinting the both-themes surfaces (HoverCard `#2C2C2E`, workspace row hover cards)** — rejected: figma fixes those surfaces identically in light and dark, so changing them alters light mode, which stays byte-identical.

## Consequences

- Dark mode reads at the muted contrast solarized is designed for; primary text on the base sits near the WCAG AA boundary instead of far above it, which is the requested trade.
- Every other dark-mode surface renders solarized; light mode renders byte-identically to before.
- The palette lives only in the compiled CSS bundle: no config keys, wire fields, or settings schema changed.
- The solarized blue `#268bd2` is a mid-saturation accent; surfaces that need a stronger signal (send buttons, focus rings) keep it as the alias layer already maps them to the 400/500 accent steps, accepting solarized's softer look.
