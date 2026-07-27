---
summary: /plugins/sdk-channel-outbound adresine yönlendir
title: Kanal mesajı API'si
x-i18n:
    generated_at: "2026-07-26T23:34:10Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: bf0d607bd3287233cbb1fe47c15958bf57a81267ae1e37e45a1881f56e1370cb
    source_path: plugins/sdk-channel-message.md
    workflow: 16
---

Bu sayfa [Kanal giden API'si](/tr/plugins/sdk-channel-outbound) sayfasına taşındı.

`openclaw/plugin-sdk/channel-message`, eski Plugin'ler için kullanımdan kaldırılmış bir uyumluluk
alt yolu olarak kalır. Yeni kanal Plugin'leri, kullanımdan kaldırılmış alt yola
yeni yardımcılar eklemek yerine ileti yaşam döngüsü, alındı bildirimi,
kalıcı gönderim ve canlı önizleme yardımcıları için
`openclaw/plugin-sdk/channel-outbound` kullanmalıdır.

Kaldırma planı: bu diğer adları harici Plugin geçiş
dönemi boyunca koruyun; ardından çağıranlar
`channel-outbound` konumuna taşındıktan sonraki büyük SDK temizliğinde kaldırın.
