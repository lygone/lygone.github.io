---
layout:     post
title:      测试UI风格 — Chirpy 日系靛蓝主题美化效果一览
tags: 测试UI
category: 博客
description: 本篇文章用于展示 Chirpy 日系靛蓝主题的所有 UI 美化元素，涵盖标题、代码、引用、表格、图片、链接等排版效果。
---

这篇文章用于集中预览**日系靛蓝主题**的各项排版美化效果。所有样式均通过 `assets/css/custom.css` 覆写实现，兼容 Chirpy v7.5。

## 标题层级展示

### 三级标题

#### 四级标题

正文段落：日系靛蓝（`#4a6fa5`）是这个主题的主色调，灵感来源于日本传统蓝染工艺中的「靛蓝」色，沉静不失温度，适合技术博客和个人主页。行内代码 `#1c2333` 是侧边栏背景色，与亮色内容区形成干净的层次对比。

---

## 代码块展示

**行内代码**：配置文件位于 `_config.yml`，主题色变量定义在 `html[data-mode="light"]` 下。

**fenced 代码块**：

```css
/* custom.css — 日系靛蓝主题核心变量 */
html[data-mode="light"] {
  --link-color:               #4a6fa5;
  --card-bg:                  #ffffff;
  --card-shadow:              0 1px 3px rgba(0,0,0,0.04), 0 1px 2px rgba(0,0,0,0.06);
  --blockquote-border-color:  #c8d4e2;
  --inline-code-bg:           #eef2f7;
}

#sidebar {
  background-color: #1c2333 !important;
}
```

```python
# 一段 Python 示例
def indigo_palette():
    colors = {
        "primary":   "#4a6fa5",
        "dark":      "#1c2333",
        "light_bg":  "#eef2f7",
        "text":      "#2d3748",
    }
    for name, hex_code in colors.items():
        print(f"{name}: {hex_code}")

indigo_palette()
```

---

## 引用块展示

> 这是单行引用。日系靛蓝主题的引用块采用靛蓝色左边框 + 淡蓝灰背景，文字颜色为 `#4a5568`。
>
> 引用内的 **加粗文字** 和 `代码` 均正常工作。

---

## 表格展示

| 区域 | 选择器 | 效果 |
|---|---|---|
| 侧边栏 | `#sidebar` | 深蓝靛背景 `#1c2333` |
| 文章卡片 | `article.card-wrapper.card` | 10px 圆角 + 阴影 + 悬停上浮 |
| 链接 | `a` | 靛蓝 `#4a6fa5` → 悬停 `#35578a` |
| 标签 | `.post-tag.btn` | 靛蓝文字 + 淡蓝背景 |
| 返回顶部 | `#back-to-top` | 靛蓝圆形按钮 |

---

## 列表展示

**有序列表**：

1. 侧边栏深蓝靛背景，白色标题文字
2. 导航链接悬停出现靛蓝高亮
3. 头像带白色描边 + 圆形阴影
4. 顶栏毛玻璃半透明效果

**无序列表**：

- 文章卡片 10px 圆角
- 悬停时上浮 2px + 阴影加深
- 代码块 8px 圆角 + 细边框
- 滚动条 6px 细条靛灰色

---

## 图片展示

![示例图片 — 蓝色立方体](https://images.unsplash.com/photo-1557672172-298e090bd0f1?w=800&q=80)
*▲ 图片标题居中灰色小字（.image-caption）*

所有图片自动居中、圆角 8px、带微妙阴影。图片标题使用 `.image-caption` 类居中显示。

---

## 链接展示

- 这是一个 [普通链接](https://github.com/lygone/lygone.github.io)
- 访问 [博客首页](/)
- 查看 [标签页](/tags/)
- 查看 [归档页](/archives/)

---

总览完成。以上就是日系靛蓝主题所有主要排版元素的展示。如果对某个区域不满意，可针对性微调。
