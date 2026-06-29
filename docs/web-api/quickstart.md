# 🎯 快速开始指南

## 1. 创建并保存书源

```javascript
const siteJson = JSON.stringify({
  siteName: "示例书源",
  host: "https://example.com",
  ruleSearch: {
    url: "https://example.com/search?keyword=@{keyword}",
    bookList: "//div[@class='book-item']",
    bookName: ".//h3/text()",
    aliasName: ".//span[@class='alias']/text()",
    bookAuthor: ".//span[@class='author']/text()",
    bookUrl: "//a/@href"
  },
  ruleChapter: {
    url: "@{bookUrl}/catalog",
    chapterList: "//div[@class='chapter']/a",
    chapterName: "./text()",
    chapterUrl: "./@href"
  },
  ruleContent: {
    url: "@{chapterUrl}",
    contents: "//div[@id='content']/text()"
  }
});

// 保存书源（参数值统一为字符串）
const result = await saveSite(siteJson, {
  index: "100",
  status: "1",
  finderStatus: "0",
  remarks: "测试书源"
});

console.log('书源ID:', result.data.id);
```

## 2. 调试书源

```javascript
// 1. 搜索调试
const searchResult = await debugSearch('斗破苍穹', siteJson);
console.log('搜索结果列表:', searchResult.data.list);

// 2. 章节列表调试
const bookJson = JSON.stringify({
  bookUrl: searchResult.data.list[0].bookUrl,
  name: searchResult.data.list[0].name
});

const chapterResult = await debugChapter(bookJson, siteJson);
console.log('章节列表:', chapterResult.data.list);

// 3. 正文调试
const catalogJson = JSON.stringify({
  url: chapterResult.data.list[0].url,
  name: chapterResult.data.list[0].name
});

const contentResult = await debugContent(catalogJson, siteJson);
console.log('正文内容:', contentResult.data.resultData);
```

## 3. 管理书源

```javascript
// 获取书源列表
// 第三个参数 order 可选；不传时跟随 App 书源管理页当前排序，
// 传 "0" 表示默认排序，传 "1"~"7" 可指定名称/时间/网址等排序。
const list = await getSiteList('示例', '1', '0');

// 获取书源详情
const detail = await fetch('http://localhost:8080/api/site/info', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ id: "1", password: "" })
}).then(res => res.json());

// 删除书源
await fetch('http://localhost:8080/api/site/delete', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ id: "1" })
});
```

---

📅 **更新日期**：2025-10-12  
📝 **文档版本**：v1.0
