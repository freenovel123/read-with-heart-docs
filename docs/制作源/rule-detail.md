# 详情规则

详情规则对应书源 JSON 的 `ruleBookInfo`，负责请求书籍详情页，补全书名、作者、简介、封面等信息，解析章节列表入口，并可通过 `toolsUrl` 提供书籍工具页面。

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
| `toolsUrl` | V2 书籍工具页面地址；规则非空时显示在详情页右上角添加菜单中，详见下方专节 |
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

<span id="tools-url"></span>

## toolsUrl 书籍工具地址

`ruleBookInfo.toolsUrl` 用于在书籍详情页提供与当前书籍相关的网页工具，例如角色列表、插图目录、书籍数据面板或站点扩展功能。规则非空时，详情页右上角添加菜单中会显示“书籍工具”；点击后通过 App 内置 WebView 弹层打开页面。

!!! info "版本边界"
    本节能力仅适用于 V2。旧书源缺少 `toolsUrl` 时按空字符串处理，添加菜单中不显示“书籍工具”；V1 详情解析流程未增加工具入口。

### 可用的详情参数

`toolsUrl` 使用详情场景的 config，不会伪造正文或章节上下文。常用参数如下：

| 参数 | 说明 |
| --- | --- |
| `config.infoUrl` | 搜索或发现阶段得到的书籍详情地址或书籍 ID |
| `config.url` | 本次详情请求地址，初始值等于 `infoUrl` |
| `config.bookName` | 当前书名 |
| `config.bookAuthor` | 当前作者 |
| `config.host` | 书源主地址 |
| `config.params` | `ruleBookInfo.params` 详情请求参数 |
| `config.openParams` | 当前生效的书源开放参数 |

`toolsUrl` 不提供 `config.chapterUrl`。如果工具页面必须依赖某一章节，应改用正文的 [`commentUrl`](rule-content.md#comment-url)，或由工具页面自行查询章节数据。

### 支持形式一览

| 形式 | 写法或返回值示例 | 请求头 | 适用场景 |
| --- | --- | --- | --- |
| 直接 URL | `https://example.com/tools?id=1` | 使用 `ruleBookInfo.header` | 地址固定，或可通过普通变量替换得到 |
| 相对 URL | `/book/tools?id=1` | 使用 `ruleBookInfo.header` | 自动按书源 `host` 补全地址 |
| 同步 JS 返回 URL | `@js:return config.infoUrl + '/tools';` | 使用 `ruleBookInfo.header` | 需要同步计算工具地址 |
| 异步 JS 返回 URL | `@js:const r = await app.get({...}); return r.url;` | 使用 `ruleBookInfo.header` | 需要先请求接口获取地址 |
| URL + 尾随 Header | `https://example.com/tools,{"header":{"X-Token":"abc"}}` | 尾随 `header` 覆盖同名详情 Header | 兼容登录 URL 的字符串配置格式 |
| JS 返回 URL 对象 | `return {url: r.url, header: {'X-Token': r.token}};` | 对象 `header` 覆盖同名详情 Header | 异步获取 URL 和 Token，推荐使用 |
| 直接 `@html:` | `@html:<html><body>书籍工具</body></html>` | 只保证 Cookie | 页面可由规则直接生成 |
| `@html:` + 尾随 Header | `@html:<html>...</html>,{"header":{"X-Token":"abc"}}` | Cookie 可写入；非 Cookie Header 不会自动用于子资源 | 与登录 URL 的 HTML 配置格式兼容 |
| JS 返回 HTML 字符串 | `return '@html:<html><body>书籍工具</body></html>';` | 只保证 Cookie | 需要用 JS 动态生成 HTML |
| JS 返回 HTML 对象 | `return {html: '<html>...</html>', baseURL: config.host};` | 只保证 Cookie | 需要明确相对资源的基准地址 |

!!! tip "选择建议"
    需要 Authorization、Referer、User-Agent 或自定义 Token 等完整请求头时，优先返回 URL；仅在页面内容可直接生成、且子资源不依赖自定义 Header 时使用 HTML。

### 异步请求

`toolsUrl` 以 `@js:` 开头时，V2 会以异步函数执行规则。可以使用 `await app.get(...)` 或 `await app.post(...)` 先请求接口，再返回最终页面。

```js
@js:
const response = await app.post({
  url: config.host + '/api/book/tools',
  params: {
    infoUrl: config.infoUrl,
    bookName: config.bookName
  },
  header: {
    'Content-Type': 'application/x-www-form-urlencoded'
  }
});

return {
  url: response.url,
  header: {
    'X-Token': response.token
  }
};
```

- `app.get`、`app.post` 返回 Promise，异步调用必须使用 `await` 或 Promise 链。
- 默认等待上限为 30 秒。
- 请求失败、规则异常、返回空值或 URL 无效时，App 提示“工具地址无效”。
- 读取书源存储值应使用 `app.sp.get('键名')`；V2 不提供全局裸 `getValue(...)`。

### 返回 URL

纯 URL：

```js
return config.host + '/book/tools?id=' + encodeURIComponent(config.infoUrl);
```

URL 加尾随请求配置：

```js
return 'https://example.com/tools?id=1,{"header":{"Referer":"https://example.com","X-Token":"abc"}}';
```

对象格式：

```js
return {
  url: 'https://example.com/tools?id=1',
  header: {
    Referer: 'https://example.com',
    'X-Token': 'abc'
  }
};
```

返回值中的 `header` 会覆盖 `ruleBookInfo.header` 的同名字段，并应用于 WebView 的首个 URLRequest。详情 Header 和公共 JS 中的 `@get{}`、`${}`、`@{}`、`@tools{}` 会在执行工具规则前完成变量替换。

### 返回 HTML

`@html:` 字符串：

```js
return '@html:<html><body><h1>' + config.bookName + '</h1></body></html>';
```

`@html:` 也兼容尾随配置：

```js
return '@html:<html><body>书籍工具</body></html>,{"header":{"X-Token":"abc"}}';
```

对象格式：

```js
return {
  html: '<html><body><h1>书籍工具</h1></body></html>',
  baseURL: config.host
};
```

未提供 `baseURL` 时使用书源 `host`。

!!! warning "HTML 模式的 Header 边界"
    HTML 通过 `WKWebView.loadHTMLString` 加载。Cookie 会按最终 `baseURL` 的域写入 WebView，但 Authorization、Referer、User-Agent 和自定义 Token 等非 Cookie Header 不会自动附加到 HTML 子资源请求。当前 HTML 模式也不提供登录页的 `window.ParsingBook` openParams bridge。

## JS 规则边界

详情普通字段的 JS 后处理使用 `<js>...</js>`；`request` 与 `response` 是两个独立入口，只使用 `@js:`。如果详情规则整体为空，App 会保留搜索或发现阶段已有的书籍信息并继续后续流程。

## 最小示例

```json
{
  "ruleBookInfo": {
    "chapterListUrl": "//a[@id='chapters']/@href",
    "toolsUrl": "@js:return {url: config.host + '/book/tools', header: {'X-Book': config.infoUrl}};",
    "bookName": "//h1/text()",
    "bookAuthor": "//span[@class='author']/text()",
    "ruleExtra": {
      "coverUrl": "//img[@class='cover']/@src",
      "introduce": "//div[@class='intro']/text()"
    }
  }
}
```
