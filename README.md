# Hexo 内嵌 HTML 容器 — `{% rawhtml %}` 标签插件

在 Hexo 的 Markdown 文章中任意位置、任意数量地嵌入原生 HTML 页面。每个 HTML 会渲染在独立容器中，包含标题栏和"扩大"按钮（全屏查看），与周围的 Markdown 内容无缝共存。

在线演示：[功能演示](https://world.sch-nie.com/test-raw-html/)

TODO：解决当子页面需锁定鼠标时，父页面锁定代码会与其冲突导致页面抖动的问题。可由鼠标从左往右滑入子页面时触发。

---

## 目录

- [文件结构](#文件结构)
- [安装方式](#安装方式)
- [使用方式](#使用方式)
- [技术原理](#技术原理)
- [踩坑记录](#踩坑记录)

---

## 文件结构

```
html-container-demo/
├── scripts/
│   └── rawhtml-tag.js              ← 核心脚本，放入 Hexo 的 scripts/ 目录
├── demo/
│   ├── index.md                     ← 演示页面，展示三种写法
│   └── internal-example.html        ← 内部链接演示用的独立 HTML 文件
└── README.md                        ← 本文档
```

---

## 安装方式

1. 将 `scripts/rawhtml-tag.js` 复制到你的 Hexo 项目的 `scripts/` 目录下
2. 重启 Hexo 服务器即可生效

无需安装任何额外的 npm 包。

---

## 使用方式

### 参数说明

| 参数 | 必填 | 默认值 | 说明 |
|------|------|--------|------|
| `title` | 否 | `内嵌页面` | 容器左上方的标题 |
| `width` | 否 | `100%` | iframe 宽度 |
| `height` | 否 | `calc(100vh - 160px)` | iframe 高度 |
| `minHeight` | 否 | `360px` | iframe 最小高度 |
| `src` | 否 | 无 | HTML 页面的 URL。支持本站内部路径（如 `/pages/demo/`）和外部链接（如 `https://example.com`） |

> **注意**：`src` 和内嵌 HTML 二选一。有 `src` 时为引用模式，无 `src` 时为内嵌模式。

### 模式一：src 引用内部 / 外部页面

`src` 参数同时支持内部相对路径和外部完整 URL。

**引用内部页面**（本站已有的 HTML 或其它 Hexo 页面）：

```
{% rawhtml src="/demo/some-page.html" title="内部页面" width="100%" height="calc(100vh - 160px)" %}
{% endrawhtml %}
```

> 本项目 `demo/internal-example.html` 即是一个可用的内部页面示例。放在 Hexo 的 `source/` 目录下并通过 `skip_render` 配置为静态文件即可访问。

**引用外部页面**（任意公网 URL）：

```
{% rawhtml src="https://example.com" title="外部页面" width="100%" height="500px" %}
{% endrawhtml %}
```

> 外部页面受跨域限制，`sandbox` 中需包含 `allow-scripts allow-same-origin` 以允许脚本执行。

### 模式二：内嵌 HTML

适合在标签内直接编写小型 HTML 片段：

```
{% rawhtml title="演示" width="100%" height="400px" %}
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <style>
    body {
      background: linear-gradient(135deg, #667eea, #764ba2);
      color: white;
      display: flex;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
    }
  </style>
</head>
<body>
  <h1>Hello World</h1>
</body>
</html>
{% endrawhtml %}
```

### 在一个 .md 文件中放置多个容器

```
这是普通的 Markdown 段落，可以自由排布。

{% rawhtml src="/demo/some-page.html" title="内部页面" width="100%" height="400px" %}
{% endrawhtml %}

中间可以穿插任意 Markdown 内容。

{% rawhtml src="https://example.com" title="外部页面" width="100%" height="500px" %}
{% endrawhtml %}

这里放一段引用或表格。

{% rawhtml title="内嵌 HTML" width="100%" height="400px" %}
<!DOCTYPE html><html><body><h1>内嵌演示</h1></body></html>
{% endrawhtml %}

容器之后继续写其他内容。
```

---

## 技术原理

### 整体架构

容器渲染采用 **Hexo Filter** 机制，利用 `before_post_render` 和 `after_post_render` 两个钩子完成 HTML 内容的注入。

```
原始 .md 内容
  │
  ├─ before_post_render:  提取 {% rawhtml %} 块 → 替换为占位符
  │
  ├─ Markdown 渲染:       占位符（HTML 注释）原样通过
  │
  └─ after_post_render:   占位符 → 替换为完整容器 HTML
```

### 为什么不用 Nunjucks Tag 插件？

Hexo 的 `hexo.extend.tag.register()` 默认将标签内容交给 Nunjucks 模板引擎处理。Nunjucks 会尝试解析 `{% rawhtml %}...{% endrawhtml %}` 之间的内容，HTML 中的 `<style>` CSS 花括号 `{}` 会导致 Nunjucks 解析失败。

**解决方案**：放弃 Tag 注册方式，改用 Filter 机制直接操作内容字符串。

### before_post_render：提取与占位

1. **代码块保护**：先提取 Markdown 的 ```` ``` ```` 代码块替换为占位符，防止代码块内的 `{% rawhtml %}` 示例文本被误匹配
2. **标签块提取**：用正则 `/{%\s*rawhtml\s+([\s\S]+?)\s*%}([\s\S]*?)\{%\s*endrawhtml\s*%}/g` 匹配标签块，提取参数和正文，替换为 `<!--RAWH-TAG-PH-N-->` 占位符
3. **代码块恢复**：将代码块占位符还原

正则要求 `[\s\S]+?`（至少一个字符）作为参数部分，避免空参数时匹配到文档中的 `` `{% rawhtml %}` `` 内联代码引用。

### after_post_render：渲染容器

Markdown 渲染完成后，将 `<!--RAWH-TAG-PH-N-->` 占位符替换为完整的容器 HTML：

```html
<div class="rawhtml-wrapper">
  <!-- 标题栏 -->
  <div class="rawhtml-bar">
    <span class="rawhtml-bar-title">标题</span>
    <button onclick="...放大逻辑...">↗ 扩大</button>
  </div>
  <!-- iframe -->
  <iframe srcdoc="..." sandbox="..."></iframe>
  <!-- 滚动隔离 -->
  <script>(mouseenter → overflow:hidden)</script>
</div>
```

### 两种 iframe 模式

**src 模式**（有 `src` 参数，支持内部路径和外部 URL）：
- `<iframe src="URL" sandbox="allow-scripts allow-same-origin allow-popups allow-popups-to-escape-sandbox">`
- 内部路径（如 `/demo/some-page.html`）走同一域名，不受跨域限制
- 外部 URL（如 `https://example.com`）走跨域，需适当的 sandbox 权限
- 扩大按钮：`window.open(src)`

**内嵌模式**（无 `src` 参数）：
- HTML 内容先经 `escapeSrcdoc()` 做 HTML 实体转义（`<` → `&lt;` 等）
- `<iframe srcdoc="...转义后的内容..." sandbox="allow-scripts allow-same-origin">`
- 扩大按钮：将原始 HTML 内容用 `Buffer.from().toString('base64')` 做 Base64 编码嵌入 `onclick`，浏览器端用 `atob()` 解码，再经字节→百分号编码→`decodeURIComponent()` 还原 UTF-8 字符串，最后 `window.open('about:blank')` + `document.write()` 写入新窗口

### CSS 隔离

iframe 的 `srcdoc` 属性将 HTML 内容完全封闭在独立的浏览上下文中，父页面的 CSS 不会泄漏进去，子页面的样式也不会污染父页面。

### 滚动隔离

每个 iframe 通过 `mouseenter` / `mouseleave` 事件控制 `document.body.style.overflow`：

- 页面加载时注入 `body { scrollbar-gutter: stable }`，预先为滚动条保留固定空间
- 鼠标进入 iframe → `body { overflow: hidden }`（滚动条消失但空间仍在，无布局抖动）
- 鼠标离开 iframe → 恢复 `overflow` 原值

---

## 踩坑记录

### 1. Nunjucks 解析 HTML 内容失败

**现象**：`Template render error: unexpected end of file`

**原因**：Hexo Tag 插件将内容交给 Nunjucks 渲染，CSS 的 `{ }` 被当成模板语法解析。

**解决**：放弃 Tag 注册，改用 Filter 机制。

### 2. 正则误匹配内联代码

**现象**：文档中 `` `{% rawhtml %}` `` 被当成真正的标签块处理。

**原因**：原正则 `[\s\S]*?` 允许参数为空，`{% rawhtml %}`（无参数）也能匹配。

**解决**：改为 `[\s\S]+?` 要求至少一个参数。

### 3. 参数值含空格被截断

**现象**：`height="calc(100vh - 160px)"` 解析后只剩 `"calc(100vh`。

**原因**：用 `split(/\s+/)` 按空白拆分参数，把引号内空格也拆了。

**解决**：改用正则 tokenizer：`/([^\s"'=]+)=(?:"([^"]*)"|'([^']*)'|(\S+))/g` 正确处理引号包裹的值。

### 4. Blob URL 在新窗口中不可靠

**现象**：扩大按钮点击后新窗口空白或报错。

**原因**：`URL.createObjectURL(new Blob(...))` 在部分浏览器中 `window.open` 后不生效。

**解决**：改用 Base64 嵌入 + `document.write` 方案。

### 5. 中文乱码

**现象**：扩大后新窗口中文变成乱码。

**原因**：`atob(base64)` 得到的是字节串（每个字符 = 1 字节），中文字符的 UTF-8 多字节序列被当成 Latin-1 字符渲染。

**解决**：将每个字节转为 `%XX` 百分号编码，再经 `decodeURIComponent()` 还原为正确 UTF-8 字符串。

```javascript
decodeURIComponent(
  atob(base64).split('').map(function(c) {
    return '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2);
  }).join('')
);
```

### 6. window.open 空 URL

**现象**：新窗口地址栏显示 `about:blank` 且无法写入内容。

**原因**：`window.open('', '_blank')` 空 URL 在不同浏览器中行为不一致。

**解决**：显式指定 `window.open('about:blank', '_blank')`。

### 7. lightbox 事件监听器在 DOM 前执行

**现象**：在 iframe 中加载的外部页面报 `Cannot read properties of null`。

**原因**：外部 HTML 的 `<script>` 中 `document.getElementById('lightbox')` 调用了尚未解析的 DOM 元素（`<div id="lightbox">` 定义在 `<script>` 之后）。

**解决**：将事件监听器包裹在 `DOMContentLoaded` 中，并添加 null 检查。

### 8. 滚动隔离导致页面疯狂抖动

**现象**：鼠标从左侧移入 iframe 时，页面疯狂抖动，鼠标同时控制两层页面滚动。Edge 中尤其明显。

**第一版修复**（不彻底）：`mouseenter` 时计算 `innerWidth - clientWidth` 算出滚动条宽度，同步设置 `body { padding-right }` 补偿位移。在部分浏览器中仍因样式应用时序产生单帧抖动。

**最终方案**：使用 CSS 原生属性 `body { scrollbar-gutter: stable }`（Edge 94+、Chrome 94+ 均支持）。该属性**始终**为滚动条预留空间（无论是否显示），`overflow: hidden` 时滚动条消失但布局完全不位移，从根本上消除 `mouseenter`/`mouseleave` 的死循环。

---

## 许可

自由使用，无需署名。
