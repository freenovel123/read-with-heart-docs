# XPath 规则

XPath 规则用于从 HTML / XML 响应中提取数据。规则段的 `engine` 设置为 `xpath` 时，页面会按 DOM 解析，字段值按 XPath 查询。

## 启用方式

```json
{
  "engine": "xpath"
}
```

## 基本写法

| 写法 | 说明 | 示例 |
| --- | --- | --- |
| `//tag` | 从全文选择指定标签 | `//a` |
| `.//tag` | 从当前节点内部选择指定标签 | `.//a` |
| `/` | 选择直接子节点 | `//ul/li` |
| `//` | 选择任意层级后代节点 | `//div//a` |
| `text()` | 取文本节点 | `//h1/text()` |
| `@attr` | 取属性值 | `//a/@href` |
| `*` | 匹配任意标签 | `//*[@id='content']` |
| `[条件]` | 过滤节点 | `//a[@class='book']` |
| `[1]` | 取第 1 个节点，XPath 索引从 1 开始 | `(//a)[1]/text()` |
| `[last()]` | 取最后 1 个节点 | `(//a)[last()]/@href` |

### 标准 XPath 示例

以下写法推荐优先掌握：

| 目标 | 示例 |
| --- | --- |
| 按 id 选择文本 | `//*[@id='title']/text()` |
| 按 class 选择文本 | `//*[@class='title']/text()` |
| class 包含某个值 | `//*[contains(concat(' ', normalize-space(@class), ' '), ' title ')]/text()` |
| 取链接地址 | `//a[@class='book']/@href` |
| 取图片地址 | `//img[@class='cover']/@src` |
| 取 meta 内容 | `//meta[@property='og:image']/@content` |
| 按文本内容筛选 | `//a[contains(text(),'下一页')]/@href` |
| 按标准化文本筛选 | `//a[contains(normalize-space(.),'下一页')]/@href` |
| 选择某个父节点下的链接 | `//*[@id='list']//a/@href` |
| 取第一个章节名 | `(//a[@class='chapter'])[1]/text()` |
| 取最后一个章节链接 | `(//a[@class='chapter'])[last()]/@href` |

### 绝对路径与相对路径

列表字段会先选中每个列表项，列表项内部字段建议使用相对 XPath。

```json
{
  "bookList": "//div[@class='book-item']",
  "bookName": ".//a[@class='title']/text()",
  "bookUrl": ".//a[@class='title']/@href"
}
```

这里 `bookList` 使用 `//div...` 从整页找列表项；`bookName` 和 `bookUrl` 使用 `.//a...`，表示从当前列表项内部继续查找。

如果在列表项内部仍然写 `//a...`，会从整页重新查找，容易让每个列表项都取到同一个结果。

### 节点文本与属性

XPath 规则最终会返回字符串：

| 写法 | 返回内容 |
| --- | --- |
| `//h1/text()` | `h1` 的直接文本节点 |
| `//h1` | `h1` 节点的文本内容 |
| `//a/@href` | `a` 标签的 `href` 属性 |
| `//img/@src` | `img` 标签的 `src` 属性 |
| `//*[@id='content']//p/text()` | 正文区域内所有段落文本，多个结果按换行拼接 |

## 搜索示例

```json
{
  "ruleSearch": {
    "engine": "xpath",
    "bookList": "//div[@class='book-item']",
    "bookName": ".//a[@class='title']/text()",
    "aliasName": ".//*[@class='alias']/text()",
    "bookAuthor": ".//*[@class='author']/text()",
    "bookUrl": ".//a[@class='title']/@href",
    "ruleExtra": {
      "coverUrl": ".//img/@src",
      "introduce": ".//*[@class='intro']/text()"
    }
  }
}
```

`aliasName` 为可选字段，用于从搜索列表项中解析书籍别名。别名为空时不参与搜索匹配；别名非空时，精准匹配和包含匹配都会按 `bookName OR aliasName` 判断。

## 章节示例

```json
{
  "ruleChapter": {
    "engine": "xpath",
    "chapterList": "//div[@class='chapter-list']//a",
    "chapterName": "./text()",
    "chapterUrl": "./@href",
    "next": "//a[contains(text(),'下一页')]/@href"
  }
}
```

## 前置请求 put

```json
{
  "response": {
    "engine": "xpath",
    "put": {
      "token": "//meta[@name='csrf-token']/@content",
      "name": "//*[@class='user-name']/text()"
    }
  }
}
```

## 注意事项

- 列表字段选中的是每个列表项，列表项内部字段建议使用相对 XPath，如 `.//a/text()`。
- HTML 响应用 `xpath` 或 `css`；JSON 响应不要用 XPath。
- XPath 规则取到的相对 URL 会继续走 App 的地址补全逻辑。
- XPath 后可以继续写 `<js>...</js>` 或 `##正则`；组合规则的输入和执行顺序见[字段规则执行流水线](rules-Introduction.md#field-rule-pipeline)。
