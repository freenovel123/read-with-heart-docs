# 详情规则

详情规则对应书源 JSON 的 `ruleBookInfo`，负责请求书籍详情页，补全书名、作者、简介、封面等信息，并解析章节列表入口。

通用规则语法、请求头继承、变量替换和地址补全见[规则说明概述](rules-Introduction.md)。

## 输入参数

| 参数 | 说明 | 用例 |
| --- | --- | --- |
| `infoUrl` | 搜索或发现阶段解析出的详情地址或书籍 ID | `config.infoUrl` |
| `url` | 本次详情请求地址，首屏初始值等于 `infoUrl` | `config.url` |
| `bookName` | 上一场景已有书名 | `config.bookName` |
| `bookAuthor` | 上一场景已有作者 | `config.bookAuthor` |

## 规则字段

| 字段 | 说明 |
| --- | --- |
| `chapterListUrl` | 章节列表地址；解析结果进入章节场景的 `config.bookUrl` |
| `bookName` | 详情书名；非空时覆盖搜索或发现结果 |
| `bookAuthor` | 详情作者；非空时覆盖搜索或发现结果 |
| `ruleExtra.coverUrl` | 封面地址，兼容旧字段 `imageUrl` |
| `ruleExtra.bookSize` | 字数或文件大小 |
| `ruleExtra.lastUpdateTime` | 最近更新时间 |
| `ruleExtra.lastChapterName` | 最新章节名 |
| `ruleExtra.introduce` | 书籍简介 |
| `ruleExtra.classify` | 分类 |
| `ruleExtra.status` | 连载、完结等状态 |
| `importUrl` | 导入书籍地址时使用的 URL 匹配或转换规则 |
| `method` / `params` / `header` | 请求方法、参数和场景请求头 |
| `preRequests` | 正式详情请求前的前置请求 |
| `request` / `response` | 请求配置 JS 与响应预处理 JS，只使用 `@js:` |

## 地址传递

```text
ruleBookInfo.chapterListUrl
        ↓
章节 config.bookUrl
        ↓
章节 config.url（首屏初始值）
```

如果 `chapterListUrl` 为空或没有解析出值，章节列表地址会回退为详情地址。

## JS 规则边界

详情普通字段的 JS 后处理使用 `<js>...</js>`；`request` 与 `response` 是两个独立入口，只使用 `@js:`。如果详情规则整体为空，App 会保留搜索或发现阶段已有的书籍信息并继续后续流程。

## 最小示例

```json
{
  "ruleBookInfo": {
    "chapterListUrl": "//a[@id='chapters']/@href",
    "bookName": "//h1/text()",
    "bookAuthor": "//span[@class='author']/text()",
    "ruleExtra": {
      "coverUrl": "//img[@class='cover']/@src",
      "introduce": "//div[@class='intro']/text()"
    }
  }
}
```
