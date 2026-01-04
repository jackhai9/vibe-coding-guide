
# 规则说明

本目录下的规则文件用于定义明确的**约束条件**(硬规则)，而不是建议。强硬的约束 AI 才会遵守，建议可能会被当成耳旁风😂。

每条规则都需要有：一句话原则、可执行的约束、明确的禁止项（MUST / NO / DO / DO NOT）

有些规则文件是通用的，不只限于当前项目；有些可能是只针对当前项目的。

---

## 规则文件概览（Rule Files Overview）

当前目录中已定义（或规划）的规则文件如下：

- **`spec_driven_rules.md`**   
  开发方法论相关规则，来源于[开发九课](https://github.com/github/spec-kit/blob/main/spec-driven.md#the-nine-articles-of-development)的前三条，是我必须让 AI 遵守的。主要包括：
  - 库优先（Library-first）原则 
  - 必须提供 CLI 接口 
  - 测试优先（Test-first）开发流程

- **`my_own_rules.md`**
  我个人开发中总结的规则

- **`fractal_doc_rules.md`**
  动态文档架构规则，是一个分形结构。来源于《哥德尔、埃舍尔、巴赫》中前半部分提到的复调、自指。参考[X上用户](https://x.com/chunxiangai/status/2002798091813171478)的分享

- **`plans.md`**  
  需要 AI 去 research, design, and implement 的复杂长耗时任务，就让其遵守 plants.md 。来源于Codex工程师[Aaron Friel](https://cookbook.openai.com/articles/codex_exec_plans)的分享

- **（规划中）`trading_safety_rules.md`**  
  与交易逻辑和风险控制相关的硬性安全约束，例如：
  - 风控边界 
  - 禁止的交易行为 
  - 实盘与回测的隔离要求

- **（规划中）`testing_rules.md`**   
  对“测试优先”原则的补充规则，例如：
  - 测试覆盖范围要求 
  - 边界条件与异常场景的最低标准

---

## 如何使用这些规则（How to Use These Rules）

想让 AI 遵守某个或某些规则时，直接在项目根目录的 AGENTS.md/CLAUDE.md 中引用这个目录中的某个或某些规则文件即可。

- 比如，建议在所有项目中都使用 spec_driven_rules.md 开发方法论相关的规则，则在 AGENTS.md/CLAUDE.md 中写入：

  ```markdown
  ## 开发方法论相关规则

  你必须遵守 rules/spec_driven_rules.md 中定义的规则。
  ```

- 比如，当你想让 AI 自行去 research, design, and implement 解决耗时长任务时，则在 AGENTS.md/CLAUDE.md 中写入：

  ```markdown
  ## ExecPlans
   
  编写复杂功能或进行重大重构时，从设计到实现，请使用 ExecPlan（在 rules/plans.md 中有描述）。
  
  ```

  
