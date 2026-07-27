---
x-i18n:
    generated_at: "2026-07-26T05:38:31Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: a8712b1aeb2e605055c22cf308049e5e74fdf33061870026be20bd55cb0c3d1d
    source_path: AGENTS.md
    workflow: 16
---

# 文档指南

此目录负责文档编写、Mintlify 链接规则和文档国际化策略。

## Mintlify 规则

- 文档托管在 Mintlify 上（`https://docs.openclaw.ai`）。
- `docs/**/*.md` 中的内部文档链接必须保持为根目录相对路径，且不得带有 `.md` 或 `.mdx` 后缀（示例：`[Config](/gateway/configuration)`）。
- 章节交叉引用应使用根目录相对路径上的锚点（示例：`[Hooks](/gateway/configuration-reference#hooks)`）。
- 文档标题应避免使用长破折号和撇号，因为 Mintlify 的锚点生成对这些字符的处理不稳定。
- README 和其他由 GitHub 渲染的文档应保留绝对文档 URL，以便链接在 Mintlify 之外也能正常工作。
- 文档内容必须保持通用：不得包含个人设备名称、主机名或本地路径；请使用类似 `user@gateway-host` 的占位符。

## 文档内容规则

- 对于文档、UI 文案和选择器列表，除非相应章节明确描述的是运行时顺序或自动检测顺序，否则服务和提供商应按字母顺序排列。
- 内置插件的命名应与根目录 `AGENTS.md` 中适用于整个仓库的插件术语规则保持一致。
- 以下文档由工具生成，切勿手动编辑：`docs/plugins/reference/**`、`docs/plugins/reference.md` 和 `docs/plugins/plugin-inventory.md` 由 `pnpm plugins:inventory:gen` 生成；`docs/docs_map.md` 由 `pnpm docs:map:gen` 生成；`docs/maturity/**` 由 `pnpm maturity:render` 生成。

## 内部文档

- 长期维护的私有运维文档应放在 `~/Projects/manager/docs/` 中。
- 仓库本地的内部临时或镜像文档可放在被忽略的 `docs/internal/` 下。
- 绝不要将 `docs/internal/**` 页面添加到 `docs/docs.json` 导航中，也不要从公开文档链接到这些页面。
- 如果之后强制添加了某个页面，`scripts/docs-sync-publish.mjs` 会从公开的 `openclaw/docs` 发布仓库中排除并清理 `docs/internal/**`。
- 内部文档可以提及仓库路径、私有应用名称、1Password 项目名称和运行手册，但绝不能包含机密值。

## 成熟度评分卡编辑

`taxonomy.yaml` 和 `qa/maturity-scores.yaml` 是源输入；`docs/maturity/` 下生成的成熟度文档是投影视图，不应为修改评分、LTS、分类法、QA 配置或证据表而手动编辑。
`scripts/qa/render-maturity-docs.ts` 负责生成；使用 `pnpm maturity:render` 刷新已提交的文档，并使用 `pnpm maturity:check` 验证这些文档。
`.github/workflows/maturity-scorecard.yml` 会渲染工件预览，并可创建生成文档的 PR；`.github/workflows/openclaw-release-checks.yml` 会为发布 QA 调度它。
除非维护者明确要求提交经过清理的投影视图，否则应将确定性的 `qa-evidence.json.scorecard` 数据保留在 GitHub Actions 工件中。
人工覆盖必须通过 PR 修改源状态，并说明原因以及公开或经过脱敏的证据。

## 文档国际化

- 此仓库不维护外语文档。生成的发布输出位于独立的 `openclaw/docs` 仓库中（本地通常克隆为 `../openclaw-docs`）。
- 不要在此处的 `docs/<locale>/**` 下添加或编辑本地化文档。
- 以此仓库中的英文文档及术语表文件为事实来源。
- 流程：在此处更新英文文档，根据需要更新 `docs/.i18n/glossary.<locale>.json`，然后让发布仓库同步并在 `openclaw/docs` 中运行 `scripts/docs-i18n`。
- 重新运行 `scripts/docs-i18n` 前，请为所有必须保留英文或采用固定译法的新技术术语、页面标题或简短导航标签添加术语表条目。
- `pnpm docs:check-i18n-glossary` 用于检查已更改的英文文档标题和简短内部文档标签。
- 翻译记忆库存储在发布仓库中生成的 `docs/.i18n/*.tm.jsonl` 文件内。
- 请参阅 `docs/.i18n/README.md`。
