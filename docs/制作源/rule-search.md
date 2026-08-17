# 搜索规则

搜索规则对应书源 JSON 的 `ruleSearch`，负责根据关键词发起请求，并把响应解析为书籍列表。

通用规则语法、请求头继承、变量替换和地址补全见[规则说明概述](rules-Introduction.md)。

## 输入参数

| 参数 | 说明 | 用例 |
| --- | --- | --- |
| `keyword` | 当前搜索关键词 | `config.keyword`、`@get{keyword}` |
| `url` | 当前搜索请求地址，支持 host 自动补全 | `config.url` |
| `pageIndex` | 当前页码 | `config.pageIndex` |

## 规则字段

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `url` | 是 | 搜索请求地址 |
| `bookList` | 是 | 从响应中提取书籍条目列表 |
| `bookName` | 是 | 从单个条目提取书名 |
| `bookAuthor` | 否 | 提取作者 |
| `aliasName` | 否 | 提取又名、原名、译名等额外书名，并参与搜索匹配 |
| `bookUrl` | 是 | 提取详情地址或书籍 ID；结果进入详情的 `config.infoUrl` |
| `ruleExtra` | 否 | 封面、简介、分类等扩展信息规则 |
| `pageMax` | 否 | 最大搜索页数规则 |
| `method` / `params` / `header` | 否 | 请求方法、参数和场景请求头 |
| `preRequests` | 否 | 搜索正式请求前的前置请求 |
| `request` / `response` | 否 | 请求配置 JS 与响应预处理 JS，只使用 `@js:` |

## 地址传递

```text
ruleSearch.bookUrl
        ↓
详情 config.infoUrl
        ↓
详情 config.url（首屏初始值）
```

`bookUrl` 可以只解析书籍 ID。进入详情后，可在 `request @js:` 中读取 `config.infoUrl`，再生成真正的 `config.url`。

## aliasName 匹配规则

`aliasName` 为空时不参与匹配；非空时，搜索结果会按 `bookName OR aliasName` 判断，任意一个命中关键词即可进入匹配结果。非空别名也会作为搜索结果和加入书架后的显示书名使用。

## 最小示例

```json
{
  "ruleSearch": {
    "url": "/search?q=@{keyword}",
    "bookList": "//div[@class='book-item']",
    "bookName": ".//h3/text()",
    "aliasName": ".//span[@class='alias']/text()",
    "bookAuthor": ".//span[@class='author']/text()",
    "bookUrl": ".//a/@href"
  }
}
```
