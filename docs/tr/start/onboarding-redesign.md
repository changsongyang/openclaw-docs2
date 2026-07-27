---
read_when:
    - İlk katılım sürecinin yeniden tasarımının bir aşamasını uyguluyor veya inceliyorsunuz
summary: Sorumlu katılım sürecinin yeniden tasarımı için uygulama planı (güncel tutulan belge)
title: İlk katılım sürecinin yeniden tasarımı
x-i18n:
    generated_at: "2026-07-27T00:03:37Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: f892991583d0b77a670e9bf7aa5a0c74af3b3eac9e7b0448706486254eb7e2a0
    source_path: start/onboarding-redesign.md
    workflow: 16
---

# İlk katılım yeniden tasarımı uygulama planı

> **Yaşayan belge.** Bu sayfa, koruyucu ilk katılım yeniden tasarımını
> uygulama düzeyinde izler ve her aşama tamamlandıkça güncellenir. Son aşama
> birleştirildiğinde bu sayfa, kullanıcıya yönelik ilk katılım kılavuzu olarak yeniden yazılır ve
> belge gezintisine eklenir. O zamana kadar kasıtlı olarak `docs.json` içinde yer almaz.

## Ana hedef

Teknik bilgisi olmayan bir kullanıcı `openclaw onboard` yazar (veya uygulamayı açar) ve
tek bir etkileşimli varlık tarafından karşılanır: sistem koruyucusu OpenClaw ("koruyucu"
yalnızca dahili addır; kullanıcı her zaman "OpenClaw" adını görür). Bu varlık kullanıcının yapay zekâsını
bulur, sorular yerine duyurulan varsayılanlarla her şeyi kurar, aracısını görünür bir kimlik anıyla
doğurur ve sonrasında sistemin bakım sorumlusu olarak sonsuza kadar erişilebilir kalır.
Varsayılan olarak sihirli, tek bir onay sınırı, çıkmaz yol yok.

Tasarım ilkeleri (kararlaştırılmıştır, gelişigüzel yeniden tartışmayın):

- **Kolayca geri alınabilen duyurulmuş varsayılanlar**, ilerlemeyi engelleyen soruların yerini alır. Tek
  kesin gereksinim, çalışan çıkarımdır; diğer her şey bir tekliftir.
- **Sıfırıncı soru onay sınırıdır**: "Tam erişim" (önerilen), keşfin
  sessiz ve otomatik olduğu anlamına gelir; "Önce sor" ise yapay zekâ
  taraması, uygulama taraması ve bellek kaynağı taraması dâhil her keşfi tek bir
  açık evet onayının arkasında tutar ve hiç tarama yapmayan tamamen manuel bir yol sunar.
- **Aşamalı zekâyla kullanıcı arayüzü olarak konuşma**: koruyucu yüzeyi,
  herhangi bir yapay zekâ çalışmadan önce mevcuttur (betikli diyalog); bir rota doğrulandığı anda
  model destekli hâle gelir ve bunu görünür biçimde belirtir. Asla zekâ taklidi yapmaz:
  rota doğrulanmadan önce serbest metin girildiğinde nazikçe "önce beynimi çalışır
  hâle getireyim" yanıtını verir.
- **Doğuş bir törendir**: aynı ileti dizisinde avatar değişir, aracı kendine ad verir
  ve kendi yüzünü seçer. Koruyucu hiyerarşiyi bir kez öğretir: "sistem hakkında
  bana sor veya doğrudan aracına sor — o iletir."
- **Güven, kaynağa göre kademelendirilir**: resmî katalog girdileri önceden seçilebilir;
  üçüncü taraf ClawHub Skills öğeleri, model sıralamasından bağımsız olarak asla önceden seçilmez
  ve etiketleri, yayıncının kodunu yüklediklerini belirtir.
- **Yapılandırılmış kurulumlar dokunulmazdır**: ilk katılımın yeniden çalıştırılması bir doğrulama
  geçişidir. Kurulumu asla yeniden uygulamaz ve Gateway hizmetini asla yeniden başlatmaz.
- **Terminal bir sorudan ziyade yedek seçenektir**: bir Gateway erişilebilir olduğunda tarayıcı
  panosunu tercih edin; asla "terminal mi, tarayıcı mı?" diye sormayın.
- **Zayıf modeller daraltılmış bir yüzey kullanır** (otomatik `localModelLean`) ve bu,
  araçlar, kod modu veya bağlam pencereleri terimleriyle değil, sade bir dille açıklanır.

## Mevcut yayımlanmış akış (1-3. aşamalardan sonra)

Yeni bir macOS kurulumunda `openclaw onboard`, sorunsuz yol — toplam dört Enter:

1. Güvenlik notu → onaylamak için bir Enter (kalıcı olarak kaydedilir; bir daha asla sorulmaz).
2. **Sıfırıncı soru**: "Her şeyi nasıl kurmalıyım?" — Tam erişim (önerilen)
   veya Önce sor. `wizard.accessMode` olarak kalıcı hâle getirilir; yeniden çalıştırmalarda kaydedilmiş
   seçim varsayılan olur. Korumalı + "manuel olarak yapılandır", herhangi bir tarama yapmadan
   sağlayıcı seçiciye ulaşır ve bellek kaynağı taramasını da atlar.
3. **Keşif gösterisi**: kodlama CLI'larını, ortam anahtarlarını ve yerel çalışma zamanlarını algılar;
   kodlama aracıları bulunduğunda esprili bir bildirim gösterir; adayları sırayla canlı olarak
   test eder ve hataları sessizce tek bir özet satırında toplar (ayrıntılar "Diğer seçenekleri
   gör" arkasındadır). Çalışan ilk rota, tam seçiciye tek tuşla ulaşılabilen
   bir varsayılan olarak duyurulur; seçenekleri incelemek ve atlamak, çalışan
   rotayı korur.
4. Bellek içe aktarma teklifi (Claude Code / Codex / Hermes); keşif
   reddedildiğinde atlanır.
5. Yalnızca yeni kurulumlar: standart kurulum planı otomatik olarak uygulanır
   (çalışma alanı, Gateway hizmeti, oturumlar — etkileşimli "evet" seçeneğinin
   çalıştırdığı planın aynısı). Yapılandırılmış kurulumlar "zaten kurulu" yazdırır ve
   hizmete asla dokunmaz.
6. **Uygulama önerileri**: doğrulanmış model tarafından resmî
   kataloglar + ClawHub ile eşleştirilen yüklü uygulamalar; resmî kanal Plugin'leri
   önceden işaretlenmiş olarak gelir, üçüncü taraf Skills öğeleri bir uyarı etiketiyle isteğe bağlıdır. Atlanabilir;
   devre dışı bırakma anahtarı `wizard.appRecommendations`.
7. **Doğuş**: bir Gateway erişilebilir olduğunda tarayıcıya aktarım açılır (GUI) veya
   pano URL'si yazdırılır (başsız/SSH) ve Control UI'ın
   bağlanması beklenir — "Pano bağlandı — tarayıcınızda devam ediliyor." Aksi durumda veya
   `--tui` ile terminal TUI, önyükleme doğuş
   mesajıyla başlatılır ve aracı kendini tanıtır.

Uzak Gateway ilk katılımı, eski etkileşimli aktarımını
(`handoffMode: "chat"`) korur; kurulum uzak Gateway üzerinde uygulanmalıdır.

## Aşamalar

| #   | Aşama                                                                                                                                                     | Yüzey               | Durum                                                                                                                             |
| --- | --------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Yüklü uygulama Plugin önerileri (tarama, adaylar, yapay zekâ eşleştiricisi, sihirbaz adımı, `device.apps` Node komutu)                                    | klasik + yönlendirmeli CLI | birleştirildi ([#109668](https://github.com/openclaw/openclaw/pull/109668))                                                   |
| 2   | CLI koruyucu omurgası (sıfırıncı soru, keşif gösterisi, otomatik uygulama + doğuş)                                                                         | yönlendirmeli CLI    | birleştirildi ([`a83ed13204f1`](https://github.com/openclaw/openclaw/commit/a83ed13204f118adf1009e5ac88d5afe1905b86c))                |
| 3   | Önce tarayıcıya aktarım (GUI oturumu algılama, pano bağlantısını bekleme, yedek seçenek olarak TUI)                                                         | CLI → web            | birleştirildi ([#110054](https://github.com/openclaw/openclaw/pull/110054))                                                   |
| 4   | Web koruyucu yüzeyi (seçenek kartları, `openclaw.chat` üzerinde türü belirlenmiş `question` alanı, sihirbaz adımı yansıtma, ilk çalıştırma aktarımı) | Control UI           | birleştirildi ([#110141](https://github.com/openclaw/openclaw/pull/110141), [#110242](https://github.com/openclaw/openclaw/pull/110242)) |
| 5   | Doğuş ve önyükleme (tek seferlik semantiğe sahip öneri deposu, kendini adlandırma doğum dizisi, yeni kurulumdan sonra otomatik doğuş aktarımı; avatar kademeleri ertelendi) | aracı önyüklemesi | birleştirildi ([#110173](https://github.com/openclaw/openclaw/pull/110173), [#110331](https://github.com/openclaw/openclaw/pull/110331)) |
| 6   | Koruyucu varlığı PR1 (sabitlenmiş kenar çubuğu girdisi, Ayarlar'da OpenClaw'a Sor, normal arayüz çerçevesinde bakım sorumlusu karşılaması; olay yorumları ve kanal çağrısı PR2'dir) | web + kanallar | birleştirildi ([#110269](https://github.com/openclaw/openclaw/pull/110269))                                                   |
| 7   | Dayanıklılık (bozuk yapılandırmada koruyucuya erişim, kısmi yüzey kurtarma, otomatik doctor)                                                                | Gateway              | takip çalışması                                                                                                                   |

## Aşama başına uygulama notları

### Aşama 1 — uygulama önerileri (PR #109668)

- Tarayıcı: `src/infra/installed-apps.ts` (TCC gerektirmeyen macOS numaralandırması;
  sembolik bağlantılı `.app` paketlerini izler).
- Adaylar: resmî kataloglar + ClawHub araması, toplam 20s bütçe, çevrimdışıyken
  yalnızca katalog adaylarına sorunsuz gerileme. Katalog girdileri, üst düzey
  `id` içermeyen paket manifestleridir — adaylar çözümlenmiş
  Plugin kimliğine göre anahtarlanır (gerçek paketlenmiş kataloglara karşı regresyon testi yapılmıştır;
  bir zamanlar `entry.id` ile anahtarlama tüm kataloğu tek girdide birleştirip
  tüm resmî önerileri düşürüyordu).
- Yapay zekâ eşleştiricisi: doğrulanmış rotada tek bir tamamlama
  (`src/system-agent/setup-app-recommendations.ts`); elle düzenlenmiş paket kimliği eşlemesi yoktur —
  model, tesadüfi ad çakışmalarını reddeder. Çıktı, çözümlenmiş modelin kendi
  `maxTokens` bütçesiyle sınırlanır (açık bir sınır geçirilmediğinde akış katmanı bunu uygular).
- **Tedarik zinciri koruması**: ClawHub listeleme metni yayıncı tarafından kontrol edilir ve
  eşleştirici istemine ulaşır; dolayısıyla bir listeleme kendisini
  "önerilen" olarak tanıtabilir. Yalnızca resmî katalog girdileri önceden seçilebilir; ClawHub
  Skills öğeleri her zaman açıkça işaretlenmelidir ve "üçüncü taraf ClawHub
  Skill öğesi; yayıncısının kodunu yükler" şeklinde etiketlenir.
- Node komutu `device.apps` (TS Node ana bilgisayarı, Android zarf uyumluluğu);
  paylaşım varsayılan olarak kapalıdır; Gateway devre dışı bırakma anahtarı `wizard.appRecommendations`.
- Sunum, klasik sihirbaz ve yönlendirmeli koruyucu akışında
  (`src/wizard/setup.app-recommendations.ts`) yer alır; önyükleme
  sonuna yeniden hedefleme 5. aşamada kalır (hizmet zaten enjekte edilebilir bir envanter
  kaynağı alır). Tek seferlik semantik (yalnızca kabul edilene kadar teklif etme, kaydedilmiş tarama)
  de 5. aşama deposuyla gelir; bugün yeniden çalıştırıldığında tekrar teklif edilir.
- Ayrıca düzeltildi: özel `completeSetupInference` istemleri artık
  32 token'lık doğrulama sondası çıktı sınırını devralmaz (`SETUP_INFERENCE_TEST_MAX_TOKENS`
  yalnızca "OK yanıtı ver" sondasına uygulanır).

### Aşama 2 — CLI koruyucu omurgası (PR #109841)

- `src/commands/onboard-guided.ts` içinde akışın yeniden düzenlenmesi; uzak Gateway ilk katılımı,
  `handoffMode: "chat"` aracılığıyla eski sohbet aktarımını korur.
- Sıfırıncı soru `wizard.accessMode` ("full" | "guarded") değerini kalıcılaştırır; yeniden çalıştırmalar
  varsayılan olarak kaydedilmiş seçimi kullanır (varsayılanı kabul etmek, korumalı ayarı asla sessizce
  tam erişime düşüremez). Korumalı + manuel, `listManualSetupInferenceOptions` kullanır
  (yalnızca yapılandırma/manifestler, sondalama yoktur) ve
  bellek kaynağı taramasını atlar.
- Keşif: sessiz hata toplama (tek özet satırı; ayrıntılar
  "Diğer seçenekleri gör" arkasındadır), kodlama aracısı bildirimi, duyurulmuş rota varsayılanı. Bildirimdeki
  oturum sayıları, düşük maliyetli bir oturum sayımı bağlantısı bulunana kadar ertelenmiştir
  (yalnızca nitel ifade).
- Yeni kurulumlar: `applySystemAgentSetup` (belirlenimci etkileşimli
  "evet"), ardından önyükleme mesajıyla başlatılan `launchTuiCli` üzerinden doğuş.
  Yapılandırılmış kurulumlar (önceden mevcut model veya Gateway yapılandırması — sihirbaz
  zaman damgaları hiçbir şeyi kanıtlamaz; configure/doctor ile paylaşılır):
  yalnızca doğrulama — uygulama yok, Gateway hizmetini yeniden başlatma yok. Uygulama hatası,
  etkileşimli sohbete geri döner.

### Aşama 3 — önce tarayıcıya aktarım (PR #110054, birleştirildi)

- `src/commands/onboard-browser-handoff.ts` yalnızca grafik oturumu
  algılamasını (`SSH_CONNECTION`/`SSH_TTY`; Linux'ta `DISPLAY`/`WAYLAND_DISPLAY`)
  ve 60 saniyelik GUI / 300 saniyelik SSH beklemesini yönetir. Yönlendirmeli ilk katılım şu anda
  devri yalnızca macOS'ta etkinleştirir; `--tui` ve diğer platformlar
  terminal çıkış yolunu korur. Linux/Windows desteği sonraki bir çalışmadır.
- Pano bağlantıları, klasik sonlandırmayla aynı `resolveAdvertisedControlUiLinks`,
  `resolveLocalControlUiProbeLinks` ve `buildOnboardingControlUiUrl` yardımcılarını
  kullanır. Tarayıcı başlatma, paylaşılan `openUrl` yardımcısını kullanır.
- Hazır olma durumu, mevcut `system-presence` RPC'sini **yapılandırılmış paylaşılan gizli anahtarı
  sunan CLI modunda bir geri döngü istemcisi** olarak yoklar — bu, her
  `openclaw` komutunun kullandığı güvenilir yoldur. Ham paylaşımlı kimlik doğrulamalı bir Control UI istemcisi,
  SecretRef gateway'lerinde "cihaz kimliği gerekli" hatasıyla reddedilir. Erişilebilirlik
  ön kontrolü, bekleme döngüsüyle aynı hedefi (ve gizli anahtarı) çözümler; böylece
  geçit ile bekleme, kimlik doğrulama konusunda asla çelişemez. Devir yalnızca,
  bağlı bir `openclaw-control-ui`/`webchat` varlık satırı başlatma öncesi
  temel çizgiye göre yeniyse tamamlanır (zaten açık olan bir pano bunu
  tamamlayamaz).
- `gateway.controlUi.enabled: false`, herhangi bir URL gösterilmeden önce kısa devre yapar.
- Yalıtılmış, aynı yapılandırmaya sahip bir gateway üzerinde uçtan uca kanıtlandı: URL'yi yazdırma → gerçek
  tarayıcı bağlantısı → "Pano bağlandı — tarayıcınızda devam ediliyor" → terminal
  çıkış yolu yok. Daha önceki "token uyuşmazlığı" beklemesi bir test düzeneği
  artefaktıydı — aşağıdaki test çalışma kılavuzuna bakın.

### Aşama 4 — web sorumlusu yüzeyi (birleştirildi: #110141, #110242)

- `openclaw.chat` üzerinde, seçenek kartı bileşenini kullanan `/custodian` sayfası
  (2-4 kart, en fazla bir önerilen, her zaman atlanabilir); `?onboarding=1` üzerinden
  ilk katılım çerçevesi; model kurulumu ilk çalıştırma tamamlaması buraya devreder.
- Yapılandırılmış sorular, `SystemAgentChatResult` üzerinde türü belirlenmiş ve eklemeli bir `question` alanıdır
  (seçenek başına `reply` metni; düzyazı macOS uygulaması/TUI için her zaman
  bağımsızdır). Üreticiler: her iki ilk katılım karşılama çeşidi ve
  2-4 kapalı seçenekli barındırılan sihirbaz seçme/onaylama adımları — gerçek kanal
  sihirbazları kart olarak işlenir. PR1'deki dize işaretleyici geçici çözümü silindi.
- Oturum sahipliği, gateway URL'si + sunulan her kimlik bilgisiyle
  (token, parola, bootstrap token'ı, saklanan cihaz token'ı — geçici
  hello kesintileri boyunca kalıcı) sınırlandırılır; başarısız kullanıcı sıraları asla yeniden oynatılamaz; hassas
  girdi olduğu gibi gönderilir ve transkriptte maskelenir.

### Aşama 5 — çıkış ve bootstrap (birleştirildi: #110173, #110331)

- Sorumlu, adsız bir ajan oluşturur (araç çağrısı); ajanın bootstrap süreci
  kendi adını belirleyerek açılır. PR1, seremoniyi üç adımla sınırlandırılmış olarak sunar (ad → öz
  satırı → beceriler sorusu) ve kendi çizdiği avatar/görüntü üretme basamaklarını
  (model tarafından oluşturulan adaylar → hazır işaretler → logoyu koru) sonraki bir çalışmaya erteler. Aynı
  ileti dizisi, avatar değişimi; pençe işareti sorumluya ayrılmış kalır. Üzerinde
  anlaşılan kimlik iki kez kalıcılaştırılır: `IDENTITY.md`/`SOUL.md` içine (ajanın
  okuduğu) ve `openclaw agents set-identity` aracılığıyla (kanalların ve UI'ın
  gösterdiği).
- Öneriler (aşama 1 hizmeti, tek seferlik anlam bilgisiyle saklanan tarama),
  bootstrap dosyası kaldırılmadan önceki son bootstrap adımı olarak sunulur: "asgari küme
  mi, azami kolaylık mı?" Bootstrap, saklanan teklifi
  `openclaw onboard recommendations --json` aracılığıyla okur (yalnızca opak kurulum kimlikleri) ve
  seçim işlendikten sonra onaylar; böylece bir daha asla sormaz. Kanal
  bağlama düğmeleri kanal başına kurulum çalışma kılavuzları taşır; ajan
  kimlik bilgilerini konuşarak toplar ve yapılandırma yazma işlemlerini sorumluya aktarır
  ("OpenClaw'a soruluyor…" kurallı ifadedir).
- Kendi kendine öğrenme duyurulmaz, sorulur ve beceri atölyesi
  onayı işlevini de görür; ClawHub'ın sürüm güveni, tarama, doğrulama ve bütünlük
  kontrollerini, ayrıca yayıncı kodu uyarısını açıklayın — asla her sürümün imzalı olduğunu ima etmeyin.
- Otomatik çıkış yayımlandı: yeni bir kurulum yapılandırması uygulandığında çıkış duyurulur ve
  devir gerçekleştirilir (terminal TUI / gateway istemcileri için `open-agent`); web sayfası,
  "Uyan, dostum!" taslağı önceden doldurulmuş olarak ajan sohbetine ulaşır. Devir,
  yalnızca yazma sonrası doğrulama temizse tetiklenir. Silme sonrasında sıfır ajan
  olduğunda otomatik işlem yerine seçenek sunulması, sonraki iyileştirme olarak kalır.

### Aşama 6 — sorumlu varlığı (PR1 birleştirildi: #110269; yorum/çağırma PR2'de)

- PR1'de yayımlandı: varsayılan olarak sabitlenmiş "OpenClaw" kenar çubuğu girdisi (yeni profiller;
  mevcut kullanıcılar kayıtlı sabitlemelerini korur ve özelleştirme/More üzerinden erişir), ilk Settings girdisi olarak "OpenClaw'a
  Sor" ve bakım görevlisi selamlamasını isteyen normal çerçeveli `/custodian` ziyaretleri
  (ilk katılım karşılama çeşidi yok); Exit setup yalnızca ilk katılım modunda işlenir.
  Yerleşik satır içi Settings bölmesi, paylaşılan konuşma görünümü çıkarımı gerektirir
  (sonraki çalışma).
- Clippy karşıtı güvenlik sınırlarına sahip olay tepkili yorumlar: yalnızca önemli veya
  başarısız değişiklikler, istenmediği sürece Settings ziyareti başına en fazla bir kez. Aynı
  olay bağlantı noktası, sorumluyu daha sonra bozulmuş kimlik doğrulamanın veya çalışmayan
  kanalların sesi hâline getirir.
- Kanallar: günlük kullanımda görünmez (ajan aktarır); açık
  çağırmayla ve aynı ileti dizisindeki ajan devre dışı olaylarında, platformun izin verdiği yerlerde kendi adı ve
  pençe avatarıyla erişilebilir.
- Kurulumda zayıf model algılandı: `localModelLean` otomatik olarak ayarlanır ve sorumlu,
  yükseltme teklifiyle birlikte bunu sade bir dille belirtir.
- Sorumlu, dahili takma adını bilir ("bazıları bana
  sorumlu diyor — OpenClaw da olur") ve ajandan her zaman adıyla söz eder.

### Aşama 7 — dayanıklılık (oluşturmadan önce sahip kararı gerekiyor)

İlk taslak — "yapılandırma ne kadar bozuk olursa olsun sorumluya
erişilebilmelidir" — deponun güvenlik politikasıyla çelişiyor: kök kılavuz,
yapılandırma yapısal olarak geçersiz olduğunda Gateway'in **başlatmayı reddettiğini**
ve yalnızca SecretRef sahibi hatalarının yapılandırılmış-kullanılamaz
yeteneklere indirgendiğini belirtir. Geçersiz yapılandırmadan herhangi bir yüzey sunmak,
bir uygulama ayrıntısı değil, politika değişikliğidir. İki kapsamdan birini seçin:

- **Seçenek A (önerilen, politikayla uyumlu): CLI tarafında otomatik doctor.** Bir
  gateway veya CLI başlatması bilinen şekilli geçersiz bir yapılandırmayla başarısız olduğunda CLI,
  `openclaw doctor --fix` çalıştırmayı teklif eder (veya onayla çalıştırır), ardından bir kez yeniden dener ve
  durumu açıkça bildirir. Gateway davranışı değişmez; sorumluya mevcut
  indirgenmiş SecretRef yolu ve terminal üzerinden erişilmeye devam edilir.
- **Seçenek B (açık sahip onayı + güvenlik incelemesi gerekir): gateway
  asgari yüzey modu.** Yapısal olarak geçersiz yapılandırmada, yalnızca sorumlu
  konuşmasını ve doctor eylemlerini sunan sıkı şekilde kilitlenmiş bir yüzey başlatılır. Bu,
  kapalı kalarak hata verme başlatma sözleşmesini yeniden yazar ve herhangi bir koddan önce kendi giriş
  koruma yaklaşımını tanımlamalıdır.

4-6. aşamalardan kalan sonraki çalışmalar (izleniyor, zamanlanmadı): çıkış için avatar/görüntü üretme
basamakları; türü belirlenmiş `question` alanının macOS uygulamasında işlenmesi;
sorumlu için yerleşik satır içi Settings bölmesi (paylaşılan konuşma görünümü
çıkarımı gerekir); olay tepkili yorumlar ve kanal çağırma/ajan devre dışı kurtarma
(aşama 6 PR2); zayıf modeller için otomatik `localModelLean`; mevcut
kullanıcıların kayıtlı kenar çubuğu sabitlemelerinin OpenClaw girdisini benimseyip benimsememesi.

## Test ve yayımlama çalışma kılavuzu (zorlukla edinildi; 4-6. aşamalardan önce okuyun)

- **`OPENCLAW_STATE_DIR`, Gateway hizmetini yalıtmaz.**
  LaunchAgent etiketi (`ai.openclaw.gateway`) makine genelindedir: yalıtılmış bir durum diziniyle yapılan
  yeni kurulum ilk katılım testi, gerçek makinenin hizmetini YENİDEN YAZAR ve YENİDEN BAŞLATIR
  (sarmalayıcı betikler yalıtılmış dizinin içine yerleşir; o dizin temizlendiğinde bir sonraki
  hizmet başlangıcı bozulur). Her yeni kurulum testinden sonra gerçek ortamdan
  `openclaw gateway install --force && openclaw gateway
restart` ile geri yükleyin ve plist'i doğrulayın. Ürün için sonraki çalışma:
  durum dizini kapsamlı hizmet etiketleri veya yabancı bir hizmeti algılayan ilk katılım.
- **Güvenli uçtan uca düzenek**: yalıtılmış yapılandırmayı bir `gateway`
  bölümüyle önceden doldurun (böylece ilk katılım yapılandırılmış kurulum yolunu izler ve hizmete asla
  dokunmaz) ve `openclaw gateway run` komutunu boş bir bağlantı noktasında düz bir token ile normal bir ön plan işlemi olarak
  çalıştırın. Bu düzenek, gerçek bir tarayıcı bağlantısı dâhil olmak üzere 3. aşama döngüsünü
  kanıtladı.
- **Kimlik doğrulama yolları yalnızca kimlik bilgilerine değil, istemci kimliğine göre de değişir.** Varlık ve
  diğer operatör okumaları, aynı yapılandırmadaki kimlik bilgilerini kullanan CLI modunda bir geri döngü
  istemcisini kullanır. Token kimlik doğrulamalı gateway'ler paylaşılan gizli anahtarı gerektirir; SecretRef/none
  gateway'leri token olmadan güvenilir geri döngü kimlik doğrulamasına geri dönebilir. Control
  UI olarak tanımlanan bir tarayıcı istemcisi, cihaz kimliği veya güvenli bağlam
  geri döngü izni gerektirir. FARKLI bir yapılandırma sunan gateway'e karşı kimlik doğrulaması yapan bir
  sonda (LaunchAgent tuzağına bakın) "token uyuşmazlığı" hatasıyla başarısız olur — bu
  artefakt 3. aşamayı kısa süreliğine bekletti.
- **Tamamlama sondaları**: `runSetupInferenceTest`, doğrulama sondasını
  32 çıktı token'ıyla sınırlar; özel istemler sınırı atlar ve modelin kendi
  `maxTokens` değeriyle sınırlandırılır. Akıl yürütme modelleri bu bütçeyi önce gizli
  akıl yürütmeyle tüketir — boş metinli bir sıra genellikle bütçenin orada tükendiği anlamına gelir.
- **Ajanın yayımlanması, tam HEAD üzerinde barındırılan CI gerektirir.** Ağır `CI` iş akışı,
  kuruluş yükü altında gönderimlerde kuyruğa girmeyebilir; bakımcı geri dönüşü,
  PR dalında bir sürüm geçidi tetiklemesidir:

  ```bash
  gh workflow run ci.yml --ref <branch> -f target_ref=<head-sha> -f release_gate=true -f pull_request_number=<pr>
  ```

  Çalıştırma, `head_sha` eşleşmesi için
  dal referansında olmalıdır ve başlık
  `CI release gate <sha>` olur; bunu `scripts/verify-pr-hosted-gates.mjs`
  kabul eder. Ardından her zamanki gibi `scripts/pr` hazırlama/birleştirme işlemini yapın.

- **CI'ın odaklanmış testlerin ötesinde zorunlu kıldığı geçitler**: doküman haritası
  (herhangi bir doküman sayfası eklendikten sonra `pnpm docs:map:gen`), oxlint (`no-map-spread`,
  `max-lines` — dosyaları bölün, asla bastırmayın), `check:test-types`, knip
  ölü kod denetimi (yalnızca üretimin tükettiği öğeleri dışa aktarın; testleri genel API'ler üzerinden yönlendirin)
  ve canlı test parçası sınıflandırıcısı
  (`test/scripts/test-live-shard.test.ts`, tüm yeni `*.live.test.ts` öğelerini listelemelidir).

## Karar günlüğü

- Onay öncelikli değil, acil durdurma anahtarlı sihirli tarama (aşama 1; kalıcı çıktı,
  taramadan önce model ve ClawHub kullanımını açıklar ve sonuç notu bunu yineler).
- Node `device.apps` komutunu içeren tam dikey akış (aşama 1).
- Üçüncü taraf ClawHub Skills hiçbir zaman önceden seçilmez ve
  yayıncının kodunun yükleneceği belirtilir; resmî girdiler önceden işaretlenebilir
  (aşama 1, yayımlanmış güvenlik duruşu).
- Üç değil, iki erişim kartı; onay seçimin başına alınmıştır (aşama 2).
- Engelleyici bir düğme değil, duyuruyla otomatik çatlama (aşamalar 2/5).
- Öncelik tarayıcıda: terminalde çatlama yedek seçenektir, hiçbir zaman "terminal mi
  tarayıcı mı?" sorusu değildir (aşama 3).
- Gözetmen yalnızca web/CLI ile sınırlı kalmaz; kanalda bulunur (çağırma + kurtarma)
  (aşama 6).
- Çatlama, avatar değişimiyle aynı ileti dizisinde gerçekleşir; tamamlandıktan sonra
  uygulama normal kullanıcı arayüzüne geçer (aşama 5).
- Ayarlar yüzeyi "Ayarlar" adını korur; gözetmen bunun yerini almak yerine
  burada (ve kenar çubuğunda) bulunur (aşama 6).
- Seçenek kartları sınırlandırılmıştır: 2-4 seçenek, tam olarak bir önerilen seçenek ve her zaman
  atlanabilir; aynı bileşen ilk katılımı ve ajan soru aracını destekler
  (aşama 4).
- "OpenClaw'a soruluyor…" standart yetkilendirme kalıbıdır; ruhlar renk katabilir,
  araç anlatımı sade kalır (aşama 5).
- Kullanıcıya yönelik metin, zayıf model kırpmasını açıklarken hiçbir zaman
  "kod modu", "araçlar" veya "bağlam penceresi" ifadelerini kullanmaz (aşama 6).

## Bilinen eksikler ve takip işleri

- LaunchAgent etiketi durum dizini kapsamına bağlı değildir (yukarıdaki test tuzağı; ayrıca
  gerçek bir çoklu örnek ürün eksiğidir).
- Önerilerin yalnızca bir kez sunulma semantiği ve saklanan tarama (aşama 5); yeniden çalıştırmalar
  şu anda tekrar sunar.
- Tarayıcıya aktarım yalnızca macOS'ta kullanılabilir; Linux/Windows desteği beklemededir.
- Oturum sayısı nüktesi niteldir; sayımlar için düşük maliyetli bir oturum sayma bağlantı noktası gerekir.
- Tarayıcıya aktarım normal panoya ulaşır; ilk katılım modu gözetmenine
  doğrudan bağlantı aşama 4 ile gelir.
