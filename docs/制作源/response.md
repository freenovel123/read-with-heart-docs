# 响应信息

书源编辑器里的「响应信息」对应规则字段 `response`。它在收到响应之后、字段提取（XPath / JSONPath / CSS）之前执行，用 JS 对原始响应做预处理：解密、解包、去广告、格式转换等。

!!! note "与字段 JS、请求信息的区别"
    `response` 和 `request` 只使用 `@js:`，不能写 `<js>...</js>`。普通字段需要 JS 时用 `<js>`，参数是 `value` 和 `config`，见[字段规则执行流水线](rules-Introduction.md#field-rule-pipeline)。请求侧的入口见[请求信息](request.md)。

## 适用位置与触发条件

| 规则段 | 字段 | 生效请求 |
| --- | --- | --- |
| `ruleSearch` | `response` | 搜索首屏与搜索分页 |
| `ruleBookInfo` | `response` | 详情首屏 |
| `ruleChapter` | `response` | 章节列表首屏与章节分页 |
| `ruleContent` | `response` | 正文首屏与正文分页 |
| `ruleFinder` | `response` | 当前发现分类的请求 |
| `preRequests[]` | `response.respones` | 该条前置请求（字段名按现有协议写作 `respones`） |

字段内容中出现 `@js:` 才会执行。V2 只执行 `@js:` 形式的响应规则：写 XPath、正则或 `{{...}}` 都不会被求值，这些内容会被忽略，响应原样进入后续解析，需要处理请写进 `@js:`。

## 执行方式

引擎把字段内容包装成下面的函数执行：

```javascript
function jsFun(html, config) {
  let document = null;
  if (html && !app.isJson(html)) {
    document = app.doc(html);
  }
  config = JSON.parse(config);
  // 你的 @js: 代码
}
```

| 参数 | 说明 |
| --- | --- |
| `html` | 原始响应正文（HTML 或 JSON 字符串），就是后续要解析的那份内容 |
| `config` | 当前请求配置对象，**已经是对象**，不要再 `JSON.parse` |
| `document` | 仅当 `html` 非空且不是合法 JSON 时可用；JSON 响应或空响应下是 `null` |

- 同步函数，不能用 `await app.get(...)` / `await app.post(...)`。
- 超时 30 秒，书源顶层 `publicJavascript` 会注入到函数开头。
- 必须 `return`：只修改 `html` 变量而不返回不会有任何效果。

### document 的可用方法

| 方法 | 说明 |
| --- | --- |
| `document.select(selector)` | CSS 选择器查找，返回数组，可用 `first()` / `last()` 取首尾 |
| `document.text()` | 元素文本 |
| `document.title()` | 页面标题 |
| `document.attr(name)` | 元素属性 |

`document` 是按原始 `html` 创建的，之后修改 `html` 变量不会同步到 `document`。

这是 `v2.6.0` 起提供的 `Document` 子集，不是完整 DOM API，也不是浏览器写法；需要更复杂的能力时请放在前置请求或 `publicJavascript` 里实现。

## 返回值

| 返回值 | 正式场景（搜索 / 详情 / 章节 / 正文 / 发现，含分页） | 前置请求 `respones` |
| --- | --- | --- |
| 字符串 | 作为新的响应内容 | 作为新的响应内容 |
| 对象 / 数组 | 转成 JSON 字符串，可继续用 JSONPath 解析 | 不支持，回退原始响应 |
| 数字 / 布尔 | 转成字符串 | 不支持，回退原始响应 |
| 空字符串 / 没有 `return` | 回退原始响应 | 回退原始响应 |
| 抛错 / 语法错误 | 回退原始响应，错误写入 JS 日志 | 回退原始响应 |

- 处理结果会替换后续字段解析的输入内容；同一份内容也会显示在调试页的「响应信息处理完成」。
- 失败不会中断场景：语法错误、`throw`、超时都只是退回原始响应，所以「规则没生效」时先看 JS 日志。

## 变量替换

响应 JS 的源码在执行前会先做变量替换，`@get{}`、`${}`、`@{}`、`@tools{}` 都可以用。

!!! warning "响应 JS 的替换不做 JS 转义"
    请求 JS 的替换值会做 JS 字符串转义，响应 JS 不会。值里带英文引号或换行时可能直接破坏 JS 源码，结果是静默回退原始响应。优先在 JS 里读 `config.*` 或 `App.sp.get(...)`。

正式场景还支持 `{{...}}` 占位：进入 JS 前按当前引擎把 `{{...}}` 替换成提取结果（`xpath` / `css` 使用解析后的文档，`jsonpath` 使用原始响应字符串）。

```javascript
@js:
return "{{//h1[@class='title']/text()}}";
```

前置请求的 `respones` 没有 `{{}}` 替换。

## 案例

### 1. 去掉 script 和广告节点再解析

```javascript
@js:
html = html.replace(/<script[\s\S]*?<\/script>/gi, "");
html = html.replace(/<div class="ad"[\s\S]*?<\/div>/gi, "");
return html;
```

### 2. JSON 响应解包

```javascript
// 接口返回 {"code":0,"result":{"html":"..."}}
@js:
let payload = JSON.parse(html);
if (payload.code !== 0) {
  throw new Error("接口返回失败: " + payload.code);
}
return payload.result.html;
```

### 3. 解密后再解析

```javascript
// 接口返回 {"data":"<base64 密文>"}，AES-CBC/PKCS7
@js:
let payload = JSON.parse(html);
return App.aes.decrypt(payload.data, "0123456789abcdef", "0123456789abcdef");
```

### 4. 返回对象，交给 JSONPath

```javascript
@js:
let payload = JSON.parse(html);
return { list: payload.data.rows, total: payload.data.count };
```

之后 `bookList` 可以写 `$.list`。对象 / 数组返回值只有正式场景支持，前置请求的 `respones` 需要自己 `JSON.stringify` 后再返回。

### 5. 用 document 读取并重组内容

```javascript
@js:
let links = "";
let items = document.select("ul.chapter li a");
for (let i = 0; i < items.length; i++) {
  links += items[i].attr("href") + "\n";
}
return links;
```

### 6. 用变量拼接改写响应

```javascript
@js:
// 把响应里的相对图片地址补成绝对地址
return html.replace(/src="\/([^"]+)"/g, 'src="' + config.host + '/$1"');
```

### 7. 正式场景里的 `{{}}` 占位

```javascript
@js:
// 先用 XPath 取到标题，再包成 JSON 交给 JSONPath 解析
return JSON.stringify({ title: "{{//h1[@class='title']/text()}}" });
```

占位结果会原样插入，内容里带引号或换行时需要自行处理转义。

### 8. 前置请求 respones 解密后交给 put

```json
{
  "url": "${host}/api/init",
  "response": {
    "engine": "jsonpath",
    "respones": "@js:\nlet payload = App.aes.decrypt(html, \"0123456789abcdef\", \"0123456789abcdef\");\nreturn payload;",
    "put": {
      "token": "$.token"
    }
  }
}
```

## 排错

| 现象 | 常见原因 |
| --- | --- |
| 响应没有变化 | 字段里没有 `@js:`；或只改了 `html` 没有 `return` |
| JSON 响应里 `document` 是 `null` | JSON 不创建 `document`，请用 `JSON.parse(html)` 自己处理 |
| 规则偶尔生效、偶尔不生效 | 替换值里带引号或换行破坏了 JS 源码，或接口偶发返回非预期结构 |
| 返回对象后解析不到 | 前置请求的 `respones` 只接受字符串，需要 `JSON.stringify` |
| 返回 `{{...}}` 内容后报语法错误 | 占位结果含引号或换行，需要转义后再拼进 JS 字符串 |
| 调试页「响应信息处理完成」没变化 | 规则没有执行成功，回退到了原始响应，检查 JS 日志 |
