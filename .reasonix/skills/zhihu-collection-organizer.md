---
name: zhihu-collection-organizer
description: 整理知乎收藏夹：将"我的收藏"按语义分类移到对应收藏夹，使用 browser-harness 单标签页操作
run_as: subagent
allowed_tools:
  - run_command
  - write_file
  - read_file
  - edit_file
  - delete_file
  - search_content
  - glob
---
# 知乎收藏夹整理 Skill

## 你对当前会话一无所知，所有信息都在下面

这是一个自动化整理知乎收藏夹的 playbook。你要在一个已有 browser-harness 浏览器标签页中操作知乎网页 UI。

## 核心数据：收藏夹 ID 映射（硬编码）

| 收藏夹名称 | ID |
|-----------|:--:|
| 我的收藏 | 926179078 |
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

## 分类关键字规则

对条目标题进行正则匹配（大小写不敏感），优先级从上到下：

### AI (935064503)
`deepseek`, `llm`, `rag\b`, `prompt`, `transformer`, `\bai\b`, `人工智能`, `大模型`, `机器学习`, `深度学习`, `神经网络`, `quantiz`, `量化.*(模型|推理)`, `\bgpt\b`, `claude`, `gemini`, `chatgpt`, `qwen`, `\bagent\b`, `diffusion`, `pytorch`, `tensorflow`, `强化学习`, `lora\b`, `npu\b`, `\bnlp\b`, `生成式`, `finetun`, `fine.?tun`, `spec kit`, `vibe coding`, `动手学深度学习`, `机器学习.*(周志华|入门|数学)`, `深度学习.*(入门|无法|路线)`, `transformer.*(理解|Q|K|V)`

### Golang (973381050)
`golang`, `go语言`, `flow库`

### Python Tech (928071901)
`python`, `numpy`, `pandas`, `seaborn`, `django`, `flask\b`

### C/Cpp & Rust & Zig (943782357)
`c[+\+]{2}`, `c语言`, `rust`, `zig`, `cmake`

### Dart&Flutter||JS/TS (973441678)
`dart`, `flutter`, `javascript`, `typescript`, `node\.?js`, `react\b`, `vue\b`, `angular`, `前端`, `svelte`

### C# (983792958)
`c#`, `\.net`, `csharp`

### Julia && HPC (958833387)
`julia`, `hpc`, `高性能计算`, `cuda`, `sycl`

### Helix (995272094)
`helix` (单独高优先级，匹配 Helix 编辑器)

### Erlang || Elixir (974290729)
`erlang`, `elixir`

### Latex && Typst (958833465)
`latex`, `typst`, `排版`

### Video Encode/Decode (988467874)
`video`, `encode`, `decode`, `视频`, `编解码`, `codec`, `ffmpeg`

### 线性代数 (934039369)
`线性代数`, `特征值`, `二次型`, `矩阵论`, `线性.*(空间|变换)`

### 数学 (928481622)
`运筹学`, `单纯形`, `最优化`, `优化.*(算法|理论)`, `数学模型`, `概率论`, `冷门.*数学`

### 统计学 & 概率学 (953837474)
`统计`, `概率`, `回归`, `贝叶斯`, `假设检验`, `方差`

### 物理学 & 工程问题 (953838207)
`物理`, `力学`, `流体`, `固体物理`, `热力学`, `量子`

### 化学 && 化工 (935931218)
`化学`, `化工`, `反应工程`, `换热`, `夹点`, `材料学`

### 函数式编程 (984326333)
`函数式`, `functional`, `haskell`, `scala`, `monad`, `纯函数`

### 编程问题 & 相关技术 (953835767)
`git`, `github`, `vscode`, `编辑器`, `ide`, `docker`, `markdown`, `wezterm`, `终端`, `terminal`, `开源`, `api`, `windows.*(兼容|技术)`, `wsl`, `nginx`, `regex`, `正则`, `计算机.*(原理|组成|基础)`, `网络.*(协议|编程)`, `输入法`, `软件`, `slidev`, `ocaml`, `代码.*(阅读|理解)`, `claude code`, `缓存`, `ramdisk`, `性能.*(优化|提升)`, `浏览器`, `ssh`, `shell`, `bash`, `zsh`, `powershell`, `cpu`, `架构`, `设计模式`, `重构`

### 学习以及方法论 (981972105)
`学习.*(方法|路径|规划|建议|习惯)`, `读书`, `阅读`, `考研`, `学术`, `文献`, `方法论`, `教育`, `科研`, `zotero`, `anki`

## Phase 1: 提取所有条目

1. 切换到已有 browser-harness 标签页（**不**调 `new_tab()`），确认 URL 包含 `collection/926179078`
2. 遍历 1-50 页。翻页：`document.querySelectorAll('button.PaginationButton')` 找对应数字或"下一页"按钮。
3. 每页提取：
   ```javascript
   document.querySelectorAll('.ContentItem[data-zop]').forEach(function(item) {
     var zop = JSON.parse(item.getAttribute('data-zop'));
     var titleEl = item.querySelector('.ContentItem-title a');
     // 收集 itemId, type, title
   });
   ```
4. 用 `wait_for_load(3)` 等翻页加载
5. 去重后写到 `_all_items.json`

## Phase 2: 分类

用上述关键字规则匹配每个条目标题，输出 `_classification.json`。
不匹配的留在"我的收藏"。

## Phase 3: 执行移动

对每个已分类条目：

1. 翻到目标条目所在页
2. 找到条目的 `button[aria-label="收藏"]` → `.click()`
3. 等待 `.FavlistsModal` 弹窗出现（约 1.5s）
4. 在弹窗中：
   - 找到目标收藏夹的 `.Favlists-updateButton`，如果文字是"收藏"则点击
   - 找到"我的收藏"的 `.Favlists-updateButton`，如果文字是"已收藏"则点击
5. 点击 `.Modal-closeButton` 关弹窗
6. 每条间隔 3-6 秒随机延迟，每 5 条存进度到 `_move_progress.json`
7. 被知乎风控时停止，下次从进度文件继续

## 注意事项

- **全程只用一个标签页**，不调 `new_tab()`／`close_tab()`
- **正则需要转义**，Python 中写 `r'...'` 或双重反斜杠
- **收藏夹名称用汉字直接匹配**（如 `'AI'`、`'数学'`），在 JS 中用 `===` 比较
- **支持断点续传**：读取 `_move_progress.json` 跳过已处理 ID
