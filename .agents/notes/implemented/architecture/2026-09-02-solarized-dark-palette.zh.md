# Agent Note: 暗色模式改为 solarized dark 色板

Status: implemented

[English](2026-09-02-solarized-dark-palette.md) | 中文

## Problem

暗色模式色板在 `design-platform.css` 的 `body[data-ds-dark-theme]` 下以浅色 static token 块的逐字拷贝声明:中性深蓝表面、近白文字。此前的一次 gruvbox dark hard 换色(实现在 `feature/gruvbox-dark-hard` 分支)日常使用后仍显对比度过高:`bg0_h` 上的 `fg0` 亮文字加上饱和暖色强调。需求是再出一版更低对比度的换色:Ethan Schoonover 的 solarized dark 色板,其克制的 `base1`/`base0` 文字阶与低饱和强调色正是为此设计。

## Decision

`body[data-ds-dark-theme]` 下的 `--dsw-static-*` 阶重映射为 solarized dark 色板:`base03`/`base02` 表面阶(底色 `rgb(0, 43, 54)`,抬升面 `rgb(7, 54, 66)`)、`base1`/`base0`/`base00`/`base01` 文字阶(主文字 `rgb(147, 161, 161)`,次级 `rgb(101, 123, 131)`,caption `rgb(88, 110, 117)`),以及 solarized 强调色(蓝 `#268bd2`、青 `#2aa198`、绿 `#859900`、黄 `#b58900`、橙 `#cb4b16`、红 `#dc322f`、品红 `#d33682`)。每个 token 名都保留声明;别名层与浅色块不动。中间阶是 solarized 锚点之间的线性 RGB 混色,保持旧暗色阶的明暗顺序(底色最暗、层级逐级变亮、文字阶从 `base1` 依次变暗)。

配套暗色覆盖同步换色:`shiki.css` 语法 token(string/constant 青、keyword 绿、function/link 蓝、comment `base01`)、`gradient-shadow-text.css` 思考渐变、`JsonTree.module.css`(property 蓝、string 绿、number 青、keyword 品红)、`packages/client/web/src/boot-page.module.css` 启动页。暗色别名块内的字面值同步换色(`bg-mask-drop`、工具栏填充、`interactive-bg-hover-danger`)。与主题无关的固定表面保持不变:双主题同色的悬停卡(`#2C2C2E`)、无法解析 CSS 变量的 data-URI SVG 颜色、katex 错误红、ANSI 终端色板。

## Testing

- `apps/web/tests/settings-chrome.e2e.ts` 启动页背景断言从 `rgb(21, 21, 23)` 更新为 `rgb(0, 43, 54)`。
- `snapshots/web/sidebar-scrollbar/geometry.expected.md` 暗色两行更新为 solarized 解析后的 l2 对(thumb 经 `neutral-600` 得 `rgb(95, 117, 124)`,hover 经 `neutral-550` 得 `rgb(101, 123, 131)`)。
- `pnpm run test:gui` 与全量 `pnpm run build` 通过;解析结果经真实 chromium 确认(底色 `rgb(0, 43, 54)`,主文字 `rgb(147, 161, 161)`,滚动条对与金样一致)。web e2e 回放车道由 Linux/macOS 拥有(seed fixture 把 `{{cwd}}` 原样替换进 JSON,Windows 反斜杠临时路径会破坏 JSON),因此金样更新由 token 链推导,并由 CI 回放验证。

## Alternatives considered

- **Gruvbox dark hard** — 先实现并日常使用;本次迭代否决,因其 `bg0_h` 上的 `fg0` 文字与饱和强调仍显高对比。实现在 `feature/gruvbox-dark-hard` 分支。
- **在内置主题旁新增可选题** — 否决:需求是直接替换暗色板,新增第三个内置主题会扩展 Host 设置 schema 与外观行,却没有消费方。
- **重绘双主题同色表面(HoverCard `#2C2C2E`、工作区行悬停卡)** — 否决:figma 规定这些表面在浅色与暗色下完全相同,改动会波及浅色模式,而浅色模式保持逐字节不变。

## Consequences

- 暗色模式呈现 solarized 设计的克制对比度:底色上的主文字接近 WCAG AA 边界而非远超其上,这是所要求的取舍。
- 其余暗色表面全部渲染 solarized;浅色模式渲染与改动前逐字节一致。
- 色板只存在于编译进 CSS bundle 的产物中:无配置键、无线字段、无设置 schema 变更。
- solarized 蓝 `#268bd2` 是中等饱和强调色;需要更强信号的表面(发送键、焦点环)仍经由别名层映射到 400/500 强调阶,接受 solarized 更柔和的外观。
