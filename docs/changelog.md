# 更新日志

## 2026-08-01

### V2 `commentUrl` 支持异步请求

V2 正文规则的 `ruleContent.commentUrl` 现支持：

- 在 `@js:` 中使用 `await app.get(...)`、`await app.post(...)`。
- 返回纯 URL、URL + Header、`{url, header}`。
- 返回 `@html:` 或 `{html, baseURL}`。
- 继续兼容原有纯 URL 和同步 `@js:` 规则；V1 行为不变。

完整协议、返回格式、示例及 HTML Header 边界，请查看[制作源 → 规则说明 → 正文](制作源/rule-content.md#comment-url)。
