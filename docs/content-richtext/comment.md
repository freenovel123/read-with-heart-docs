# 💬 段评

`<comment>` 标签用于在段落末尾插入"段评按钮"——一个显示评论数的小按钮，点击后由 App 接管段评交互。常用于热门段落评论、互动讨论等场景。

---

## 1. 基本语法

`<comment>` 标签的 `ident` 属性支持三种写法，对应不同的点击行为：

### 标识符写法（默认）

`ident` 为段落业务标识，App 内部用于定位段评内容：

```html
<comment ident="段落ID" count="评论数" />
```

### URL 写法

`ident` 为完整 URL 时，点击段评按钮会打开 WebView 加载该地址：

```html
<comment ident="https://api.example.com/comments?para=1" count="3" />
```

### URL + 自定义请求头写法

URL 后追加 `,{"header":{...}}` JSON 块，可在点击段评按钮打开 WebView 时注入自定义 HTTP Header：

```html
<comment ident="https://api.example.com/p,{'header':{'x-test':'tag'}}" count="3" />
```

!!! warning "格式约束"
    - 标签必须是**自闭合**的（以 `/>` 结尾）
    - JSON 顶层只识别 `header` 字段（`{String:String}`）
    - 支持**单引号**写 JSON，避免与 HTML 双引号冲突
    - URL 自身含 `,` 时仍能正确解析
    - iOS 上 `Cookie` 头受 `HTTPCookieStorage` 接管，可能被忽略或合并，建议先用自定义头（如 `x-test`）验证链路

## 2. 属性规则

| 属性 | 是否必填 | 说明 |
|------|----------|------|
| `ident` | 否 | 段落标识。点击段评按钮时会作为 `paragraphIdent` 传给后续处理逻辑。 |
| `count` | 否 | 评论数，默认 `0`。只有解析为整数且大于 `0` 时才会显示段评按钮。 |

!!! note "显示条件"
    段评功能关闭时会忽略所有 `<comment>` 标签；`count` 缺失、为 `0`、小于 `0` 或无法解析为整数时，不会插入按钮。

## 3. 示例

### 基础用法

```html
<p>这是第一段文字。<comment ident="para_001" count="5" /></p>
```

**效果**：

- 段落末尾显示 `[5条评论]` 按钮
- 点击按钮触发回调，传递 `paragraphIndex` 和 `paragraphIdent`
- 段评按钮点击优先于普通翻页点击

### 不指定 ident

```html
<p>这是一段简单文字。<comment count="3" /></p>
```

**效果**：

- 仅通过 `paragraphIndex` 识别段落
- 适合简单场景

### count 为 0

```html
<p>这段没有评论。<comment ident="para_empty" count="0" /></p>
```

**效果**：

- 不显示段评按钮
- 不占用分页空间

### 携带自定义请求头

```html
<p>正文 <comment ident="https://api.example.com/p,{'header':{'x-test':'tag','x-user':'u123'}}" count="3" /></p>
```

**效果**：

- 段落末尾显示 `[3条评论]` 按钮
- 点击按钮后打开 WebView 请求 `https://api.example.com/p`，自动携带 `x-test: tag` 和 `x-user: u123` 请求头
- 适合需要鉴权或登录态的段评接口

### 复杂示例

```html
<style>
.dialogue {
  margin: 1em 0em 1em 2em;
  line-height: 1.9;
}
.speaker {
  color: #3498db;
  font-weight: bold;
}
</style>

<p class="dialogue">
  <span class="speaker">张飞：</span>
  "大丈夫不能为国出力，更待何时！"
  <comment ident="para_dlg_001" count="12" />
</p>
```

**效果**：

- 样式化的对话 + 段评按钮
- 完美融合

## 4. 书源侧输出规则

- 书源只需要在正文字符串中输出 `<comment ... />` 标签，不需要写任何 App 内部代码。
- `ident` 建议使用能稳定定位段落的业务 ID，例如 `chapter1_para_001`、`dlg_zhangfei_001`。
- `count` 应由书源或接口返回的评论数填入；没有评论时可以不输出 `<comment>`，或输出 `count="0"`。
- 段评是否最终显示，还会受 App 内部段评开关影响；书源侧只负责提供标签和评论数。

## 5. 段评样式

段评按钮样式由 App 阅读主题和段评图标配置决定，书源正文中不需要额外写 CSS 控制按钮样式。

自定义段评图标的 SVG 模板变量（`{{count}}` + 四种阅读主题颜色变量）、SVG 编辑器快捷输入和解析属性编辑，请查看[美化 → 段评样式](../beautify/comment-style.md)。
