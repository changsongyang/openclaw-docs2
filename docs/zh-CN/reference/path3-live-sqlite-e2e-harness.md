---
read_when:
    - 你正在针对实时 Gateway 网关验证 Path 3 SQLite 存储切换。
    - 你需要区分预期的旧版 JSONL 漂移与运行时故障
    - 你正在构建或审查由智能体驱动的实时 SQLite 端到端测试工具链
summary: Path 3 SQLite 会话/转录切换的实时 Gateway 网关验证设计
title: Path 3 live SQLite E2E harness
x-i18n:
    generated_at: "2026-07-26T06:59:52Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 2749bf47cb4967bc80a5ed37a12f2a553f3b388ed8cd90cfb3217e1b5e8afae9
    source_path: reference/path3-live-sqlite-e2e-harness.md
    workflow: 16
---

Path 3 live SQLite E2E harness 用于证明 Gateway 网关将 SQLite 用作规范的会话和转录存储，而旧版 JSONL 文件仍作为迁移输入或归档材料。它是供维护者使用的验证工具，而不是普通的用户诊断工具。

Gateway 网关处理迁移后流量后，旧版 JSONL 一致性不再是有效的运行时健康信号。健康的已迁移 Gateway 网关中，SQLite 转录行数可能与旧版 JSONL 计数不同，因为新轮次应仅推进 SQLite。因此，live harness 必须在每一步测量 Gateway 网关行为、SQLite 行变动、旧版文件静止状态和日志健康状况。

## 命令形式

预期的实时命令为：

```bash
node scripts/path3-live-sqlite-e2e.mjs \
  --url http://127.0.0.1:18789 \
  --agent main \
  --session-key agent:main:path3-live-e2e:<timestamp> \
  --json
```

该命令连接到已在运行的 Gateway 网关。除非以后添加显式迁移模式，否则它不会启动、停止、导入或重新运行迁移。CI 或隔离的本地变体可以使用
`test/helpers/openclaw-test-instance.ts`，但实时验证路径应检查实际的操作员 Gateway 网关及其真实的每 Agent SQLite 数据库。

## 隔离的构建版 CLI 验证

构建版 CLI 验证运行器会植入隔离的旧版会话存储，启动重新构建的 Gateway 网关，并证明启动过程会在运行时读取开始前将活跃的旧版会话导入 SQLite。首次启动 Gateway 网关前不得运行 `openclaw doctor --fix`，否则验证的是手动迁移路径，而不是用户在切换后首次启动时获得的升级路径。

启动导入后，隔离验证可以运行
`openclaw doctor --session-sqlite inspect` 和
`openclaw doctor --session-sqlite validate` 作为诊断证据。这些 Doctor 命令不是启动升级验证的迁移驱动程序。单独的 Doctor 导入场景应植入旧版转录文件和轨迹 sidecar，并验证 Doctor 会归档这些工件，而 SQLite 仍保持规范存储地位。

## 预检

预检会收集基线；如果 Gateway 网关不可用，则在发送验证轮次前失败：

- `GET /health` 和 Gateway 网关深度状态必须报告 Gateway 网关正在运行且可访问。
- CLI 和 Gateway 网关版本必须与正在测试的分支匹配。
- harness 会记录活动 Gateway 网关文件日志的日志游标。
- harness 会记录每 Agent SQLite 中 `sessions`、
  `session_entries`、`transcript_events`、`transcript_event_identities` 和
  `session_routes` 的表计数。
- harness 会记录旧版 `sessions.json`、所引用的 JSONL 文件和候选验证会话 JSONL
  路径的 `mtime`、`size` 以及存在状态。
- `lsof -p <gateway-pid>` 必须显示 SQLite DB/WAL/SHM 句柄，且没有活跃的
  `.jsonl` 或 `sessions.json` 句柄。

在实时模式下，`openclaw doctor --session-sqlite validate` 仅供参考。
在切换后流量产生后，它可能会报告与旧版文件之间的预期偏差。harness 应使用 Doctor 输出进行分类和迁移清点，而不是将其作为运行时通过/失败的判定依据。

## Agent 驱动的场景

实时场景使用专用验证会话键，并尽可能通过公共 RPC 路径驱动 Gateway 网关。一个 Agent 轮次应足以触发常规持久化，但完整验证应覆盖此前需要逐项实时检查的 3.1b 接缝：

- 普通聊天轮次：创建或复用验证会话，发送真实的 Agent 提示，等待最终智能体结果，并验证 `chat.history` 或等效的 Gateway 网关投影。
- 转录身份：验证同一标记同时出现在 Gateway 网关历史记录和 SQLite 转录行中，包括存在时的稳定事件身份行。
- 会话元数据访问器：通过 Gateway 网关/会话访问器读取验证会话和选定的现有实时会话，并将其与 SQLite 行进行比较。
- 会话补丁投影：对验证会话应用可逆的模型/会话元数据更改，然后验证投影行与 Gateway 网关响应一致。
- 压缩检查点生命周期：仅在验证会话或 harness 创建的合成夹具会话上列出、分支和恢复检查点。
- 重启恢复：针对受控验证会话或隔离测试实例运行安全恢复标记路径；仅当目标会话集明确且操作可逆时，实时模式才能运行此步骤。
- 清理生命周期：删除或重置验证会话，然后验证 SQLite 生命周期行和已归档的转录状态。

无法在实时操作员 Gateway 网关上安全触发的特定传输接缝（如 WhatsApp 或语音通话入口）应针对相同的 SQLite 契约使用所有者级运行时探针，而不是伪造外部传输。

## 每步断言

每个步骤都会对操作前后的状态拍摄快照，并写入结构化断言记录：

- SQLite 行计数仅在预期位置增加。
- 对于记录运行时事件且由标记支持的验证会话，轨迹运行时行会增加。
- 验证会话行具有预期的 `session_id`、状态、时间戳、元数据和路由行。
- Gateway 网关历史记录/会话投影与 SQLite 转录末尾一致。
- 不会创建或修改验证会话 JSONL 文件。
- 不会创建验证会话的 `.trajectory.jsonl`、`.trajectory-path.json` 或
  由标记派生的 `trajectory/<session>.jsonl` sidecar。
- 现有旧版 JSONL 文件和 `sessions.json` 保持不变，除非该步骤明确属于离线迁移或归档操作。
- Gateway 网关进程不会打开 `.jsonl` 或 `sessions.json` 句柄。
- 自上一个游标以来的日志不包含 `ERROR`、`FATAL`、`SQLITE_`、
  `no such column`、会话存储不可用、重启恢复失败或转录协调警告，除非场景明确将其列入允许列表。

日志扫描是通过/失败契约的一部分。即使 Gateway 网关能响应健康检查，只要它发出 SQLite schema 错误或反复出现转录协调失败，在 Path 3 中就不能视为通过。

## 证据工件

harness 应将证据写入 `.artifacts/path3-live-e2e/<timestamp>/`
并确保其不进入 git：

- `summary.json`：命令参数、Gateway 网关版本、结果、失败的断言和工件路径。
- `sqlite-before.json` 和 `sqlite-after.json`：行计数和选定的验证行。
- `legacy-files.json`：旧版文件存在状态、`mtime`、大小以及每个文件是否发生更改。
- `gateway-log-scan.json`：游标范围、匹配的日志行和允许列表决策。
- `events.jsonl`：按顺序排列的每步观察结果，适合用于 PR 验证评论。

PR 验证应总结这些工件，而不是粘贴完整转录或私密消息内容。

## 安全规则

- 实时模式绝不能在 Gateway 网关运行时重新导入旧版 JSONL。
- 除明确选择且可逆的修复探针外，实时模式不得修改非验证会话。
- 任何破坏性或大范围迁移步骤都需要对受影响的 SQLite DB 和旧版会话目录进行全新备份。
- 备份范围应限于涉及的 Agent DB/会话目录，并在一次验证运行期间复用，以避免磁盘占用无限增长。
- 除非调用方传入 `--keep-artifacts`，否则清理步骤结束后不得遗留验证会话、验证 JSONL 或被修改的旧版文件。

## 通过结果

实时运行通过意味着 Gateway 网关接受了真实的 Agent 驱动会话流程，观察到的所有规范状态均位于 SQLite 中，旧版运行时文件保持静止，并且在测量窗口内日志健康状况保持正常。这并不意味着实时流量产生后旧版 JSONL 一致性仍然无偏差；一旦 SQLite 成为规范存储，就会出现预期的实时偏差。
