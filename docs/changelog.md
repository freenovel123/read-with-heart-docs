# 更新日志

## 2026-08-17

### 段评图标支持阅读主题颜色变量

自定义段评图标 SVG 模板新增 4 个阅读主题颜色变量：

- `{{readBG}}` 正文背景色、`{{readFont}}` 正文字色、`{{menuBG}}` 菜单背景色、`{{menuFont}}` 菜单文字色
- 可用于 `fill` / `stroke`、内联 `style`、渐变 `stop-color` 和文字颜色，图标随阅读主题实时变色
- SVG 编辑器输入 `{{` 的快捷菜单同步提供 `{{count}}` 与 4 个主题变量
- 固定颜色不受影响；旧图标未指定颜色时按正文字色渲染，编辑保存后自动迁移为 `{{readFont}}`

完整变量表、示例和兼容规则，请查看[美化 → 段评样式](beautify/comment-style.md#1-svg)。

### 段评图标解析属性编辑

段评图标编辑页新增“解析属性”编辑 Sheet：

- 按全部 / 颜色 / 字号 / 对齐 / 坐标分类筛选自动解析的属性
- 支持单项编辑颜色来源（4 个主题变量 + 自定义颜色）、字号、坐标与对齐
- 颜色分类下支持批量修改（全部颜色 / 全部填充 / 全部描边 / 字体颜色），修改即时预览

详见[美化 → 段评样式 → 解析属性编辑](beautify/comment-style.md#3)。

### 批注正文跟随阅读字体

- 批注气泡正文使用与正文一致的阅读字体家族，字号保持 15pt 不变
- 左侧标签样式保持 10 bold 不变

### 文档新增“美化”菜单

- 新增[美化](beautify/comment-style.md)栏目，集中管理阅读美化配置文档：[段评样式](beautify/comment-style.md)、[高亮样式](beautify/read-text-style.md)、[正文头样式](beautify/header-style.md)
- 高亮样式、正文头样式为首份系统化文档，覆盖正则匹配、双主题配色、背景图/装饰/阴影/描边、HTML 变量模板等能力

## 2026-08-10

### V2 详情规则新增 `toolsUrl`

V2 书籍详情规则新增 `ruleBookInfo.toolsUrl`：

- 规则非空时，在书籍详情页右上角添加菜单中显示“书籍工具”。
- 支持纯 URL、相对 URL、同步或异步 `@js:`。
- 支持 `await app.get(...)`、`await app.post(...)`。
- 支持 URL + Header、`{url, header}`、`@html:` 和 `{html, baseURL}`。
- 使用详情场景的 `config.infoUrl`、`config.bookName`、`config.bookAuthor`、`config.params` 和 `config.openParams`。
- 旧书源缺少字段时不显示“书籍工具”菜单项；V1 详情解析流程不变。

完整协议、返回格式和示例，请查看[制作源 → 规则说明 → 详情 → toolsUrl](制作源/rule-detail.md#tools-url)。

## 2026-08-01

### V2 `commentUrl` 支持异步请求

V2 正文规则的 `ruleContent.commentUrl` 现支持：

- 在 `@js:` 中使用 `await app.get(...)`、`await app.post(...)`。
- 返回纯 URL、URL + Header、`{url, header}`。
- 返回 `@html:` 或 `{html, baseURL}`。
- 继续兼容原有纯 URL 和同步 `@js:` 规则；V1 行为不变。

完整协议、返回格式、示例及 HTML Header 边界，请查看[制作源 → 规则说明 → 正文](制作源/rule-content.md#comment-url)。
