# 知乎收藏夹整理工具

通过 browser-harness 自动化整理知乎收藏夹。将「我的收藏」中的内容按语义分类移动到对应专题收藏夹。

## 原理

1. 在单个浏览器标签页中打开「我的收藏」页面
2. 遍历所有分页提取收藏条目（标题、ID、类型）
3. 根据收藏夹名称自动分类（正则匹配标题关键字）
4. 通过弹窗操作：点击目标收藏夹「收藏」+ 点击「我的收藏」「已收藏」，实现移动

## 使用

1. 用 browser-harness 打开知乎并登录
2. 导航到 `https://www.zhihu.com/collection/926179078`
3. 执行 skill 中的流程

## Skill

Reasonix skill 位于 `.reasonix/skills/zhihu-collection-organizer.md`。

## 分类规则

| 目标收藏夹 | 关键字匹配 |
|-----------|-----------|
| AI | deepseek, llm, rag, prompt, transformer, 大模型, 机器学习… |
| Golang | golang, go语言 |
| Python Tech | python, numpy, pandas |
| C/Cpp & Rust & Zig | c++, rust, zig, cmake |
| 编程问题 & 相关技术 | git, vscode, docker, 编辑器, 开源… |
| 数学 | 运筹学, 最优化, 数学模型… |
| 线性代数 | 线性代数, 特征值, 矩阵… |
| 统计学 & 概率学 | 统计, 概率, 回归, 贝叶斯… |
| 物理学 & 工程问题 | 物理, 力学, 流体, 热力学… |
| 化学 && 化工 | 化学, 化工, 反应工程… |
| C# | c#, .net |
| 函数式编程 | 函数式, haskell, scala, monad… |
| Latex && Typst | latex, typst, 排版 |
| Julia && HPC | julia, hpc, cuda |
| Dart&Flutter||JS/TS | dart, flutter, typescript, react… |
| 学习以及方法论 | 学习方法, 考研, 读书, 文献… |
| Video Encode/Decode | video, encode, decode… |
| Helix | helix (编辑器) |
| Erlang || Elixir | erlang, elixir |

## 注意事项

- 知乎有反刷机制，操作间隔建议 3-6 秒
- 建议分批次运行，每次 20-30 条
- 有断点续传机制
