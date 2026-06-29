# 💡 使用示例

本页提供 JavaScript、Python、cURL 三种语言的调用示例。所有示例的请求体参数值均为字符串类型，响应 `code` 字段为字符串。

---

## JavaScript (Fetch API)

```javascript
// 获取书源列表
async function getSiteList(keyword = '', groupId = '', order = '') {
  const response = await fetch('http://localhost:8080/api/site/list', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      keyword: keyword,
      groupId: groupId,
      order: order
    })
  });

  const data = await response.json();
  return data;
}

// 保存书源
async function saveSite(siteJson, options = {}) {
  const response = await fetch('http://localhost:8080/api/site/save', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      siteJson: siteJson,
      ...options
    })
  });

  const data = await response.json();
  return data;
}

// 搜索调试
async function debugSearch(keyword, siteJson) {
  const response = await fetch('http://localhost:8080/api/site/search', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      keyword: keyword,
      siteJson: siteJson
    })
  });

  const data = await response.json();
  return data;
}

// 正文调试
async function debugContent(catalogJson, siteJson) {
  const response = await fetch('http://localhost:8080/api/site/content', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      catalogJson: catalogJson,
      siteJson: siteJson
    })
  });

  const data = await response.json();
  return data;
}
```

---

## Python (requests)

```python
import requests
import json

BASE_URL = 'http://localhost:8080/api'

headers = {
    'Content-Type': 'application/json'
}

# 获取书源列表
def get_site_list(keyword='', group_id='', order=''):
    response = requests.post(
        f'{BASE_URL}/site/list',
        headers=headers,
        json={
            'keyword': keyword,
            'groupId': group_id,
            'order': order
        }
    )
    return response.json()

# 保存书源
def save_site(site_json, **options):
    data = {
        'siteJson': site_json,
        **options
    }
    response = requests.post(
        f'{BASE_URL}/site/save',
        headers=headers,
        json=data
    )
    return response.json()

# 搜索调试
def debug_search(keyword, site_json):
    response = requests.post(
        f'{BASE_URL}/site/search',
        headers=headers,
        json={
            'keyword': keyword,
            'siteJson': site_json
        }
    )
    return response.json()

# 章节列表调试
def debug_chapter(book_json, site_json):
    response = requests.post(
        f'{BASE_URL}/site/chapter',
        headers=headers,
        json={
            'bookJson': book_json,
            'siteJson': site_json
        }
    )
    return response.json()

# 正文调试
def debug_content(catalog_json, site_json):
    response = requests.post(
        f'{BASE_URL}/site/content',
        headers=headers,
        json={
            'catalogJson': catalog_json,
            'siteJson': site_json
        }
    )
    return response.json()

# 清除调试日志
def clear_debug_log(clear_type, site_ident=None):
    data = {'type': str(clear_type)}
    if site_ident:
        data['siteIdent'] = site_ident

    response = requests.post(
        f'{BASE_URL}/debug/clear',
        headers=headers,
        json=data
    )
    return response.json()
```

---

## cURL

```bash
# 获取书源列表
curl -X POST "http://localhost:8080/api/site/list" \
  -H "Content-Type: application/json" \
  -d '{
    "keyword": "示例",
    "groupId": "1",
    "order": "0"
  }'

# 保存书源
curl -X POST "http://localhost:8080/api/site/save" \
  -H "Content-Type: application/json" \
  -d '{
    "siteJson": "{\"name\":\"示例书源\",\"host\":\"https://example.com\"}",
    "index": "100",
    "status": "1"
  }'

# 搜索调试
curl -X POST "http://localhost:8080/api/site/search" \
  -H "Content-Type: application/json" \
  -d '{
    "keyword": "斗破苍穹",
    "siteJson": "{\"name\":\"示例书源\",\"search\":{...}}"
  }'

# 正文调试
curl -X POST "http://localhost:8080/api/site/content" \
  -H "Content-Type: application/json" \
  -d '{
    "catalogJson": "{\"url\":\"https://example.com/chapter/1\"}",
    "siteJson": "{\"name\":\"示例书源\",\"content\":{...}}"
  }'

# 清除搜索日志
curl -X POST "http://localhost:8080/api/debug/clear" \
  -H "Content-Type: application/json" \
  -d '{
    "type": "1"
  }'
```

---

📅 **更新日期**：2025-10-12
