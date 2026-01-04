# Vibe Coding 工作流

以下流程尽量保持“先讨论、再实现”的节奏，确保需求清晰、计划可执行、验证可落地。

## 初始化

1. 创建项目文件夹，记录想法到文档`0_idea.md`，并执行 `git init`。

## 产出设计文档、技术文档

2. 在 [ChatGPT](https://chatgpt.com/?temporary-chat=true) 或 **Claude Code** 或 **Codex CLI** 讨论设计文档。
   - Prompt：
     ```text
     根据0_idea.md的内容，请向我提问并与我讨论，最终确认没问题了就形成一个设计文档，并输出设计文档的纯md源码（外层用四个反引号来包裹防止浏览器直接渲染）。现在你有什么问题要问我吗
     
     或者
     
     根据0_idea.md的内容，请向我提问并与我讨论，最终确认没问题了就输出设计文档“1_design.md“。现在你有什么问题要问我吗
     ```
   - 保存为项目根目录 `1_design.md`。
   - 审查并完善该文档，确保符合需求；可以简陋，不要过度设计，后续会迭代。
   
3. 继续让其推荐最合适（**最简单但最健壮**）的技术栈。
   - Prompt：
     ```text
     请推荐最合适（最简单但最健壮）的技术栈，输出纯md源码（外层用四个反引号来包裹防止浏览器直接渲染）
     
     或者
     
     请推荐最合适（最简单但最健壮）的技术栈，输出文档“2_tech.md“
     ```
   - 保存为项目根目录 `2_tech.md`。

## 创建规则

4. 手动在项目根目录下创建 `CLAUDE.md` 或 `AGENTS.md` 文件：
   ```markdown
   # 核心工作规则
   
   ## 0. 关键约束（违反即任务失败）
   - **必须使用中文**：所有回复、产出文档、日志打印、telegram消息、注释等必须使用中文
   - **文档即代码 (Docs-as-Code)**：功能、架构或代码的更新必须在工作结束前同步更新相关文档
   - **安全红线**：禁止生成恶意代码，必须通过基础安全检查
   - **先讨论后编码**：不明白的地方反问我，先不着急编码，需求澄清完成、关键假设达成一致后，方可进入编码
   - **名词定义**：不要擅自造词，使用我们已明确定义过的词汇。如果还有没定义过的并且需要用到的词汇，请先明确定义后再使用
   
   ## 1. 动态文档架构体系 (Fractal Documentation System)
   > 核心原则：系统必须具备"自我描述性"，任何代码变更必须反映在以下三个层级中
   
   ### 1.1 根目录文档 (Root MD)
   - 任何功能、架构、写法更新，必须在工作结束后更新主目录的相关子文档
   
   ### 1.2 文件夹级文档 (Folder MD)
   - 每个文件夹下应有 `README.md`
   - 内容：极简架构说明（3行内）+ 文件清单（名字/地位/功能）
   - **自维护声明**：文件开头注释必须声明："一旦我所属的文件夹有所变化，请更新我。"
   
   ### 1.3 文件级注释（File Header）
   每个文件开头应包含三行极简注释：
   - `Input`: 依赖外部的什么（参数/模块/数据）
   - `Output`: 对外提供什么（API/组件/结果）
   - `Pos`: 在系统中的地位是什么
   - **自维护声明**：注释后必须声明："一旦我被更新，务必更新我的开头注释，以及所属文件夹的MD。"
   
   **例外（无需文件头注释）**：
   - 代理指引/协作约束文件：`AGENTS.md`、`CLAUDE.md`
   - 运行时本地配置：`config/*.yaml`、`.env`（含 `.env.*`）
   - 自动生成目录/文件：`.pytest_cache/`、`__init__.py`、`logs/`、`__pycache__/`、`*.log`、`*.log.gz`
   - 其他明确标注“不要改动/自动生成”的文件
   
   ## 2. 开发规范
   - 强调模块化（多文件），避免单体巨型文件
   - 编码前必须阅读 memory-bank/architecture.md 和 memory-bank/1_design.md
   - 完成重要功能或里程碑后，更新 memory-bank/architecture.md（含每个文件的作用说明）和 memory-bank/progress.md（记录做了什么以便知晓进度）
   - 个人项目：**No backward compatibility** - 可自由打破旧格式，重构时可移除 legacy 代码
   - 总是运行`python -m pytest`或`python -m pytest -q`来进行测试，不要直接运行 `pytest`
   
   ## 3. Markdown 编写规范
   - **换行**：普通单行换行不会渲染为换行，需用 `<br>` 或改为 Markdown 标准结构（列表/分段/标题）实现换行
   
   ## 4. 安全审计
   - ultrathink：完整研究这个项目，重点看其是否有安全问题，是否有密钥泄漏。列出其对外交互的所有ur和接口。
   
   ## 5. 其他补充
   - README中需要有整个项目的完整的逻辑流程图
   ```

## 产出实施计划文档

5. 将 `1_design.md` 和 `2_tech.md` 提交给 ChatGPT。
   - Prompt：
     ```markdown
     根据这两个上传的文件，请生成一份详细的 实施计划md源码（外层用四个反引号来包裹防止浏览器直接渲染），包含一系列给 AI 开发者的分步指令。
     - 每一步要小而具体。
     - 每一步都必须包含验证正确性的测试。
     - 严禁包含代码，只写清晰、具体的指令。
     - 先聚焦于基础功能，完整功能后面再加。
     
     或者
     
     根据1_design.md和2_tech.md，请生成一份详细的实施计划，输出“3_implementation-plan.md”，包含一系列给 AI 开发者的分步指令。
     - 每一步要小而具体。
     - 每一步都必须包含验证正确性的测试。
     - 严禁包含代码，只写清晰、具体的指令。
     - 先聚焦于基础功能，完整功能后面再加。
     ```
   - 得到 `3_implementation-plan.md`。
   
6. 手动在项目根目录下创建 `memory-bank` 子文件夹，将以下文件放入其中（前 3 个为刚生成的文件，后 2 个为空文件）：
   - `1_design.md`
   - `2_tech.md`
   - `3_implementation-plan.md`
   - `progress.md`（新建空文件，用于记录已完成步骤）
   - `architecture.md`（新建空文件，用于记录每个文件的作用）

7. 生成工程基础文件并提交 git：
   - Prompt：
     ```text
     生成工程基础文件（例如requirements.txt和.gitignore等文件），并提交 git（基于Conventional Commits）
     ```

8. 让 **Claude Code** 或 **Codex CLI** 开始提问以澄清计划。
   - Prompt：
     ```text
     阅读 /memory-bank 里所有文档，3_implementation-plan.md 是否完全清晰？你有哪些问题需要我澄清，让它对你来说 100% 明确？
     ```
   - 它通常会问 9–10 个问题。全部回答并讨论完后，让它根据你的回答更新 `3_implementation-plan.md`，让计划更完善。
   - 更新后让其提交到 git。

## 编码与验收

9. 真正开始编码、开始实现：
   - Prompt：
     ```text
     阅读 /memory-bank 中的所有文档，并执行实施计划的第一步和测试。然后等待我审核和验证。在我审核和验证之前，不要开始第二步。验证完后，打开 progress.md 并记录你做了什么，供未来的开发者参考；然后在 architecture.md 中加入任何架构见解，解释每个文件的作用。
     ```

10. 验收流程：`我要如何验证？` → `验证通过。提交修改到git然后继续下一步`，重复此流程直到整个 `3_implementation-plan.md` 全部完成。

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
  有md会因为我们这次的提交而需要更新内容的吗，全面检查下
  ```
  
- 项目安全性检查：
  
  ```text
  ultrathink：检查这个项目，看是否有密钥泄漏的问题。认证过程会调用可疑的第三方接口或者地址吗？并总结其实现方式。用流程图表示。
  ultrathink：完整研究这个项目，重点看其是否有安全问题，是否有密钥泄漏。列出其对外交互的所有ur和接口。
  ```
  
- 跑测试：
  
  ```text
  python -m pytest -q
  python -m pytest -vv tests -s
  ```
  
- ```text
  implementation和idea和design中还有哪些未实现？
  
  ```
  
  

## AI 工具使用总结

- 用 Claude Opus 干活，用 Codex 做 review、修问题、找 bug。
- Gemini 审美好，写前端不错；Claude 开发快、体验好，用来做 plan 和日常中低难度任务；Codex 找 bug、做重构以及 long context 的任务。
- 个人项目必加的 rule：**No backward compatibility** - Break old formats freely。否则 Codex/Claude 必定帮你写一堆兼容性代码，很快变成屎山。

## 其他类似的工作流

[OpenAI官方推荐的](https://cookbook.openai.com/articles/codex_exec_plans) <br>
[X上这个人的方法](https://x.com/nummanthinks/status/2002724188436738459)