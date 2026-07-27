---
read_when:
    - 将 OpenClaw 运行时数据、缓存、会话记录、任务状态或临时文件迁移到 SQLite
    - 设计从旧版 JSON 或 JSONL 文件迁移的 Doctor 迁移方案
    - 更改备份、恢复、VFS 或工作节点存储行为
    - 移除会话锁、清理、截断或 JSON 兼容路径
summary: 将 SQLite 设为主要持久状态和缓存层、同时保留基于文件的配置的迁移计划
title: 数据库优先的状态重构
x-i18n:
    generated_at: "2026-07-26T06:22:12Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: ae4d72f04c1228742cc7ea3cc87a96b06aa1e2b750ace23cca5b387844746186
    source_path: refactor/database-first.md
    workflow: 16
---

# 数据库优先的状态重构

## 决策

采用两级 SQLite 布局：

- 全局数据库：`~/.openclaw/state/openclaw.sqlite`
- Agent 数据库：每个 Agent 使用一个 SQLite 数据库，用于存储 Agent 所有的工作区、
  转录、VFS、工件以及大型的每 Agent 运行时状态
- 配置继续由文件承载：`openclaw.json` 仍位于
  数据库之外。运行时身份验证配置文件迁移到 SQLite；外部提供商或 CLI
  凭据文件仍由其所有者管理，并位于 OpenClaw 数据库之外。

全局数据库是控制平面数据库。它负责 Agent 发现、
共享 Gateway 网关状态、配对、设备/节点状态、任务和流程账本、插件
状态、调度器运行时状态、备份元数据以及迁移状态。

Agent 数据库是数据平面数据库。它负责 Agent 的会话
元数据、转录事件流、VFS 工作区或暂存命名空间、工具
工件、运行工件以及可搜索、可索引的 Agent 本地缓存数据。

这样既能提供一个持久的全局视图，又不必将大型 Agent 工作区、
转录和二进制暂存数据塞入共享 Gateway 网关写入通道。

## 硬性契约

此迁移只有一种规范运行时形态：

- 会话行仅持久化会话元数据。它们不得持久化
  `transcriptLocator`、转录文件路径、同级 JSONL 路径、锁路径、
  剪枝元数据或文件时代的兼容性指针。
- 转录标识始终采用 SQLite 标识：`{agentId, sessionId}`，并在协议需要时
  附带可选的主题元数据。
- `sqlite-transcript://...` 不是运行时或协议标识。新代码不得
  派生、持久化、传递、解析或迁移转录定位符。运行时和
  测试中完全不应包含伪定位符；文档只能为明确禁止它而提及该字符串。
- 旧版 `sessions.json`、转录 JSONL、`.jsonl.lock`、剪枝、截断
  以及旧会话路径逻辑只能存在于 Doctor 迁移/导入路径中。
- 旧版会话配置别名只能存在于 Doctor 迁移中。运行时不
  解释 `session.idleMinutes`、`session.resetByType.dm`，也不解释
  将另一个已配置 Agent 作为目标的跨 Agent `agent:main:*` 主会话别名。
- 会话路由标识是类型化关系状态。高频运行时和 UI 路径
  应读取 `sessions.session_scope`、`sessions.account_id`、
  `sessions.primary_conversation_id`、`conversations` 和
  `session_conversations`；不得解析 `session_key`，也不得从
  `session_entries.entry_json` 中挖掘提供商标识，但在删除旧调用点期间可将其作为兼容性
  影子数据。
- 渠道级私信标记（例如 `dm` 与 `direct`）属于路由
  词汇，而不是转录定位符或文件存储兼容性句柄。
- 旧版钩子处理程序配置只能存在于 Doctor 警告/迁移界面中。
  运行时不得加载 `hooks.internal.handlers`；钩子只能通过发现的
  钩子目录和 `HOOK.md` 元数据运行。
- 运行时启动、高频回复路径、压缩、重置、恢复、诊断、
  TTS、记忆钩子、子智能体、插件命令路由、协议边界和
  钩子必须在运行时中传递 `{agentId, sessionId}`。
- 测试应通过 `{agentId, sessionId}` 填充并断言 SQLite 转录行。
  仅用于证明 JSONL 路径转发、
  保留调用方提供的定位符或转录文件兼容性的测试应予删除，除非
  它们覆盖 Doctor 导入、非会话支持/调试
  实体化或协议形态。
- `runEmbeddedPiAgent(...)`、预备的工作器运行以及内部嵌入式
  尝试不得接受转录定位符。它们通过 `{agentId, sessionId}` 打开 SQLite 转录
  管理器，并将该管理器传递给内部化的
  PI 兼容 Agent 会话，因此过时的调用方无法让运行器写入
  JSON/JSONL 转录。
- 运行器诊断必须将运行时/缓存/载荷跟踪记录存储在 SQLite 中。
  运行时诊断不得暴露 JSONL 文件覆盖开关或通用的
  转录 JSONL 导出辅助程序；面向用户的导出可以从数据库行实体化明确的
  工件，而不会将文件名重新反馈给运行时。
- 原始流日志使用 `OPENCLAW_RAW_STREAM=1` 加 SQLite 诊断行。
  旧版 pi-mono `PI_RAW_STREAM`、`PI_RAW_STREAM_PATH` 和
  `raw-openai-completions.jsonl` 文件日志记录器契约不属于 OpenClaw
  运行时或测试。
- QMD 记忆索引不得将 SQLite 转录导出为 Markdown 文件。
  QMD 只索引已配置的记忆文件；会话转录搜索继续由
  SQLite 承载。
- 对于新代码，QMD SDK 子路径仅供 QMD 使用。SQLite 会话转录
  索引辅助程序位于 `memory-core-host-engine-session-transcripts`；任何
  QMD 再导出仅用于兼容，运行时代码不得使用。
- 内置记忆索引位于所属 Agent 数据库中。运行时配置和
  已解析的运行时契约不得暴露 `memorySearch.store.path`；Doctor
  删除该旧版配置键，当前代码在内部传递 Agent 的
  `databasePath`。

实现工作应持续删除代码，直到上述陈述全部成立，且除
Doctor/导入/导出/调试边界外不再存在例外。

## 目标状态和进度

### 硬性目标

- 一个全局 SQLite 数据库负责控制平面状态：
  `state/openclaw.sqlite`。
- 每个 Agent 使用一个 SQLite 数据库负责数据平面状态：
  `agents/<agentId>/agent/openclaw-agent.sqlite`。
- 配置继续由文件承载。`openclaw.json` 不属于此次数据库
  重构范围。
- 旧版文件仅作为 Doctor 迁移输入。
- 运行时绝不将会话或转录 JSONL 作为活动状态读写。

### 目标状态

- `not-started`：文件时代的运行时代码仍在写入活动状态。
- `migrating`：Doctor/导入代码可以将文件数据迁移到 SQLite。
- `dual-read`：临时桥接同时读取 SQLite 和旧版文件。此次重构
  禁止出现此状态，除非明确记录为仅供 Doctor 使用。
- `sqlite-runtime`：运行时仅读写 SQLite。
- `clean`：删除旧版运行时 API 和测试，并通过防护措施防止
  回归。
- `done`：文档、测试、备份、Doctor 迁移和变更检查证明
  已达到整洁状态。

### 当前状态

- 会话：运行时处于 `clean`。会话行位于每 Agent 数据库中，
  运行时 API 使用 `{agentId, sessionId}` 或 `{agentId, sessionKey}`，而
  `sessions.json` 是仅供 Doctor 使用的旧版输入。
- 转录：运行时处于 `clean`。转录事件、标识、快照
  和轨迹运行时事件位于每 Agent 数据库中。运行时不再
  接受转录定位符或 JSONL 转录路径。
- PI 嵌入式运行器：`clean`。嵌入式 PI 运行、预备的工作器、压缩
  和重试循环使用 SQLite 会话作用域，并拒绝过时的转录句柄。
- 定时任务：运行时处于 `clean`。运行时使用 `cron_jobs` 和定时任务所有的 `task_runs`；
  运行时测试使用 SQLite `storeKey` 命名，而文件时代的定时任务路径仅保留在
  Doctor 旧版迁移测试中。
- 任务注册表：`clean`。任务和 Task Flow 运行时行位于
  `state/openclaw.sqlite`；未发布的边车 SQLite 导入器已删除。
- 插件状态：`clean`。插件状态/二进制大对象行位于共享全局
  数据库中；已有防护措施阻止使用旧版插件状态边车 SQLite 辅助程序。
- 记忆：内置记忆和会话转录索引处于 `sqlite-runtime`。
  记忆索引表位于每 Agent 数据库中，插件记忆状态使用
  共享插件状态行，而旧版记忆文件是 Doctor 迁移输入
  或用户工作区内容。
- 备份：`sqlite-runtime`。备份会暂存压缩后的 SQLite 快照，忽略活动的
  WAL/SHM 边车文件，验证 SQLite 完整性，并在
  全局数据库中记录备份运行。
- 工作区设置：`sqlite-runtime`。设置完成状态、工作区证明
  和生成的引导哈希位于类型化的共享 SQLite 表中。运行时
  不再读写已退役的工作区 JSON 和 `.attested` 边车文件；
  Doctor 负责对它们进行验证导入并在验证后删除。
- Doctor 迁移：有意保持为 `migrating`。Doctor 将旧版 JSON、
  JSONL 和已退役的边车存储导入 SQLite，记录迁移运行/来源，
  并删除已成功导入的来源。
- Exec 审批：`file-runtime`。TypeScript 和 macOS 仍会读写
  活动状态目录中的 `exec-approvals.json`；预留的
  `exec_approvals_config` 架构尚无运行时所有者。未来切换时必须
  添加同状态 Doctor 导入，并同时迁移两个运行时。
- E2E 脚本：运行时覆盖处于 `clean`。Docker MCP 填充会写入 SQLite
  行。运行时上下文 Docker 脚本仅在
  Doctor 迁移种子中创建旧版 JSONL，并明确命名旧版会话索引路径。

### 剩余工作

- [x] 重命名定时任务运行时测试中的存储变量，使其不再使用 `storePath`，除非
      它们是 Doctor 旧版输入。
      文件：`src/cron/service.test-harness.ts`、
      `src/cron/service.runs-one-shot-main-job-disables-it.test.ts`、
      `src/cron/service/timer.regression.test.ts`、
      `src/cron/service/ops.test.ts`、`src/cron/service/store.test.ts`、
      `src/cron/service.heartbeat-ok-summary-suppressed.test.ts`、
      `src/cron/service.main-job-passes-heartbeat-target-last.test.ts`、
      `src/cron/store.test.ts`。
      证明：`pnpm check:database-first-legacy-stores`；`rg -n 'storePath' src/cron --glob '!**/commands/doctor/**'`。
- [x] 删除或重命名过时的文件时代导出测试模拟。
      文件：`src/auto-reply/reply/commands-export-test-mocks.ts`。
      证明：`rg -n 'resolveSessionFilePath|sessionFile|storePath|transcriptLocator' src/auto-reply/reply`。
- [x] 明确标示 Docker 运行时上下文中的旧版 JSONL 种子仅供 Doctor 使用。
      文件：`scripts/e2e/session-runtime-context-docker-client.ts`。
      证明：`rg -n 'sessions\\.json|sessionFile|\\.jsonl' scripts/e2e/session-runtime-context-docker-client.ts` 仅显示
      `seedBrokenLegacySessionForDoctorMigration`。
- [x] 在任何架构变更后保持 Kysely 生成类型一致。
      文件：`src/state/openclaw-state-schema.sql`、
      `src/state/openclaw-agent-schema.sql`、
      `src/state/*generated*`。
      证明：本轮没有架构变更；`pnpm db:kysely:check`；
      `pnpm lint:kysely`。
- [x] 重新运行所触及存储、命令和脚本的聚焦测试。
      证明：`pnpm test src/cron/service/store.test.ts src/cron/store.test.ts src/cron/service.heartbeat-ok-summary-suppressed.test.ts src/cron/service.main-job-passes-heartbeat-target-last.test.ts src/cron/service.every-jobs-fire.test.ts src/cron/service.persists-delivered-status.test.ts src/cron/service.runs-one-shot-main-job-disables-it.test.ts src/cron/service/ops.test.ts src/cron/service/timer.regression.test.ts src/auto-reply/reply/commands-export-session.test.ts extensions/telegram/src/thread-bindings.test.ts extensions/slack/src/monitor/message-handler/prepare.test.ts src/acp/translator.session-lineage-meta.test.ts`；`git diff --check`。
- [x] 在宣布 `done` 之前，运行变更门禁或远程广泛验证。
      证明：`pnpm check:changed --timed -- <changed extension paths>` 已在
      Hetzner Crabbox 运行 `run_3f1cabf6b25c` 中通过；该运行先进行了临时 Node 24/pnpm 设置，并
      为同步后的无 `.git` 工作区配置了明确的路径路由。

### 不得回归

- 不得使用转录定位符。
- 不得使用活动会话文件。
- 除 Doctor 旧版迁移测试外，不得使用伪造的 JSONL 测试固件。
- 需要使用 Kysely 时，不得直接访问原始 SQLite。
- 不得新增文件时代的数据库迁移。全局架构版本保持为 `1`。
  已发布的每 Agent `1` 版架构只有一次有界运行时迁移，即迁移到
  `2` 版，以获得稳定的记忆源标识。

## 代码阅读假设

没有后续产品决策阻碍此计划。实现应
基于以下假设继续推进：

- 直接使用 `node:sqlite`，并要求此存储路径使用可安全重置 WAL 的 Node 运行时
  （22.22.3+、24.15+ 或 25.9+）。
- 只保留一个常规配置文件。在此次重构中，不要将配置、插件
  清单或 Git 工作区移入 SQLite。
- 不需要运行时兼容文件。旧版 JSON 和 JSONL 文件仅作为
  迁移输入。分支本地的 SQLite 辅助文件从未发布，因此直接删除而不导入。
- `openclaw doctor --fix` 负责从旧版文件迁移到数据库。运行时
  启动仅负责在已发布的 SQLite 模式版本之间进行有限升级；
  不得导入文件时代的状态。
- 凭据兼容性遵循相同规则：运行时凭据存储在
  SQLite 中。旧的 `auth-profiles.json`、每个 Agent 的 `auth.json` 和共享的
  `credentials/oauth.json` 文件作为 Doctor 迁移输入，导入后
  将被删除。
- 生成的模型目录状态由数据库支持。运行时代码不得写入
  `agents/<agentId>/agent/models.json`；现有的 `models.json` 文件是旧版
  Doctor 输入，导入 `agent_model_catalogs` 后将被删除。
- 运行时不得迁移、规范化或桥接转录记录定位器。活动
  转录记录在 SQLite 中的标识是 `{agentId, sessionId}`。文件路径
  仅作为旧版 Doctor 输入，而 `sqlite-transcript://...` 必须从
  运行时、协议、钩子和插件表面中消失，而不能被视为边界句柄。
- 运行时读取 SQLite 转录记录时，不运行旧版 JSONL 条目结构迁移，也不
  为兼容性重写整个转录记录。旧版条目规范化保留在
  显式的 Doctor/导入工具中。Doctor 在插入 SQLite 行之前规范化旧版 JSONL 转录记录
  文件；当前运行时行已使用当前转录记录模式写入。轨迹/会话导出
  按原样读取这些行，不得在导出时执行旧版迁移。
- 旧版转录记录 JSONL 解析/迁移辅助函数仅供 Doctor 使用。运行时
  转录记录格式代码仅构建当前 SQLite 转录记录上下文；Doctor
  负责在插入行之前升级旧版 JSONL 条目。
- 旧的运行时自主管理 JSONL 转录记录流式传输辅助函数已删除。Doctor
  导入代码负责显式读取旧版文件；运行时会话历史记录读取
  SQLite 行。
- Codex app-server 绑定使用 OpenClaw `sessionId` 作为 Codex
  插件状态命名空间中的规范键。`sessionKey` 是用于
  路由/显示的元数据，不得替代持久会话 ID，也不得恢复
  转录记录文件标识。
- 上下文引擎直接接收当前运行时契约。注册表
  不得使用删除 `sessionKey`、
  `transcriptScope` 或 `prompt` 的重试兼容层来包装引擎；无法接受当前
  数据库优先参数的引擎应明确失败，而不是通过桥接兼容。
- 备份输出应保持为单个归档文件。数据库内容应以
  紧凑的 SQLite 快照形式进入该归档，而不是原始的实时 WAL 辅助文件。
- 转录记录搜索很有用，但不是首次数据库优先
  改造所必需的。设计模式时应确保以后可以添加 FTS。
- 在数据库边界稳定之前，工作节点执行应继续作为实验性功能置于设置之后。

## 代码阅读发现

当前分支已经越过概念验证阶段。共享
数据库已存在，Node `node:sqlite` 已通过一个小型运行时辅助函数接入，原有
存储现在写入 `state/openclaw.sqlite` 或所属的
`openclaw-agent.sqlite` 数据库。

剩余工作并非选择 SQLite，而是保持新边界整洁，
并删除任何仍呈现旧文件世界形态的兼容性接口：

- 会话 `storePath` 不再是运行时标识、测试固件结构或
  状态载荷字段。运行时和桥接测试不再包含
  `storePath` 契约名称；Doctor/迁移代码负责该旧版词汇。
- 会话写入不再经过旧的进程内 `store-writer.ts`
  队列。SQLite 补丁写入在事务外进行准备，然后使用短暂的
  同步验证/应用事务，并进行显式冲突检测。
- 旧版路径发现仍有有效的迁移用途，但运行时代码应
  停止将 `sessions.json` 和转录记录 JSONL 文件视为可能的写入
  目标。
- Agent 自有表位于每个 Agent 的 SQLite 数据库中。全局数据库保留
  注册表/控制平面行；转录记录在每个 Agent 的转录记录行中的标识是
  `{agentId, sessionId}`。运行时代码不得持久化转录记录文件
  路径或迁移转录记录定位器。
- Doctor 已导入若干旧版文件。清理工作的目标是将其整理为
  一个由 Doctor 调用的显式迁移实现，并生成持久化的
  迁移报告。

没有其他产品问题阻碍实现。

## 当前代码结构

该分支已具备真正的共享 SQLite 基础：

- 运行时最低版本现在要求使用可安全重置 WAL 的 Node 构建版本：22.22.3+、
  24.15+ 或 25.9+。`package.json`、CLI 运行时防护、安装程序默认值、
  macOS 运行时定位器、CI 和公开安装文档现已保持一致。
- `src/state/openclaw-state-db.ts` 打开 `openclaw.sqlite`，设置 WAL、
  `synchronous=NORMAL`、`busy_timeout=30000`、`foreign_keys=ON`，并应用
  从 `src/state/openclaw-state-schema.sql`
  派生的生成式架构模块。
- Kysely 表类型和运行时架构模块由一次性
  SQLite 数据库生成，这些数据库基于已提交的 `.sql` 文件创建；运行时代码不再
  为全局数据库、每 Agent 数据库或代理捕获数据库保留复制粘贴的
  架构字符串。
- 运行时存储从这些生成的 Kysely `DB` 接口派生查询和插入行类型，
  而不再手动维护 SQLite 行结构的影子定义。原始 SQL
  仍仅限于架构应用、pragma 和仅用于迁移的 DDL。
- 全局 SQLite 架构仍为 `user_version = 1`。每 Agent 架构
  版本为 `2`；其打开器会以原子方式将已发布版本 `1` 的
  内存源键迁移为稳定的整数标识。文件到数据库的导入
  仍由 Doctor 代码负责。
- 在所有权边界具有规范定义的位置强制执行关系所有权：
  源迁移行从 `migration_runs` 级联，任务投递状态
  从 `task_runs` 级联，转录标识行则从
  转录事件级联。
- 当前共享表包括 `agent_databases`、
  `auth_profile_stores`、`auth_profile_state`、
  `plugin_state_entries`、`plugin_blob_entries`、`media_blobs`、
  `skill_uploads`、`capture_sessions`、`capture_events`、`capture_blobs`、
  `sandbox_registry_entries`、`cron_jobs`、`commitments`、
  `delivery_queue_entries`、`model_capability_cache`、
  `workspace_setup_state`、`workspace_path_aliases`、`workspace_attestations`、
  `workspace_generated_bootstrap_hashes`、`native_hook_relay_bridges`、
  `current_conversation_bindings`、`plugin_binding_approvals`、
  `tui_last_sessions`、`acp_sessions`、`acp_replay_sessions`、
  `acp_replay_events`、`task_runs`、`task_delivery_state`、`flow_runs`、
  `subagent_runs`、`migration_runs` 和 `backup_runs`。
- 插件任意拥有的状态不会获得由宿主拥有的类型化表。已安装的
  插件使用 `plugin_state_entries` 存储带版本的 JSON 载荷，并使用
  `plugin_blob_entries` 存储字节，同时支持命名空间/键所有权、TTL 清理、
  备份和插件迁移记录。当宿主拥有查询契约时，由宿主拥有的插件编排状态
  仍可使用类型化表，例如
  `plugin_binding_approvals`。
- 插件迁移是针对插件自有命名空间的数据迁移，而不是宿主
  架构迁移。插件可以通过迁移提供商迁移自己的带版本状态/Blob 条目，
  宿主则在常规迁移账本中记录来源/运行状态。安装新插件
  不需要更改 `openclaw-state-schema.sql`，除非宿主本身要取得
  新跨插件契约的所有权。
- `src/state/openclaw-agent-db.ts` 打开
  `agents/<agentId>/agent/openclaw-agent.sqlite`，在全局数据库中注册该数据库，并拥有 Agent 本地的会话、
  转录、VFS、工件、缓存和内存索引表。共享运行时发现现在读取生成的类型化
  `agent_databases` 注册表，而不再在每个调用点重新实现该查询。
- 全局数据库和每 Agent 数据库都会记录一条 `schema_meta` 行，其中包含数据库角色、
  架构版本、时间戳，以及 Agent 数据库的 Agent ID。全局数据库
  仍为 `user_version = 1`；在完成限定范围的内存源标识迁移后，每 Agent 数据库使用版本 `2`。
- 每 Agent 会话标识现在有一个规范的 `sessions` 根表，以
  `session_id` 为键，并将 `session_key`、`session_scope`、`account_id`、
  `primary_conversation_id`、时间戳、显示字段、模型元数据、
  harness ID 以及父级/派生链接作为可查询列。`session_routes`
  是从 `session_key` 到当前
  `session_id` 的唯一活跃路由索引，因此路由键可以转移到新的持久会话，
  而不会导致热读取需要在重复的 `sessions.session_key` 行之间选择。旧的
  `session_entries.entry_json` 兼容形态载荷通过外键挂接到
  持久的 `session_id` 根；它不再是会话在
  架构层面的唯一表示。
- 每 Agent 外部对话标识也采用关系结构：
  `conversations` 存储规范化的提供商/账户/对话标识，而
  `session_conversations` 将一个 OpenClaw 会话链接到一个或多个外部
  对话。这涵盖共享主私信会话，其中多个对等方可以有意映射到同一个会话，
  而无需在 `session_key` 中提供不实信息。SQLite 还会
  强制自然提供商标识的唯一性，因此相同的
  渠道/账户/类型/对等方/线程元组不能分叉到不同的对话 ID。
  共享主会话中的直接对等方以 `participant` 角色链接，因此一个
  OpenClaw 会话可以表示多个外部私信对等方，而不必将
  较早的对等方降级为含义模糊的相关行。`sessions.primary_conversation_id` 仍
  指向当前的类型化投递目标。关闭状态的路由/状态列
  通过 SQLite `CHECK` 约束强制执行，而不是仅依赖
  TypeScript 联合类型。
  运行时会话投影会先从 `session_entries.entry_json` 清除兼容路由影子字段，
  再应用类型化会话/对话列，因此过期的 JSON 载荷
  无法恢复投递目标。
  子智能体通知路由同样要求类型化 SQLite 投递上下文；
  不再回退到兼容的 `SessionEntry` 路由字段。
  Gateway 网关 `chat.send` 的显式投递继承读取类型化 SQLite
  投递上下文，而不是 `origin`/`last*` 兼容字段。
  `tools.effective` 同样从类型化 SQLite 投递/路由行派生提供商/账户/线程上下文，
  而不是从过期的 `last*` 会话条目影子字段派生。
  系统事件提示词上下文从类型化投递字段重建渠道/目标/账户/线程字段，
  而不是使用 `origin` 影子字段。
  共享的 `deliveryContextFromSession` 辅助函数和会话到对话映射器
  现在完全忽略 `SessionEntry.origin`；只有类型化投递字段
  和关系型对话行可以创建热路由标识。
  运行时会话条目规范化会在持久化或投影 `entry_json` 之前移除
  `origin`，入站元数据则写入类型化渠道/聊天字段
  和关系型对话行，而不再创建新的来源影子字段。
- 转录事件、转录快照和轨迹运行时事件现在
  引用规范的每 Agent `sessions` 根，并在删除会话时级联删除。
  转录标识/幂等性行继续从对应的确切转录事件行级联。
- 内存核心索引现在使用明确的 Agent 数据库表
  `memory_index_meta`、`memory_index_sources`、`memory_index_chunks` 和
  `memory_embedding_cache`，并由 `memory_index_state` 跟踪修订变更。
  可选的 FTS/向量辅助索引命名为 `memory_index_chunks_fts` 和
  `memory_index_chunks_vec`，不再使用通用的 `meta`、`files`、`chunks`、
  `chunks_fts` 或 `chunks_vec` 表。规范名称保留当前的
  路径/来源行结构和序列化嵌入兼容性。这些表属于派生数据/搜索缓存，
  而不是规范的转录存储；可以删除它们，并根据内存工作区文件
  和配置的来源重新构建。
  打开使用已发布通用名称的内存索引时，会将其元数据、来源、
  分块和嵌入缓存迁移到规范表；派生的 FTS/向量
  表会使用规范名称重新构建。
- 子智能体运行恢复状态现在存储在类型化共享 `subagent_runs` 行中，
  并为子会话键、请求方会话键和控制器会话键建立索引。旧的
  `subagents/runs.json` 文件仅作为 Doctor 清理输入。其中的运行条目是
  临时恢复状态，因此 Doctor 会记录停用回执并
  丢弃文件，而不导入内容。由于 SQLite 行被清理后，
  文件无法证明其中的条目处于活动还是过期状态，操作员
  必须先等待文件时代的活动运行结束，再跨越此边界升级。
- 当前对话绑定现在存储在类型化共享
  `current_conversation_bindings` 行中，并以规范化对话 ID 为键；目标 Agent/会话列、
  对话类型、状态、过期时间和元数据均存储为关系型列，
  而不是重复的不透明绑定记录。
  持久绑定键包含规范化对话类型，因此
  直接/群组/渠道引用不会冲突，并且 SQLite 会拒绝无效的绑定
  类型/状态值。旧的
  `bindings/current-conversations.json` 文件仅作为 Doctor 迁移输入。
- 投递队列恢复现在会在重放 JSON 上叠加渠道、目标、
  账户、会话、重试、错误、平台发送和恢复状态的类型化队列列。
  `entry_json` 保留重放载荷、钩子和格式化
  载荷，但类型化列是热队列路由/状态的权威来源。
- TUI 上次会话恢复指针现在存储在类型化共享
  `tui_last_sessions` 行中，并以经过哈希的 TUI 连接/会话作用域为键。
  运行时仅读写 SQLite，以原子方式对每个作用域执行更新插入，并
  排除 Heartbeat 会话。`openclaw doctor --fix` 会严格验证
  旧 TUI JSON 文件，保留较新的 SQLite 行，验证规范结果，
  并删除未发生变化的旧文件，而不是留下归档。
- Discord 命令部署哈希现在存储在共享插件状态 SQLite
  存储中。运行时仅读写精确的应用作用域键。Doctor
  会删除可重建的旧 `discord/command-deploy-cache.json` 文件，
  而不导入内容，因此下次启动会执行一次规范协调。
- 默认 TTS 偏好设置现在存储在共享插件状态 SQLite 行中，其键位于
  `speech-core` 插件下。旧的 `settings/tts.json` 文件仅作为 Doctor 迁移
  输入；运行时不再读写 TTS 偏好设置 JSON 文件，旧路径解析器
  则位于 Doctor 迁移模块中。
- 密钥目标元数据现在使用“存储”表述，不再假装每个
  凭据目标都是配置文件。`openclaw.json` 仍是配置存储；
  身份验证配置文件目标使用类型化 SQLite `auth_profile_stores` 行，
  并将符合提供商结构的凭据保存为 JSON 载荷。
- 密钥审计不再扫描已停用的每 Agent `auth.json` 文件。Doctor 负责
  对该旧文件发出警告、执行导入并将其删除。
- 旧身份验证配置文件路径辅助函数现在位于 Doctor 旧版代码中。核心身份验证
  配置文件路径辅助函数公开 SQLite 身份验证存储标识和显示位置，
  而不是 `auth-profiles.json` 或 `auth-state.json` 运行时路径。
- 子智能体运行恢复和 OpenRouter 模型能力缓存运行时模块
  现在将 SQLite 快照读取器/写入器与仅供 Doctor 使用的旧 JSON
  导入辅助函数分离。OpenRouter 能力使用 `provider_id = "openrouter"` 下类型化的通用
  `model_capability_cache` 行，而不是
  单个不透明缓存 Blob 或提供商专用宿主表。子智能体运行
  `taskName` 存储在类型化 `subagent_runs.task_name` 列中；
  `payload_json` 副本是重放/调试数据，而不是热显示或
  查询字段的数据源。
- `src/agents/filesystem/virtual-agent-fs.sqlite.ts` 在 Agent 数据库
  `vfs_entries` 表之上实现 SQLite VFS。目录读取、递归
  导出、删除和重命名使用带索引的 `(namespace, path)` 前缀范围，
  而不是扫描整个命名空间或依赖 `LIKE` 路径匹配。
- `src/agents/runtime-worker.entry.ts` 为工作进程创建每次运行独立的 SQLite VFS、工具工件、运行工件和限定作用域的缓存存储。
- 工作区引导完成状态、证明时效性和生成的引导哈希现在存储在类型化共享 `workspace_setup_state`、`workspace_path_aliases`、`workspace_attestations` 和 `workspace_generated_bootstrap_hashes` 行中，并以规范工作区身份为键。持久化的词法路径和真实路径别名可在配置的符号链接消失后继续稳定地防止已消失工作区问题；重新指向的别名会以关闭方式失败。运行时不再读取或写入 `openclaw-workspace-state.json`、`.openclaw/workspace-state.json`、状态目录中的 `workspace-attestations/*.attested`，或同级 `<workspace>.attested` 边车文件。`openclaw doctor --fix` 会验证并认领旧版来源，将其连同迁移回执导入 SQLite，验证规范行，然后才移除已认领的文件。
- 共享架构预留了一个 `exec_approvals_config` 单例行，但运行时切换仍待完成。TypeScript 和 macOS 配套应用仍在使用状态作用域的 JSON 文件，必须一同迁移到 SQLite。
- TypeScript 设备身份现在使用类型化 `device_identities` 行，仅供 Doctor 使用的旧版 JSON 导入保留在运行时所有者之外。设备身份验证在协调完成架构和跨运行时迁移前仍由文件支持；`device_auth_tokens` 继续为该后续工作预留。
- GitHub Copilot 令牌交换缓存使用 `github-copilot/token-cache/default` 下的共享 SQLite 插件状态表。它是由提供商拥有的缓存状态，因此有意不添加主机架构表。
- GitHub Copilot 压缩不再写入 `openclaw-compaction-*.json` 工作区边车文件。Harness 针对所跟踪的 SDK 会话调用 SDK 历史记录压缩 RPC，而 OpenClaw 将持久会话和转录状态保存在 SQLite 中，而不是兼容性标记文件中。
- 共享 Swift 运行时（`OpenClawKit`）对设备身份使用相同的 `state/openclaw.sqlite#table/device_identities` 结构和行键。Apple 容器旧版文件由 Swift 迁移所有者导入，因为 TypeScript Doctor 无法访问这些容器。Swift 设备身份验证在协调完成身份验证后续工作前仍由文件支持。
- Android 设备身份和缓存的设备身份验证仍使用应用本地存储。它们需要单独由 Android 所有的迁移；主机 SQLite 声明并不描述当前的 Android 行为。
- Android 通知的近期软件包历史记录使用类型化 `android_notification_recent_packages` 行。运行时不再迁移或读取旧的 SharedPreferences CSV 键。
- 存在旧版 `identity/device.json`、SQLite 身份行无效或无法打开 SQLite 身份存储时，设备身份创建会以关闭方式失败。Doctor 会先导入并移除该文件，因此运行时启动无法在迁移前静默轮换配对身份。
- 设备身份选择使用 SQLite 行键，而不是 JSON 文件定位器。测试和 Gateway 网关辅助函数会传入显式身份键；只有 Doctor 迁移和以关闭方式失败的启动门控知道已停用的 `identity/device.json` 文件名。
- 会话重置兼容性现在位于 Doctor 配置迁移中：`session.idleMinutes` 会移至 `session.reset.idleMinutes`，`session.resetByType.dm` 会移至 `session.resetByType.direct`，而运行时重置策略仅读取规范重置键。
- 旧版配置兼容性现在位于 `src/commands/doctor/` 下。常规 `readConfigFileSnapshot()` 验证不会导入 Doctor 旧版检测器或标注旧版问题；`runDoctorConfigPreflight()` 会添加这些问题，以供 Doctor 修复和报告。Doctor 配置流程会导入 `src/commands/doctor/legacy-config.ts`，旧 OAuth 配置文件 ID 修复位于 `src/commands/doctor/legacy/oauth-profile-ids.ts` 下。
- 非 Doctor 命令不会自动运行旧版配置修复。例如，`openclaw update --channel` 现在遇到无效旧版配置时会失败，并要求用户运行 Doctor，而不是静默导入 Doctor 迁移代码。
- Web 推送、APNs、语音唤醒、更新检查和配置健康现在使用类型化共享 SQLite 表来存储订阅、VAPID 密钥、节点注册、触发行、路由行、更新通知状态和配置健康条目，而不是完整的不透明 JSON Blob。Web 推送和 APNs 写入仅更新插入受影响的主键行；配置健康按配置路径进行协调。它们的运行时模块仍与仅供 Doctor 使用的旧版 JSON 导入辅助函数分离。
- APNs 运行时仅读写 `apns_registrations`。显式 `openclaw doctor --fix` 会严格导入已停用的 `push/apns-registrations.json`，保留现有规范行，验证事务，记录回执，并移除包含密钥的 JSON。基于回执的重试仅执行清理，而 `apns_registration_tombstones` 会涵盖首次修复前的失效项，因此过期的中继授权或设备令牌无法恢复。
- 节点主机配置现在使用共享 SQLite 数据库中的类型化单例行。旧 `node.json` 文件或中断的认领仍存在时，运行时会以关闭方式失败；显式 `openclaw doctor --fix` 会在正常运行时使用前严格导入并移除它。
- 设备/节点配对、渠道配对、渠道允许列表和引导状态现在使用类型化 SQLite 行，而不是完整的不透明 JSON Blob。插件绑定审批和定时任务状态采用相同的拆分方式：运行时模块公开由 SQLite 支持的操作和中立快照辅助函数；配对/引导以及插件绑定审批快照写入按主键协调行，而不是截断表；Doctor 则通过 `src/commands/doctor/legacy/*` 模块导入并移除旧 JSON 文件。
- 已安装插件记录现在存储在 SQLite 已安装插件索引中。运行时配置读写不再迁移或保留旧的 `plugins.installs` 创作配置数据；Doctor 会在正常运行时使用前将该旧版配置结构导入 SQLite。
- QQ Bot 凭据恢复快照现在存储在 `qqbot/credential-backups` 下的 SQLite 插件状态中。运行时不再写入 `qqbot/data/credential-backup*.json`；QQ Bot Doctor 契约会从活动状态目录导入并归档这些旧版备份文件。
- Gateway 网关重载规划会在内部 `installedPluginIndex.installRecords.*` 差异命名空间下比较 SQLite 已安装插件索引快照。运行时重载决策不再将这些行包装到虚假的 `plugins.installs` 配置对象中。
- Matrix 账户凭据现在存储在 SQLite 插件状态中。运行时仅从该规范存储读取；当能够解析相应账户时，Doctor 会导入、验证并归档已停用的 `credentials/matrix/credentials*.json` 文件。
- 核心配对和定时任务运行时模块不再使用旧版 JSON 路径构建器。已弃用的配对路径 SDK 辅助函数仅保留迁移兼容性；Doctor 状态迁移负责其文件读取和导入。Doctor 所有的旧版模块构造 `pending.json`、`paired.json`、`bootstrap.json` 和 `cron/jobs.json` 源路径，仅用于导入测试和迁移。旧版定时任务结构规范化和 JSONL 历史记录导入位于 `src/commands/doctor/cron/` 下；旧版 SQLite 历史记录最终确定会在打开状态数据库时运行。
- `src/commands/doctor/legacy/runtime-state.ts` 从 Doctor 将包括节点主机配置在内的旧版 JSON 状态文件导入 SQLite。新的旧版文件导入器保留在 `src/commands/doctor/legacy/` 下。
- `src/commands/doctor/state-migrations.ts` 将旧版 `sessions.json` 和 `*.jsonl` 转录直接导入 SQLite，并移除成功导入的源文件。它不再通过 `agents/<agentId>/sessions/*.jsonl` 暂存根目录旧版转录，也不会在导入前创建规范 JSONL 目标文件。
- 状态完整性 Doctor 检查不再扫描旧版会话目录，也不再提供删除孤立 JSONL 文件的选项。旧版转录文件仅作为迁移输入，迁移步骤负责导入并移除源文件。
- 旧版沙箱注册表导入位于 `src/commands/doctor/legacy/sandbox-registry.ts` 下；活动沙箱注册表仍仅通过 SQLite 读写。
- 旧版会话转录健康检查/导入修复位于 `src/commands/doctor/legacy/session-transcript-health.ts` 下；运行时命令模块不再包含 JSONL 转录解析或活动分支修复代码。

已完成的整合/删除要点：

- 插件状态现在使用共享的 `state/openclaw.sqlite` 数据库。旧的
  分支本地 `plugin-state/state.sqlite` 辅助文件导入器已移除，因为
  该 SQLite 布局从未发布。探测/测试辅助函数报告共享的
  `databasePath`，不再公开插件状态专用的 SQLite 路径。
- 任务和 Task Flow 运行时表现在位于共享的
  `state/openclaw.sqlite` 数据库中，而不是 `tasks/runs.sqlite` 和
  `tasks/flows/registry.sqlite`；出于同样的布局未发布原因，旧的辅助文件导入器已移除。
- `src/config/sessions/store.ts` 不再需要使用 `storePath` 处理入站
  元数据、路由更新或更新时间读取。命令持久化、CLI
  会话清理、子智能体深度、身份验证覆盖和转录会话
  身份标识均使用智能体/会话行 API。写入以 SQLite 行补丁形式应用，
  并通过乐观冲突重试处理冲突。
- 会话目标解析现在公开按智能体划分的数据库目标，而不是旧版
  `sessions.json` 路径。共享 Gateway 网关、ACP 元数据、Doctor 路由修复和
  `openclaw sessions` 会枚举 `agent_databases` 以及已配置的智能体。
- Gateway 网关会话路由现在使用 `resolveGatewaySessionDatabaseTarget`；
  返回的目标携带 `databasePath` 和候选 SQLite 行键，而不是
  旧版会话存储文件路径。
- 渠道会话运行时类型现在公开 `{agentId, sessionKey}`，用于
  读取更新时间、入站元数据和更新最后路由。旧的
  `saveSessionStore(storePath, store)` 兼容类型已移除。
- 插件运行时、扩展 API 和插件 SDK 会话接口现在公开
  基于 SQLite 的会话行辅助函数，而不是活动会话全存储/文件
  兼容辅助函数。根库兼容导出仅在插件 SDK 之外保留，
  供旧版内部调用方和迁移调用方使用。旧的
  `resolveLegacySessionStorePath` 辅助函数已移除；旧版 `sessions.json` 路径
  构造现在仅用于迁移和测试夹具。
- `src/config/sessions/session-entries.sqlite.ts` 现在将规范会话
  条目存储在按智能体划分的数据库中，并支持行级读取、更新插入和删除补丁。
  运行时更新插入/修补/删除不再扫描大小写变体，也不再
  清理旧版别名键；规范化由 Doctor 负责。独立的
  JSON 导入辅助函数已移除，迁移会更新插入较新的行，
  而不是替换整个会话表。公共读取/列出/加载辅助函数
  从类型化的 `sessions` 和 `conversations` 行中投影热会话元数据；
  `entry_json` 是兼容/调试影子数据，即使其陈旧或无效，
  也不会丢失类型化会话身份或投递上下文。
- `src/config/sessions/delivery-info.ts` 现在从类型化的按智能体划分
  `sessions` + `conversations` + `session_conversations` 行中解析投递上下文。
  它不再从 `session_entries.entry_json` 重建运行时投递身份；
  缺少类型化对话行属于 Doctor 迁移/修复问题，而不是运行时回退场景。
- 已存储会话的重置决策现在优先使用类型化的 `sessions.session_scope`、
  `sessions.chat_type` 和 `sessions.channel` 元数据。`sessionKey` 解析
  仅为命令目标上的显式线程/主题后缀保留；群组与
  私信的重置分类不再源自键的形状。
- 会话列表/状态的显示分类现在使用类型化聊天元数据和
  Gateway 网关会话种类。它不再将 `session_key` 内的 `:group:` 或
  `:channel:` 子字符串视为持久可靠的群组/私信事实。
- 静默回复策略选择现在仅使用显式对话类型或界面
  元数据。它不再根据 `session_key` 子字符串猜测私信/群组策略。
- 会话显示模型解析现在从 SQLite 会话数据库目标接收
  智能体 ID，而不是从 `session_key` 中拆分出来。
- 智能体间通告目标填充现在仅使用类型化的 `sessions.list`
  `deliveryContext`。它不再从旧版 `origin`、镜像的
  `last*` 字段或 `session_key` 形状中恢复渠道/账户/线程路由。
- `sessions_send` 线程目标拒绝现在读取类型化 SQLite 路由
  元数据。它不再通过从目标键中解析线程后缀来拒绝或接受目标。
- 群组范围的工具策略验证现在读取当前会话或已生成会话的
  类型化 SQLite 对话路由。它不再通过解码 `sessionKey` 来信任群组/渠道
  身份；如果没有类型化会话行予以确认，调用方提供的群组 ID 会被丢弃。
- 渠道模型覆盖匹配现在使用显式群组和父级
  对话元数据。它不再从 `parentSessionKey` 解码父级对话 ID。
- 已存储模型覆盖的继承现在要求类型化会话上下文提供显式父级会话键。
  它不再根据 `sessionKey` 中的 `:thread:` 或
  `:topic:` 后缀推导父级覆盖。
- 旧的会话线程信息包装器和已加载插件线程解析器已移除；
  运行时代码不再导入 `config/sessions/thread-info`。
- 渠道对话辅助函数不再公开完整会话键解析
  桥接。核心仍通过 `resolveSessionConversation(...)` 规范化提供商拥有的原始对话 ID，
  但不会从 `sessionKey` 重建路由事实。
- 完成投递、发送策略和任务维护不再从 `session_key`
  形状推导聊天类型。旧的聊天类型键解析器已删除；
  这些路径要求类型化会话元数据、类型化投递上下文或
  显式投递目标词汇。
- 会话列表/状态、诊断、审批账户绑定、TUI Heartbeat
  筛选和用量摘要不再从 `SessionEntry.origin` 中挖掘
  提供商/账户/线程/显示路由信息。运行时仅存的
  `origin` 读取均用于非会话概念或当前轮次投递对象。
- 审批请求的原生对话查找现在读取类型化的按智能体划分会话
  路由行。它不再从 `sessionKey` 解析渠道/群组/线程对话身份；
  缺少类型化元数据属于迁移/修复问题。
- Gateway 网关会话变更/聊天/会话事件载荷不再回显
  `SessionEntry.origin` 或 `last*` 路由影子数据；客户端会收到类型化的
  `channel`、`chatType` 和 `deliveryContext`。
- Heartbeat 投递解析现在可以直接接收类型化 SQLite
  `deliveryContext`，Heartbeat 运行时会传递按智能体划分的
  会话投递行，而不是依赖兼容的 `session_entries`
  影子数据来确定当前路由。
- 定时任务隔离智能体投递目标解析也会先从类型化的
  按智能体划分会话投递行填充当前路由，然后才回退到
  兼容条目载荷。
- 子智能体通告来源解析现在通过 `loadRequesterSessionEntry` 传递
  类型化请求方会话投递上下文，并优先使用该行，而不是
  兼容的 `last*`/`deliveryContext` 影子数据。
- 入站会话元数据更新现在会先与类型化的按智能体划分
  投递行合并；仅当不存在类型化对话行时，才回退使用旧的
  `SessionEntry` 投递字段。
- 重启/更新投递提取现在会让类型化 SQLite 投递
  `threadId` 优先于从 `sessionKey` 解析的主题/线程片段；
  解析仅作为旧版线程形状键的回退方案。
- Hook 智能体上下文渠道 ID 现在优先使用类型化 SQLite 对话身份，
  其次使用显式消息元数据。它们不再从 `sessionKey` 解析提供商/群组/渠道
  片段。
- Gateway 网关 `chat.send` 外部路由继承现在读取类型化 SQLite 会话
  路由元数据，而不是根据 `sessionKey` 片段推断渠道/私信/群组范围。
  仅当类型化会话渠道和聊天类型与已存储的投递上下文匹配时，
  渠道范围的会话才会继承；共享主会话继续采用更严格的
  CLI/无客户端元数据规则。
- 重启哨兵唤醒和继续路由现在会先读取类型化 SQLite
  投递/路由行，然后再将 Heartbeat 唤醒或已路由的智能体轮次继续项加入队列。
  它不再从会话条目 JSON 影子数据重建投递上下文。
- Gateway 网关 `tools.effective` 上下文解析现在读取类型化 SQLite
  投递/路由行，以获取提供商、账户、目标、线程和回复模式
  输入。它不再从陈旧的 `session_entries.entry_json` 来源影子数据中恢复这些热路由字段。
- 实时语音咨询路由现在从类型化的按智能体划分 SQLite 会话行中
  解析父级/通话投递。在选择嵌入式智能体
  消息路由时，它不再回退到兼容的 `SessionEntry.deliveryContext` 影子数据。
- ACP 生成 Heartbeat 中继和父级流路由现在从类型化 SQLite
  会话行读取父级投递。它们不再从兼容会话条目影子数据
  重建父级投递上下文。
- 会话投递路由保留现在遵循类型化聊天元数据和
  持久化投递列。它不再从 `sessionKey` 提取渠道提示、私信/主会话
  标记或线程形状；只有当 SQLite 已为该会话保存类型化/持久化
  投递身份时，内部 Webchat 路由才会继承外部目标。
- 通用会话投递提取现在仅读取精确的类型化 SQLite
  会话投递行。它不再解析线程/主题后缀，也不再
  从线程形状键回退到基础会话键。
- 回复分派、重启哨兵恢复和实时语音咨询路由
  现在使用精确的类型化 SQLite 会话/对话行处理线程路由。它们
  不再通过解析线程形状的会话键来恢复线程 ID 或基础会话投递上下文。
- 嵌入式 PI 历史记录限制现在使用类型化 SQLite 会话路由
  投影（`sessions` + 主 `conversations`）获取提供商、聊天类型
  和对端身份。它不再从 `sessionKey` 中解析提供商、私信、群组或线程形状。
- 定时任务工具投递推断现在仅使用显式投递或当前类型化
  投递上下文。它不再从 `agentSessionKey` 解码渠道、对端、账户或线程目标。
- 运行时会话行不再携带旧的 `lastProvider` 路由别名。
  辅助函数和测试使用类型化的 `lastChannel` 和 `deliveryContext` 字段；
  Doctor 迁移是唯一应转换旧路由别名或持久化
  `origin` 影子数据的位置。
- 转录事件、VFS 行和工具工件行现在写入按智能体划分的
  数据库。未发布的全局转录文件映射表已移除；Doctor
  改为在持久迁移行中记录旧版源路径。
- 运行时转录查找不再扫描 JSONL 字节偏移量或探测旧版
  转录文件。Gateway 网关聊天/媒体/历史记录路径从 SQLite
  读取转录行；会话 JSONL 现在仅作为旧版 Doctor 输入，
  而不是运行时状态或导出格式。
- 转录的父级和分支关系使用 SQLite 转录
  标头中的结构化 `parentTranscriptScope: {agentId, sessionId}` 元数据，而不是路径形式的
  `agent-db:...transcript_events...` 定位器字符串。
- 转录管理器契约不再公开隐式持久化的
  `create(cwd)` 或 `continueRecent(cwd)` 构造函数。持久化转录
  管理器使用显式 `{agentId, sessionId}` 作用域打开；仅
  内存管理器在测试和纯转录转换中仍不受作用域约束。
- 运行时转录存储 API 解析 SQLite 作用域，而非文件系统路径。旧的
  `resolve...ForPath` 辅助函数和未使用的 `transcriptPath` 写入选项
  已从运行时调用方中移除。
- 运行时会话解析现在使用 `{agentId, sessionId}`，且不得为外部边界派生
  `sqlite-transcript://<agent>/<session>` 字符串。
  旧版绝对 JSONL 路径仅作为 Doctor 迁移输入。
- 原生钩子中继的直接桥接记录现在存储在按中继 ID 索引的强类型共享
  `native_hook_relay_bridges` 行中。运行时不再为这些短期桥接记录写入
  `/tmp` JSON 注册表或不透明的通用记录。
- `runEmbeddedPiAgent(...)` 不再具有转录定位器参数。
  预备好的工作进程描述符也不再包含转录定位器。运行时会话
  状态和排队的后续运行携带 `{agentId, sessionId}`，而不是
  派生的转录句柄。
- 嵌入式压缩现在从 `agentId` 和 `sessionId` 获取 SQLite 作用域。
  压缩钩子、上下文引擎调用、CLI 委派和协议回复
  不得接收派生的 `sqlite-transcript://...` 句柄。导出/调试代码
  可以从行中生成显式用户工件，但不会提供
  通用会话 JSONL 导出路径，也不会将文件名反馈到运行时
  身份中。
- `/export-session` 从 SQLite 读取转录行，并且只写入请求的
  独立 HTML 视图。嵌入式查看器不再根据这些行重建或
  下载会话 JSONL。
- 上下文引擎委派不再通过解析转录定位器来恢复
  Agent 身份。预备好的运行时上下文会将已解析的 `agentId`
  传入内置压缩适配器。
- 转录重写和实时工具结果截断现在通过 `{agentId, sessionId}`
  读取并持久化转录状态，并且不会为转录更新事件载荷派生
  临时定位器。
- 转录状态辅助函数表面不再具有基于定位器的
  `readTranscriptState`、`replaceTranscriptStateEvents` 或
  `persistTranscriptStateMutation` 变体。运行时调用方必须使用
  `{agentId, sessionId}` API。Doctor 导入通过显式文件
  路径读取旧版文件并写入 SQLite 行；它不会迁移定位器字符串。
- 运行时会话管理器契约不再公开 `open(locator)`、
  `forkFrom(locator)` 或 `setTranscriptLocator(...)`。持久化会话
  管理器仅通过 `{agentId, sessionId}` 打开；列表/分叉辅助函数位于
  面向行的会话和检查点 API 上，而不是转录管理器
  门面上。
- Gateway 网关转录读取器 API 以作用域为先。它们接受
  `{agentId, sessionId}`，且不接受可能意外成为运行时身份的
  位置转录定位器。活动转录定位器解析已移除；旧版源路径
  仅由 Doctor 导入代码读取。
- 转录更新事件也以作用域为先。`emitSessionTranscriptUpdate`
  不再接受裸定位器字符串，监听器通过
  `{agentId, sessionId}` 路由，而不解析句柄。
- Gateway 网关会话消息广播从 Agent/会话
  作用域解析会话键，而不是从转录定位器解析。旧的转录定位器到会话
  键解析器/缓存已移除。
- Gateway 网关会话历史 SSE 按 Agent/会话作用域筛选实时更新。它不再
  规范化转录定位器候选项、真实路径或文件形态的
  转录身份来决定流是否应接收更新。
- 会话生命周期钩子不再在 `session_end` 上派生或公开转录定位器。
  钩子使用方会获得 `sessionId`、`sessionKey`、下一会话
  ID 和 Agent 上下文；转录文件不属于生命周期
  契约。
- 重置钩子也不再派生或公开转录定位器。
  `before_reset` 载荷携带恢复的 SQLite 消息以及重置
  原因，而会话身份保留在钩子上下文中。
- Agent harness 重置不再接受转录定位器。重置分发
  通过 `sessionId`/`sessionKey` 加原因来限定作用域。
- Agent 扩展会话类型不再公开 `transcriptLocator`；扩展
  应使用会话上下文和运行时 API，而不是获取
  文件形态的转录身份。
- 插件压缩钩子不再公开转录定位器。钩子上下文
  已携带会话身份，转录读取必须通过可感知 SQLite
  作用域的 API，而不是文件形态的句柄。
- `before_agent_finalize` 钩子不再公开 `transcriptPath`，包括
  原生钩子中继载荷。终结钩子仅使用会话上下文。
- Gateway 网关重置响应不再在返回的条目上合成转录定位器。
  重置操作会创建 SQLite 转录行，返回干净的
  会话条目，并将转录访问留给可感知作用域的读取器。
- 嵌入式运行和压缩结果不再公开用于
  会话统计的转录定位器。自动压缩仅更新活动的 `sessionId`、
  压缩计数器和令牌元数据。
- 嵌入式尝试结果不再返回 `transcriptLocatorUsed`，上下文引擎
  `compact()` 结果也不再返回转录定位器。
  运行时重试循环仅接受后继 `sessionId`。
- 投递镜像转录追加结果不再返回转录
  定位器。调用方会获得已追加的 `messageId`；转录更新信号使用
  SQLite 作用域。
- 父会话分叉辅助函数仅返回分叉后的 `sessionId`。子智能体
  准备过程将子 Agent/会话作用域传给引擎。
- CLI 运行器参数和历史记录重新播种不再接受转录定位器。
  CLI 历史记录读取从 `{agentId,
sessionId}` 和会话键上下文解析 SQLite 转录作用域。
- CLI 和嵌入式运行器测试夹具现在按会话 ID 播种和读取 SQLite 转录行，
  而不是假装活动会话是 `*.jsonl` 文件，或
  通过运行时参数传递 `sqlite-transcript://...` 字符串。
- 即使内存管理器没有派生定位器，会话工具结果防护事件也会从已知会话作用域发出。
  其测试不再伪造活动的
  `/tmp/*.jsonl` 转录文件。
- BTW 和压缩检查点辅助函数现在按 SQLite 作用域读取和分叉转录行。
  检查点元数据现在仅存储会话 ID 和叶节点/条目 ID；
  派生定位器不再写入检查点载荷。
- Gateway 网关转录键查找在协议边界使用 SQLite 转录作用域，
  不再对转录文件名调用 realpath 或 stat。
- 自动压缩转录轮换通过 SQLite 转录存储直接写入后继转录行。
  会话行仅保留后继会话身份，而不保留持久的 JSONL 路径或持久化定位器。
- 嵌入式上下文引擎压缩使用以 SQLite 命名的转录轮换
  辅助函数。轮换测试不再构造 JSONL 后继路径，也不再
  将活动会话建模为文件。
- 托管式传出图像保留机制根据 SQLite 转录统计信息为其转录消息缓存生成键，
  而不是调用文件系统 stat。
- 运行时会话锁和独立的旧版 `.jsonl.lock` Doctor
  通道已移除。
- Microsoft Teams 运行时 barrel 和公共插件 SDK 不再重新导出
  旧的文件锁辅助函数；持久插件状态路径由 SQLite 支持。
- 会话按时长/数量清理和显式会话清理已移除。
  Doctor 负责旧版导入；过期会话会被显式重置或删除。
- Doctor 完整性检查不再将旧版 JSONL 文件视为 SQLite 会话行的有效活动
  转录。活动转录健康状态仅以 SQLite 为准；
  旧版 JSONL 文件会报告为迁移/孤立项清理输入。
- Doctor 不再将 `agents/<agent>/sessions/` 视为必需的运行时
  状态。仅当该目录已存在时，才会扫描它，并将其作为旧版导入
  或孤立项清理输入。
- Gateway 网关 `sessions.resolve`、会话修补/重置/压缩路径、子智能体
  生成、快速中止、ACP 元数据、Heartbeat 隔离会话和 TUI
  修补不再在正常运行时工作中附带迁移或清理旧版会话键。
- CLI 命令会话解析现在返回所属的 `agentId`，而不是
  `storePath`，并且在正常的 `--to` 或 `--session-id`
  解析期间不再复制旧版主会话行。旧版主行规范化
  仅属于 Doctor。
- 运行时子智能体深度解析不再读取 `sessions.json` 或 JSON5
  会话存储。它按 Agent ID 读取 SQLite `session_entries`，旧版
  深度/会话元数据只能通过 Doctor 导入路径进入。
- 身份验证配置文件会话覆盖通过直接的 `{agentId, sessionKey}`
  行 upsert 持久化，而不是延迟加载文件形态的会话存储运行时。
- 自动回复详细输出门控和会话更新辅助函数现在按会话身份读取/upsert SQLite
  会话行，并且在操作持久化行状态前不再需要旧版存储路径。
- 命令运行会话元数据辅助函数现在使用面向条目的名称和模块
  路径；旧的 `session-store` 命令辅助函数表面已移除。
- 引导标头播种和手动压缩边界强化现在直接修改
  SQLite 转录行。运行时调用方传递会话身份，而不是
  可写的 `.jsonl` 路径。
- 静默会话轮换重放按 `{agentId, sessionId}` 从 SQLite 转录行复制
  最近的用户/助手轮次。它不再接受
  源或目标转录定位器。
- 新的运行时会话行不再存储转录定位器。调用方直接使用
  `{agentId, sessionId}`；导出/调试命令在生成行时可以选择输出文件
  名称。
- 启动新的持久化转录会话时，现在始终按作用域打开 SQLite 行。
  会话管理器不再复用之前文件时代的转录
  路径或定位器作为新会话的身份。
- 持久化转录会话使用显式的
  `openTranscriptSessionManagerForSession({agentId, sessionId})` API。旧的
  静态 `SessionManager.create/openForSession/list/forkFromSession` 门面已
  移除，因此测试和运行时代码无法意外地重新创建文件时代的会话
  发现机制。
- 插件运行时不再公开 `api.runtime.agent.session.resolveTranscriptLocatorPath`；
  插件代码使用 SQLite 行辅助函数和作用域值。
- 公共 `session-store-runtime` SDK 表面现在仅导出会话行
  和转录行辅助函数。专用的 SQLite 架构/路径/事务辅助函数
  位于 `sqlite-runtime`；原始打开/关闭/重置辅助函数仍仅供
  第一方测试在本地使用。
- 旧版 `.jsonl` 轨迹/检查点文件名分类器现在位于
  Doctor 旧版会话文件模块中。核心会话验证不再导入
  文件工件辅助函数来判断正常的 SQLite 会话 ID。
- 主动记忆阻塞式子智能体运行使用 SQLite 转录行，而不是
  在插件状态下创建临时或持久化的 `session.jsonl` 文件。旧的
  `transcriptDir` 选项已移除。
- 一次性短名称生成和系统 Agent 规划器运行使用 SQLite 转录行，
  而不是创建临时 `session.jsonl` 文件。
- `llm-task` 辅助运行和隐藏的跟进承诺提取也使用 SQLite
  转录行，因此这些仅供模型使用的辅助会话不再创建
  临时 JSON/JSONL 转录文件。
- `TranscriptSessionManager` 现在只是一个已打开的 SQLite 转录作用域。
  运行时代码使用 `openTranscriptSessionManagerForSession({agentId,
sessionId})` 打开它；创建、分支、继续、列出和派生流程位于其
  所属的 SQLite 行辅助函数中，而不是静态管理器门面中。
  Doctor/导入/调试代码在
  运行时会话管理器之外处理显式的旧版源文件。
- 过时的 `SessionManager.newSession()` 和
  `SessionManager.createBranchedSession()` 门面方法已被移除。新
  会话及其转录后代由所属的 SQLite
  工作流创建，而不是通过改变已打开的管理器，使其指向另一个
  持久化会话。
- 父转录派生决策和派生创建不再接受
  `storePath` 或 `sessionsDir`；它们使用 `{agentId, sessionId}` SQLite
  转录作用域，而不是保留的文件系统路径元数据。
- Memory-host 不再导出无操作的会话目录转录
  分类辅助函数；现在，转录筛选在构造条目时从 SQLite 行
  元数据派生。
- Memory-host 和 QMD 会话导出测试使用 SQLite 转录作用域。旧
  `agents/<agentId>/sessions/*.jsonl` 路径仅在测试
  有意验证 Doctor/导入/导出兼容性时继续覆盖。
- QA-lab 原始会话检查现在通过 Gateway 网关使用 `sessions.list`，
  而不是读取 `agents/qa/sessions/sessions.json`；MSteams 反馈
  直接追加到 SQLite 转录中，不再虚构 JSONL 路径。
- 共享入站渠道轮次现在携带 `{agentId, sessionKey}`，而不是
  旧版 `storePath`。LINE、WhatsApp、Slack、Discord、Telegram、Matrix、Signal、
  iMessage、BlueBubbles、Feishu、Google Chat、IRC、Nextcloud Talk、Zalo、
  Zalo Personal、QA Channel、Microsoft Teams、Mattermost、Synology Chat、Tlon、
  Twitch 和 QQ Bot 的记录路径现在读取更新时间元数据，并通过
  SQLite 身份记录入站会话行。
- 活动会话行中不再持久化转录定位器。
  `resolveSessionTranscriptTarget` 返回 `agentId`、`sessionId` 和可选的
  主题元数据；只有 Doctor 代码会导入旧版转录文件
  名称。
- 运行时转录头从 SQLite 版本 `1` 开始。旧 JSONL V1/V2/V3
  结构升级仅存在于 Doctor 导入中，并会在存储行之前将
  导入的头规范化为当前 SQLite 转录版本。
- 数据库优先守卫现在禁止 `SessionManager.listAll` 和
  `SessionManager.forkFromSession`；会话列表以及派生/恢复工作流
  必须始终使用基于行/作用域的 SQLite API。
- 该守卫还禁止在 Doctor/导入代码之外使用旧版转录 JSONL 解析/活动分支修复辅助函数
  名称，因此运行时无法产生第二条旧版
  转录迁移路径。
- 嵌入式 PI 运行拒绝传入的转录句柄。它们在工作进程启动前
  使用 SQLite `{agentId, sessionId}` 身份，并在
  尝试操作转录状态前再次使用该身份。过时的 `/tmp/*.jsonl` 输入无法选择
  运行时写入目标。
- 缓存跟踪、Anthropic 载荷、原始流和诊断时间线记录
  现在写入带类型的 SQLite `diagnostic_events` 行。Gateway 网关稳定性包
  现在写入带类型的 SQLite `diagnostic_stability_bundles` 行。旧的
  `diagnostics.cacheTrace.filePath`、`OPENCLAW_CACHE_TRACE_FILE`、
  `OPENCLAW_ANTHROPIC_PAYLOAD_LOG_FILE` 和
  `OPENCLAW_DIAGNOSTICS_TIMELINE_PATH` JSONL 覆盖路径已移除，并且
  常规稳定性捕获不再写入 `logs/stability/*.json` 文件。
- 定时任务持久化现在协调 SQLite `cron_jobs` 行，而不是
  每次保存时删除并重新插入整个任务表。插件目标
  回写直接更新匹配的定时任务行，并在
  同一个状态数据库事务中维护运行时定时任务状态。
- 定时任务运行时调用方现在使用稳定的 SQLite 定时任务存储键。旧版
  `cron.store` 路径仅作为 Doctor 导入输入；生产环境 Gateway 网关、任务
  维护、状态、运行历史记录和 Telegram 目标回写路径使用
  `resolveCronStoreKey`，且不再对键进行路径规范化。定时任务状态现在
  报告 `storeKey`，而不是旧的文件形态 `storePath` 字段。
- 定时任务运行时加载和调度不再规范化旧版持久化任务
  结构，例如 `jobId`、`schedule.cron`、数值型 `atMs`、字符串布尔值或
  缺失的 `sessionTarget`。在将行插入 SQLite 前，这些修复由 Doctor 旧版导入
  负责。
- ACP 派生不再解析或持久化转录 JSONL 文件路径。派生
  和线程绑定设置直接持久化 SQLite 会话行，并保留
  会话 ID 作为转录身份。
- ACP 会话元数据 API 现在按 `agentId` 读取/列出/更新或插入 SQLite 行，
  并且不再将 `storePath` 作为 ACP 会话条目契约的一部分公开。
- 会话用量统计和 Gateway 网关用量聚合现在仅按 `{agentId, sessionId}` 解析转录。
  成本/用量缓存和已发现会话
  摘要不再合成或返回转录定位器字符串。
- Gateway 网关聊天追加、中止时的部分内容持久化、`/sessions.send` 和
  Web 聊天媒体转录写入现在通过 SQLite 转录
  作用域直接追加。Gateway 网关转录注入辅助函数不再接受
  `transcriptLocator` 参数。
- SQLite 转录发现现在仅列出转录作用域和统计信息：
  `{agentId, sessionId, updatedAt, eventCount}`。废弃的
  `listSqliteSessionTranscriptLocators` 兼容性辅助函数和逐行
  `locator` 字段已被移除。
- 转录修复运行时现在仅公开
  `repairTranscriptSessionStateIfNeeded({agentId, sessionId})`。旧的
  基于定位器的修复辅助函数已删除；Doctor/调试代码读取显式
  源文件路径，且绝不会迁移定位器字符串。
- ACP 重放账本运行时现在将每个会话的重放行存储在共享
  SQLite 状态数据库中，而不是 `acp/event-ledger.json`；Doctor 会导入并
  移除旧版文件。
- Gateway 网关转录读取器辅助函数现在位于
  `src/gateway/session-transcript-readers.ts` 中，而不是旧的
  `session-utils.fs` 模块名称中。后备重试历史记录检查现在根据
  SQLite 转录内容命名，而不是沿用旧的文件辅助函数表面。
- Gateway 网关注入聊天和压缩辅助函数现在通过内部辅助 API 传递 SQLite 转录作用域，
  而不再将值命名为转录路径或
  源文件。
- 引导继续检测现在通过
  `hasCompletedBootstrapTranscriptTurn` 检查 SQLite 转录行；它不再公开文件形态的
  辅助函数名称。
- 嵌入式运行器测试现在使用 SQLite 转录身份，并且打开新的
  转录管理器始终需要显式的 `sessionId`。
- 记忆索引辅助函数现在端到端使用 SQLite 转录术语：
  主机导出 `listSessionTranscriptScopesForAgent` 和
  `sessionTranscriptKeyForScope`，定向同步将 `sessionTranscripts` 加入队列，
  公共会话搜索命中项公开不透明的 `transcript:<agent>:<session>` 路径，
  内部数据库源键则是在
  `source_kind='sessions'` 下的 `session:<session>`，而不是虚假的文件路径。
- 通用插件 SDK 持久去重辅助函数不再公开文件形态的
  选项。调用方提供 SQLite 作用域键，持久去重行位于
  共享插件状态中。
- Microsoft Teams SSO 令牌已从加锁的 JSON 文件迁移到 SQLite 插件
  状态。Doctor 导入 `msteams-sso-tokens.json`，根据载荷重建规范的 SSO 令牌
  键，并移除源文件。委派的 OAuth 令牌仍保留在
  现有的私有凭据文件边界中。
- Matrix 同步缓存状态已从 `bot-storage.json` 迁移到 SQLite 插件
  状态。Doctor 导入旧版原始或封装的同步载荷，并移除
  源文件。活动的 Matrix 和 QA Lab Matrix 适配器客户端传递 SQLite 同步存储根
  目录，而不是虚假的 `sync-store.json` 或 `bot-storage.json` 路径。
- Matrix 旧版加密迁移状态已从
  `legacy-crypto-migration.json` 迁移到 SQLite 插件状态。Doctor 导入
  旧状态文件；Matrix SDK IndexedDB 快照已从
  `crypto-idb-snapshot.json` 迁移到 SQLite 插件 Blob。Matrix 恢复密钥和
  凭据是 SQLite 插件状态行；其旧 JSON 文件仅作为 Doctor
  迁移输入。
- Memory Wiki 活动日志现在使用 SQLite 插件状态，而不是
  `.openclaw-wiki/log.jsonl`。Memory Wiki 迁移提供商会导入旧
  JSONL 日志；wiki Markdown 和用户保管库内容仍作为
  工作区内容存储在文件中。
- Memory Wiki 不再创建 `.openclaw-wiki/state.json` 或未使用的
  `.openclaw-wiki/locks` 目录。如果旧保管库中仍存在这些已淘汰的
  插件元数据文件，迁移提供商会将其移除。
- 系统智能体审计条目现在使用核心 SQLite 插件状态，而不是
  `audit/crestodian.jsonl`。Doctor 导入旧版 JSONL 审计日志，并在
  成功导入后将其移除。
- 配置写入/观察审计条目现在使用核心 SQLite 插件状态，而不是
  `logs/config-audit.jsonl`。Doctor 导入旧版 JSONL 审计日志，并在
  成功导入后将其移除。
- macOS 配套应用在编辑 `openclaw.json` 时不再写入应用本地的
  `logs/config-audit.jsonl` 或
  `logs/config-health.json` 边车文件。配置
  文件仍使用文件存储，恢复快照仍放在配置文件旁边，
  持久配置审计/健康状态则属于 Gateway 网关 SQLite 存储。
- 系统智能体救援的待审批项现在使用核心 SQLite 插件状态，而不是
  `crestodian/rescue-pending/*.json` 或 `openclaw/rescue-pending/*.json`。
  这些短期安全能力绝不会被导入；Doctor 会丢弃
  这两个已淘汰的目录，防止升级重新激活过时的写入。
- Phone Control 临时布防状态现在使用 SQLite 插件状态，而不是
  `plugins/phone-control/armed.json`。Doctor 将旧版布防状态
  文件导入 `phone-control/arm-state` 命名空间并移除该文件。
- Doctor 不再原地修复 JSONL 转录或创建备份 JSONL
  文件。它会将活动分支导入 SQLite，并移除旧版源文件。
- 会话记忆钩子的转录查找使用 `{agentId, sessionId}` 仅作用域
  SQLite 读取。其辅助函数不再接受或派生转录定位器、
  旧版文件读取或文件重写选项。
- Codex 应用服务器会话绑定现在按
  OpenClaw 会话键或显式 `{agentId, sessionId}` 作用域为 SQLite 插件状态设置键。它们不得
  保留转录路径后备绑定。
- Codex 应用服务器镜像历史记录读取仅使用 SQLite 转录作用域；
  它们不得从转录文件路径恢复身份。
- 角色排序和压缩重置路径不再取消链接旧转录
  文件；重置仅轮换 SQLite 会话行和转录身份。
- Gateway 网关重置和检查点响应返回干净的会话行及会话
  ID。它们不再为客户端合成 SQLite 转录定位器。
- Memory-core Dreaming 不再通过探测缺失的
  JSONL 文件来清理会话行。子智能体清理通过会话运行时 API 执行，而不是
  检查文件系统是否存在。其转录摄取测试直接植入 SQLite 行，
  而不是创建 `agents/<id>/sessions` 固件或定位器
  占位符。
- 记忆转录索引可以将 `transcript:<agentId>:<sessionId>` 公开为
  供引用/读取辅助函数使用的虚拟搜索命中路径。持久索引源是
  关系型的（`source_kind='sessions'`、`source_key='session:<sessionId>'`、
  `session_id=<sessionId>`)，因此该值不是运行时转录记录定位符，
  也不是文件系统路径，绝不能将其传回会话运行时 API。
- Gateway 网关 Doctor 的记忆状态从 SQLite 插件状态行读取短期回忆和阶段信号计数，
  而不是从 `memory/.dreams/*.json` 读取；CLI 和
  Doctor 输出现在将该存储标记为 SQLite 存储，而不是路径。
- Memory-core 运行时、CLI 状态、Gateway 网关 Doctor 方法和插件 SDK
  门面不再审计或归档旧版 `.dreams/session-corpus` 文件。
  这些文件仅作为迁移输入；Doctor 将其导入 SQLite，并在验证后
  删除源文件。活跃的会话摄取证据行
  现在使用虚拟 SQLite 路径 `memory/session-ingestion/<day>.txt`；运行时
  绝不会向 `.dreams/session-corpus` 写入数据或从中派生状态。
- Memory-core 公共工件将 SQLite 主机事件公开为虚拟 JSON
  工件 `memory/events/memory-host-events.json`；它们不再复用
  旧版 `.dreams/events.jsonl` 源路径。
- 沙箱容器/浏览器注册表现在使用共享的
  `sandbox_registry_entries` SQLite 表，其中包含类型化的会话、镜像、时间戳、
  后端/配置和浏览器端口列。Doctor 会导入旧版单体及
  分片 JSON 注册表文件，并删除成功导入的源文件。运行时读取
  以类型化行列作为事实来源；`entry_json` 仅作为重放/调试
  副本。
- 跟进承诺现在使用类型化的共享 `commitments` 表，而不是
  整个存储的 JSON blob。运行时使用带索引的范围、投递窗口、滚动
  上限、状态和尝试查询，以及同步 SQLite 事务；
  `record_json` 仅作为重放/调试副本。显式 Doctor 修复会验证
  完整的旧版 `commitments.json`，保留较新的 SQLite 行，验证
  结果，然后才删除未发生变化的源文件。运行时绝不读取或
  写入已弃用的文件。
- Web Push 订阅和生成的 VAPID 身份现在使用类型化的共享
  `web_push_subscriptions` 和 `web_push_vapid_keys` 行。运行时注册、
  过期清理和首次使用时的密钥生成使用行级 SQLite
  事务。显式 Doctor 修复会验证两个已弃用的 JSON 存储，
  在写入 SQLite 前声明对它们的处理权，以原子方式导入，拒绝
  冲突的 VAPID 身份，验证结果，然后才移除这些
  声明。Doctor 在整个导入期间持有状态目录维护锁，
  以防旧版 Gateway 网关重新创建已弃用的文件。在 Doctor 解决
  待处理的旧版源文件或中断的声明之前，注册、投递、删除和密钥解析
  都会以失败关闭方式处理。
- 定时任务定义、调度状态和运行历史不再有运行时
  JSON 写入器或读取器。运行时使用 `cron_jobs` 行，其中包含类型化的调度、
  载荷、投递、失败警报、会话、状态和运行时状态列，以及
  由 cron 所有的 `task_runs` 详细信息，用于诊断、投递、会话/运行、模型
  和 token 总数。`job_json` 仅作为重放/调试副本；`state_json` 保留
  尚无热查询字段的嵌套运行时诊断信息，而运行时
  从类型化列重新填充热状态字段。Doctor 会导入
  旧版 `jobs.json`、`jobs-state.json` 和 `runs/*.jsonl` 文件，并删除
  已导入的源文件。插件目标回写会更新匹配的 `cron_jobs`
  行，而不是加载并替换整个 cron 存储。
- Gateway 网关启动时会忽略运行时投影中的旧版 `notify: true` 标记。
  Doctor 仅在将这些标记转换为显式 SQLite 投递时读取已弃用的原始
  `cron.webhook`，随后删除该配置键。
- 出站和会话投递队列现在将队列状态、条目类型、
  会话键、渠道、目标、账户 ID、重试次数、最后一次尝试/错误、
  恢复状态和平台发送标记存储为共享
  `delivery_queue_entries` 表中的类型化列。运行时恢复会从
  类型化列读取这些热字段，重试/恢复变更会直接更新这些列，
  无需重写重放 JSON。完整 JSON 载荷仅保留为
  消息正文和其他冷重放数据的重放/调试 blob。
- 托管出站图像记录现在使用类型化的共享
  `managed_outgoing_image_records` 行。运行时仅从类型化列读取；
  JSON 列是重放/调试副本。原始图像字节仍作为具名
  附件工件保留在托管媒体目录中。
- Discord 模型选择器偏好设置、命令部署哈希和线程绑定
  现在使用共享 SQLite 插件状态。其旧版 JSON 导入计划位于
  Discord 插件设置/Doctor 迁移界面中，而不是核心迁移代码中。
- 插件旧版导入检测器使用以 Doctor 命名的模块，例如
  `doctor-legacy-state.ts` 或 `doctor-state-imports.ts`；常规渠道运行时
  模块不得导入旧版 JSON 检测器。
- BlueBubbles 补录游标和入站去重标记现在使用共享 SQLite
  插件状态。其旧版 JSON 导入计划位于 BlueBubbles 插件
  设置/Doctor 迁移界面中，而不是核心迁移代码中。
- Telegram 更新偏移量、贴纸缓存行、已发送消息缓存行、
  话题名称缓存行和线程绑定现在使用共享 SQLite 插件
  状态。其旧版 JSON 导入计划位于 Telegram 插件
  设置/Doctor 迁移界面中，而不是核心迁移代码中。
- iMessage 补录游标、回复短 ID 映射和已发送回显去重行
  现在使用共享 SQLite 插件状态。旧的 `imessage/catchup/*.json`、
  `imessage/reply-cache.jsonl` 和 `imessage/sent-echoes.jsonl` 文件仅作为
  Doctor 输入。
- Feishu 消息去重行现在使用核心可声明去重机制
  （共享 SQLite 插件状态中的 `feishu.dedup.*` 命名空间），而不是
  `feishu/dedup/*.json` 文件或已弃用的手工实现 `dedup.*` 存储；
  不进行旧版导入，因为重放保护缓存会在升级后重建。
- Microsoft Teams 对话、投票、待处理上传缓冲区和反馈
  学习数据现在使用共享 SQLite 插件状态/blob 表。待处理上传
  路径使用 `plugin_blob_entries`，因此媒体缓冲区存储为 SQLite BLOB，
  而不是 base64 JSON。运行时辅助程序名称现在使用 SQLite/状态命名，
  而不是 `*-fs` 文件存储命名，并且旧的 `storePath` 垫片已从
  这些存储中移除。其旧版 JSON 导入计划位于 Microsoft Teams
  插件设置/Doctor 迁移界面中。
- Zalo 托管出站媒体现在使用共享 SQLite `plugin_blob_entries`，
  而不是 `openclaw-zalo-outbound-media` JSON/bin 临时旁车文件。
- Diffs 查看器 HTML 和元数据现在使用共享 SQLite `plugin_blob_entries`，
  而不是 `meta.json`/`viewer.html` 临时文件。查看器 HTML 以
  gzip blob 形式存储，并且仅持久化 URL token 哈希。渲染后的 PNG/PDF 输出
  仍作为临时物化文件，因为渠道投递仍需要文件路径；
  其过期元数据由 SQLite 管理，不使用 JSON 旁车文件。
- Canvas 托管文档现在使用共享 SQLite `plugin_blob_entries`，
  而不是默认的 `state/canvas/documents` 目录。Canvas 主机直接提供这些
  blob；仅针对显式 `host.root` 操作员内容，或下游媒体读取器
  需要路径时的临时物化，才会创建本地文件。
- File Transfer 审计决策现在使用共享 SQLite `plugin_state_entries`，
  而不是无限增长的 `audit/file-transfer.jsonl` 运行时日志。Doctor
  会将旧版 JSONL 审计文件导入插件状态，并在无错误导入后删除源文件。
- ACPX 进程租约和 Gateway 网关实例身份现在使用共享 SQLite 插件
  状态。Doctor 会将旧版 `gateway-instance-id` 文件导入插件状态
  并删除源文件。
- ACPX 生成的包装脚本和隔离的 Codex 主目录是 OpenClaw
  临时根目录下的临时物化内容，而不是持久 OpenClaw 状态。
  持久 ACPX 运行时记录是 SQLite 租约和 Gateway 网关实例行；
  旧的 ACPX `stateDir` 配置界面已被移除，因为运行时状态
  不再写入该位置。
- Gateway 网关媒体附件现在使用共享 `media_blobs` SQLite 表作为
  规范字节存储。返回给渠道和沙箱
  兼容性界面的本地路径是数据库行的临时物化内容，而不是
  持久媒体存储。运行时媒体允许列表不再包含旧版
  `$OPENCLAW_STATE_DIR/media` 或配置目录下的 `media` 根目录；这些目录
  仅作为 Doctor 导入源。
- Shell 补全不再写入 `$OPENCLAW_STATE_DIR/completions/*` 缓存
  文件。安装、Doctor、更新和发布冒烟测试路径使用生成的
  补全输出或 profile 加载，而不是持久补全缓存
  文件。
- Gateway 网关技能上传暂存现在使用共享 `skill_uploads` 和
  `skill_upload_chunks` 行。上传期间，各分块仍分别以事务方式处理；
  随后提交操作会组装一个已验证的归档 BLOB，并删除分块
  行。安装程序仅在安装运行期间接收临时物化的归档路径。
  Doctor 会丢弃已弃用的一小时文件系统
  暂存树，而不是导入临时上传内容。
- 子智能体内联附件不再物化到工作区
  `.openclaw/attachments/*` 下。生成路径会准备 SQLite VFS 种子条目，
  内联运行会将这些条目植入每个 Agent 的运行时暂存命名空间，
  而基于磁盘的工具会在该 SQLite 暂存空间上叠加附件路径。
  旧的子智能体运行附件目录注册表列和清理钩子已被移除。
- CLI 图像加载不再维护稳定的 `openclaw-cli-images` 缓存
  文件。外部 CLI 后端仍会接收文件路径，但这些路径是
  每次运行的临时物化内容，并会进行清理。
- 缓存跟踪诊断、Anthropic 载荷诊断、原始模型流
  诊断、诊断时间线事件和 Gateway 网关稳定性包现在
  写入 SQLite 行，而不是 `logs/*.jsonl` 或
  `logs/stability/*.json` 文件。
  运行时路径覆盖标志和环境变量已被移除；导出/调试
  命令可以显式地从数据库行物化文件。
- macOS 配套应用不再有滚动式 `diagnostics.jsonl` 写入器。应用
  日志写入统一日志系统，持久 Gateway 网关诊断仍以 SQLite 为后端。
- macOS 端口守护记录列表现在使用类型化的共享 SQLite
  `macos_port_guardian_records` 行，而不是 Application Support JSON 文件
  或不透明的单例 blob。所有 macOS 应用 profile 都使用同一个主机全局原生
  数据库，因为它们需要协调机器本地端口。只要旧版仍写入 JSON 的应用副本
  正在运行，每项台账操作都会阻塞。迁移仅为创建快照并稍后重新验证
  源数据而加入旧台账的稳定文件锁协议。它在不持有该锁的情况下，
  根据实时命令和进程启动事实解析每个旧版行，然后重新读取权威 SQLite 行，
  应用计划，验证每条记录并删除源文件。删除重试会针对
  缺失行重新制定计划，因此已弃用的陈旧记录无法复活。该锁保持
  短生命周期，因此不会在 SSH 生成进程后导致旧版写入器永久受阻。切换
  刻意设计为单向：稳态运行时绝不读取、投影或写入 JSON，
  回滚到仅支持 JSON 的构建版本不会保留较新的 SQLite 记录。
- Gateway 网关单例锁现在使用 `gateway_locks` 范围下类型化的共享 SQLite
  `state_leases` 行，而不是临时目录锁文件。Fly 和 OAuth
  故障排除文档现在指向 SQLite 租约/身份验证刷新锁，
  而不是过时的文件锁清理。
- Gateway 网关重启哨兵状态现在使用带类型的共享 SQLite
  `gateway_restart_sentinel` 行，而不再使用 `restart-sentinel.json`；运行时
  从带类型的列中读取哨兵种类、状态、路由、消息、延续信息和统计信息。
  这些列是权威数据；`payload_json` 仅用作重放/调试影子副本。运行时的读取、写入和清除路径仅使用 SQLite。
  一个有界状态迁移模块会在启动期间和 Doctor 中运行，以在正常的重启恢复前导入
  经过验证的旧版更新后哨兵，验证带类型的行，
  并删除源文件。任何稳态运行时模块都不会
  读取、写入或清理旧版文件。
- Gateway 网关重启意图和监督器交接状态现在使用带类型的共享
  SQLite `gateway_restart_intent` 和 `gateway_restart_handoff` 行，而不再使用
  `gateway-restart-intent.json` 和
  `gateway-supervisor-restart-handoff.json` 边车文件。
- Gateway 网关单例协调现在使用 `gateway_locks` 下带类型的
  `state_leases` 行，而不再写入 `gateway.<hash>.lock` 文件。租约行
  保存锁所有者、到期时间、心跳和调试载荷；SQLite 负责
  原子获取/释放边界。已移除弃用的文件锁目录选项；
  测试直接使用 SQLite 行标识。
- 已删除扫描 `cron/runs/*.jsonl`
  文件的旧版未引用定时任务用量报告辅助函数。定时任务运行历史报告读取由定时任务拥有的 `task_runs` 行。
- 主会话重启恢复现在通过 SQLite `agent_databases`
  注册表发现候选智能体，而不再扫描 `agents/*/sessions`
  目录。
- Gemini 会话损坏恢复现在仅删除 SQLite 会话行；
  不再需要旧版 `storePath` 门控，也不再尝试取消链接派生的
  脚本记录 JSONL 路径。
- 路径覆盖处理现在将字面量 `undefined`/`null` 环境
  值视为未设置，防止在测试或 shell 交接期间意外创建位于仓库根目录的 `undefined/state/*.sqlite`
  数据库。
- 配置健康指纹现在使用带类型的共享 SQLite `config_health_entries`
  行，而不再使用 `logs/config-health.json`，使常规配置文件成为
  唯一的非凭据配置文档。macOS 配套应用仅保留
  进程本地健康状态，不会重新创建旧版 JSON 边车文件。
- 身份验证配置文件运行时不再导入或写入凭据 JSON 文件。
  规范凭据存储为 SQLite；`auth-profiles.json`、每智能体
  `auth.json` 和共享 `credentials/oauth.json` 是 Doctor 迁移输入，
  导入后会被删除。
- 身份验证配置文件保存/状态测试现在直接断言带类型的 SQLite 身份验证表，
  并且仅将旧版身份验证配置文件名用作 Doctor 迁移输入。
- `openclaw secrets apply` 仅清理配置文件、环境文件和 SQLite
  身份验证配置文件存储。它不再包含编辑
  已弃用的每智能体 `auth.json` 的兼容逻辑；Doctor 负责导入并删除该文件。
- Hermes 密钥迁移计划将导入的 API 密钥配置文件直接应用到
  SQLite 身份验证配置文件存储中。它不再将
  `auth-profiles.json` 作为中间目标写入或验证。
- 面向用户的身份验证文档现在描述
  `state/openclaw.sqlite#table/auth_profile_stores/<agentDir>`，而不再
  告知用户检查或复制 `auth-profiles.json`；旧版 OAuth/身份验证 JSON
  名称仅作为 Doctor 导入输入保留在文档中。
- MCP OAuth 会话现在使用共享
  `state/openclaw.sqlite` 中带版本的 `mcp_oauth_stores` 行。由 SDK 拥有的令牌、客户端注册和发现
  对象仍保留为一个经过验证的 JSON 载荷，以便保留依赖项扩展字段，
  而每次读取/修改/写入都在一个简短的 Kysely
  事务中提交。一个共享 SQLite 租约会串行化刷新、登录和登出；
  嵌入式 MCP 传输不再允许 MCP SDK 在该
  租约之外刷新。Doctor 独占导入并删除已弃用的 `mcp-oauth/*.json`
  存储并附带来源回执，运行时没有文件回退。
- 核心状态路径辅助函数不再公开已弃用的 `credentials/oauth.json`
  文件。旧版文件名仅存在于 Doctor 身份验证导入路径中。
- 安装、安全、新手引导、模型身份验证和 SecretRef 文档现在描述
  SQLite 身份验证配置文件行以及整体状态备份/迁移，而不再描述
  每智能体身份验证配置文件 JSON 文件。
- PI 模型发现现在将规范凭据传入内存中的
  `pi-coding-agent` 身份验证存储。它在发现期间不再创建、清理或写入
  每智能体 `auth.json`。
- Voice Wake 触发和路由设置现在使用带类型的共享 SQLite 表，
  而不再使用 `settings/voicewake.json`、`settings/voicewake-routing.json` 或
  不透明通用行；Doctor 会导入旧版 JSON 文件，并在迁移成功后将其删除。
- 更新检查状态现在使用带类型的共享 `update_check_state` 行，而不再使用
  `update-check.json` 或不透明通用 blob；Doctor 会导入
  旧版 JSON 文件，并在迁移成功后将其删除。
- 配置健康状态现在使用带类型的共享 `config_health_entries` 行，
  而不再使用 `logs/config-health.json` 或不透明通用 blob；Doctor
  会导入旧版 JSON 文件，并在迁移成功后将其删除。
- 插件会话绑定审批现在使用带类型的
  `plugin_binding_approvals` 行，而不再使用不透明共享 SQLite 状态或
  `plugin-binding-approvals.json`；旧版文件是 Doctor 迁移输入。
- 通用当前会话绑定现在存储带类型的
  `current_conversation_bindings` 行，而不再重写
  `bindings/current-conversations.json`；Doctor 会导入旧版 JSON 文件，并
  在迁移成功后将其删除。
- Memory Wiki 导入来源同步账本现在按每个保管库/来源键存储一个 SQLite 插件状态行，
  而不再重写 `.openclaw-wiki/source-sync.json`；
  迁移提供商会导入并删除旧版 JSON 账本。
- Memory Wiki ChatGPT 导入运行记录现在按每个保管库/运行 ID 存储一个 SQLite 插件状态行，
  而不再写入 `.openclaw-wiki/import-runs/*.json`。
  在导入运行快照的归档迁移到 blob 存储之前，
  回滚快照仍保留为显式保管库文件。
- Memory Wiki 编译摘要现在存储压缩的 SQLite 插件 blob 行，
  而不再写入 `.openclaw-wiki/cache/agent-digest.json` 和
  `.openclaw-wiki/cache/claims.jsonl`。缓存可重建，因此 Doctor
  会删除旧缓存文件而不导入它们。
- ClawHub 技能安装跟踪现在按每个工作区/技能存储一个 SQLite 插件状态行，
  而不再在运行时写入或读取 `.clawhub/lock.json` 和
  `.clawhub/origin.json` 边车文件。运行时代码使用跟踪安装
  状态对象，而不是文件形态的锁文件/来源抽象。Doctor
  会从已配置的 Agent 工作区导入旧版边车文件，并在
  完整导入后将其删除。
- 已安装插件索引现在读取和写入带类型的共享 SQLite
  `installed_plugin_index` 单例行，而不再使用 `plugins/installs.json`；
  旧版 JSON 文件仅作为 Doctor 迁移输入，并在导入后删除。
- 旧版 `plugins/installs.json` 路径辅助函数现在位于 Doctor 旧版
  代码中。运行时插件索引模块仅公开由 SQLite 支持的持久化
  选项，不公开 JSON 文件路径。
- Gateway 网关重启哨兵、重启意图和监督器交接状态现在使用
  带类型的共享 SQLite 行（`gateway_restart_sentinel`、
  `gateway_restart_intent` 和 `gateway_restart_handoff`），而不再使用通用
  不透明 blob。运行时重启代码不再有文件形态的哨兵/意图/交接
  契约。
- Matrix 同步缓存、存储元数据、线程绑定、入站去重标记、
  启动验证冷却状态、SDK IndexedDB 加密快照、
  凭据和恢复密钥现在使用共享 SQLite 插件状态/blob
  表。运行时路径结构不再公开 `storage-meta.json` 元数据
  路径；该文件名仅作为旧版迁移输入。其旧版 JSON 导入
  计划位于 Matrix 插件设置/Doctor 迁移界面中。入站
  去重标记使用核心可声明去重机制（共享状态数据库中的 `matrix.inbound-dedupe.*`
  命名空间）；Matrix Doctor 状态迁移会一次性导入
  已弃用的每根目录 `inbound-dedupe` 行和 `inbound-dedupe.json`，
  此后运行时仅从可声明去重存储读取。
- Matrix 启动不再扫描、报告或补全旧版 Matrix 文件
  状态。Matrix 文件检测、旧版加密快照创建、房间密钥
  恢复迁移状态、导入和源文件删除均由 Doctor 负责。
- 已移除 Matrix 运行时迁移桶文件。旧版状态/加密检测
  和变更辅助函数由 Matrix Doctor 直接导入，不再
  属于运行时 API 界面。
- Matrix 迁移快照复用标记现在位于 SQLite 插件状态中，
  而不再位于 `matrix/migration-snapshot.json`；Doctor 仍可复用同一个
  已验证的迁移前归档，而无需写入边车状态文件。
- Nostr 总线游标和配置文件发布状态现在使用共享 SQLite 插件
  状态。其旧版 JSON 导入计划位于 Nostr 插件设置/Doctor
  迁移界面中。
- 主动记忆会话开关现在使用共享 SQLite 插件状态，而不再使用
  `session-toggles.json`；重新开启记忆时会删除该行，而不是
  重写 JSON 对象。
- Skill Workshop 提案和审查计数器现在使用共享 SQLite 插件
  状态，而不再使用每工作区 `skill-workshop/<workspace>.json` 存储。每个
  提案都是 `skill-workshop/proposals` 下的独立行，审查
  计数器则是 `skill-workshop/reviews` 下的独立行。
- Skill Workshop 审查者子智能体运行现在使用运行时会话脚本记录
  解析器，而不再创建 `skill-workshop/<sessionId>.json` 边车会话
  路径。
- ACPX 进程租约现在使用 `acpx/process-leases` 下的共享 SQLite 插件状态，
  而不再使用整文件 `process-leases.json` 注册表。
  每个租约单独存储为一行，在无需运行时 JSON 重写路径的情况下，
  保留启动时清理过期进程的功能。
- ACPX 包装脚本和隔离的 Codex 主目录在
  OpenClaw 临时根目录中生成。它们会按需重新创建，不属于备份或
  迁移输入。
- 子智能体运行注册表持久化使用带类型的共享 `subagent_runs` 行。
  旧 `subagents/runs.json` 路径现在仅作为 Doctor 清理输入。Doctor
  在状态维护锁下声明该路径，将丢弃决策记录到
  SQLite 中，并在不导入临时运行状态的情况下将其删除。不再保留任何运行时 JSON
  读取器、写入器、缓存或回退；在此弃用边界上，有意不支持
  对仅存在于文件中的进行中运行进行跨版本恢复。
  运行时测试不再创建无效或空的 `runs.json` 固件来验证
  注册表行为；它们直接填充/读取 SQLite 行。
- 备份会在归档前暂存状态目录，复制非数据库文件，
  使用在线备份及离线 `VACUUM` 创建数据库快照，省略实时 WAL/SHM 边车文件，在归档清单中记录
  快照元数据，并将已完成的备份运行及归档清单
  记录到 SQLite 中。`openclaw backup
create` 默认验证已写入的归档；`--no-verify` 是
  显式快速路径。
- `openclaw backup restore` 会在提取前验证归档，复用
  验证器规范化后的清单，并将经过验证的清单资源恢复到其
  记录的源路径。写入操作需要 `--yes`，并支持使用 `--dry-run`
  生成恢复计划。
- 已删除旧版备份易变路径过滤器。备份不再需要针对
  旧版会话或定时任务 JSON/JSONL 文件的实时 tar 跳过列表，因为 SQLite
  快照会在创建归档前暂存。
- 普通设置和新手引导的工作区准备不再创建
  `agents/<agentId>/sessions/` 目录。它们仅创建配置/工作区；
  SQLite 会话行和转录行会按需在
  每个 Agent 的数据库中创建。
- 安全权限修复现在针对全局和每个 Agent 的 SQLite
  数据库及 WAL/SHM 辅助文件，而不是 `sessions.json` 和转录
  JSONL 文件。
- 沙箱注册表运行时名称现在直接描述 SQLite 注册表类型，
  不再将旧版 JSON 注册表术语带入活动存储。
- `openclaw reset --scope config+creds+sessions` 会移除每个 Agent 的
  `openclaw-agent.sqlite` 数据库及 WAL/SHM 辅助文件，而不只是旧版
  `sessions/` 目录。
- Gateway 网关聚合会话辅助函数现在使用面向条目的名称：
  `loadCombinedSessionEntriesForGateway` 返回 `{ databasePath, entries }`。
  旧的组合存储命名已从运行时调用方中移除。
- Docker MCP 渠道种子数据写入现在会将主会话行和转录
  事件写入每个 Agent 的 SQLite 数据库，而不是创建
  `sessions.json` 和 JSONL 转录文件。
- 内置的会话记忆钩子现在通过 `{agentId, sessionId}` 从 SQLite
  解析上一会话的上下文。它不再扫描、存储或合成
  转录路径或 `workspace/sessions` 目录。
- 内置的命令记录器钩子现在将命令审计行写入共享
  SQLite `command_log_entries` 表，而不是追加到
  `logs/commands.log`。
- 渠道配对允许列表现在仅在运行时公开由 SQLite 支持的读写辅助函数。
  已弃用的插件 SDK 路径解析器仍保留以实现迁移
  兼容性；文件读取器仅存在于 Doctor 状态迁移代码中。
- `migration_runs` 会记录旧版状态迁移的执行情况，包括状态、
  时间戳和 JSON 报告。
- `migration_sources` 会记录每个导入的旧版文件源，包括哈希值、大小、
  记录数、目标表、运行 ID、状态和源文件移除状态。
- `backup_runs` 会记录备份归档路径、状态和 JSON 清单。
- 全局架构不会保留未使用的 `agents` 注册表。Agent
  数据库发现是规范的 `agent_databases` 注册表，直到运行时
  拥有真正的 Agent 记录所有者。
- 生成的模型目录配置存储在有类型的全局 SQLite
  `agent_model_catalogs` 行中，并以 Agent 目录为键。运行时调用方使用
  `ensureOpenClawModelCatalog`；运行时代码中不存在 `models.json` 兼容性 API。
  该实现写入 SQLite，并从所存储的载荷填充嵌入式 PI 注册表，
  而不会创建 `models.json` 文件。
- 可选的 `memory.qmd.sessions` 导出会从每个 Agent 的数据库读取规范的转录行，
  并在 QMD 主目录下具体化经过净化的 Markdown，作为显式的 QMD 输入工件。
  因此，QMD 会话集合和工件身份映射仍是已配置外部工具
  桥接的一部分；它们不是第二个规范转录存储。
- QMD 自身的 `index.sqlite`、YAML 集合配置和模型下载仍是
  `~/.openclaw/agents/<agentId>/qmd` 下的外部工具工件；它们不会
  镜像到 `plugin_blob_entries`。OpenClaw 所有的 QMD 协调以数据库为先：
  共享的 `state_leases` 在全局范围串行执行嵌入操作，而每个 Agent 的
  `state_leases` 则串行执行集合/更新/嵌入写入操作。运行时不会创建
  QMD 锁辅助文件。
- 可选的 `memory-lancedb` 插件不再创建
  `~/.openclaw/memory/lancedb` 作为 OpenClaw 隐式管理的存储。它是一个
  外部 LanceDB 后端，并会保持禁用状态，直到操作员配置显式的
  `dbPath`。
- `check:database-first-legacy-stores` 会阻止将旧版存储名称与
  写入型文件系统 API 配对的新运行时源代码。它还会阻止运行时
  源代码重新引入已退役的转录桥接标记
  `transcriptLocator` 或 `sqlite-transcript://...`。迁移、Doctor、导入
  和显式的非会话导出代码仍被允许。`sessionFile`、
  `storePath` 和旧的 `SessionManager` 文件时代门面等更宽泛的旧版契约名称
  仍有当前所有者，需要单独完成迁移防护工作，
  才能将其设为必需的预检检查。该防护现在还涵盖
  运行时 `cache/*.json` 存储、通用
  `thread-bindings.json` 辅助文件、定时任务状态/运行日志 JSON、配置健康 JSON、
  重启和锁辅助文件、Voice Wake 设置、插件绑定审批、
  已安装插件索引 JSON、File Transfer 审计 JSONL、Memory Wiki 活动
  日志、旧的内置 `command-logger` 文本日志，以及 pi-mono 原始流 JSONL
  诊断开关。它还禁止旧的根级 Doctor 旧版模块名称，以确保
  兼容性代码位于 `src/commands/doctor/` 下。Android 调试处理程序
  也使用 logcat/内存输出，而不是暂存 `camera_debug.log` 或
  `debug_logs.txt` 缓存文件。

## 目标 Schema 结构

保持 Schema 明确。宿主所有的运行时状态使用类型化表。插件所有的
不透明状态使用 `plugin_state_entries` / `plugin_blob_entries`；不存在
通用的宿主 `kv` 表。

全局数据库：

```text
state_leases(scope, lease_key, owner, expires_at, heartbeat_at, payload_json, created_at, updated_at)
exec_approvals_config(config_key, raw_json, socket_path, has_socket_token, default_security, default_ask, default_ask_fallback, auto_allow_skills, agent_count, allowlist_count, updated_at_ms)
schema_meta(meta_key, role, schema_version, agent_id, app_version, created_at, updated_at)
agent_databases(agent_id, path, schema_version, last_seen_at, size_bytes)
task_runs(...)
task_delivery_state(...)
flow_runs(...)
subagent_runs(run_id, child_session_key, requester_session_key, controller_session_key, created_at, ended_at, cleanup_handled, payload_json)
current_conversation_bindings(binding_key, binding_id, target_agent_id, target_session_id, target_session_key, channel, account_id, conversation_kind, parent_conversation_id, conversation_id, target_kind, status, bound_at, expires_at, metadata_json, updated_at)
plugin_binding_approvals(plugin_root, channel, account_id, plugin_id, plugin_name, approved_at)
tui_last_sessions(scope_key, session_key, updated_at)
plugin_state_entries(plugin_id, namespace, entry_key, value_json, created_at, expires_at)
plugin_blob_entries(plugin_id, namespace, entry_key, metadata_json, blob, created_at, expires_at)
media_blobs(subdir, id, content_type, size_bytes, blob, created_at, updated_at)
skill_uploads(upload_id, kind, slug, force, size_bytes, sha256, actual_sha256, received_bytes, archive_blob, created_at, expires_at, committed, committed_at, idempotency_key_hash)
skill_upload_chunks(upload_id, byte_offset, size_bytes, chunk_blob)
web_push_subscriptions(endpoint_hash, subscription_id, endpoint, p256dh, auth, created_at_ms, updated_at_ms)
web_push_vapid_keys(key_id, public_key, private_key, subject, updated_at_ms)
apns_registrations(node_id, transport, token, relay_handle, send_grant, installation_id, relay_origin, topic, environment, distribution, token_debug_suffix, updated_at_ms)
apns_registration_tombstones(node_id, deleted_at_ms)
node_host_config(config_key, version, node_id, token, display_name, gateway_host, gateway_port, gateway_tls, gateway_tls_fingerprint, gateway_context_path, updated_at_ms)
device_identities(identity_key, device_id, public_key_pem, private_key_pem, created_at_ms, updated_at_ms)
device_auth_tokens(device_id, role, token, scopes_json, updated_at_ms)
macos_port_guardian_records(pid, port, command, mode, timestamp)
workspace_setup_state(workspace_key, workspace_path, version, bootstrap_seeded_at, setup_completed_at, updated_at)
workspace_path_aliases(alias_key, alias_path, workspace_key, workspace_path, updated_at_ms)
workspace_attestations(workspace_key, attested_at_ms, updated_at_ms)
workspace_generated_bootstrap_hashes(workspace_key, filename, sha256)
native_hook_relay_bridges(relay_id, pid, hostname, port, token, expires_at_ms, updated_at_ms)
model_capability_cache(provider_id, model_id, name, input_text, input_image, reasoning, supports_tools, context_window, max_tokens, cost_input, cost_output, cost_cache_read, cost_cache_write, updated_at_ms)
agent_model_catalogs(catalog_key, agent_dir, raw_json, updated_at)
managed_outgoing_image_records(attachment_id, session_key, agent_id, message_id, created_at, updated_at, retention_class, alt, original_media_id, original_media_subdir, original_content_type, original_width, original_height, original_size_bytes, original_filename, record_json, cleanup_pending)
gateway_restart_sentinel(sentinel_key, version, kind, status, ts, session_key, thread_id, delivery_channel, delivery_to, delivery_account_id, message, continuation_json, doctor_hint, stats_json, payload_json, updated_at_ms)
channel_pairing_requests(channel_key, account_id, request_id, code, created_at, last_seen_at, meta_json)
channel_pairing_allow_entries(channel_key, account_id, entry, sort_order, updated_at)
voicewake_triggers(config_key, position, trigger, updated_at_ms)
voicewake_routing_config(config_key, version, default_target_mode, default_target_agent_id, default_target_session_key, updated_at_ms)
voicewake_routing_routes(config_key, position, trigger, target_mode, target_agent_id, target_session_key, updated_at_ms)
update_check_state(state_key, last_checked_at, last_notified_version, last_notified_tag, last_available_version, last_available_tag, auto_install_id, auto_first_seen_version, auto_first_seen_tag, auto_first_seen_at, auto_last_attempt_version, auto_last_attempt_at, auto_last_success_version, auto_last_success_at, updated_at_ms)
config_health_entries(config_path, last_known_good_json, last_promoted_good_json, last_observed_suspicious_signature, updated_at_ms)
sandbox_registry_entries(registry_kind, container_name, session_key, backend_id, runtime_label, image, created_at_ms, last_used_at_ms, config_label_kind, config_hash, cdp_port, no_vnc_port, entry_json, updated_at)
cron_jobs(store_key, job_id, name, description, enabled, delete_after_run, created_at_ms, agent_id, session_key, schedule_kind, schedule_expr, schedule_tz, every_ms, anchor_ms, at, stagger_ms, session_target, wake_mode, payload_kind, payload_message, payload_model, payload_fallbacks_json, payload_thinking, payload_timeout_seconds, payload_allow_unsafe_external_content, payload_external_content_source_json, payload_light_context, payload_tools_allow_json, delivery_mode, delivery_channel, delivery_to, delivery_thread_id, delivery_account_id, delivery_best_effort, failure_delivery_mode, failure_delivery_channel, failure_delivery_to, failure_delivery_account_id, failure_alert_disabled, failure_alert_after, failure_alert_channel, failure_alert_to, failure_alert_cooldown_ms, failure_alert_include_skipped, failure_alert_mode, failure_alert_account_id, next_run_at_ms, running_at_ms, last_run_at_ms, last_run_status, last_error, last_duration_ms, consecutive_errors, consecutive_skipped, schedule_error_count, last_delivery_status, last_delivery_error, last_delivered, last_failure_alert_at_ms, job_json, state_json, runtime_updated_at_ms, schedule_identity, sort_order, updated_at)
delivery_queue_entries(queue_name, id, status, entry_kind, session_key, channel, target, account_id, retry_count, last_attempt_at, last_error, recovery_state, platform_send_started_at, entry_json, enqueued_at, updated_at, failed_at)
commitments(id, agent_id, session_key, channel, account_id, recipient_id, thread_id, sender_id, kind, sensitivity, source, status, reason, suggested_text, dedupe_key, confidence, due_earliest_ms, due_latest_ms, due_timezone, source_message_id, source_run_id, created_at_ms, updated_at_ms, attempts, last_attempt_at_ms, sent_at_ms, dismissed_at_ms, snoozed_until_ms, expired_at_ms, record_json)
migration_runs(id, started_at, finished_at, status, report_json)
migration_sources(source_key, migration_kind, source_path, target_table, source_sha256, source_size_bytes, source_record_count, last_run_id, status, imported_at, removed_source, report_json)
backup_runs(id, created_at, archive_path, status, manifest_json)
```

Agent 数据库：

```text
schema_meta(meta_key, role, schema_version, agent_id, app_version, created_at, updated_at)
sessions(session_id, session_key, session_scope, created_at, updated_at, started_at, ended_at, status, chat_type, channel, account_id, primary_conversation_id, model_provider, model, agent_harness_id, parent_session_key, spawned_by, display_name)
conversations(conversation_id, channel, account_id, kind, peer_id, parent_conversation_id, thread_id, native_channel_id, native_direct_user_id, label, metadata_json, created_at, updated_at)
session_conversations(session_id, conversation_id, role, first_seen_at, last_seen_at)
session_routes(session_key, session_id, updated_at)
session_entries(session_id, session_key, entry_json, updated_at)
transcript_events(session_id, seq, event_json, created_at)
transcript_event_identities(session_id, event_id, seq, event_type, has_parent, parent_id, message_idempotency_key, created_at)
transcript_snapshots(session_id, snapshot_id, reason, event_count, created_at, metadata_json)
vfs_entries(namespace, path, kind, content_blob, metadata_json, updated_at)
tool_artifacts(run_id, artifact_id, kind, metadata_json, blob, created_at)
run_artifacts(run_id, path, kind, metadata_json, blob, created_at)
trajectory_runtime_events(session_id, run_id, seq, event_json, created_at)
memory_index_meta(key, value)
memory_index_sources(id, path, source, hash, mtime, size)
memory_index_chunks(id, path, source, start_line, end_line, hash, model, text, embedding, updated_at)
memory_embedding_cache(provider, model, provider_key, hash, embedding, dims, updated_at)
memory_index_state(id, revision)
cache_entries(scope, key, value_json, blob, expires_at, updated_at)
```

`memory_index_sources.id` 是稳定的整数主键；`(path, source)` 保持唯一。

未来可以添加 FTS 表以支持搜索，而无需更改规范事件表：

```text
transcript_events_fts(session_id, seq, text)
vfs_entries_fts(namespace, path, text)
```

大型值应使用 `blob` 列，而不是 JSON 字符串编码。对于必须能使用普通
SQLite 工具检查的小型结构化数据，请保留
`value_json`。

`agent_databases` 是此分支的规范注册表。在真正的 Agent 记录所有者出现之前，不要添加
`agents` 表；Agent 配置仍保留在
`openclaw.json` 中。

## Doctor 迁移结构

Doctor 应调用一个明确、可报告且可安全重复运行的迁移步骤：

```bash
openclaw doctor --fix
```

`openclaw doctor --fix` 在常规配置预检之后调用状态迁移实现，并在导入前创建经过验证的备份。运行时
启动和 `openclaw migrate` 不得导入旧版 OpenClaw 状态文件。

迁移属性：

- 一次迁移过程会发现所有旧版文件源，并在进行任何修改之前生成计划。
- Doctor 在导入旧版文件之前创建经过验证的迁移前备份归档。
- 导入是幂等的，并以源路径、mtime、大小、哈希和目标表为键。
- 目标数据库提交后，成功处理的源文件将被删除或归档。
- 导入失败时保持源文件不变，并在
  `migration_runs` 中记录警告。
- 迁移存在后，运行时代码仅从 SQLite 读取。
- 无需降级或导出到运行时文件的路径。

## 迁移清单

将以下内容移入全局数据库：

- 任务注册表的运行时写入现在使用共享数据库；未发布的
  `tasks/runs.sqlite` 边车导入器已删除。保存快照时按任务
  id 执行 upsert，并且仅删除缺失的任务/投递行。
- Task Flow 的运行时写入现在使用共享数据库；未发布的
  `tasks/flows/registry.sqlite` 边车导入器已删除。保存快照时
  按流程 id 执行 upsert，并且仅删除缺失的流程行。
- 插件状态的运行时写入现在使用共享数据库；未发布的
  `plugin-state/state.sqlite` 边车导入器已删除。
- 内置记忆搜索不再默认使用 `memory/<agentId>.sqlite`；其
  索引表位于所属 Agent 数据库中，显式选择启用
  `memorySearch.store.path` 边车的方式已停用，改由 Doctor 配置
  迁移处理。
- 内置记忆重建索引仅重置 Agent 数据库中归记忆功能所有的表。
  它不得替换整个 SQLite 文件，因为同一数据库还存储
  会话、转录、VFS 行、工件和运行时缓存。
- 沙箱容器/浏览器注册表从单体和分片 JSON 迁移。运行时
  写入现在使用共享数据库；仍保留旧版 JSON 导入。
- 定时任务定义、调度状态和运行历史现在使用共享 SQLite；
  Doctor 会导入并删除旧版 `jobs.json`、`jobs-state.json` 和
  `cron/runs/*.jsonl` 文件
- 设备身份/身份验证、推送、更新检查、跟进承诺、OpenRouter 模型
  缓存、已安装插件索引和应用服务器绑定
- 设备/节点配对和引导记录现在使用类型化 SQLite 表
- 设备配对通知订阅者和已投递请求标记现在使用
  共享 SQLite 插件状态表，而不是 `device-pair-notify.json`。
- 语音通话记录现在使用 `voice-call` / `calls` 命名空间下的
  共享 SQLite 插件状态表，而不是 `calls.jsonl`；插件 CLI
  会跟踪并汇总由 SQLite 支持的通话历史。
- QQ Bot Gateway 网关会话、已知用户记录和引用索引引用缓存现在使用
  `qqbot` 命名空间（`gateway-sessions`、
  `known-users`、`ref-index`）下的 SQLite 插件状态，而不是 `session-*.json`、`known-users.json`
  和 `ref-index.jsonl`。这些旧版文件属于缓存，不会迁移。
- Discord 模型选择器偏好设置、命令部署哈希和线程绑定
  现在使用 `discord` 命名空间
  （`model-picker-preferences`、`command-deploy-hashes`、`thread-bindings`）
  下的 SQLite 插件状态，而不是 `model-picker-preferences.json`、`command-deploy-cache.json` 和
  `thread-bindings.json`；Discord Doctor/设置迁移会导入并
  删除旧版文件。
- BlueBubbles 补录游标和入站去重标记现在使用 `bluebubbles` 命名空间（`catchup-cursors`、`inbound-dedupe`）
  下的 SQLite 插件状态，而不是 `bluebubbles/catchup/*.json` 和
  `bluebubbles/inbound-dedupe/*.json`；BlueBubbles Doctor/设置迁移
  会导入并删除旧版文件。
- Telegram 更新偏移量、贴纸缓存条目、回复链消息缓存
  条目、已发送消息缓存条目、主题名称缓存条目和线程
  绑定现在使用 `telegram` 命名空间
  （`update-offsets`、`sticker-cache`、`message-cache`、`sent-messages`、
  `topic-names`、`thread-bindings`）下的 SQLite 插件状态，而不是 `update-offset-*.json`、
  `sticker-cache.json`、`*.telegram-messages.json`、
  `*.telegram-sent-messages.json`、`*.telegram-topic-names.json` 和
  `thread-bindings-*.json`；Telegram Doctor/设置迁移会导入并
  删除旧版文件。
- iMessage 补录游标、回复短 id 映射和已发送回显去重行
  现在使用 `imessage` 命名空间（`catchup-cursors`、
  `reply-cache`、`sent-echoes`）下的 SQLite 插件状态，而不是 `imessage/catchup/*.json`、
  `imessage/reply-cache.jsonl` 和 `imessage/sent-echoes.jsonl`；iMessage
  Doctor/设置迁移会导入并删除旧版文件。
- Microsoft Teams 对话、投票、SSO 令牌和反馈学习数据现在
  使用 SQLite 插件状态命名空间（`conversations`、`polls`、`sso-tokens`、
  `feedback-learnings`），而不是 `msteams-conversations.json`、
  `msteams-polls.json`、`msteams-sso-tokens.json` 和 `*.learnings.json`；Microsoft Teams
  Doctor/设置迁移会导入并归档旧版文件。
  待处理上传是短期 SQLite 缓存，旧 JSON 缓存文件
  不会迁移。
- Matrix 同步缓存、存储元数据、线程绑定、入站去重标记、
  启动验证冷却状态、凭据、恢复密钥和 SDK
  IndexedDB 加密快照现在使用
  `matrix` 下的 SQLite 插件状态/blob 命名空间（`sync-store`、`storage-meta`、`thread-bindings`、
  `matrix.inbound-dedupe.*` 通过核心可认领去重机制处理、
  `startup-verification`、`credentials`、`recovery-key`、`idb-snapshots`），
  而不是 `bot-storage.json`、`storage-meta.json`、`thread-bindings.json`、
  `inbound-dedupe.json`、`startup-verification.json`、`credentials.json`、
  `recovery-key.json` 和 `crypto-idb-snapshot.json`；Matrix Doctor/设置
  迁移会从账户范围的 Matrix 存储根目录中导入并删除这些旧版文件
  （以及已停用的每根目录 `inbound-dedupe` SQLite 行）。
- Nostr 总线游标和个人资料发布状态现在使用
  `nostr` 命名空间（`bus-state`、`profile-state`）下的 SQLite 插件状态，而不是
  `bus-state-*.json` 和 `profile-state-*.json`；Nostr Doctor/设置
  迁移会导入并删除旧版文件。
- 主动记忆会话开关现在使用
  `active-memory/session-toggles` 下的 SQLite 插件状态，而不是 `session-toggles.json`。
- 技能工作坊提案队列和审查计数器现在使用
  `skill-workshop/proposals` 和 `skill-workshop/reviews` 下的 SQLite 插件状态，而不是
  每个工作区的 `skill-workshop/<workspace>.json` 文件。
- 出站投递队列和会话投递队列现在通过不同队列名称
  （`outbound-delivery`、`session-delivery`）共享全局 SQLite
  `delivery_queue_entries` 表，而不是持久化的
  `delivery-queue/*.json`、`delivery-queue/failed/*.json` 和
  `session-delivery-queue/*.json` 文件。Doctor 的旧版状态步骤会导入
  待处理行和失败行，删除过时的已投递标记，并在导入后删除旧
  JSON 文件。热路径路由和重试字段使用类型化列；
  仅为重放/调试保留 JSON 载荷。
- ACPX 进程租约现在使用 `acpx/process-leases`
  下的 SQLite 插件状态，而不是 `process-leases.json`。
- 备份和迁移运行元数据

将以下内容移入 Agent 数据库：

- Agent 会话根目录和兼容形状的会话条目载荷。运行时写入
  已完成：热会话元数据可在 `sessions` 中查询，而
  旧版形状的完整 `SessionEntry` 载荷仍保留在 `session_entries` 中。
- Agent 转录事件。运行时写入已完成。
- 压缩检查点和转录快照。运行时写入已完成：
  检查点转录副本是 SQLite 转录行，检查点
  元数据记录在 `transcript_snapshots` 中。Gateway 网关检查点辅助函数
  现在将这些值称为转录快照，而不是源文件。
- Agent VFS 临时/工作区命名空间。运行时 VFS 写入已完成。
- 子智能体附件载荷。运行时写入已完成：它们是 SQLite VFS
  种子条目，绝不会成为持久工作区文件。
- 工具工件。运行时写入已完成。
- 运行工件。工作节点现在通过每个 Agent 的
  `run_artifacts` 表完成运行时写入。
- Agent 本地运行时缓存。工作节点现在通过每个 Agent 的
  `cache_entries` 表完成限定范围的运行时缓存写入。Gateway 网关范围的模型缓存仍保留在
  全局数据库中，除非它们变为特定于 Agent。
- ACP 父流日志。运行时写入已完成。
- ACP 重放账本会话。现在通过
  `acp_replay_sessions` 和 `acp_replay_events` 完成运行时写入；旧版 `acp/event-ledger.json`
  仅保留为 Doctor 输入。
- ACP 会话元数据。现在通过 `acp_sessions` 完成运行时写入；`sessions.json` 中的旧版
  `entry.acp` 块仅作为 Doctor 迁移输入。
- 并非显式导出文件的轨迹边车。运行时
  写入已完成：轨迹捕获会写入 Agent 数据库的 `trajectory_runtime_events`
  行，并将运行范围的工件镜像到 SQLite 中。旧版边车仅作为 Doctor
  导入输入；导出可以生成新的 JSONL 支持包输出，
  但不会在运行时读取或迁移旧轨迹/转录边车。
  运行时轨迹捕获会公开 SQLite 作用域；JSONL 路径辅助函数
  仅用于导出/调试支持，不会从运行时模块重新导出。
  嵌入式运行器轨迹元数据记录 `{agentId, sessionId, sessionKey}`
  身份，而不是持久化转录定位器。

目前仍将以下内容保留为文件存储：

- `openclaw.json`
- 提供商或 CLI 凭据文件
- 插件/包清单
- 选择磁盘模式时的用户工作区和 Git 仓库
- 供操作员跟踪的日志，除非特定日志表面已迁移

## 迁移计划

### 阶段 0：冻结边界

在迁移更多行之前明确持久状态边界：

- 向全局数据库添加 `migration_runs` 表。
  已用于旧版状态迁移执行报告。
- 添加由 Doctor 单独所有的状态迁移服务，用于从文件导入数据库。
  已完成：`openclaw doctor --fix` 使用旧版状态迁移实现。
- 将 `plan` 设为只读，并让 `apply` 创建备份、导入、验证，
  然后删除或隔离旧文件。
  已完成：Doctor 会创建经过验证的迁移前备份，将备份路径
  传入 `migration_runs`，并复用导入器/删除路径。
- 添加静态禁令，使新的运行时代码无法写入旧版状态文件，同时
  迁移代码和测试仍可植入/读取这些文件。
  当前已迁移的旧版存储已完成；该守卫还会扫描嵌套
  测试中被禁止的运行时转录定位器契约。

### 阶段 1：完成全局控制平面

将共享协调状态保留在 `state/openclaw.sqlite` 中：

- Agent 和 Agent 数据库注册表
- 任务和 Task Flow 账本
- 插件状态
- 沙箱容器/浏览器注册表
- 定时任务/调度器运行历史
- 配对、设备、推送、更新检查、TUI、OpenRouter/模型缓存和其他
  小型 Gateway 网关范围运行时状态
- 备份和迁移元数据
- Gateway 网关媒体附件字节。运行时写入已完成；直接文件路径
  是为了与渠道发送器和沙箱暂存保持兼容而生成的临时物化文件。
  运行时允许列表接受 SQLite 物化路径，而不是旧版
  状态/配置媒体根目录。Doctor 会将旧版媒体文件导入
  `media_blobs`，并在成功写入行后删除源文件。
- 调试代理捕获会话、事件和载荷 blob。已完成：捕获数据
  位于共享状态数据库中，并通过共享状态数据库的引导、架构、
  WAL 和忙碌超时设置打开。载荷字节在
  `capture_blobs.data` 中使用 gzip 压缩；不存在调试代理运行时边车数据库覆盖、
  blob 目录或仅供代理捕获使用的生成架构/代码生成目标。
  Doctor/启动迁移会导入已发布的 `debug-proxy/capture.sqlite` 行
  和引用的载荷 blob，包括仍生效的旧版数据库/blob 环境变量
  覆盖，然后归档这些源，同时保留 CA 证书不变。

此阶段还会从这些子系统中删除重复的旁路文件打开器、权限辅助函数、WAL
设置、文件系统清理以及兼容性写入器。

### 阶段 2：引入每 Agent 数据库

为每个 Agent 创建一个数据库，并从全局数据库中注册：

```text
~/.openclaw/state/openclaw.sqlite
~/.openclaw/agents/<agentId>/agent/openclaw-agent.sqlite
```

全局 `agent_databases` 行存储路径、架构版本、最后发现时间戳以及基本的
大小/完整性元数据。运行时代码向注册表查询 Agent 数据库，而不是直接派生
文件路径。

Agent 数据库负责：

- `sessions` 作为规范会话根，`session_entries` 是附加到该根的
  兼容性形状载荷表，`session_routes` 是唯一的活动 `session_key` 查找
- `conversations` 和 `session_conversations` 作为附加到会话的标准化提供商
  路由标识
- `transcript_events`
- 转录快照和压缩检查点。运行时写入已完成。
- `vfs_entries`
- `tool_artifacts` 和运行工件
- Agent 本地运行时/缓存行。工作进程作用域缓存已完成。
- ACP 父级流事件
- 当轨迹运行时事件不是显式导出工件时存储这些事件

### 阶段 3：替换会话存储 API

运行时部分已完成。文件形状的会话存储表面不再是有效的
运行时契约：

- 运行时不再调用 `loadSessionStore(storePath)`，也不再将 `storePath` 视为
  会话标识。
- 运行时行操作为 `getSessionEntry`、`upsertSessionEntry`、
  `patchSessionEntry`、`deleteSessionEntry` 和 `listSessionEntries`。
- 全存储重写辅助函数、文件写入器、队列测试、别名清理以及
  旧键删除参数均已从运行时中移除。
- 弃用的根软件包兼容性导出会委托给仅供 Doctor 使用的
  `sessions.json` 导入器，直至 2026-10-12；插件 SDK 兼容性读取会
  继续投影规范 SQLite 行。
- `sessions.json` 解析仅保留在 Doctor 迁移/导入代码和
  Doctor 测试中。
- 运行时生命周期回退读取 SQLite 转录标头，而不是 JSONL 首行。

继续删除任何重新引入文件锁参数、
将清理/截断视为文件维护的术语、存储路径标识，或唯一断言为 JSON 持久化的测试。

### 阶段 4：迁移转录、ACP 流、轨迹和 VFS

使每个 Agent 数据流都原生使用数据库：

- 转录追加写入通过一个 SQLite 事务完成，该事务确保会话标头存在、
  检查消息幂等性、选择父级尾部、插入
  `transcript_events`，并在 `transcript_event_identities` 中记录可查询的标识元数据。
  直接追加转录消息和普通持久化 `TranscriptSessionManager` 的追加已完成；
  显式分支操作会保留其明确选择的父级，并继续写入 SQLite 行，
  而不派生任何文件定位符。
- ACP 父级流日志改为行，而不是 `.acp-stream.jsonl` 文件。已完成。
- ACP 生成设置不再持久化转录 JSONL 路径。已完成。
- 运行时轨迹捕获直接写入事件行/工件。显式的
  支持/导出命令仍可生成支持包 JSONL 工件作为导出格式，
  但会话导出不会重新创建会话 JSONL。已完成。
- 配置为磁盘模式时，磁盘工作区仍保留在磁盘上。
- VFS 暂存区和实验性的仅 VFS 工作区模式使用 Agent 数据库。

迁移会一次性导入旧 JSONL 文件，在
`migration_runs` 中记录计数/哈希值，并在完整性检查后删除已导入文件。

### 阶段 5：备份、恢复、Vacuum 和验证

备份仍为单个归档文件：

- 为每个全局数据库和 Agent 数据库创建检查点。
- 使用 SQLite 在线备份为每个数据库创建快照，随后离线执行 `VACUUM`。
- 归档紧凑的数据库快照、配置、外部凭据以及请求的
  工作区导出。
- 省略原始实时 `*.sqlite-wal` 和 `*.sqlite-shm` 文件。
- 通过打开每个数据库快照并运行 `PRAGMA integrity_check` 进行验证。
  `openclaw backup create` 默认执行此归档验证；
  `--no-verify` 仅跳过写入后的归档检查，而不跳过快照
  创建完整性检查。
- 恢复操作会将快照复制回其目标路径。恢复后的全局数据库使用
  版本 `1`；恢复后的每 Agent 数据库使用版本 `2`，版本 `1` 的快照
  会在打开时以原子方式升级。

### 阶段 6：工作进程运行时

在数据库拆分落地期间，工作进程模式保持为实验性功能：

- 工作进程接收 Agent ID、运行 ID、文件系统模式和数据库注册表标识。
- 每个工作进程打开自己的 SQLite 连接。
- 父级保留渠道递送、审批、配置和取消权限。
- 初始采用每个活动运行一个工作进程；只有在生命周期和数据库
  连接所有权稳定后才添加池化。

### 阶段 7：删除旧世界

运行时会话管理部分已完成。旧世界只能作为显式的
Doctor 输入或支持/导出输出存在：

- 运行时不会写入 `sessions.json`、转录 JSONL、沙箱注册表 JSON、任务
  旁路 SQLite 或插件状态旁路 SQLite。
- 不再进行 JSON/会话文件清理、文件转录截断、会话文件锁，
  也不再保留锁形状的会话测试。
- 不再提供以保持旧会话文件最新为目的的运行时兼容性导出。
- 显式支持导出仍是用户请求的归档/物化
  格式，且不得将文件名反馈到运行时标识中。

## 备份和恢复

备份应为单个归档文件，但数据库捕获应
原生使用 SQLite：

1. 保持写入事务有界，以便在线备份能够持续推进。
2. 捕获前验证每个实时全局数据库和 Agent 数据库。
3. 使用 SQLite 在线备份将每个数据库捕获到临时备份
   目录中，然后关闭实时连接，并对私有副本执行 `VACUUM`。
   需要所有者定义 SQLite 能力的插件架构会以失败关闭方式处理，
   直到所有者提供安全的快照契约。
4. 归档数据库快照、配置文件、凭据目录、选定的
   工作区以及清单。
5. 验证每个 SQLite 快照的文件形状，然后打开规范 OpenClaw
   数据库，并运行 `PRAGMA integrity_check` 和角色验证。专用
   插件架构保持不透明，除非其所有者提供验证器。
   `openclaw backup create` 默认执行此操作；`--no-verify` 仅用于
   有意跳过写入后的归档检查。

不要依赖原始实时 `*.sqlite`、`*.sqlite-wal` 和 `*.sqlite-shm` 副本作为
主要备份格式。归档清单应记录数据库角色、
Agent ID、架构版本、源路径、快照路径、字节大小和完整性
状态。

恢复操作应从归档快照重建全局数据库和 Agent 数据库文件。
全局架构保持为版本 `1`；每 Agent 版本 `1`
快照会接受有界的运行时升级，升至版本 `2`。Doctor 仍是
文件到数据库导入的唯一所有者。恢复命令会先验证
归档，然后使用已验证的解压载荷替换清单中的每项资产。

## 运行时重构计划

1. 添加数据库注册表 API。
   - 解析全局数据库和每 Agent 数据库路径。
   - 将全局架构保持在 `user_version = 1`。每 Agent 数据库使用版本 `2`，
     并从已发布的版本 `1` 内存源形状进行一次原子迁移。
   - 添加供测试、备份和 Doctor 使用的关闭/检查点/完整性辅助函数。

2. 合并旁路 SQLite 存储。
   - 将插件状态表迁移到全局数据库。运行时
     写入已完成；未发布的旧旁路导入器已删除。
   - 将任务注册表迁移到全局数据库。运行时
     写入已完成；未发布的旧旁路导入器已删除。
   - 将 Task Flow 表迁移到全局数据库。运行时写入已完成；
     未发布的旧旁路导入器已删除。
   - 将内置记忆搜索表迁移到每个 Agent 数据库。已完成；显式
     自定义 `memorySearch.store.path` 现在由 Doctor 配置迁移移除。
     完整重建索引仅针对记忆表原地运行；旧的全文件
     交换路径和旁路索引交换辅助函数已删除。
   - 从这些子系统中删除重复的数据库打开器、WAL 设置、权限辅助函数和
     关闭路径。

3. 将 Agent 所有的表迁移到每 Agent 数据库。
   - 通过全局数据库注册表按需创建 Agent 数据库。已完成。
   - 将运行时会话条目、转录事件、VFS 行和工具
     工件迁移到 Agent 数据库。已完成。
   - 不要迁移分支本地共享数据库中的会话条目、转录事件、
     VFS 行或工具工件；该布局从未发布。仅在 Doctor 中保留旧版
     文件到数据库的导入。

4. 替换会话存储 API。
   - 移除 `storePath` 的运行时标识用途。运行时部分已完成，并由
     `check:database-first-legacy-stores` 提供保障：会话元数据、路由更新、
     命令持久化、CLI 会话清理、Feishu 推理预览、
     转录状态持久化、子智能体深度、身份验证配置文件会话
     覆盖、父级分叉逻辑以及 QA Lab 检查现在都通过规范的
     Agent/会话键解析数据库。
     Gateway 网关/TUI/UI/macOS 会话列表响应现在公开 `databasePath`
     而不是旧版 `path`；macOS 调试表面将每 Agent 数据库
     显示为只读状态，而不是写入 `session.store` 配置。
     `/status`、由聊天驱动的轨迹导出以及 CLI 依赖代理不再
     传播旧版存储路径；转录用量回退通过 Agent/会话标识读取
     SQLite。运行时和桥接测试不再公开
     `storePath`；Doctor/迁移输入负责该旧字段名称。
     Gateway 网关组合会话加载不再为非模板化的
     `session.store` 值保留特殊运行时分支；它会聚合每 Agent SQLite 行。
     旧版会话锁 Doctor 通道及其 `.jsonl.lock` 清理辅助函数
     已移除；SQLite 现在是会话并发边界。
     热点运行时调用位置使用面向行的辅助函数名称，例如
     `resolveSessionRowEntry`；旧的 `resolveSessionStoreEntry` 兼容性
     别名已从运行时和插件 SDK 导出中移除。

- 使用 `{ agentId, sessionKey }` 行操作。
  已完成：`getSessionEntry`、`upsertSessionEntry`、`deleteSessionEntry`、
  `patchSessionEntry` 和 `listSessionEntries` 现已成为 SQLite 优先的 API，
  不需要会话存储路径。状态摘要、本地智能体状态、健康状态和
  `openclaw sessions` 列表命令现在直接读取每个智能体的行，
  并显示每个智能体的 SQLite 数据库路径，而不是 `sessions.json` 路径。
- 将全存储删除/插入替换为 `upsertSessionEntry`、
  `deleteSessionEntry`、`listSessionEntries` 和 SQL 清理查询。
  运行时已完成：热路径现在使用行 API 和冲突重试的行补丁；
  剩余的全存储导入/替换辅助函数仅限迁移导入代码和 SQLite 后端测试。
  - 删除 `store-writer.ts` 和写入器队列测试。已完成。
  - 从会话行更新插入/补丁中删除运行时旧键清理和别名删除参数。
    已完成。

5. 删除运行时 JSON 注册表行为。
   - 使沙箱注册表仅通过 SQLite 读写。已完成。
   - 仅在迁移步骤中导入单体和分片 JSON。已完成。
   - 移除分片注册表锁和 JSON 写入。已完成。

- 如果注册表行的结构仍属于热路径操作状态，则使用一个类型化注册表表，
  而不是将注册表行存储为通用不透明 JSON。已完成。

6. 删除文件锁形式的会话变更。
   - 运行时锁创建和运行时锁 API 已完成。
   - 独立的旧版 `.jsonl.lock` Doctor 清理通道已移除。
   - 状态完整性不再包含单独的孤立转录文件清理路径；
     Doctor 迁移会在同一位置导入/移除旧版 JSONL 源。
   - Gateway 网关单例协调在 `gateway_locks` 下使用类型化 SQLite
     `state_leases` 行，不再暴露文件锁目录接缝。
   - 通用插件 SDK 去重持久化不再使用文件锁或 JSON 文件；
     它会写入共享 SQLite 插件状态行。已完成。
   - QMD 协调对嵌入使用共享 SQLite 租约，并对每个集合/更新/嵌入写入器
     使用每个智能体的 SQLite 租约。运行时不再创建 `qmd/embed.lock.lock`
     或 `agents/<agentId>/qmd-write.lock.lock`；Doctor 仅移除确定已过时的退役旁路文件。已完成。

7. 使工作进程感知数据库。
   - 工作进程打开各自的 SQLite 连接。
   - 父进程负责交付、渠道回调和配置。
   - 工作进程接收智能体 ID、运行 ID、文件系统模式和数据库注册表标识，
     而不是实时句柄。
   - `vfs-only` 保持实验性，并使用智能体数据库作为其存储根目录。
   - 首先为每个活动运行保留一个工作进程。池化可以等到数据库连接生命周期
     和取消行为足够稳定后再进行。

8. 备份集成。
   - 让备份通过在线备份后执行离线 `VACUUM`，
     对全局、智能体和插件数据库创建快照。对于状态资产下发现的
     `*.sqlite` 文件，此项已完成；需要不可用所有者能力的插件架构会以失败关闭方式处理。
   - 为规范 SQLite 完整性和架构标识添加备份验证，
     并为专用插件快照添加通用文件结构验证。备份创建和默认归档验证已完成。
   - 在 SQLite 中记录备份运行元数据。已通过共享 `backup_runs`
     表完成，其中包含归档路径、状态和清单 JSON。
   - 添加从已验证归档快照恢复的功能。已完成：`openclaw backup
restore`
     在提取前进行验证，使用验证器的规范化清单，支持 `--dry-run`，
     并要求在替换记录的源路径前提供 `--yes`。
   - 仅在请求时包含 VFS/工作区导出；不要将会话内部数据导出为 JSON 或 JSONL。

9. 删除过时的测试和代码。已完成已知的运行时会话表面。

- 移除断言运行时创建 `sessions.json` 或转录
  JSONL 文件的测试。核心会话存储、聊天、Gateway 网关转录事件、
  预览、生命周期、命令会话条目更新、自动回复重置/跟踪、
  memory-core 梦境固件、审批目标路由、会话转录修复、安全权限修复、
  轨迹导出和会话导出均已完成。
  主动记忆转录测试现在断言 SQLite 作用域，并且不会创建临时或持久化的
  JSONL 文件。
  旧的 Heartbeat 转录清理回归测试已移除，因为运行时不再截断 JSONL 转录。
  智能体会话列表工具测试不再将旧版 `sessions.json` 路径建模为
  Gateway 网关响应结构；应用/UI/macOS 测试使用 `databasePath`。
  `/status` 转录使用情况测试现在直接植入 SQLite 转录行，
  而不是写入 JSONL 文件。
  Gateway 网关会话生命周期测试现在直接使用 SQLite 转录植入辅助函数；
  旧的单行会话文件固件结构已从重置和删除覆盖中移除。
  `sessions.delete` 不再返回文件时代的 `archived: []` 字段；
  删除操作仅报告行变更结果。旧的 `deleteTranscript` 选项也已移除：
  删除会话会移除规范的 `sessions` 根，并让 SQLite 级联删除会话所有的
  转录、快照和轨迹行，因此调用方无法遗留转录孤儿，也不会遗漏清理分支。
  上下文引擎轨迹捕获测试现在从隔离的智能体数据库读取
  `trajectory_runtime_events` 行，而不是读取 `session.trajectory.jsonl`。
  Docker MCP 渠道植入脚本现在直接植入 SQLite 行。直接写入
  `sessions.json` 仅限 Doctor 固件。
  工具搜索 Gateway 网关 E2E 从 SQLite 转录行读取工具调用证据，
  而不是扫描 `agents/<agentId>/sessions/*.jsonl` 文件。
  memory-core 主机事件和会话语料库暂存行现在位于共享 SQLite 插件状态中；
  `events.jsonl` 和 `session-corpus/*.txt` 仅作为旧版 Doctor 迁移输入。
  活动行使用 `memory/session-ingestion/` 虚拟路径，而不是 `.dreams/session-corpus`。
  旧的 memory-core 梦境修复模块及其 CLI/Gateway 网关测试已移除，
  因为运行时不再负责该语料库的文件归档修复。memory-core
  桥接/公共工件测试不再显示 `.dreams/events.jsonl`；它们使用由 SQLite
  支持的虚拟 JSON 工件名称。
  公共 SDK/Codex 测试文档现在使用“SQLite 会话状态”而不是“会话文件”，
  且渠道轮次示例不再暴露 `storePath` 参数。
  Matrix 同步状态现在直接使用 SQLite 插件状态存储。活动客户端/运行时契约
  传递账户存储根目录，而不是 `bot-storage.json` 路径；Doctor 会先将旧版
  `bot-storage.json` 导入 SQLite，然后删除源。QA Lab Matrix 重启/破坏性场景
  现在直接修改 SQLite 同步行，而不是创建或删除虚假的 `bot-storage.json`
  文件；E2EE 基础层传递同步存储根目录，而不是虚假的 `sync-store.json` 路径。
  Matrix 存储根目录选择不再根据旧版同步/线程 JSON 文件为根目录评分；
  它使用持久根元数据和真实加密状态。
  运行时 SQLite 会话后端测试套件不再伪造 `sessions.json`；
  旧版源固件现在位于导入它们的 Doctor 测试中。
  Gateway 网关会话测试不再暴露 `createSessionStoreDir` 辅助函数或
  未使用的临时会话存储路径设置；固件目录会显式指定，直接行设置使用
  SQLite 会话行命名。
  仅限 Doctor 的 JSON5 会话存储解析器覆盖已从基础设施测试移至
  Doctor 迁移测试，因此运行时测试套件不再负责旧版会话文件解析。
  Microsoft Teams 运行时 SSO/待上传测试不再携带 JSON 旁路文件固件或解析器；
  旧版 SSO 令牌解析仅存在于插件迁移模块中。Telegram 测试不再植入虚假的
  `/tmp/*.json` 存储路径；它们直接重置由 SQLite 支持的消息缓存。
  通用 OpenClaw 测试状态辅助函数不再暴露旧版 `auth-profiles.json`
  写入器；Doctor 身份验证迁移测试在本地负责该固件。
  TUI 最后会话指针、Exec 审批、主动记忆开关、Matrix 去重/启动验证、
  Memory Wiki 源同步、当前对话绑定、新手引导身份验证和 Hermes 密钥导入的
  运行时测试不再制造旧旁路文件，也不再断言旧文件名不存在。
  它们通过 SQLite 行和公共存储 API 证明行为；只有 Doctor/迁移测试中
  才应出现旧版源文件名。
  设备/节点配对、渠道 allowFrom、重启意图、重启交接、会话交付队列条目、
  配置健康、iMessage 缓存、定时任务、PI 转录标头、子智能体注册表和托管图像
  附件的运行时测试，也不再为了证明退役 JSON/JSONL 文件会被忽略或不存在
  而创建这些文件。
  PI 溢出恢复不再具有 SessionManager 重写/截断回退：工具结果截断和
  上下文引擎转录重写会修改 SQLite 转录行，然后从数据库刷新活动提示状态。
  持久化的 SessionManager 消息追加会委托给原子 SQLite 转录追加辅助函数，
  以处理父项选择和幂等性。常规元数据/自定义条目追加也会在 SQLite 内选择
  当前父项，因此过时的管理器实例不会重新引入 SQLite 之前的父链竞态。
  用于轮次中途预检查和 `sessions_yield` 的合成 PI 尾部清理现在直接修剪
  SQLite 转录状态；旧的 SessionManager 尾部移除桥接及其测试已删除。
  压缩检查点捕获也仅从 SQLite 创建快照；调用方不再将实时 SessionManager
  作为备用转录源传递。
- 仅为迁移保留植入旧版文件的测试。
- 对于活动运行时表面，JSON 文件证明已替换为 SQL 行证明。

- 添加静态禁令，禁止运行时写入旧版会话/缓存 JSON 路径。
  仓库防护已完成。

10. 使迁移报告可审计。
    - 在 SQLite 中记录迁移运行，包括开始/完成时间戳、源路径、
      源哈希、计数、警告和备份路径。
      已完成：旧版状态迁移执行现在会持久化 `migration_runs`
      报告，其中包含源路径/表清单、源文件 SHA-256、大小、记录数、
      警告和备份路径。
      已完成：旧版状态迁移执行还会持久化 `migration_sources` 行，
      用于源级审计以及未来的跳过/回填决策。
    - 使应用操作具备幂等性。部分导入后重新运行时，应跳过
      已导入的源，或按稳定键合并。
      已完成：会话索引、转录、交付队列、插件状态、任务账本以及智能体所有的
      全局 SQLite 行通过稳定键或更新插入/替换语义导入，因此重新运行时会合并，
      而不会重复持久行。
    - 导入失败时必须将原始源文件保留在原位。
      已完成：转录导入失败时，原始 JSONL 源现在会保留在检测到的路径，
      `migration_sources` 会将该源记录为 `warning`，
      并带有供下一次 Doctor 运行使用的 `removed_source=0`。

## 性能规则

- 每个线程/进程使用一个连接即可；不要在工作线程之间共享句柄。
- 使用 WAL、`foreign_keys=ON`、5 秒的忙碌超时和简短的 `BEGIN IMMEDIATE`
  写事务。不要在 SQLite 的单次忙碌等待机制之上叠加同步锁重试。
- 除非/直到异步事务 API 增加明确的互斥锁/背压语义，否则应保持写事务辅助函数同步执行。
- 保持父级交付写入小型化并使用事务。
- 避免重写整个存储；使用行级 upsert/delete。
- 在迁移热路径代码之前，为按智能体列出、按会话列出、更新时间、运行 ID 和
  过期路径添加索引。
- 将大型工件、媒体和向量存储为 BLOB 或分块 BLOB 行，而不是
  base64 或数值数组 JSON。
- 保持不透明的插件状态条目小型化且限定作用域。
- 使用 SQL 清理 TTL/过期数据，而不是通过文件系统修剪。
  数据库所有的运行时存储已完成此项：媒体、插件状态、插件 BLOB、
  持久化去重和 Agent 缓存均通过 SQLite 行过期。剩余的
  文件系统清理仅限于临时物化内容或显式
  删除命令。

## 静态禁令

添加仓库检查，使针对旧版状态路径的新运行时写入失败：

- `sessions.json`
- `*.trajectory.jsonl`，物化的支持包输出除外
- `.acp-stream.jsonl`
- `acp/event-ledger.json`
- `cache/*.json` 运行时缓存文件
- `agents/<agentId>/agent/auth.json`
- `agents/<agentId>/agent/models.json`
- `credentials/oauth.json`
- `github-copilot.token.json`
- `openrouter-models.json`
- `auth-profiles.json`
- `auth-state.json`
- `exec-approvals.json`
- `openclaw-workspace-state.json`
- `workspace-state.json`
- `workspace-attestations/*.attested`
- 同级 `<workspace>.attested`
- Matrix `credentials*.json` 和 `recovery-key.json`
- `cron/runs/*.jsonl`
- `cron/jobs.json`
- `jobs-state.json`
- `device-pair-notify.json`
- `devices/pending.json` / `devices/paired.json` / `devices/bootstrap.json`
  （已于 2026.7 停用：运行时存储为共享状态数据库中的 `device_pairing_*` /
  `device_bootstrap_tokens`；配对记录在 Gateway 网关启动时导入，
  临时待处理/引导行会被丢弃）
- `nodes/pending.json` / `nodes/paired.json`（已于 2026.7 停用：在 Gateway 网关启动时并入已配对设备记录）
- `identity/device.json`
- `identity/device-auth.json`（已停用；仅由 Doctor 导入 `device_auth_tokens`）
- `push/web-push-subscriptions.json`（已停用；仅由 Doctor 导入 `web_push_subscriptions`）
- `push/vapid-keys.json`（已停用；仅由 Doctor 导入 `web_push_vapid_keys`）
- `push/apns-registrations.json`（已停用；仅由 Doctor 导入 `apns_registrations`）
- `process-leases.json`
- `gateway-instance-id`
- `session-toggles.json`
- Memory-core `.dreams/events.jsonl`
- Memory-core `.dreams/session-corpus/`
- Memory-core `.dreams/daily-ingestion.json`
- Memory-core `.dreams/session-ingestion.json`
- Memory-core `.dreams/short-term-recall.json`
- Memory-core `.dreams/phase-signals.json`
- Memory-core `.dreams/short-term-promotion.lock`
- Skill Workshop `skill-workshop/<workspace>.json`
- Skill Workshop `skill-workshop/skill-workshop-review-*.json`
- Nostr `bus-state-*.json`
- Nostr `profile-state-*.json`
- `calls.jsonl`
- `known-users.json`
- `ref-index.jsonl`
- QQ Bot `session-*.json`
- BlueBubbles `bluebubbles/catchup/*.json`
- BlueBubbles `bluebubbles/inbound-dedupe/*.json`
- Telegram `update-offset-*.json`
- Telegram `sticker-cache.json`
- Telegram `*.telegram-messages.json`
- Telegram `*.telegram-sent-messages.json`
- Telegram `*.telegram-topic-names.json`
- Telegram `thread-bindings-*.json`
- iMessage `catchup/*.json`
- iMessage `reply-cache.jsonl`
- iMessage `sent-echoes.jsonl`
- Microsoft Teams `msteams-conversations.json`
- Microsoft Teams `msteams-polls.json`
- Microsoft Teams `msteams-sso-tokens.json`
- Microsoft Teams `*.learnings.json`
- Matrix `bot-storage.json`
- Matrix `sync-store.json`
- Matrix `thread-bindings.json`
- Matrix `inbound-dedupe.json`
- Matrix `startup-verification.json`
- Matrix `storage-meta.json`
- Matrix `crypto-idb-snapshot.json`
- Discord `model-picker-preferences.json`
- Discord `command-deploy-cache.json`
- 沙箱注册表分片 JSON 文件
- `plugin-state/state.sqlite`
- 临时拼凑的 `openclaw-state.sqlite` 运行时边车文件
- `tasks/runs.sqlite`
- `tasks/flows/registry.sqlite`
- `bindings/current-conversations.json`
- `restart-sentinel.json`
- `gateway-restart-intent.json`
- `gateway-supervisor-restart-handoff.json`
- `gateway.<hash>.lock`
- `qmd/embed.lock.lock`
- `agents/<agentId>/qmd-write.lock.lock`
- `commands.log`
- `config-health.json`
- `port-guard.json`
- `settings/voicewake.json`
- `settings/voicewake-routing.json`
- `plugin-binding-approvals.json`
- `plugins/installs.json`
- `audit/file-transfer.jsonl`
- `audit/crestodian.jsonl`
- `crestodian/rescue-pending/*.json`
- `openclaw/rescue-pending/*.json`
- `plugins/phone-control/armed.json`
- Memory Wiki `.openclaw-wiki/log.jsonl`
- Memory Wiki `.openclaw-wiki/state.json`
- Memory Wiki `.openclaw-wiki/locks/`
- Memory Wiki `.openclaw-wiki/source-sync.json`
- Memory Wiki `.openclaw-wiki/import-runs/*.json`
- Memory Wiki `.openclaw-wiki/cache/agent-digest.json`
- Memory Wiki `.openclaw-wiki/cache/claims.jsonl`
- ClawHub `.clawhub/lock.json`
- ClawHub `.clawhub/origin.json`
- 浏览器配置文件装饰 `.openclaw-profile-decorated`
- `SessionManager.open(...)` 基于文件的会话打开器
- `SessionManager.listAll(...)` 和 `TranscriptSessionManager.listAll(...)`
  转录内容列出门面
- `SessionManager.forkFromSession(...)` 和
  `TranscriptSessionManager.forkFromSession(...)` 转录内容分叉门面
- `SessionManager.newSession(...)` 和 `TranscriptSessionManager.newSession(...)`
  可变会话替换门面
- `SessionManager.createBranchedSession(...)` 和
  `TranscriptSessionManager.createBranchedSession(...)` 分支会话门面

该禁令应允许测试创建旧版固件，并允许迁移代码
读取/导入/删除旧版文件源。未发布的 SQLite 边车文件仍被禁止，
且不允许由 Doctor 导入。

## 完成标准

- 运行时数据和缓存写入全局或 Agent SQLite 数据库。
- 运行时不再写入会话索引、转录 JSONL、沙箱注册表
  JSON、任务边车 SQLite 或插件状态边车 SQLite。未发布的任务
  和插件状态边车 SQLite 导入器已删除。
- 旧版文件仅可由 Doctor 导入。
- 备份生成一个归档，其中包含紧凑的 SQLite 快照和完整性证明。
- Agent 工作线程可使用磁盘、VFS 暂存区或实验性的纯 VFS
  存储运行。
- 配置和显式凭据文件仍是仅有的预期持久化
  非数据库控制文件。
- 仓库检查可防止重新引入旧版运行时文件存储。
