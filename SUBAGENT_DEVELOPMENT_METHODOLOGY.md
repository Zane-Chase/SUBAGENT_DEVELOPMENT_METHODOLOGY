# Subagent 并发开发方法论

> 基于 Sequential Thinking MCP + 文档化工作流的 AI 协作开发方法
> **通用 Web 项目开发方法论模板**
> **适用**: 大型复杂 Web 项目的 AI 辅助开发
> **更新**: 改造为通用 Web 项目模板，涵盖前端/后端/数据库/DevOps 等全栈开发

---

## 📌 重要说明

**本文档是通用 Web 项目开发方法论模板**

本文档中的所有示例（用户认证、数据库设计等）仅供演示方法论的使用方式。

⚠️ **重要提示**：
- 文档中可能仍有部分来自原始项目的代码示例和场景描述
- 这些示例**仅用于说明方法论**，而非具体实现建议
- 在实际项目中，请根据你的业务领域和技术栈进行相应替换

在应用到你的项目时：
1. **必须**根据项目实际需求定义 Subagent 角色
2. **必须**替换示例代码为你的技术栈和业务场景
3. **建议**参考 `~/.claude/agents/` 中的标准 agent 定义
4. **建议**从小功能开始，逐步扩展到复杂系统

详细使用步骤请参考：[🎯 使用本模板的步骤](#-使用本模板的步骤)

## 📋 目录

- [📌 重要说明](#-重要说明)
- [方法论概述](#方法论概述)
- [核心原理](#核心原理)
- [架构设计](#架构设计)
- [TDD 工作流程](#tdd-工作流程)
- [Token 控制策略](#token-控制策略)
- [文档化工作流](#文档化工作流)
- [完整工作流程](#完整工作流程)
  - [Phase 0: MCP 环境准备](#phase-0-mcp-环境准备)
  - [Phase 1: 准备阶段](#phase-1-准备阶段)
  - [Phase 2: 执行阶段](#phase-2-执行阶段)
  - [Phase 3: 整合阶段](#phase-3-整合阶段)
  - [Phase 4: 迭代](#phase-4-迭代)
- [优点总结](#优点总结)
- [最佳实践](#最佳实践)
  - [🚨 核心工作流强制规范](#0-核心工作流强制规范-)
  - [本地开发环境配置规范](#6-本地开发环境配置规范-)
  - [MCP 使用规范](#7-mcp-使用规范-)
  - [任务设计原则](#1-任务设计原则)
  - [代码质量和注释规范](#10-代码质量和注释规范-)
  - [调试日志规范](#11-调试日志规范-)
  - [测试验证规范](#12-测试验证规范-)
  - [文档编写原则](#2-文档编写原则)
  - [Subagent 角色设计](#3-subagent-角色设计)
  - [Controller 决策原则](#4-controller-决策原则)
  - [Token 预算监控](#5-token-预算监控)
- [实际应用示例](#实际应用示例)
- [🎯 使用本模板的步骤](#-使用本模板的步骤)
- [💼 常见 Web 项目应用场景](#-常见-web-项目应用场景)
  - [场景1：电商平台](#场景1电商平台-e-commerce)
  - [场景2：SaaS 管理平台](#场景2saas-管理平台)
  - [场景3：社交/内容平台](#场景3社交内容平台)
  - [场景4：企业管理系统](#场景4企业管理系统-erpcrm)
  - [场景5：实时协作工具](#场景5实时协作工具)

---

## 方法论概述

### 什么是 Subagent 并发开发？

Subagent 并发开发是一种基于 **AI Agent 角色分工** 和 **文档化知识管理** 的软件开发方法论。它通过以下方式提升开发效率：

1. **Sequential Thinking Controller**（总协调器）负责全局规划和决策
2. **多个专业 Subagent**（通过不同 prompt 定义）并发处理独立任务
3. **文档化工作流** 实现知识持久化和上下文隔离
4. **Token 预算控制** 确保每个 Agent 高效运行

### 适用场景

✅ **适合**：
- 大型项目（多模块、多层次）
- 复杂系统架构（微服务、多组件集成）
- 长期迭代开发（需要知识积累）
- 团队协作项目（需要文档化沟通）

❌ **不适合**：
- 简单的单文件修改
- 一次性小脚本开发
- 无需复用的一次性任务

---

## 核心原理

### 1. 角色分离原则（Separation of Concerns）

```
┌─────────────────────────────────────────────────────────┐
│     Sequential Thinking Controller (我)                 │
│  - 全局分析和决策                                        │
│  - 任务分解和分配                                        │
│  - 结果整合和质量控制                                     │
└─────────────────────────────────────────────────────────┘
                         │
          ┌──────────────┼──────────────┐
          │              │              │
    ┌─────▼─────┐  ┌─────▼─────┐  ┌────▼──────┐
    │ Subagent1 │  │ Subagent2 │  │ Subagent3 │
    │ 前端开发  │  │ 后端架构  │  │ 数据库设计│
    └───────────┘  └───────────┘  └───────────┘
          │              │              │
    ┌─────▼─────┐  ┌─────▼─────┐  ┌────▼──────┐
    │ Subagent4 │  │ Subagent5 │  │ Subagent6 │
    │ API 安全  │  │ DevOps   │  │ 测试部署  │
    └───────────┘  └───────────┘  └───────────┘
```

**关键点**：
- ⚠️ **Controller（主窗口）永远不写代码**：只做任务规划、分配和进度跟踪
- ⚠️ **所有代码由 Subagent 编写**：通过独立的 Task tool 调用
- ✅ **Subagent 专注于特定领域**：职责单一，边界清晰
- ✅ **独立上下文窗口**：每个 Subagent 有独立的 token 预算
- ✅ **避免 token 累积**：主窗口只读取输出文档，不参与代码编写

**工作流强制规则**：
```
❌ 错误做法（主窗口直接写代码）：
Controller → 直接修改代码文件 → Token 快速累积 → 上下文溢出

✅ 正确做法（使用 Subagent）：
Controller → Sequential Thinking MCP 拆解任务
         → 创建任务文档
         → 启动 Subagent (Task tool)
         → Subagent 写代码并输出文档
         → Controller 读取输出更新进度
         → 主窗口 Token 保持低位
```

### 2. 文档化知识管理（Structured Note-Taking）

**灵感来源**：Anthropic 的 Context Engineering 最佳实践

> "Agent 定期将结果写入持久化存储，后续任务只需读取文档，而不是重新执行所有历史操作"

**实现方式**：
```
任务 A → 输出文档 A
              ↓
任务 B（读取文档 A）→ 输出文档 B
                            ↓
任务 C（读取文档 B）→ 输出文档 C
```

**优势**：
- 避免上下文累积（每个任务都是轻量级的）
- 知识可复用（文档可被后续任务或人类查阅）
- 可追溯（完整的任务链记录）

### 3. Token 预算控制（Context Window Management）

**核心理念**：不要填满上下文窗口（Andrej Karpathy："填充恰当信息的艺术"）

**科学依据**：
- Claude Sonnet 4 的有效上下文长度约 **60k-120k tokens**
- 超过有效长度会导致性能下降（lost-in-the-middle effect）
- 设定 **50% 预算限制**（100k tokens）确保最佳性能

**控制策略**：
| 组件 | Token 预算 | 占比 |
|------|-----------|------|
| Controller（我） | 40k-60k | 4-6% |
| 单个 Subagent | 20k-30k | 2-3% |
| N 个并发 Subagent | 20k×N - 30k×N | 根据任务数动态调整 |
| 可用总预算 | 1,000k tokens | 100% |

**并发能力**：
- 最小并发：4 个任务（100k tokens）
- 常规并发：10-15 个任务（250k-450k tokens）
- 大规模并发：30+ 个任务（600k-900k tokens）
- 理论上限：40 个任务（1M tokens 预算内）

### 4. 任务粒度控制（Task Granularity）

**原则**：一个任务只修改 1-2 个文件，每个文件 200-300 行

**Token 估算公式**：
```
单个任务 Token 使用 = 任务文档 (5k)
                    + 读取代码 (6k)
                    + 生成代码 (10k)
                    + 测试说明 (3k)
                    ≈ 25k tokens
```

**大任务分解示例**：
```
❌ 大任务："实现完整的用户管理系统"（预计 80k tokens）

✅ 拆分为 4 个小任务：
  - 任务 1a: 实现用户认证 API（25k）
  - 任务 1b: 实现权限管理（20k）
  - 任务 1c: 添加密码重置功能（15k）
  - 任务 1d: 编写测试用例（20k）
```

---

## 架构设计

### 目录结构

```
your-web-project/
├── SUBAGENT_DEVELOPMENT_METHODOLOGY.md   # 本文档（包含所有 Subagent 角色定义）
├── docs/
│   ├── tasks/                            # 任务输入（给 Subagent 的指令）
│   │   ├── phase1/
│   │   │   ├── task-1a-user-auth-api.md
│   │   │   ├── task-1b-database-schema-design.md
│   │   │   ├── task-1c-frontend-login-ui.md
│   │   │   └── task-1d-api-security-audit.md
│   │   ├── phase2/
│   │   │   ├── task-2a-product-catalog-api.md
│   │   │   ├── task-2b-shopping-cart-component.md
│   │   │   └── ...
│   │   └── phase3/
│   │       └── ...
│   │
│   ├── outputs/                          # 任务输出（Subagent 完成的成果）
│   │   ├── phase1/
│   │   │   ├── output-1a-auth-api-implementation.md
│   │   │   ├── output-1b-database-schema.md
│   │   │   └── ...
│   │   ├── phase2/
│   │   └── phase3/
│   │
│   ├── progress/                         # 进度跟踪
│   │   ├── phase1-progress.md
│   │   ├── phase2-progress.md
│   │   └── phase3-progress.md
│   │
│   └── knowledge-base/                   # 可复用知识库
│       ├── api-design-patterns.md
│       ├── database-migration-guide.md
│       ├── frontend-component-library.md
│       └── deployment-checklist.md
│
├── src/                                  # 源代码
│   ├── frontend/                         # 前端代码 (React/Next.js等)
│   ├── backend/                          # 后端代码 (Node.js/Python等)
│   └── database/                         # 数据库脚本和迁移
│
└── infrastructure/                       # 基础设施代码 (Terraform/K8s等)
    └── ...
```

### Subagent 角色定义

> **说明**：以下是通用 Web 项目的 Subagent 角色模板。实际项目应根据具体需求调整角色定义。
> 可参考 `~/.claude/agents/` 目录中的 agent 定义进行扩展。

#### 1. Frontend Developer 🎨
- **专长**：React/Next.js/Vue.js，响应式UI，状态管理
- **职责**：实现前端组件、页面布局、用户交互
- **工作范围**：`src/frontend/`, `src/components/`
- **参考**：`~/.claude/agents/frontend-developer.md`

#### 2. Backend Architect 🏗️
- **专长**：RESTful/GraphQL API 设计，微服务架构，事件驱动系统
- **职责**：设计后端服务、API 接口、服务边界定义
- **工作范围**：`src/backend/`, `src/api/`, `src/services/`
- **参考**：`~/.claude/agents/backend-architect.md`

#### 3. Database Architect 🗄️
- **专长**：数据建模，SQL/NoSQL 数据库选型，性能优化
- **职责**：设计数据库 schema、索引策略、迁移方案
- **工作范围**：`src/database/`, `migrations/`
- **参考**：`~/.claude/agents/database-architect.md`

#### 4. API Security Auditor 🔒
- **专长**：API 安全审计，OWASP Top 10，认证授权
- **职责**：安全漏洞扫描、身份验证设计、数据保护
- **工作范围**：所有 API 端点、认证模块
- **参考**：`~/.claude/agents/api-security-audit.md`

#### 5. DevOps Engineer 🚀
- **专长**：CI/CD、容器化、Kubernetes、监控告警
- **职责**：部署自动化、基础设施即代码、故障排查
- **工作范围**：`infrastructure/`, `.github/workflows/`, `k8s/`
- **参考**：`~/.claude/agents/devops-troubleshooter.md`

#### 6. Full-Stack Integration Specialist 🔗
- **专长**：前后端集成、API 对接、数据流设计
- **职责**：确保前后端无缝协作、数据一致性
- **工作范围**：跨前后端的集成层

#### 💡 **如何扩展和自定义 Subagent 角色**

##### 1. 使用现有角色模板

`~/.claude/agents/` 目录提供了丰富的预定义角色模板：

- **Performance Engineer**: 性能优化、缓存策略（参考 `performance-engineer.md`）
- **Cloud Architect**: 云架构设计、多云部署（参考 `cloud-architect.md`）
- **Data Engineer**: 数据管道、ETL 流程（参考 `data-engineer.md`）
- **Mobile Developer**: iOS/Android 原生开发（参考 `ios-developer.md`）
- **Blockchain Developer**: Web3 集成（参考 `blockchain-developer.md`）
- **AI Engineer**: LLM 集成、RAG 系统（参考 `ai-engineer.md`）

##### 2. 自定义 Subagent 角色（当现有角色不满足需求时）

**场景示例**：
- 电商项目需要"支付集成专家"
- 社交平台需要"实时推送服务专家"
- IoT 项目需要"设备协议适配专家"

**创建步骤**：

**Step 1**: 定义角色属性

在 `docs/agents/` 创建新文件，例如 `payment-integration-specialist.md`：

```markdown
---
name: payment-integration-specialist
description: 支付集成专家，精通 Stripe/PayPal/微信支付等多种支付方式
model: opus  # 或 sonnet，根据复杂度选择
---

## 角色定义

### 专长领域
- Stripe API 集成和 Webhook 处理
- PayPal Express Checkout 和订阅支付
- 微信支付/支付宝支付集成
- PCI-DSS 合规性要求
- 支付安全和反欺诈

### 核心职责
- 设计支付流程和状态机
- 实现支付回调和异步通知处理
- 处理退款、争议和对账逻辑
- 确保支付数据安全和合规性

### 工作范围
- `src/payments/` - 支付服务实现
- `src/webhooks/` - 支付回调处理
- `src/billing/` - 账单和对账系统

### 依赖知识
- 理解 Idempotency（幂等性）设计
- 熟悉分布式事务和最终一致性
- 掌握 Webhook 安全验证（签名校验）
- 了解货币精度处理（避免浮点数误差）

### 输出规范
- 必须包含完整的错误处理（网络超时、支付失败、重复支付等）
- 必须添加详细的支付状态流转日志
- 必须实现 Webhook 重试机制
- 必须考虑并发场景下的幂等性保护

### 测试要求
- 单元测试覆盖率 > 90%
- 集成测试覆盖核心支付流程
- 使用 Stripe Test Mode 验证功能
- 模拟各种异常场景（超时、拒绝、取消等）
```

**Step 2**: 在项目中引用

在 `SUBAGENT_DEVELOPMENT_METHODOLOGY.md` 的角色列表中添加：

```markdown
#### 7. Payment Integration Specialist 💳
- **专长**：Stripe/PayPal/微信支付集成
- **职责**：支付流程设计、Webhook 处理、退款和对账
- **工作范围**：`src/payments/`, `src/webhooks/`
- **参考**：`docs/agents/payment-integration-specialist.md`
```

**Step 3**: 创建任务文档时使用

```markdown
# Task 3a: Stripe 支付集成

## Role
Payment Integration Specialist

## Objective
实现 Stripe 支付完整流程，包括创建订单、处理支付、Webhook 回调

## Context
- 项目使用 Stripe 作为主要支付方式
- 需要支持一次性支付和订阅支付
- 必须通过 PCI-DSS 合规性检查

## Requirements
1. 实现 /api/payments/create-checkout-session 接口
2. 实现 Webhook 接收和验证（/webhooks/stripe）
3. 支付成功后更新订单状态
4. 添加支付失败重试机制
5. 实现幂等性保护（防止重复扣款）

## Output Location
docs/outputs/phase3/output-3a-stripe-payment.md
```

##### 3. 自定义 Subagent 的最佳实践

**原则**：
1. ✅ **职责单一**：每个 Subagent 只负责一个明确的领域
2. ✅ **边界清晰**：定义明确的工作范围，避免职责重叠
3. ✅ **可测试性**：输出必须包含测试用例
4. ✅ **文档完整**：包含完整的接口设计和实现说明
5. ✅ **可复用性**：设计时考虑后续任务可能的复用场景

**何时创建新角色 vs 使用现有角色**：

| 场景 | 建议 |
|------|------|
| 标准的前端/后端/数据库任务 | 使用现有通用角色 |
| 特定技术栈（如 Stripe、Kafka、Redis） | 创建专门角色 |
| 领域特定逻辑（支付、推荐算法） | 创建领域专家角色 |
| 一次性简单任务 | 使用通用角色即可 |
| 需要深度领域知识的复杂模块 | 必须创建专门角色 |

##### 4. 动态调整角色

随着项目演进，角色定义应该不断优化：

```markdown
## 角色演进记录

### v1.0 (2025-01-15)
- 初始定义：基础支付集成

### v1.1 (2025-02-01)
- 新增职责：订阅管理和周期性扣款
- 新增职责：多货币支持

### v1.2 (2025-03-10)
- 拆分角色：将"对账系统"独立为新角色
- 原因：对账逻辑变得复杂，需要独立处理
```

**总结**：
- 优先使用 `~/.claude/agents/` 中的标准角色
- 根据项目特殊需求自定义专业角色
- 保持角色定义的清晰性和可维护性
- 随项目演进持续优化角色定义

#### 7. TDD Red-Phase Specialist 🔴

**专长**：测试驱动开发，测试用例设计

**核心职责**：
- 编写失败的测试用例来明确功能需求
- 从使用者角度设计清晰的 API
- 覆盖正常、边界、异常场景
- 使用 AAA 模式（Arrange-Act-Assert）

**工作流程**：
1. 理解需求 → 提取核心功能点
2. 设计测试用例 → 编写测试代码
3. 验证测试失败 → 确保测试有效
4. 文档输出 → 记录测试用例和接口设计

**最佳实践**：
- ✅ 测试名称清晰描述行为
- ✅ 一个测试只验证一个行为
- ✅ 测试独立性（不依赖其他测试）
- ✅ 覆盖边界条件和异常情况
- ❌ 一次写太多测试
- ❌ 测试之间相互依赖

**Token 预算**：~23k tokens

#### 8. TDD Green-Phase Developer 🟢

**专长**：最小化实现，快速让测试通过

**核心职责**：
- 用最简单的代码让测试从红色变为绿色
- 允许硬编码和"笨"方法
- 专注当前测试，避免过度设计
- 快速迭代，频繁运行测试

**实现策略**：
1. **硬编码优先（Fake It）**：直接返回测试期望的值
2. **明显实现（Obvious Implementation）**：如果实现很简单，直接写出来
3. **三角法（Triangulation）**：通过多个测试用例"三角定位"正确实现

**工作流程**：
1. 理解测试 → 查看测试文件和 RED 阶段文档
2. 最小化实现 → 用最简单方式让测试通过
3. 逐步完善 → 处理不同输入和边界情况
4. 验证所有测试 → 确保全部通过
5. 文档输出 → 记录实现方式和临时方案

**最佳实践**：
- ✅ 从最简单的测试开始
- ✅ 频繁运行测试（使用 watch 模式）
- ✅ 先硬编码，再泛化
- ✅ 允许重复代码（REFACTOR 阶段会消除）
- ❌ 过度设计和预先优化
- ❌ 添加测试未要求的功能
- ❌ 跳过失败的测试

**Token 预算**：~23k tokens

#### 9. TDD Refactor Specialist 🔧

**专长**：代码重构，设计优化

**核心职责**：
- 在保持所有测试通过的前提下优化代码质量
- 消除重复代码（DRY 原则）
- 提升可读性和可维护性
- 应用设计模式和最佳实践
- 优化性能

**重构技巧**：
1. **提取方法（Extract Method）**：将长方法拆分为小方法
2. **消除重复（Remove Duplication）**：提取公共逻辑
3. **引入策略模式**：替换条件分支
4. **替换魔法数字**：使用命名常量
5. **简化条件表达式**：提高可读性

**工作流程**：
1. 评估当前代码 → 识别可改进的地方
2. 制定重构计划 → 优先级排序
3. 逐步重构 → 小步前进，频繁测试
4. 验证质量 → 运行测试、检查覆盖率
5. 文档输出 → 记录改进点和质量指标

**代码质量检查**：
- 消除代码异味（长方法、重复代码、魔法数字等）
- 遵循 SOLID 原则
- 提升测试覆盖率
- 降低代码复杂度

**最佳实践**：
- ✅ 小步重构，每次只改一个地方
- ✅ 频繁运行测试确保安全
- ✅ 提交前后对比质量指标
- ✅ 保持或提升测试覆盖率
- ❌ 不要改变功能行为
- ❌ 不要跳过测试
- ❌ 不要一次改太多
- ❌ 不要过度重构

**Token 预算**：~23k tokens

---

## TDD 工作流程

### 什么是测试驱动开发 (TDD)？

TDD 是一种软件开发过程，核心理念是**"先写测试，再写实现"**。开发者首先编写一个简短的、描述新功能或改进的自动化测试用例，这个测试在最初必然是失败的。然后，开发者编写最少的代码来让这个测试通过。最后，对新代码进行重构，以符合编码规范。

### TDD 核心流程：红-绿-重构 (Red-Green-Refactor)

```
┌─────────────────────────────────────────────────────────┐
│                    TDD 循环流程                          │
│                                                         │
│   🔴 RED                                               │
│   写一个失败的测试                                        │
│   ↓                                                     │
│   - 明确功能需求和期望行为                                │
│   - 测试必须失败（证明功能尚未实现）                        │
│   - 思考输入、输出和边界条件                               │
│                                                         │
│   🟢 GREEN                                             │
│   写最少代码让测试通过                                     │
│   ↓                                                     │
│   - 快速实现，不求完美                                    │
│   - 只关注让测试变绿                                      │
│   - 避免过度设计                                         │
│                                                         │
│   🔧 REFACTOR                                          │
│   优化代码质量                                           │
│   ↓                                                     │
│   - 消除重复代码                                         │
│   - 提高可读性和可维护性                                  │
│   - 优化设计和性能                                       │
│   - 确保测试依然通过                                     │
│                                                         │
│   ↻ 回到 RED，开始下一个功能                             │
└─────────────────────────────────────────────────────────┘
```

#### 阶段 1: 红色 (Red) - 写一个失败的测试

**目标**: 明确你想要实现的功能是什么

**操作**:
- 针对一个极小的功能点，编写自动化测试
- 因为还没有实现代码，测试必然失败（红色）
- 测试失败可能是编译错误或断言失败

**意义**:
- 强迫思考需求和功能细节
- 明确输入、输出和期望行为
- 建立清晰的成功标准

**示例**:
```typescript
// tests/user-auth.test.ts
describe('UserAuthService', () => {
  it('should authenticate user with valid credentials', async () => {
    const authService = new UserAuthService();
    const result = await authService.login('user@example.com', 'password123');

    expect(result).toBeDefined();
    expect(result.token).toBeTruthy();
    expect(result.user.email).toBe('user@example.com');
  });
});

// 运行测试 → ❌ 失败（UserAuthService 类还不存在）
```

#### 阶段 2: 绿色 (Green) - 写最少代码让测试通过

**目标**: 快速实现功能，满足测试要求

**操作**:
- 编写刚好能让测试通过的代码
- 不追求完美，允许"笨"方法
- 关键在于"最少" - 只做必要的事

**意义**:
- 专注于当前唯一目标：满足测试
- 避免过度工程和偏离需求
- 快速获得反馈

**示例**:
```typescript
// src/user-auth-service.ts
export class UserAuthService {
  async login(email: string, password: string): Promise<AuthResult> {
    // 最简单的实现 - 先让测试通过
    return {
      token: 'mock-jwt-token-12345',  // 硬编码一个 token
      user: { email }
    };
  }
}

// 运行测试 → ✅ 通过（虽然是硬编码，但测试通过了）
```

#### 阶段 3: 重构 (Refactor) - 优化代码

**目标**: 在不改变外部行为的前提下提升代码质量

**操作**:
- 有测试作为"安全网"，可以放心重构
- 消除重复代码
- 提高可读性和可维护性
- 优化算法和设计
- 每次修改后重新运行测试

**意义**:
- 确保代码长期可维护
- 提升代码质量和性能
- 测试保护让你敢于重构

**示例**:
```typescript
// src/user-auth-service.ts
import { hash, compare } from 'bcrypt';
import { sign } from 'jsonwebtoken';
import { UserRepository } from './user-repository';

export class UserAuthService {
  private userRepo: UserRepository;
  private jwtSecret: string;

  constructor(userRepo: UserRepository, jwtSecret: string) {
    this.userRepo = userRepo;
    this.jwtSecret = jwtSecret;
  }

  async login(email: string, password: string): Promise<AuthResult> {
    const user = await this.findUserByEmail(email);
    await this.validatePassword(password, user.passwordHash);

    return {
      token: this.generateToken(user),
      user: { email: user.email, id: user.id }
    };
  }

  private async findUserByEmail(email: string) {
    const user = await this.userRepo.findByEmail(email);
    if (!user) {
      throw new Error('User not found');
    }
    return user;
  }

  private async validatePassword(password: string, hash: string) {
    const isValid = await compare(password, hash);
    if (!isValid) {
      throw new Error('Invalid password');
    }
  }

  private generateToken(user: User): string {
    return sign({ userId: user.id, email: user.email }, this.jwtSecret, {
      expiresIn: '24h'
    });
  }
}

// 运行测试 → ✅ 依然通过（重构成功！）
```

### TDD 与 Subagent 方法论的整合

在 Subagent 并发开发方法论中，TDD 可以通过专门的 Subagent 角色来实现：

```
┌─────────────────────────────────────────────────────────┐
│           Controller (Sequential Thinking)              │
│         分析需求 → 分解为 TDD 任务 → 分配角色             │
└─────────────────────────────────────────────────────────┘
                         │
          ┌──────────────┼──────────────┐
          │              │              │
    ┌─────▼─────┐  ┌─────▼─────┐  ┌────▼──────┐
    │  Red 🔴   │→ │ Green 🟢  │→ │Refactor🔧 │
    │  写测试    │  │  写实现    │  │  优化代码  │
    └───────────┘  └───────────┘  └───────────┘
          │              │              │
          └──────────────┴──────────────┘
                         │
                   ↻ 循环迭代
```

#### TDD 任务文档模板

```markdown
# Task [ID]: [功能名称] - TDD 循环

## Phase: RED 🔴
### Role
TDD Red-Phase Specialist

### Objective
为 [功能名称] 编写失败的测试用例

### Requirements
1. 编写测试用例描述预期行为
2. 确保测试失败（功能未实现）
3. 覆盖正常情况和边界情况
4. 测试应该清晰、可读

### Output Location
docs/outputs/tdd/[feature]-red-phase.md

---

## Phase: GREEN 🟢
### Role
TDD Green-Phase Developer

### Objective
编写最少代码让测试通过

### Input
- 测试文件：tests/[feature].test.ts
- RED 阶段输出：docs/outputs/tdd/[feature]-red-phase.md

### Requirements
1. 实现功能让所有测试通过
2. 代码尽可能简单
3. 不追求完美设计
4. 确保测试覆盖率

### Output Location
docs/outputs/tdd/[feature]-green-phase.md

---

## Phase: REFACTOR 🔧
### Role
TDD Refactor Specialist

### Objective
优化代码质量，保持测试通过

### Input
- GREEN 阶段输出：docs/outputs/tdd/[feature]-green-phase.md
- 测试文件：tests/[feature].test.ts

### Requirements
1. 消除重复代码
2. 提高可读性
3. 优化性能和设计
4. 确保所有测试依然通过

### Output Location
docs/outputs/tdd/[feature]-refactor-phase.md
```

### TDD 并发执行策略

对于多个独立功能，可以并发执行多个 TDD 循环：

```
功能 A: RED → GREEN → REFACTOR  ─┐
功能 B: RED → GREEN → REFACTOR  ─┤
功能 C: RED → GREEN → REFACTOR  ─┼→ Controller 整合
功能 D: RED → GREEN → REFACTOR  ─┘

每个功能独立的 TDD 循环，互不干扰
```

### TDD 最佳实践

#### ✅ DO（推荐做法）

**RED 阶段**:
- 一次只测试一个小功能点
- 测试名称清晰描述行为
- 确保测试真的失败（不是假阳性）
- 测试要简单、快速、独立

**GREEN 阶段**:
- 写最简单能通过的代码
- 允许硬编码和"笨"方法
- 不要跳跃式实现多个功能
- 快速获得绿色测试

**REFACTOR 阶段**:
- 频繁运行测试确保安全
- 小步重构，每次一个改进
- 消除重复（DRY 原则）
- 提取方法、类、模块

#### ❌ DON'T（避免做法）

**RED 阶段**:
- 一次写太多测试
- 测试之间相互依赖
- 测试名称模糊不清
- 忘记运行测试确认失败

**GREEN 阶段**:
- 过度设计和预先优化
- 实现超出测试要求的功能
- 跳过测试直接写实现
- 修改测试让其通过

**REFACTOR 阶段**:
- 修改功能行为
- 在没有测试保护下重构
- 大规模重构不分步
- 重构时添加新功能

### TDD 的优势

1. **需求明确**: 测试即文档，清晰描述功能
2. **设计驱动**: 先考虑接口，促进良好设计
3. **快速反馈**: 立即知道代码是否正确
4. **回归保护**: 测试防止破坏现有功能
5. **重构信心**: 有测试保护，敢于改进代码
6. **减少调试**: 问题早发现，调试时间少
7. **文档化**: 测试是最好的使用示例

---

## Token 控制策略

### 五层防护机制

#### 1️⃣ 任务粒度控制
```yaml
规则:
  - 每个任务只修改 1-2 个文件
  - 每个文件限制在 200-300 行
  - 明确输入和输出范围

Token 估算:
  任务文档: ~5k
  读取代码: ~6k (1-2个文件)
  生成代码: ~10k
  测试说明: ~3k
  总计: ~25k tokens
```

#### 2️⃣ 输入文档化
```markdown
# 任务文档模板

## Role
[Subagent 角色名称]

## Objective
[简短的任务目标，1-2 句话]

## Input Files
- path/to/file1.ts (line 100-150)
- path/to/file2.ts (specific function)

## Requirements
1. 具体要求 1
2. 具体要求 2
3. 具体要求 3

## Output Location
docs/outputs/[phase]/output-[id]-[name].md

## Token Budget
预计：25k tokens
```

**优势**：
- Subagent 读取文档（~3k tokens），而不是在 prompt 中包含大量代码
- Controller 创建文档时可以精确控制范围

#### 3️⃣ 输出文档化
```markdown
# 输出文档模板

## Modified Files
- path/to/file1.ts
- path/to/file2.ts

## Implementation
[核心代码改动或完整实现]

## Testing Results
- ✅ 单元测试通过
- ✅ 集成测试通过
- ⏱️ 性能指标

## Usage
[代码使用示例]

## Next Steps
- 建议 1
- 建议 2
```

**优势**：
- 后续任务只需读取输出文档（~5k tokens）
- 避免重新执行所有历史操作
- 成为可复用的知识库

#### 4️⃣ 增量式开发
```
任务 1a → 输出 1a (25k)
              ↓
任务 1b (读取输出 1a，5k) → 输出 1b (25k)
                                  ↓
任务 1c (读取输出 1b，5k) → 输出 1c (25k)
```

**关键**：每个新任务只读取前一个的输出文档，不重新生成

#### 5️⃣ 并发执行隔离
```
任务 1a (用户认证)   → 输出 1a  ─┐
任务 1b (数据库设计)  → 输出 1b  ─┤
任务 1c (前端UI)     → 输出 1c  ─┼→ Controller 整合 → 决策
任务 1d (API安全)    → 输出 1d  ─┘

每个任务独立上下文：25k × 4 = 100k tokens
```

---

## 文档化工作流

### 任务文档（Task Document）

**作用**：给 Subagent 的详细指令

**标准格式**：
```markdown
# Task [ID]: [任务名称]

## Role
[Subagent 角色名称]

## Objective
[1-2 句话的任务目标]

## Context
[必要的背景信息]

## Input Files
- src/path/to/file1.ts (line 100-150)
- src/path/to/file2.ts (function: functionName)

## Requirements
1. 功能要求 1
2. 功能要求 2
3. 非功能要求（性能、安全等）
4. **⚠️ 必须添加终端调试日志**（函数入口、出口、关键分支、错误处理）
5. **⚠️ 前端代码日志必须输出到终端**（不仅是浏览器控制台）

## Constraints
- 保持与现有接口兼容
- 遵循项目代码规范
- 添加必要的错误处理
- 所有日志使用统一格式：`[级别] [模块名] 消息内容`

## Output Location
docs/outputs/[phase]/output-[id]-[name].md

## Token Budget
预计：25k tokens

## Success Criteria
- [ ] 功能实现完整
- [ ] 所有关键节点添加了日志输出
- [ ] 运行项目后终端日志无错误
- [ ] 测试通过（包括终端日志验证）
- [ ] 文档完善
```

### 输出文档（Output Document）

**作用**：Subagent 完成任务后的交付成果

**标准格式**：
```markdown
# Output [ID]: [任务名称] - 实现报告

## Status
✅ 已完成 / ⚠️ 部分完成 / ❌ 失败

## Modified Files
- src/path/to/file1.ts
- src/path/to/file2.ts

## Implementation

### 核心改动
[关键代码片段或完整实现]

### 设计决策
- 决策 1：[原因]
- 决策 2：[原因]

## Testing

### 单元测试
```typescript
[测试代码]
```

### 测试结果
- ✅ 测试 A 通过
- ✅ 测试 B 通过
- ⏱️ 性能：平均 150ms

### 终端日志验证
**项目运行测试**：
```bash
# 运行命令
npm run dev

# 终端输出（关键日志）
[INFO] [ModuleName] Function started with params: {...}
[INFO] [ModuleName] Step 1 completed
[INFO] [ModuleName] Step 2 completed
[INFO] [ModuleName] SUCCESS - Result: {...}
```

**验证结果**：
- ✅ 项目成功启动，无启动错误
- ✅ 终端日志无 ERROR 输出
- ✅ 所有关键操作都有日志记录
- ✅ 错误处理路径日志完整
- ✅ 前端日志在终端可见（非仅浏览器控制台）
- ✅ 连续 3 次运行无问题

**日志覆盖情况**：
- [✅] 函数入口日志
- [✅] 关键分支日志
- [✅] 外部调用日志（API/RPC/DB）
- [✅] 错误处理日志（含堆栈）
- [✅] 函数出口日志

## Usage

### 基本用法
```typescript
[代码示例]
```

### 高级用法
[更复杂的示例]

## Dependencies
- 新增依赖：[package@version]
- 依赖的其他模块：[module names]

## Next Steps
- 建议 1：[具体描述]
- 建议 2：[具体描述]

## Issues & Limitations
- 已知问题 1
- 限制 1

## Token Usage
实际使用：23k tokens (预算：25k)
```

### 进度文档（Progress Document）

**作用**：跟踪整个 Phase 的进度

**标准格式**：
```markdown
# Phase 1: Shadow Mode - 进度跟踪

## 总体状态
🔄 进行中 | 完成度：60%

## 任务组 A（并发执行）

### Task 1a: 用户认证 API
- 状态：✅ 已完成
- Subagent：Backend Architect
- 完成时间：2025-10-01
- 输出：docs/outputs/phase1/output-1a-auth-api-implementation.md

### Task 1b: 数据库 Schema 设计
- 状态：✅ 已完成
- Subagent：Database Architect
- 完成时间：2025-10-01
- 输出：docs/outputs/phase1/output-1b-database-schema.md

### Task 1c: 前端登录组件
- 状态：🔄 进行中
- Subagent：Frontend Developer
- 预计完成：2025-10-02

### Task 1d: API 安全审计
- 状态：📅 待开始
- Subagent：API Security Auditor

## 决策记录
- [2025-10-01] 决定使用 JWT 认证方案（24小时过期）
- [2025-10-01] 密码强度要求：至少8位，包含大小写字母和数字

## 风险和问题
- ⚠️ Session 管理方案待定（Redis vs Database）
- ⚠️ 需要考虑 CSRF 保护措施

## 下一步行动
1. 完成任务组 A 的剩余任务
2. 开始任务组 B（用户权限、密码重置）
3. 集成前后端并进行端到端测试
```

---

## 完整工作流程

### Phase 0: MCP 环境准备

#### Step 1: 安装和配置 Sequential Thinking MCP

**快速安装（推荐）**：

使用 Claude CLI 一键安装：

```bash
claude mcp add-json "sequential-thinking" '{"command":"npx","args":["-y","@modelcontextprotocol/server-sequential-thinking"]}'
```

**手动安装（备用方案）**：

```typescript
// 1. 检查 MCP 是否已安装
async function checkAndInstallMCP() {
  try {
    // 读取 MCP 配置文件
    const mcpConfig = await readFile('C:/Users/YOUR_USERNAME/AppData/Roaming/Code/User/globalStorage/saoudrizwan.claude-dev/settings/cline_mcp_settings.json');
    const config = JSON.parse(mcpConfig);

    // 检查 sequential-thinking 是否已配置
    if (!config.mcpServers || !config.mcpServers['sequential-thinking']) {
      console.log('[INFO] Sequential Thinking MCP 未安装，开始安装...');

      // 更新配置文件
      config.mcpServers = config.mcpServers || {};
      config.mcpServers['sequential-thinking'] = {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
      };

      await writeFile('cline_mcp_settings.json', JSON.stringify(config, null, 2));
      console.log('[SUCCESS] Sequential Thinking MCP 已安装并配置');
    } else {
      console.log('[INFO] Sequential Thinking MCP 已存在');
    }
  } catch (error) {
    console.error('[ERROR] MCP 安装失败:', error);
  }
}
```

**MCP 配置文件示例**：
```json
{
  "mcpServers": {
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
    }
  }
}
```

**验证安装**：

安装完成后，重启 Claude Code 并验证 MCP 是否可用：

```bash
# 在 Claude Code 中执行
# 查看 MCP 工具列表，应该能看到 create_thinking_process 等工具
```

#### Step 2: 使用 MCP 进行任务规划

**工作流程**：
```
复杂需求
  ↓
调用 Sequential Thinking MCP
  ↓
MCP 分析和拆分任务
  ↓
识别任务依赖关系
  ↓
生成任务执行计划
  ↓
Controller 根据计划并发启动 Subagent
```

**使用示例**：
```typescript
// Controller 使用 MCP 分析任务
// 输入：复杂的开发需求
const requirement = "实现完整的电商系统，包括用户管理、商品管理、订单处理和支付集成";

// MCP 分析并拆分任务
思考 1：识别主要模块
  - 用户认证和授权模块
  - 商品目录和搜索模块
  - 购物车和订单模块
  - 支付和结算模块
  - 后台管理模块

思考 2：分析模块间依赖
  - 用户认证 → 独立，可并发（所有模块的前置依赖）
  - 商品目录 → 依赖用户认证
  - 购物车 → 依赖用户认证和商品目录
  - 订单处理 → 依赖购物车
  - 支付集成 → 依赖订单处理

思考 3：划分任务批次
  批次 1（并发）：用户认证 + 数据库设计 + 前端基础框架
  批次 2（并发）：商品目录 API + 商品前端页面
  批次 3（并发）：购物车功能 + 订单系统
  批次 4（串行）：支付集成 + 安全审计

思考 4：生成任务文档
  - Task 1a: 用户认证 API 开发
  - Task 1b: 数据库 Schema 设计
  - Task 1c: React 前端框架搭建
  - Task 2a: 商品管理 API
  - Task 2b: 商品展示页面
  - Task 3a: 购物车功能实现
  - Task 3b: 订单处理流程
  - Task 4a: Stripe 支付集成
  - Task 4b: API 安全审计

// 输出：结构化的任务计划
```

#### Step 3: 验证 MCP 功能

**验证清单**：
- [ ] MCP 配置文件已正确创建
- [ ] Sequential Thinking MCP 可以正常调用
- [ ] MCP 能够分析复杂任务
- [ ] MCP 能够识别任务依赖关系
- [ ] MCP 能够生成并发任务计划

### Phase 1: 准备阶段

#### Step 1: 创建 Subagent 角色定义
```bash
docs/agents/
├── frontend-developer.md
├── backend-architect.md
├── database-architect.md
├── api-security-auditor.md
├── devops-engineer.md
├── tdd-red-phase-specialist.md
├── tdd-green-phase-developer.md
└── tdd-refactor-specialist.md
```

#### Step 2: 任务分解
Controller（我）使用 Sequential Thinking 分析项目，分解为小任务：
```
Phase 1: 用户管理模块
  ├── 任务组 A（并发）
  │   ├── Task 1a: 用户认证 API
  │   ├── Task 1b: 数据库 Schema 设计
  │   ├── Task 1c: 前端登录组件
  │   └── Task 1d: API 安全审计
  └── 任务组 B（依赖 A）
      ├── Task 2a: 权限管理系统
      ├── Task 2b: 用户资料页面
      └── Task 2c: 密码重置功能
```

### Phase 2: 执行阶段

#### Step 1: 创建任务文档
Controller 使用 Write tool 创建：
```
docs/tasks/phase1/task-1a-user-auth-api.md
docs/tasks/phase1/task-1b-database-schema-design.md
docs/tasks/phase1/task-1c-frontend-login-ui.md
docs/tasks/phase1/task-1d-api-security-audit.md
```

#### Step 2: 并发启动 Subagent

**⚠️ 关键原则：必须并行多开 Subagent 进行并发开发**

Controller 在**单个消息**中使用 Task tool 启动 4 个 Subagent：

**并行开发规则**：
1. **强制并发**：所有独立任务必须在同一个消息中启动，不允许串行执行
2. **无数量限制**：并发 Subagent 数量无上限，根据实际任务数量决定
3. **依赖检查**：只有存在依赖关系的任务才能串行，其他必须并发
4. **效率优先**：并发开发效率与并发数量成正比（N 个任务 = N 倍提升）

**并发启动示例**：
```typescript
// ✅ 正确做法：在单个消息中启动所有独立任务（无数量限制）
Controller 发送单个消息，包含 N 个并发 Task tool 调用：

// 小规模并发（4 个任务）
Task 1-4: Mainnet Expert, Monitoring Engineer, Testing Architect...
  - 每个任务独立执行
  - 同时启动，同时完成

// 中规模并发（8 个任务）
Task 1-8: 数据集成 × 3, 风险管理 × 2, 监控 × 2, 部署 × 1
  - 所有独立任务一次性启动
  - 效率提升 8 倍

// 大规模并发（16+ 个任务）
Task 1-16+: 多模块、多功能并行开发
  - 前端组件 × 4
  - 后端 API × 4
  - 数据集成 × 4
  - 测试用例 × 4
  - 效率提升 16+ 倍

// ❌ 错误做法：串行启动独立任务
// 不要这样做：启动 Task 1 → 等待完成 → 启动 Task 2 → 等待完成...
```

**Token 使用**：N × 25k tokens
- 4 个任务：100k tokens
- 8 个任务：200k tokens
- 16 个任务：400k tokens
- 只要在 1M tokens 预算内，可以并发任意数量的任务

**并发效率对比**：
- 串行执行：N 个任务 × 每个 30 分钟 = 30N 分钟
- 并发执行：N 个任务同时进行 = 30 分钟
- **效率提升**：N 倍（无上限）

**实际案例**：
- 4 个任务并发：120 分钟 → 30 分钟（4 倍提升）
- 8 个任务并发：240 分钟 → 30 分钟（8 倍提升）
- 16 个任务并发：480 分钟 → 30 分钟（16 倍提升）

**并发开发决策流程**：
```
使用 Sequential Thinking MCP 分析任务依赖：
  ↓
识别可并发任务组（无依赖关系）
  ↓
在单个消息中启动所有并发任务
  ↓
等待所有任务完成
  ↓
收集和整合结果
  ↓
识别下一批可并发任务
  ↓
重复循环
```

### Phase 3: 整合阶段

#### Step 1: 收集输出文档

⚠️ **Controller（主窗口）只读取输出文档，不查看源代码**

Controller 使用 Read tool 读取所有 Subagent 的输出文档：
```bash
# 读取输出文档（而非源代码）
Read docs/outputs/phase1/output-1a-auth-api-implementation.md
Read docs/outputs/phase1/output-1b-database-schema.md
Read docs/outputs/phase1/output-1c-frontend-login-ui.md
Read docs/outputs/phase1/output-1d-api-security-audit.md
```

**为什么只读输出文档？**
- ✅ 输出文档已包含实现摘要和关键信息（~5k tokens）
- ✅ 避免读取大量源代码（可能 20k-50k tokens）
- ✅ 保持主窗口 token 消耗在低位
- ✅ 快速了解任务完成情况和下一步行动

#### Step 2: 更新进度文档

⚠️ **每次 Subagent 完成任务后必须立即更新进度**

Controller 更新 `docs/progress/phase1-progress.md`：

```markdown
# Phase 1: 用户管理模块 - 进度跟踪

## 总体状态
🔄 进行中 | 完成度：75% (3/4 任务完成)
**最后更新**: 2025-10-07 14:30

## 任务组 A（并发执行）

### ✅ Task 1a: 用户认证 API
- 状态：✅ 已完成
- Subagent：Backend Architect
- 完成时间：2025-10-07 10:00
- 输出文档：docs/outputs/phase1/output-1a-auth-api-implementation.md
- 关键成果：
  - POST /api/auth/register 注册接口
  - POST /api/auth/login 登录接口
  - JWT token 生成（24h 有效期）
  - bcrypt 密码加密
- Token 使用：23k

### ✅ Task 1b: 数据库 Schema 设计
- 状态：✅ 已完成
- Subagent：Database Architect
- 完成时间：2025-10-07 10:00
- 输出文档：docs/outputs/phase1/output-1b-database-schema.md
- 关键成果：
  - User 表设计（id, email, passwordHash, createdAt）
  - 索引策略（email unique index）
  - Prisma migration 文件
- Token 使用：20k

### ✅ Task 1c: 前端登录组件
- 状态：✅ 已完成
- Subagent：Frontend Developer
- 完成时间：2025-10-07 14:00
- 输出文档：docs/outputs/phase1/output-1c-frontend-login-ui.md
- 关键成果：
  - LoginForm 组件
  - 表单验证（email 格式、密码长度）
  - API 调用集成
  - 错误提示 UI
- Token 使用：25k

### 📅 Task 1d: API 安全审计
- 状态：📅 待开始
- Subagent：API Security Auditor
- 计划开始：2025-10-07 15:00

## 决策记录
- [2025-10-07 10:00] 采用 Prisma ORM 管理数据库
- [2025-10-07 10:00] JWT secret 存储在环境变量
- [2025-10-07 14:00] 前端使用 React Hook Form 表单管理

## 风险和问题
- ⚠️ Session 管理方案待定（建议使用 Redis）
- ⚠️ CSRF 保护未实现（Task 1d 将处理）

## 下一步行动
1. ⏳ 启动 Task 1d: API 安全审计
2. 📝 规划 Phase 2: 用户权限系统
3. 🔗 进行前后端集成测试
```

**进度更新最佳实践**：
- ✅ 每次 Subagent 完成立即更新（而非批量更新）
- ✅ 记录完成时间和 Token 使用情况
- ✅ 摘要关键成果和输出文档路径
- ✅ 更新总体完成度百分比
- ✅ 记录决策和遇到的问题
- ✅ 明确下一步行动计划

#### Step 3: Sequential Thinking 分析下一步

⚠️ **使用 Sequential Thinking MCP 分析和规划，而非凭直觉决策**

Controller 使用 Sequential Thinking MCP 工具分析当前状态：

```typescript
// 调用 Sequential Thinking MCP
思考 1：检查所有任务是否成功完成
  - Task 1a, 1b, 1c: ✅ 完成
  - Task 1d: 📅 待开始
  - 评估：Phase 1 接近完成

思考 2：评估代码质量和集成可行性
  - 输出文档显示所有接口符合 RESTful 规范
  - 前后端 API contract 一致
  - 数据库 schema 满足认证需求
  - 评估：质量良好，可以集成测试

思考 3：识别潜在问题和风险
  - Session 管理方案未定 → 需要在 Phase 2 处理
  - CSRF 保护未实现 → Task 1d 将审计并添加
  - 单元测试覆盖率未知 → 需要确认测试情况

思考 4：规划下一批任务
  批次选项：
  A. 完成 Task 1d 后立即进入 Phase 2
  B. 先进行集成测试，确保 Phase 1 完全稳定

  决策：选择 B（先稳固再拓展）

思考 5：下一批任务拆分
  并发任务组：
  - Task 1d: API 安全审计（独立）
  - Task 1e: 集成测试编写（可并发）
  - Task 1f: E2E 测试场景（可并发）
```

**MCP 分析输出**：
- 下一步行动：并发启动 Task 1d, 1e, 1f
- 预计完成时间：30 分钟（并发执行）
- 风险评估：低风险，可以安全推进

**MCP 安装检查和使用**：
```typescript
// 1. 检查 sequentialthinking MCP 是否已安装
// 如果没有安装，执行以下步骤：

// 安装 sequentialthinking MCP
npx @modelcontextprotocol/create-server sequentialthinking

// 2. 配置 MCP 服务器（在 cline_mcp_settings.json 中）
{
  "mcpServers": {
    "sequentialthinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/sequentialthinking"]
    }
  }
}

// 3. 使用 MCP 进行任务规划
// Controller 调用 sequentialthinking MCP 来分析和拆分复杂任务
```

#### Step 3: 更新进度文档
Controller 更新 `docs/progress/phase1-progress.md`

#### Step 4: 决策下一批任务
- 如果任务组 A 全部成功 → 启动任务组 B
- 如果有失败任务 → 分析原因，重新分配
- 如果需要调整 → 创建补充任务

### Phase 4: 迭代

重复 Phase 2-3，直到整个项目完成。

---

## 优点总结

### ✅ 1. Token 使用高效
- **精确控制**：每个任务 <30k tokens，远低于 50% 限制
- **无累积**：增量式开发避免上下文膨胀
- **最佳性能**：在 Claude 的有效上下文范围内（60k-120k）

### ✅ 2. 并发执行高效
- **真正并发**：无限制的 Subagent 同时工作
- **互不干扰**：每个 Subagent 有独立上下文窗口
- **时间节省**：相比串行开发，效率提升 = 并发任务数（无上限）
- **大规模并行**：支持 10+、20+、甚至 50+ 任务同时执行

### ✅ 3. 知识可复用
- **输出文档**：成为项目知识库
- **任务模板**：相似任务可复用
- **最佳实践**：成功经验文档化

### ✅ 4. 质量可控
- **专业分工**：每个 Subagent 专注其擅长领域
- **职责单一**：降低错误率
- **可追溯**：完整的任务 → 输出链

### ✅ 5. 易于协作
- **文档化**：团队成员可查阅所有任务和输出
- **标准化**：统一的文档格式
- **透明度**：进度和决策可见

### ✅ 6. 容错性强
- **隔离失败**：单个任务失败不影响其他
- **易于重试**：任务文档可重新执行
- **渐进式**：分阶段验证

### ✅ 7. 可扩展性
- **模块化**：新任务遵循相同模式
- **角色扩展**：可添加新的 Subagent 角色
- **项目迁移**：方法论可应用于其他项目

### ✅ 8. 成本优化
- **缓存利用**：输出文档可被多次读取（缓存 token 价格低 10 倍）
- **减少重复**：避免重新生成已有内容
- **精准范围**：只处理必要的代码

---

## 最佳实践

### 0. 核心工作流强制规范 ⚠️

#### 🚫 主窗口（Controller）禁止事项

**永远不要在主窗口做以下操作**：

1. ❌ **不要直接写代码**
   ```typescript
   // ❌ 错误：Controller 直接使用 Write/Edit tool 修改代码
   Edit src/services/auth.ts

   // ✅ 正确：创建任务文档，启动 Subagent
   Write docs/tasks/task-1a-auth-service.md
   Task "实现用户认证服务" → Backend Architect
   ```

2. ❌ **不要直接读取源代码文件**
   ```typescript
   // ❌ 错误：读取大量源代码（消耗 20k-50k tokens）
   Read src/services/auth.ts
   Read src/services/user.ts
   Read src/api/routes.ts

   // ✅ 正确：只读取输出文档（每个 ~5k tokens）
   Read docs/outputs/output-1a-auth-implementation.md
   ```

3. ❌ **不要跳过 Sequential Thinking MCP**
   ```typescript
   // ❌ 错误：凭直觉拆分任务
   "我觉得应该做 A、B、C 三个任务"

   // ✅ 正确：使用 MCP 系统性分析
   Sequential Thinking MCP → 分析依赖 → 识别并发机会 → 生成任务计划
   ```

4. ❌ **不要忘记更新进度文档**
   ```typescript
   // ❌ 错误：Subagent 完成后不更新进度
   Task 完成 → 直接进入下一个任务

   // ✅ 正确：立即更新进度
   Task 完成 → 读取输出文档 → 更新 progress.md → 规划下一步
   ```

#### ✅ 主窗口（Controller）正确职责

**Controller 的唯一职责**：

1. ✅ **使用 Sequential Thinking MCP 分析和规划**
   ```typescript
   // 步骤 1: 使用 MCP 拆解任务
   Sequential Thinking MCP:
     思考 1: 分析需求和目标
     思考 2: 识别主要模块
     思考 3: 分析依赖关系
     思考 4: 划分并发批次
     思考 5: 生成任务列表
   ```

2. ✅ **创建任务文档**
   ```markdown
   # Task 1a: 用户认证 API

   ## Role
   Backend Architect

   ## Objective
   实现用户注册和登录 API

   ## Requirements
   - POST /api/auth/register
   - POST /api/auth/login
   - JWT token 生成

   ## Output Location
   docs/outputs/output-1a-auth-api.md
   ```

3. ✅ **并发启动 Subagent**
   ```typescript
   // 在单个消息中启动所有独立任务
   Task 1a: Backend Architect - 用户认证 API
   Task 1b: Database Architect - 数据库设计
   Task 1c: Frontend Developer - 登录 UI
   Task 1d: Security Auditor - 安全审计

   // 4 个任务同时执行（并发）
   ```

4. ✅ **读取输出文档并更新进度**
   ```markdown
   # 读取输出（不读源代码）
   Read docs/outputs/output-1a-auth-api.md  (5k tokens)
   Read docs/outputs/output-1b-database.md  (5k tokens)

   # 更新进度文档
   docs/progress/phase1-progress.md:
   - Task 1a: ✅ 完成 (2025-10-07 10:00)
   - Task 1b: ✅ 完成 (2025-10-07 10:00)
   - 总体进度: 50% (2/4 完成)
   ```

5. ✅ **使用 MCP 分析下一步**
   ```typescript
   Sequential Thinking MCP:
     思考 1: 检查任务完成情况
     思考 2: 评估代码质量
     思考 3: 识别集成问题
     思考 4: 规划下一批任务

   输出: 下一批并发任务列表
   ```

#### 📊 Token 消耗对比

| 操作模式 | Token 消耗 | 说明 |
|---------|-----------|------|
| ❌ 主窗口直接写代码 | 累积增长 | 每次操作都累积历史上下文 |
| ❌ 主窗口读取源代码 | 20k-50k/次 | 大型文件消耗严重 |
| ✅ 只读输出文档 | 5k/次 | 精简摘要，低消耗 |
| ✅ 使用 Subagent | 25k/任务 | 独立上下文，不累积 |
| ✅ 并发 N 个任务 | 25k×N | 同时执行，总时间不变 |

**示例对比**：

```typescript
// ❌ 错误模式（主窗口写代码）
开始: 0 tokens
读取 3 个文件: +45k tokens (累积到 45k)
修改代码: +15k tokens (累积到 60k)
读取更多文件: +30k tokens (累积到 90k)
继续修改: +20k tokens (累积到 110k)
... 很快达到上下文上限

// ✅ 正确模式（使用 Subagent）
开始: 0 tokens
MCP 分析任务: +5k tokens (主窗口累积 5k)
创建 4 个任务文档: +2k tokens (主窗口累积 7k)
启动 4 个 Subagent: 4×25k = 100k tokens (独立上下文)
读取 4 个输出文档: +20k tokens (主窗口累积 27k)
更新进度: +2k tokens (主窗口累积 29k)
MCP 分析下一步: +5k tokens (主窗口累积 34k)

结果: 主窗口仅用 34k tokens，远低于上限
```

#### 🎯 实际执行检查清单

每次开发时检查：

- [ ] ✅ 主窗口是否使用了 Sequential Thinking MCP 拆解任务？
- [ ] ✅ 是否为每个任务创建了独立的任务文档？
- [ ] ✅ 是否并发启动了所有独立任务（单个消息多个 Task）？
- [ ] ✅ 主窗口是否只读取输出文档（而非源代码）？
- [ ] ✅ 是否在每个任务完成后立即更新了进度文档？
- [ ] ✅ 是否使用 MCP 分析了下一步行动？
- [ ] ❌ 主窗口是否直接修改了代码？（应该是 NO）
- [ ] ❌ 主窗口是否读取了源代码文件？（应该是 NO）

---

### 6. 本地开发环境配置规范 🖥️

#### 核心原则
**本地开发调试时，前端和后端应用必须在本地运行，不使用 Docker；数据库、Redis 等基础设施服务使用 Docker；生产环境全部使用 Docker**

#### 环境配置策略

**本地开发环境**：
```yaml
# ✅ 本地运行（不用 Docker）
前端应用:
  - 原因: 热更新快、调试方便、IDE 集成好
  - 运行方式: npm run dev / yarn dev
  - 端口: 3000 (或配置文件指定)

后端应用:
  - 原因: 断点调试、快速重启、日志实时查看
  - 运行方式: npm run dev / python main.py
  - 端口: 8000 (或配置文件指定)

# ✅ Docker 运行
数据库服务:
  - PostgreSQL / MySQL / MongoDB
  - 原因: 版本一致、隔离环境、快速重置
  - 配置: docker-compose.yml

Redis / 缓存:
  - 原因: 轻量、易重置、版本一致
  - 配置: docker-compose.yml

消息队列:
  - RabbitMQ / Kafka / NATS
  - 原因: 复杂配置、统一管理
  - 配置: docker-compose.yml
```

**生产环境**：
```yaml
# ✅ 全部使用 Docker
所有服务:
  - 前端应用 (Nginx + 静态文件)
  - 后端应用 (API 服务)
  - 数据库服务
  - 缓存服务
  - 消息队列
  - 监控服务
配置方式:
  - Docker Compose (单机)
  - Kubernetes (集群)
```

#### 本地开发配置示例

**docker-compose.dev.yml**（仅基础设施）：
```yaml
version: '3.8'

services:
  # 数据库
  postgres:
    image: postgres:15
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: dev_db
      POSTGRES_USER: dev_user
      POSTGRES_PASSWORD: dev_pass
    volumes:
      - postgres_data:/var/lib/postgresql/data

  # Redis
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  # 消息队列（如需要）
  rabbitmq:
    image: rabbitmq:3-management
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq

volumes:
  postgres_data:
  redis_data:
  rabbitmq_data:
```

**启动本地开发环境**：
```bash
# 1. 启动基础设施服务（Docker）
docker-compose -f docker-compose.dev.yml up -d

# 2. 启动后端（本地）
cd backend
npm install
npm run dev  # 或 yarn dev

# 3. 启动前端（本地）- 新终端
cd frontend
npm install
npm run dev  # 或 yarn dev

# 4. 验证服务
curl http://localhost:8000/health  # 后端
curl http://localhost:3000          # 前端
```

#### 配置文件管理

**环境变量配置**：
```bash
# .env.development（本地开发）
# 数据库连接（Docker 服务）
DATABASE_URL=postgresql://dev_user:dev_pass@localhost:5432/dev_db
REDIS_URL=redis://localhost:6379

# 应用配置（本地运行）
BACKEND_URL=http://localhost:8000
FRONTEND_URL=http://localhost:3000
NODE_ENV=development

# 调试选项
DEBUG=true
LOG_LEVEL=debug
HOT_RELOAD=true
```

```bash
# .env.production（生产环境）
# 数据库连接（Docker 内部网络）
DATABASE_URL=postgresql://prod_user:prod_pass@postgres:5432/prod_db
REDIS_URL=redis://redis:6379

# 应用配置（Docker 容器）
BACKEND_URL=http://backend:8000
FRONTEND_URL=https://example.com
NODE_ENV=production

# 生产选项
DEBUG=false
LOG_LEVEL=info
HOT_RELOAD=false
```

#### 开发工作流程

**标准开发流程**：
```
1. 启动基础设施
   → docker-compose -f docker-compose.dev.yml up -d

2. 开发代码
   → 前端/后端在本地运行，支持热更新
   → 代码改动立即生效，无需重建镜像

3. 调试
   → 使用 IDE 断点调试（VSCode / WebStorm）
   → 查看实时日志（终端输出）
   → 使用 Chrome DevTools（前端）

4. 提交前测试
   → 运行单元测试: npm run test
   → 运行集成测试: npm run test:e2e
   → 检查代码规范: npm run lint

5. 部署前验证
   → 构建生产镜像: docker build -t app:latest .
   → 本地运行生产镜像: docker-compose up
   → 验证功能正常后部署
```

#### 常见问题处理

**问题 1：本地应用连接不到 Docker 服务**
```bash
# 解决方案：检查端口映射
docker-compose ps
docker-compose logs postgres
telnet localhost 5432
```

**问题 2：Docker 服务数据持久化**
```bash
# 使用命名卷（已在 docker-compose.yml 配置）
volumes:
  postgres_data:  # 数据持久化
  redis_data:     # 数据持久化
```

**问题 3：重置开发环境**
```bash
# 重置所有 Docker 服务
docker-compose -f docker-compose.dev.yml down -v
docker-compose -f docker-compose.dev.yml up -d

# 重置应用依赖
rm -rf node_modules package-lock.json
npm install
```

#### 最佳实践

**✅ 推荐做法**：
1. **分离关注点**：应用代码（本地）vs 基础设施（Docker）
2. **统一版本**：团队使用相同的 Docker 镜像版本
3. **快速迭代**：本地运行支持热更新，提升开发效率
4. **生产一致**：生产环境完全容器化，避免"在我机器上能跑"问题
5. **文档化配置**：在 README.md 中说明启动步骤

**❌ 避免做法**：
1. 本地开发也用 Docker 运行前后端（热更新慢、调试困难）
2. 生产环境混用 Docker 和非 Docker 部署
3. 数据库直接安装在本地（版本不一致、难以重置）
4. 缺少环境变量文档（团队成员配置困难）

#### Subagent 开发规范

**在编写 Subagent 任务文档时**：
```markdown
## Environment Setup

**Development**:
- Frontend: Run locally with `npm run dev` (port 3000)
- Backend: Run locally with `npm run dev` (port 8000)
- Database: Docker container (port 5432)
- Redis: Docker container (port 6379)

**Production**:
- All services run in Docker containers
- Use docker-compose.prod.yml or Kubernetes manifests
```

**在实现代码时**：
```typescript
// ✅ 正确：使用环境变量适配不同环境
const config = {
  database: {
    host: process.env.DB_HOST || 'localhost',  // 本地: localhost, 生产: postgres
    port: parseInt(process.env.DB_PORT || '5432'),
  },
  redis: {
    host: process.env.REDIS_HOST || 'localhost',  // 本地: localhost, 生产: redis
    port: parseInt(process.env.REDIS_PORT || '6379'),
  }
};

// ❌ 错误：硬编码 Docker 服务名
const config = {
  database: {
    host: 'postgres',  // 本地开发无法连接
    port: 5432,
  }
};
```

---

### 7. MCP 使用规范 🔧

#### 核心原则
**使用 Sequential Thinking MCP 进行任务规划和拆分是强制要求，必须在开发前完成**

#### 使用场景

**✅ 必须使用 MCP 的场景**：
1. **复杂需求分析**：当需求涉及多个模块或系统时
2. **任务拆分规划**：在启动 Subagent 之前规划任务
3. **依赖关系分析**：识别任务间的依赖关系
4. **并发机会识别**：找出可以并发执行的任务组
5. **风险评估**：分析潜在的技术风险和复杂度
6. **集成决策**：评估多个 Subagent 的输出并做出决策

**❌ 不需要使用 MCP 的场景**：
1. **简单单任务**：只需要单个 Subagent 完成的简单任务
2. **紧急修复**：快速 bug 修复，无需深度规划
3. **配置调整**：简单的配置文件修改

#### MCP 安装和验证流程

**自动化检查**：
```typescript
// Controller 在每次开始规划前执行
async function ensureMCPReady() {
  // 1. 读取 MCP 配置
  const configPath = 'C:/Users/53123/AppData/Roaming/Code/User/globalStorage/saoudrizwan.claude-dev/settings/cline_mcp_settings.json';
  const config = JSON.parse(await readFile(configPath));

  // 2. 检查 sequentialthinking MCP
  if (!config.mcpServers?.sequentialthinking) {
    console.log('[WARN] Sequential Thinking MCP 未配置，正在安装...');

    // 3. 添加配置
    config.mcpServers = config.mcpServers || {};
    config.mcpServers.sequentialthinking = {
      command: "npx",
      args: ["-y", "@modelcontextprotocol/sequentialthinking"]
    };

    // 4. 保存配置
    await writeFile(configPath, JSON.stringify(config, null, 2));
    console.log('[SUCCESS] Sequential Thinking MCP 已安装');

    // 5. 提示重启
    console.log('[INFO] 请重启 Claude Code 以加载 MCP');
    return false;
  }

  console.log('[INFO] Sequential Thinking MCP 已就绪');
  return true;
}
```

#### MCP 使用工作流程

**标准流程**：
```
1. 接收开发需求
   ↓
2. 检查 MCP 是否安装（自动）
   ↓
3. 调用 Sequential Thinking MCP
   ↓
4. MCP 分析和拆分任务
   思考 1：理解需求核心
   思考 2：识别关键模块
   思考 3：分析模块依赖
   思考 4：划分任务批次
   思考 5：评估技术风险
   ↓
5. 生成任务文档（基于 MCP 分析）
   ↓
6. 并发启动 Subagent（根据 MCP 建议）
   ↓
7. 收集结果后再次使用 MCP 分析
   ↓
8. MCP 决策下一步行动
```

#### MCP 分析示例

**输入**：
```
需求：实现完整的 Raydium DEX 集成，包括价格获取、流动性查询和交易执行
```

**MCP 分析过程**：
```
思考 1：理解需求范围
- 核心功能：价格获取、流动性、交易
- 技术栈：Raydium SDK、Solana Web3.js
- 复杂度：中高（涉及多个 SDK 调用）

思考 2：识别独立模块
模块 A：价格获取服务
  - 输入：代币对
  - 输出：实时价格数据
  - 依赖：Raydium SDK、RPC 连接

模块 B：流动性查询服务
  - 输入：代币对
  - 输出：流动性池信息
  - 依赖：Raydium SDK、RPC 连接

模块 C：交易执行器
  - 输入：交易参数
  - 输出：交易签名
  - 依赖：价格服务、流动性服务、钱包

思考 3：分析依赖关系
- 模块 A 和 B：独立，可并发
- 模块 C：依赖 A 和 B，必须等待

思考 4：划分执行批次
批次 1（并发）：
  - Task 1a: 实现价格获取服务
  - Task 1b: 实现流动性查询服务

批次 2（依赖批次 1）：
  - Task 2a: 实现交易执行器

思考 5：评估风险
- 技术风险：Raydium SDK API 可能变化
- 性能风险：RPC 调用延迟
- 安全风险：私钥管理

建议：
- 为 SDK 调用添加错误处理和重试
- 使用连接池管理 RPC 连接
- 交易执行使用模拟模式测试
```

**MCP 输出**：
```markdown
# 任务执行计划

## 批次 1：数据服务（并发）
- Task 1a: Raydium 价格获取服务
  - 估计：25k tokens
  - 时间：30 分钟
  - Subagent: Mainnet Integration Expert

- Task 1b: Raydium 流动性查询服务
  - 估计：25k tokens
  - 时间：30 分钟
  - Subagent: Mainnet Integration Expert

## 批次 2：交易执行（依赖批次 1）
- Task 2a: 交易执行器
  - 估计：30k tokens
  - 时间：40 分钟
  - Subagent: Transaction Execution Specialist

## 总体估算
- Token 预算：80k tokens
- 总时间：70 分钟（并发）vs 100 分钟（串行）
- 效率提升：30%
```

#### 并发开发强制规则

**基于 MCP 分析的并发策略**：

```typescript
// ✅ 正确：根据 MCP 分析结果并发启动
const mcpAnalysis = {
  batch1: [
    { task: '1a', independent: true },
    { task: '1b', independent: true }
  ],
  batch2: [
    { task: '2a', dependsOn: ['1a', '1b'] }
  ]
};

// 启动批次 1（并发）
Controller 发送单个消息：
  - Task tool call 1: 启动 Task 1a
  - Task tool call 2: 启动 Task 1b

等待批次 1 完成后：
  - Task tool call 3: 启动 Task 2a

// ❌ 错误：忽略 MCP 分析，串行执行独立任务
// 不要这样做：
启动 Task 1a → 等待 → 启动 Task 1b → 等待 → 启动 Task 2a
```

#### MCP 质量检查

**MCP 分析质量标准**：
- [ ] 识别了所有关键模块
- [ ] 正确分析了依赖关系
- [ ] 划分了合理的任务批次
- [ ] 评估了技术风险
- [ ] 估算了 Token 和时间成本
- [ ] 提供了并发执行建议

**验证 MCP 输出**：
```typescript
// Controller 验证 MCP 分析结果
function validateMCPAnalysis(analysis) {
  // 1. 检查任务拆分是否合理
  if (analysis.tasks.some(t => t.estimatedTokens > 40000)) {
    console.warn('[WARN] 任务过大，建议进一步拆分');
  }

  // 2. 检查依赖关系是否正确
  const dependencyGraph = buildDependencyGraph(analysis);
  if (hasCyclicDependency(dependencyGraph)) {
    console.error('[ERROR] 检测到循环依赖');
  }

  // 3. 检查并发机会
  const concurrentTasks = findConcurrentTasks(analysis);
  console.log(`[INFO] 可并发任务数：${concurrentTasks.length}`);

  return true;
}
```

#### 实际应用示例

**场景：新功能开发**
```
1. 开发者需求：
   "添加多 DEX 价格聚合功能，支持 Raydium、Orca、Jupiter"

2. Controller 调用 MCP：
   思考 1：识别 3 个独立的集成任务
   思考 2：发现可以完全并发
   思考 3：建议并发启动 3 个 Subagent

3. Controller 执行：
   单个消息启动 3 个 Task tool 调用
   - Task 1a: Raydium 集成
   - Task 1b: Orca 集成
   - Task 1c: Jupiter 集成

4. 结果：
   - 时间：30 分钟（vs 串行 90 分钟）
   - 效率提升：3 倍
```

#### 最佳实践

**✅ 推荐做法**：
1. **每次规划前检查 MCP**：确保 MCP 已安装和配置
2. **复杂任务必用 MCP**：超过 3 个子任务的必须使用 MCP 分析
3. **相信 MCP 分析**：严格按照 MCP 的并发建议执行
4. **记录 MCP 输出**：将 MCP 分析结果保存到文档
5. **验证 MCP 建议**：执行前检查依赖关系和资源估算

**❌ 避免做法**：
1. 跳过 MCP 直接规划复杂任务
2. 忽略 MCP 的并发建议，采用串行执行
3. 不记录 MCP 分析过程
4. 对 MCP 的依赖分析提出质疑但不验证
5. 在 MCP 未安装时继续执行

#### 监控和优化

**MCP 使用指标**：
```typescript
// 跟踪 MCP 使用情况
const mcpMetrics = {
  totalAnalyses: 0,        // MCP 分析次数
  tasksIdentified: 0,      // 识别的任务数
  concurrentTasks: 0,      // 并发任务数
  timeSaved: 0,           // 节省的时间（分钟）
  tokenEfficiency: 0      // Token 效率提升（%）
};

// 每次使用 MCP 后更新指标
function trackMCPUsage(analysis) {
  mcpMetrics.totalAnalyses++;
  mcpMetrics.tasksIdentified += analysis.tasks.length;
  mcpMetrics.concurrentTasks += analysis.concurrentGroups;

  const serialTime = analysis.tasks.reduce((sum, t) => sum + t.estimatedTime, 0);
  const parallelTime = Math.max(...analysis.batches.map(b => b.estimatedTime));
  mcpMetrics.timeSaved += (serialTime - parallelTime);
}
```

### 1. 任务设计原则

#### ✅ DO（推荐做法）
- **单一职责**：一个任务只做一件事
- **明确边界**：清晰的输入和输出
- **可测试**：任务完成后可验证
- **文档先行**：先写任务文档，再启动 Subagent

#### ❌ DON'T（避免做法）
- **任务过大**：一个任务修改 >3 个文件
- **职责混杂**：一个任务涉及多个领域
- **模糊目标**：没有明确的成功标准
- **跳过文档**：直接在 prompt 中写大量代码

### 1.0 代码质量和注释规范 📝

#### 核心原则
**代码必须包含详细的注释、实现真正的业务逻辑、具备生产环境质量（健壮、可维护、可扩展）、及时发现并反馈不合理需求**

#### 代码注释要求

**✅ 必须添加注释的地方**：

1. **文件头部注释**：
```typescript
/**
 * 文件名: userAuthService.ts
 * 描述: 用户认证和授权服务
 * 功能:
 *   - 用户注册和登录
 *   - JWT token 生成和验证
 *   - 密码加密和验证
 * 依赖:
 *   - bcrypt: v5.1.0
 *   - jsonwebtoken: v9.0.2
 *   - @prisma/client: v5.0.0
 * 作者: Subagent-BackendArchitect
 * 日期: 2025-01-15
 */
```

2. **类和接口注释**：
```typescript
/**
 * 用户认证服务类
 * 负责处理用户注册、登录和 JWT token 管理
 *
 * 使用示例:
 * ```typescript
 * const authService = new UserAuthService(prisma, jwtSecret);
 * const result = await authService.login('user@example.com', 'password123');
 * console.log(`Token: ${result.token}`);
 * ```
 *
 * @throws {AuthenticationError} - 认证失败时抛出
 * @throws {ValidationError} - 参数验证失败时抛出
 */
export class UserAuthService {
  // 实现...
}
```

3. **函数注释**：
```typescript
/**
 * 用户登录
 *
 * @param email - 用户邮箱
 * @param password - 用户密码
 * @param options - 可选配置
 * @param options.rememberMe - 是否保持登录（默认 false）
 * @param options.tokenExpiry - Token 有效期（秒，默认 86400）
 * @returns Promise<AuthResult> - 返回 token 和用户信息
 *
 * @example
 * ```typescript
 * // 基本登录
 * const result = await login('user@example.com', 'password123');
 *
 * // 保持7天登录
 * const result = await login('user@example.com', 'password123', {
 *   rememberMe: true,
 *   tokenExpiry: 604800
 * });
 * ```
 *
 * @throws {ValidationError} - 邮箱或密码格式错误
 * @throws {AuthenticationError} - 用户不存在或密码错误
 */
async login(
  email: string,
  password: string,
  options?: LoginOptions
): Promise<AuthResult> {
  // 1. 参数验证
  if (!email || !password) {
    throw new ValidationError('邮箱和密码不能为空');
  }

  // 2. 查找用户
  const user = await this.userRepo.findByEmail(email);
  if (!user) {
    console.log(`[AUTH] 用户不存在: ${email}`);
    throw new AuthenticationError('用户不存在或密码错误');
  }

  // 3. 验证密码
  try {
    const isValid = await compare(password, user.passwordHash);
    if (!isValid) {
      console.log(`[AUTH] 密码错误: ${email}`);
      throw new AuthenticationError('用户不存在或密码错误');
    }

    // 4. 生成 JWT token
    const tokenExpiry = options?.tokenExpiry || 86400;
    const token = this.generateToken(user, tokenExpiry);

    // 5. 记录登录日志
    await this.logLogin(user.id, 'success');

    console.log(`[AUTH] 登录成功: ${email}`);
    return {
      token,
      user: { id: user.id, email: user.email, name: user.name }
    };
  } catch (error) {
    await this.logLogin(user.id, 'failed');
    console.error(`[ERROR] 登录失败 ${email}:`, error);
    throw new AuthenticationError('登录失败', { cause: error });
  }
}
```

4. **复杂逻辑注释**：
```typescript
// 计算用户权限
// 算法说明：
// 1. 获取用户的所有角色
// 2. 获取每个角色关联的权限
// 3. 合并所有权限并去重
// 4. 检查是否有全局管理员权限
// 5. 返回权限列表
const userRoles = await this.roleRepo.findByUserId(userId);
const allPermissions = new Set<string>();

for (const role of userRoles) {
  const permissions = await this.permissionRepo.findByRoleId(role.id);
  permissions.forEach(p => allPermissions.add(p.name));
}

// 检查全局管理员
const isAdmin = userRoles.some(role => role.name === 'SUPER_ADMIN');
if (isAdmin) {
  // 管理员拥有所有权限
  allPermissions.add('*');
}

return {
  userId,
  permissions: Array.from(allPermissions),
  isAdmin
};
```

5. **TODO 和 FIXME 注释**：
```typescript
// TODO: 添加OAuth第三方登录支持（优先级：高）
// 背景: 用户希望使用 Google/GitHub 快速登录
// 预计完成: v2.0.0
// 负责人: @auth-team

// FIXME: 并发登录时Session冲突问题（Issue #123）
// 复现: 用户同时在多设备登录时，5% 概率发生session覆盖
// 临时方案: 添加了设备ID字段区分session
// 根本方案: 需要实现Redis集群保证session一致性
```

**❌ 避免的注释**：
```typescript
// ❌ 废话注释（代码已经很清楚）
const price = 100; // 设置价格为100

// ❌ 过时注释（代码已改，注释未更新）
// 使用 axios 获取数据
const data = await fetch(url); // 实际已改用 fetch

// ❌ 注释掉的代码（应该删除或移到 git history）
// const oldPrice = await getOldPrice();
// return oldPrice * 1.1;
```

#### 生产环境代码质量标准

**✅ 必须实现的功能**：

1. **真实业务逻辑（不要 Mock 数据）**：
```typescript
// ❌ 错误：返回假数据
async getPrice(): Promise<number> {
  return 100; // Mock 数据
}

// ✅ 正确：真实实现
async getPrice(base: string, quote: string): Promise<number> {
  // 1. 连接 Raydium RPC
  const connection = new Connection(this.rpcUrl);

  // 2. 获取池地址
  const poolAddress = await this.getPoolAddress(base, quote);

  // 3. 获取账户信息
  const accountInfo = await connection.getAccountInfo(poolAddress);

  // 4. 解析池数据
  const poolState = this.parsePoolState(accountInfo.data);

  // 5. 计算价格
  return poolState.quoteReserve / poolState.baseReserve;
}
```

2. **健壮性（错误处理和边界条件）**：
```typescript
// ✅ 正确：完整的错误处理
async getPrice(base: string, quote: string): Promise<number> {
  // 参数验证
  if (!base || !quote) {
    throw new ValidationError('代币符号不能为空');
  }

  if (base === quote) {
    throw new ValidationError('基础代币和报价代币不能相同');
  }

  // 网络请求重试
  let lastError: Error | null = null;
  for (let attempt = 1; attempt <= this.maxRetries; attempt++) {
    try {
      const result = await this.fetchPoolData(base, quote);

      // 数据验证
      if (!result || result.baseReserve <= 0 || result.quoteReserve <= 0) {
        throw new DataError('池数据无效');
      }

      return result.quoteReserve / result.baseReserve;
    } catch (error) {
      lastError = error as Error;
      console.warn(`[RETRY] 第 ${attempt}/${this.maxRetries} 次尝试失败:`, error);

      // 等待后重试（指数退避）
      if (attempt < this.maxRetries) {
        await sleep(Math.pow(2, attempt) * 1000);
      }
    }
  }

  // 所有重试都失败
  throw new NetworkError(`获取价格失败（已重试 ${this.maxRetries} 次）`, {
    cause: lastError
  });
}
```

3. **可维护性（代码组织和设计模式）**：
```typescript
// ✅ 正确：单一职责，依赖注入
export class RaydiumPriceService {
  constructor(
    private connection: Connection,
    private cache: CacheService,
    private config: RaydiumConfig,
    private logger: Logger
  ) {}

  // 公共接口：获取价格
  async getPrice(base: string, quote: string): Promise<number> {
    this.validateTokens(base, quote);

    const cached = await this.getCachedPrice(base, quote);
    if (cached) return cached;

    const price = await this.fetchPrice(base, quote);
    await this.cachePrice(base, quote, price);

    return price;
  }

  // 私有方法：职责单一
  private validateTokens(base: string, quote: string): void {
    // 验证逻辑
  }

  private async getCachedPrice(base: string, quote: string): Promise<number | null> {
    // 缓存逻辑
  }

  private async fetchPrice(base: string, quote: string): Promise<number> {
    // 获取逻辑
  }

  private async cachePrice(base: string, quote: string, price: number): Promise<void> {
    // 缓存更新逻辑
  }
}
```

4. **可扩展性（配置化和插件化）**：
```typescript
// ✅ 正确：配置驱动，易扩展
export interface DEXPriceProvider {
  getName(): string;
  getPrice(base: string, quote: string): Promise<number>;
  getSupportedTokens(): Promise<string[]>;
}

export class PriceAggregator {
  private providers: Map<string, DEXPriceProvider> = new Map();

  // 动态注册 DEX 提供商
  registerProvider(provider: DEXPriceProvider): void {
    this.providers.set(provider.getName(), provider);
  }

  // 聚合多个 DEX 的价格
  async getAveragePrice(base: string, quote: string): Promise<number> {
    const prices = await Promise.all(
      Array.from(this.providers.values()).map(p => p.getPrice(base, quote))
    );

    return prices.reduce((sum, p) => sum + p, 0) / prices.length;
  }
}

// 使用示例
const aggregator = new PriceAggregator();
aggregator.registerProvider(new RaydiumPriceService(...));
aggregator.registerProvider(new OrcaPriceService(...));
aggregator.registerProvider(new SerumPriceService(...)); // 未来可轻松添加
```

#### 需求合理性检查

**Controller 必须在启动 Subagent 前检查需求**：

**✅ 合理需求示例**：
- "实现 Raydium 价格获取功能，支持缓存和错误重试"
- "添加监控告警，当价差 > 1% 时发送通知"
- "优化交易执行器，减少 gas 费用"

**❌ 不合理需求示例及处理**：
1. **需求：实现无风险套利机器人，保证 100% 盈利**
   ```
   ⚠️ 停止开发
   原因：技术上不可能实现无风险套利（存在滑点、gas 费、网络延迟等风险）
   建议：改为"实现风险可控的套利机器人，目标胜率 > 70%"
   ```

2. **需求：直接使用用户私钥进行交易**
   ```
   ⚠️ 停止开发
   原因：安全风险极高，违反最佳实践
   建议：使用硬件钱包或多重签名方案，私钥加密存储
   ```

3. **需求：绕过 DEX 合约，直接修改链上状态**
   ```
   ⚠️ 停止开发
   原因：区块链不可篡改，技术上不可能实现
   建议：通过合法的交易接口与 DEX 交互
   ```

4. **需求：抓取所有 DEX 的实时数据，无限制并发请求**
   ```
   ⚠️ 停止开发
   原因：违反 RPC 节点使用政策，会导致 IP 被封禁
   建议：使用速率限制（如每秒 10 次请求），或使用付费 RPC 服务
   ```

**Controller 检查流程**：
```typescript
async function validateRequirement(requirement: string): Promise<ValidationResult> {
  // 1. 技术可行性检查
  const technicalIssues = checkTechnicalFeasibility(requirement);
  if (technicalIssues.length > 0) {
    return {
      valid: false,
      reason: '技术不可行',
      issues: technicalIssues,
      action: '停止开发，与用户讨论替代方案'
    };
  }

  // 2. 安全风险检查
  const securityIssues = checkSecurityRisks(requirement);
  if (securityIssues.length > 0) {
    return {
      valid: false,
      reason: '存在安全风险',
      issues: securityIssues,
      action: '停止开发，建议安全的实现方式'
    };
  }

  // 3. 合规性检查
  const complianceIssues = checkCompliance(requirement);
  if (complianceIssues.length > 0) {
    return {
      valid: false,
      reason: '违反使用政策或法律法规',
      issues: complianceIssues,
      action: '停止开发，提醒用户合规风险'
    };
  }

  // 4. 性能和成本检查
  const costIssues = checkCostEffectiveness(requirement);
  if (costIssues.length > 0) {
    return {
      valid: true, // 可以实现，但需警告
      warnings: costIssues,
      action: '继续开发，但在文档中说明成本和性能影响'
    };
  }

  return { valid: true, action: '继续开发' };
}
```

**停止开发的通知模板**：
```
⚠️ 发现需求问题，已停止开发

**问题类型**: [技术不可行 / 安全风险 / 合规问题]

**具体问题**:
- [问题描述 1]
- [问题描述 2]

**建议方案**:
- 方案 A: [安全/合规的替代方案]
- 方案 B: [简化版实现]
- 方案 C: [使用第三方服务]

**是否继续？请确认修改后的需求**
```

### 1.1 调试日志规范 🔍

#### 核心原则
**所有代码必须输出终端调试日志，包括前端和后端代码**

#### 日志要求

**✅ 必须做到**：
- **后端日志**：使用 `console.log/console.error` 输出到终端
- **前端日志**：使用服务端日志或 Node.js 日志输出到终端，**不要只输出到浏览器控制台**
- **关键节点**：函数入口、出口、关键分支、错误处理都要记录
- **上下文信息**：包含必要的参数值、状态信息、时间戳
- **错误堆栈**：捕获并输出完整的错误堆栈信息
- **分级输出**：区分 INFO、WARN、ERROR 等级别

**日志格式示例**：
```typescript
// 后端 - Node.js/Express
console.log('[INFO] [RaydiumPriceFetcher] Fetching price for:', tokenPair);
console.error('[ERROR] [RaydiumPriceFetcher] Failed to fetch price:', error.message, error.stack);

// 前端 - Next.js Server Component
console.log('[INFO] [PriceDisplay] Rendering price component:', { price, timestamp });

// 前端 - API Route
export async function GET(request: Request) {
  console.log('[INFO] [API /api/price] Request received:', request.url);
  // ...
  console.log('[INFO] [API /api/price] Response:', result);
}
```

**❌ 禁止做法**：
- 前端代码仅使用 `console.log` 输出到浏览器控制台
- 缺少关键操作的日志记录
- 日志信息不包含上下文
- 吞掉错误不输出堆栈

#### 日志覆盖范围

**必须记录的场景**：
1. **函数入口**：记录参数和调用时间
2. **外部调用**：API 请求、数据库查询、RPC 调用
3. **条件分支**：重要的 if/else、switch 分支
4. **循环操作**：批量处理的进度和结果
5. **错误处理**：所有 try/catch 块
6. **异步操作**：Promise、async/await 的状态
7. **状态变更**：关键数据的修改

**日志示例**：
```typescript
async function fetchRaydiumPrice(tokenPair: string): Promise<PriceData | null> {
  console.log(`[INFO] [fetchRaydiumPrice] START - tokenPair: ${tokenPair}, timestamp: ${Date.now()}`);

  const maxRetries = 3;
  let attempt = 0;

  while (attempt < maxRetries) {
    attempt++;
    console.log(`[INFO] [fetchRaydiumPrice] Attempt ${attempt}/${maxRetries}`);

    try {
      const connection = new Connection(process.env.SOLANA_RPC_URL!);
      console.log(`[INFO] [fetchRaydiumPrice] Connection established to RPC`);

      const poolInfo = await this.getRaydiumPool(tokenPair);
      console.log(`[INFO] [fetchRaydiumPrice] Pool info retrieved:`, poolInfo ? 'success' : 'not found');

      if (!poolInfo) {
        throw new Error(`Pool not found for ${tokenPair}`);
      }

      const price = this.calculatePrice(poolInfo);
      console.log(`[INFO] [fetchRaydiumPrice] Price calculated: ${price}`);

      const result = {
        tokenPair,
        price,
        timestamp: Date.now(),
        source: 'raydium'
      };

      console.log(`[INFO] [fetchRaydiumPrice] SUCCESS - Result:`, JSON.stringify(result));
      return result;

    } catch (error) {
      console.error(`[ERROR] [fetchRaydiumPrice] Attempt ${attempt} failed:`, error.message);
      console.error(`[ERROR] [fetchRaydiumPrice] Stack trace:`, error.stack);

      if (attempt >= maxRetries) {
        console.error(`[ERROR] [fetchRaydiumPrice] FAILED - All ${maxRetries} attempts exhausted`);
        return null;
      }

      const delay = Math.pow(2, attempt) * 1000;
      console.log(`[INFO] [fetchRaydiumPrice] Retrying in ${delay}ms...`);
      await this.sleep(delay);
    }
  }

  console.log(`[INFO] [fetchRaydiumPrice] END - Returning null`);
  return null;
}
```

### 1.2 测试验证规范 ✅

#### 核心原则
**每次代码实现完成后，必须运行项目并检查终端日志，确保没有错误或警告**

#### 测试流程

**✅ 必须执行的步骤**：

1. **运行项目**
   ```bash
   # 后端项目
   npm run dev
   # 或
   npm start

   # 前端项目
   npm run dev
   # 或
   yarn dev
   ```

2. **观察终端日志**
   - 检查启动过程中是否有错误
   - 检查是否有未捕获的异常
   - 检查是否有废弃 API 警告
   - 检查是否有类型错误

3. **触发代码路径**
   - 访问相关页面或 API 端点
   - 触发新实现的功能
   - 验证日志输出符合预期

4. **检查日志质量**
   - 确认关键操作有日志输出
   - 确认日志信息完整（包含上下文）
   - 确认错误有完整堆栈信息
   - 确认日志级别正确（INFO/WARN/ERROR）

5. **修复问题**
   - 如果发现错误，立即修复
   - 如果发现警告，评估并处理
   - 重新运行测试直到没有问题

#### 验证清单

**在提交代码前必须确认**：

- [ ] 项目能够成功启动，无启动错误
- [ ] 终端日志中没有 ERROR 级别的输出
- [ ] 终端日志中没有未处理的 Promise rejection
- [ ] 新功能的日志输出完整且清晰
- [ ] 错误处理路径有正确的日志输出
- [ ] 没有冗余或垃圾日志（如过度的 debug 信息）
- [ ] 前端代码的日志在终端可见（不仅在浏览器控制台）

#### 常见问题检查

**必须排查的问题**：

1. **类型错误**
   ```
   ❌ TypeError: Cannot read property 'xxx' of undefined
   ✅ 检查对象是否存在，添加可选链或默认值
   ```

2. **异步错误**
   ```
   ❌ UnhandledPromiseRejectionWarning
   ✅ 所有 Promise 必须有 .catch() 或在 try/catch 中
   ```

3. **环境变量**
   ```
   ❌ process.env.XXX is undefined
   ✅ 检查 .env 文件配置，添加默认值或错误提示
   ```

4. **依赖缺失**
   ```
   ❌ Cannot find module 'xxx'
   ✅ 运行 npm install 或 yarn install
   ```

5. **端口占用**
   ```
   ❌ EADDRINUSE: address already in use :::3000
   ✅ 更换端口或关闭占用进程
   ```

#### 日志验证示例

**良好的测试输出**：
```
[INFO] [Server] Starting on port 3000...
[INFO] [Database] Connecting to MongoDB...
[INFO] [Database] Connected successfully
[INFO] [RaydiumPriceFetcher] Initializing with RPC: https://api.mainnet-beta.solana.com
[INFO] [Server] Server ready on http://localhost:3000

[INFO] [API /api/price] Request received: /api/price?pair=SOL/USDC
[INFO] [fetchRaydiumPrice] START - tokenPair: SOL/USDC, timestamp: 1696234567890
[INFO] [fetchRaydiumPrice] Attempt 1/3
[INFO] [fetchRaydiumPrice] Connection established to RPC
[INFO] [fetchRaydiumPrice] Pool info retrieved: success
[INFO] [fetchRaydiumPrice] Price calculated: 142.35
[INFO] [fetchRaydiumPrice] SUCCESS - Result: {"tokenPair":"SOL/USDC","price":142.35,"timestamp":1696234567890,"source":"raydium"}
[INFO] [API /api/price] Response: {"success":true,"data":{"price":142.35}}
```

**有问题的测试输出**（需要修复）：
```
[INFO] [Server] Starting on port 3000...
[INFO] [Database] Connecting to MongoDB...
[ERROR] [Database] Connection failed: MongoNetworkError: connect ECONNREFUSED 127.0.0.1:27017
    at connectionFailureError (/app/node_modules/mongodb/lib/core/connection/connect.js:362:14)
    at Socket.<anonymous> (/app/node_modules/mongodb/lib/core/connection/connect.js:330:16)
    ...
❌ 需要修复：检查 MongoDB 连接配置或启动 MongoDB 服务

[INFO] [API /api/price] Request received: /api/price?pair=SOL/USDC
[ERROR] [fetchRaydiumPrice] Attempt 1 failed: Cannot read property 'baseReserve' of null
[ERROR] [fetchRaydiumPrice] Stack trace: TypeError: Cannot read property 'baseReserve' of null
    at RaydiumPriceFetcher.calculatePrice (/app/src/fetcher.ts:45:30)
    ...
❌ 需要修复：添加 null 检查或改进错误处理
```

#### 持续改进

**迭代循环**：
```
1. 实现代码 + 添加日志
   ↓
2. 运行项目
   ↓
3. 检查终端日志
   ↓
4. 发现问题？
   ├─ 是 → 修复代码 → 回到步骤 2
   └─ 否 → 测试通过 ✅
```

**成功标准**：
- 连续 3 次运行都没有错误
- 所有功能路径都被触发并有日志记录
- 日志输出清晰、完整、易于调试

### 2. 文档编写原则

#### 任务文档
```markdown
✅ 好的任务文档：
- 目标清晰（1-2 句话）
- 输入明确（具体文件和行号）
- 要求具体（可量化的标准）
- 约束明确（兼容性、性能等）

❌ 差的任务文档：
- 目标模糊（"优化系统"）
- 输入不明（"相关代码"）
- 要求笼统（"尽可能好"）
- 无约束（导致不兼容改动）
```

#### 输出文档
```markdown
✅ 好的输出文档：
- 包含完整代码或关键改动
- 测试结果清晰
- 使用示例完整
- 后续建议具体

❌ 差的输出文档：
- 只有代码，无说明
- 无测试结果
- 无使用示例
- 无后续建议
```

### 3. Subagent 角色设计

#### 专业化原则
每个 Subagent 应该：
- 有明确的专长领域
- 工作范围清晰（特定目录/模块）
- 不重叠职责

#### 示例
```
✅ 好的角色分工：
- Data Integration Expert: 专注 DEX 数据获取
- Risk Management Expert: 专注风险控制逻辑
- Transaction Expert: 专注交易执行

❌ 差的角色分工：
- General Developer: 什么都做
- Backend Expert: 范围过大
- Helper: 职责不明
```

### 4. Controller 决策原则

#### 使用 Sequential Thinking
```
思考 1：分析当前状态
思考 2：识别可并发任务
思考 3：检查依赖关系
思考 4：评估风险
思考 5：决定执行顺序
```

#### 质量检查
- 每个 Phase 结束后，审查所有输出
- 运行集成测试
- 更新知识库

### 5. Token 预算监控

#### 预防性措施
```
1. 创建任务时估算 token
2. 限制文件数量（1-2 个）
3. 限制文件大小（200-300 行）
4. 使用文档引用而非复制代码
```

#### 应对超预算
```
如果任务预计 >40k tokens：
  → 拆分为 2-3 个小任务
  → 使用更精细的文件范围
  → 减少背景信息
```

---

## TDD 与 Subagent 方法论整合

### 整合优势

将 TDD 工作流程与 Subagent 并发开发相结合，可以获得以下优势：

1. **质量保证**：TDD 确保每个功能都有测试覆盖
2. **并发开发**：多个功能的 TDD 循环可以并发进行
3. **渐进式交付**：每个 TDD 循环都是一个可交付的增量
4. **文档化**：每个阶段都有详细的文档记录

### 实际工作流程

#### 场景：实现 4 个独立的数据获取功能

```
Controller 分析和规划
         ↓
┌────────┴────────┬────────┬────────┐
│                 │        │        │
功能 A            功能 B   功能 C   功能 D
Raydium价格      Orca价格  监控    Shadow模式
│                 │        │        │
├─ RED 🔴        ├─ RED    ├─ RED   ├─ RED
│  ↓             │  ↓      │  ↓     │  ↓
├─ GREEN 🟢      ├─ GREEN  ├─ GREEN ├─ GREEN
│  ↓             │  ↓      │  ↓     │  ↓
└─ REFACTOR 🔧   └─ REFAC  └─ REFAC └─ REFAC
         │                 │        │        │
         └────────┬────────┴────────┴────────┘
                  ↓
         Controller 整合和验证
```

#### 并发执行策略

**第一轮：RED 阶段并发**

Controller 创建 4 个 RED 任务文档，并发启动 4 个 TDD Red-Phase Specialist：

```markdown
任务 1a: Raydium 价格获取 - RED
任务 1b: Orca 价格获取 - RED
任务 1c: 监控框架 - RED
任务 1d: Shadow 模式执行器 - RED

并发执行：4 × 23k tokens = 92k tokens
```

所有 Subagent 完成后，Controller 收集输出：
- 4 个测试文件
- 4 个 RED 阶段输出文档
- 所有测试都是失败状态（符合预期）

**第二轮：GREEN 阶段并发**

Controller 创建 4 个 GREEN 任务文档，并发启动 4 个 TDD Green-Phase Developer：

```markdown
任务 1a: Raydium 价格获取 - GREEN
任务 1b: Orca 价格获取 - GREEN
任务 1c: 监控框架 - GREEN
任务 1d: Shadow 模式执行器 - GREEN

并发执行：4 × 23k tokens = 92k tokens
```

所有 Subagent 完成后，Controller 收集输出：
- 4 个实现文件
- 4 个 GREEN 阶段输出文档
- 所有测试都通过（绿色状态）

**第三轮：REFACTOR 阶段并发**

Controller 创建 4 个 REFACTOR 任务文档，并发启动 4 个 TDD Refactor Specialist：

```markdown
任务 1a: Raydium 价格获取 - REFACTOR
任务 1b: Orca 价格获取 - REFACTOR
任务 1c: 监控框架 - REFACTOR
任务 1d: Shadow 模式执行器 - REFACTOR

并发执行：4 × 23k tokens = 92k tokens
```

所有 Subagent 完成后，Controller 收集输出：
- 4 个优化后的实现文件
- 4 个 REFACTOR 阶段输出文档
- 所有测试依然通过，代码质量提升

### TDD 任务文档完整示例

```markdown
# Task Group 1: Raydium 价格获取功能 - TDD 完整循环

## Task 1a-RED: Raydium 价格获取 - 编写测试

### Role
TDD Red-Phase Specialist

### Objective
为 Raydium mainnet 价格获取功能编写失败的测试用例

### Context
- 当前系统使用模拟数据
- 需要集成真实的 Raydium SDK
- 支持常见代币对（SOL/USDC, SOL/USDT 等）

### Requirements
1. 测试正常情况：成功获取价格
2. 测试边界情况：无效代币对、空值处理
3. 测试异常情况：网络失败、重试机制
4. 测试性能：延迟应 <500ms
5. 测试缓存：相同请求应使用缓存

### Output Location
docs/outputs/tdd/phase1/raydium-red.md

### Success Criteria
- [ ] 至少 5 个测试用例
- [ ] 覆盖正常、边界、异常场景
- [ ] 所有测试失败（功能未实现）
- [ ] 测试代码清晰可读

---

## Task 1a-GREEN: Raydium 价格获取 - 实现功能

### Role
TDD Green-Phase Developer

### Objective
实现 Raydium 价格获取功能，让所有测试通过

### Input
- 测试文件：tests/raydium-price-fetcher.test.ts
- RED 阶段输出：docs/outputs/tdd/phase1/raydium-red.md

### Requirements
1. 让所有测试从红色变为绿色
2. 使用最简单的实现方式
3. 允许硬编码和临时方案
4. 不追求完美设计

### Output Location
docs/outputs/tdd/phase1/raydium-green.md

### Success Criteria
- [ ] 所有测试通过
- [ ] 代码简单直接
- [ ] 满足所有功能要求

---

## Task 1a-REFACTOR: Raydium 价格获取 - 优化代码

### Role
TDD Refactor Specialist

### Objective
优化代码质量，保持测试通过

### Input
- GREEN 阶段输出：docs/outputs/tdd/phase1/raydium-green.md
- 实现文件：src/raydium-price-fetcher.ts
- 测试文件：tests/raydium-price-fetcher.test.ts

### Requirements
1. 消除重复代码
2. 提取方法，提升可读性
3. 应用设计模式（如需要）
4. 优化性能
5. 确保所有测试依然通过

### Output Location
docs/outputs/tdd/phase1/raydium-refactor.md

### Success Criteria
- [ ] 代码质量提升
- [ ] 无重复代码
- [ ] 方法平均长度 <15 行
- [ ] 所有测试通过
- [ ] 测试覆盖率 ≥90%
```

### Token 使用优化

通过 TDD 三阶段方法，每个功能的 token 使用如下：

```
RED 阶段:    ~23k tokens (测试编写)
GREEN 阶段:  ~23k tokens (功能实现)
REFACTOR 阶段: ~23k tokens (代码优化)
─────────────────────────────────
总计:        ~69k tokens (单个功能完整 TDD 循环)
```

**并发执行 4 个功能**：
- RED 阶段并发：   4 × 23k = 92k tokens
- GREEN 阶段并发： 4 × 23k = 92k tokens
- REFACTOR 阶段并发：4 × 23k = 92k tokens

**总计**：约 276k tokens，分 3 轮执行

**串行执行对比**：
- 如果串行执行，需要 12 次任务（4 功能 × 3 阶段）
- 时间消耗：12 次交互
- 并发执行只需 3 次交互（每个阶段一次）

**效率提升**：**4 倍**（12 次 → 3 次）

### 实际应用建议

#### 何时使用 TDD + Subagent

**✅ 推荐场景**：
1. **新功能开发**：需求明确，可以先写测试
2. **Bug 修复**：先写复现 bug 的测试，再修复
3. **重构优化**：已有测试保护，放心重构
4. **API 开发**：接口设计先行，TDD 最合适
5. **关键模块**：风险管理、交易执行等核心功能

**❌ 不推荐场景**：
1. **探索性开发**：需求不明确，难以先写测试
2. **快速原型**：验证想法阶段，测试成本高
3. **简单脚本**：一次性任务，无需测试
4. **UI 调整**：视觉效果难以自动化测试

#### 混合模式

对于复杂项目，可以混合使用：

```
核心功能（数据集成、风险控制）
  → 使用 TDD + Subagent（高质量，有测试）

辅助功能（日志、配置）
  → 使用传统 Subagent（快速实现）

原型验证（新策略尝试）
  → 快速开发，验证后再 TDD 重写
```

### 进度跟踪示例

```markdown
# Phase 1: Shadow Mode - TDD 进度

## 功能 A: Raydium 价格获取
- ✅ RED 阶段完成（2025-10-02 10:00）
- ✅ GREEN 阶段完成（2025-10-02 10:30）
- ✅ REFACTOR 阶段完成（2025-10-02 11:00）
- 📄 输出文档：
  - docs/outputs/tdd/phase1/raydium-red.md
  - docs/outputs/tdd/phase1/raydium-green.md
  - docs/outputs/tdd/phase1/raydium-refactor.md

## 功能 B: Orca 价格获取
- ✅ RED 阶段完成
- ✅ GREEN 阶段完成
- 🔄 REFACTOR 阶段进行中

## 功能 C: 监控框架
- ✅ RED 阶段完成
- 🔄 GREEN 阶段进行中
- 📅 REFACTOR 阶段待开始

## 功能 D: Shadow 模式执行器
- 🔄 RED 阶段进行中
- 📅 GREEN 阶段待开始
- 📅 REFACTOR 阶段待开始

## 总体进度
- 已完成：5/12 任务（42%）
- 进行中：2/12 任务
- 待开始：5/12 任务
```

---

## 实际应用示例

### 案例：用户认证 API 实现（使用 TDD 方法）

#### Step 1: 创建任务文档
```markdown
# Task 1a: 用户认证 API 端点

## Role
Backend Architect

## Objective
实现用户登录和注册的 RESTful API 端点，包括 JWT 认证

## Context
- 新建项目，尚无认证系统
- 使用 Express.js + TypeScript
- 数据库使用 PostgreSQL + Prisma ORM

## Input Files
- src/api/routes/ (新建)
- src/services/auth/ (新建)
- prisma/schema.prisma (需要创建 User model)

## Requirements
1. 实现 POST /api/auth/register 注册端点
2. 实现 POST /api/auth/login 登录端点
3. 密码使用 bcrypt 加密
4. 返回 JWT token（24小时有效期）
5. 添加输入验证（email 格式、密码强度）
6. 添加错误处理（用户已存在、密码错误等）

## Constraints
- 不修改其他模块
- 保持 RESTful 规范
- API 响应时间 <200ms

## Output Location
docs/outputs/phase1/output-1a-auth-api-implementation.md

## Token Budget
预计：25k tokens

## Success Criteria
- [ ] 注册接口正常工作
- [ ] 登录返回有效 JWT
- [ ] 所有边界条件有错误处理
- [ ] 单元测试和集成测试通过
```

#### Step 2: 启动 Subagent
Controller 使用 Task tool：
```
请执行以下任务：

你是 Backend Architect，专精 RESTful API 设计和认证系统。

1. 读取任务文档：docs/tasks/phase1/task-1a-user-auth-api.md
2. 按照要求实现功能
3. 编写测试用例
4. 将结果写入：docs/outputs/phase1/output-1a-auth-api-implementation.md

确保代码简洁、可维护，并包含必要的注释。
```

#### Step 3: Subagent 输出
```markdown
# Output 1a: 用户认证 API - 实现报告

## Status
✅ 已完成

## Modified Files
- src/api/routes/auth.routes.ts (新建)
- src/services/auth/auth.service.ts (新建)
- src/middleware/validation.middleware.ts (新建)
- prisma/schema.prisma (更新)

## Implementation

### 核心改动
```typescript
// src/services/auth/auth.service.ts
import { hash, compare } from 'bcrypt';
import { sign } from 'jsonwebtoken';
import { PrismaClient } from '@prisma/client';

export class AuthService {
  private prisma = new PrismaClient();
  private jwtSecret = process.env.JWT_SECRET!;

  async register(email: string, password: string) {
    // 检查用户是否已存在
    const existing = await this.prisma.user.findUnique({ where: { email } });
    if (existing) {
      throw new Error('User already exists');
    }

    // 密码加密
    const passwordHash = await hash(password, 10);

    // 创建用户
    const user = await this.prisma.user.create({
      data: { email, passwordHash }
    });

    return {
      token: this.generateToken(user),
      user: { id: user.id, email: user.email }
    };
  }

  async login(email: string, password: string) {
    const user = await this.prisma.user.findUnique({ where: { email } });
    if (!user) {
      throw new Error('Invalid credentials');
    }

    const isValid = await compare(password, user.passwordHash);
    if (!isValid) {
      throw new Error('Invalid credentials');
    }

    return {
      token: this.generateToken(user),
      user: { id: user.id, email: user.email }
    };
  }

  private generateToken(user: any) {
    return sign(
      { userId: user.id, email: user.email },
      this.jwtSecret,
      { expiresIn: '24h' }
    );
  }
}

    } catch (error) {
      attempt++;
      if (attempt >= maxRetries) {
        console.error(`Failed to fetch Raydium price after ${maxRetries} attempts:`, error);
        return null;
      }
      // 指数退避
      await this.sleep(Math.pow(2, attempt) * 1000);
    }
  }

  return null;
}

private async getRaydiumPool(tokenA: string, tokenB: string) {
  // 实现池子查询逻辑
  // ...
}
```

### 设计决策
- **重试机制**：使用指数退避（1s, 2s, 4s）避免频繁请求
- **错误处理**：返回 null 而非抛出异常，保持与原接口兼容
- **缓存友好**：不在函数内部缓存，由上层 retrievePriceData 处理

## Testing

### 单元测试
```typescript
describe('fetchRaydiumPrice', () => {
  it('should fetch SOL/USDC price', async () => {
    const price = await rag.fetchRaydiumPrice('SOL/USDC');
    expect(price).toBeDefined();
    expect(price.source).toBe('raydium');
    expect(price.price).toBeGreaterThan(0);
  });

  it('should handle invalid token pair', async () => {
    const price = await rag.fetchRaydiumPrice('INVALID/PAIR');
    expect(price).toBeNull();
  });

  it('should retry on network error', async () => {
    // Mock network failure then success
    // ...
  });
});
```

### 测试结果
- ✅ SOL/USDC 价格获取成功：$142.35
- ✅ SOL/USDT 价格获取成功：$142.28
- ✅ 无效代币对正确返回 null
- ✅ 重试机制验证通过
- ⏱️ 平均延迟：187ms（符合 <500ms 要求）

## Usage

### 基本用法
```typescript
const rag = new MarketDataRAG(process.env.SOLANA_RPC_URL!);
const price = await rag.fetchRaydiumPrice('SOL/USDC');
console.log(price?.price); // 142.35
```

## Dependencies
- 已有：@raydium-io/raydium-sdk@1.3.1-beta.58 ✅
- 已有：@solana/web3.js@1.98.4 ✅

## Next Steps
- 建议：添加对更多代币对的支持（当前只支持主流代币）
- 建议：考虑使用 WebSocket 订阅实时价格更新
- 建议：添加价格合理性检查（防止异常数据）

## Issues & Limitations
- 当前依赖 Raydium SDK 的 API，如果 SDK 更新可能需要调整
- 某些小众代币对可能没有对应的 pool

## Token Usage
实际使用：23k tokens (预算：25k tokens) ✅
```

#### Step 4: Controller 整合
Controller 读取输出文档，使用 Sequential Thinking 分析：
```
思考 1：任务完成质量如何？
  → 代码实现完整，测试充分，延迟符合要求

思考 2：是否有需要改进的地方？
  → 建议中提到的 WebSocket 订阅可以作为后续优化任务

思考 3：是否可以启动下一个任务？
  → 可以，Task 1b (Orca) 与 Task 1a 独立，可并发执行

思考 4：是否需要创建新任务？
  → 可以将"WebSocket 实时订阅"作为 Phase 2 的优化任务

决策：
  ✅ Task 1a 成功完成
  ✅ 可以并发启动 Task 1b, 1c, 1d
  📝 记录建议到知识库
```

---

---

## 🎯 使用本模板的步骤

本文档是一个**通用 Web 项目开发方法论模板**。要将其应用到你的具体项目，请按以下步骤操作：

### 1. 定义项目特定的 Subagent 角色

根据你的技术栈和项目需求，在 [Subagent 角色定义](#subagent-角色定义) 部分：

- **保留**：TDD Red/Green/Refactor Specialist（通用）
- **调整**：Frontend/Backend/Database 等角色的具体技术栈
- **添加**：项目特定的专业角色（如：支付集成、推荐算法、实时通信等）

**示例**：
- 电商项目：添加 "Payment Integration Specialist"、"Inventory Management Expert"
- SaaS 项目：添加 "Multi-tenancy Architect"、"Billing System Expert"
- 实时应用：添加 "WebSocket Engineer"、"Message Queue Specialist"

### 2. 更新目录结构

将 [目录结构](#目录结构) 中的示例路径替换为你的项目结构：

```bash
# 替换这些路径为你的实际项目路径
your-web-project/       → my-ecommerce-app/
src/frontend/           → client/
src/backend/            → server/
```

### 3. 替换代码示例

文档中的代码示例（用户认证）仅供参考，将其替换为你的实际功能需求：

- **示例场景**：用户认证 → 你的核心功能（如：商品搜索、订单处理等）
- **技术栈**：Express.js → 你的框架（Django、Spring Boot、Laravel 等）

### 4. 调整任务模板

根据你的开发流程，修改 [任务文档模板](#任务文档task-document)：

- **添加**：项目特定的约束（性能指标、安全要求等）
- **调整**：Token 预算根据任务复杂度
- **更新**：Success Criteria 匹配你的质量标准

### 5. 配置 MCP 环境

按照 [Phase 0: MCP 环境准备](#phase-0-mcp-环境准备) 的步骤，确保：

- Sequential Thinking MCP 正确安装
- 配置文件路径正确
- 测试 MCP 功能正常

### 6. 开始你的第一个任务

从一个小功能开始，验证整个流程：

1. 创建任务文档（参考模板）
2. 启动 Subagent
3. 收集输出
4. 使用 Sequential Thinking 分析
5. 迭代改进

---

## 总结

### 核心价值

这套方法论的核心价值在于：

1. **工程化 AI 协作**：将 AI 辅助开发从"对话式编程"提升到"工程化流程"
2. **知识资产化**：每个任务的输出成为可复用的知识资产
3. **规模化能力**：支持大型复杂项目的系统化开发
4. **质量可控**：通过专业分工和文档化保证质量

### 适用场景

- ✅ 大型项目（>10 个模块）
- ✅ 复杂架构（微服务、多层次）
- ✅ 长期维护（需要知识积累）
- ✅ 团队协作（需要文档化）

### 未来展望

随着 AI 能力的提升，这套方法论可以进一步演化：

1. **自动化工作流**：自动生成任务文档和进度跟踪
2. **智能任务分解**：AI 自动识别依赖关系和并发机会
3. **质量自动检查**：AI 自动审查代码质量和一致性
4. **知识图谱构建**：自动建立任务、代码、知识之间的关联

---

## 💼 常见 Web 项目应用场景

本节提供不同类型 Web 项目的 Subagent 角色配置建议和任务分解示例。

### 场景1：电商平台 (E-Commerce)

#### 推荐 Subagent 配置

```markdown
核心角色：
- Frontend Developer（商品展示、购物车UI）
- Backend Architect（订单API、库存管理）
- Database Architect（商品表、订单表设计）
- Payment Integration Specialist（支付流程）
- Search Engineer（商品搜索和推荐）
- API Security Auditor（支付安全审计）

专业角色：
- Inventory Management Expert（库存系统）
- Email Notification Specialist（订单通知）
- Image Optimization Specialist（商品图片处理）
```

#### 典型任务拆分

```markdown
Phase 1: 商品管理
- Task 1a: 商品数据库设计（Database Architect）
- Task 1b: 商品列表API（Backend Architect）
- Task 1c: 商品展示页面（Frontend Developer）
- Task 1d: 商品搜索功能（Search Engineer）

Phase 2: 购物流程
- Task 2a: 购物车功能（Frontend + Backend）
- Task 2b: 订单创建API（Backend Architect）
- Task 2c: 库存扣减逻辑（Inventory Expert）

Phase 3: 支付集成
- Task 3a: Stripe 支付集成（Payment Specialist）
- Task 3b: 支付安全审计（Security Auditor）
- Task 3c: 订单状态同步（Backend Architect）
```

### 场景2：SaaS 管理平台

#### 推荐 Subagent 配置

```markdown
核心角色：
- Frontend Developer（Dashboard UI）
- Backend Architect（多租户API）
- Database Architect（租户隔离设计）
- Auth & Permission Specialist（RBAC系统）
- Subscription Billing Specialist（订阅计费）

专业角色：
- Multi-tenancy Architect（租户架构）
- Analytics Dashboard Specialist（数据报表）
- Webhook Integration Specialist（第三方集成）
```

#### 典型任务拆分

```markdown
Phase 1: 租户基础
- Task 1a: 多租户数据隔离设计（Multi-tenancy Architect）
- Task 1b: 租户注册和认证（Auth Specialist）
- Task 1c: 租户管理后台（Frontend Developer）

Phase 2: 权限系统
- Task 2a: RBAC 权限模型设计（Auth Specialist）
- Task 2b: 权限API实现（Backend Architect）
- Task 2c: 权限管理界面（Frontend Developer）

Phase 3: 订阅计费
- Task 3a: 订阅计划设计（Billing Specialist）
- Task 3b: Stripe 订阅集成（Payment Specialist）
- Task 3c: 使用量统计（Analytics Specialist）
```

### 场景3：社交/内容平台

#### 推荐 Subagent 配置

```markdown
核心角色：
- Frontend Developer（Feed流、互动UI）
- Backend Architect（内容API、关注系统）
- Database Architect（图关系设计）
- Real-time Specialist（WebSocket、推送）
- Content Moderation Specialist（内容审核）

专业角色：
- Feed Algorithm Specialist（推荐算法）
- Media Processing Specialist（图片/视频处理）
- Notification System Specialist（通知系统）
```

#### 典型任务拆分

```markdown
Phase 1: 用户社交
- Task 1a: 用户关系数据模型（Database Architect）
- Task 1b: 关注/粉丝API（Backend Architect）
- Task 1c: 用户主页UI（Frontend Developer）

Phase 2: 内容发布
- Task 2a: 内容发布API（Backend Architect）
- Task 2b: 图片上传和处理（Media Specialist）
- Task 2c: 内容审核流程（Moderation Specialist）

Phase 3: 实时互动
- Task 3a: WebSocket 服务（Real-time Specialist）
- Task 3b: 实时通知推送（Notification Specialist）
- Task 3c: Feed 流算法（Algorithm Specialist）
```

### 场景4：企业管理系统 (ERP/CRM)

#### 推荐 Subagent 配置

```markdown
核心角色：
- Frontend Developer（表单、列表UI）
- Backend Architect（业务流程API）
- Database Architect（复杂业务模型）
- Report Generator Specialist（报表系统）
- Workflow Engine Specialist（审批流程）

专业角色：
- Excel Import/Export Specialist（数据导入导出）
- Print Template Designer（打印模板）
- Integration Hub Specialist（第三方系统集成）
```

#### 典型任务拆分

```markdown
Phase 1: 基础数据
- Task 1a: 客户/供应商数据模型（Database Architect）
- Task 1b: 基础数据CRUD API（Backend Architect）
- Task 1c: 数据管理界面（Frontend Developer）

Phase 2: 业务流程
- Task 2a: 审批工作流设计（Workflow Specialist）
- Task 2b: 订单管理流程（Backend Architect）
- Task 2c: 流程状态界面（Frontend Developer）

Phase 3: 报表分析
- Task 3a: 报表数据模型（Database Architect）
- Task 3b: 报表生成引擎（Report Generator）
- Task 3c: 可视化图表（Frontend Developer）
```

### 场景5：实时协作工具

#### 推荐 Subagent 配置

```markdown
核心角色：
- Frontend Developer（协作UI、Canvas）
- Backend Architect（协作API）
- Database Architect（文档存储）
- WebSocket Specialist（实时同步）
- CRDT Specialist（冲突解决）

专业角色：
- Document Versioning Specialist（版本控制）
- Presence Management Specialist（在线状态）
- Export & Sharing Specialist（文档分享）
```

#### 典型任务拆分

```markdown
Phase 1: 文档编辑
- Task 1a: 文档数据模型（Database Architect）
- Task 1b: 编辑器UI实现（Frontend Developer）
- Task 1c: 文档保存API（Backend Architect）

Phase 2: 实时协作
- Task 2a: WebSocket 实时通信（WebSocket Specialist）
- Task 2b: CRDT 冲突解决（CRDT Specialist）
- Task 2c: 在线用户展示（Presence Specialist）

Phase 3: 版本和分享
- Task 3a: 版本历史记录（Versioning Specialist）
- Task 3b: 文档分享权限（Backend Architect）
- Task 3c: 导出功能（Export Specialist）
```

### 通用原则

不同项目类型选择 Subagent 的原则：

1. **核心功能优先**：先配置核心业务相关的 Subagent
2. **技术栈匹配**：选择与项目技术栈对应的专家角色
3. **复杂度决定**：简单功能用通用角色，复杂功能创建专门角色
4. **迭代调整**：随项目演进动态增减 Subagent
