# Vibe Coding 工作流

以下流程尽量保持“先讨论、再实现”的节奏，确保需求清晰、计划可执行、验证可落地。

## 初始化

1. 创建项目文件夹，记录想法到文档`0_idea.md`，并执行 `git init`。

- <details>
  <summary>点击查看0_idea.md模版</summary>

   ```text
   # 初始想法
   
   ## 期望最终效果
   
   最终期望的效果：自动实盘运行，不需要我手工干预。通过配置 trade.dry_run: true/false 控制干跑或实盘模式
   
   用 Python 实现，连接 Binance 交易所进行自动化交易。
   
   Binance U 本位永续合约、Hedge 模式、x5倍杠杆、全仓、开仓后不加仓。
   
   方向：先只做多，等后续程序跑稳定了再加入做空功能实现双向自动交易。
   
   ## 整体流程：
   
   我希望 XXX
   
   
   [ 还有哪些你认为需要补充或完善的？请猜测我的意图，引导我思考并与我一起梳理清楚。。。  ]
   
   ---
   
   ## 一些补充
   
   - YYY
   
   
   ---
   
   ## 其他通用要求
   
   - 两种运行模式：live 模式 、dry_run 模式（只打印，不下单）
     - 最大同时持仓数，只对 live 模式有限制，干跑模式下没有最大同时持仓数限制。
   
   - '交易'时的'浮盈'应该实时监测，需要订阅 markPrice WebSocket。也就是说：实时止损只在'交易'时才实现，'观察'时不需要考虑实时止损
   
   - 涉及到具体数值的，优先考虑可配置
   
     比如
     
       ```yaml
       indicators:
         fast_ma:
           type: EMA
           period: 20
           price: close
         slow_ma:
           type: SMA
           period: 170
           price: close
       ```
   
   - 公共的逻辑和功需要抽取出来，方便复用。新增功能时也要先看下项目中是否已经存在类似的或可复用的功能，避免重复造轮子
   
   - 每个关键步骤和关键操作节点都需要打印日志和发送telegram消息，除非明确说不需要
   
   - 订单类型
   
     - 止损: STOP_MARKET（触发后市价）
     - 止盈: TAKE_PROFIT_MARKET 或 TAKE_PROFIT_LIMIT
   
     都带 reduceOnly=true, 暂时都带 closePosition=true（后续再考虑分批止盈/止损）
   
   - 一些注意事项
     - K线缓存策略：
       - KlineCache 按 symbol#interval 缓存最近 N 根 K 线
       - 首次：获取完整 200 根
       - 后续：增量获取新 K 线，追加到缓存
       - 自动清理过期数据
   
   - 参考如下设计模式（来自 ~/Projects/vibe-quant）：
     - 状态机 + 模式轮转（如果需要的话）
     - 订单隔离：run_id 前缀，避免误撤其他实例或手动订单
     - 日志系统：结构化事件，按事件类型记录，便于追踪和分析，按天自动滚动，自动压缩旧日志为 .gz，错误日志单独文件，成交日志高亮（控制台绿字提示）
     - WebSocket 重连（如果需要ws的话）：指数退避
     - 数据陈旧检测（如果需要的话）
     - 优雅退出机制（参考"业界最佳实践"方向做 Ctrl+C 快速退出）
   
   - Ctrl+C 快速退出的 业界最佳实践 大致思路（尤其是 asyncio + CLI/WS 场景）：
     - 以"取消任务"为核心：收到 SIGINT/SIGTERM 后，向主任务发 cancel，并在各子任务里及时处理 CancelledError，而不是只靠一个 flag。
     - 所有耗时等待都可中断：长时间 sleep、轮询、IO 都要用 asyncio.wait 同时监听 shutdown_event，保证一收到就退出。
     - 阻塞输入要"可忽略"：input() 放到线程里，但主流程不等它；shutdown 后立即返回，忽略线程结果（线程自然挂着没关系）。
     - 外部调用统一加超时：HTTP/WS 都加短超时与取消处理，避免卡住 shutdown。
     - 清理阶段单独封装：在 finally 里关闭 WS、exchange、session，并设置最大清理时间，避免"清理本身卡死"。
     - 日志要清晰区分：shutdown requested、shutdown in progress、shutdown complete，便于排查到底卡在哪一步。
     一句话：用"任务取消 + 可中断等待 + 有界清理"来保证立即退出。
     
   - （如果需要的话）关于 fee 手续费资产：USDⓈ-M 永续通常 commissionAsset 会是 USDT，但不要假设；实现上读取成交回报里的 commissionAsset，若不是 USDT（比如 BNB），就按成交时刻 BNBUSDT（或对应计价对）换算成 USDT 记账，这样最稳。
   
   - （如果需要的话）K-V形式：
     Key: BTC#15m
     StateValue: signal_k_open_time:"xxxx年xx月xx日 xx:xx:xx", entry_k_open_time:"xxxx年xx月xx日 xx:xx:xx", entry_k_open_price:yyy, fill_time:"xxxx年xx月xx日 xx:xx:xx", fill_price:yyy, qty:xxx, take_profit_price:yyy, init_stop_price:yyy, realtime_stop_price:yyy, exit_reason:"mmm", exit_time:"xxxx年xx月xx日 xx:xx:xx", exit_price:yyy, pnl:yyy, pnl_pct:xxx, fee:yyy, fill_order_id:xxx, take_profit_order_id:xxx, init_stop_order_id:xxx, realtime_stop_order_id:xxx, exit_order_id:xxx, risk_usdt:uuu
   
   - （如果需要的话）TTL：条目进入 *_DONE 阶段后，保留 N 个周期。在刚进入 *_DONE 阶段时，把条目持久化 SQLite 数据。过期后直接从 StateMap 中标记删除状态即可
   明确 TTL 的计算方式，否则依旧可能长期堆积。
     建议实现思路：
     - 配置项：done_ttl_bars（按周期数）
     - 当条目进入 BREAKOUT_DONE 或 PULLBACK_DONE 阶段时，记录 done_expire_at
         - done_expire_at = exit_open_time + done_ttl_bars * interval_ms
         - 若 exit_open_time 缺失，用最新 K 线 open_time 兜底
     - 每次阶段处理器跑该周期时，先清理 now >= done_expire_at 的 *_DONE 条目
     - “now”建议用最新 K 线的 open_time，避免本地时间漂移
   
   - （如果需要的话）前期推荐 dry_run: true（干跑模式，只打印，不下单）。等根据历史统计出或聚合出摘要之后发现效果不错再 dry_run: false（实盘模式），摘要推荐：
     - 今日/最近N天：完成次数、胜率、总盈亏、平均盈亏、平均持仓时长
     - 按周期/币种维度的胜率与盈亏
     - 最大连续亏损、最大回撤（如果有权益曲线可算）
   
   - （如果需要的话）程序退出时如何处理未完成挂单：是否取消挂单（做成可配置，默认不主动撤单）
   
   - （如果需要的话）clientOrderId 规范<br>
     结构：`{pnameN}_{role}_{symbol}_{interval}_{side}_{run_id}_{seq}`<br>
     示例：`{pnameN}_E_BTCUSDT_15m_L_rk9x3f_0001`<br>
     字段含义：
     - `pnameN`：系统标识与版本号。比如`tf1`。
     - `role`：订单角色（`E` 入场，`T` 止盈，`I` 初始止损，`R` 实时止损，`X` 退出）。
     - `symbol`：币种。
     - `interval`：周期。
     - `side`：方向（`L`/`S`，当前仅多，预留做空）。
     - `run_id`：本次运行标识（`r` + 6 位 base36 时间戳）。
     - `seq`：同一 key 的递增序号（4 位十进制）。
     解析目标：可稳定解析 `symbol#interval` 与订单角色。
   
   - （如果需要的话）2倍盈亏比: '入场k'的开盘价 kentry_k_open_price 与 初始止损价 init_stop_price 的2倍盈亏比的地方，比如：入场价是7.561，初始止损价是7.184，则止盈单的价格是8.315
   
   - （如果需要的话）部分成交处理: 已成交部分视为持仓并记录成交信息；未成交部分在超时撤单后，作为“剩余数量”继续按入场逻辑重挂；若不再满足“实际可入场”，则接受已成交数量，撤销剩余挂单，按已成交数量挂止损/止盈。
   
   - 
     
   ```
  </details>


## 产出设计文档、技术文档

2. 在 [ChatGPT](https://chatgpt.com/?temporary-chat=true) 或 **Claude Code** 或 **Codex CLI** 讨论设计文档。
   - Prompt：
     ```text
     根据0_idea.md的内容，请先输出你读取到的已经明确的信息，然后再向我提问和讨论你觉得不明确的地方，经过讨论完善后，最终确认没问题后，输出设计文档“1_design.md“（或输出纯md源码(外层用四个反引号来包裹防止浏览器直接渲染)）。现在开始吧
     ```
   - 保存到项目根目录下 `1_design.md`。
   - 审查并完善该文档，确保符合需求；可以简陋，不要过度设计，后续会迭代。
   
3. 继续让其推荐最合适（**最简单但最健壮**）的技术栈。
   - Prompt：
     ```text
     请推荐最合适（最简单但最健壮）的技术栈，输出文档“2_tech.md“（或输出纯md源码(外层用四个反引号来包裹防止浏览器直接渲染)）
     ```
   - 保存到项目根目录下 `2_tech.md`。

## 创建规则

4. 把`rules`目录整个复制到项目根目录下，并手动在项目根目录下创建 `CLAUDE.md` 或 `AGENTS.md` 文件：
   ```markdown
   ## 开发方法论相关规则
   
   - 你必须遵守 rules/spec_driven_rules.md 中定义的规则。
   
   ## 个性化规则
   
   - 你必须遵守 rules/my_own_rules.md 中定义的规则。
   
   ## 动态文档架构体系
   
   - 你必须遵守 rules/fractal_doc_rules.md 中定义的规则。
   ```

## 产出实施计划文档

5. 将 `1_design.md` 和 `2_tech.md` 提交给 ChatGPT。
   - Prompt：
     ```markdown
     根据1_design.md和2_tech.md，请生成一份详细的实施计划，输出“3_implementation-plan.md”（或输出纯md源码(外层用四个反引号来包裹防止浏览器直接渲染)），包含一系列给 AI 开发者的分步指令。
     - 每一步要小而具体。
     - 每一步都必须包含验证正确性的测试。
     - 严禁包含代码，只写清晰、具体的指令。
     - 不仅聚焦于基础功能，也要包含完整功能、集成、集成测试等，要求**全面而且详细**。
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
     
   - 它通常会问 9–10 个问题。全部回答并讨论完后，Prompt：
   
     ```text
     根据我的回答更新 `3_implementation-plan.md`，让计划更完善。更新后提交到 git。
     ```

## 编码与验收

9. 真正开始编码、开始实现：
   - Prompt：
     ```text
     阅读 /memory-bank 中的所有文档，并执行实施计划的第一步和测试。然后等待我审核和验证。在我审核和验证之前，不要开始第二步。验证完后，打开 progress.md 并记录你做了什么，供未来的开发者参考；然后在 architecture.md 中加入任何架构见解，解释每个文件的作用。
     ```

10. 验收流程：

    - Prompt：

      ```
      我要如何验证？
      
      验证通过。提交修改到git然后继续下一步
      ```

    重复此流程直到整个 `3_implementation-plan.md` 中的内容全部实现和完成。

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

