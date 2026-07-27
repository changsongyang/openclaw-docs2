---
read_when: You want agent sessions to run on ephemeral cloud machines instead of the Gateway host, or you are configuring cloudWorkers profiles.
sidebarTitle: Cloud Workers
status: active
summary: 将会话分派到一次性云端机器：资源配置、工作节点运行时、代理推理和流式返回结果
title: 云端工作节点
x-i18n:
    generated_at: "2026-07-26T06:41:48Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 5620be5957a20019d4687b3ec935ec1116fdef6ea05e42ab766508d2b54322a2
    source_path: gateway/cloud-workers.md
    workflow: 16
---

云端工作节点允许会话在一次性云端机器上运行其智能体循环，而会话的一切仍保留在原处：显示在侧边栏中、实时流式传输，并且对话记录由 Gateway 网关所有。Gateway 网关租用一台机器，在其上安装固定版本的 OpenClaw，将会话的工作区同步过去，然后把轮次循环交给受限的 `openclaw worker` 进程。模型调用通过 Gateway 网关代理返回，因此提供商凭据绝不会离开你的机器；由于提供商看到的是一个连续的数据流，提示词缓存也会继续工作。

工作完成后（或机器发生故障时），该机器会被丢弃。持久状态——对话记录、工作区提交、放置记录——保留在 Gateway 网关中。

<Note>
云端工作节点为选择启用功能，在配置配置文件之前不可见。未配置的安装不会看到任何新的 RPC、配置或 UI。
</Note>

## 各部分在哪里运行

| 关注项                                                  | 位置                                                                             |
| ------------------------------------------------------- | -------------------------------------------------------------------------------- |
| 智能体循环 + 工具（`exec`、`read`、`write`、`edit`、……） | 云端工作节点机器                                                                 |
| 模型推理和提供商凭据                                    | Gateway 网关（通过 `{provider, model}` 引用代理）                                 |
| 对话记录（持久，会话存储）                              | Gateway 网关                                                                     |
| 实时流式传输到侧边栏                                    | Gateway 网关扇出，由工作节点的可重放事件流提供数据                               |
| 工作区 Git 历史                                         | 在机器上无凭据创建；Gateway 网关接管提交并负责推送/PR                            |

除 `sshd` 外，该机器不需要任何入站端口：Gateway 网关通过固定的 SSH 主动向外连接，反向隧道则将工作节点的 WebSocket 传回。内置的 Crabbox 提供商强制使用公共 SSH 路由，并禁用托管式 Tailscale 注册。出站互联网访问由提供商策略决定；除非限制其网络或安全组，否则默认 AWS 配置文件可以访问互联网。

## 要求

- 一个工作节点提供商插件。内置的 `crabbox` 插件驱动 [Crabbox](https://github.com/openclaw/crabbox) CLI，该 CLI 负责在各云后端（AWS、Hetzner 等）之间代理租约。`crabbox` 二进制文件必须位于 `PATH` 中（或设置 `settings.binary`），且提供商凭据必须已配置。AWS 准入要求 Crabbox 0.38.1 或更高版本。
- 对于 Crabbox AWS 工作节点，生效的 `aws.instanceProfile` 必须为空。提供商在分配前检查 `crabbox config show --json`，随后要求 `crabbox inspect --json` 报告来自 EC2 `DescribeInstances` 的 `providerMetadata.instanceProfileAttached: false`。具有实例角色或缺少权威元数据的租约会被停止并拒绝。
- 租用机器上需要安装 Node.js。裸云镜像通常不包含它——请在配置文件的 `setup` 命令中安装。
- 一个具有会话自有托管工作树的会话（使用 `worktree: true` 创建）。分派会移动该工作树的内容；普通目录则作为清单镜像同步。

## 配置

在 `openclaw.json` 的 `cloudWorkers.profiles` 下添加配置文件：

```json
{
  "cloudWorkers": {
    "profiles": {
      "aws": {
        "provider": "crabbox",
        "install": "bundle",
        "settings": {
          "provider": "aws",
          "class": "standard",
          "ttl": "8h",
          "idleTimeout": "45m",
          "setup": "test -x /usr/bin/node || (curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash - && sudo apt-get install -y nodejs)"
        }
      }
    }
  }
}
```

配置文件字段：

| 键         | 含义                                                                                                                                                                                                                                           |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `provider` | 由插件注册的工作节点提供商 ID（内置插件使用 `crabbox`）。                                                                                                                                                                     |
| `install`  | `bundle`（默认）交付当前运行中 Gateway 网关的构建；`npm` 使用固定的完整性值安装与 Gateway 网关完全相同的已发布版本。`npm` 要求 Gateway 网关从打包发布版本运行。                                               |
| `settings` | 提供商所有的 JSON。对于 crabbox：`provider`（后端）、`class`（机器类别）、`ttl`、`idleTimeout`（Go 时长），以及可选的 `setup` 和绝对 `binary` 路径。OpenClaw 强制这些租约使用公共 SSH，并禁用托管式 Tailscale。 |
| `lifetime` | 可选的已存储策略（`idleTimeoutMinutes`、`maxLifetimeMinutes`）。                                                                                                                                                                            |

### 设置命令

`settings.setup` 在租用机器可通过 SSH 访问后、安装 OpenClaw 前运行。它会在**每次**预配尝试中运行（包括分派中断后的重放），因此必须具有幂等性——请像示例中那样使用 `command -v`/`test -x` 检查来保护安装操作。如果设置失败，提供商会停止租约，分派以安全关闭方式失败；不会留下仍在运行但只完成部分配置的机器。

### 安装渠道

- **`bundle`** 会打包当前运行中 Gateway 网关的 `dist`、经过精简的 `package.json`，以及构建所引用的所有工作区软件包，并以内容哈希覆盖所有这些内容。机器会根据该哈希验证原始软件包，然后安装生产环境 npm 依赖项（禁用脚本）。这是在工作节点上运行开发构建的方式。
- **`npm`** 会证明该发布版本存在于公共注册表中，固定其 SHA-512 完整性值，并安装与 Gateway 网关完全匹配的 `openclaw@<version>`。

## 分派会话

在 Control UI 中，打开 **New Session**，选择已配置运行时为 OpenClaw 的智能体，从 **Where** 菜单中选择已配置的 **Cloud · profile** 目标，然后启动任务。选择云端目标会自动启用所需的托管工作树；Gateway 网关创建会话并完成分派，之后才发送第一个轮次。会话侧边栏中的服务器徽章会显示持久放置状态。外部 CLI 会话目录不会提供云端目标。

等效的 RPC 流程如下：

创建具有托管工作树的会话，然后分派它（该 RPC 要求 `operator.admin`，且仅在已配置配置文件时存在）：

云端工作节点运行 OpenClaw 智能体运行时。请选择 `openai/*` 或其他可解析到该运行时的模型；配置为 `claude-cli` 等外部 CLI 运行时的会话无法分派。

```bash
openclaw gateway call sessions.create \
  --params '{"key":"agent:main:big-refactor","worktree":true,"cwd":"/path/to/repo","worktreeName":"big-refactor"}'

openclaw gateway call sessions.dispatch \
  --timeout 1500000 \
  --params '{"key":"agent:main:big-refactor","profileId":"aws"}'
```

`sessions.dispatch` 会关闭本地轮次准入、排空正在进行的工作、预配租约、运行设置、引导 OpenClaw、同步工作区，并在放置达到 `active` 工作节点所有权后返回。首次分派请预留几分钟；如果提供商支持，租约和安装会被缓存。此后，可像平常一样与会话交互——轮次会自动路由到工作节点。

已完成的工作节点轮次会在释放轮次声明前，将符合条件且大小受限的工作区文件协调回会话的托管工作树。终止工作节点事件会在得到确认前创建持久的待处理结果栅栏。随后，Gateway 网关会先将完整云端结果暂存为 `refs/openclaw/worker-results/` 下的 Git 引用，再应用该结果，因此即使 Gateway 网关在应用期间停止，云端版本仍可恢复。工作区结果使用 Git 文件语义：保留常规文件、可执行位、符号链接、新增、修改和删除，但不保留空目录及其他目录模式。产生的文件更改会留在托管工作树中，以便正常审查和提交。

应用操作使用分派时的清单作为合并基础。仅云端的更改会被应用，仅本地的更改会保留，双方都发生更改的路径则使用三方保留本地策略。即使存在冲突，轮次仍会完成：对话记录会报告受限的路径摘要和暂存结果引用，放置状态会向 Control UI 暴露同一冲突，且无冲突的云端更改仍会应用。通知中包含 `git show <ref>:<path>`，用于检查存在的云端文件，以及顶层字面路径规范 `git checkout <ref> -- <path>` 命令，用于从任意工作区目录获取该文件。请在 Bash 或 zsh（Windows 上使用 Git Bash）中运行这些命令。如果检查结果表示路径不存在，则云端结果已删除该路径；请验证后手动删除保留的本地路径。如果签出操作报告文件/目录阻碍，请移动或删除造成阻碍的本地路径，然后重试。如果暂存引用本身已消失，请将该通知视为过期通知，不要更改本地路径。发生冲突的暂存引用会在正常轮次栅栏释放后继续保留；后续无冲突的结果会清除通知并停用旧引用，而显式移除栅栏则是最终清理边界。

当带栅栏的结果仍在协调时，新轮次最多等待 15 秒，直到先前的声明释放。如果届时仍忙碌，该轮次会失败，并显示可操作的“上一个云端轮次的工作区结果仍在协调”消息，稍后可以重试。重启时，恢复过程会在清理过期声明前发现待处理和暂存结果，完成或重试其本地应用，并且仅在保留结果后回收已失效的环境。受限的 SQLite 回滚日志可使中断的文件系统应用恢复，而无需重放已经接受的变更。

工作完成且没有正在运行的轮次时，打开会话菜单并选择 **Stop cloud worker…**。Gateway 网关会在销毁环境前执行最后一次工作区协调。已处于 `draining` 或 `reconciling` 状态的放置正在完成拆除；请等待其徽章变为 `reclaimed`，再删除会话。

对于故障或失控但仍处于附加状态的工作节点，操作员可将 `environments.destroy` 与 `{ "force": true }` 配合调用，作为最后手段。强制拆除会持久地将放置标记为失败，并在销毁环境前放弃任何尚未协调的远程结果。

等效的管理 RPC 如下：

```bash
openclaw gateway call sessions.reclaim \
  --timeout 600000 \
  --params '{"key":"agent:main:big-refactor"}'
```

放置过程通过一个持久状态机（`local → requested → provisioning → syncing → starting → active`）推进，因此在分派过程中重启 Gateway 网关时，系统会执行协调，而不会造成机器泄漏。模型轮次失败后，活跃放置仍可用于重试。工作区路径冲突会保留本地版本、应用云端结果的其余部分，并保留已暂存的云端 ref 以供检查；其他协调或生命周期故障会保留其持久恢复栅栏和诊断输出尾部，直到恢复流程能够安全地重试或回收环境。

## 安全模型

- **封闭的工作节点入口。** 工作节点通过隧道套接字使用专用协议通信，并受封闭的方法允许列表限制——工作节点无法调用操作员 RPC。
- **Gateway 网关拥有的工具权限。** 每个轮次开始前，Gateway 网关都会针对工作节点固定的编码工具目录，应用当前的配置文件、提供商、智能体、群组、发送者、沙箱、委托、继承及运行时上限策略。启动信封仅携带由最终封闭词汇表限定的子集。明确设置上限的定时轮次会复用其受信任的所有者群组上下文，而不会将该身份发送到机器，也不会重新应用新的发送者覆盖层。工作节点目录之外的工具仍不可用；如果结果为空，则不使用任何工具运行。
- **凭据按需签发，静态存储时采用哈希。** 每次分派都会签发一个工作节点凭据；Gateway 网关仅存储其哈希。凭据轮换和所有者 epoch 栅栏可确保每个会话最多只有一个活跃所有者——过期工作节点重新连接时会被隔离，绝不会合并。
- **主机密钥固定。** 提供商必须在预配时公开机器的 SSH 主机密钥；引导程序使用严格固定的密钥连接，如果没有该密钥则以关闭方式失败。
- **机器上不存放常驻的模型、代码托管平台或云凭据。** 模型身份验证保留在 Gateway 网关上（推理通过 `{provider, model}` 引用传输），工作区 git 提交不使用代码托管平台凭据，且在设置前会以权威方式检查 Crabbox AWS 租约元数据中是否存在实例角色。设置命令也必须不含凭据。
- **由提供商控制出站流量。** 反向隧道消除了 OpenClaw 直接访问模型的需要，但 OpenClaw 不会重写提供商防火墙。如果任务需要，请在工作节点提供商中限制出站流量。
- **持久且恰好一次的转录记录。** 工作节点通过针对会话叶节点的比较并交换协议提交转录批次；过期的基准会让运行立即停止，而不会复制或变基已付费的输出。

## 故障排除

- **`sessions.dispatch` 是未知方法**——未配置 `cloudWorkers.profiles`，或调用方缺少 `operator.admin`。
- **“云端工作节点轮次需要 OpenClaw 运行时”**——选择已将运行时配置为 OpenClaw 的模型。`claude-cli` 等外部 CLI 运行时不支持工作节点推理。
- **“工作节点引导需要租用主机安装 Node.js”**——在 `settings.setup` 中添加 Node 安装步骤（见上文）。
- **AWS 实例角色证明失败**——清除 `aws.instanceProfile`（如果设置了 `CRABBOX_AWS_INSTANCE_PROFILE`，也将其清除）。安装 Crabbox 0.38.1 或更高版本；旧版二进制文件不会公开 AWS 准入所需的权威 `providerMetadata.instanceProfileAttached` 合约。
- **分派因提供商错误而失败**——放置记录和 `environments.list` 会保留最后一个错误，包括设置/引导程序的 stderr 输出尾部。机器在失败时会被销毁，因此该输出尾部是主要的取证依据。
- **分派期间客户端超时**——`openclaw gateway call` 默认超时时间为 10s；请为 `--timeout` 传入充足的值（无论如何，分派都会继续在服务器端运行；预配期间重试会被拒绝并返回 `session cannot dispatch from placement provisioning`）。
- **从 2026.7.2 beta 升级后工作节点被回收**——这些 beta 版本使用了旧版工作节点启动合约。重启时，OpenClaw 会销毁空闲且不兼容的工作节点，保留会话和工作区，将放置标记为已回收，并在下一次分派或轮次中预配当前版本的工作节点。如果 beta 工作节点在仍处于启动状态时被中断，则会在清理后标记为失败；请重试分派，以使用当前合约进行预配。
- **云端工作区冲突通知**——轮次已完成，并保留了所列各路径的本地版本。使用通知中的已暂存 ref 命令检查或采用云端版本；无需为无冲突的更改重试，因为这些更改已经应用。
- **“上一个云端轮次的工作区结果仍在协调中”**——Gateway 网关短暂等待了先前结果的持久栅栏，但无法取得会话声明。等待协调完成，然后重试该轮次；重启 Gateway 网关是安全的，因为恢复流程会先保留已暂存的结果，再回收已失效的工作节点。
- **租约日常维护**——`crabbox list --provider <backend>` 显示活跃租约；`crabbox stop --provider <backend> --id <lease>` 可手动释放一个租约。空闲租约会在配置文件的 `idleTimeout` 到期。

## 相关内容

- [沙箱隔离](/zh-CN/gateway/sandboxing)——缩小本地工具执行的影响范围
- [会话 CLI](/zh-CN/cli/sessions)——检查已存储的会话
- [配置参考](/zh-CN/gateway/configuration-reference)
