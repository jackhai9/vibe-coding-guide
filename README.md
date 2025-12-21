# AI 开发工作流

以下流程尽量保持“先讨论、再实现”的节奏，确保需求清晰、计划可执行、验证可落地。

## 初始化

1. 创建项目文件夹，并执行 `git init`。

## 产出设计与技术文档

2. 在 [ChatGPT](https://chatgpt.com/?temporary-chat=true) 讨论设计文档。
   - Prompt：

```text
我想实现一个xxx，请向我提问并与我讨论，等讨论多次后最终确认没问题了就形成一个设计文档，并输出设计文档的纯md源码（外层用四个反引号来包裹防止浏览器直接渲染）。现在你有什么问题要问我吗
```

   - 保存为项目根目录 `design-document.md`。
   - 审查并完善该文档，确保符合需求；可以简陋，不要过度设计，后续会迭代。

3. 让 ChatGPT 推荐最合适（**最简单但最健壮**）的技术栈。
   - Prompt：

```text
请推荐最合适（最简单但最健壮）的技术栈，输出纯md源码（外层用四个反引号来包裹防止浏览器直接渲染）
```

   - 保存为项目根目录 `tech-stack.md`。

4. 在终端打开 **Claude Code** 或 **Codex CLI**，使用 `/init` 命令。
   - 它会读取已创建的两个 `.md` 文件，生成一套规则来正确引导大模型。
   - 在生成的 `CLAUDE.md` 或 `AGENTS.md` 文件底部追加：

```markdown
## Markdown 编写规范

编写 `.md` 文件时，注意 GitHub 渲染规则：

- **换行**：单个换行符不会渲染为换行。使用 `<br>` 标签换行（行尾两个空格容易被编辑器自动删除），如果是 Markdown 标准的语法比如开头 `- ` 或类似的就没必要再加 `<br>` 标签换行了


# IMPORTANT:
# emphasize modularity (multiple files) and discourage a monolith (one giant file).
# Always read memory-bank/architecture.md before writing any code. Include entire database schema.
# Always read memory-bank/design-document.md before writing any code.
# After adding a major feature or completing a milestone, update memory-bank/architecture.md with any new architectural insights (including an explanation of each file's role), and record what was done in memory-bank/progress.md for future developers.
# 个人项目，**No backward compatibility** - Break old formats freely.
# 个人项目，重构时可以移除 legacy 代码
# 先讨论，不明白的地方反问我，先不着急编码
```

5. 将 `design-document.md` 和 `tech-stack.md` 提交给 ChatGPT。
   - Prompt：

```markdown
根据这两个上传的文件，请生成一份详细的 实施计划（Markdown 格式），包含一系列给 AI 开发者的分步指令。
- 每一步要小而具体。
- 每一步都必须包含验证正确性的测试。
- 严禁包含代码，只写清晰、具体的指令。
- 先聚焦于基础功能，完整功能后面再加。
```

   - 得到 `implementation-plan.md`。

6. 在项目根目录下创建 `memory-bank` 子文件夹，将以下文件放入其中（前 3 个为刚生成的文件，后 2 个为空文件）：
   - `design-document.md`
   - `tech-stack.md`
   - `implementation-plan.md`
   - `progress.md`（新建空文件，用于记录已完成步骤）
   - `architecture.md`（新建空文件，用于记录每个文件的作用）

## 初始化工程

7. `生成工程基础文件（例如requirements.txt和.gitignore等文件），并提交 git（标记为 chore: project bootstrap）`

## 校准计划

8. 让 **Claude Code** 或 **Codex CLI** 开始提问以澄清计划。
   - Prompt：

```text
阅读 /memory-bank 里所有文档，implementation-plan.md 是否完全清晰？你有哪些问题需要我澄清，让它对你来说 100% 明确？
```

   - 它通常会问 9–10 个问题。全部回答并讨论完后，让它根据你的回答更新 `implementation-plan.md`，让计划更完善。
   - 更新后让其提交到 git。

## 实施与验收

9. 真正开始编码、开始实现：
   - Prompt：

```text
阅读 /memory-bank 中的所有文档，并执行实施计划的第一步和测试。然后等待我审核和验证。在我审核和验证之前，不要开始第二步。验证完后，打开 progress.md 并记录你做了什么，供未来的开发者参考；然后在 architecture.md 中加入任何架构见解，解释每个文件的作用。
```

10. 验收流程：`我要如何验证？` → `验证通过。提交修改到git然后继续下一步`，重复此流程直到整个 `implementation-plan.md` 全部完成。

## 其他常用 Prompt

- `vscode中看到代码有红线报错提醒，是不是因为配置了"python.analysis.typeCheckingMode": "standard"？请使用命令pyright --pythonpath "$(python -c 'import sys; print(sys.executable)')"进行检查和修复`
- `检查所有md文件、config文件、example文件、其他文件，是否有内容需要新增或删除或更新`

## 各 AI 工具使用总结

- 用 Claude Opus 干活，用 Codex 做 review、修问题、找 bug。
- Gemini 审美好，写前端不错；Claude 开发快、体验好，用来做 plan 和日常中低难度任务；Codex 找 bug、做重构以及 long context 的任务。
- 个人项目必加的 rule：**No backward compatibility** - Break old formats freely。否则 Codex/Claude 必定帮你写一堆兼容性代码，很快变成屎山。
