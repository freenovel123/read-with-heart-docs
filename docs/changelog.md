# 更新日志

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
