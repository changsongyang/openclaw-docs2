---
summary: 重定向到 /plugins/sdk-channel-outbound
title: 频道消息 API
x-i18n:
    generated_at: "2026-07-26T06:27:59Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: bf0d607bd3287233cbb1fe47c15958bf57a81267ae1e37e45a1881f56e1370cb
    source_path: plugins/sdk-channel-message.md
    workflow: 16
---

此页面已移至[渠道出站 API](/zh-CN/plugins/sdk-channel-outbound)。

`openclaw/plugin-sdk/channel-message` 仍作为面向旧版插件的已弃用兼容
子路径保留。新的渠道插件应使用
`openclaw/plugin-sdk/channel-outbound` 提供消息生命周期、回执、
持久发送和实时预览辅助函数，而不是向已弃用的
子路径添加新的辅助函数。

移除计划：在外部插件迁移
窗口期间保留这些别名；调用方迁移至
`channel-outbound` 后，在下一次主要 SDK 清理中将其移除。
