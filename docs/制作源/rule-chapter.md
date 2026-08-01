# 章节规则

章节规则对应书源 JSON 的 `ruleChapter`，负责请求章节列表，并从响应中解析每一章的名称、地址和时间。

通用规则语法、请求头继承、变量替换和地址补全见[规则说明概述](rules-Introduction.md)。

## 输入参数

| 参数 | 说明 | 用例 |
| --- | --- | --- |
| `bookUrl` | 详情阶段解析出的章节列表地址 | `config.bookUrl` |
| `infoUrl` | 书籍详情地址 | `config.infoUrl` |
| `bookName` | 书籍名称 | `config.bookName` |
| `bookAuthor` | 作者 | `config.bookAuthor` |
| `url` | 当前章节列表请求地址，首屏初始值等于 `bookUrl` | `config.url` |
| `pageIndex` | 当前页码 | `config.pageIndex` |

## 规则字段

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `chapterList` | 是 | 从响应中提取章节条目列表 |
| `chapterName` | 是 | 从单个条目提取章节名称 |
| `chapterUrl` | 是 | 提取章节正文地址；结果进入正文的 `config.chapterUrl` |
| `chapterTime` | 否 | 提取章节发布时间 |
| `page` | 否 | 当前分页规则 |
| `next` | 否 | 下一页地址规则 |
| `bookAuthor` | 否 | 章节列表响应中的作者规则 |
| `method` / `params` / `header` | 否 | 请求方法、参数和场景请求头 |
| `preRequests` | 否 | 正式章节请求前的前置请求 |
| `request` / `response` | 否 | 请求配置 JS 与响应预处理 JS，只使用 `@js:` |

## 地址传递

```text
ruleChapter.chapterUrl
        ↓
正文 config.chapterUrl
        ↓
正文 config.url（首屏初始值）
```

这里的 `chapterUrl` 是每个章节条目的正文地址，不是章节列表本身的请求地址。

## 分页地址

章节首屏使用 `config.url`。分页时 `config.nextUrl` 优先；为空时使用 `config.bookUrl`。在 `request @js:` 中只修改 `config.url` 不会自动改变后续分页地址。

## 最小示例

```json
{
  "ruleChapter": {
    "chapterList": "//ul[@id='chapter-list']/li",
    "chapterName": ".//a/text()",
    "chapterUrl": ".//a/@href",
    "chapterTime": ".//time/text()",
    "next": "//a[@rel='next']/@href"
  }
}
```
