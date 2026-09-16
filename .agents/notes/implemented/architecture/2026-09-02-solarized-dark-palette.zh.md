# Agent Note: 暗色模式改为 solarized dark 色板

Status: implemented

[English](2026-09-02-solarized-dark-palette.md) | 中文

## Problem

暗色模式色板在 `design-platform.css` 的 `body[data-ds-dark-theme]` 下以浅色 static token 块的逐字拷贝声明:中性深蓝表面、近白文字。此前的一次 gruvbox dark hard 换色(实现在 `feature/gruvbox-dark-hard` 分支)日常使用后仍显对比度过高:`bg0_h` 上的 `fg0` 亮文字加上饱和暖色强调。需求是再出一版更低对比度的换色:Ethan Schoonover 的 solarized dark 色板,其克制的文字阶与低饱和强调色正是为此设计。

## Decision

`body[data-ds-dark-theme]` 下的 `--dsw-static-*` 阶重映射为 solarized dark 色板:`base03`/`base02` 表面阶(底色 `rgb(0, 43, 54)`,抬升面 `rgb(7, 54, 66)`)、`base1`–`base01` 文字阶(主文字 `rgb(147, 161, 161)`,主文字弱化 `rgb(139, 155, 156)`,次级 `rgb(131, 148, 150)`,三级 `rgb(101, 123, 131)`,caption `rgb(95, 117, 124)`)、deepseek/blue 400/450/500 三阶取青色强调 `rgb(42, 161, 152)`,以及其余 solarized 色相(绿 `#859900`、黄 `#b58900`、橙 `#cb4b16`、红 `#dc322f`、品红 `#d33682`)。每个 token 名都保留声明;别名层与浅色块不动。中间阶是 solarized 锚点之间的线性 RGB 混色,保持旧暗色阶的明暗顺序(底色最暗、层级逐级变亮、文字阶从 `base1` 依次变暗)。

配套暗色覆盖同步换色:`shiki.css` 语法 token(string/constant 青、keyword 绿、function/link 蓝、comment `base01`)、`gradient-shadow-text.css` 思考渐变、`JsonTree.module.css`(property 蓝、string 绿、number 青、keyword 品红)、`packages/client/web/src/boot-page.module.css` 启动页。暗色别名块内的字面值同步换色(`bg-mask-drop`、工具栏填充、`interactive-bg-hover-danger`)。与主题无关的固定表面保持不变:双主题同色的悬停卡(`#2C2C2E`)、无法解析 CSS 变量的 data-URI SVG 颜色、katex 错误红、ANSI 终端色板。

插件加载前的画布底色归 `packages/client/ui-theme/src/boot-theme.ts` 所有:它注入的 head CSS 在任何插件样式表加载前就给文档上色,因此其 `DARK_BACKGROUND` 必须等于 `--dsw-static-neutral-bluish-950`,并与本色板同步。

## Contrast calibration

色板刻意比中性色板安静,但不能抵消 0.1.6 版本线里已交付的链接与设置面板可读性工作([clickable links](../feature/2026-09-04-web-clickable-link-styles.md))。以 `base03` 为基准实测:

- 链接经 `--dsw-alias-link` → `deepseek-400` 解析为青色 `#2aa198`,对比度 4.7:1;solarized 蓝 `#268bd2` 仅为 4.07:1。青色同样是 solarized 指定给深色背景的强调色,且强调三阶同色可避免 `button-info-fill`(400)与 `button-info-hover`(500)在悬停时跳色。
- 文字阶在 solarized 原始锚点基础上整体上调一档,使任何标签都不低于 3:1:主文字弱化 5.2:1、次级 4.7:1、三级 3.4:1、caption 3.1:1。
- solarized 红 `#dc322f` 与橙 `#cb4b16` 约 3.2:1:错误与警告文字依靠字重与图标配对,与 solarized 自身做法一致。
- 暗色强调色取青色,覆盖链接、焦点环、填充信息控件、文件类型图标与进行中状态点;蓝色调的 deepseek/blue 淡色阶仍保留声明,但暗色下已无别名解析到 solarized 蓝。

## Testing

- `packages/client/ui-theme/tests/boot-theme.client.spec.ts` 钉住注入的暗色画布 CSS 为 `#002b36`。
- `apps/web/tests/settings-chrome.e2e.ts` 启动页背景断言从 `rgb(21, 21, 23)` 更新为 `rgb(0, 43, 54)`;该值现在来自注入的启动样式,因为启动页自身已不再持有暗色底色。
- `snapshots/web/sidebar-scrollbar/geometry.expected.md` 暗色两行更新为 solarized 解析后的 l2 对(thumb 经 `neutral-600` 得 `rgb(95, 117, 124)`,hover 经 `neutral-550` 得 `rgb(101, 123, 131)`)。
- `pnpm run test:gui` 与全量 `pnpm run build` 通过;解析结果经真实 chromium 确认(底色 `rgb(0, 43, 54)`、主文字 `rgb(147, 161, 161)`、链接 `rgb(42, 161, 152)`、滚动条对与金样一致)。web e2e 回放车道由 Linux/macOS 拥有(seed fixture 把 `{{cwd}}` 原样替换进 JSON,Windows 反斜杠临时路径会破坏 JSON),因此金样更新由 token 链推导,并由 CI 回放验证。

## Alternatives considered

- **Gruvbox dark hard** — 先实现并日常使用;本次迭代否决,因其 `bg0_h` 上的 `fg0` 文字与饱和强调仍显高对比。实现在 `feature/gruvbox-dark-hard` 分支。
- **链接沿用 solarized 蓝 `#268bd2`** — 否决:`base03` 上仅 4.07:1,削弱了 clickable-link 工作所要提供的可读性,且 solarized 把蓝留给浅色背景。
- **在内置主题旁新增可选题** — 否决:需求是直接替换暗色板,新增第三个内置主题会扩展 Host 设置 schema 与外观行,却没有消费方。
- **重绘双主题同色表面(HoverCard `#2C2C2E`、工作区行悬停卡)** — 否决:figma 规定这些表面在浅色与暗色下完全相同,改动会波及浅色模式,而浅色模式保持逐字节不变。

## Consequences

- 暗色模式呈现 solarized 设计的克制对比度;文字阶下限为 3.1:1,而非中性色板的 8:1 以上,这是所要求的取舍。
- 其余暗色表面全部渲染 solarized;浅色模式渲染与改动前逐字节一致。
- 色板只存在于编译进 CSS bundle 的产物中:无配置键、无线字段、无设置 schema 变更。
- 首帧与解析后的应用只有在 `boot-theme.ts` 的 `DARK_BACKGROUND` 与 `--dsw-static-neutral-bluish-950` 相等时才一致;后续换色必须同时更新两处。
