---
read_when:
    - 你正在实现 clawdbot-d63.2 / clawdbot-04b
    - 你正在修改 SQLite 会话保留、重置、删除或智能体删除归档相关逻辑
    - 你需要区分 SQLite 时代的工件族与旧版 JSONL 边车文件
summary: 归档属于某个会话的所有 SQLite 转录工件的路径 3 计划
title: Path 3 SQLite 会话工件族
x-i18n:
    generated_at: "2026-07-26T05:52:31Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 29f4d541b2df5a06468fd0cee620b4340ee33eea1064f7d3ee823580c7b5760e
    source_path: plan/path3-sqlite-session-artifact-family.md
    workflow: 16
---

# Path 3 SQLite 会话工件族

本文档界定了 `clawdbot-d63.2` 的范围，而 `clawdbot-d63.1` 负责
`src/config/sessions/session-accessor.sqlite.ts` 中与之重叠的重置/删除归档辅助函数。
本轮处理期间，实现文件存在未提交更改，因此本文档记录了
确切的契约和补丁位置，以避免与同级工作线程发生冲突。

## 权威工件族

切换到 SQLite 后，活动会话转录记录是 SQLite 行。一个会话的
归档工件族包括：

- 条目当前 `sessionId` 对应的 `transcript_events`、`transcript_event_identities` 和 `sessions` 行。
- `entry.compactionCheckpoints[*].preCompaction.sessionId` 引用的每个 `sessionId` 对应的同一组 SQLite 转录记录行。
- `entry.compactionCheckpoints[*].postCompaction.sessionId` 引用的每个 `sessionId` 对应的同一组 SQLite 转录记录行。
- `entry.usageFamilySessionIds` 中每个 `sessionId` 对应的同一组 SQLite 转录记录行。

仅归档不再被任何剩余 `session_entries` 行或任何剩余条目的压缩或用量工件族
元数据引用的行。这会保留检查点分支/恢复和用量汇总状态，直到
最后一个活动引用消失。

## 切换后的非工件族工件

生成的主题转录文件变体和轨迹旁车文件并非活动的
SQLite 运行时状态。它们是旧版文件工件：

- `<sessionId>-topic-<thread>.jsonl` 等主题变体仅存在于
  基于文件的转录记录格式中。SQLite 使用规范会话 ID 加上
  `session_routes`/条目投递元数据，而不是按主题拆分的 JSONL 文件。
- `.trajectory.jsonl` 和 `.trajectory-path.json` 等轨迹旁车文件
  根据真实的 JSONL `sessionFile` 路径命名。SQLite `sessionFile` 值是
  `sqlite:<agentId>:<sessionId>:<storePath>` 标记，并不指向旁车
  文件。
- 归档层读取器必须继续读取旧版归档 JSONL 文件，但
  运行时保留逻辑不得扫描活动会话目录，也不得为 SQLite 会话重新打开 JSONL
  转录文件。

Doctor 导入仍是旧版主 JSONL 文件及其相邻轨迹旁车文件的迁移所有者。
SQLite 运行时保留逻辑不应添加第二个导入器或文件回退路径。

## 补丁位置

扩展 `clawdbot-d63.1` 引入的 SQLite 归档辅助函数，而不是
添加并行路径。

1. 在 `deleteSqliteSessionStateIfUnreferenced` 附近添加一个本地收集器：
   - `collectSqliteSessionArtifactFamily(entry: SessionEntry): Set<string>`
   - 包括 `entry.sessionId`、检查点压缩前/后的会话 ID，以及
     `usageFamilySessionIds`。
   - 过滤空字符串，并以确定性方式去重。

2. 为移除后的存储添加引用收集器：
   - `readReferencedSqliteSessionArtifactFamilyIds(database): Set<string>`
   - 遍历当前 `session_entries`，解析每个 `entry_json`，并从
     每个保留的条目中收集相同的工件族 ID。

3. 修改当前仅归档一个已移除 `sessionId` 的重置/删除/维护调用方，
   使其传入已移除条目的完整工件族。

4. 对于每个工件族 ID，使用调用方提供的原因（`reset` 或 `deleted`）
   归档 SQLite 转录记录行，然后仅在该工件族 ID 不存在于移除后的引用集合中时删除 `sessions` 行。

5. 继续通过现有的 SQLite 会话行清理路径集中删除转录事件。
   不要添加活动 JSONL 读取。

## 聚焦测试

在 `src/config/sessions/session-accessor.conformance.test.ts` 中添加仅针对 SQLite 的测试，
或者在 `clawdbot-d63.1` 提交后添加到同级生命周期测试中：

- 删除带有压缩前转录记录的条目时，会归档当前会话和压缩前会话，
  然后移除两组 SQLite 行。
- 删除共享同一个压缩前会话的两个条目之一时，
  在最后一个引用条目被移除之前，不归档该共享的压缩前会话。
- 删除带有 `usageFamilySessionIds` 的条目时，如果没有其他条目引用该用量工件族，
  则归档前序 SQLite 转录记录行。
- 带有 SQLite 标记的主题形态会话键不会导致读取任何生成的
  主题 JSONL 文件或查找旁车文件。

聚焦验证应使用：

```bash
node scripts/run-vitest.mjs src/config/sessions/session-accessor.conformance.test.ts
```

对于此 Codex 工作树，范围较广的 `pnpm` 检查应继续在 Crabbox/Testbox 上运行。
