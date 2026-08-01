# 正文规则

正文规则对应书源 JSON 的 `ruleContent`，负责请求当前章节，解析正文、正文分页和听书播放地址，也包含当前章节的评论页面地址 `commentUrl`。

通用规则语法、请求头继承、变量替换和地址补全见[规则说明概述](rules-Introduction.md)。正文富文本标签另见[正文富文本](../content-richtext/index.md)。

## 输入参数

| 参数 | 说明 | 用例 |
| --- | --- | --- |
| `bookUrl` | 章节列表地址 | `config.bookUrl` |
| `infoUrl` | 书籍详情地址 | `config.infoUrl` |
| `bookName` | 书籍名称 | `config.bookName` |
| `bookAuthor` | 作者 | `config.bookAuthor` |
| `chapterUrl` | 章节规则解析出的正文地址 | `config.chapterUrl` |
| `chapterName` | 当前章节名称 | `config.chapterName` |
| `url` | 当前正文请求地址，首屏初始值等于 `chapterUrl` | `config.url` |
| `pageIndex` | 当前页码 | `config.pageIndex` |

## 规则字段

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `contents` | 是 | 正文内容提取规则 |
| `cleaner` | 否 | 正文净化规则 |
| `playUrl` | 否 | 听书播放地址；为空时使用正文 URL 和正文 Header |
| `page` | 否 | 当前分页规则 |
| `next` | 否 | 下一页地址规则 |
| `commentUrl` | 否 | 当前章节评论页面地址，详见下方专节 |
| `method` / `params` / `header` | 否 | 请求方法、参数和场景请求头 |
| `preRequests` | 否 | 正式正文请求前的前置请求 |
| `request` / `response` | 否 | 请求配置 JS 与响应预处理 JS，只使用 `@js:` |

## 正文分页地址

正文首屏使用 `config.url`。分页时优先使用 `config.chapterUrl`，为空才使用 `config.url`。在 `request @js:` 中需要改变正文分页入口时，应修改 `config.chapterUrl`。

<span id="comment-url"></span>

## commentUrl 章节评论地址

`ruleContent.commentUrl` 用于打开当前章节的评论页面。V2 支持纯 URL、同步或异步 `@js:`，并支持返回 URL 页面或本地 HTML 页面。

!!! info "版本边界"
    本节新增能力仅适用于 V2。V1 行为未修改；原有纯 URL 和同步 `@js:` 返回 URL 的规则继续兼容。

### 支持形式一览

| 形式 | 写法或返回值示例 | 请求头 | 适用场景 |
| --- | --- | --- | --- |
| 直接 URL | `https://example.com/review?id=1` | 使用 `ruleContent.header` | 地址固定，或可通过普通变量替换得到 |
| 同步 JS 返回 URL | `@js:return config.host + '/review';` | 使用 `ruleContent.header` | 需要同步计算评论地址 |
| 异步 JS 返回 URL | `@js:const r = await app.get({...}); return r.url;` | 使用 `ruleContent.header` | 需要先请求接口获取地址 |
| URL + 尾随 Header | `https://example.com/review,{"header":{"X-Token":"abc"}}` | 尾随 `header` 覆盖同名正文 Header | 兼容登录 URL 的字符串配置格式 |
| JS 返回 URL 对象 | `return {url: r.url, header: {'X-Token': r.token}};` | 对象 `header` 覆盖同名正文 Header | 异步获取 URL 和 Token，推荐使用 |
| 直接 `@html:` | `@html:<html><body>评论</body></html>` | 只保证 Cookie | 评论页面可由规则直接生成 |
| `@html:` + 尾随 Header | `@html:<html>...</html>,{"header":{"X-Token":"abc"}}` | Cookie 可写入；非 Cookie Header 不会自动用于子资源 | 与登录 URL 的 HTML 配置格式兼容 |
| JS 返回 HTML 字符串 | `return '@html:<html><body>评论</body></html>';` | 只保证 Cookie | 需要用 JS 动态生成 HTML |
| JS 返回 HTML 对象 | `return {html: '<html>...</html>', baseURL: config.host};` | 只保证 Cookie | 需要明确 HTML 的相对资源基准地址 |

!!! tip "选择建议"
    需要 Authorization、Referer、User-Agent 或自定义 Token 等完整请求头时，优先返回 URL；仅在页面内容可直接生成、且子资源不依赖自定义 Header 时使用 HTML。

### 异步请求

`commentUrl` 以 `@js:` 开头时，V2 会以异步函数执行规则。可以使用 `await app.get(...)` 或 `await app.post(...)` 先请求接口，再返回最终页面。

```js
@js:
const response = await app.post({
  url: app.sp.get('线路') + '/comment/token',
  params: {
    bookUrl: config.bookUrl,
    chapterUrl: config.chapterUrl
  },
  header: {
    'Content-Type': 'application/x-www-form-urlencoded'
  }
});

return response.url;
```

- `app.get`、`app.post` 返回 Promise，异步调用必须使用 `await` 或 Promise 链。
- 默认等待上限为 30 秒。
- 请求失败、规则异常或没有有效返回值时，评论入口按“评论地址无效”处理。
- 读取书源存储值应使用 `app.sp.get('键名')`；V2 不提供全局裸 `getValue(...)`。

### 返回 URL

纯 URL：

```js
return 'https://example.com/review?id=1';
```

URL 加登录 URL 同款尾随请求配置：

```js
return 'https://example.com/review?id=1,{"header":{"Referer":"https://example.com","X-Token":"abc"}}';
```

对象格式：

```js
return {
  url: 'https://example.com/review?id=1',
  header: {
    Referer: 'https://example.com',
    'X-Token': 'abc'
  }
};
```

返回值中的 `header` 会覆盖 `ruleContent.header` 的同名字段，并应用于 WebView 的首个 URLRequest。需要 Authorization、Referer、User-Agent 或自定义 Token 等完整请求头时，应使用 URL 模式。

### 返回 HTML

`@html:` 字符串：

```js
return '@html:<html><body><h1>章节评论</h1></body></html>';
```

`@html:` 也兼容登录 URL 同款尾随配置：

```js
return '@html:<html><body>章节评论</body></html>,{"header":{"X-Token":"abc"}}';
```

对象格式：

```js
return {
  html: '<html><body>章节评论</body></html>',
  baseURL: 'https://example.com'
};
```

未提供 `baseURL` 时使用书源 `host`。

!!! warning "HTML 模式的 Header 边界"
    HTML 通过 `WKWebView.loadHTMLString` 加载。Cookie 会按最终 `baseURL` 的域写入 WebView，但 Authorization、Referer、User-Agent 和自定义 Token 等非 Cookie Header 不会自动附加到 HTML 子资源请求。当前 HTML 模式也不提供登录页的 `window.ParsingBook` openParams bridge。

## 最小正文示例

```json
{
  "ruleContent": {
    "contents": "//div[@id='content']",
    "next": "//a[@rel='next']/@href",
    "commentUrl": "@js:return {url: config.host + '/review', header: {'X-Chapter': config.chapterUrl}};"
  }
}
```
