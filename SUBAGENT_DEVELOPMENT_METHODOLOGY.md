# Subagent 并发开发方法论

> 基于 Sequential Thinking MCP + 文档化工作流的 AI 协作开发方法
> **通用 Web 项目开发方法论模板**
> **适用**: 大型复杂 Web 项目的 AI 辅助开发
> **更新**: 改造为通用 Web 项目模板,涵盖前端/后端/数据库/DevOps 等全栈开发

---

## 📌 重要说明

**本文档是通用 Web 项目开发方法论模板**

本文档中的所有示例(用户认证、数据库设计等)仅供演示方法论的使用方式。

⚠️ **重要提示**:
- 文档中可能仍有部分来自原始项目的代码示例和场景描述
- 这些示例**仅用于说明方法论**,而非具体实现建议
- 在实际项目中,请根据你的业务领域和技术栈进行相应替换

在应用到你的项目时:
1. **必须**根据项目实际需求定义 Subagent 角色
2. **必须**替换示例代码为你的技术栈和业务场景
3. **建议**参考 `~/.claude/agents/` 中的标准 agent 定义
4. **建议**从小功能开始,逐步扩展到复杂系统


## 📋 目录

- [方法论概述](#方法论概述)
- [核心原理](#核心原理)
- [架构设计](#架构设计)
- [TDD 工作流程](#tdd-工作流程)
- [上下文优化策略](#上下文优化策略)
  - [科学研究基础](#科学研究基础)
  - [三大核心原则](#三大核心原则)
  - [动态上下文分配策略](#动态上下文分配策略)
  - [上下文健康度监控与调优](#上下文健康度监控与调优)
- [文档化工作流](#文档化工作流)
- [完整工作流程](#完整工作流程)
- [优点总结](#优点总结)
- [最佳实践](#最佳实践)
- [TDD 与 Subagent 方法论整合](#tdd-与-subagent-方法论整合)
- [实际应用示例](#实际应用示例)

---

## 方法论概述

### 什么是 Subagent 并发开发?

Subagent 并发开发是一种基于 **AI Agent 角色分工** 和 **文档化知识管理** 的软件开发方法论。它通过以下方式提升开发效率:

1. **Sequential Thinking Controller**(总协调器)负责全局规划和决策
2. **多个专业 Subagent**(通过不同 prompt 定义)并发处理独立任务
3. **文档化工作流** 实现知识持久化和上下文隔离
4. **Token 预算控制** 确保每个 Agent 高效运行

### 适用场景

**最适合**:
- 大型项目(多模块、多层次)
- 复杂系统架构(微服务、多组件集成)
- 长期迭代开发(需要知识积累)
- 团队协作项目(需要文档化沟通)

---

## 核心原理

### 1. 角色分离原则(Separation of Concerns)

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

**关键点**:
- ⚠️ **Controller(主窗口)永远不写代码**: 只做任务规划、分配和进度跟踪
- ⚠️ **所有代码由 Subagent 编写**: 通过独立的 Task tool 调用
- ✅ **Subagent 专注于特定领域**: 职责单一,边界清晰
- ✅ **独立上下文窗口**: 每个 Subagent 有独立的 token 预算
- ✅ **避免 token 累积**: 主窗口只读取输出文档,不参与代码编写

**正确工作流程**:
```
Controller → Sequential Thinking MCP 拆解任务
         → 创建任务文档
         → 启动 Subagent (Task tool)
         → Subagent 写代码并输出文档
         → Controller 读取输出更新进度
         → 主窗口 Token 保持低位
```

### 2. 文档化知识管理(Structured Note-Taking)

**实现方式**:
```
任务 A → 输出文档 A
              ↓
任务 B(读取文档 A)→ 输出文档 B
                            ↓
任务 C(读取文档 B)→ 输出文档 C
```

**优势**:
- 避免上下文累积(每个任务都是轻量级的)
- 知识可复用(文档可被后续任务或人类查阅)
- 可追溯(完整的任务链记录)

### 3. 上下文质量管理(Context Quality Management)

**核心理念**: 优化上下文质量而非限制数量（基于 Liu et al. 2024 研究）

**科学依据**:
- 模型对中间位置信息召回率可低至40%（"Lost in the Middle"现象）
- **信息位置比信息量**对性能影响更大
- 通过位置设计和工具增强，可在大上下文中保持高性能

**优化策略**:
| 组件 | 上下文策略 | 预估Token | 优化重点 |
|------|-----------|----------|---------|
| Controller | 精简指令 | 40k-60k | 只读输出文档 |
| 单个 Subagent | 精准加载 | 20k-40k | 工具检索+位置优化 |
| 并发 Subagent | 独立上下文 | 20k×N | 信息密度>80% |
| 可用总预算 | 1,000k tokens | 100% | 质量优先于数量 |

**并发能力**:
- 简单任务: 20个并发(15K×20 = 300K)
- 中等任务: 15个并发(30K×15 = 450K)
- 复杂任务: 10个并发(60K×10 = 600K)
- 动态调整: 基于信息密度而非固定百分比

### 4. 任务粒度控制(Task Granularity)

**原则**: 一个任务只修改 1-2 个文件,每个文件 200-300 行

**Token 估算公式**:
```
单个任务 Token 使用 = 任务文档 (5k)
                    + 读取代码 (6k)
                    + 生成代码 (10k)
                    + 测试说明 (3k)
                    ≈ 25k tokens
```

**任务拆分示例**:
```
"实现用户管理系统" 拆分为 4 个小任务:
  - 任务 1a: 实现用户认证 API(25k)
  - 任务 1b: 实现权限管理(20k)
  - 任务 1c: 添加密码重置功能(15k)
  - 任务 1d: 编写测试用例(20k)
```

---

## 架构设计

### 目录结构

```
your-web-project/
├── SUBAGENT_DEVELOPMENT_METHODOLOGY.md   # 本文档(包含所有 Subagent 角色定义)
├── docs/
│   ├── tasks/                            # 任务输入(给 Subagent 的指令)
│   │   ├── phase1/
│   │   ├── phase2/
│   │   └── phase3/
│   │
│   ├── outputs/                          # 任务输出(Subagent 完成的成果)
│   │   ├── phase1/
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
│       └── deployment-checklist.md
│
├── src/                                  # 源代码
│   ├── frontend/                         # 前端代码 (React/Next.js等)
│   ├── backend/                          # 后端代码 (Node.js/Python等)
│   └── database/                         # 数据库脚本和迁移
│
└── infrastructure/                       # 基础设施代码 (Terraform/K8s等)
```

### Subagent 角色定义

> **说明**: 以下是通用 Web 项目的 Subagent 角色模板。实际项目应根据具体需求调整角色定义。
> 可参考 `~/.claude/agents/` 目录中的 agent 定义进行扩展。

#### 1. Frontend Developer 🎨
- **专长**: React/Next.js/Vue.js,响应式UI,状态管理
- **职责**: 实现前端组件、页面布局、用户交互
- **工作范围**: `src/frontend/`, `src/components/`

#### 2. Backend Architect 🏗️
- **专长**: RESTful/GraphQL API 设计,微服务架构,事件驱动系统
- **职责**: 设计后端服务、API 接口、服务边界定义
- **工作范围**: `src/backend/`, `src/api/`, `src/services/`

#### 3. Database Architect 🗄️
- **专长**: 数据建模,SQL/NoSQL 数据库选型,性能优化
- **职责**: 设计数据库 schema、索引策略、迁移方案
- **工作范围**: `src/database/`, `migrations/`

#### 4. API Security Auditor 🔒
- **专长**: API 安全审计,OWASP Top 10,认证授权
- **职责**: 安全漏洞扫描、身份验证设计、数据保护
- **工作范围**: 所有 API 端点、认证模块

#### 5. DevOps Engineer 🚀
- **专长**: CI/CD、容器化、Kubernetes、监控告警
- **职责**: 部署自动化、基础设施即代码、故障排查
- **工作范围**: `infrastructure/`, `.github/workflows/`, `k8s/`

#### 6. Full-Stack Integration Specialist 🔗
- **专长**: 前后端集成、API 对接、数据流设计
- **职责**: 确保前后端无缝协作、数据一致性
- **工作范围**: 跨前后端的集成层

#### 7. TDD Red-Phase Specialist 🔴

**专长**: 测试驱动开发,测试用例设计

**核心职责**:
- 编写失败的测试用例来明确功能需求
- 从使用者角度设计清晰的 API
- 覆盖正常、边界、异常场景
- 使用 AAA 模式(Arrange-Act-Assert)

**最佳实践**:
- ✅ 测试名称清晰描述行为
- ✅ 一个测试只验证一个行为
- ✅ 测试独立性(不依赖其他测试)
- ✅ 覆盖边界条件和异常情况

**Token 预算**: ~23k tokens

#### 8. TDD Green-Phase Developer 🟢

**专长**: 最小化实现,快速让测试通过

**核心职责**:
- 用最简单的代码让测试从红色变为绿色
- 允许硬编码和"笨"方法
- 专注当前测试,避免过度设计

**实现策略**:
1. **硬编码优先(Fake It)**: 直接返回测试期望的值
2. **明显实现(Obvious Implementation)**: 如果实现很简单,直接写出来
3. **三角法(Triangulation)**: 通过多个测试用例"三角定位"正确实现

**Token 预算**: ~23k tokens

#### 9. TDD Refactor Specialist 🔧

**专长**: 代码重构,设计优化

**核心职责**:
- 在保持所有测试通过的前提下优化代码质量
- 消除重复代码(DRY 原则)
- 提升可读性和可维护性
- 应用设计模式和最佳实践

**重构技巧**:
1. **提取方法(Extract Method)**: 将长方法拆分为小方法
2. **消除重复(Remove Duplication)**: 提取公共逻辑
3. **引入策略模式**: 替换条件分支
4. **替换魔法数字**: 使用命名常量
5. **简化条件表达式**: 提高可读性

**Token 预算**: ~23k tokens

#### 💡 如何扩展和自定义 Subagent 角色

##### 1. 使用现有角色模板

`~/.claude/agents/` 目录提供了丰富的预定义角色模板:
- **Performance Engineer**: 性能优化、缓存策略
- **Cloud Architect**: 云架构设计、多云部署
- **Data Engineer**: 数据管道、ETL 流程
- **Mobile Developer**: iOS/Android 原生开发
- **AI Engineer**: LLM 集成、RAG 系统

##### 2. 何时创建新角色 vs 使用现有角色

| 场景 | 建议 |
|------|------|
| 标准的前端/后端/数据库任务 | 使用现有通用角色 |
| 特定技术栈(如 Stripe、Kafka、Redis) | 创建专门角色 |
| 领域特定逻辑(支付、推荐算法) | 创建领域专家角色 |
| 一次性简单任务 | 使用通用角色即可 |
| 需要深度领域知识的复杂模块 | 必须创建专门角色 |

---

## TDD 工作流程

### 什么是测试驱动开发 (TDD)?

TDD 是一种软件开发过程,核心理念是**"先写测试,再写实现"**。开发者首先编写一个简短的、描述新功能或改进的自动化测试用例,这个测试在最初必然是失败的。然后,开发者编写最少的代码来让这个测试通过。最后,对新代码进行重构,以符合编码规范。

### TDD 核心流程: 红-绿-重构 (Red-Green-Refactor)

```
┌─────────────────────────────────────────────────────────┐
│                    TDD 循环流程                          │
│                                                         │
│   🔴 RED - 写一个失败的测试                             │
│   - 明确功能需求和期望行为                                │
│   - 测试必须失败(证明功能尚未实现)                        │
│   - 思考输入、输出和边界条件                               │
│   ↓                                                     │
│   🟢 GREEN - 写最少代码让测试通过                         │
│   - 快速实现,不求完美                                    │
│   - 只关注让测试变绿                                      │
│   - 避免过度设计                                         │
│   ↓                                                     │
│   🔧 REFACTOR - 优化代码质量                            │
│   - 消除重复代码                                         │
│   - 提高可读性和可维护性                                  │
│   - 优化设计和性能                                       │
│   - 确保测试依然通过                                     │
│   ↓                                                     │
│   ↻ 回到 RED,开始下一个功能                             │
└─────────────────────────────────────────────────────────┘
```

### TDD 最佳实践

**RED 阶段**:
- 一次只测试一个小功能点
- 测试名称清晰描述行为
- 确保测试失败后再继续

**GREEN 阶段**:
- 写最简单能通过的代码
- 允许硬编码快速验证
- 专注当前测试的功能点

**REFACTOR 阶段**:
- 频繁运行测试确保安全
- 小步重构,每次一个改进
- 消除重复代码(DRY 原则)

---

## 上下文优化策略

> 核心理念：优化上下文质量而非限制数量（Liu et al. 2024）

### 科学基础

**研究结论**（Liu et al. 2024, SWE-bench, Machlab & Battle 2024）：
- 中间位置信息召回率可低至40%（U型曲线）
- 工具设计比窗口大小更重要（性能提升20%）
- 信息组织方式影响召回率差异达40%

### 三大原则

**1. 质量优先**：加载任务相关的5个函数，而非整个文件

**2. 位置敏感**（U型曲线 ~85%/~40%/~80%）：
- 开头区：核心约束、任务目标（10-15K）
- 结尾区：技术规范、代码片段（20-30K）
- 中间区：工具按需检索，预留思考空间 >30%

**3. 工具增强**：
```typescript
tools = {
  get_function_signature(file, func),
  get_class_dependencies(file, class),
  find_usage_examples(func),
  search_codebase(pattern, type)
}
```

### 动态分配策略

| 任务类型 | Token | 优化手段 |
|---------|-------|---------|
| 简单修改 | 15-25K | 目标函数+类型定义 |
| 中等复杂 | 30-50K | 工具检索+位置优化 |
| 复杂重构 | 40-60K/阶段 | 分阶段执行 |

**关键指标**：信息密度 >80%，剩余空间 >30%，中间召回率 >90%

### 五层优化机制

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
[简短的任务目标,1-2 句话]

## Input Files
- path/to/file1.ts (line 100-150)
- path/to/file2.ts (specific function)

## Requirements
1. 具体要求 1
2. 具体要求 2

## Output Location
docs/outputs/[phase]/output-[id]-[name].md

## Token Budget
预计: 25k tokens
```

#### 3️⃣ 输出文档化
```markdown
# 输出文档模板

## Modified Files
- path/to/file1.ts

## Implementation
[核心代码改动或完整实现]

## Testing Results
- ✅ 单元测试通过
- ✅ 集成测试通过

## Next Steps
- 建议 1
- 建议 2
```

#### 4️⃣ 增量式开发
```
任务 1a → 输出 1a (25k)
              ↓
任务 1b (读取输出 1a,5k) → 输出 1b (25k)
                                  ↓
任务 1c (读取输出 1b,5k) → 输出 1c (25k)
```

#### 5️⃣ 并发执行隔离
```
任务 1a (用户认证)   → 输出 1a  ─┐
任务 1b (数据库设计)  → 输出 1b  ─┤
任务 1c (前端UI)     → 输出 1c  ─┼→ Controller 整合 → 决策
任务 1d (API安全)    → 输出 1d  ─┘

每个任务独立上下文: 25k × 4 = 100k tokens
```

### 上下文健康度监控

主动验证 AI 是否真正理解关键信息。

#### "Needle in a Haystack"测试

在任务要求中间埋入测试约束，检查召回率：
```markdown
3. **[测试约束]** 函数名必须以 `auth_` 开头  ← 中间位置
```

**目标**：中间信息召回率 > 90%

#### 关键指标

| 指标 | 阈值 | 优化方向 |
|-----|------|---------|
| 信息密度 | >80% | 删除无关背景 |
| 中间召回率 | >90% | 调整信息位置 |
| 剩余空间 | >30% | 减少预填充 |
| 重启率 | <5% | 拆分任务 |

#### 优化信号

- **反复询问任务目标** → Objective 移到开头并强调
- **违反宪法约束** → 宪法精简为核心10条，放System Prompt开头
- **简单任务Token >70%** → 检查Input Files是否指定具体行号/函数名

---

## 文档化工作流

### 任务文档(Task Document)

**标准格式**:
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

## Requirements
1. 功能要求 1
2. **⚠️ 必须添加终端调试日志**

## Constraints
- 保持与现有接口兼容
- 遵循项目代码规范

## Output Location
docs/outputs/[phase]/output-[id]-[name].md

## Token Budget
预计: 25k tokens

## Success Criteria
- [ ] 功能实现完整
- [ ] 测试通过
- [ ] 文档完善
```

### 输出文档(Output Document)

**标准格式**:
```markdown
# Output [ID]: [任务名称] - 实现报告

## Status
✅ 已完成 / ⚠️ 部分完成 / ❌ 失败

## Modified Files
- src/path/to/file1.ts

## Implementation

### 核心改动
[关键代码片段或完整实现]

### 设计决策
- 决策 1: [原因]

## Testing

### 测试结果
- ✅ 测试 A 通过

## Usage
[代码使用示例]

## Next Steps
- 建议 1

## Token Usage
实际使用: 23k tokens (预算: 25k)
```

### 进度文档(Progress Document)

**标准格式**:
```markdown
# Phase 1: [阶段名称] - 进度跟踪

## 总体状态
🔄 进行中 | 完成度: 60%

## 任务组 A(并发执行)

### Task 1a: 用户认证 API
- 状态: ✅ 已完成
- Subagent: Backend Architect
- 输出: docs/outputs/phase1/output-1a.md

## 决策记录
- [2025-10-01] 决定使用 JWT 认证方案

## 风险和问题
- ⚠️ Session 管理方案待定

## 下一步行动
1. 完成剩余任务
2. 集成测试
```

---

## 完整工作流程

### Phase 0: MCP 环境准备

#### Step 1: 安装和配置 Sequential Thinking MCP

**快速安装(推荐)**:

```bash
claude mcp add-json "sequential-thinking" '{"command":"npx","args":["-y","@modelcontextprotocol/server-sequential-thinking"]}'
```

**验证安装**:

重启 Claude Code 后,应该能看到 `create_thinking_process` 等 MCP 工具。

#### Step 2: 使用 MCP 进行任务规划

**工作流程**:
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

### Phase 1: 准备阶段

#### Step 1: 创建 Subagent 角色定义
```bash
docs/agents/
├── frontend-developer.md
├── backend-architect.md
├── database-architect.md
└── tdd-red-phase-specialist.md
```

#### Step 2: 任务分解
Controller(我)使用 Sequential Thinking 分析项目,分解为小任务:
```
Phase 1: 用户管理模块
  ├── 任务组 A(并发)
  │   ├── Task 1a: 用户认证 API
  │   ├── Task 1b: 数据库 Schema 设计
  │   └── Task 1c: 前端登录组件
  └── 任务组 B(依赖 A)
      └── Task 2a: 权限管理系统
```

### Phase 2: 执行阶段

#### Step 1: 创建任务文档
Controller 使用 Write tool 创建:
```
docs/tasks/phase1/task-1a-user-auth-api.md
docs/tasks/phase1/task-1b-database-schema-design.md
```

#### Step 2: 并发启动 Subagent

**⚠️ 关键原则: 必须并行多开 Subagent 进行并发开发**

**并行开发规则**:
1. **强制并发**: 所有独立任务必须在同一个消息中启动
2. **无数量限制**: 并发 Subagent 数量无上限
3. **依赖检查**: 只有存在依赖关系的任务才能串行
4. **效率优先**: 并发开发效率与并发数量成正比

**并发启动示例**:
```typescript
Controller 在单个消息中启动所有独立任务(无数量限制):

小规模并发(4 个任务): Task 1-4 同时启动
中规模并发(8 个任务): Task 1-8 一次性启动,效率提升 8 倍
大规模并发(15+ 任务): 所有独立任务并行处理
```

**Token 使用**: N × 25k tokens
- 4 个任务: 100k tokens
- 8 个任务: 200k tokens
- 只要在 1M tokens 预算内,可以并发任意数量的任务

### Phase 3: 整合阶段

#### Step 1: 收集输出文档

**Controller 读取输出文档**:

```bash
Read docs/outputs/phase1/output-1a-auth-api-implementation.md
Read docs/outputs/phase1/output-1b-database-schema.md
```

**优势**:
- 输出文档已包含实现摘要和关键信息(~5k tokens)
- 保持主窗口 token 消耗在低位

#### Step 2: 更新进度文档

⚠️ **每次 Subagent 完成任务后必须立即更新进度**

Controller 更新 `docs/progress/phase1-progress.md`

### Phase 4: 迭代

基于前一阶段的输出,规划下一阶段任务,重复 Phase 1-3。

---

## 优点总结

### ✅ 1. Token 使用高效
- Controller 只读输出文档(~5k tokens),不读源代码
- 每个 Subagent 独立上下文,互不干扰
- 文档化复用,避免重复生成相同内容

### ✅ 2. 并发执行高效
- N 个独立任务并发执行,效率提升 N 倍
- 无并发数量上限,只受 1M token 预算限制
- Sequential Thinking MCP 自动识别并发机会

### ✅ 3. 知识可复用
- 输出文档成为知识库,后续任务可直接引用
- 进度文档记录决策,避免重复讨论
- 可追溯的任务链

### ✅ 4. 质量可控
- 专业 Subagent 确保领域质量
- TDD 工作流确保测试覆盖
- 文档化输出便于审查

### ✅ 5. 易于协作
- 清晰的任务分工和输出
- 文档化沟通,降低协作成本
- 进度透明,便于跟踪

### ✅ 6. 容错性强
- 任务失败不影响其他任务
- 可单独重试失败任务
- 文档保留失败信息

### ✅ 7. 可扩展性
- 动态添加 Subagent 角色
- 支持复杂项目的多阶段开发
- 适应不同技术栈和场景

### ✅ 8. 成本优化
- 减少无效 token 消耗
- 并发提升开发效率
- 知识复用降低重复成本

---

## 最佳实践

### 0. 核心工作流强制规范 ⚠️

**Controller 标准工作流**:

```
1. 使用 Sequential Thinking MCP 分析任务
2. 创建任务文档(Write tool)
3. 并发启动 Subagent (Task tool × N)
4. 等待 Subagent 完成
5. 读取输出文档(Read tool)
6. 更新进度文档(Edit/Write tool)
7. 决策下一步(Sequential Thinking MCP)
```

### 1. 任务设计原则

**推荐做法**:
- **单一职责**: 一个任务只做一件事
- **明确边界**: 清晰的输入和输出
- **可测试**: 任务完成后可验证
- **文档先行**: 先写任务文档,再启动 Subagent
- **合理粒度**: 每个任务修改 1-2 个文件
- **明确目标**: 设置清晰的成功标准

### 2. 代码质量和注释规范 📝

**核心原则**: 代码必须包含详细的注释、实现真正的业务逻辑、具备生产环境质量

**必须添加注释的地方**:
- 文件头部注释(描述、功能、依赖)
- 类和接口注释(用法示例)
- 函数注释(参数、返回值、异常)
- 复杂逻辑注释(算法说明)
- TODO 和 FIXME 注释

### 3. 调试日志规范 🔍

**核心原则**: 所有代码必须输出终端调试日志,包括前端和后端代码

**日志要求**:
- 后端日志使用 `console.log/console.error` 输出到终端
- 前端日志使用服务端日志或 Node.js 日志输出到终端
- 关键节点必须记录: 函数入口、出口、关键分支、错误处理
- 包含上下文信息: 参数值、状态信息、时间戳
- 错误必须输出完整堆栈

**日志格式示例**:
```typescript
console.log('[INFO] [ModuleName] Operation started:', params);
console.error('[ERROR] [ModuleName] Failed:', error.message, error.stack);
```

### 4. 测试验证规范 ✅

**核心原则**: 每次代码实现完成后,必须运行项目并检查终端日志

**验证清单**:
- 项目能够成功启动,无启动错误
- 终端日志中没有 ERROR 级别的输出
- 新功能的日志输出完整且清晰
- 错误处理路径有正确的日志输出
- 前端代码的日志在终端可见

### 5. 文档编写原则

- **简洁明确**: 避免冗长描述
- **结构化**: 使用标题、列表、代码块
- **可操作**: 包含具体的步骤和示例
- **及时更新**: 代码变更后同步更新文档

### 6. Subagent 角色设计

- **职责单一**: 每个角色专注一个领域
- **边界清晰**: 定义明确的工作范围
- **可复用**: 设计通用的角色定义
- **文档完整**: 包含专长、职责、工作范围、参考

### 7. Controller 决策原则

- **使用 MCP 分析**: 复杂任务必须使用 Sequential Thinking MCP
- **读文档不读代码**: 只读取输出文档,不查看源代码
- **并发优先**: 独立任务必须并发执行
- **记录决策**: 将重要决策记录到进度文档

### 8. 上下文质量监控

**关键指标**:
- **信息密度**: 任务相关token / 总token > 80%
- **中间召回率**: 通过埋点测试验证 > 90%
- **剩余思考空间**: 生成完成时剩余 > 30%
- **Subagent重启率**: 因上下文问题重启 < 5%

**优化措施**:
- **任务前规划**: 使用 Sequential Thinking 评估复杂度
- **位置优化**: 核心信息放开头/结尾，避免中间
- **工具增强**: 提供专用检索工具，减少预填充
- **埋点测试**: 在要求中插入测试约束验证召回
- **持续改进**: 分析失败案例，优化上下文设计

### 9. 本地开发环境配置规范 🖥️

**推荐配置**:
- 应用代码(前端/后端): 本地运行,支持热更新
- 基础设施(数据库/Redis): Docker 运行,统一版本
- 生产环境: 完全容器化

**最佳实践**:
- 分离关注点: 应用代码(本地) vs 基础设施(Docker)
- 统一版本: 团队使用相同的 Docker 镜像版本
- 快速迭代: 本地运行支持热更新
- 文档化配置: 在 README.md 中说明启动步骤

### 10. MCP 使用规范 🔧

**核心原则**: 使用 Sequential Thinking MCP 进行任务规划和拆分是强制要求

**必须使用 MCP 的场景**:
- 复杂需求分析(涉及多个模块)
- 任务拆分规划(启动 Subagent 之前)
- 依赖关系分析
- 并发机会识别

**MCP 分析质量标准**:
- 识别所有关键模块
- 正确分析依赖关系
- 划分合理的任务批次
- 评估技术风险
- 估算 Token 和时间成本
- 提供并发执行建议

---

## TDD 与 Subagent 方法论整合

### 整合优势

1. **职责分离**: RED/GREEN/REFACTOR 分别由专门的 Subagent 处理
2. **并发测试**: 多个功能的 TDD 循环可以并发执行
3. **知识积累**: 测试用例和重构经验文档化
4. **质量保证**: TDD 确保测试覆盖,Subagent 确保领域质量

### 实际工作流程

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
```

### TDD 任务文档模板

```markdown
# Task [ID]: [功能名称] - TDD 循环

## Phase: RED 🔴
### Role
TDD Red-Phase Specialist

### Objective
为 [功能名称] 编写失败的测试用例

### Requirements
1. 编写测试用例描述预期行为
2. 确保测试失败(功能未实现)
3. 覆盖正常情况和边界情况

### Output Location
docs/outputs/tdd/[feature]-red-phase.md

---

## Phase: GREEN 🟢
### Role
TDD Green-Phase Developer

### Objective
编写最少代码让测试通过

### Input
- 测试文件: tests/[feature].test.ts
- RED 阶段输出: docs/outputs/tdd/[feature]-red-phase.md

### Output Location
docs/outputs/tdd/[feature]-green-phase.md

---

## Phase: REFACTOR 🔧
### Role
TDD Refactor Specialist

### Objective
优化代码质量,保持测试通过

### Input
- GREEN 阶段输出: docs/outputs/tdd/[feature]-green-phase.md

### Output Location
docs/outputs/tdd/[feature]-refactor-phase.md
```

### TDD 并发执行策略

对于多个独立功能,可以并发执行多个 TDD 循环:

```
功能 A: RED → GREEN → REFACTOR  ─┐
功能 B: RED → GREEN → REFACTOR  ─┤
功能 C: RED → GREEN → REFACTOR  ─┼→ Controller 整合
功能 D: RED → GREEN → REFACTOR  ─┘

每个功能独立的 TDD 循环,互不干扰
```

---

## 实际应用示例

### 案例: 用户认证 API 实现(使用 TDD 方法)

**任务文档**:

```markdown
# Task 1a: 用户认证 API 端点

## Role
Backend Architect

## Objective
实现用户注册和登录 API 端点,使用 JWT 认证

## Input Files
- src/services/auth/ (新建)
- src/routes/auth.ts (新建)

## Requirements
1. POST /api/auth/register - 用户注册
2. POST /api/auth/login - 用户登录
3. 使用 bcrypt 加密密码
4. 生成 JWT token (24h 有效期)
5. **⚠️ 必须添加终端调试日志**

## Constraints
- 密码至少 8 位
- Email 格式验证
- 防止重复注册

## Output Location
docs/outputs/phase1/output-1a-auth-api.md

## Token Budget
预计: 25k tokens

## Success Criteria
- 注册和登录接口实现
- 密码正确加密
- JWT token 生成
- 单元测试覆盖率 >80%
- 终端日志完整
```

**输出文档**:

```markdown
# Output 1a: 用户认证 API - 实现报告

## Status
✅ 已完成

## Modified Files
- src/services/auth/auth.service.ts (新建,150行)
- src/routes/auth.ts (新建,50行)
- tests/auth.test.ts (新建,100行)

## Implementation

### 核心改动
实现了 AuthService 类,包含注册和登录方法:
- `register(email, password)`: 创建新用户
- `login(email, password)`: 验证并返回 token
- `generateToken(user)`: 生成 JWT

### 设计决策
- 使用 bcrypt (10 rounds) 加密密码
- JWT secret 从环境变量读取
- 错误统一返回"Invalid credentials"(防止用户枚举)

## Testing

### 测试结果
- ✅ 注册成功: 200 OK
- ✅ 重复注册: 400 Bad Request
- ✅ 登录成功: 200 OK + token
- ✅ 登录失败: 401 Unauthorized
- ✅ 测试覆盖率: 85%

## Usage
```typescript
const authService = new AuthService();
const result = await authService.register('user@example.com', 'password123');
console.log(result.token); // JWT token
```

## Dependencies
- bcrypt@5.1.0
- jsonwebtoken@9.0.2
- @prisma/client@5.0.0

## Next Steps
- 建议: 添加密码重置功能
- 建议: 实现 refresh token 机制

## Token Usage
实际使用: 23k tokens (预算: 25k) ✅
```
