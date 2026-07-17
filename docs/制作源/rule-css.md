# CSS 规则

CSS 规则是 HTML 解析方式之一。规则段的 `engine` 设置为 `css` 后，规则字段会按 CSS selector 提取内容。

CSS 引擎主要用于搜索、发现、详情、章节、正文，以及前置请求 `response.put` 的 HTML 响应解析。

## 启用方式

在对应规则段设置：

```json
{
  "engine": "css"
}
```

搜索规则示例：

```json
{
  "ruleSearch": {
    "engine": "css",
    "bookList": ".book-item",
    "bookName": ".title@text",
    "aliasName": ".alias@text",
    "bookAuthor": ".author@text",
    "bookUrl": "a.title@href",
    "ruleExtra": {
      "coverUrl": "img@src",
      "introduce": ".intro@text"
    }
  }
}
```

`aliasName` 为可选字段，用于从搜索列表项中解析书籍别名。别名为空时不参与搜索匹配；别名非空时，精准匹配和包含匹配都会按 `bookName OR aliasName` 判断。

## 基本写法

| 写法 | 说明 | 示例 |
| --- | --- | --- |
| `selector` | 选择元素并取文本 | `.book-title` |
| `selector@text` | 取元素文本，包含子节点文本 | `.title@text` |
| `selector@ownText` | 只取元素自身文本 | `.title@ownText` |
| `selector@textNodes` | 取直接文本节点 | `.content@textNodes` |
| `selector@html` | 取元素内部 HTML | `.content@html` |
| `selector@all` | 取元素外部 HTML | `.content@all` |
| `selector@href` | 取 `href` 属性 | `a.book@href` |
| `selector@src` | 取 `src` 属性 | `img@src` |
| `selector@attr(name)` | 取指定属性 | `img@attr(data-src)` |
| `@all` | 直接返回当前原文或当前节点 HTML | `@all` |

常用 CSS selector 可直接使用标准写法：

```text
.book-item
div.book-item > a.title
.chapter-list a
img.lazy[data-src]
a:contains(下一页)
```

### 标准 CSS selector 示例

以下写法推荐优先使用：

| 写法 | 说明 | 示例 |
| --- | --- | --- |
| `.class` | 按 class 选择 | `.book-item@text` |
| `#id` | 按 id 选择 | `#content@html` |
| `tag` | 按标签选择 | `a@href` |
| `A B` | 后代选择 | `.book-list a@text` |
| `A > B` | 直接子节点选择 | `.book-list > li@text` |
| `A + B` | 相邻兄弟选择 | `.title + .author@text` |
| `A ~ B` | 后续兄弟选择 | `.intro ~ .chapter-list a@text` |
| `[attr]` | 有指定属性 | `img[data-src]@attr(data-src)` |
| `[attr=value]` | 属性等于指定值 | `meta[property="og:image"]@content` |
| `[attr*=value]` | 属性包含指定值 | `div[class*=content]@text` |
| `:contains(text)` | 文本包含指定内容 | `a:contains(下一页)@href` |
| `:nth-of-type(n)` | 标准 CSS 1 基索引 | `td:nth-of-type(1) a@text` |

`nth-of-type(n)` 是标准 CSS 写法，`n` 从 1 开始；不会被当成阅读 3.0 的 0 基索引再转换。

## 阅读风格简写

为方便迁移部分阅读 3.0 / Jsoup 风格规则，CSS 引擎仅支持以下常用简写。未列出的阅读 3.0 扩展语法不要默认认为可用。

| 写法 | 等效含义 | 示例 |
| --- | --- | --- |
| `class.xxx` | `.xxx` | `class.book-item@text` |
| `id.xxx` | `#xxx` | `id.content@html` |
| `tag.xxx` | `xxx` | `tag.a@href` |
| `a.0` | 第 1 个同类型元素 | `.chapter-list a.0@text` |
| `a.-1` | 最后 1 个同类型元素 | `.chapter-list a.-1@href` |
| `a!0` | 排除第 1 个同类型元素 | `.chapter-list a!0@text` |
| `text.下一页` | 文本包含“下一页”的链接 | `text.下一页@href` |

### 特殊写法与标准写法对照

| 特殊写法 | 标准 CSS 写法 | 说明 |
| --- | --- | --- |
| `class.book-item@text` | `.book-item@text` | class 简写 |
| `id.content@html` | `#content@html` | id 简写 |
| `tag.a@href` | `a@href` | 标签简写 |
| `.chapter-list a.0@text` | `.chapter-list a:nth-of-type(1)@text` | 阅读风格索引从 0 开始 |
| `.chapter-list a.1@text` | `.chapter-list a:nth-of-type(2)@text` | 第 2 个同类型元素 |
| `.chapter-list a.-1@href` | `.chapter-list a:nth-last-of-type(1)@href` | 倒数第 1 个同类型元素 |
| `.chapter-list a!0@text` | `.chapter-list a:not(:nth-of-type(1))@text` | 排除第 1 个同类型元素 |
| `text.下一页@href` | `a:contains(下一页)@href` | 查找文本包含“下一页”的链接 |

建议新书源优先写标准 CSS；特殊写法主要用于迁移旧规则或让规则更短。

### 当前不支持的阅读 3.0 扩展写法

以下写法在部分阅读 3.0 书源中常见，但当前 CSS 引擎没有作为正式能力支持：

| 不建议写法 | 推荐写法 |
| --- | --- |
| `#content@p@text` | `#content p@text` |
| `class.list.0@a.0@text` | `.list.0 a.0@text` 或 `.list:nth-of-type(1) a:nth-of-type(1)@text` |
| `span.2:3:5@text` | 拆成明确的 CSS selector，当前只支持单个索引简写 |
| `tr!0:1` | 拆成明确的 CSS selector，当前只支持单个排除索引 |
| `+dl@dd a` | 使用标准兄弟 / 后代 selector 表达 |

## 多段与兜底

CSS 规则支持当前规则引擎的常用组合符：

| 写法 | 说明 | 示例 |
| --- | --- | --- |
| `A||B` | A 为空时再取 B | `.name@text||h1@text` |
| `A&&B` | 拼接多个规则结果，按换行连接 | `.title@text&&.author@text` |
| `A%%B` | 拼接多个规则结果，按换行连接 | `.content p@text%%.content div@text` |

## 列表解析

`bookList`、`chapterList` 这类列表字段应写成能选中每个列表项的 selector；列表项内部字段会相对当前 item 解析。

```json
{
  "engine": "css",
  "chapterList": ".chapter-list a",
  "chapterName": "@text",
  "chapterUrl": "@href"
}
```

也可以在 item 内继续写子 selector：

```json
{
  "engine": "css",
  "bookList": ".book-item",
  "bookName": ".title@text",
  "bookUrl": "a.title@href"
}
```

## 前置请求 put

HTML 响应可以把 `response.engine` 设置为 `css`，`put` 中的 value 使用 CSS 规则：

```json
{
  "response": {
    "engine": "css",
    "put": {
      "token": "meta[name=csrf-token]@content",
      "userName": ".user-name@text",
      "nextUrl": "a.next@href"
    }
  }
}
```

## 注意事项

- `css` 是独立解析引擎；同一个规则段内的字段会按该段 `engine` 统一解析，不建议在同一字段里混写 XPath 与 CSS。
- 原有 `xpath`、`jsonpath` 规则不受影响；只有把规则段 `engine` 改成 `css` 时才走 CSS 解析。
- `@href`、`@src` 只负责取属性值，最终 URL 仍会按 App 原有逻辑结合书源 `host` 做地址补全。
- CSS 解析只在选择 `engine: css` 的规则段中生效。
- CSS 基础规则后可以继续写 `<js>...</js>` 或 `##正则`；组合规则的输入和执行顺序见[字段规则执行流水线](rules-Introduction.md#field-rule-pipeline)。
