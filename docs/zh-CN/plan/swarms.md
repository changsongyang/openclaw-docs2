---
x-i18n:
    generated_at: "2026-07-26T06:13:47Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 90c6c85a837448f4e5ceccdccf73489db801ad502cbbb2f3eb04d6aff7e902f0
    source_path: plan/swarms.md
    workflow: 16
---

# Swarm — 代码模式中的智能体扇出与编排

状态：已发布 — 已由 `docs/tools/swarm.md` 取代。本文档保留作为
实现设计记录。

## 1. 是什么以及为什么

**Swarm** 是通过代码模式脚本以确定性方式编排的多个子智能体：
扇出 N 个读取器，以对抗方式验证发现，通过有状态的优先级排序器进行综合，并在决策门控上循环。
控制流（`Promise.all`、`while`、`if`）_就是_编排本身 — 特意**不提供图 DSL、
不新增模式，也不新增顶层工具表面**。

OpenClaw 代码模式（QuickJS-WASI、快照/恢复、桥接请求）是其
基础。处于停驻状态的桥接调用可以跨越 VM 快照和 Gateway 网关重启继续存在，并从
停止处准确恢复 — 这比日志重放设计更强，且
不对脚本施加确定性约束。

命名：产品/文档名称为 **Swarm**。代码标识符保持原样：
`agents.*` 客体 API、`tools.swarm` 配置、`swarm` 组列。

## 2. 决策（维护者，2026-07-17）

- 成本：强制执行配置上限；每个 Swarm 的 token 预算可选。不强制要求预算。
- 审批：子智能体以**失败时关闭 / 非交互式**方式运行。需要审批的
  操作会被拒绝；拒绝信息会在子智能体结果中报告；由脚本
  决定如何处理。扇出不会向操作员发送大量提示。
- v1 仅支持由模型编写的临时脚本。已保存/命名的工作流、CLI/定时任务
  入口：后续提供（无头代码模式已可用于定时任务）。
- 子智能体身份：默认通过 `tools.swarm.defaultAgentId`
  配置使用专用工作智能体（根据现有子智能体目标允许列表进行验证）；可在每次生成时通过
  `agentId` 覆盖。核心不内置智能体 ID；文档建议使用精简的
  `worker` 智能体配置。
- 不更改 Codex 源代码。Codex harness 使用 spawn/wait 惯用法（§8）。

## 3. 架构概览

```
代码模式脚本（QuickJS VM，Gateway 网关）       Codex V8 脚本（codex 进程）
  agents.run(...) ── 停驻的桥接调用               tools.sessions_spawn / tools.agents_wait
        │                                                │ item/tool/call RPC（每次 ≤600s）
        ▼                                                ▼
             核心（与 harness 无关，位于此仓库）
  sessions_spawn {collect:true, outputSchema, fastMode, groupId}
  agents_wait {ids, timeoutSeconds}
        │
  子智能体注册表（SQLite）：收集器完成记录、Swarm 组 ID
        │
  子智能体 = 普通子智能体会话（受通道上限约束、审批失败时关闭）
        │
  sessions.changed SSE ──► Control UI 圆点 / 侧边栏 / 渠道状态消息
```

生成/完成/结算语义只有一个规范所有者（核心工具 + 注册表）。
有两种等待传输方式：QuickJS 无限期停驻桥接调用（快照）；
Codex 通过有界 RPC 轮询 `agents_wait`。

## 4. 配置门控（v1）

新增 `tools.swarm`（全局 + 每个智能体覆盖，采用与
`tools.codeMode` 相同的合并模式）：

```jsonc
"tools": {
  "swarm": {
    "enabled": false,            // 主门控，默认关闭
    "maxConcurrent": 8,          // 同时运行的子智能体数（Swarm 通道上限）
    "maxChildrenPerGroup": 50,   // 每个 Swarm 组中的活动子智能体数
    "maxTotalPerGroup": 200,     // 每组生命周期内的生成总数（失控防线）
    "waitTimeoutSecondsMax": 600,
    "defaultAgentId": ""         // 可选；生成时省略 agentId 所使用的子智能体 ID
  }
}
```

- Zod：像 `CodeModeSchema`
  （`src/config/zod-schema.agent-runtime.ts`）一样联合 `boolean | strict object`；`swarm: true` → `{enabled: true}`。
- 类型位于 `src/config/types.tools.ts`（每个智能体和顶层 `tools` 均包含），
  标签位于 `schema.labels.ts`，帮助信息位于 `schema.help.runtime.ts`。
- 解析辅助函数 `resolveSwarmConfig(cfg, agentId)` 仿照
  `resolveCodeModeConfig`（`src/agents/code-mode.ts:215`），并限制所有数值。
- 禁用时的门控效果：`agents_wait` 工具不会出现在目录中；
  `sessions_spawn` 上的 `collect`/`outputSchema`/`fastMode`/`groupId` 参数
  会被拒绝，并给出明确指出配置键的错误。其他行为不变。
- `defaultAgentId` 通过 `resolveSubagentAllowedTargetIds`
  （`src/agents/subagent-target-policy.ts`）验证；未知 ID → 生成错误，而非回退。

## 5. 核心：收集器模式生成 + `agents_wait`（v1）

### 5.1 `sessions_spawn` 新增项（均受 Swarm 启用状态门控）

- `collect: boolean` — 为 true 时，子智能体运行会注册到
  `expectsCompletionMessage: false` 并生成一条**收集器完成记录**，
  而不是进行通知/Steering 交付。工具会立即返回 `{ runId, sessionKey }`。
  不绑定渠道/线程。
- `outputSchema: object` — JSON Schema。子智能体的工具表面会追加一个合成的
  `structured_output` 工具；系统提示词附加内容会指示它使用最终结果
  准确调用该工具一次。验证失败时，子智能体会收到一次提醒以重试；再次失败后，完成记录
  将包含 `structured: undefined`、原始文本以及 `schemaError`。
- `fastMode: true | "auto" | false` — 通过 `resolveSubagentModelAndThinkingPlan`
  （`src/agents/subagent-spawn-plan.ts`），沿用现有 `FastMode` 维度
  （`src/shared/fast-mode.ts`），与模型/思考设置一起传入子智能体会话补丁。
  省略 = 继承。
- `groupId: string` — Swarm 组标记。默认为
  `swarm:<requesterSessionKey>:<runId-of-requesting-run>`。持久化到
  注册表记录和子智能体会话行中。用于上限控制、列出、批量
  归档和圆点。
- `label: string` 已存在 — 显示在圆点和 `subagents list` 中。
- 子智能体 ID：`params.agentId` → 否则 `tools.swarm.defaultAgentId` → 否则
  请求方智能体（现有行为）。

### 5.2 审批失败时关闭

收集器子智能体使用非交互式审批上下文运行：任何需要操作员审批的工具调用
都会解析为对子智能体可见的结构化拒绝
（`approval_required`），并期望子智能体在其结果中报告
受阻情况。实现方式：复用现有的 Exec/工具审批
策略管道，并为收集器模式的子智能体运行强制使用 `deny` 解析器。
收集器子智能体不会向操作员表面发送审批事件。

### 5.3 `agents_wait` 工具（新增，受门控）

```
agents_wait({ ids: string[], timeoutSeconds?: number })
→ {
    completed: [{ runId, status: "done"|"failed"|"killed"|"timeout",
                  result: string, structured?: unknown, schemaError?: string,
                  sessionKey, label?, usage?: {inputTokens, outputTokens} }],
    pending: string[]
  }
```

- 只要**至少一个** ID 完成便立即返回（首次完成 / 竞速
  语义，可实现流水线），或在超时时返回 `completed: []`。
- `timeoutSeconds` 默认为 30，并限制到 `waitTimeoutSecondsMax`。
- 幂等：已完成的 ID 会再次返回其记录（记录将
  保留至组归档）。未知 ID → 每个 ID 对应一个错误条目，而不是抛出异常。
- 所有权：仅生成某项运行的会话（或其父级链）可以等待
  该运行 — 与代码模式中的 `wait` 使用相同的所有权规则（`code-mode.ts:1684`）。
- 注册表：完成记录存储在现有的子智能体注册表 SQLite
  存储（`subagent-registry.store.sqlite.ts`）中 — 新增字段，不新增存储，
  不提升架构版本（仅添加列；参见 §9 约束）。

### 5.4 上限执行

- `maxConcurrent`：收集器子智能体在现有子智能体通道上运行，但
  按 Swarm 组计数；超过上限的生成按 FIFO 排队（在主机端的
  生成路径中 — 立即返回 runId，并在插槽空闲时开始运行）。
- `maxChildrenPerGroup` / `maxTotalPerGroup`：超过上限后，生成会返回类型化错误并被拒绝；
  错误文本会指出配置键。
- 深度：收集器子智能体保留 `DEFAULT_SUBAGENT_MAX_SPAWN_DEPTH` 语义
  （除非明确配置嵌套，否则子智能体为叶节点）。

## 6. 测试契约（v1，通道 A）

- 单元测试：配置解析/限制；禁用时的门控拒绝；groupId
  默认值；上限执行（排队 + 拒绝）；等待竞速语义；等待
  幂等性；所有权拒绝；结构化输出验证 + 提醒重试 +
  schemaError 路径；将 fastMode 传入会话补丁；defaultAgentId
  验证。
- 集成测试（vitest，模拟模型运行时）：生成 3 个收集器子智能体，在循环中
  等待，断言首次完成顺序和最终排空；Gateway 网关重启
  模拟：重新加载注册表 → 等待从持久化的完成记录中解析。
- 所有测试与 `*.test.ts` 并置；不进行实时模型调用。

## 7. QuickJS 客体表面（通道 B，在核心之后）

- 客体全局对象安装在 `CONTROLLER_SOURCE`
  （`src/agents/code-mode.worker.ts:190-374`）中，保留名称添加到
  `code-mode-namespaces.ts`：
  - `agents.run(prompt, opts) → Promise<result|structured>` — 语法糖：
    收集器生成 + 在专用桥接方法（`agentWait`）上停驻等待，
    由主机在完成时结算（无轮询；快照安全）。
  - `agents.session(system, opts) → Promise<handle>`；
    `handle.send(input, opts) → Promise<...>`；`handle.close()`。（v1.1 —
    在 run() 之后发布；使用 `mode:"session"` + 每轮收集器记录。）
  - `phase(title)`、`log(message)` — 即发即弃的桥接通知 →
    Swarm 进度事件。
- 添加到 `CodeModeBridgeMethod`（`code-mode.ts:91`）的桥接方法：
  `agentSpawn`、`agentWait`、`swarmNote`。`agentSpawn`/`agentWait`
  **按构造即具有重放安全性**：幂等键 `(codeModeRunId, bridgeId)`
  存储在注册表记录中；重启后根据持久化的完成记录重新结算，
  且绝不会重复生成。
- 待处理的 `agentWait` 桥接调用会延长运行的快照 TTL（待处理的
  智能体集合即为信号；无需标志）。
- `API.read("agents.d.ts")` 虚拟文件记录了类型化表面以及
  扇出 / 门控 / 循环惯用法（`createCodeModeApiVirtualFiles`、
  `code-mode-namespaces.ts:876`）。

## 8. Codex harness 投影（后续通道）

- `sessions_spawn`（包含新参数）和 `agents_wait` 通过
  现有动态工具桥接流转；在 Codex 代码模式脚本中，它们会自动显示为
  `tools.*`（已验证：`codex-rs/code-mode/src/runtime/globals.rs:14-65`、
  `codex-rs/core/src/tools/spec_plan.rs:448-507`）。
- `agents_wait` 使用长时动态工具超时类别（上限 600s；
  `extensions/codex/src/app-server/dynamic-tool-execution.ts:37-39`），并标记为超时/重放安全。
- Codex 父级的组键：`swarm:<parentSessionKey>:<turnId>`。
- Codex 原生 `spawn_agent` 子智能体可以共存；它们的任务镜像行会馈送到
  同一个进度表面。

## 9. 持久化与保留

- 不新增存储。注册表记录扩展现有子智能体注册表
  SQLite 表；子智能体是普通的 `sessions` 行。仅添加列
  — **任何需要提升 SQLite 架构版本的更改都必须先获得
  维护者明确批准**（仓库策略）。
- Swarm 组 ID 位于注册表记录 + 子智能体会话元数据中。
- 保留：已完成的收集器记录会保留至**组归档**：
  当父级运行完成（或 TTL 到期）时，该组的子智能体会
  批量归档（扩展现有 `DEFAULT_SUBAGENT_ARCHIVE_AFTER_MINUTES`
  清理，使其按组运行）。

## 10. 进度表面（“圆点”）— 后续通道

- 隐式，由 harness 驱动。派生自现有 `sessions.changed` SSE +
  注册表；`phase`/`log` 注释补充语义。不由智能体驱动渲染。
- Control UI：工作区小组件系列
  （`ui/src/lib/workspace/widgets/`）中的 `swarm` 渲染器 — 按阶段分组的圆点网格、旁白
  行、每个圆点的状态/标签/模型；侧边栏子树保持不变。
- 渠道：每组一条受节流控制的可编辑状态消息（遵循
  `docs/concepts/streaming.md`；绝不为每个子智能体发送消息）。

## 11. Labs 页面（Control UI，独立工作线）

Settings → **Labs**：实验性功能开关，首批条目为**代码模式**
和 **Swarm**。每一行包含：名称、单行描述、文档链接、通过
现有 `config.patch` RPC 连接的开关（RFC 7396 合并补丁——设置
`tools.codeMode.enabled` / `tools.swarm.enabled`），以及适用时显示的“需要重启”
提示。功能易于发现，但文案会明确说明其实验性质。i18n：所有字符串均通过常规
`en.ts` + 同步流水线处理。

## 12. 部署位置（后续）

- `placement` 生成时选择：`"local"`（默认）| 通过
  现有工作节点环境分派机制（`sessions.dispatch`）使用 `"cloud:<profile>"`；如果共享主机上的 SSH 沙箱子进程
  被证明无法满足需求，再于后续引入池化部署。
- 编排器 VM 始终保留在 Gateway 网关上；归并/点图/预算
  不感知部署位置。

## 13. 非目标

- 不提供图 DSL——控制流就是图（有意如此，并已记录于文档）。
- 不更改 Codex 源码；不复用 Codex 代码模式内部机制。
- v1 不提供已保存/已命名的工作流；不提供 CLI 入口点。
- 不将每个子进程的操作员审批向上冒泡。
- 不在扇出规模下进行 1:1 云端资源预配。
- 不提供稳态运行时兼容垫片；Swarm 是受门控的新功能界面。

## 14. 构建阶段 / PR 拆分

1. **工作线 A（核心）**：§4 配置 + §5 生成/等待/上限/审批 + §6 测试。
2. **工作线 C（Labs 页面）**：§11——独立进行，可以率先合入。
3. **工作线 B（QuickJS 界面）**：§7——在 A 的契约合入后进行。
4. 点图渲染器（§10）、Codex 投影（§8）、`agents.session`（§7 v1.1）、
   部署位置（§12）、用户文档重写——按此顺序通过后续 PR 完成。

每个 PR：CI 通过、`$autoreview` 无问题、默认关闭门控功能、主分支可发布。
