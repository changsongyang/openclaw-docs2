---
summary: 重新導向至 /plugins/sdk-channel-outbound
title: 頻道訊息 API
x-i18n:
    generated_at: "2026-07-26T08:07:22Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: bf0d607bd3287233cbb1fe47c15958bf57a81267ae1e37e45a1881f56e1370cb
    source_path: plugins/sdk-channel-message.md
    workflow: 16
---

此頁面已移至[頻道輸出 API](/zh-TW/plugins/sdk-channel-outbound)。

`openclaw/plugin-sdk/channel-message` 仍是供舊版外掛使用的已淘汰相容性
子路徑。新的頻道外掛應使用
`openclaw/plugin-sdk/channel-outbound` 來取得訊息生命週期、回執、
持久傳送及即時預覽輔助函式，而非在已淘汰的子路徑中新增輔助函式。

移除計畫：在外部外掛遷移期間保留這些別名，
待呼叫端移至 `channel-outbound` 後，再於下一次 SDK 主要清理中
將其移除。
