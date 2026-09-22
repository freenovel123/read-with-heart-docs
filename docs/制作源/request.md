# 请求信息

书源编辑器里的「请求信息」对应规则字段 `request`。它在请求真正发出前执行，用 JS 修改这次请求的配置：动态地址、动态 Header、签名参数、补充 Cookie 等。

!!! note "与字段 JS、响应信息的区别"
    `request` 和 `response` 只使用 `@js:`，不能写 `<js>...</js>`。URL、书名、作者、列表、正文等普通字段需要 JS 时用 `<js>`，参数是 `value` 和 `config`，并会与正则按书写顺序执行，见[字段规则执行流水线](rules-Introduction.md#field-rule-pipeline)。响应侧的入口见[响应信息](response.md)。

## 适用位置

| 规则段 | 字段 | 生效请求 |
| --- | --- | --- |
| `ruleSearch` | `request` | 搜索首屏与搜索分页 |
| `ruleBookInfo` | `request` | 详情首屏 |
| `ruleChapter` | `request` | 章节列表首屏与章节分页 |
| `ruleContent` | `request` | 正文首屏与正文分页 |
| `ruleFinder` | `request` | 当前发现分类的请求 |
| `preRequests[]` | `request` | 该条前置请求自身 |

发现规则的 `request` 写在当前分类的发现规则里。

## 触发条件

字段内容中出现 `@js:` 才会按 JS 执行，并且所有 `@js:` 字面量会在执行前被移除：

```javascript
@js:
config.url = config.url + "&page=" + config.pageIndex;
return config;
```

字段里没有 `@js:` 时 `request` 不生效：写成纯 URL、`@{preUrl1}` 或 `{{...}}` 都不会被求值。要改地址请用场景自己的 `url` 字段，或写进 `@js:`。

## 执行方式

引擎把字段内容包装成下面的函数执行：

```javascript
function jsFun(config) {
  config = JSON.parse(config);
  // 你的 @js: 代码
}
```

| 项目 | 说明 |
| --- | --- |
| `config` | 当前请求配置对象，**已经是对象**，直接读写 `config.url`、`config.header.xxx` |
| 返回值 | 见[返回值与生效规则](#request-return-value) |
| 同步性 | 同步函数，不能用 `await app.get(...)` / `await app.post(...)`；需要先请求接口请放到前置请求，再用 `@get{}` 读值 |
| 超时 | 30 秒，超时按执行失败处理 |
| 公共 JS 库 | 书源顶层 `publicJavascript` 会注入到函数开头，可以直接调用其中定义的函数 |

!!! warning "不要再写一次 JSON.parse"
    `config` 已经是对象，再写 `config = JSON.parse(config)` 会把对象转成 `"[object Object]"` 并抛错，整条规则被静默丢弃，表现为「请求信息没生效」。

## 可读写的 config 字段

| 键 | 说明 |
| --- | --- |
| `config.url` | 当前请求地址，可以写相对路径，之后会按最终 `config.host` 补全 |
| `config.method` | 请求方法 `GET` / `POST` |
| `config.mode` | 请求模式 `http` / `webview` |
| `config.engine` | 解析引擎 `xpath` / `jsonpath` / `css` |
| `config.header` | 请求头字典，最终会用于本次请求 |
| `config.params` | 请求参数字典 |
| `config.cookies` | Cookie 字典，优先级高于登录 Cookie 和 HTTP 缓存 Cookie |
| `config.host` | 站点主地址，修改后会影响随后相对地址的补全 |
| `config.keyword` / `config.pageIndex` | 搜索关键词 / 当前页码 |
| `config.infoUrl` / `config.bookUrl` / `config.chapterUrl` | 场景地址 |
| `config.bookName` / `config.chapterName` | 书籍名称 / 章节名称 |
| `config.openParams` | 开放参数扁平字典，详见 [openParams 开放参数](rules-Introduction.md#openparams) |
| `config.siteIdent` / `config.deviceId` / `config.verifyCode` | 书源标识、设备标识、验证码 |

完整可补全项以编辑器为准，字段含义见[公共内置请求参数](rules-Introduction.md#builtin-request-params)。

<span id="request-return-value"></span>

## 返回值与生效规则

| 返回值 | 结果 |
| --- | --- |
| 对象，或 JSON 字符串 | 与原始 config 合并 |
| 空字符串 / 没有 `return` | 保持原始 config |
| 数字、布尔、数组等其它类型 | 保持原始 config |
| 抛错或语法错误 | 保持原始 config，错误写入 JS 日志 |

合并细则：

- 以原始 config 为底，JS 结果中的**非空**字段覆盖原值。
- 顶层空字符串、`null`、空数组、空对象会被跳过，不会覆盖原值，因此不能靠赋空值删除已有字段。
- 嵌套对象（`header`、`params`、`openParams`）按字段递归合并，只写其中一个键不会清掉其它键。
- 数组字段是**拼接**：原数组 + JS 返回的数组，不是替换；处理数组型字段时需要留意重复。
- JS 结果顶层类型和 config 不一致（例如直接返回数组）时合并失败，整条规则的结果被放弃。

!!! warning "必须 return"
    只修改 `config` 而不返回不会有任何效果：引擎拿不到返回值时会退回原始 config。

## 执行时序

```text
构建初始 config（场景字段 + 公共 header）
  → 解析 params 规则
  → 解析 header 规则
  → 解析 url 规则
  → 执行 request @js:               ← 本页
  → 用最终 config.host 补全 config.url
  → 合并 Cookie（请求显式 Cookie > 登录 Cookie > HTTP 缓存 Cookie）
  → 发出请求
```

由此判断该改哪个字段：

| 目标 | 要改的字段 |
| --- | --- |
| 改首屏请求地址 | `config.url`，可以写相对路径 |
| 改章节列表分页地址 | `config.nextUrl`（为空时才使用 `config.bookUrl`） |
| 改正文分页地址 | `config.chapterUrl` |
| 改请求头 | `config.header` |
| 改请求参数 | `config.params` |
| 改实际访问域名 | `config.host`，随后的地址补全会使用新值 |

分页请求会**重新执行**一次 `request` JS：章节分页和正文分页的每一页都会再跑一次，签名和地址需要按 `config.pageIndex` 自行计算。

## 变量替换

`request` JS 的源码在执行前会先做变量替换，可以像普通规则一样写占位符：

| 写法 | 说明 |
| --- | --- |
| `@get{token}` | 读取前置请求 `response.put` 保存的值 |
| `${keyword}` / `@{keyword}` | 读取 config 字段 |
| `@tools{timestamp()}` | 内置工具函数 |

- 替换值会做 JS 字符串转义（`\`、`"`、换行、制表符），所以占位符写在字符串里是安全的：`config.header["X-Token"] = "@get{token}";`
- `@{}` 取不到值时替换为 `{}`，避免破坏 JS 语法；`${}` 取不到值时保留原文。
- 运行时计算出来的字符串不会再走一遍规则解析，返回值里现拼的 `@{...}` 不会被二次替换。
- Native 方法完整列表见 [Native to Javascript](../native-to-js.md)。

## 案例

### 1. 用书籍 ID 生成详情首屏地址

搜索的 `bookUrl` 只解析出书籍 ID 时，详情首屏地址在 `request` 里生成：

```javascript
// ruleBookInfo.request
@js:
let matched = config.infoUrl.match(/bookid=(\d+)/);
if (!matched) {
  throw new Error("infoUrl 缺少 bookid");
}
config.url = "/api/book/" + matched[1] + "/info";
return config;
```

`config.url` 用相对路径，随后会按 `config.host` 补成完整地址。

### 2. 计算签名参数

```javascript
// ruleSearch.request
@js:
let ts = String(Date.now());
config.params = Object.assign({}, config.params, {
  ts: ts,
  sign: App.md5(config.keyword + ts + "salt")
});
return config;
```

### 3. 注入前置请求拿到的 token

```javascript
// ruleSearch.request
@js:
config.header["Authorization"] = "Bearer @get{token}";
return config;
```

`@get{token}` 在 JS 执行前被替换，值来自前置请求的 `response.put`，写法见[前置请求](pre-request.md)。

### 4. 补充 Cookie

```javascript
// ruleContent.request
@js:
config.cookies["session"] = String(App.sp.get("session"));
config.cookies["t"] = "@tools{timestamp()}";
return config;
```

请求里显式设置的 Cookie 优先级最高，会覆盖同名的登录 Cookie 和 HTTP 缓存 Cookie。

### 5. 切换域名并改相对地址

```javascript
// ruleSearch.request
@js:
config.host = "https://api.example.com";
config.url = "/v2/search?q=" + encodeURIComponent(config.keyword);
return config;
```

### 6. 分页地址

```javascript
// ruleChapter.request
@js:
config.nextUrl = "/catalog?page=" + config.pageIndex;
return config;
```

```javascript
// ruleContent.request
@js:
config.chapterUrl = config.chapterUrl + "&page=" + config.pageIndex;
return config;
```

只改 `config.url` 不会影响分页请求。

### 7. 前置请求里的 request

前置请求的返回值是**整体替换**该前置请求配置，推荐直接 `return config`：

```javascript
// preRequests[0].request
@js:
config.url = config.host + "/init";
config.method = "POST";
config.params = { client: "reader" };
return config;
```

不要只返回 `{ url: "..." }` 这样的部分对象，否则 `host`、`siteIdent`、`header`、`cookies` 会一起变空。

### 8. 完整字段示例

```json
{
  "ruleBookInfo": {
    "url": "https://www.example.com/book/0",
    "request": "@js:\nlet matched = config.infoUrl.match(/bookid=(\\d+)/);\nif (matched) {\n  config.url = '/api/book/' + matched[1] + '/info';\n}\nconfig.header['X-Client'] = 'reader';\nreturn config;",
    "name": "$.data.title"
  }
}
```

## 排错

| 现象 | 常见原因 |
| --- | --- |
| 规则完全没生效 | 字段里没有 `@js:`；或写了 `config = JSON.parse(config)` 导致抛错 |
| 改了 config 但值没变 | 没有 `return`；或返回了数组、数字、布尔等不被支持的类型 |
| 地址没有被补全 | `config.host` 为空，检查书源顶层 `host` |
| 数组字段出现重复内容 | 数组是拼接语义，不是替换 |
| 分页地址没有改变 | 改的是 `config.url`，分页要改 `config.nextUrl` / `config.chapterUrl` |
| 出现 `await` 相关报错 | 请求 JS 是同步函数，异步取数据请放到前置请求 |
| 想删除某个 header 但删不掉 | 合并会跳过空值，`request` JS 不支持删除字段 |
