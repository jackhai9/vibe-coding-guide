AI开发工作流

0、创建项目文件夹，git init

1、先在[ChatGPT](https://chatgpt.com/?temporary-chat=true)讨论出一个设计文档，prompt：`我想实现一个xxx，请向我提问并与我讨论，等讨论多次后最终确认没问题了就形成一个设计文档，并输出设计文档的纯md源码（外层用四个反引号来包裹防止浏览器直接渲染）。现在你有什么问题要问我吗`，将结果保存到项目根目录`design-document.md`。审查并完善这个文档，确保符合需求，可以简陋，不要过度设计，后续会迭代

2、让ChatGPT推荐最合适（**最简单但最健壮**）的技术栈，prompt：`请推荐最合适（最简单但最健壮）的技术栈，输出纯md源码（外层用四个反引号来包裹防止浏览器直接渲染）`，将结果保存到项目根目录`tech-stack.md`

3、在终端中打开 **Claude Code** 或 **Codex CLI**，使用 `/init` 命令，它会读取已创建的两个 .md 文件，生成一套规则来正确引导大模型。在生成的 CLAUDE.md 或 AGENTS.md 文件底部追加：

```markdown
## Markdown 编写规范

编写 `.md` 文件时，注意 GitHub 渲染规则：

- **换行**：单个换行符不会渲染为换行。使用 `<br>` 标签换行（行尾两个空格容易被编辑器自动删除），如果是Markdown标准的语法比如开头 `- ` 或类似的就没必要再加 `<br>` 标签换行了


# IMPORTANT:
# emphasize modularity (multiple files) and discourage a monolith (one giant file).
# Always read memory-bank/architecture.md before writing any code. Include entire database schema.
# Always read memory-bank/design-document.md before writing any code.
# After adding a major feature or completing a milestone, update memory-bank/architecture.md with any new architectural insights (including an explanation of each file's role), and record what was done in memory-bank/progress.md for future developers.
# 个人项目，**No backward compatibility** - Break old formats freely.
# 个人项目，重构时可以移除 legacy 代码
# 先讨论，不明白的地方反问我，先不着急编码
```

4、将`design-document.md`和`tech-stack.md`提交给ChatGPT，prompt如下：

```markdown
根据这两个上传的文件，请生成一份详细的 实施计划（Markdown 格式），包含一系列给 AI 开发者的分步指令。
- 每一步要小而具体。
- 每一步都必须包含验证正确性的测试。
- 严禁包含代码，只写清晰、具体的指令。
- 先聚焦于基础功能，完整功能后面再加。
```

得到`implementation-plan.md`

5、在项目根目录下创建子文件夹 `memory-bank`，将以下文件(刚才的3个文件+2个空文件)放入`memory-bank`：

- `design-document.md`
- `tech-stack.md`
- `implementation-plan.md`
- `progress.md`（新建一个空文件，用于记录已完成步骤）
- `architecture.md`（新建一个空文件，用于记录每个文件的作用）

6、`生成工程基础文件（例如requirements.txt和.gitignore等文件），并提交 git（标记为 chore: project bootstrap`   

7、 **Claude Code** 或 **Codex CLI** 开始prompt：`阅读 /memory-bank 里所有文档，implementation-plan.md 是否完全清晰？你有哪些问题需要我澄清，让它对你来说 100% 明确？`它通常会问 9-10 个问题。全部回答和讨论完后，让它根据你的回答更新 `implementation-plan.md`，让计划更完善。并让其提交到git

8、真正开始实现：开始编码了：prompt：`阅读 /memory-bank 中的所有文档，并执行实施计划的第一步和测试。然后等待我审核和验证。在我审核和验证之前，不要开始第二步。验证完后，打开 progress.md 并记录你做了什么，供未来的开发者参考；然后在 architecture.md 中加入任何架构见解，解释每个文件的作用。`

9、`我要如何验证？`

10、`验证通过。提交修改到git然后继续下一步`。。。重复此流程，直到整个 `implementation-plan.md` 全部完成



其他常用prompt：

11、`vscode中看到代码有红线报错提醒，是不是因为配置了"python.analysis.typeCheckingMode": "standard"？请使用命令pyright --pythonpath "$(python -c 'import sys; print(sys.executable)')"进行检查和修复`

12、`检查所有md文件、config文件、example文件、其他文件，是否有内容需要新增或删除或更新`