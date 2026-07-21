# JSONPath 规则

JSONPath 规则用于从 JSON 响应中提取数据。规则段的 `engine` 设置为 `jsonpath` 后，规则字段会按 JSONPath 查询。

JSONPath 引擎主要用于搜索、发现、详情、章节、正文，以及前置请求 `response.put` 的 JSON 响应解析。

## 启用方式

```json
{
  "engine": "jsonpath"
}
```

## 基本写法

| 写法 | 说明 | 示例 |
| --- | --- | --- |
| `$` | 根对象 | `$` |
| `$.field` | 读取字段 | `$.name` |
| `$.field.sub` | 读取嵌套字段 | `$.book.author` |
| `$['field-name']` | 读取包含特殊字符的字段名 | `$['book-name']` |
| `[0]` | 取数组第 1 个元素，JSONPath 索引从 0 开始 | `$.list[0].name` |
| `[*]` | 取数组所有元素 | `$.list[*].name` |
| `[0:3]` | 数组切片，取第 1 到第 3 个元素 | `$.list[0:3]` |
| `$..field` | 递归查找指定字段 | `$..name` |
| `[?()]` | 按条件过滤数组元素 | `$.list[?(@.status == '完结')].name` |

### 标准 JSONPath 示例

以下写法推荐优先掌握：

| 目标 | 示例 |
| --- | --- |
| 取根对象字段 | `$.name` |
| 取嵌套字段 | `$.data.book.author` |
| 取数组列表 | `$.data.list` |
| 取数组第一个元素字段 | `$.data.list[0].name` |
| 取数组所有元素字段 | `$.data.list[*].name` |
| 取对象整体 | `$.data.user` |
| 取数组整体 | `$.data.list` |
| 递归查找所有 `name` 字段 | `$..name` |
| 按状态过滤书籍 | `$.data.list[?(@.status == '完结')]` |
| 按分类过滤书名 | `$.data.list[?(@.category == '玄幻')].name` |

### 列表与相对查询

列表字段会先选中数组，列表项内部字段建议从当前 item 继续查询。

```json
{
  "bookList": "$.data.list",
  "bookName": "$.name",
  "bookAuthor": "$.author",
  "bookUrl": "$.url"
}
```

如果 `bookList` 选中数组后，当前 item 是：

```json
{
  "name": "书名",
  "author": "作者",
  "url": "/book/1"
}
```

则 `bookName` 可写 `$.name`，`bookUrl` 可写 `$.url`。

简单字段名也可以被识别，例如 `name` 会按当前 item 的 `$.name` 查询；但为了规则更清晰，文档示例统一推荐写完整的 `$.name`。

### 返回值说明

JSONPath 规则最终会返回可用于后续字段的值：

| 返回内容 | 说明 |
| --- | --- |
| 字符串 / 数字 | 转成文本使用 |
| 对象 | 转成 JSON 字符串使用，也可在 `put` 后通过 `@get{字段.子字段}` 读取 |
| 数组 | 转成 JSON 数组字符串使用 |
| 多个结果 | 可能按 JSON 数组字符串返回 |
| `@all` | 返回当前 JSON 原文或当前对象内容 |

## 搜索示例

```json
{
  "ruleSearch": {
    "engine": "jsonpath",
    "bookList": "$.data.list",
    "bookName": "$.name",
    "aliasName": "$.aliasName",
    "bookAuthor": "$.author",
    "bookUrl": "$.url",
    "ruleExtra": {
      "coverUrl": "$.cover",
      "introduce": "$.intro"
    }
  }
}
```

`aliasName` 为可选字段，用于从搜索列表项中解析书籍别名。别名为空时不参与搜索匹配；别名非空时，精准匹配和包含匹配都会按 `bookName OR aliasName` 判断。

## 前置请求 put

JSON 响应可以把 `response.engine` 设置为 `jsonpath`，`put` 中的 value 使用 JSONPath 规则：

```json
{
  "response": {
    "engine": "jsonpath",
    "put": {
      "token": "$.token",
      "userName": "$.user.name",
      "info": "$.user.info"
    }
  }
}
```

如果 `info` 是对象，则会直接保存对象，后续可以通过 `@get{info.name}` 读取对象子字段。

## 注意事项

- JSON 响应用 `jsonpath`；HTML 响应应使用 `xpath` 或 `css`。
- `bookList`、`chapterList` 等列表字段通常要指向数组。
- 列表项内部字段建议写成 `$.字段名`，避免和整页根对象查询混淆。
- `@all` 可用于返回当前 JSON 原文或当前对象内容，配合正则或 JS 做进一步处理。
- JSONPath 后可以继续写 `<js>...</js>` 或 `##正则`；组合规则的输入和执行顺序见[字段规则执行流水线](rules-Introduction.md#field-rule-pipeline)。
