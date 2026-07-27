---
summary: تغییر مسیر به /plugins/sdk-channel-outbound
title: API پیام کانال
x-i18n:
    generated_at: "2026-07-27T15:45:44Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: bf0d607bd3287233cbb1fe47c15958bf57a81267ae1e37e45a1881f56e1370cb
    source_path: plugins/sdk-channel-message.md
    workflow: 16
---

این صفحه به [API خروجی کانال](/fa/plugins/sdk-channel-outbound) منتقل شده است.

`openclaw/plugin-sdk/channel-message` همچنان یک زیرمسیر سازگاری منسوخ‌شده
برای Pluginهای قدیمی‌تر است. Pluginهای جدید کانال باید برای چرخهٔ عمر پیام، رسید،
ارسال پایدار و ابزارهای پیش‌نمایش زنده، به‌جای افزودن ابزارهای جدید به
زیرمسیر منسوخ‌شده، از `openclaw/plugin-sdk/channel-outbound` استفاده کنند.

برنامهٔ حذف: این نام‌های مستعار در طول بازهٔ مهاجرت Pluginهای خارجی
حفظ شوند و سپس، پس از انتقال فراخوان‌ها به `channel-outbound`، در پاک‌سازی عمدهٔ بعدی SDK
حذف شوند.
