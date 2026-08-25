# Agent Note: 暗色模式改为 gruvbox dark hard 色板

Status: implemented

[English](2026-08-25-gruvbox-dark-hard-palette.md) | 中文

## Problem

GUI 的暗色模式使用中性深蓝色阶——近黑表面(底色 `#151517`)、近白文字(`#f9fafb`)——在 `body[data-ds-dark-theme]` 下以浅色 static token 块的逐字拷贝声明。需求是把暗色板整体替换为 gruvbox dark hard 色板,浅色模式保持不变。

## Decision

`packages/client/ui-theme/src/styles/design-platform.css` 中 `body[data-ds-dark-theme]` 下的 `--dsw-static-*` 阶重映射为 gruvbox dark hard 色板(morhetz/gruvbox):`bg0_h`/`bg0`/`bg0_s` 表面阶、`fg0`–`fg4`/`gray` 文字阶,以及深/亮成对强调色(`#458588`/`#83a598` 蓝、`#98971a`/`#b8bb26` 绿、`#d79921`/`#fabd2f` 黄、`#d65d0e` 橙、`#cc241d`/`#fb4934` 红、`#d3869b` 紫)。每个 token 名都保留声明;别名层与浅色块不动,组件仍按原样消费语义别名。中间阶是 gruvbox 锚点之间的线性 RGB 混色,保持旧暗色阶的明暗顺序(底色最暗、层级逐级变亮、文字阶从 `fg0` 依次变暗到 `gray`/`bg4`)。

配套暗色覆盖同步换色:`shiki.css` 语法 token、`gradient-shadow-text.css` 思考渐变、`JsonTree.module.css`、以及 `packages/client/web/src/boot-page.module.css` 启动页。暗色别名块内的字面值同样换色(`bg-mask-drop`、工具栏填充、`interactive-bg-hover-danger`)。与主题无关的固定表面保持不变:双主题同色的悬停卡(`#2C2C2E`)、无法解析 CSS 变量的 data-URI SVG 颜色、katex 错误红、ANSI 终端色板。

## Testing

- `apps/web/tests/settings-chrome.e2e.ts` 的启动页背景断言从 `rgb(21, 21, 23)` 更新为 `rgb(29, 32, 33)`。
- `apps/web/tests/snapshots/sidebar-scrollbar/geometry.expected.md` 更新为 gruvbox 解析后的滚动条 token(`rgb(124, 111, 100)` thumb、`rgb(146, 131, 116)` hover — 暗色 l2 对现在经由 `neutral-600`/`neutral-550` 解析)。
- `pnpm run test:gui`(284 个文件)与全量 `pnpm run build` 在本机通过;其余颜色断言均为 token 相对断言(浅暗对比、token 探针 vs 计算值、`theme-color` 等于计算背景)。web e2e 回放车道由 Linux/macOS 拥有(seed fixture 把 `{{cwd}}` 原样替换进 JSON,Windows 反斜杠临时路径会破坏 JSON),因此金样更新由 token 链推导,并由 CI 回放验证。

## Alternatives considered

- **Solarized dark** — 最初的需求;实施前按需求方指示改为 gruvbox dark hard。
- **把 gruvbox 注册为内置主题之一供选择** — 否决:需求是直接替换暗色模式,新增第三个内置主题会扩展 Host 设置 schema 与外观行,却没有消费方。
- **重绘双主题同色表面(HoverCard `#2C2C2E`、工作区行悬停卡)** — 否决:figma 规定这些表面在浅色与暗色下完全相同,改动会波及浅色模式,而浅色模式保持逐字节不变。

## Consequences

- 所有暗色表面均渲染 gruvbox:底色 `rgb(29, 32, 33)`,主文字 `fg0`,强调色按映射表;`theme-color` 元数据自动跟随计算背景。
- 浅色模式渲染与改动前逐字节一致。
- data-URI SVG 颜色与 ANSI 终端色板按设计保持偏离色板,代码注释已记录该限制。
- 色板只存在于编译进 CSS bundle 的产物中:无配置键、无线字段、无设置 schema 变更。
