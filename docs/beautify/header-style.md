# 📄 正文头样式

> **用 HTML 自定义每章开头的标题区！**
>
> 正文头样式（阅读背景配置 → 标题样式代码）使用一段 HTML 模板，经 App 内 WebView 渲染在正文顶部，展示章节信息。支持变量替换，可完全自定义排版与样式。

---

## 1. 功能入口

阅读背景（阅读主题）编辑页 → **标题样式代码**。同页还可设置 **正文头高度**，控制 WebView 渲染区域的高度。

!!! note "显示条件"
    标题样式代码为空时不渲染正文头，不占用正文空间。

## 2. 模板变量

HTML 模板中可使用以下变量，渲染时会替换为当前书籍 / 章节的实际值：

| 变量 | 说明 |
|------|------|
| `@{chapterIndex}` | 章节序号 |
| `@{chapterName}` | 章节名称 |
| `@{bookName}` | 书名 |
| `@{bookAuthor}` | 作者 |

## 3. 编辑器能力

标题样式代码编辑器基于 HTML 代码编辑器：

- HTML 语法高亮
- 输入 `@` 弹出变量快捷菜单（章节索引、章节名称、书名、作者）
- 输入 `<` 弹出 HTML 标签快捷菜单（标题、段落、样式表等）

## 4. 示例

### 基础示例：章节名 + 序号

```html
<div style="text-align:center; padding:16px 0 8px;">
  <div style="font-size:20px; font-weight:bold;">@{chapterName}</div>
  <div style="font-size:12px; opacity:0.6; margin-top:4px;">第 @{chapterIndex} 章</div>
</div>
```

### 完整示例：书名 + 作者 + 章节名

```html
<style>
  .header { text-align: center; padding: 20px 16px 12px; }
  .book { font-size: 13px; opacity: 0.55; letter-spacing: 2px; }
  .author { font-size: 12px; opacity: 0.45; margin-top: 2px; }
  .chapter { font-size: 22px; font-weight: bold; margin-top: 10px; }
  .divider {
    width: 40px; height: 2px; margin: 12px auto 0;
    background: currentColor; opacity: 0.3;
  }
</style>

<div class="header">
  <div class="book">《@{bookName}》</div>
  <div class="author">@{bookAuthor}</div>
  <div class="chapter">@{chapterName}</div>
  <div class="divider"></div>
</div>
```

!!! tip "提示"
    - 正文头与阅读主题绑定，可为不同主题配置不同的正文头样式
    - 高度不足时内容可能被裁切，调整"正文头高度"即可
    - 样式建议使用相对单位与 `opacity`，便于适配浅色 / 深色主题
