---
name: zhihu-collection-organizer
description: 整理知乎收藏夹：批量将"我的收藏"中的内容按语义分类移到对应收藏夹，使用 browser-harness 在单个标签页内操作
runAs: subagent
allowed-tools: run_command, write_file, read_file, edit_file, search_content, glob
---
# 知乎收藏夹整理 Skill

## 概述

自动化整理知乎（zhihu.com）收藏夹：将"我的收藏"中的内容按语义分类移到对应的专题收藏夹。

## 前置条件

1. 浏览器已登录知乎并打开了一个标签页
2. 已通过 `new_tab()` 打开 `https://www.zhihu.com/collection/926179078`（"我的收藏"页面）
3. 项目根目录下有 `_all_items.json` 和 `_classification.json` 文件

## 分类规则文件 (`_classification.json` 格式)

```json
{
  "classified": {
    "935064503": [{"id": "xxx", "type": "answer"}, ...],
    "973381050": [{"id": "xxx", "type": "article"}, ...]
  },
  "uncategorized": ["id1", "id2", ...]
}
```

## 收藏夹 ID 映射

| 名称 | ID |
|------|:--:|
| 我的收藏 💎 | 926179078 |
| Golang | 973381050 |
| 统计学 & 概率学 | 953837474 |
| C/Cpp & Rust & Zig | 943782357 |
| 编程问题 & 相关技术 | 953835767 |
| AI | 935064503 |
| Helix | 995272094 |
| C# | 983792958 |
| 学习以及方法论 | 981972105 |
| 化学 && 化工 | 935931218 |
| 物理学 & 工程问题 | 953838207 |
| Video Encode/Decode | 988467874 |
| Erlang \|\| Elixir | 974290729 |
| Latex && Typst | 958833465 |
| 线性代数 | 934039369 |
| Python Tech | 928071901 |
| 函数式编程 | 984326333 |
| 数学 | 928481622 |
| Julia && HPC | 958833387 |
| Dart&Flutter\|\|JS/TS | 973441678 |

## 操作流程

### Phase 1: 提取所有条目

1. 使用现有标签页（**不要**调用 `new_tab()`），确认 URL 包含 `collection/926179078`
2. 通过点击页面按钮 `button.PaginationButton` 遍历所有分页
3. 每页用 `document.querySelectorAll('.ContentItem[data-zop]')` 提取条目的 `itemId`、`type` 和标题
4. 翻页通过 `document.querySelectorAll('button.PaginationButton')` 找到对应页码按钮并 `.click()`
5. 去重后保存到 `_all_items.json`

### Phase 2: 按语义分类

根据收藏夹名称判断条目归属：
- 标题匹配关键字（正则）→ 映射到对应收藏夹 ID
- 不匹配任何分类 → 留在"我的收藏"

输出 `_classification.json`

### Phase 3: 执行移动（每个条目两步操作）

对每个已分类条目：

```
Step A: 点击目标收藏夹的「收藏」按钮 → 添加到目标
Step B: 点击「我的收藏」的「已收藏」按钮 → 从我的收藏移除
```

详细操作：

1. 在页面上找到该条目（可能需要翻页）
2. **点击条目的「收藏」按钮**（`button[aria-label="收藏"]`）→ 弹出"添加收藏"弹窗
3. **弹窗中点击目标收藏夹的"收藏"按钮**（蓝色 `Button--blue`）→ 变为"已收藏"（灰色 `Button--grey`）
4. **弹窗中点击"我的收藏"的"已收藏"按钮**（灰色 `Button--grey`）→ 变为"收藏"（蓝色 `Button--blue`）
5. 弹窗自动关闭（或点击关闭按钮 `.Modal-closeButton` 关闭）
6. 记录结果，处理下一个条目

### 弹窗 HTML 结构

```html
<div class="Modal FavlistsModal">
  <div class="Favlists-items">
    <div class="Favlists-item">
      <span class="Favlists-itemNameText">AI</span>
      <button class="Favlists-updateButton Button--blue">收藏</button>  <!-- 未收藏 -->
      <!-- 或 -->
      <button class="Favlists-updateButton Button--grey">已收藏</button>  <!-- 已收藏 -->
    </div>
  </div>
</div>
```

## 注意事项

- **标签页复用**：全程只用一个标签页，不调 `new_tab()`
- **弹窗等待**：点击"收藏"后等待弹窗出现（MutationObserver 或轮询 `.FavlistsModal`）
- **翻页**：自动翻页到包含目标条目的页面
- **进度保存**：每处理 20 个条目保存一次进度到 `_move_progress.json`
- **幂等性**：支持断点续传——跳过已处理的条目
