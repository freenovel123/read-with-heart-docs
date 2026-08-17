# 响应信息

## 响应处理 - Javascript 规则
在每次请求后的响应中都会返回2个参数，参数一：html（json）内容，参数二：config。可以在源编辑的响应信息中使用js进行处理，获取相应的值。

!!! note "与字段级 JS 的区别"
    本页说明 `response` 字段专用的 `@js:`，参数是 `html` 和 `config`。`@js:` 只用于 `request`、`response` 两个规则字段；URL、书名、作者、列表、正文等普通字段需要运行 JS 时使用 `<js>...</js>`。字段级 `<js>` 的参数是 `value` 和 `config`，并会与正则按书写顺序执行，详见[字段规则执行流水线](rules-Introduction.md#field-rule-pipeline)。

由于app使用的是原生的写法，故采用最了基础的JS写法，不支持浏览器中的一些写法以及Node.js的写法，所以我们内置了一些比如像浏览器中操作Document的方法等。

## 响应 JS 运行时

- 响应 JS 仍可使用 `App.*` / `app.*` / `APP.*` 一致别名访问 Native 能力，详细见 [Native to Javascript](../native-to-js.md)。
- 当前响应 JS 仅暴露 `Document` 子集（`v2.6.0+`），不要把它当成完整的桥接 API；规则作者需要的能力若超出 `Document` 子集，应放到前置请求 JS 中实现。
- 响应 JS 不会自动拿到前置请求的 Put 存储；如需读前置产物，使用 `@get{}` 读取，或者在 JS 里用 `app.sp.get(key)`。
- 响应 JS 里不要依赖 `@put{}` / `@tools{}` 之外的“运行时通用存储”，按场景用 `@get{}` 和 JS 桥接。

### Document的处理
`v2.6.0 以上版本支持`

使用CSS选择器查找元素，具体可以参考 [Jsoup](https://jsoup.org/cookbook/extracting-data/selector-syntax)
```javascript
// 获取标题
let title = document.title();
app.log(title);

// 使用CSS查询选择元素，使用select必定返回的是一个数组,可以使用list.length获取数组长度
let list = document.select("ul li a");
list.forEach(element => {
    // 获取内容
    app.log(element.text());

    // 获取链接
    app.log(element.attr("href"));
});
list.first(); // 获取第一个元素
list.last(); // 获取最后一个元素

// 获取单个元素
let firstElement = document.select("#title").first();
app.log(firstElement.text());

```
