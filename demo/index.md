---
title: 演示页 — HTML 容器三种写法
layout: pages
---

本页面演示 `{% rawhtml %}` 标签的三种用法。你可以在 Markdown 文件的**任意位置、任意数量**地插入 HTML 容器。

## 参数总览

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `title` | 容器左上方的标题 | `内嵌页面` |
| `width` | iframe 宽度 | `100%` |
| `height` | iframe 高度 | `calc(100vh - 160px)` |
| `minHeight` | 最小高度 | `360px` |
| `src` | 页面 URL（内部路径或外部链接） | 无（进入内嵌模式） |

---

## 一、内部链接

`src` 填入本站的相对路径，指向 Hexo 项目内的独立 HTML 页面：

```
{% rawhtml src="/demo/some-page.html" title="内部页面" width="100%" height="calc(100vh - 160px)" %}
{% endrawhtml %}
```

{% rawhtml src="/demo/some-page.html" title="内部页面示例" width="100%" height="400px" minHeight="200px" %}
{% endrawhtml %}

---

## 二、外部链接

`src` 填入完整的 `https://` 外部 URL，可以加载任意公网页面：

```
{% rawhtml src="https://example.com" title="外部页面" width="100%" height="500px" %}
{% endrawhtml %}
```

{% rawhtml src="https://example.com" title="外部页面示例" width="100%" height="500px" minHeight="300px" %}
{% endrawhtml %}

---

## 三、内嵌 HTML

不写 `src` 参数，直接把 HTML 写在 `{% rawhtml %}` 与 `{% endrawhtml %}` 之间：

```
{% rawhtml title="内嵌 HTML" width="100%" height="400px" %}
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      color: white;
      display: flex;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      font-family: 'Segoe UI', system-ui, sans-serif;
    }
    .card {
      background: rgba(255,255,255,0.15);
      backdrop-filter: blur(10px);
      border-radius: 20px;
      padding: 2rem 3rem;
      text-align: center;
    }
    h1 { font-size: 2rem; margin-bottom: 0.5rem; }
    p  { font-size: 1.1rem; opacity: 0.9; }
    .tip { margin-top: 1rem; font-size: 0.85rem; opacity: 0.6; }
  </style>
</head>
<body>
<div class="card">
  <h1>内嵌 HTML 示例</h1>
  <p>这段 HTML 完全隔离在 iframe 中</p>
  <p class="tip">不受父页面 CSS 影响，点击右上 ↗ 扩大 可全屏查看</p>
</div>
</body>
</html>
{% endrawhtml %}
```

{% rawhtml title="内嵌 HTML 示例" width="100%" height="400px" minHeight="200px" %}
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      color: white;
      display: flex;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      font-family: 'Segoe UI', system-ui, sans-serif;
    }
    .card {
      background: rgba(255,255,255,0.15);
      backdrop-filter: blur(10px);
      border-radius: 20px;
      padding: 2rem 3rem;
      text-align: center;
    }
    h1 { font-size: 2rem; margin-bottom: 0.5rem; }
    p  { font-size: 1.1rem; opacity: 0.9; }
    .tip { margin-top: 1rem; font-size: 0.85rem; opacity: 0.6; }
  </style>
</head>
<body>
<div class="card">
  <h1>内嵌 HTML 示例</h1>
  <p>这段 HTML 完全隔离在 iframe 中</p>
  <p class="tip">不受父页面 CSS 影响，点击右上 ↗ 扩大 可全屏查看</p>
</div>
</body>
</html>
{% endrawhtml %}

---

> 三种模式可以混排在同一个 `.md` 文件中，容器之间可以穿插任意 Markdown 内容。
