# 动态文档架构体系 (Fractal Documentation System)

核心原则：系统必须具备"自我描述性"。

核心同步协议（Mandatory）

1. **逆向触发器**：任何功能、架构、写法更新，必须在代码修改完成后 → 更新 File Header → 更新所属文件夹的 Folder MD →（若影响全局）更新 Root MD。
2. **分形自治**：确保系统在任何一个子目录下，能通过该目录的 Folder MD 重建局部世界观。

## 1.1 根目录主控文档 (Root MD)

- 根目录下的 `README.md`

## 1.2 文件夹级文档 (Folder MD)

- 每个文件夹下的 `README.md`
- `PROTOCOL`：关于变更触发的声明
- 内容：极简架构说明（3行）+ 文件清单（名字/地位/功能）

**好的示例**

```md
# Folder: src/core

> [PROTOCOL]: 一旦本文件有变更，必须上浮检查 src/README.md 描述是否依然准确，不准确则立即更新(如有必要可重写)

1. **功能域**：系统心脏，处理所有业务状态转换与领域规则，不依赖外部框架。
2. **逻辑流**：接收由 api/ 传入的 DTO，通过 Domain Service 处理，返回领域对象。
3. **约束**：所有计算必须幂等，严禁直接调用 infra/。

## 文件清单

- user_entity.py：用户核心领域模型（State Buffer）
- auth_service.py：鉴权逻辑流（Logic Processor）
- validator.py：领域规则校验器（Gatekeeper）

```

## 1.3 文件级注释（File Header）

每个代码文件开头的注释中应包含：
- `INPUT`: 依赖外部的什么（参数/模块/数据）
- `OUTPUT`: 对外提供什么（API/组件/结果）
- `POS`: 在系统中的地位是什么
- `PROTOCOL`：关于变更触发的声明

**好的示例**

```python
"""
[INPUT]: (Credentials, UserRepo_Interface) - 原始凭证与用户数据访问接口。
[OUTPUT]: (AuthToken, SessionContext) | Exception - 授权令牌或会话上下文。
[POS]: 位于 core 的中位层，作为 api 层与 data 层的逻辑粘合剂。

[PROTOCOL]:
1. 一旦本文件逻辑变更，必须同步更新此 Header。
2. 更新后必须上浮检查 src/core/README.md 描述是否依然准确，不准确则立即更新(如有必要可重写)
"""

class AuthService:
    # …业务逻辑…
    def authenticate(self, creds):
        pass
```

## **例外**

### **无需文件级注释**：
- Root MD
- Folder MD
- 规则/协作约束文件：`AGENTS.md`、`CLAUDE.md`、`rules/`
- 运行时本地配置：`config/*.yaml`、`.env`（含 `.env.*`）
- 自动生成目录/文件：`.pytest_cache/`、`__init__.py`、`logs/`、`__pycache__/`、`*.log`、`*.log.gz`
- 其他明确标注“不要改动/自动生成”的文件

### **无需文件夹级文档**：
- 规则目录：`rules/`
- 自动生成目录：`.pytest_cache/`、`logs/`、`__pycache__/`
- 其他明确标注“不要改动/自动创建”的文件夹

## **Last but not least**

- "Keep the map aligned with the terrain, or the terrain will be lost."
- 必须保持描述与实现一致，否则系统将逐渐变得不可理解。
- 你是这个分形系统的守护者。任何时候你感到逻辑模糊，请先通过更新各级 MD 来澄清你的认知。
