---
read_when:
    - exec veya plugin onayı yaşam döngüsünü, depolamayı, protokolü ya da yetkilendirmeyi değiştirme
    - Bir kanala onay bağlantıları veya yerel onay denetimleri ekleme
    - Alt oturum onaylarını üst veya orkestratör görünümlerine yansıtma
summary: Control UI, yerel uygulamalar, kanallar ve üst oturumlar genelinde kalıcı, doğrudan bağlantı kurulabilir onaylar için tasarım
title: Çoklu yüzey operatör onayları
x-i18n:
    generated_at: "2026-07-26T23:38:30Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 9defdaada1911df1184f64429e1787c4881e735c433d6dbc30a5946e11cc7cce
    source_path: refactor/operator-approvals.md
    workflow: 16
---

# Çok yüzeyli operatör onayları

Bu tasarım [#103505](https://github.com/openclaw/openclaw/issues/103505) sorununu izler. Süreç yerelindeki onay yetkisinin yerine Gateway'in sahip olduğu, SQLite destekli tek bir yaşam döngüsü getirir. Gateway'in sahip olduğu her exec veya plugin/araç onayı; tek bir kararlı kimlik, kimliği doğrulanmış tek bir Control UI rotası, atomik olarak ilk yanıtın kazandığı çözümleme ve kaynak ile üst oturum akışlarına yalnızca operatörlere yönelik izdüşümler alır.

Satır içi eylemler ve derin bağlantılar birlikte bulunur. Onay modu anahtarı yoktur.

## Hedefler

- Exec ve plugin/araç geçitleri için tek bir kalıcı onay nesnesi.
- Kararlı `${controlUiBasePath}/approve/{approvalId}` rotası.
- Yetkili herhangi bir Control UI, yerel uygulama veya kanal yüzeyinden çözümleme.
- Eşzamanlı yüzeyler genelinde atomik olarak ilk yanıtın kazanması davranışı.
- Aynı yeniden denemeler idempotenttir; çakışan geç yanıtlar kazanan yanıtın üzerine yazamaz.
- Zaman aşımı, hatalı biçimlendirilmiş güvenilir kararlar, eksik rotalar, iptal ve yeniden başlatma güvenli biçimde reddeder.
- İstek ve terminal olayları kaynak oturuma ve ilgili tüm üst/orkestratör sahiplerine ulaşır.
- Kanallar türü belirlenmiş onay ve gezinme eylemleri alır; taşıma geri çağırma verileri kanala özel kalır.
- Mevcut exec/plugin Gateway yöntemleri, uygulamaları tek bir hizmet üzerinde birleşirken uyumlu kalır.

## Hedef olmayanlar

- Engellenen araç yürütmesinin kendisini Gateway yeniden başlatmaları arasında kalıcı hâle getirmek veya sürdürmek.
- Bir onay kimliğini veya URL'yi taşıyıcı kimlik bilgisi hâline getirmek.
- Onay istemlerini modelin görebildiği dökümlere eklemek veya üst ajanları uyandırmak.
- Onay politikasını, ürün komutlarını veya inceleyici yetkilendirmesini kanal pluginlerine taşımak.
- Onay durumunu kanal, cihaz veya üst öğe başına kopyalamak.
- Terminal sonuçlarını kesinleştirmek için gereken durumlar dışında exec izin listelerini, plugin politikası bileşimini veya `allow-always` kalıcılığını yeniden tasarlamak.
- Gateway'siz gömülü bir TUI'yi ilk artırımda uzaktan erişilebilir hâle getirmek. Yalnızca yerel kalır ve inceleyici bulunmadığında güvenli biçimde reddetmelidir.

## Dağıtım öncesi temel durum ve kanıt haritası

Bu tablo, #103505 açıldığındaki uygulama durumunu kaydeder. Aşağıdaki dağıtım bölümleri, bu temel durumun üzerine inşa edilen kalıcı kayıt defterini, türü belirlenmiş eylemleri, derin bağlantı sayfasını ve yerel istemci artırımlarını izler.

| Yüzey           | Temel giriş noktası ve sahibi                                                                                                                                  | Temel davranış ve eksiklik                                                                                                                                                                    |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Ajan exec        | `src/agents/bash-tools.exec-approval-request.ts`, `src/agents/bash-tools.exec-host-shared.ts`                                                                   | İki aşamalı `exec.approval.*` kaydı, erken bir `/approve` yarışını önler; ancak zaman aşımı `askFallback` aracılığıyla yine de izne dönüşebilir.                                                        |
| Plugin araç geçidi  | `src/agents/agent-tools.before-tool-call.ts`                                                                                                                    | `plugin.approval.*` ister; `timeoutBehavior: "allow"` zaman aşımına uğramış bir geçidi onaylayabilir. Gömülü mod, `src/infra/embedded-plugin-approval-broker.ts` içinde süreç yerelinde ayrı bir yetkiye sahiptir. |
| Plugin node geçidi  | `src/gateway/node-invoke-plugin-policy.ts`                                                                                                                      | Doğrudan plugin yöneticisi üzerinden oluşturur ve yayınlar; sunucu yöntemi yaşam döngüsünün bir bölümünü yineler.                                                                                 |
| Gateway yetkisi | `src/gateway/server-aux-handlers.ts`, `src/gateway/exec-approval-manager.ts`, `src/gateway/server-methods/approval-shared.ts`                                   | Ayrı exec ve plugin yöneticileri süreç yerelindeki eşlemeleri kullanır. Terminal girdileri 15 saniye boyunca kalır. İlk yanıtın kazanması yalnızca tek bir süreç içinde geçerlidir.                                          |
| Gateway protokolü  | `packages/gateway-protocol/src/schema/exec-approvals.ts`, `packages/gateway-protocol/src/schema/plugin-approvals.ts`, `src/gateway/methods/core-descriptors.ts` | Exec yalnızca bekleyenleri içeren `get` öğesine sahiptir; plugin `get` öğesine sahip değildir; derin bağlantı için türden bağımsız terminal araması yoktur.                                                                                   |
| Teslimat          | `src/infra/exec-approval-channel-runtime.ts`, `src/infra/approval-native-runtime.ts`, `src/infra/approval-handler-runtime.ts`                                   | Kaynak yönlendirmeyi, onaylayanlara gönderilen DM'leri, bekleyenlerin yeniden yürütülmesini, yerel işleyicileri ve süreç içi terminal temizliğini destekler. Ayrı bir takip çalışması kalıcı terminal uzlaştırması ekler.                          |
| Taşınabilir eylemler  | `src/interactive/payload.ts`, `src/plugin-sdk/interactive-runtime.ts`, `src/plugin-sdk/approval-reply-runtime.ts`                                               | Onay düğmeleri, `/approve ...` içeren komut eylemleridir; URL ve Web App hedefleri türü belirlenmemiş düğme alanlarıdır.                                                                           |
| Telegram          | `extensions/telegram/src/approval-handler.runtime.ts`, `extensions/telegram/src/button-types.ts`                                                                | Oluşturucu, özel geri çağırma verileri üretmeden önce onay semantiğini tanımak için komut metnini ayrıştırır.                                                                                     |
| Control UI        | `ui/src/app/exec-approval.ts`, `ui/src/app/overlays.ts`, `ui/src/components/exec-approval.ts`                                                                   | Onay kullanıcı arayüzü genel bir kipli penceredir. `ui/src/app-route-paths.ts` ve `ui/src/app-routes.ts` tam rotaları kullanır ve bilinmeyen yolları Chat'e yeniden yazar.                                                    |
| Oturum sahipliği | `src/agents/subagent-registry.types.ts`, `src/agents/subagent-registry-read.ts`, `src/config/sessions/types.ts`                                                 | Denetleyici, istekte bulunan, açık üst öğe ve eski oluşturma sahipliği mevcuttur; ancak onay olayları bu oturum akışlarına yansıtılmaz.                                                    |
| Paylaşılan durum      | `src/state/openclaw-state-schema.sql`, `src/state/openclaw-state-db.ts`                                                                                         | Mevcut anlık işlemler ve Kysely koşullu güncellemeleri, `state/openclaw.sqlite` içinde kalıcı karşılaştırma-ve-ayarlamayı destekler.                                                                   |

Güncel temsili testler arasında `src/gateway/exec-approval-manager.test.ts`, `src/gateway/server-methods/approval-shared.test.ts`, `src/agents/bash-tools.exec-gateway-approval.e2e.test.ts`, `extensions/telegram/src/approval-handler.runtime.test.ts` ve `ui/src/e2e/approval-flow.e2e.test.ts` bulunur.

Plugin SDK, tek kanal/plugin sınırı olmayı sürdürür. Onay çalışma zamanı ve sunum değişiklikleri mevcut `src/plugin-sdk/approval-*.ts` ve `src/plugin-sdk/interactive-runtime.ts` alt yolları üzerinden dışa aktarılmalıdır; plugin üretim kodu Gateway iç bileşenlerini içe aktarmamalıdır.

## Önceki çalışmalar

Omnigent, yararlı kullanıcı deneyimi ve hata semantiği sağlar:

- [`approval.py`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/omnigent/runtime/policies/approval.py), ASK'i bekletir, politika başına zaman aşımı uygular ve yalnızca tam bir kabulü onay olarak değerlendirir.
- [`sessions.py`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/omnigent/server/routes/sessions.py), sunucu tarafındaki yerel test düzeneği geçidini ve üst öğe istek/çözümleme izdüşümünü içerir.
- [`ApprovePage.tsx`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/web/src/pages/ApprovePage.tsx), bağımsız mobil onay sayfasını sağlar.

Depolama iddiasını sorgulamadan kopyalamayın. Mevcut etkin bekleme durumu [`_elicitation_registry.py`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/omnigent/server/_elicitation_registry.py) içinde süreç yerelindedir ve kullanılmayan bekleme tablosu [`e3b1f2a4c9d7_drop_pending_tool_calls_table.py`](https://github.com/omnigent-ai/omnigent/blob/46e3cd9754c3b8567f7b09f4d19b6249dabe0e80/omnigent/db/migrations/versions/e3b1f2a4c9d7_drop_pending_tool_calls_table.py) tarafından kaldırılır. OpenClaw bilinçli olarak daha ileri gider: SQLite yetkili kaynaktır ve her terminal geçişi bir veritabanı karşılaştırma-ve-ayarlama işlemidir.

## Mimari ve sahiplik

Gateway yaşam döngüsünün sahibidir:

1. Bir ajan, plugin kancası veya node politikası, türe özgü bir istek ve süreç yerelinde yürütme bağlaması sağlar.
2. Gateway bunu doğrular ve temizlenmiş bir inceleyici izdüşümü oluşturur.
3. Onay hizmeti bir kaynak/sahip kitlesi hesaplar, kurallı satırı ekler ve ardından süreç içi bekleyiciyi kaydeder.
4. Kalıcı eklemeden sonra Gateway; mevcut onay olaylarını, oturum izdüşümlerini, kanal bildirimlerini ve yerel anlık bildirimleri yayınlar.
5. Her yüzey aynı hizmet üzerinden çözümleme yapar.
6. Hizmet tek bir terminal geçişini kesinleştirir, çalışma zamanı bekleyicisini uyandırır ve terminal izdüşümlerini yayınlar.
7. Başarısız bir olay teslimatı, kesinleştirilmiş kararı hiçbir zaman geri almaz; istemciler `approval.get` veya listeyi yeniden yürütme yoluyla kurtarır.

Sahiplik sınırları:

- `src/gateway/`: onay hizmeti, yetkilendirme, RPC bağdaştırıcıları, URL oluşturma, bekleyici yaşam döngüsü ve olay yayını.
- `src/state/`: paylaşılan şema ve oluşturulan Kysely türleri.
- `src/infra/`: temizlenmiş onay görünüm modelleri ve taşınabilir sunum oluşturma.
- `src/agents/`: döndürülen kararı ister, bekler ve uygular; kalıcılık yoktur.
- `src/channels/` ve `extensions/*`: türü belirlenmiş eylemleri oluşturur, kanal kullanıcılarını yetkilendirir, özel geri çağırmaları kodlar ve teslim edilen denetimleri günceller.
- `src/plugin-sdk/`: yalnızca herkese açık onay ve sunum sözleşmeleri.
- `ui/`: bağımsız sayfa ve mevcut kuyruk/kipli pencere istemcileri.

Süreç içi bekleyici bir bildirim mekanizmasıdır, yetki değildir. Kayıt işlemi, isteği yayınlamadan önce satırı ekler ve bekleyiciyi eşzamanlı olarak kurar; böylece bir çözümleyici bu adımların arasına giremez. Sonraki her çözümleyici, bu bekleyiciyi sonuçlandırmadan önce SQLite üzerinden kesinleştirme yapar.

## Kalıcı kayıt

Paylaşılan durum veritabanına bir adet `operator_approvals` tablosu ekleyin.

| Sütun                                             | Amaç                                                                                                                                       |
| -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| `approval_id`                                      | Küresel olarak benzersiz kanonik kimlik. Protokol uyumluluğu için mevcut yürütme kimliklerini ve `plugin:` kimliklerini koruyun, ancak türü hiçbir zaman önekten çıkarsamayın.      |
| `resolution_ref`                                   | Kanonik kimliği taşıyamayan aktarım geri çağrıları için benzersiz, tam SHA-256 base64url konumlandırıcısı. Bu bir yetkilendirme veya genel URL kimliği değildir. |
| `kind`                                             | Kapalı `exec \| plugin` ayırıcısı.                                                                                                        |
| `status`                                           | Kapalı `pending \| allowed \| denied \| expired \| cancelled` durumu.                                                                          |
| `presentation_json`                                | Doğrulanmış, tür etiketi taşıyan inceleyici izdüşümü. Ham çalışma zamanı istekleri, komut bağlamaları ve geri çağrı yükleri işlem kapsamında kalır.               |
| `source_agent_id`, `source_session_key`            | Kaynak kimliği ve oturum izdüşümü sabitleyicisi. Oturum anahtarı kalıcıdır; dönüşümlü oturum UUID'si kalıcı değildir.                                          |
| `audience_session_keys_json`                       | Sınırlı genişlik öncelikli sahiplik dolaşımı tarafından üretilen sıralı, yinelenen öğeleri kaldırılmış JSON dizisi. İstek ve terminal olayları aynı anlık görüntüyü kullanır. |
| `requested_by_device_id`, `requested_by_client_id` | Kalıcı istekte bulunan/denetim meta verileri. Bağlantı kimliği bellekte kalır ve yüzeyler arası bir sorumlu değildir.                                         |
| `reviewer_device_ids_json`                         | Yalnızca güvenilir onay çalışma zamanı tarafından sağlanan, isteğe bağlı ve açıkça hedeflenmiş inceleyici cihazları.                                                  |
| `runtime_epoch`                                    | Bekletilen yürütmenin sahibi olan işlem dönemi; yeniden başlatmadan sonra sahipsiz satırları iptal etmek için kullanılır.                                                     |
| `created_at_ms`, `expires_at_ms`, `updated_at_ms`  | Yetkili zamanlama.                                                                                                                         |
| `decision`                                         | Mevcut olduğunda açık kullanıcı kararı.                                                                                                       |
| `terminal_reason`                                  | `user`, `timeout`, `malformed-verdict`, `no-route`, `run-aborted` veya `gateway-restart` gibi kapalı neden.                                |
| `resolved_at_ms`, `resolver_kind`, `resolver_id`   | Kazanan ve denetim kimliği sunucu tarafında saklanır. İnceleyici izdüşümleri ham çözümleyici tanımlayıcılarını içermez.                                           |
| `consumed_at_ms`, `consumed_by`                    | `allow-once` için ayrı yeniden oynatma koruması; tüketme işlemi kaydedilen kararı silmemelidir.                                                       |

Gerekli indeksler:

| İndeks                                      | Amaç                                                                     |
| ------------------------------------------ | --------------------------------------------------------------------------- |
| benzersiz `(resolution_ref)`                  | Ekleme sırasında sütunlar arası `approval_id`/`resolution_ref` belirsizliğini reddedin. |
| `(status, expires_at_ms)`                  | Bekleyen onayları bulun ve yetkili son tarihleri uzlaştırın.               |
| `(source_session_key, created_at_ms DESC)` | Bir kaynak oturumu için son onayları yeniden oynatın.                             |
| `(resolved_at_ms)`                         | Sabit saklama politikasına göre saklanan terminal onaylarını temizleyin.  |

Hedef kitle dizileri küçük ve sınırlıdır. Oturuma göre filtrelenmiş yeniden oynatma, önce Kysely aracılığıyla görünür bekleyen satırları seçer, ardından sınırlı hedef kitle dizilerinin kodunu uygulama kodunda çözüp bunları filtreler; dize eşleştirme veya ham SQL JSON sorguları kullanmaz.

Terminal satırlarını, `src/audit/audit-event-store.ts` içindeki meta veri denetim saklama süresiyle uyumlu olarak 30 gün saklayın. Temizleme, yeni bir yapılandırma yüzeyi değil, sabit bir bakım politikasıdır. Veritabanı özel yerel kontrol düzlemi durumudur, ancak inceleyici API'leri saklanan isteğin veya çalışma zamanı bağlamasının tamamını hiçbir zaman açığa çıkarmamalıdır.

## Durum makinesi ve karşılaştırıp ayarlama

Yalnızca şu geçişler geçerlidir:

- `pending -> allowed`: açık `allow-once` veya `allow-always`.
- `pending -> denied`: açık ret, güvenilir hatalı biçimlendirilmiş terminal kararı veya teslimat yolunun olmaması.
- `pending -> expired`: yetkili son tarihe ulaşıldı.
- `pending -> cancelled`: çalıştırmanın durdurulması, düzgün kapatma veya yeniden başlatma sonrası sahipsiz kayıt kurtarma.

İzin verilmeyen her terminal durumunun etkin kararı rettir.

Çözümleme, tek bir anlık SQLite işlemi ve şuna eşdeğer bir Kysely koşullu güncellemesi kullanır:

```sql
UPDATE operator_approvals
SET status = ?, decision = ?, terminal_reason = ?, resolved_at_ms = ?
WHERE approval_id = ?
  AND status = 'pending'
  AND expires_at_ms > ?;
```

Güncelleme hiçbir satırı etkilemezse aynı işlem kaydı okur:

- Eksik veya yetkisiz: bulunamadı sonucunu döndürün; varlığını açığa çıkarmayın.
- Hâlâ bekliyor ancak son tarihe ulaşıldı: karşılaştırıp `expired` olarak ayarlayın, ardından bu terminal satırını döndürün.
- Aynı kaydedilmiş karar: kaydedilmiş kazananla birlikte eş etkili başarı döndürün.
- Farklı karar: birleşik API, kaydedilmiş kazananla birlikte `applied: false` döndürür; eski bağdaştırıcılar, yayımlanmış sözleşmelerinin gerektirdiği yerlerde `APPROVAL_ALREADY_RESOLVED` değerini korur.
- Herhangi bir terminal durumu: hiçbir zaman değiştirmeyin.

`now == expires_at_ms` süresi dolmuştur. Gateway zamanı yetkilidir.

`allow-once` yürütmesi, mevcut tam komut/sistem çalıştırma bağlamına bağlı olarak `consumed_at_ms IS NULL` üzerinde ikinci bir CAS kullanır. Onay satırı tüketimden sonra denetim kaydı olarak kalır.

Kimliği doğrulanamayan veya bir onayı tanımlayamayan hatalı biçimlendirilmiş HTTP/RPC girdisi değişiklik yapılmadan reddedilir ve hiçbir zaman onay veremez. Bilinen bir onay için güvenilir bir düzenek/bekleyiciden alınan hatalı biçimlendirilmiş terminal kararı `denied` durumuna geçer.

## Gateway API

Türden bağımsız inceleyici yöntemleri ekleyin:

| Yöntem                                    | Sözleşme                                                                                                                                                                                                            |
| ----------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `approval.get { id }`                     | Görünür bir bekleyen veya saklanan terminal izdüşümünü döndürür.                                                                                                                                                          |
| `approval.resolve { id, kind, decision }` | Kanonik kimliği veya sabit boyutlu aktarım referansını kabul eder; ardından yetkilendirme, tür ve izin verilen karar doğrulaması, son tarih uzlaştırması ve terminal CAS işlemlerini çalıştırır. Yanıt her zaman kanonik kimliği taşır. |

Başarılı bir CAS işleminden sonra kaydedilmiş izdüşümü hemen döndürün. Eski olaylar, kanal yönlendiricileri ve anlık bildirim sonlandırıcıları azami çaba gösterilen takip işlemleridir; yavaş veya başarısız bir yüzey, kazanan yanıtı geciktirmemeli veya geri almamalıdır.

Türe özgü istek doğrulaması `exec.approval.request` ve `plugin.approval.request` içinde kalır. Mevcut `exec.approval.get/list/waitDecision/resolve` ve `plugin.approval.list/waitDecision/resolve`, yayımlanmış Gateway API oldukları için kanonik hizmete yönelik protokol sınırı bağdaştırıcılarına dönüşür. Dahili çağıranlar aynı değişiklik kapsamında hizmete geçirilir.

İnceleyici izdüşümü, etiketli bir birleşimdir:

```ts
type OperatorApproval = {
  id: string;
  status: OperatorApprovalStatus;
  presentation:
    | { kind: "exec"; commandText: string /* safe exec preview */ }
    | { kind: "plugin"; title: string; description: string /* safe plugin preview */ };
  // common lifecycle fields
};
```

Kararlı yol kalıcı olarak saklanmaz, türetilir. `approval.get`, `urlPath` döndürür; onaylanmış genel kökeni bilen yüzeyler ayrıca mutlak bir `url` alabilir. İnceleyici anlık görüntüleri, kaynak ve hedef kitle oturum anahtarlarını içermez. Gateway, bu yönlendirme anahtarlarını ayrı `session.approval` izdüşümü için sunucu tarafında tutar.

## Olaylar ve taşınabilir eylemler

PR 1, yayımlanmış olay adlarını, yükleri ve mevcut kayıt düzeyindeki alıcı filtrelerini korur:

- `exec.approval.requested`
- `exec.approval.resolved`
- `plugin.approval.requested`
- `plugin.approval.resolved`

Bu eski olaylar çalışma zamanı isteğinin tamamını içerebilir; bu nedenle onay kapsamındaki her istemciye yayımlanmamalıdır. PR 5, eski olay teslimatını genişletmek yerine temizlenmiş yaşam döngüsü izdüşümü aracılığıyla etiketli yaşam döngüsü alanları (`status`, `sourceSessionKey`, `urlPath`, terminal meta verileri ve sunum düzeyinde bir `kind`) ekler.

Onay kapsamlı bir `session.approval` izdüşüm olayı ekleyin. Kanonik olayı kalıcı hedef kitle anahtarlarıyla bir kez yayımlayın; tam oturum aboneleri eşleşen her anahtar için aynı olayı alır:

- `sessionKey`: izdüşümü alan akış.
- `sourceSessionKey`: geçidi oluşturan alt öğe/kaynak.
- `phase`: onay durumuna göre ayrıştırılan `pending \| terminal`.
- bir güvenli `OperatorApproval` izdüşümü.

İstemciler `sessions.messages.subscribe { key, agentId?, includeApprovals: true }` ile katılım sağlar. Başarılı yanıt, abone olan istemcinin ayrıca kayıt düzeyinde inceleme yetkisine sahip olduğu, tam olarak bu akış anahtarına ait en fazla 1.000 güncel bekleyen onayı içeren bir `approvalReplay` ekler. `truncated: false`, filtrelenmiş yeniden oynatmayı yetkili kılar ve yeniden bağlanan istemciler yerel bekleyen kümelerini bununla değiştirir; `truncated: true` bir aşırı yük sinyalidir ve istemciler, kanonik arama veya sonraki yaşam döngüsü olayları bunları sonuçlandırana kadar henüz görülmemiş yerel girdileri korumalıdır. Yeniden oynatma sırasında keşfedilen sonraki bir kalıcı zaman aşımı, yeni anlık görüntü döndürülmeden önce terminal mezar taşlarını yalnızca abone olmuş, kayıt düzeyinde yetkilendirilmiş hedef kitlelere gönderir. `operator.admin` doğrudan katılım sağlayabilir; daha dar kapsamlı istemciler hem eşleştirilmiş bir cihaz kimliği hem de `operator.approvals` gerektirir. Yalnızca oturum aboneliği hiçbir zaman onay görünürlüğü sağlamaz.

Olayı `src/gateway/server-broadcast.ts` içindeki `operator.approvals` altında kaydedin. İzdüşüm gözlemseldir: hiçbir zaman transkript satırları eklemez, `sessions.changed` yaymaz veya bir aracıyı uyandırmaz.

`src/interactive/payload.ts` içindeki `MessagePresentationAction` öğesini genişletin:

```ts
type MessagePresentationAction =
  | { type: "command"; command: string }
  | { type: "callback"; value: string }
  | {
      type: "approval";
      approvalId: string;
      approvalKind: "exec" | "plugin";
      decision: ExecApprovalDecision;
    }
  | { type: "url"; url: string }
  | { type: "web-app"; url: string };
```

Core, onaylanmış mutlak bir Control UI kaynağı kullanılabilir olduğunda türü belirlenmiş karar eylemleri ve ayrı bir İncele bağlantısı oluşturur. Kanallar, bir onay eylemini kendi geri çağırma biçimlerinde kodlar ve çözümü kanonik hizmete gönderir. Geri çağırma, sığıyorsa tam kanonik kimliği; aksi takdirde satırın benzersiz tam özetli `resolution_ref` değerini kullanır. Referans yalnızca kompakt bir arama anahtarıdır: normal Gateway kimlik doğrulaması, kayıt yetkilendirmesi, açık tür, izin verilen karar doğrulaması, son tarih uzlaştırması ve ilk yanıt CAS işlemi uygulanmaya devam eder. Kanallar kimlikleri kısaltmamalı, karma öneklerini çözümlememeli, `/approve` metnini ayrıştırmamalı veya türü bir kimlik önekinden çıkarsamamalıdır.

`button.url`, `button.webApp` ve komut destekli onay denetimlerini kullanımdan kaldırılmış plugin SDK uyumluluk girdileri olarak koruyun. Bunları SDK sınırında normalleştirin; aynı PR içinde paketlenmiş tüm dâhilî çağıranları geçirin. `/approve {id} {decision}`, düğmenin anlamsal sözleşmesi değil, bir metin geri dönüşü ve CLI/sohbet komutu olarak kalır.

## Control UI

Rota `${basePath}/approve/{approvalId}` şeklindedir. Kimlik tek yol parametresidir; kaynak oturum kimliği kayıttan gelir.

Mevcut yönlendirici tam statik rotalara sahip olduğundan ve bilinmeyen yolları Sohbet'e yeniden yazdığından, normal rota normalleştirmesinden önce `ui/src/app/bootstrap.ts` içinde bu derin bağlantıyı algılayın. Normal Gateway/kimlik doğrulama kurulumunu yeniden kullanın ancak kenar çubuğu kabuğunun ve genel modalın dışında bağımsız bir onay sayfası oluşturun.

Belgenin sahibi, URL'sini sunan Gateway'dir. İlk bağlantısı, bu seçimin ayarlarını değiştirmeden veya kopyalamadan tam uygulamanın kalıcı uzak Gateway seçimini yok sayar; yalnızca kimlik doğrulama, hizmet veren Gateway oturumu kapsamında kalır. Güvenilir yerel kimlik doğrulama veya ayrıca doğrulanmış bir `gatewayUrl` geçersiz kılması hedefi değiştirebilir. Core, `.json` veya `.js` ile biten kimlikler dâhil olmak üzere, plugin HTTP rotaları ve statik uzantı algılamasından önce tek segmentli `/approve` ad alanını ayırır; Control UI sunumu devre dışıyken ayrılmış rota `404` ile güvenli biçimde başarısız olur. Başarısız bir geç yüklenen parçanın güvenlik kararını bir yükleme göstergesinde bırakmaması için sayfayı ana Control UI paketinde tutun.

Sayfa durumları:

- yükleniyor
- kimlik doğrulaması gerekli
- bekliyor
- çözümleniyor
- burada onaylandı veya reddedildi
- başka yerde çözümlendi
- süresi doldu
- iptal edildi
- yasak/bulunamadı
- yeniden denemeli bağlantı hatası

Sayfa, kimliği doğrulanmamış ikinci bir REST API'yi değil Gateway RPC'yi çağırır. Tarayıcı yenilemesi kalıcı durumu yeniden okur. Gateway kimlik bilgilerini hiçbir zaman URL'ye, sorguya veya parçaya yerleştirmez.

## Yetkilendirme ve gizlilik

URL bir konum belirleyicidir, yetki değildir. Çözümleme şunları gerektirir:

1. kimliği doğrulanmış Gateway bağlantısı;
2. `operator.approvals` veya `operator.admin`;
3. kayıt düzeyinde inceleyen yetkilendirmesi.

Kayıt düzeyindeki kurallar:

- `operator.admin` inceleyebilir.
- `reviewer_device_ids` mevcut olduğunda belirleyicidir. Yalnızca listelenen eşleştirilmiş
  `operator.approvals` cihazı inceleyebilir; istekte bulunan cihaz ayrıca
  listelenmedikçe örtük erişime sahip değildir.
- Açık bir inceleyen listesi olmadığında, istekte bulunan eşleştirilmiş
  `operator.approvals` cihazı kendi kaydını inceleyebilir.
- İstekte bulunan veya inceleyen bağlaması bulunmayan gerçekten eski kayıtlar,
  yükseltmelerin zaten bekleyen işleri kullanılamaz hâlde bırakmaması için eşleştirilmiş cihazlara yönelik geniş görünürlüğü korur.
- Cihazsız dâhilî çalışma zamanları, kapsamlı onay çalışma zamanı bağlantısı üzerinden
  okuyamaz ancak çözümleyebilir. Bu yetki yalnızca sunucu tarafından kimliği doğrulanmış çalışma zamanı
  belirtecinden gelir; herkese açık `approval.resolve` alanları bunu
  oluşturamaz.
- Canlı istekte bulunan bağlantı sahipliği eski bağdaştırıcılar için geçerliliğini korur;
  hiçbir zaman eşleşen bir istemci adından çıkarılmaz.
- Hedef kitle üyeliği yalnızca sunumu değiştirir. Yetkilendirmeyi hiçbir zaman genişletmez.

`approval.get` yalnızca temizlenmiş inceleyen izdüşümünü sunar ve dâhilî kaynak/hedef kitle yönlendirme anahtarlarını içermez. PR 5 `session.approval` olayı, Gateway kalıcı hedef kitle anlık görüntüsünü sunucu tarafında uyguladıktan sonra tek hedefi olan `sessionKey` ile `sourceSessionKey` değerini taşır. Mevcut exec/plugin olayları, tüketiciler geçiş yapana kadar geçmiş yüklerini ve kısıtlı alıcılarını korur. Yürütülebilir istek, komut bağlaması ve devam işlemi yalnızca işlem içi bekleyicide kalır. Kalıcı satır; güvenli sunumun yanı sıra yaşam döngüsü, yönlendirme ve denetim meta verilerini içerir; ham ortam değerlerini, kimlik bilgilerini, kimlik doğrulama üstbilgilerini veya kanal geri çağırma verilerini hiçbir zaman depolamaz.

## Hedef kitle izdüşümü

Hedef kitleyi eklemeden önce bir kez hesaplayın ve sıralı anlık görüntüyü kalıcılaştırın. Sahiplik her zaman tek bir üst zincir değil, bir grafiktir: bir alt öğenin hem mevcut denetleyicisi hem de ilk istekte bulunanı olabilir ve bu sahipler farklı köklere ulaşabilir.

Belirlenimci bir genişlik öncelikli dolaşım kullanın:

1. Kuyruğu kaynak oturum anahtarıyla başlatın.
2. Kuyruktan çıkarılan her anahtar için en son alt ajan kayıt defteri satırını okuyun ve iki farklı sahiplik kenarını sabit sırayla kuyruğa ekleyin: önce `controllerSessionKey`, ardından `requesterSessionKey`.
3. Kullanılabilir bir kayıt defteri satırı mevcut olduğunda, yönlendirme sonrasında eski kalmış olabilecek oturum girdisi soyunu ayrıca izlemeyin. Aksi hâlde tek mevcut geri dönüş kenarı olan `parentSessionKey ?? spawnedBy` değerini kuyruğa ekleyin.
4. İlk ve en kısa yolun kazanması için kuyruğa eklerken normalleştirin ve yinelenenleri kaldırın.
5. 64 benzersiz anahtarda durun; bu hedef kitle boyutu sınırı dolaşım derinliğini de sınırlar.

Kayıt defteri kaynağı `src/agents/subagent-registry-read.ts` şeklindedir; sahiplik alanları `src/agents/subagent-registry.types.ts` içinde tanımlanır. Oturum geri dönüş alanları `src/config/sessions/types.ts` içinde tanımlanır.

İstenen ve sonlandırılmış izdüşümler, onay beklerken odak/denetleyici sahipliği değişse bile aynı kalıcı hedef kitleyi kullanır. Bu, istek izdüşümünü alan her hedef kitle oturumu akışı için sonlandırma temizliğini garanti eder. Çözümleme her zaman kaynak onay kimliğini hedefler; hedef kitle oturumları hiçbir zaman klonlanmış onay durumu almaz. İletilen kanal iletisi temizliği, aşağıdaki ayrı teslimat konumu takibi olarak kalır.

Yalnızca bir onay nedeniyle transkript iletileri yazmayın, sistem istemleri eklemeyin, sahip turlarını başlatmayın veya `sessions.changed` yayınlamayın.

## Teslim edilen yüzeylerin yakınsaması

Yerel onay işleyicileri, etkin denetimleri değiştirecek veya kullanımdan kaldıracak kadar uzun süre teslim edilmiş ileti girdilerini zaten saklar. Genel iletilmiş onay iletileri şu anda `MessageReceipt` değerini atar; bu nedenle başka bir yüzeydeki karar, eski denetimlerinin hâlâ bekliyormuş gibi görünmesine yol açabilir. Ayrı bir takip işlemi, paylaşılan durum veritabanındaki bir `operator_approval_deliveries` alt tablosuyla bu açığı kapatır.

Her satır; onay kimliğini, benzersiz bir teslimat kimliğini, kanalı/hesabı/tam rotayı, sınırlandırılmış ve JSON ile doğrulanmış kanala özel ileti konum belirleyicisini, teslimat zaman damgalarını ve sonlandırma durumunu depolar. Geri çağırma verilerini, karar belirteçlerini veya ham onay isteklerini hiçbir zaman depolamaz. Konum belirleyici kodlamasının ve ileti değişikliğinin sahibi kanaldır; kanonik durumun, hedef seçiminin, yeniden deneme politikasının ve geri dönüş sonlandırma metninin sahibi core'dur.

Teslimat kaydı ve sonlandırma çözümlemesi yarış durumunu güvenli biçimde ele alır:

1. Bekleyen bir gönderim makbuzunu döndürdükten sonra teslimat konum belirleyicisini ekleyin ve üst onay durumunu tek işlem içinde okuyun.
2. Üst öğe zaten sonlandırılmışsa geç teslimatı beklemede bırakmak yerine hemen sonlandırma planlayın.
3. Kaydedilen her sonlandırılmış geçiş, sonlandırılmamış tüm teslimat satırlarını ayrıca planlar; bırakılabilir yayınlar tetikleyici değildir.
4. Bir kanal sonlandırıcısı `replaced`, `retired` veya `unsupported` bildirir. Değiştirildi durumu yinelenen bir sonlandırma iletisini engeller; kullanımdan kaldırıldı durumu mevcut sonlandırma takibini gönderir; desteklenmeme veya hata, onay CAS işlemini geri almadan geri dönüşe geçer.
5. Başlangıç, tamamlanmamış teslimatları bulunan sonlandırılmış onayları yeniden dener ve temizliği Gateway yeniden başlatmasına karşı dayanıklı kılar.

Bu taşıma yaşam döngüsü isteğe bağlı bir teslimat bağdaştırıcısı kancasıdır; oluşturucu veya modele yönelik ileti eylemi değildir. QQ C2C/grup iletilerinin şu anda düzenleme, silme veya klavye temizleme API'si yoktur; bu bağdaştırıcı desteklenmemiş olarak kalır ve taşıma bir değiştirme API'si edinene kadar yalnızca daha sonraki bir tıklamadan sonra kanonik gerçeği gösterebilir.

## Yeniden başlatma, zaman aşımı ve rota anlamları

SQLite kalıcılığı, yürütmenin devam ettirileceği anlamına gelmez. Komut/araç bağlamaları, güvenlik açısından hassas çalışma zamanı olguları içerebildikleri ve devam ettirilebilir bir iş sözleşmesi olmadıkları için bellekte kalır.

Gateway başlangıcında:

- yeni bir çalışma zamanı dönemi oluşturun;
- eski dönemlerdeki bekleyen satırları `gateway-restart` nedeniyle atomik olarak `cancelled` durumuna geçirin;
- URL'lerinin ne olduğunu açıklayabilmesi için satırları koruyun;
- eksik çalışma zamanı bağlamasına karşı daha sonraki bir onayı hiçbir zaman yürütmeyin.

Zamanlayıcılar uyandırma optimizasyonlarıdır. Son tarih yetkisi `expires_at_ms` içinde depolanır; okumalar, beklemeler ve çözümlemelerin tümü süre sonu uzlaştırmasını çalıştırır.

Nihai katı davranış:

- zaman aşımı -> `expired`, reddet;
- rota yok -> `denied`, reddet;
- çalıştırma iptali -> `cancelled`, reddet;
- hatalı biçimlendirilmiş güvenilir karar -> `denied`, reddet;
- yalnızca izin verilen açık bir izin kararı -> `allowed`.

Şu anda yayımlanan exec davranışı hâlâ bu sözleşmeyle çelişir:

- `src/agents/bash-tools.exec-host-shared.ts`, `askFallback` uygulayabilir.
- `docs/tools/exec-approvals.md` ve `docs/cli/approvals.md` bu yüzeyi belgeler.

Plugin onayları artık zaman aşımında ve hatalı biçimlendirilmiş kararlarda güvenli biçimde başarısız olur; eski
`timeoutBehavior` alanı kabul edilmeye devam eder ancak yok sayılır. Exec katı anlamları
takibi; kodu, türleri, belgeleri, testleri ve değişiklik günlüğünü açık
sahip/güvenlik incelemesiyle birlikte güncellemelidir. `askFallback`, geçiş
sırasında geçit öncesi politika seçimini açıklamaya devam edebilir ancak oluşturulmuş
bekleyen bir kaydın zaman aşımını onaya dönüştürmemelidir.

## Uyumluluk planı

- Eklemeli Gateway protokolü; protokol sürümü yükseltilmez.
- Mevcut exec/plugin yöntemlerini ve olaylarını dış sınırda koruyun.
- `plugin:` önekleri dâhil olmak üzere mevcut kimlikleri koruyun ancak önekleri tür bilgisi olarak kullanmayı bırakın.
- `/approve` metin komutu davranışını koruyun.
- Eski düğme URL/Web App alanlarını ve komut eylemlerini plugin SDK uyumluluk girdisi olarak koruyun; yeni core çıktısının türü belirlenmiştir.
- Tüm paketlenmiş kanalları ve dâhilî çağıranları aynı türü belirlenmiş eylem değişikliğinde geçirin.
- Yeni URL/sayfa ve sonraki zaman aşımı davranışı değişikliği için bir değişiklik günlüğü girdisi ekleyin.
- Bir bilgi isteme modu ayarı eklemeyin.

## Kullanıma sunma

### PR 1: kalıcı yaşam döngüsü

- Bu tasarım notu.
- Paylaşılan SQLite şeması, Kysely üretimi, depo ve 30 günlük budama.
- Gateway onay hizmeti, çalışma zamanı bekleyici köprüsü ve yeniden başlatma sonrası sahipsiz kayıt işleme.
- Birleştirilmiş `approval.get/resolve`.
- Exec/plugin yöntem bağdaştırıcıları.
- İlk yanıt kazanır, yinelenebilirlik, süre sonu, yetkilendirme ve tüketim testleri.
- Henüz UI veya kanal davranışı değişikliği yok.

### PR 2: türü belirlenmiş eylemler ve kanal geri çağırmaları

- Türü belirtilmiş onay, URL ve Web App eylemleri.
- Çekirdek sunum oluşturucuları ve plugin SDK dışa aktarımları.
- Açık sahip türüyle taşıma katmanına özel geri çağırma kodlaması.
- Taşıma sınırlarını aşan kanonik kimlikler için kalıcı, sabit boyutlu geri çağırma referansları.
- Paketlenmiş kanalların komut metni ve onay kimliği çıkarımından uzaklaştırılması.
- Tıklanan yüzeyde kanonik ilk yanıt gerçeği ve mümkün olan en iyi şekilde etkin yerel terminal güncellemeleri; kalıcı kanal iletisi terminalleştirmesi sonraki bir çalışma olarak kalır.
- SDK ve paketlenmiş kanal testleri.

### PR 3: Control UI derin bağlantısı

- Bağımsız, kimliği doğrulanmış onay sayfası ve temel yolu dikkate alan başlangıç yönlendirmesi.
- Operatörün kaydedilmiş uzak seçimini değiştirmeden hizmet veren Gateway'e bağlanma.
- Varlık benzeri kimlikler dâhil, çekirdeğin sahip olduğu onay HTTP ad alanı.
- Gateway tarafından oluşturulan URL yükü ve yaşam döngüsü olayları sunulana kadar bekleyen durum yoklaması.
- Mobil genişlik, yeniden bağlanma, yarışan yanıt, yeniden yükleme ve bağlanmış yol kanıtı.

### PR 4: yerel istemciler

- iOS ve Android inceleme yüzeyleri türü dikkate alan `approval.get/resolve` kullanır; watchOS, inceleyen kişi için güvenli istemleri ve kararları eşleştirilmiş iPhone üzerinden aktarır.
- Watch, kompakt aktarım sözleşmesinin desteklediği yürütme kararlarını sunar: bir kez izin ver ve reddet.
- Kanonik ilk yanıt terminal gerçeği, yerel denenmiş karar durumunun yerini alır.
- Kaybolan veya belirsiz çözümleme alındıları, kanonik geri okuma gerçekleşene kadar denetimleri dondurur.
- Daha önce yayımlanmış Gateway v4 örnekleri, dar kapsamlı bir eski yöntem geri dönüşü aracılığıyla yürütme incelemesini korur; yüzeyler arasında korunan terminal durumu, birleştirilmiş yöntemleri gerektirir.
- İnceleyen kişi uyarıları ve sahip bağlamı iPhone, Watch ve Android genelinde görünür kalır.
- Yerel birim, derleme ve platform kanıtı.

### PR 5: üst öğe yaşam döngüsü yayılımı

- PR 1'de kalıcılaştırılan hedef kitle anlık görüntüsünden `session.approval` bekleyen/terminal teslimatı.
- Transkript değişikliği veya ajan uyandırma olmadan tam oturum aboneliği, yeniden bağlanma tekrarı ve terminal mezar taşları.
- Yaşam döngüsü geri çağırmaları, kalıcı ekleme/CAS sonrasında çalışır ve hiçbir zaman onay yetkisi hâline gelmez.
- İç içe alt ajan ve yeniden bağlanma kanıtı.

### PR 6: kapalı başarısızlık davranışı

- `node-invoke-plugin-policy.ts` ve gömülü plugin aracısını yinelenen yetkiden uzaklaştırın.
- Katı zaman aşımı, hatalı biçim, rota yokluğu, bağlama ve bir kez izin verme tüketim semantiği.
- Bir istek beklemeye alındıktan sonra bunları uygulamadan, yayımlanmış izin verici zaman aşımı ayarlarını kullanım dışı bırakın.
- Çok yüzeyli çekişme ve hata enjeksiyonu kanıtı.

### Sonraki çalışma: kalıcı uzak ileti temizliği

- İletilen teslimat konumlandırıcılarını kalıcılaştırın ve yeniden başlatma sonrasında teslim edilen her kanal iletisini terminalleştirin.
- Bu taşıma yaşam döngüsünü kanonik onay yetkisinden ve türü belirtilmiş sunum eylemlerinden ayrı tutun.

## Testler

Gerekli odaklı kapsam:

- SQLite'ın yeniden açılması, bekleyen ve terminal projeksiyonlarını korur.
- Eşzamanlı iki çözümleyici tam olarak bir CAS kazananı üretir.
- Aynı kararın yeniden denenmesi idempotent biçimde başarılı olur; çelişen yeniden deneme kaydedilmiş kazananı döndürür.
- Son tarihte veya sonrasında çözümleme onay veremez.
- `allow-once`, terminal denetim durumunu silmeden tam olarak bir kez tüketilebilir.
- Başlangıç, eski çalışma zamanı dönemlerini iptal eder.
- Yetkisiz arama ve çözümleme, kaydın varlığını açığa çıkarmaz.
- Açık inceleyen kişi izin listesi ve genel eşleştirilmiş `operator.approvals` davranışı.
- Yürütme ve plugin eski yöntemleri aynı depoyu paylaşır.
- Gateway istek/listeleme/alma/çözümleme şemaları ve eklemeli olay yükleri.
- Türü belirtilmiş eylem normalleştirmesi, geri dönüşlü işleme, SDK dışa aktarımları ve paketlenmiş kanal geçişleri.
- Telegram geri çağırma kodlaması, taşıma katmanına özel veriler içerir ve komut dizesi çıkarımı içermez.
- Doğrudan alt öğe, dallanmış denetleyici/isteyen sahipleri, iç içe sahipler, yeniden atama, oturum alanı geri dönüşü, döngü ve hedef kitle boyutu üst sınırı.
- İstenen ve terminal hedef kitle dizileri aynıdır.
- Sahip projeksiyonları transkript değişikliğine veya ajan uyanmasına neden olmaz.
- Control UI rotası `/` konumunda ve yapılandırılmış bir temel yolda çalışır; yenileme, bekleyen veya terminal gerçeğini gösterir.
- Eşzamanlı Control UI ve Telegram yanıtları bir kazanan ve kaybedende "başka yerde çözümlendi" ifadesini gösterir.
- Yerel onay tanımlayıcıları ve Gateway sahip tanımlayıcıları, yönlendirme ve uzlaştırma boyunca UTF-8 baytlarını tam olarak korur.
- Yerel RPC ailesi uzlaşımı, kabul edilen her Gateway rotası için tek bir kanonik veya eski aileyi sabitler ve kullanımdan sonra hiçbir zaman sessizce alt sürüme geçmez.
- Kaybolan yerel çözümleme alındıları, kanonik geri okumaya kadar eylemleri dondurur; başarısız geri okuma bir kazanan uyduramaz veya bir Watch yenilemesini onaylayamaz.
- Watch anlık görüntü isteği korelasyonu yalnızca tam olarak eşleştirilmiş Gateway sahibi ve tamamlanmış kanonik iPhone geri okuması için kabul edilir.
- Mobil genişlikte bir onay sayfası, Telegram eylem temizliği ve Android, iPhone ve Watch genelinde bir bekleyen/çözümleme/geç-kaybeden gidiş dönüşü dâhil, Testbox/Crabbox üzerinden kullanıcı yolu kanıtı.

## Gözlemlenebilirlik

Onay kimliği, tür, kaynak oturum anahtarı, durum, neden ve gecikme ile yapılandırılmış, içeriksiz geçiş günlükleri yayınlayın. Önizlemeyi veya ham bağlamayı asla günlüğe kaydetmeyin.

Şunları izleyin:

- türe göre istek sayısı;
- tür/durum/nedene göre terminal sayısı;
- bekleyen göstergesi;
- istekten terminale gecikme;
- çözümleme yarışı sonuçları: kazanan, idempotent yeniden deneme, çakışma, süresi dolmuş;
- teslimat rotası sayısı ve rota yokluğu reddetmeleri;
- başlangıçta sahipsiz kalanların iptalleri;
- hedef kitle boyutu.

Daha sonraki olay teslimatı başarısız olsa bile kaydedilmiş bir geçiş başarılıdır. Yaşam döngüsü aboneleri, PR 5 tekrarı ve kanonik arama aracılığıyla toparlanır. Kalıcı kanal iletisi terminalleştirmesi, yukarıdaki ayrı sonraki çalışma olarak kalır.

## Açık kararlar

1. **Dışarıdan erişilebilen Control UI kaynağı.** Her anlık görüntü, kararlı göreli `urlPath` taşır. Mutlak URL yalnızca Gateway erişimi başarıyla sağlandıktan sonra önbelleğe alınmış bir Tailscale Serve/Funnel konumundan duyurulabilir; `allowedOrigins`, istek Host üstbilgileri, `gateway.remote.url` ve yalnızca görüntüleme amaçlı geri döngü/LAN adayları kanonik kaynaklar değildir. Telegram, onay yolunu önyükleme boyunca korumak için kimliği doğrulanmış Mini App sarmalayıcısını kullanabilir. İsteğe bağlı ters proxy'ler, ayrıca incelenmiş açık bir genel URL sözleşmesi var olana kadar yalnızca göreli kalır. Bir kanalın kaynağı tahmin etmesine asla izin vermeyin.
2. **Yürütme katı zaman aşımı uyumluluk geçişi.** Plugin onay zaman aşımları artık kapalı biçimde başarısız olur ve `timeoutBehavior` kullanım dışı bırakılmıştır. Kalan yayımlanmış `askFallback` sözleşmesinin, bekleyen bir istek zaman aşımına uğradıktan sonra yürütmeye yetki vermeyi bırakmasından önce açık sahip/güvenlik incelemesi, değişiklik günlüğü, belgeler ve bir geçiş/kullanımdan kaldırma kararı gerekir.
3. **Gateway'siz gömülü mod.** Öneri: başlangıçta yalnızca yerel tutun, ardından bir Gateway mevcut olduğunda kanonik hizmetin istemcisi hâline getirin. Hiçbir sunucunun çözümleyemeyeceği bir derin bağlantıyı duyurmayın.
