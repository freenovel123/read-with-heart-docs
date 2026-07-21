# MEMORY.md - Read With Heart Docs

## 项目信息
- 项目名：read-with-heart-docs
- 日期：2026-06-26
- 技术栈：MkDocs、Material for MkDocs、Markdown
- 项目类型：Read With Heart / ParsingBook 文档站

## 架构与约定
- 文档入口由 `mkdocs.yml` 的 `nav` 管理。
- 书源 JS API 主要维护在 `docs/native-to-js.md`。
- 文档默认使用中文，示例代码使用 JavaScript。

## 工作记录
- 2026-07-17：V2 字段规则文档以 `docs/制作源/rules-Introduction.md#field-rule-pipeline` 为公共事实源：基础规则为空且存在 JS 或正则后处理时以原始内容为初始值，`<js>` 与 `##正则` 按书写顺序逐步消费结果；字段 JS 使用 `value/config`，保留 `${value}` 源码替换和空值/无返回/失败回退。XPath、JSONPath、CSS、响应 JS 与 Native API 页面只补边界和交叉链接，禁止复制多套顺序语义。
- 2026-06-26：项目缺少 `AGENTS.md` 和 `MEMORY.md`，且 `/Users/xutaoyu/.codex/templates` 不存在，已根据目录结构和 README 创建最小可用项目说明与记忆文件。
- 2026-06-26：书源 JS API 教程位置为 `docs/native-to-js.md`；新增 SHA 哈希文档时应放在 MD5 附近，强调哈希不是加解密，返回小写十六进制字符串。
- 2026-06-28：Web API `/api/site/list` 文档已补充 `order` 参数；不传时跟随 App 书源管理页当前排序，`0`~`7` 分别对应默认、名称、更新时间、创建时间和源网址排序。
- 2026-06-28：V2 搜索规则新增/维护 `ruleSearch.aliasName` 作为书籍别名规则；Web API 示例必须使用当前 V2 结构，并说明 PC 写源保存完整 `siteJson` 时会保留该字段。
- 2026-06-28：规则说明页 URL 字段统一跳转到 `#request-url-host`；该段说明请求地址会按运行时 `config.host` 自动补全，并覆盖完整 URL、相对路径、request JS 改 host、前置请求、跨场景地址延迟补全和当前场景立即消费地址。
