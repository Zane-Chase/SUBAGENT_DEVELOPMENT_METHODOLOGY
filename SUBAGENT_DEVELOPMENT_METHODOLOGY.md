# Subagent 并发开发方法论

> 基于 Sequential Thinking MCP + 文档化工作流的 AI 协作开发方法
>
> **版本**: 0.0.12
> **日期**: 2025-10-06
> **适用**: 大型复杂项目的 AI 辅助开发
> **新增**: Sequential Thinking MCP 强制集成，自动任务规划和并行开发

---

## 📋 目录

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
  - [MCP 使用规范](#6-mcp-使用规范-)
  - [任务设计原则](#1-任务设计原则)
  - [调试日志规范](#11-调试日志规范-)
  - [测试验证规范](#12-测试验证规范-)
  - [文档编写原则](#2-文档编写原则)
  - [Subagent 角色设计](#3-subagent-角色设计)
  - [Controller 决策原则](#4-controller-决策原则)
  - [Token 预算监控](#5-token-预算监控)
- [实际应用示例](#实际应用示例)

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
    │ 数据集成  │  │ 风险管理  │  │ 策略优化  │
    └───────────┘  └───────────┘  └───────────┘
          │              │              │
    ┌─────▼─────┐  ┌─────▼─────┐  ┌────▼──────┐
    │ Subagent4 │  │ Subagent5 │  │ Subagent6 │
    │ 交易执行  │  │ 监控告警  │  │ 测试部署  │
    └───────────┘  └───────────┘  └───────────┘
```

**关键点**：
- Controller 不写具体代码，只做分析和决策
- Subagent 专注于特定领域，职责单一
- 每个 Subagent 有独立的上下文窗口

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
❌ 大任务："实现完整的 Raydium 集成"（预计 80k tokens）

✅ 拆分为 4 个小任务：
  - 任务 1a: 实现价格获取函数（25k）
  - 任务 1b: 实现缓存机制（20k）
  - 任务 1c: 添加错误处理（15k）
  - 任务 1d: 编写测试用例（20k）
```

---

## 架构设计

### 目录结构

```
dex_bot/
├── SUBAGENT_DEVELOPMENT_METHODOLOGY.md   # 本文档（包含所有 Subagent 角色定义）
├── docs/
│   ├── tasks/                            # 任务输入（给 Subagent 的指令）
│   │   ├── phase1/
│   │   │   ├── task-1a-raydium-price-fetch.md
│   │   │   ├── task-1b-orca-price-fetch.md
│   │   │   ├── task-1c-monitoring-setup.md
│   │   │   └── task-1d-shadow-mode.md
│   │   ├── phase2/
│   │   │   ├── task-2a-virtual-wallet.md
│   │   │   └── ...
│   │   └── phase3/
│   │       └── ...
│   │
│   ├── outputs/                          # 任务输出（Subagent 完成的成果）
│   │   ├── phase1/
│   │   │   ├── output-1a-raydium-implementation.md
│   │   │   ├── output-1b-orca-implementation.md
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
│       ├── raydium-integration-guide.md
│       ├── risk-management-patterns.md
│       └── deployment-checklist.md
│
└── src/                                  # 实际代码
    └── ...
```

### Subagent 角色定义

#### 1. Mainnet Data Integration Expert 🔗
- **专长**：Solana DEX 数据集成，精通 Raydium SDK、Orca SDK
- **职责**：实现真实 mainnet 数据源集成
- **工作范围**：`src/context-engineering/rag/`, `src/dex/`

#### 2. Risk Management Architect 🛡️
- **专长**：DeFi 风险管理，套利交易风险控制
- **职责**：设计 Circuit Breaker、资金限额、滑点保护
- **工作范围**：新增风险管理模块

#### 3. Strategy & Learning Optimizer 🧠
- **专长**：AI 策略优化，机器学习决策系统
- **职责**：优化 Prompt Engineering 和 Memory 模块
- **工作范围**：`src/context-engineering/prompt/`, `src/context-engineering/memory/`

#### 4. Transaction Execution Specialist ⚡
- **专长**：Solana 交易执行，DEX swap，Gas 优化
- **职责**：实现真实的交易执行逻辑
- **工作范围**：`src/executor/`, `src/dex/`

#### 5. Monitoring & Dashboard Engineer 📊
- **专长**：系统监控，数据可视化
- **职责**：构建实时监控和前端仪表板
- **工作范围**：`src/monitoring/`, `frontend/`

#### 6. Testing & Deployment Architect 🚀
- **专长**：DevOps，渐进式部署
- **职责**：设计 Shadow Mode、Paper Trading、Canary Deploy
- **工作范围**：`src/phases/`, `tests/`

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
// tests/raydium-price-fetch.test.ts
describe('RaydiumPriceFetcher', () => {
  it('should fetch SOL/USDC price from mainnet', async () => {
    const fetcher = new RaydiumPriceFetcher();
    const price = await fetcher.getPrice('SOL/USDC');

    expect(price).toBeDefined();
    expect(price.value).toBeGreaterThan(0);
    expect(price.source).toBe('raydium');
  });
});

// 运行测试 → ❌ 失败（RaydiumPriceFetcher 类还不存在）
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
// src/raydium-price-fetcher.ts
export class RaydiumPriceFetcher {
  async getPrice(pair: string): Promise<PriceData> {
    // 最简单的实现 - 先让测试通过
    return {
      value: 142.35,  // 硬编码一个合理值
      source: 'raydium'
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
// src/raydium-price-fetcher.ts
import { Connection } from '@solana/web3.js';
import { Liquidity } from '@raydium-io/raydium-sdk';

export class RaydiumPriceFetcher {
  private connection: Connection;

  constructor(rpcUrl: string) {
    this.connection = new Connection(rpcUrl);
  }

  async getPrice(pair: string): Promise<PriceData> {
    const [tokenA, tokenB] = this.parsePair(pair);
    const poolInfo = await this.fetchPoolInfo(tokenA, tokenB);

    return {
      value: this.calculatePrice(poolInfo),
      source: 'raydium',
      timestamp: Date.now()
    };
  }

  private parsePair(pair: string): [string, string] {
    const tokens = pair.split('/');
    if (tokens.length !== 2) {
      throw new Error(`Invalid pair format: ${pair}`);
    }
    return [tokens[0], tokens[1]];
  }

  private async fetchPoolInfo(tokenA: string, tokenB: string) {
    // 真实的 Raydium pool 查询逻辑
    // ...
  }

  private calculatePrice(poolInfo: any): number {
    // 价格计算逻辑
    // ...
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
任务 1a (Raydium)   → 输出 1a  ─┐
任务 1b (Orca)      → 输出 1b  ─┤
任务 1c (Monitor)   → 输出 1c  ─┼→ Controller 整合 → 决策
任务 1d (Shadow)    → 输出 1d  ─┘

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

### Task 1a: Raydium 价格获取
- 状态：✅ 已完成
- Subagent：Mainnet Integration Expert
- 完成时间：2025-10-01
- 输出：docs/outputs/phase1/output-1a-raydium-implementation.md

### Task 1b: Orca 价格获取
- 状态：✅ 已完成
- Subagent：Mainnet Integration Expert
- 完成时间：2025-10-01
- 输出：docs/outputs/phase1/output-1b-orca-implementation.md

### Task 1c: 监控框架
- 状态：🔄 进行中
- Subagent：Monitoring Engineer
- 预计完成：2025-10-02

### Task 1d: Shadow Mode 执行器
- 状态：📅 待开始
- Subagent：Testing Architect

## 决策记录
- [2025-10-01] 决定使用 Helius RPC 作为主节点
- [2025-10-01] 缓存 TTL 设置为 100ms

## 风险和问题
- ⚠️ Raydium SDK 文档不完整，需要查看源码
- ⚠️ RPC 速率限制需要注意

## 下一步行动
1. 完成任务组 A 的剩余任务
2. 开始任务组 B（风险评估、策略优化）
3. 集成所有模块并测试
```

---

## 完整工作流程

### Phase 0: MCP 环境准备

#### Step 1: 安装和配置 Sequential Thinking MCP

**自动检查和安装流程**：

```typescript
// 1. 检查 MCP 是否已安装
async function checkAndInstallMCP() {
  try {
    // 读取 MCP 配置文件
    const mcpConfig = await readFile('C:/Users/53123/AppData/Roaming/Code/User/globalStorage/saoudrizwan.claude-dev/settings/cline_mcp_settings.json');
    const config = JSON.parse(mcpConfig);

    // 检查 sequentialthinking 是否已配置
    if (!config.mcpServers || !config.mcpServers.sequentialthinking) {
      console.log('[INFO] Sequential Thinking MCP 未安装，开始安装...');

      // 更新配置文件
      config.mcpServers = config.mcpServers || {};
      config.mcpServers.sequentialthinking = {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/sequentialthinking"]
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
    "sequentialthinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/sequentialthinking"]
    }
  }
}
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
const requirement = "实现完整的 DEX 交易系统，包括价格获取、风险控制、交易执行和监控";

// MCP 分析并拆分任务
思考 1：识别主要模块
  - 数据集成模块（Raydium、Orca）
  - 风险管理模块
  - 交易执行模块
  - 监控和告警模块

思考 2：分析模块间依赖
  - 数据集成 → 独立，可并发
  - 风险管理 → 依赖数据集成
  - 交易执行 → 依赖数据集成和风险管理
  - 监控 → 独立，可并发

思考 3：划分任务批次
  批次 1（并发）：数据集成 + 监控
  批次 2（并发）：风险管理
  批次 3（并发）：交易执行

思考 4：生成任务文档
  - Task 1a: Raydium 价格获取
  - Task 1b: Orca 价格获取
  - Task 1c: 监控框架搭建
  - Task 2a: 风险评估引擎
  - Task 3a: 交易执行器

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
├── mainnet-integration-expert.md
├── risk-management-architect.md
├── strategy-optimizer.md
├── transaction-specialist.md
├── monitoring-engineer.md
└── testing-architect.md
```

#### Step 2: 任务分解
Controller（我）使用 Sequential Thinking 分析项目，分解为小任务：
```
Phase 1: Shadow Mode
  ├── 任务组 A（并发）
  │   ├── Task 1a: Raydium 价格获取
  │   ├── Task 1b: Orca 价格获取
  │   ├── Task 1c: 监控框架
  │   └── Task 1d: Shadow 执行器
  └── 任务组 B（依赖 A）
      ├── Task 1e: 风险评估
      ├── Task 1f: 策略优化
      └── Task 1g: 性能指标
```

### Phase 2: 执行阶段

#### Step 1: 创建任务文档
Controller 使用 Write tool 创建：
```
docs/tasks/phase1/task-1a-raydium-price-fetch.md
docs/tasks/phase1/task-1b-orca-price-fetch.md
docs/tasks/phase1/task-1c-monitoring-setup.md
docs/tasks/phase1/task-1d-shadow-mode.md
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

#### Step 1: 收集输出
Controller 使用 Read tool 读取所有输出文档：
```
Read docs/outputs/phase1/output-1a-raydium-implementation.md
Read docs/outputs/phase1/output-1b-orca-implementation.md
Read docs/outputs/phase1/output-1c-monitoring-implementation.md
Read docs/outputs/phase1/output-1d-shadow-implementation.md
```

#### Step 2: Sequential Thinking 分析

**使用 Sequential Thinking MCP 规划和拆分任务**

Controller 使用 Sequential Thinking MCP 分析：
```
思考 1：检查所有任务是否成功完成
思考 2：评估代码质量和一致性
思考 3：识别潜在的集成问题
思考 4：决定下一步行动
```

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

### 6. MCP 使用规范 🔧

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

### 案例：Raydium 真实数据集成（使用 TDD 方法）

#### Step 1: 创建任务文档
```markdown
# Task 1a: Raydium Mainnet 价格获取

## Role
Mainnet Data Integration Expert

## Objective
将 market-data-rag.ts 中的 fetchRaydiumPrice() 从模拟实现改为真实 mainnet 数据获取

## Context
- 当前实现返回随机价格（line 163-178）
- 需要使用 @raydium-io/raydium-sdk
- RPC endpoint 在 .env 中配置

## Input Files
- src/context-engineering/rag/market-data-rag.ts (line 163-178)
- .env (SOLANA_RPC_URL)
- package.json (检查依赖)

## Requirements
1. 使用 Raydium SDK 获取真实价格
2. 支持常见代币对（SOL/USDC, SOL/USDT 等）
3. 添加错误处理（网络失败、无效代币对）
4. 添加重试机制（最多 3 次）
5. 保持接口兼容（不改变函数签名）

## Constraints
- 不修改其他函数
- 保持 async/await 模式
- 延迟应 <500ms

## Output Location
docs/outputs/phase1/output-1a-raydium-implementation.md

## Token Budget
预计：25k tokens

## Success Criteria
- [ ] 能成功获取 SOL/USDC 价格
- [ ] 错误处理覆盖所有异常
- [ ] 单元测试通过
```

#### Step 2: 启动 Subagent
Controller 使用 Task tool：
```
请执行以下任务：

你是 Mainnet Data Integration Expert，专精 Solana DEX 集成。

1. 读取任务文档：docs/tasks/phase1/task-1a-raydium-price-fetch.md
2. 按照要求实现功能
3. 编写测试用例
4. 将结果写入：docs/outputs/phase1/output-1a-raydium-implementation.md

确保代码简洁、可维护，并包含必要的注释。
```

#### Step 3: Subagent 输出
```markdown
# Output 1a: Raydium Mainnet 价格获取 - 实现报告

## Status
✅ 已完成

## Modified Files
- src/context-engineering/rag/market-data-rag.ts

## Implementation

### 核心改动
```typescript
import { Connection, PublicKey } from '@solana/web3.js';
import { Liquidity } from '@raydium-io/raydium-sdk';

private async fetchRaydiumPrice(tokenPair: string): Promise<PriceData | null> {
  const maxRetries = 3;
  let attempt = 0;

  while (attempt < maxRetries) {
    try {
      const connection = new Connection(process.env.SOLANA_RPC_URL!);
      const [tokenA, tokenB] = tokenPair.split('/');

      // 查询 Raydium pool
      const poolInfo = await this.getRaydiumPool(tokenA, tokenB);
      if (!poolInfo) {
        throw new Error(`Pool not found for ${tokenPair}`);
      }

      // 计算价格
      const price = poolInfo.baseReserve.toNumber() / poolInfo.quoteReserve.toNumber();

      return {
        tokenA,
        tokenB,
        price,
        volume24h: poolInfo.volume24h,
        liquidity: poolInfo.liquidity,
        timestamp: Date.now(),
        source: 'raydium'
      };

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

## 参考资料

### 理论基础
- [Anthropic: Effective Context Engineering for AI Agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [Anthropic: Claude Code Best Practices](https://www.anthropic.com/engineering/claude-code-best-practices)
- [Agenta.ai: Top Techniques to Manage Context Lengths in LLMs](https://agenta.ai/blog/top-6-techniques-to-manage-context-length-in-llms)

### 技术实现
- [Task Decomposition in Agent Systems - Matoffo](https://matoffo.com/task-decomposition-in-agent-systems/)
- [LLM Context Management: How to Improve Performance and Lower Costs](https://eval.16x.engineer/blog/llm-context-management-guide)

### 最佳实践
- Andrej Karpathy: "The delicate art and science of filling the context window"
- Sequential Thinking MCP: Dynamic and reflective problem-solving

---

**版本历史**:
- v0.0.12 (2025-10-06): 添加 Sequential Thinking MCP 集成，强制并行开发规范
- v0.0.11 (2025-10-06): 添加调试日志规范和测试验证规范，强化代码质量控制
- v0.0.10 (2025-10-02): 整合 TDD Subagent 角色定义到主文档，移除独立角色文件
- v0.0.9 (2025-10-02): 添加 TDD 工作流程和 TDD Subagent 角色定义
- v0.0.8 (2025-10-01): 初始版本，基于 DEX Bot 项目实践总结

**维护者**: Claude AI + 人类协作

**License**: MIT
