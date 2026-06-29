# 🔧 书源调试接口

本页涵盖 6 个书源调试接口：搜索、书籍详情、章节列表、正文、发现、取消请求。所有接口均为 `POST`，请求体参数值统一为字符串类型。

!!! tip "调试响应结构"
    5 个调试接口（搜索/书籍详情/章节列表/正文/发现）的 `data` 字段统一遵循 `ResponsesModel` 结构：`{ resultData, list, jsLog, gets }`。详见 [响应格式说明](response.md#调试响应字段)。

---

## 1. 搜索调试

**接口：** `POST /api/site/search`

**功能：** 测试书源的搜索功能

**请求参数：**

| 参数名称 | 类型 | 必填 | 说明 |
|:---|:---|:---|:---|
| `keyword` | String | 是 | 书名搜索词 |
| `siteJson` | String | 是 | 书源 JSON 规则串 |

!!! note "书籍别名"
    搜索调试会按 V2 规则解析 `siteJson.ruleSearch.aliasName`。该字段为空时不参与匹配；非空时，搜索结果会按书名或别名命中关键词。

**请求示例：**

```http
POST /api/site/search
Content-Type: application/json

{
  "keyword": "斗破苍穹",
  "siteJson": "{\"siteName\":\"示例书源\",\"host\":\"https://example.com\",\"ruleSearch\":{\"url\":\"https://example.com/search?keyword=@{keyword}\",\"bookList\":\"//div[@class='book-item']\",\"bookName\":\".//h3/text()\",\"aliasName\":\".//span[@class='alias']/text()\",\"bookAuthor\":\".//span[@class='author']/text()\",\"bookUrl\":\".//a/@href\"}}"
}
```

**响应示例：**

```json
{
  "code": "200",
  "message": "成功",
  "data": {
    "resultData": {},
    "list": [
      {
        "name": "斗破苍穹",
        "author": "天蚕土豆",
        "bookUrl": "https://example.com/book/123"
      }
    ],
    "jsLog": "搜索日志...",
    "gets": {}
  }
}
```

---

## 2. 章节详情调试

**接口：** `POST /api/site/bookinfo`

**功能：** 测试书源的书籍详情功能

**请求参数：**

| 参数名称 | 类型 | 必填 | 说明 |
|:---|:---|:---|:---|
| `bookJson` | String | 是 | 书籍 JSON（包含 `bookUrl` 字段） |
| `siteJson` | String | 是 | 书源 JSON 规则串 |

**请求示例：**

```http
POST /api/site/bookinfo
Content-Type: application/json

{
  "bookJson": "{\"bookUrl\":\"https://example.com/book/123\",\"name\":\"斗破苍穹\"}",
  "siteJson": "{\"name\":\"示例书源\",\"detail\":{...}}"
}
```

**响应示例：**

```json
{
  "code": "200",
  "message": "成功",
  "data": {
    "resultData": {
      "name": "斗破苍穹",
      "author": "天蚕土豆",
      "intro": "三十年河东，三十年河西...",
      "category": "玄幻",
      "status": "完本"
    },
    "list": [],
    "jsLog": "详情日志...",
    "gets": {}
  }
}
```

---

## 3. 章节列表调试

**接口：** `POST /api/site/chapter`

**功能：** 测试书源的章节列表功能

**请求参数：**

| 参数名称 | 类型 | 必填 | 说明 |
|:---|:---|:---|:---|
| `bookJson` | String | 是 | 书籍 JSON |
| `siteJson` | String | 是 | 书源 JSON 规则串 |

**请求示例：**

```http
POST /api/site/chapter
Content-Type: application/json

{
  "bookJson": "{\"bookUrl\":\"https://example.com/book/123\",\"name\":\"斗破苍穹\"}",
  "siteJson": "{\"name\":\"示例书源\",\"catalog\":{...}}"
}
```

**响应示例：**

```json
{
  "code": "200",
  "message": "成功",
  "data": {
    "resultData": {},
    "list": [
      {
        "name": "第一章 落魄少年",
        "url": "https://example.com/chapter/1"
      },
      {
        "name": "第二章 斗之力等级",
        "url": "https://example.com/chapter/2"
      }
    ],
    "jsLog": "章节日志...",
    "gets": {}
  }
}
```

---

## 4. 正文调试

**接口：** `POST /api/site/content`

**功能：** 测试书源的正文内容功能

**请求参数：**

| 参数名称 | 类型 | 必填 | 说明 |
|:---|:---|:---|:---|
| `catalogJson` | String | 是 | 章节 JSON（包含章节 `url`） |
| `siteJson` | String | 是 | 书源 JSON 规则串 |

**请求示例：**

```http
POST /api/site/content
Content-Type: application/json

{
  "catalogJson": "{\"url\":\"https://example.com/chapter/1\",\"name\":\"第一章\"}",
  "siteJson": "{\"name\":\"示例书源\",\"content\":{...}}"
}
```

**响应示例：**

```json
{
  "code": "200",
  "message": "成功",
  "data": {
    "resultData": "章节正文内容...",
    "list": [],
    "jsLog": "正文日志...",
    "gets": {}
  }
}
```

---

## 5. 发现调试

**接口：** `POST /api/site/finder`

**功能：** 测试书源的发现功能

**请求参数：**

| 参数名称 | 类型 | 必填 | 说明 |
|:---|:---|:---|:---|
| `siteJson` | String | 是 | 书源 JSON 规则串 |
| `categoryUUID` | String | 是 | 分类 UUID |
| `selectList` | String | 是 | 选择筛选项（JSON 字符串，如 `{"cat": "0"}`） |
| `pageIndex` | String | 是 | 页码 |

**请求示例：**

```http
POST /api/site/finder
Content-Type: application/json

{
  "siteJson": "{\"name\":\"示例书源\",\"finder\":{...}}",
  "categoryUUID": "finder_cat_001",
  "selectList": "{\"cat\": \"0\", \"status\": \"1\"}",
  "pageIndex": "1"
}
```

**响应示例：**

```json
{
  "code": "200",
  "message": "成功",
  "data": {
    "resultData": {},
    "list": [
      {
        "name": "书籍名称",
        "author": "作者",
        "bookUrl": "书籍URL"
      }
    ],
    "jsLog": "发现日志...",
    "gets": {}
  }
}
```

---

## 6. 取消调试请求

**接口：** `POST /api/site/cancelRequest`

**功能：** 取消正在进行的调试请求

**请求参数：**

| 参数名称 | 类型 | 必填 | 说明 |
|:---|:---|:---|:---|
| `type` | String | 是 | 请求类型（`"0"`: 搜索，`"1"`: 目录，`"2"`: 正文，`"3"`: 发现） |

**请求示例：**

```http
POST /api/site/cancelRequest
Content-Type: application/json

{
  "type": "0"
}
```

**响应示例：**

```json
{
  "code": "200",
  "message": "已取消",
  "data": true
}
```

---

📅 **更新日期**：2025-10-12
