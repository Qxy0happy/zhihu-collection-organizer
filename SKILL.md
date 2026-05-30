---
name: zhihu-collection-organizer
description: 整理知乎收藏夹。当你需要将"我的收藏"中大量内容按主题分类移到各专题收藏夹时使用。自动抓取收藏夹列表、用 LLM 判断每条内容归属、执行移动操作。
license: MIT
compatibility: Requires browser-harness (pip install browser-harness) and an active browser session logged into zhihu.com
metadata:
  author: Qxy0happy
---

# 知乎收藏夹整理

## 流程概览

三步：

1. **抓收藏夹** — 从知乎页面抓取所有收藏夹的名称和 ID
2. **LLM 分类** — 逐条判断"我的收藏"中的内容应归入哪个收藏夹
3. **执行移动** — 通过网页弹窗操作，将内容加入目标收藏夹并从"我的收藏"移除

---

## 第一步：抓收藏夹列表

用 browser-harness 打开 `https://www.zhihu.com/collections/mine`（已登录）。

提取页面上所有收藏夹：

```javascript
document.querySelectorAll('a.SelfCollectionItem-title').forEach(a => {
  const id = a.getAttribute('href').replace('/collection/', '');
  const name = a.innerText.trim();
  console.log(id, name);
});
```

保存为 `_collections.json`。

## 第二步：提取所有条目

在同一标签页导航到"我的收藏"页面。遍历 1-50 页：

```javascript
document.querySelectorAll('.ContentItem[data-zop]').forEach(item => {
  const zop = JSON.parse(item.getAttribute('data-zop'));
  // 收集 itemId, type, title
});
```

翻页：`document.querySelectorAll('button.PaginationButton')` 点对应数字或"下一页"。

去重保存为 `_all_items.json`。

## 第三步：LLM 分类

读取 `_all_items.json` 和 `_collections.json`（排除"我的收藏"自身）。对每条内容的标题，用 LLM 判断归属：

- 标题明显与某个收藏夹主题相关 → 归入该收藏夹
- 无法判断 → 留在"我的收藏"不动

输出 `_classification.json`。

## 第四步：执行移动

每个已分类条目两步操作：

1. 翻到条目所在页，点 `button[aria-label="收藏"]` → 弹 `.FavlistsModal`
2. 弹窗中点目标收藏夹的 `.Favlists-updateButton`（文字"收藏"时点）
3. 点"我的收藏"的 `.Favlists-updateButton`（文字"已收藏"时点）
4. 点 `.Modal-closeButton` 关弹窗

每条间隔 3-6 秒，每 5 条存进度。被风控时停止，下次续传。
