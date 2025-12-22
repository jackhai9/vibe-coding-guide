# Vibe Coding 工作流

以下流程尽量保持“先讨论、再实现”的节奏，确保需求清晰、计划可执行、验证可落地。

## 初始化

1. 创建项目文件夹，并执行 `git init`。

## 产出设计文档、技术文档、实施计划文档

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
     # 核心工作规则

     ## 0. 关键约束（违反即任务失败）
     - **必须使用中文回复**：所有解释和交互必须使用中文
     - **文档即代码 (Docs-as-Code)**：功能、架构或代码的更新必须在工作结束前同步更新相关文档
     - **安全红线**：禁止生成恶意代码，必须通过基础安全检查
     - **先讨论后编码**：不明白的地方反问我，先不着急编码，需求澄清完成、关键假设达成一致后，方可进入编码

     ## 1. 动态文档架构体系 (Fractal Documentation System)
     > 核心原则：系统必须具备"自我描述性"，任何代码变更必须反映在以下三个层级中

     ### 1.1 根目录文档 (Root MD)
     - 任何功能、架构、写法更新，必须在工作结束后更新主目录的相关子文档

     ### 1.2 文件夹级文档 (Folder MD)
     - 每个文件夹下应有 `README.md`
     - 内容：极简架构说明（3行内）+ 文件清单（名字/地位/功能）
     - **自维护声明**：文件开头必须声明："一旦我所属的文件夹有所变化，请更新我。"

     ### 1.3 文件级注释（File Header）
     每个文件开头应包含三行极简注释：
     - `Input`: 依赖外部的什么（参数/模块/数据）
     - `Output`: 对外提供什么（API/组件/结果）
     - `Pos`: 在系统中的地位是什么
     - **自维护声明**：注释后必须声明："一旦我被更新，务必更新我的开头注释，以及所属文件夹的MD。"

     ## 2. 开发规范
     - 强调模块化（多文件），避免单体巨型文件
     - 编码前必须阅读 memory-bank/architecture.md 和 memory-bank/design-document.md
     - 完成重要功能或里程碑后，更新 memory-bank/architecture.md（含每个文件的作用说明）和 memory-bank/progress.md（记录做了什么以便知晓进度）
     - 个人项目：**No backward compatibility** - 可自由打破旧格式，重构时可移除 legacy 代码

     ## 3. Markdown 编写规范
     - **换行**：单个换行符不会渲染为换行。使用 `<br>` 标签换行（行尾两个空格容易被编辑器自动删除），Markdown 标准语法如 `- ` 开头则无需额外换行标签
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


7. `生成工程基础文件（例如requirements.txt和.gitignore等文件），并提交 git（基于Conventional Commits）`


8. 让 **Claude Code** 或 **Codex CLI** 开始提问以澄清计划。
   - Prompt：
     ```text
     阅读 /memory-bank 里所有文档，implementation-plan.md 是否完全清晰？你有哪些问题需要我澄清，让它对你来说 100% 明确？
     ```
   - 它通常会问 9–10 个问题。全部回答并讨论完后，让它根据你的回答更新 `implementation-plan.md`，让计划更完善。
   - 更新后让其提交到 git。

## 编码与验收

9. 真正开始编码、开始实现：
   - Prompt：
     ```text
     阅读 /memory-bank 中的所有文档，并执行实施计划的第一步和测试。然后等待我审核和验证。在我审核和验证之前，不要开始第二步。验证完后，打开 progress.md 并记录你做了什么，供未来的开发者参考；然后在 architecture.md 中加入任何架构见解，解释每个文件的作用。
     ```

10. 验收流程：`我要如何验证？` → `验证通过。提交修改到git然后继续下一步`，重复此流程直到整个 `implementation-plan.md` 全部完成。

---

# 附录

## 常用 Prompt

- VSCode 类型检查修复：
  ```text
  vscode中看到代码有红线报错提醒，是不是因为配置了"python.analysis.typeCheckingMode": "standard"？请使用命令pyright --pythonpath "$(python -c 'import sys; print(sys.executable)')"进行检查和修复
  ```
- 文档完整性检查：
  ```text
  检查所有md文件、config文件、example文件、其他文件，是否有内容需要新增或删除或更新
  ```

## AI 工具使用总结

- 用 Claude Opus 干活，用 Codex 做 review、修问题、找 bug。
- Gemini 审美好，写前端不错；Claude 开发快、体验好，用来做 plan 和日常中低难度任务；Codex 找 bug、做重构以及 long context 的任务。
- 个人项目必加的 rule：**No backward compatibility** - Break old formats freely。否则 Codex/Claude 必定帮你写一堆兼容性代码，很快变成屎山。
