# 📋 书源管理接口

本页涵盖 6 个书源管理接口：列表、分组、详情、保存、删除、导出。所有接口均为 `POST`，请求体参数值统一为字符串类型。

!!! tip "字段差异"
    响应字段以 App 端实际返回为准，本档部分示例字段名（如 `name/host/index/remarks`）可能滞后于实现（实际为 `siteName/siteIdent/url/version/time/isPassword`）。详见 [响应格式说明](response.md)。

---

## 1. 源列表

**接口：** `POST /api/site/list`

**功能：** 获取书源列表，支持搜索、分组筛选和排序

**请求参数：**

| 参数名称 | 类型 | 必填 | 说明 |
|:---|:---|:---|:---|
| `keyword` | String | 否 | 搜索关键词（可搜索源名称/源网址） |
| `groupId` | String | 否 | 源分组 ID |
| `order` | String | 否 | 排序类型；不传时默认跟随 App 书源管理页当前保存的排序 |


**`order` 取值说明：**

所有排序都会先按置顶状态 `isTop` 降序排列，最后再按书源 ID 降序兜底，保证同条件下顺序稳定。

| 值 | App 排序名称 | 排序规则 |
|:---|:---|:---|
| `0` | 默认 | 置顶优先；同置顶状态下按排序值 `index` 降序、更新时间 `updateTime` 降序 |
| `1` | 名称降序 | 置顶优先；同置顶状态下按书源名称 `siteName` 降序 |
| `2` | 名称升序 | 置顶优先；同置顶状态下按书源名称 `siteName` 升序 |
| `3` | 更新时间降序 | 置顶优先；同置顶状态下按更新时间 `updateTime` 降序 |
| `4` | 更新时间升序 | 置顶优先；同置顶状态下按更新时间 `updateTime` 升序 |
| `5` | 创建时间降序 | 置顶优先；同置顶状态下按创建时间 `time` 降序 |
| `6` | 创建时间升序 | 置顶优先；同置顶状态下按创建时间 `time` 升序 |
| `7` | 源网址升序 | 置顶优先；同置顶状态下按源网址 `url` 升序 |

!!! note "默认排序口径"
    如果网页端不传 `order`，接口会读取 App 书源管理页当前保存的排序设置；如果传入未知值，则回退为 `0` 默认排序。

**请求示例：**

```http
POST /api/site/list
Content-Type: application/json

{
  "keyword": "示例",
  "groupId": "1",
  "order": "0"
}
```

**响应示例：**

```json
{
  "code": "200",
  "message": "成功",
  "data": [
    {
      "id": 1,
      "version": 3,
      "time": "2025-10-12",
      "siteName": "示例阅读站",
      "groupId": 1,
      "siteIdent": "example_site",
      "url": "https://novel.example.org",
      "isPassword": false,
      "status": 1,
      "finderStatus": 1
    }
  ]
}
```

---

## 2. 书源分组列表

**接口：** `POST /api/site/group`

**功能：** 获取所有书源分组

**请求参数：** 无

**请求示例：**

```http
POST /api/site/group
Content-Type: application/json

{}
```

**响应示例：**

```json
{
  "code": "200",
  "message": "成功",
  "data": [
    {
      "groupName": "官方源",
      "groupId": 1
    },
    {
      "groupName": "优质源",
      "groupId": 2
    }
  ]
}
```

---

## 3. 书源详情

**接口：** `POST /api/site/info`

**功能：** 获取单个书源的详细信息（需通过密码校验）

**请求参数：**

| 参数名称 | 类型 | 必填 | 说明 |
|:---|:---|:---|:---|
| `id` | String | 是 | 书源 ID |
| `password` | String | 否 | 加密书源密码（书源加密时必填） |

**请求示例：**

```http
POST /api/site/info
Content-Type: application/json

{
  "id": "1",
  "password": "123456"
}
```

**响应示例：**

```json
{
  "code": "200",
  "message": "成功",
  "data": {
    "id": 1,
    "siteName": "示例阅读站",
    "url": "https://novel.example.org",
    "siteJson": "{...完整书源JSON规则...}",
    "groupId": 1,
    "status": 1,
    "finderStatus": 1,
    "version": 3
  }
}
```

---

## 4. 书源保存

**接口：** `POST /api/site/save`

**功能：** 创建新书源或更新现有书源

**请求参数：**

| 参数名称 | 类型 | 必填 | 说明 |
|:---|:---|:---|:---|
| `siteJson` | String | 是 | 书源 JSON 规则串 |
| `id` | String | 否 | 书源 ID（编辑时必填） |
| `index` | String | 否 | 排序（数字越大越靠前） |
| `groupId` | String | 否 | 源分组 ID |
| `status` | String | 否 | 搜索状态（`"1"` 启用，`"0"` 禁用） |
| `finderStatus` | String | 否 | 发现状态（`"1"` 启用，`"0"` 禁用） |
| `remarks` | String | 否 | 源笔记 |

!!! note "V2 规则字段"
    `siteJson` 应提交完整 V2 规则 JSON。搜索规则里的 `siteJson.ruleSearch.aliasName` 为可选字段，含义是“书籍别名规则”；PC 写源保存接口会用完整 `siteJson` 构建 App 规则模型并保存，不会单独白名单过滤该字段。

**请求示例：**

```http
POST /api/site/save
Content-Type: application/json

{
  "siteJson": "{\"siteName\":\"示例书源\",\"host\":\"https://example.com\",\"ruleSearch\":{\"url\":\"https://example.com/search?keyword=@{keyword}\",\"bookList\":\"//div[@class='book-item']\",\"bookName\":\".//h3/text()\",\"aliasName\":\".//span[@class='alias']/text()\",\"bookAuthor\":\".//span[@class='author']/text()\",\"bookUrl\":\".//a/@href\"}}",
  "index": "100",
  "groupId": "1",
  "status": "1",
  "finderStatus": "1",
  "remarks": "测试书源"
}
```

**响应示例：**

```json
{
  "code": "200",
  "message": "保存成功",
  "data": {
    "id": 1
  }
}
```

---

## 5. 书源删除

**接口：** `POST /api/site/delete`

**功能：** 删除指定书源

**请求参数：**

| 参数名称 | 类型 | 必填 | 说明 |
|:---|:---|:---|:---|
| `id` | String | 是 | 书源 ID |

**请求示例：**

```http
POST /api/site/delete
Content-Type: application/json

{
  "id": "1"
}
```

**响应示例：**

```json
{
  "code": "200",
  "message": "删除成功",
  "data": true
}
```

---

## 6. 单个源导出

**接口：** `POST /api/site/export`

**功能：** 导出单个书源（返回 AES 加密后的规则 JSON 字符串）

**请求参数：**

| 参数名称 | 类型 | 必填 | 说明 |
|:---|:---|:---|:---|
| `id` | String | 是 | 书源 ID |

**请求示例：**

```http
POST /api/site/export
Content-Type: application/json

{
  "id": "1"
}
```

**响应示例：**

```json
{
  "code": "200",
  "message": "成功",
  "data": "<AES 加密后的规则 JSON 字符串>"
}
```

---

📅 **更新日期**：2025-10-12
