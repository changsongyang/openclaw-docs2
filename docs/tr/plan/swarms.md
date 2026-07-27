---
x-i18n:
    generated_at: "2026-07-26T22:51:10Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 90c6c85a837448f4e5ceccdccf73489db801ad502cbbb2f3eb04d6aff7e902f0
    source_path: plan/swarms.md
    workflow: 16
---

# Swarms — kod modunda agent dağıtımı ve orkestrasyon

Durum: Yayınlandı — yerini `docs/tools/swarm.md` aldı. Bu belge,
uygulama tasarım kaydı olarak korunmaktadır.

## 1. Nedir ve neden kullanılır

Bir **swarm**, kod modu betiğinden deterministik biçimde orkestre edilen çok
sayıda alt agent'tan oluşur: N okuyucuyu dağıtma, bulguları karşıt yaklaşımla
doğrulama, durum bilgisi tutan bir önceliklendirici aracılığıyla sentezleme,
karar kapılarında döngü kurma. Kontrol akışı (`Promise.all`,
`while`, `if`) orkestrasyonun _kendisidir_ — özellikle **grafik DSL'si,
yeni mod veya yeni üst düzey araç yüzeyi yoktur**.

OpenClaw kod modu (QuickJS-WASI, anlık görüntü/devam ettirme, köprü istekleri)
temeli oluşturur. Beklemeye alınmış bir köprü çağrısı VM anlık görüntüsünden ve
gateway yeniden başlatmasından etkilenmez ve tam olarak durduğu yerden devam
eder — betikler üzerinde hiçbir determinizm kısıtlaması olmadan günlük yeniden
oynatma tasarımlarından daha güçlüdür.

Adlandırma: ürün/belge adı **Swarm**'dır. Kod tanımlayıcıları değişmeden kalır:
`agents.*` konuk API'si, `tools.swarm` yapılandırması, `swarm` grup sütunları.

## 2. Kararlar (bakım sorumlusu, 2026-07-17)

- Maliyet: yapılandırma sınırları zorunlu tutulur; swarm başına token bütçesi isteğe bağlıdır. Zorunlu bütçe yoktur.
- Onaylar: alt öğeler **kapalı başarısızlık / etkileşimsiz** olarak çalışır. Onay gerektiren
  eylemler reddedilir; ret, alt öğe sonucunda bildirilir; kararı betik
  verir. Dağıtımdan kaynaklanan operatör istemi yığını oluşmaz.
- v1 yalnızca model tarafından yazılmış geçici betikleri kapsar. Kaydedilmiş/adlandırılmış iş akışları, CLI/Cron
  girişi: daha sonra (başsız kod modu Cron için zaten mevcuttur).
- Alt öğe kimliği: varsayılan olarak `tools.swarm.defaultAgentId`
  yapılandırması aracılığıyla ayrılmış worker agent (mevcut alt agent hedef izin listesine göre doğrulanır); her oluşturma için
  `agentId` geçersiz kılması. Çekirdek, paketlenmiş agent kimliği sunmaz; belgeler yalın bir
  `worker` agent yapılandırması önerir.
- Codex kaynak değişikliği yoktur. Codex düzeneği oluşturma/bekleme kalıbını kullanır (§8).

## 3. Mimariye genel bakış

```
kod modu betiği (QuickJS VM, gateway)            Codex V8 betiği (codex işlemi)
  agents.run(...) ── beklemeye alınmış köprü çağrısı tools.sessions_spawn / tools.agents_wait
        │                                                │ öğe/araç/çağrı RPC'si (her biri ≤600 sn)
        ▼                                                ▼
             ÇEKİRDEK (düzenekten bağımsız, bu depo)
  sessions_spawn {collect:true, outputSchema, fastMode, groupId}
  agents_wait {ids, timeoutSeconds}
        │
  alt agent kayıt defteri (SQLite): toplayıcı tamamlanma kayıtları, swarm grup kimliği
        │
  alt öğeler = sıradan alt agent oturumları (şerit sınırlı, onaylarda kapalı başarısızlık)
        │
  sessions.changed SSE ──► Control UI noktaları / kenar çubuğu / kanal durum mesajı
```

Oluşturma/tamamlama/sonuçlandırma semantiğinin tek kurallı sahibi (çekirdek
araçlar + kayıt defteri). İki bekleme aktarımı: QuickJS, bir köprü çağrısını
süresiz olarak beklemeye alır (anlık görüntü); Codex, `agents_wait` öğesini
sınırlı RPC'lerde yoklar.

## 4. Yapılandırma kapısı (v1)

Yeni `tools.swarm` (genel + agent başına geçersiz kılma, `tools.codeMode`
ile aynı birleştirme kalıbı):

```jsonc
"tools": {
  "swarm": {
    "enabled": false,            // ana kapı, varsayılan KAPALI
    "maxConcurrent": 8,          // aynı anda çalışan alt öğeler (swarm şerit sınırı)
    "maxChildrenPerGroup": 50,   // swarm grubu başına etkin alt öğeler
    "maxTotalPerGroup": 200,     // grup başına ömür boyu oluşturma sayısı (kontrolsüz çalışmaya karşı son önlem)
    "waitTimeoutSecondsMax": 600,
    "defaultAgentId": ""         // isteğe bağlı; oluşturmada agentId belirtilmediğinde alt öğe agent kimliği
  }
}
```

- Zod: `CodeModeSchema` gibi `boolean | strict object` birleşimi
  (`src/config/zod-schema.agent-runtime.ts`); `swarm: true` → `{enabled: true}`.
- `src/config/types.tools.ts` içindeki türler (hem agent başına hem üst düzey `tools`),
  `schema.labels.ts` içindeki etiketler, `schema.help.runtime.ts` içindeki yardım.
- `resolveCodeModeConfig` işlevini yansıtan `resolveSwarmConfig(cfg, agentId)`
  çözümleme yardımcısı (`src/agents/code-mode.ts:215`), tüm sayıları sınırlar.
- Devre dışıyken kapı etkileri: `agents_wait` aracı kataloglarda bulunmaz;
  `sessions_spawn` üzerindeki `collect`/`outputSchema`/`fastMode`/`groupId`
  parametreleri, yapılandırma anahtarını belirten açık bir hatayla reddedilir. Başka hiçbir davranış değişmez.
- `defaultAgentId`, `resolveSubagentAllowedTargetIds` aracılığıyla doğrulanır
  (`src/agents/subagent-target-policy.ts`); bilinmeyen kimlik → geri dönüş değil, oluşturma hatası.

## 5. Çekirdek: toplayıcı modu oluşturma + `agents_wait` (v1)

### 5.1 `sessions_spawn` eklemeleri (tümü swarm'ın etkin olmasına bağlıdır)

- `collect: boolean` — true olduğunda alt öğe çalıştırması,
  duyuru/yönlendirme teslimi yerine `expectsCompletionMessage: false` ve bir **toplayıcı tamamlanma kaydı**
  ile kaydedilir. Araç, `{ runId, sessionKey }` değerini
  hemen döndürür. Kanal/iş parçacığı bağlaması yoktur.
- `outputSchema: object` — JSON Schema. Alt öğenin araç yüzeyine
  sentetik bir `structured_output` aracı eklenir; sistem istemi eki,
  nihai sonucuyla bunu tam olarak bir kez çağırmasını söyler. Doğrulama
  başarısız olduğunda alt öğeye bir kez yeniden deneme dürtmesi verilir; bundan sonra tamamlanma kaydı
  ham metin ve bir `schemaError` ile birlikte `structured: undefined`
  taşır.
- `fastMode: true | "auto" | false`, mevcut `FastMode` ekseni
  (`src/shared/fast-mode.ts`) kullanılarak `resolveSubagentModelAndThinkingPlan`
  (`src/agents/subagent-spawn-plan.ts`) aracılığıyla model/düşünme ile birlikte alt öğe oturum yamasına
  geçirilir. Atlanırsa devralınır.
- `groupId: string` — swarm grup damgası. Varsayılan değer
  `swarm:<requesterSessionKey>:<runId-of-requesting-run>`.
  Kayıt defteri kaydında ve alt öğe oturum satırında kalıcılaştırılır. Sınırlar, listeleme, toplu
  arşivleme ve noktalar için kullanılır.
- `label: string` zaten mevcuttur — noktalarda ve `subagents list` içinde gösterilir.
- Alt öğe agent kimliği: `params.agentId` → aksi hâlde `tools.swarm.defaultAgentId` → aksi hâlde
  istekte bulunan agent (mevcut davranış).

### 5.2 Onaylarda kapalı başarısızlık

Toplayıcı alt öğeleri etkileşimsiz bir onay bağlamıyla çalışır: operatör onayı
gerektirecek herhangi bir araç çağrısı, alt öğenin görebildiği yapılandırılmış
bir ret (`approval_required`) olarak sonuçlanır; alt öğenin bu engeli sonucunda
bildirmesi beklenir. Uygulama: toplayıcı modu alt öğe çalıştırmaları için zorunlu
bir `deny` çözümleyicisiyle mevcut yürütme/araç onay ilkesi tesisatını yeniden kullanır.
Toplayıcı alt öğelerinden operatör yüzeylerine hiçbir onay olayı gönderilmez.

### 5.3 `agents_wait` aracı (yeni, kapılı)

```
agents_wait({ ids: string[], timeoutSeconds?: number })
→ {
    completed: [{ runId, status: "done"|"failed"|"killed"|"timeout",
                  result: string, structured?: unknown, schemaError?: string,
                  sessionKey, label?, usage?: {inputTokens, outputTokens} }],
    pending: string[]
  }
```

- **En az bir** kimlik tamamlanır tamamlanmaz (ilk tamamlanma / yarış
  semantiği, işlem hatlarını etkinleştirir) veya `completed: []` ile zaman aşımında döner.
- `timeoutSeconds` varsayılan olarak 30'dur, `waitTimeoutSecondsMax` ile sınırlanır.
- İdempotenttir: zaten tamamlanmış kimlikler kayıtlarını yeniden döndürür (kayıtlar
  grup arşivlenene kadar tutulur). Bilinmeyen kimlik → fırlatma değil, kimlik başına hata girdisi.
- Sahiplik: yalnızca bir çalıştırmayı oluşturan oturum (veya üst zinciri) onu
  bekleyebilir — kod modundaki `wait` ile aynı sahiplik kuralı (`code-mode.ts:1684`).
- Kayıt defteri: tamamlanma kayıtları mevcut alt agent kayıt defteri SQLite
  deposunda (`subagent-registry.store.sqlite.ts`) tutulur — yeni alanlar, yeni depo yok,
  şema sürümü artışı yok (yalnızca ek sütunlar; §9 kısıtlamasına bakın).

### 5.4 Sınırların uygulanması

- `maxConcurrent`: toplayıcı alt öğeleri mevcut alt agent şeridinde çalışır ancak
  swarm grubu başına sayılır; sınırı aşan oluşturmalar FIFO kuyruğuna alınır (sunucu tarafında,
  oluşturma yolunda — runId hemen döndürülür, bir yuva boşaldığında çalışma başlar).
- `maxChildrenPerGroup` / `maxTotalPerGroup`: sınır aşıldığında oluşturma, türü belirlenmiş bir hatayla
  reddedilir; hata metni yapılandırma anahtarını belirtir.
- Derinlik: toplayıcı alt öğeleri `DEFAULT_SUBAGENT_MAX_SPAWN_DEPTH` semantiğini korur
  (iç içe yerleştirme açıkça yapılandırılmadıkça alt öğeler yapraktır).

## 6. Test sözleşmesi (v1, A şeridi)

- Birim: yapılandırma çözümleme/sınırlama; devre dışıyken kapı retleri; groupId
  varsayılanı; sınır uygulaması (kuyruk + ret); bekleme yarış semantiği; bekleme
  idempotansı; sahiplik reddi; yapılandırılmış çıktı doğrulaması + yeniden deneme dürtmesi +
  schemaError yolu; fastMode'un oturum yamasına aktarılması; defaultAgentId
  doğrulaması.
- Entegrasyon (vitest, sahte model çalışma zamanı): 3 toplayıcı alt öğesi oluşturma, döngüde
  bekleme, ilk tamamlanma sırasını ve son boşaltmayı doğrulama; gateway yeniden başlatma
  simülasyonu: kayıt defterini yeniden yükleme → bekleme, kalıcılaştırılmış tamamlanmadan sonuçlanır.
- Tüm testler `*.test.ts` ile aynı konumdadır; canlı model çağrısı yoktur.

## 7. QuickJS konuk yüzeyi (B şeridi, çekirdekten sonra)

- Konuk genelleri `CONTROLLER_SOURCE`
  (`src/agents/code-mode.worker.ts:190-374`) içinde kurulur, ayrılmış adlar
  `code-mode-namespaces.ts` içine eklenir:
  - `agents.run(prompt, opts) → Promise<result|structured>` — kolaylık katmanı:
    toplayıcı oluşturma + sunucunun tamamlanmada sonuçlandırdığı özel bir köprü yöntemi (`agentWait`)
    üzerinde beklemeye alınmış bekleme (yoklama yok; anlık görüntüye dayanıklı).
  - `agents.session(system, opts) → Promise<handle>`;
    `handle.send(input, opts) → Promise<...>`; `handle.close()`. (v1.1 —
    run() sonrasında yayınlanır; `mode:"session"` + tur başına toplayıcı kayıtlarını kullanır.)
  - `phase(title)`, `log(message)` — gönder ve unut köprü bildirimleri →
    swarm ilerleme olayları.
- `CodeModeBridgeMethod` (`code-mode.ts:91`) içine eklenen köprü yöntemleri:
  `agentSpawn`, `agentWait`, `swarmNote`. `agentSpawn`/`agentWait`,
  **yapıları gereği** yeniden oynatmaya dayanıklıdır: kayıt defteri kaydında saklanan
  idempotans anahtarı `(codeModeRunId, bridgeId)`; yeniden başlatma, kalıcılaştırılmış tamamlanmalardan yeniden sonuçlandırır
  ve asla iki kez oluşturmaz.
- Bekleyen `agentWait` köprü çağrıları çalıştırmanın anlık görüntü TTL'sini uzatır (bekleyen
  agent kümesi sinyaldir; bayrak yoktur).
- `API.read("agents.d.ts")` sanal dosyası, türü belirlenmiş yüzeyi ve
  dağıtım / kapı / döngü kalıplarını belgeler (`createCodeModeApiVirtualFiles`,
  `code-mode-namespaces.ts:876`).

## 8. Codex düzeneği izdüşümü (sonraki şerit)

- `sessions_spawn` (yeni parametrelerle) ve `agents_wait`, mevcut
  dinamik araç köprüsünden geçer; Codex kod modu betiklerinde otomatik olarak
  `tools.*` şeklinde görünürler (doğrulandı: `codex-rs/code-mode/src/runtime/globals.rs:14-65`,
  `codex-rs/core/src/tools/spec_plan.rs:448-507`).
- `agents_wait`, uzun dinamik araç zaman aşımı sınıfını alır (600 sn sınırı;
  `extensions/codex/src/app-server/dynamic-tool-execution.ts:37-39`) ve
  zaman aşımına/yeniden oynatmaya dayanıklı olarak işaretlenir.
- Codex üst öğeleri için grup anahtarı: `swarm:<parentSessionKey>:<turnId>`.
- Codex'e özgü `spawn_agent` alt agent'ları birlikte çalışır; görev yansıtma satırları
  aynı ilerleme yüzeyini besler.

## 9. Kalıcılık ve saklama

- Yeni depo yoktur. Kayıt defteri kayıtları mevcut alt agent kayıt defteri
  SQLite tablolarını genişletir; alt öğeler sıradan `sessions` satırlarıdır. Yalnızca ek sütunlar
  — **SQLite şema sürümü artışı gerektiren herhangi bir değişiklik için önce
  bakım sorumlusunun açık onayı gerekir** (depo ilkesi).
- Kayıt defteri kaydı + alt öğe oturum meta verilerinde swarm grup kimliği.
- Saklama: tamamlanmış toplayıcı kayıtları **grup arşivine** kadar korunur:
  üst öğe çalıştırması tamamlandığında (veya TTL sona erdiğinde), grubun alt öğeleri
  toplu olarak arşivlenir (mevcut `DEFAULT_SUBAGENT_ARCHIVE_AFTER_MINUTES`
  taraması grup başına çalışacak şekilde genişletilir).

## 10. İlerleme yüzeyi ("noktalar") — sonraki şerit

- Örtük ve düzenek odaklıdır. Mevcut `sessions.changed` SSE +
  kayıt defterinden türetilir; `phase`/`log` notları semantik ekler. Agent odaklı işleme yoktur.
- Control UI: çalışma alanı widget ailesindeki `swarm` işleyicisi
  (`ui/src/lib/workspace/widgets/`) — aşamaya göre gruplandırılmış nokta ızgarası, anlatıcı
  satırı, nokta başına durum/etiket/model; kenar çubuğu alt öğe ağacı değişmez.
- Kanallar: grup başına hız sınırlı, düzenlenen tek bir durum mesajı (şunu izler:
  `docs/concepts/streaming.md`; hiçbir zaman alt öğe başına mesaj yoktur).

## 11. Labs sayfası (Control UI, bağımsız hat)

Settings → **Labs**: deneysel özellik açma/kapatma seçenekleri; ilk girdiler **Code Mode**
ve **Swarm**. Her satır: ad, tek satırlık açıklama, doküman bağlantısı, mevcut
`config.patch` RPC aracılığıyla bağlanan açma/kapatma seçeneği (RFC 7396 merge-patch — 
`tools.codeMode.enabled` / `tools.swarm.enabled` değerini ayarlayın) ve uygun olduğunda
"yeniden başlatma gerekli" ipucu. Keşfedilebilir, ancak metin deneysel durumu
açıkça belirtir. i18n: tüm dizeler normal `en.ts` + eşitleme işlem hattı üzerinden geçer.

## 12. Yerleştirme (daha sonra)

- `placement` oluşturma sırasında seçimi: `"local"` (varsayılan) | mevcut
  çalışan ortamı yönlendirmesi (`sessions.dispatch`) aracılığıyla `"cloud:<profile>"`; paylaşımlı kutu
  SSH korumalı alan alt süreçlerinin yetersiz kaldığı kanıtlanırsa havuzlanmış yerleştirme daha sonra.
- Orkestratör VM her zaman gateway üzerinde kalır; settle/dots/budget
  yerleştirmeden bağımsızdır.

## 13. Hedef dışı öğeler

- Grafik DSL'si yoktur — kontrol akışı grafiğin kendisidir (bilinçli tercih, dokümante edilmiştir).
- Codex kaynak değişikliği yoktur; Codex Code Mode iç bileşenleri yeniden kullanılmaz.
- v1'de kaydedilmiş/adlandırılmış iş akışları ve CLI giriş noktası yoktur.
- Alt süreç başına operatör onayının üst katmana aktarılması yoktur.
- Yayılım ölçeğinde 1:1 bulut kaynak sağlama yoktur.
- Kararlı durum çalışma zamanı uyumluluk katmanları yoktur; swarm yeni ve kapılı bir yüzeydir.

## 14. Derleme aşamaları / PR dilimleme

1. **Hat A (çekirdek)**: §4 yapılandırma + §5 oluşturma/bekleme/sınırlar/onaylar + §6 testler.
2. **Hat C (Labs sayfası)**: §11 — bağımsızdır, önce birleştirilebilir.
3. **Hat B (QuickJS yüzeyi)**: §7 — A sözleşmeleri birleştirildikten sonra.
4. Dots oluşturucu (§10), Codex projeksiyonu (§8), `agents.session` (§7 v1.1),
   yerleştirme (§12), kullanıcı dokümanlarının yeniden yazılması — bu sırayla takip PR'ları.

Her PR: CI yeşil, `$autoreview` temiz, varsayılan olarak kapalı, main yayımlanabilir.
