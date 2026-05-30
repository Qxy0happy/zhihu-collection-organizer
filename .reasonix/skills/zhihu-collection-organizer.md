---
name: zhihu-collection-organizer
description: 整理知乎收藏夹：将"我的收藏"按语义分类移到对应收藏夹，使用 browser-harness 单标签页操作
runAs: subagent
allowed-tools: run_command, write_file, read_file, edit_file, delete_file, search_content, glob
max-iters: 32
---
# 知乎收藏夹整理 Skill

## 你对当前会话一无所知，所有信息都在下面

这是一个自动化整理知乎收藏夹的 playbook。你要在一个已有 browser-harness 浏览器标签页中操作知乎网页 UI。

## 核心数据：收藏夹 ID 映射

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

## 分类关键字规则（优先级从上到下）

### AI (935064503)
deepseek, llm, rag, prompt, transformer, ai, 人工智能, 大模型, 机器学习, 深度学习, 神经网络, quantiz, 量化, gpt, claude, gemini, chatgpt, qwen, agent, diffusion, pytorch, tensorflow, 强化学习, lora, npu, nlp, 生成式, finetun, fine-tun, spec kit, vibe coding

### Golang (973381050)
golang, go语言, flow库

### Python Tech (928071901)
python, numpy, pandas, seaborn, django, flask

### C/Cpp & Rust & Zig (943782357)
c++, c语言, rust, zig, cmake

### Dart&Flutter||JS/TS (973441678)
dart, flutter, javascript, typescript, node, react, vue, angular, 前端, svelte

### C# (983792958)
c#, .net, csharp

### Julia && HPC (958833387)
julia, hpc, 高性能计算, cuda, sycl

### Helix (995272094)
helix（单独高优先级，匹配 Helix 编辑器）

### Erlang || Elixir (974290729)
erlang, elixir

### Latex && Typst (958833465)
latex, typst, 排版

### Video Encode/Decode (988467874)
video, encode, decode, 视频, 编解码, codec, ffmpeg

### 线性代数 (934039369)
线性代数, 特征值, 二次型, 矩阵论

### 数学 (928481622)
运筹学, 单纯形, 最优化, 优化算法, 数学模型, 概率论

### 统计学 & 概率学 (953837474)
统计, 概率, 回归, 贝叶斯, 假设检验, 方差

### 物理学 & 工程问题 (953838207)
物理, 力学, 流体, 固体物理, 热力学, 量子

### 化学 && 化工 (935931218)
化学, 化工, 反应工程, 换热, 夹点, 材料学

### 函数式编程 (984326333)
函数式, functional, haskell, scala, monad, 纯函数

### 编程问题 & 相关技术 (953835767)
git, github, vscode, ide, docker, markdown, 终端, terminal, 开源, api, wsl, nginx, 正则, regex, 计算机原理, 网络编程, 输入法, slidev, ocaml, claude code, shell, bash, zsh, powershell, cpu, 架构, 设计模式, 重构

### 学习以及方法论 (981972105)
学习方法, 读书, 阅读, 考研, 学术, 文献, 方法论, 科研, zotero, anki

## 操作流程

### Phase 1: 提取条目

1. 用已有标签页（**不**调 `new_tab()`），确认 URL 含 `collection/926179078`
2. 遍历 1-50 页。翻页：`document.querySelectorAll('button.PaginationButton')` 点对应数字或"下一页"
3. 每页提取：
   ```js
   document.querySelectorAll('.ContentItem[data-zop]').forEach(item => {
     var zop = JSON.parse(item.getAttribute('data-zop'));
     // 收集 itemId, type, title
   });
   ```
4. 去重写 `_all_items.json`

### Phase 2: 分类

每条标题匹配上述关键字（正则不区分大小写），输出 `_classification.json`。不匹配的留在"我的收藏"。

### Phase 3: 移动

对每个已分类条目：
1. 翻到所在页
2. 点其 `button[aria-label="收藏"]` → 等 `.FavlistsModal` 弹窗
3. 在弹窗中：
   - 点目标收藏夹的 `Favlists-updateButton`（文字"收藏"时点）
   - 点"我的收藏"的 `Favlists-updateButton`（文字"已收藏"时点）
4. 点 `.Modal-closeButton` 关弹窗
5. 每条间隔 3-6s，每 5 条存 `_move_progress.json`
6. 被风控时停，下次续传

## 注意事项

- 全程只用一个标签页，不调 `new_tab()` / `close_tab()`
- 收藏夹名称汉字比较用 JS 的 `===`
- 支持断点续传：读 `_move_progress.json` 跳过已处理的 ID
