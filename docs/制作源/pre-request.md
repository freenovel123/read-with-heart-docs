# 前置请求

前置请求是在正式请求之前执行的预处理请求链。它常用于先获取 token、session、验证码或过盾 Cookie，再把这些结果交给搜索、详情、章节列表、正文或发现请求继续使用。

前置请求的核心是 `response.put`：从前置请求的响应中提取数据并保存，后续规则通过 `@get{}` 读取这些数据。

## 适用位置与基本结构

`preRequests` 是一个数组，可配置在以下规则段中：

| 规则段 | 使用场景 |
| --- | --- |
| `ruleSearch` | 搜索前先获取加密参数、token、Cookie |
| `ruleBookInfo` | 详情页请求前先获取入口参数 |
| `ruleChapter` | 章节列表请求前先获取章节列表接口参数 |
| `ruleContent` | 正文请求前先获取章节正文、播放地址或校验参数 |
| `ruleFinder` | 发现页请求前先获取筛选接口所需参数 |

最小结构示例：

```json
{
  "ruleSearch": {
    "url": "https://api.example.com/search?keyword=${keyword}",
    "header": {
      "Authorization": "Bearer @get{token}"
    },
    "preRequests": [
      {
        "url": "https://api.example.com/token",
        "method": "POST",
        "params": {
          "client": "app"
        },
        "response": {
          "engine": "jsonpath",
          "put": {
            "token": "$.data.token"
          }
        }
      }
    ],
    "bookList": "$.data.books",
    "bookName": "$.name",
    "bookUrl": "$.url"
  }
}
```

执行结果：

- 正式搜索请求执行前，先请求 `https://api.example.com/token`
- 从响应中用 `$.data.token` 提取 `token`
- 正式请求头里的 `@get{token}` 会替换为前置请求保存的 token

## 字段说明

### 前置请求字段

| 字段 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `url` | String | 必填 | 前置请求地址，支持 `${}`、`@{}`、`@get{}`、`<js>`、正则替换等规则语法。 |
| `method` | String | `GET` | 请求方式，支持 `GET`、`POST`。 |
| `mode` | String | `http` | 请求模式，常用 `http`；需要 WebView 加载时可使用 `webview`。 |
| `header` | Object | 空 | 请求头，value 可使用规则语法。 |
| `params` | Object | 空 | 请求参数，value 可使用规则语法。 |
| `request` | String | 空 | 请求前处理规则，可用 `@js:` 动态修改请求配置。 |
| `response` | Object | 空 | 响应处理配置，见下方 `response` 字段。 |
| `preRequestType` | Number | `0` | 前置请求类型：`0` 常规请求，`1` 浏览器过盾，`2` 图片验证码。 |
| `requestEncode` | String | `utf-8` | 请求编码，常用 `utf-8`、`gbk`。 |
| `responseEncode` | String | `utf-8` | 响应编码，常用 `utf-8`、`gbk`。 |
| `forbidCookie` | Boolean | `false` | 是否禁用 Cookie。 |
| `forbidSSL` | Boolean | `false` | 是否忽略 SSL 校验。 |
| `forbidRedirect` | Boolean | `false` | 是否禁止重定向。 |
| `filterErrorCodes` | Array | 空 | 要过滤的 HTTP 状态码列表，只作用于 HTTP 状态码。 |
| `domains` | String / Array | 空 | 额外 Cookie 收集域名范围，主要用于登录或过盾相关场景。 |

### response 字段

| 字段 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `engine` | String | `xpath` | `put` 解析使用的引擎，支持 `xpath`、`jsonpath`、`css`。 |
| `respones` | String | 空 | 响应 JS 处理规则。注意字段名按现有协议写作 `respones`。 |
| `put` | Object | 空 | 数据提取字典，key 是保存名，value 是提取规则。 |

`put` 的 value 会按 `engine` 解析：

```json
{
  "response": {
    "engine": "jsonpath",
    "put": {
      "token": "$.token",
      "info": "$.user.info"
    }
  }
}
```

如果 `info` 是对象，后续可以通过 `@get{info.name}` 读取对象子字段。

## 执行流程

前置请求会在正式请求前按数组顺序执行：

```text
初始化可用参数
  -> 执行第 1 个 preRequest
  -> 保存 put 数据与 preUrl1/preResHeader1/preRepHeader1
  -> 执行第 2 个 preRequest
  -> 保存 put 数据与 preUrl2/preResHeader2/preRepHeader2
  -> ...
  -> 执行正式请求
```

自动保存的 key：

| key | 说明 |
| --- | --- |
| `preUrlN` | 第 N 个前置请求最终请求地址，N 从 1 开始。 |
| `preResHeaderN` | 第 N 个前置请求的请求头。 |
| `preRepHeaderN` | 第 N 个前置请求的响应头。 |

示例：

```text
@get{preUrl1}
@get{preRepHeader1.contentType}
```

多个前置请求之间会继承上一个请求的请求头、Cookie 和配置上下文。第 2 个及后续前置请求可以直接读取前面 `put` 保存的数据：

```json
{
  "preRequests": [
    {
      "url": "${host}/api/init",
      "response": {
        "engine": "jsonpath",
        "put": {
          "token": "$.token"
        }
      }
    },
    {
      "url": "${host}/api/detail?token=@get{token}",
      "header": {
        "Authorization": "Bearer @get{token}"
      },
      "response": {
        "engine": "jsonpath",
        "put": {
          "detailUrl": "$.data.url"
        }
      }
    }
  ]
}
```

### Header 与登录 Cookie

前置请求的 Header 继承关系：

1. **第一个前置请求有自己的 `header`**：使用该前置请求 Header。
2. **第一个前置请求没有 `header`**：继承书源顶层公共 `header`。
3. **后续前置请求有自己的 `header`**：使用当前前置请求 Header，并更新后续请求继承的请求上下文。
4. **后续前置请求没有 `header`**：继续继承上一个前置请求的 Header 上下文。
5. **前置请求结束后进入正式请求**：正式请求仍按自己的场景 Header 规则选择，前置请求 Header 仅作为没有同名字段时的补充上下文。

公共 Header 不会逐字段补入一个已经配置自身 Header 的前置请求。完整的场景继承关系见[请求头继承关系列表](rules-Introduction.md#request-header-inheritance)。

只要 `forbidCookie` 为 `false`，登录流程保存的 `loginCookies` 会自动合并到前置请求，后续前置请求继续继承前一步产生的 Cookie。冲突优先级为“请求显式 Cookie > 登录 Cookie > HTTP Cookie 缓存”。

`forbidCookie: true` 会关闭登录 Cookie 和缓存 Cookie 的自动注入。

## 三类典型前置请求

### 常规请求

常规请求用于先访问接口，提取 token、session、签名参数等信息。

```json
{
  "url": "${host}/api/token",
  "method": "POST",
  "params": {
    "client": "reader"
  },
  "response": {
    "engine": "jsonpath",
    "put": {
      "token": "$.access_token",
      "sessionId": "$.session_id"
    }
  }
}
```

正式请求中读取：

```json
{
  "header": {
    "Authorization": "Bearer @get{token}",
    "Cookie": "session_id=@get{sessionId}"
  }
}
```

### 浏览器过盾

`preRequestType: 1` 用于处理需要浏览器完成验证的站点。验证完成后，系统会自动保存：

| key | 说明 |
| --- | --- |
| `verifyHeader` | 过盾后的请求头 |
| `verifyCookies` | 过盾后的 Cookie |

```json
{
  "url": "https://protected.example.com/",
  "preRequestType": 1,
  "response": {
    "engine": "xpath"
  }
}
```

!!! warning "注意"
    `verifyHeader` 和 `verifyCookies` 是系统自动保存的 key，规则作者不需要、也不建议主动覆写。只有过盾流程完成后，它们才会有值。

### 图片验证码

`preRequestType: 2` 用于图片验证码。系统会弹出验证码输入界面，用户输入的验证码会注入到 `verifyCode`，后续请求可直接读取。

```json
{
  "url": "${host}/captcha",
  "preRequestType": 2
}
```

后续请求使用：

```json
{
  "url": "${host}/api/search?keyword=${keyword}&code=${verifyCode}"
}
```

## 完整示例

### 搜索前获取 token

```json
{
  "ruleSearch": {
    "url": "${host}/api/search?keyword=${keyword}",
    "header": {
      "Authorization": "Bearer @get{token}"
    },
    "preRequests": [
      {
        "url": "${host}/api/token",
        "method": "POST",
        "params": {
          "client": "reader"
        },
        "response": {
          "engine": "jsonpath",
          "put": {
            "token": "$.data.token"
          }
        }
      }
    ],
    "engine": "jsonpath",
    "bookList": "$.data.books",
    "bookName": "$.title",
    "bookUrl": "$.url"
  }
}
```

### 正文前置请求获取章节内容

```json
{
  "ruleContent": {
    "url": "@get{preUrl2}",
    "preRequests": [
      {
        "url": "${host}/api/init",
        "response": {
          "engine": "jsonpath",
          "put": {
            "token": "$.data.token"
          }
        }
      },
      {
        "url": "${host}/api/chapter?id=${chapterUrl}&token=@get{token}",
        "header": {
          "Authorization": "Bearer @get{token}"
        },
        "response": {
          "engine": "jsonpath",
          "put": {
            "content": "$.data.content"
          }
        }
      }
    ],
    "contents": "@get{content}"
  }
}
```

### 先用 respones 转换响应再 put

有些接口返回的数据需要先用 JS 解密、解包或整理，再交给 `put` 提取。

```json
{
  "url": "${host}/api/init",
  "response": {
    "engine": "jsonpath",
    "respones": "@js:\nlet data = JSON.parse(html);\nreturn JSON.stringify(data.result);",
    "put": {
      "token": "$.token",
      "apiBase": "$.api.base"
    }
  }
}
```

### 读取自动保存的请求信息

```text
@get{preUrl1}
@get{preResHeader1.userAgent}
@get{preRepHeader1.contentType}
```

## PUT 存储案例

### XPath 规则获取

```json
{
  "name": "//*[@class='name']/text()",
  "test": "//*[@class='text']/text()"
}
```

### JSONPath 规则获取

```json
{
  "info": "$.info"
}
```

### CSS 规则获取

HTML 响应也可以把 `response.engine` 设置为 `css`，`put` 中的 value 使用 CSS selector 解析。

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

如果 `info` 是对象，则会直接保存对象，后续可读取子路径：

```text
@get{info.name}
@get{info.avatar}
```

## 正式请求读取前置结果

可以在 JS 中读取：

```javascript
let name = "@get{name}";
let test = "@get{test}";
let userName = "@get{info.name}";
```

也可以直接在规则字段中读取：

```text
http://www.example.com/search?keyword=@get{name}
http://www.example.com/search?keyword=@get{info.name}
```

`@get{}` 支持对象子路径，也支持递归 key 解析，例如：

```text
@get{@{bookName}.url}
```

## 排错与注意事项

- `@get{}` 读取为空时，先检查前置请求是否成功、`response.put` 是否配置、key 名是否一致。
- JSON 响应用 `jsonpath`，HTML 响应可用 `xpath` 或 `css`；引擎选错时，`put` 往往会提取为空。
- `filterErrorCodes` 只在 HTTP 状态码层生效，不会吞掉 JS 语法错误、解析失败或请求取消等非 HTTP 错误。
- `domains` 主要用于登录与过盾相关的额外 Cookie 收集域名范围，不要当成所有请求通用的独立能力。
- 自定义 `put` key 应避免和 `verifyHeader`、`verifyCookies`、`preUrlN`、`preResHeaderN`、`preRepHeaderN` 等内置 key 冲突。
- `@put{}` / `response.put` 是前置请求数据传递能力，不要把它理解成通用运行时 KV。正式请求应通过 `@get{}` 读取前置结果。
- 前置请求中的图片验证码和浏览器过盾可能需要用户操作；用户取消时，请求链会终止。
