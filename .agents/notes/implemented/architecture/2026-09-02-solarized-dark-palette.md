# Agent Note: dark mode retinted to the solarized dark palette

Status: implemented

English | [中文](2026-09-02-solarized-dark-palette.zh.md)

## Problem

The dark mode palette ships as a verbatim copy of the light static token block under `body[data-ds-dark-theme]` in `design-platform.css`: neutral dark-blue surfaces, near-white text. An earlier retint to the gruvbox dark hard palette (implemented on the `feature/gruvbox-dark-hard` branch) read as too high-contrast in daily use: `fg0`-bright text on `bg0_h` plus saturated warm accents. The request was a second retint with lower contrast: Ethan Schoonover's solarized dark palette, whose muted text steps and desaturated accents are designed for exactly that.

## Decision

The `--dsw-static-*` scale under `body[data-ds-dark-theme]` remaps to the solarized dark palette: `base03`/`base02` surface steps (`rgb(0, 43, 54)` base, `rgb(7, 54, 66)` elevated), a `base1`-`base01` text ramp (primary `rgb(147, 161, 161)`, primary-dimmed `rgb(139, 155, 156)`, secondary `rgb(131, 148, 150)`, tertiary `rgb(101, 123, 131)`, caption `rgb(95, 117, 124)`), the cyan accent trio `rgb(42, 161, 152)` on the deepseek/blue 400/450/500 steps, and the remaining solarized hues (green `#859900`, yellow `#b58900`, orange `#cb4b16`, red `#dc322f`, magenta `#d33682`). Every token name stays declared; alias tokens and the light block are untouched. Intermediate ramp steps are linear RGB blends between the solarized anchors preserving the old ramp's ordering (base darkest, layers progressively lighter, text steps dimming from `base1`).

Applied to the 0.2.1 token set, the same mapping also covers the tokens added since: the alpha steps `green-500-a08`/`a12` and `red-400-a12`/`red-600-a08` carry the solarized green and red at the same alphas; the `--dsw-alias-file-diff-*` literal surfaces take solarized tints (added `rgb(20, 60, 46)` over a `rgb(11, 52, 50)` gutter, deleted `rgb(33, 44, 53)` over `rgb(18, 44, 53)`) with the solarized green and red markers; the translucent menu fills take `rgba(7, 54, 66, 0.45)` and `rgba(4, 49, 60, 0.94)`, the latter also for the `html[data-platform='darwin']` override; and `bg-document-selection` follows the cyan accent through `--dsw-static-blue-500`.

Companion dark overrides move with the palette: `shiki.css` syntax tokens (string/constant cyan, keyword green, function/link blue, comment `base01`), `gradient-shadow-text.css` thinking gradient, `JsonTree.module.css` (property blue, string green, number cyan, keyword magenta), and the boot splash in `packages/client/web/src/boot-page.module.css`. Literal values inside the dark alias block retint too (`bg-mask-drop`, tool-bar fills, `interactive-bg-hover-danger`, `file-diff-*`, `menu-surface-fill`, `menu-group-header-fill`). Theme-independent fixed surfaces stay fixed: the both-themes hover card (`#2C2C2E`), data-URI SVG colors that cannot resolve CSS variables, the katex error red, and the ANSI terminal palette.

The pre-plugin canvas belongs to `packages/client/ui-theme/src/boot-theme.ts`: its injected head CSS colors the document before any plugin stylesheet loads, so its `DARK_BACKGROUND` has to equal `--dsw-static-neutral-bluish-950` and track this palette.

## Contrast calibration

The palette is deliberately quieter than the neutral one, but it must not undo the already-released link and settings-panel legibility work ([clickable links](../feature/2026-09-04-web-clickable-link-styles.md)). Measured against `base03`:

- Links resolve through `--dsw-alias-link` → `deepseek-400`, which is cyan `#2aa198` at 4.7:1; solarized blue `#268bd2` gave 4.07:1. Cyan is also the hue solarized assigns to dark backgrounds, and one hue across the accent trio keeps `button-info-fill` (400) and `button-info-hover` (500) from shifting color on hover.
- The text ramp is lifted one step above the plain solarized anchors so no label falls below 3:1: primary-dimmed 5.2:1, secondary 4.7:1, tertiary 3.4:1, caption 3.1:1.
- Solarized red `#dc322f` and orange `#cb4b16` sit near 3.2:1: error and warning text keeps its weight and icon pairing, as solarized itself does.
- The dark accent takes cyan for links, focus rings, filled info controls, file-type icons, and the ongoing state dot; the blue-tinted deepseek/blue tint steps stay declared but no dark alias resolves to solarized blue.

## Testing

- `packages/client/ui-theme/tests/boot-theme.client.spec.ts` pins the injected dark canvas CSS at `#002b36`, alongside the `dataset.dsThemeSource` assertions the Host bootstrap gained since.
- `packages/client/ui-theme/tests/elevation-styles.client.spec.ts` pins the dark translucent menu material at `rgba(7, 54, 66, 0.45)` and its darwin override at `rgba(4, 49, 60, 0.94)`.
- `apps/web/tests/settings-chrome.e2e.ts` boot-page background pin updated from `rgb(21, 21, 23)` to `rgb(0, 43, 54)`; that value comes from the injected boot style, since the boot page no longer owns a dark background of its own.
- `snapshots/web/sidebar-scrollbar/geometry.expected.md` dark lines updated to the solarized-resolved l2 pair (`rgb(95, 117, 124)` thumb through `neutral-600`, `rgb(101, 123, 131)` hover through `neutral-550`), keeping master's 5px band.
- `pnpm run test:gui` passes (9773 tests) apart from environment-bound cases: the Windows-only symlink refusal in `ui-deliverables/tests/present-open.host.spec.ts`, and `ui-trajectory`'s `client-bundle` mount, which reads a built artifact predating this branch; four timeout flakes pass in isolation. The web e2e replay lane is Linux/macOS-owned (the seed fixture substitutes `{{cwd}}` raw into JSON, which a Windows backslash temp path breaks), so the golden update is derived from the token chain and verified by CI replay.

## Alternatives considered

- **Gruvbox dark hard** — implemented first and used daily; rejected for this iteration because its `fg0` text on `bg0_h` and saturated accents still read high-contrast. Its implementation lives on the `feature/gruvbox-dark-hard` branch.
- **Solarized blue `#268bd2` for links** — rejected: 4.07:1 on `base03` weakens the legibility the clickable-link work exists to provide, and solarized reserves blue for light backgrounds.
- **Registering a selectable theme alongside the built-in palettes** — rejected: the request is a direct replacement of the dark palette, and a third built-in theme would extend the Host settings schema and the Appearance row without a consumer.
- **Retinting the both-themes surfaces (HoverCard `#2C2C2E`, workspace row hover cards)** — rejected: figma fixes those surfaces identically in light and dark, so changing them alters light mode, which stays byte-identical.

## Consequences

- Dark mode reads at the muted contrast solarized is designed for; the text ramp's floor is 3.1:1 rather than the neutral palette's 8:1 and above, which is the requested trade.
- Every other dark-mode surface renders solarized; light mode renders byte-identically to before.
- The palette lives only in the compiled CSS bundle: no config keys, wire fields, or settings schema changed.
- The pre-plugin frame and the resolved app now agree only while `boot-theme.ts`'s `DARK_BACKGROUND` matches `--dsw-static-neutral-bluish-950`; a future palette change has to update both.
