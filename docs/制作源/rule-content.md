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
| `playUrl` | 否 | 听书播放地址规则；为空时使用正文 URL 和正文 Header，详见下方 [playUrl](#play-url) |
| `page` | 否 | 当前分页规则 |
| `next` | 否 | 下一页地址规则 |
| `commentUrl` | 否 | 当前章节评论页面地址，详见下方专节 |
| `method` / `params` / `header` | 否 | 请求方法、参数和场景请求头 |
| `preRequests` | 否 | 正式正文请求前的前置请求 |
| `request` / `response` | 否 | 请求配置 JS 与响应预处理 JS，只使用 `@js:`，详见[请求信息](request.md)、[响应信息](response.md) |

## 正文分页地址

正文首屏使用 `config.url`。分页时优先使用 `config.chapterUrl`，为空才使用 `config.url`。在 `request @js:` 中需要改变正文分页入口时，应修改 `config.chapterUrl`。

<span id="play-url"></span>

## playUrl 听书播放地址

`ruleContent.playUrl` 用于取得已有章节音频的地址，不负责把文本合成为语音。规则字段本身仍为字符串，App 使用规则的**执行结果**构造音频请求。

- 普通结果可以是纯 URL，或带尾随 Header 的 `URL,{"header":{...}}` 字符串，原有写法保持兼容。
- `playUrl` 为空时，使用正文地址与正文请求 Header。
- 完整对象返回值可携带音频 Header 与 CENC 解密描述；该对象必须写在 `ruleContent.playUrl` 中，格式和适用范围见下节。

普通音频地址示例：

```javascript
@js:
return 'https://example.com/audio/chapter-1.mp3';
```

需要显式指定音频请求头时：

```javascript
@js:
return 'https://example.com/audio/chapter-1.mp3,' + JSON.stringify({
  header: { Referer: 'https://example.com/' }
});
```

!!! warning "不要套用 commentUrl 的异步语义"
    当前 `playUrl @js:` 作为同步字段函数执行，不能直接使用顶层 `await`。本页的 `commentUrl` 有独立的异步入口，两者不能混用。需要额外网络请求取得音频地址或密钥时，先使用正文请求或[前置请求](pre-request.md)准备数据，再在 `playUrl` 中同步提取。

<span id="play-url-cenc"></span>

### CENC 加密章节音频

!!! info "可用状态"
    截至 2026-09-21，本节所述 CENC 音频解密功能尚未发布。使用前请确认安装的 App 版本已支持该功能。

如果音频接口返回的是 CENC 加密 MP4，不能将密文地址直接交给普通播放器。规则应返回完整对象，让 App 完整下载、解密并验证后再播放。

#### 返回契约

!!! warning "必须写在 playUrl"
    解密描述只能作为 `ruleContent.playUrl` 的**执行结果**返回。App 只在这一个入口识别该对象；写在其它规则字段、其它场景或自定义 TTS 里均不会生效。

| 返回位置 | 是否识别对象与 `decrypt` |
| --- | --- |
| `ruleContent.playUrl` 的执行结果 | 是；必须整体返回对象 |
| `ruleContent.playUrl` 返回纯 URL 或尾随 Header | 否；按普通音频地址处理，不解密 |
| `playUrl` 为空（使用正文地址） | 否 |
| `contents`、`next`、`page`、`commentUrl` 等其它字段 | 否 |
| 搜索、目录、详情等其它场景的返回值 | 否 |
| 自定义 TTS 返回值 | 否；返回 `decrypt` 会被明确拒绝 |

该对象必须是完整的 JSON 对象：返回值去掉首尾空白后**以 `{` 开头**才会进入结构化解析。否则整段按普通播放地址处理，即使里面包含 `decrypt` 也不会解密，而是把密文交给播放器。

```javascript
@js:
// 假设当前正文接口返回 JSON，包含 url、key 和可选 kid。
const audio = JSON.parse(value);
return {
  url: audio.url,
  header: {
    Referer: 'https://example.com/'
  },
  decrypt: {
    type: 'cenc-aes-ctr',
    key: audio.key,
    kid: audio.kid
  }
};
```

示例中的 `value` 是传入该字段 JS 的当前响应内容；若接口有嵌套字段，应按真实响应调整提取表达式。不要把示例原样当作适用于任意接口的规则，也不要将真实密钥写进日志。

对象中的 `url` 与 `header` 会同时决定音频下载地址与请求头：

- `url` 支持相对地址，以书源 `host` 补全；不是 HTTP(S) 地址时直接失败。
- 不返回对象时（纯 URL 或尾随 Header），只能指定地址与 Header，无法携带 `decrypt`。

| 字段 | 必填 | 说明 |
| --- | --- | --- |
| `url` | 是 | HTTP(S) 音频地址；相对地址以书源 `host` 补全 |
| `header` | 否 | 音频请求头，必须是字符串字典，键和值不得含回车换行 |
| `decrypt` | 否 | 省略或 `null` 表示普通音频；非空时必须为对象 |
| `decrypt.type` | 是 | 仅支持 `cenc-aes-ctr`，对应 MP4 容器的 `cenc` 保护方案 |
| `decrypt.key` | 是 | 32 位十六进制 AES-128 密钥 |
| `decrypt.kid` | 否 | 非空时须为 32 位十六进制；提供后会核对容器中的 KID |

!!! warning "完整对象与 Header 边界"
    对象中的 `header` 是音频下载使用的请求头，不自动合并 `ruleContent.header`。需要的 Referer、Authorization 或 Cookie 应显式包含在 `header` 中。不要把 `commentUrl` 的 Header 合并行为套用到这里。

- `decrypt` 是 **playUrl 执行结果**的字段，不是书源顶层或 `ruleContent` 的独立配置项。
- 加密配置必须使用完整对象；不要使用 `URL,{"decrypt":...}` 尾随形式。
- 新对象契约不提供 TTS 的 `method`、`params`、`cookie` 字段，音频下载使用 GET。若需要 Cookie，请使用 `header.Cookie`。
- 对象中的 `domains`、`kidHex` 不在新契约内。普通旧 URL 尾随配置的既有行为不因此改变。
- 不带 `decrypt` 的完整对象也可返回普通 URL/Header；`decrypt` 类型或内容非法时直接失败，不会降级播放密文。

#### 播放和离线行为

App 的处理顺序是：

```text
解析章节请求 → 完整下载 → CENC 解密与 MP4 重封装
→ 完整音轨解码验证 → 取消检查 → 原子发布明文缓存 → 本地播放
```

- 首次播放需等待完整下载和验证，不支持密文边下载边播放。
- 在线播放与离线下载共用准备服务；只有解密、验证和缓存发布成功才算离线完成。
- 已准备的缓存优先用于播放，重启后也可离线使用；普通密文缓存不能替代它。
- 下载、解密或验证失败不会把未验证结果交给播放器。切章、停止或取消后，旧请求不能启动旧章节或停止新章节。
- 播放进度使用稳定章节身份，不使用临时文件路径；听书缓存统计和清理包含解密后的文件。
- 缓存文件是**明文音频**，位于系统缓存目录并排除备份。清除缓存后，若离线且无法重新取得动态密钥，就不能重新准备音频。
- URL、Header 或解密参数重新解析后发生变化，会形成新的请求指纹；已命中的缓存不会仅因普通播放启动而强制重新获取签名地址。

#### 限制与排错

| 情况 | 当前行为或建议 |
| --- | --- |
| 对象缺少 `url`、地址为空或不是 HTTP(S) | 章节解析失败；不会退回普通地址，也不会继续播放 |
| `header` 不是字符串字典，或键值含回车换行 | 解析失败；改用字符串值并去掉换行 |
| `decrypt` 不是对象，`type`/`key` 不是字符串 | 解析失败；不要返回数组、布尔值或数字 |
| `key`/`kid` 不是 32 位十六进制 | 解析失败；检查接口字段是否取错 |
| 返回纯 URL 或尾随 Header，但接口实际返回密文 | 不解密；密文会被交给播放器并播放失败，此时改用完整对象 |
| 保护方案不是 `cenc` | 不支持；不会自动尝试 CBCS/CBC1 等其它方案 |
| key/kid 格式错误、KID 不匹配 | 准备失败；检查字段类型和当前章节对应参数 |
| 损坏容器、非音频响应、多个音轨或无法完整解码 | 验证失败，不发布为可播放缓存 |
| 文件过大或结构过于复杂 | 当前保留 128 MiB 文件、250,000 个样本、1,000,000 个子样本上限 |
| 验证耗时过长 | 完整音轨解码有 120 秒协作期限；不保证任意长章节都可处理 |
| TTS 返回 `decrypt` | 明确拒绝；将已有章节音频配置迁到本节听书入口 |

完整解码检查用于确认输出可解码，不构成密码学真实性认证。实现仍有整文件内存开销；长章节、后台挂起及真实服务兼容性需设备验证。

#### 密钥与调试

运行时解密描述不随章节 JSON 写入数据库或缓存回执；章节仅保留请求指纹和安全的播放信息。相关日志、JS 调试输出及技术诊断对密钥字段脱敏。**不要在 URL 或 Header 中重复附加解密密钥，也不要把动态密钥写入源存储或手动日志。** 若作者把常量密钥直接写入书源脚本，该脚本仍属于其源文件内容，不能据此认为设备上没有密钥。

<span id="comment-url"></span>

## commentUrl 章节评论地址

`ruleContent.commentUrl` 用于打开当前章节的评论页面，支持纯 URL、同步或异步 `@js:`，并支持返回 URL 页面或本地 HTML 页面。原有纯 URL 和同步 `@js:` 返回 URL 的规则继续兼容。

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

`commentUrl` 以 `@js:` 开头时，会以异步函数执行规则。可以使用 `await app.get(...)` 或 `await app.post(...)` 先请求接口，再返回最终页面。

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
- 读取书源存储值应使用 `app.sp.get('键名')`；不提供全局裸 `getValue(...)`。

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
