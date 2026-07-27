---
read_when: Finding which docs page covers a topic before reading the page
summary: OpenClaw dokümantasyon sayfaları için oluşturulan başlık eşlemesi
title: Doküman haritası
x-i18n:
    generated_at: "2026-07-26T23:17:06Z"
    model: gpt-5.6
    postprocess_version: locale-links-v1
    prompt_version: 32
    provider: openai
    source_hash: 0b58762e88df339b48cd4d15bd5c6e8490c278ed78acc5f50c415649cb7f2719
    source_path: docs_map.md
    workflow: 16
---

# OpenClaw dokümantasyon haritası

Bu dosya, aracıların dokümantasyon ağacında gezinmesine yardımcı olmak için `docs/**/*.md` ve `docs/**/*.mdx` başlıklarından oluşturulur.
Dosyayı elle düzenlemeyin; `pnpm docs:map:gen` komutunu çalıştırın.

## agent-runtime-architecture.md

- Rota: /agent-runtime-architecture
- Başlıklar:
  - H2: Çalışma Zamanı Düzeni
  - H2: Sınırlar
  - H2: Manifestler
  - H2: Çalışma Zamanı Seçimi
  - H2: Model Çalışma Zamanı Nesilleri
  - H2: İlgili Konular

## announcements/bluebubbles-imessage.md

- Rota: /announcements/bluebubbles-imessage
- Başlıklar:
  - H1: BlueBubbles'ın kaldırılması ve imsg iMessage yolu
  - H2: Neler değişti
  - H2: Yapılması gerekenler
  - H2: Geçiş notları
  - H2: Ayrıca bkz.

## auth-credential-semantics.md

- Rota: /auth-credential-semantics
- Başlıklar:
  - H2: Kararlı yoklama neden kodları
  - H2: Belirteç kimlik bilgileri
  - H3: Uygunluk kuralları
  - H3: Çözümleme kuralları
  - H2: Aracı kopyasının taşınabilirliği
  - H2: Yalnızca yapılandırmaya dayalı kimlik doğrulama rotaları
  - H2: Açık kimlik doğrulama sırası filtrelemesi
  - H2: Yoklama hedefi çözümleme
  - H2: Harici CLI kimlik bilgilerini keşfetme
  - H2: OAuth SecretRef İlke Koruması
  - H2: Eski Sürümlerle Uyumlu Mesajlaşma
  - H2: İlgili Konular

## automation/auth-monitoring.md

- Rota: /automation/auth-monitoring
- Başlıklar:
  - H2: İlgili Konular

## automation/clawflow.md

- Rota: /automation/clawflow
- Başlıklar:
  - H2: İlgili Konular

## automation/cron-jobs.md

- Rota: /automation/cron-jobs
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Cron nasıl çalışır
  - H2: Zamanlama türleri
  - H3: Heartbeat görevi geçişi
  - H3: Akış kaynakları
  - H3: Dinamik sıklık (tempo)
  - H3: Ayın günü ile haftanın günü VEYA mantığını kullanır
  - H2: Olay tetikleyicileri (koşul izleyicileri)
  - H2: Yükler
  - H3: Aracı turu seçenekleri
  - H3: Komut yükleri
  - H3: Betik yükleri
  - H2: Yürütme biçimleri
  - H2: Teslimat ve çıktı
  - H3: Hata bildirimleri
  - H3: Çıktı dili
  - H2: CLI örnekleri
  - H2: İşleri yönetme
  - H2: Webhook'lar
  - H3: Kimlik doğrulama
  - H2: Gmail PubSub entegrasyonu
  - H3: Sihirbazla kurulum (önerilir)
  - H3: Gateway'i otomatik başlatma
  - H3: Bir defalık elle kurulum
  - H3: Gmail modelini geçersiz kılma
  - H2: Yapılandırma
  - H2: Sorun giderme
  - H3: Komut basamakları
  - H2: İlgili Konular

## automation/cron-vs-heartbeat.md

- Rota: /automation/cron-vs-heartbeat
- Başlıklar:
  - H2: İlgili Konular

## automation/gmail-pubsub.md

- Rota: /automation/gmail-pubsub
- Başlıklar:
  - H2: İlgili Konular

## automation/hooks.md

- Rota: /automation/hooks
- Başlıklar:
  - H2: Doğru yüzeyi seçme
  - H2: Hızlı başlangıç
  - H2: Olay türleri
  - H2: Kancaları yazma
  - H3: Kanca yapısı
  - H3: HOOK.md biçimi
  - H3: İşleyici uygulaması
  - H3: Olay bağlamında öne çıkanlar
  - H2: Kanca keşfi
  - H3: Kanca paketleri
  - H2: Birlikte gelen kancalar
  - H3: session-memory ayrıntıları
  - H3: bootstrap-extra-files yapılandırması
  - H3: command-logger ayrıntıları
  - H3: compaction-notifier ayrıntıları
  - H3: boot-md ayrıntıları
  - H2: Plugin kancaları
  - H2: Yapılandırma
  - H2: CLI referansı
  - H2: En iyi uygulamalar
  - H2: Sorun giderme
  - H3: Kanca keşfedilmedi
  - H3: Kanca uygun değil
  - H3: Kanca yürütülmüyor
  - H2: İlgili Konular

## automation/index.md

- Rota: /automation
- Başlıklar:
  - H2: Hızlı karar kılavuzu
  - H3: Zamanlanmış Görevler (Cron) ile Heartbeat karşılaştırması
  - H2: Temel kavramlar
  - H3: Zamanlanmış görevler (Cron)
  - H3: Görevler
  - H3: Görev Akışı
  - H3: Daimî emirler
  - H3: Kancalar
  - H3: Heartbeat
  - H2: Birlikte nasıl çalışırlar
  - H2: İlgili Konular

## automation/poll.md

- Rota: /automation/poll
- Başlıklar:
  - H2: İlgili Konular

## automation/standing-orders.md

- Rota: /automation/standing-orders
- Başlıklar:
  - H2: Neden daimî emirler
  - H2: Nasıl çalışırlar
  - H2: Daimî bir emrin yapısı
  - H2: Daimî emirler ile Cron işleri
  - H2: Örnekler
  - H3: Örnek 1: içerik ve sosyal medya (haftalık döngü)
  - H3: Örnek 2: finans operasyonları (olayla tetiklenen)
  - H3: Örnek 3: izleme ve uyarılar (sürekli)
  - H2: Yürüt-doğrula-raporla kalıbı
  - H2: Çok programlı mimari
  - H2: En iyi uygulamalar
  - H3: Yapılması gerekenler
  - H3: Kaçınılması gerekenler
  - H2: İlgili Konular

## automation/taskflow.md

- Rota: /automation/taskflow
- Başlıklar:
  - H2: Görev Akışı ne zaman kullanılmalı
  - H2: Eşitleme modları
  - H3: Yönetilen mod
  - H3: Yansıtılmış mod
  - H2: Akış durumları
  - H2: Kalıcı durum ve revizyon takibi
  - H2: İptal davranışı
  - H2: CLI komutları
  - H2: Güvenilir zamanlanmış iş akışı kalıbı
  - H2: Akışların görevlerle ilişkisi
  - H2: İlgili Konular

## automation/tasks.md

- Rota: /automation/tasks
- Başlıklar:
  - H2: Özet
  - H2: Hızlı başlangıç
  - H2: Görevi ne oluşturur
  - H2: Görev yaşam döngüsü
  - H2: Teslimat ve bildirimler
  - H3: Bildirim ilkeleri
  - H2: CLI referansı
  - H2: Sohbet görev panosu (/tasks)
  - H3: Denetim Arayüzü
  - H2: Durum entegrasyonu (görev baskısı)
  - H2: Depolama ve bakım
  - H3: Görevlerin bulunduğu yer
  - H3: Otomatik bakım
  - H2: Görevlerin diğer sistemlerle ilişkisi
  - H2: İlgili Konular

## automation/troubleshooting.md

- Rota: /automation/troubleshooting
- Başlıklar:
  - H2: İlgili Konular

## automation/webhook.md

- Rota: /automation/webhook
- Başlıklar:
  - H2: İlgili Konular

## brave-search.md

- Rota: /brave-search
- Başlıklar:
  - H2: İlgili Konular

## channels/access-groups.md

- Rota: /channels/access-groups
- Başlıklar:
  - H2: Statik ileti gönderen grupları
  - H2: İzin verilenler listelerindeki referans grupları
  - H2: Desteklenen ileti kanalı yolları
  - H2: Discord kanal kitleleri
  - H2: Plugin tanılamaları
  - H2: Güvenlik notları
  - H2: Sorun giderme

## channels/ambient-room-events.md

- Rota: /channels/ambient-room-events
- Başlıklar:
  - H2: Önerilen kurulum
  - H2: Neler değişir
  - H2: Discord örneği
  - H2: Slack örneği
  - H2: Telegram örneği
  - H2: Aracıya özgü ilke
  - H2: Görünür yanıt modları
  - H2: Geçmiş
  - H2: Sorun giderme
  - H2: İlgili Konular

## channels/bot-loop-protection.md

- Rota: /channels/bot-loop-protection
- Başlıklar:
  - H2: Varsayılanlar
  - H2: Paylaşılan varsayılanları yapılandırma
  - H2: Kanal, hesap veya oda bazında geçersiz kılma
  - H2: Kanal desteği

## channels/broadcast-groups.md

- Rota: /channels/broadcast-groups
- Başlıklar:
  - H2: Genel bakış
  - H2: Yapılandırma
  - H3: Temel kurulum
  - H3: İşleme stratejisi
  - H3: Tam örnek
  - H2: Nasıl çalışır
  - H3: İleti akışı
  - H3: Oturum yalıtımı
  - H3: Örnek: yalıtılmış oturumlar
  - H2: Kullanım alanları
  - H2: En iyi uygulamalar
  - H2: Uyumluluk
  - H3: Sağlayıcılar
  - H3: Yönlendirme
  - H2: Sorun giderme
  - H2: Örnekler
  - H2: API referansı
  - H3: Yapılandırma şeması
  - H3: Alanlar
  - H2: Sınırlamalar
  - H2: İlgili Konular

## channels/channel-routing.md

- Rota: /channels/channel-routing
- Başlıklar:
  - H1: Kanallar ve yönlendirme
  - H2: Temel terimler
  - H2: Giden hedef önekleri
  - H2: Oturum anahtarı biçimleri (örnekler)
  - H2: Ana doğrudan ileti rotasını sabitleme
  - H2: Korumalı gelen ileti kaydı
  - H2: Yönlendirme kuralları (bir aracının nasıl seçildiği)
  - H2: Yayın grupları (birden çok aracı çalıştırma)
  - H2: Yapılandırmaya genel bakış
  - H2: Oturum depolama
  - H2: WebChat davranışı
  - H2: Yanıt bağlamı
  - H2: İlgili Konular

## channels/clickclack.md

- Rota: /channels/clickclack
- Başlıklar:
  - H2: Hızlı kurulum
  - H3: Alternatif: elle belirteç
  - H3: Alternatif: ortam tabanlı belirteç
  - H3: JSON5 referansı
  - H3: Hesap yapılandırma anahtarları
  - H3: Kimlik doğrulama korumalı herkese açık ana bilgisayar adını koruma
  - H2: Birden çok bot
  - H2: Oturum görüşmeleri
  - H2: Yanıt modları
  - H2: Komut menüsü
  - H2: Kalıcı medya teslimatı
  - H2: Aracı etkinliği satırları
  - H2: Hedefler
  - H2: İzinler
  - H2: Sorun giderme

## channels/discord-activities.md

- Rota: /channels/discord-activities
- Başlıklar:
  - H2: Ön koşullar
  - H2: Kurulum
  - H2: Güvenlik modeli
  - H2: Sorun giderme
  - H3: Etkinlikte “Gateway çevrimdışı” yazıyor
  - H3: Discord boş bir sayfa açıyor veya blocked:csp bildiriyor
  - H3: “Widget kullanılamıyor”
  - H3: “Bu kanalda Etkinlikleri başlatamazsınız”

## channels/discord.md

- Rota: /channels/discord
- Başlıklar:
  - H2: Hızlı kurulum
  - H2: Önerilen: Bir sunucu çalışma alanı kurun
  - H2: Çalışma zamanı modeli
  - H2: Forum kanalları
  - H2: Etkileşimli bileşenler
  - H2: Erişim denetimi ve yönlendirme
  - H3: Rol tabanlı aracı yönlendirme
  - H2: Yerel komutlar ve komut yetkilendirmesi
  - H2: Özellik ayrıntıları
  - H2: Araçlar ve eylem geçitleri
  - H2: Components v2 kullanıcı arayüzü
  - H2: Ses
  - H3: Ses kanalları
  - H3: Kullanıcıları ses kanalında takip etme
  - H3: Sesli mesajlar
  - H2: Sorun giderme
  - H2: Yapılandırma başvurusu
  - H3: Discord Etkinlikleri
  - H2: Güvenlik ve operasyonlar
  - H2: İlgili

## channels/feishu.md

- Rota: /channels/feishu
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Gelen iletilerin dayanıklılığı
  - H2: Erişim denetimi
  - H3: Doğrudan mesajlar
  - H3: Grup sohbetleri
  - H2: Grup yapılandırma örnekleri
  - H3: Tüm gruplara izin ver, @bahsetme gerekmesin
  - H3: Tüm gruplara izin ver, yine de @bahsetme gerektir
  - H3: Yalnızca belirli gruplara izin ver
  - H3: Bir grup içindeki gönderenleri kısıtla
  - H3: Bot tarafından oluşturulan mesajlar
  - H2: Grup/kullanıcı kimliklerini alma
  - H3: Grup kimlikleri (`chat_id`, biçim: `oc_xxx`)
  - H3: Kullanıcı kimlikleri (`open_id`, biçim: `ou_xxx`)
  - H2: Yaygın komutlar
  - H2: Sorun giderme
  - H3: Bot grup sohbetlerinde yanıt vermiyor
  - H3: Bot mesajları almıyor
  - H3: QR kurulumu Feishu mobil uygulamasında tepki vermiyor
  - H3: App Secret sızdırıldı
  - H2: Gelişmiş yapılandırma
  - H3: Birden çok hesap
  - H3: Mesaj sınırları
  - H3: Akış
  - H3: Kota optimizasyonu
  - H3: Grup oturumu kapsamı ve konu dizileri
  - H3: Feishu çalışma alanı araçları
  - H3: ACP oturumları
  - H4: Kalıcı ACP bağlama
  - H4: Sohbetten ACP başlatma
  - H3: Çok aracılı yönlendirme
  - H2: Kullanıcı başına aracı yalıtımı (Dinamik Aracı Oluşturma)
  - H3: Hızlı kurulum
  - H3: Nasıl çalışır
  - H3: Yapılandırma seçenekleri
  - H3: Oturum kapsamı
  - H3: Tipik çok kullanıcılı dağıtım
  - H3: Doğrulama
  - H3: Notlar
  - H2: Yapılandırma başvurusu
  - H2: Desteklenen mesaj türleri
  - H3: Alma
  - H3: Gönderme
  - H3: Diziler ve yanıtlar
  - H2: İlgili

## channels/googlechat.md

- Rota: /channels/googlechat
- Başlıklar:
  - H2: Yükleme
  - H2: Hızlı kurulum (başlangıç düzeyi)
  - H2: Google Chat'e ekleme
  - H2: Genel URL (yalnızca Webhook)
  - H3: A Seçeneği: Tailscale Funnel (Önerilen)
  - H3: B Seçeneği: Ters Proxy (Caddy)
  - H3: C Seçeneği: Cloudflare Tunnel
  - H2: Nasıl çalışır
  - H3: Gelen iletilerin dayanıklılığı
  - H2: Hedefler
  - H2: Yapılandırmada öne çıkanlar
  - H2: Sorun giderme
  - H3: 405 Yönteme İzin Verilmiyor
  - H3: Diğer sorunlar
  - H2: İlgili

## channels/group-messages.md

- Rota: /channels/group-messages
- Başlıklar:
  - H2: Davranış
  - H2: Yapılandırma örneği (WhatsApp)
  - H3: Etkinleştirme komutu (yalnızca sahip)
  - H2: Kullanım
  - H2: Test / doğrulama
  - H2: Bilinen hususlar
  - H2: İlgili

## channels/groups.md

- Rota: /channels/groups
- Başlıklar:
  - H2: Başlangıç düzeyinde giriş (2 dakika)
  - H2: Görünür yanıtlar
  - H2: Bağlam görünürlüğü ve izin listeleri
  - H2: Oturum anahtarları
  - H2: Örüntü: kişisel doğrudan mesajlar + genel gruplar (tek aracı)
  - H2: Görünen etiketler
  - H2: Grup ilkesi
  - H2: Bahsetme geçidi (varsayılan)
  - H2: Kapsama göre yapılandırılmış bahsetme örüntüleri
  - H2: Grup/kanal aracı kısıtlamaları (isteğe bağlı)
  - H2: Grup izin listeleri
  - H2: Etkinleştirme (yalnızca sahip)
  - H2: Bağlam alanları
  - H2: iMessage'e özgü ayrıntılar
  - H2: WhatsApp sistem istemleri
  - H2: WhatsApp'a özgü ayrıntılar
  - H2: İlgili

## channels/imessage-from-bluebubbles.md

- Rota: /channels/imessage-from-bluebubbles
- Başlıklar:
  - H2: Geçiş kontrol listesi
  - H2: imsg ne yapar
  - H2: Başlamadan önce
  - H2: Yapılandırma dönüşümü
  - H2: Grup kayıt defteri tuzağı
  - H2: Adım adım
  - H2: Bir bakışta eylem denkliği
  - H2: Eşleştirme, oturumlar ve ACP bağlamaları
  - H2: Geri dönüş kanalı yok
  - H2: İlgili

## channels/imessage.md

- Rota: /channels/imessage
- Başlıklar:
  - H2: Hızlı kurulum
  - H2: Gereksinimler ve izinler (macOS)
  - H2: imsg özel API'sini etkinleştirme
  - H3: Kurulum
  - H3: SIP etkin kaldığında
  - H2: Erişim denetimi ve yönlendirme
  - H2: ACP konuşma bağlamaları
  - H2: Dağıtım örüntüleri
  - H2: Medya, parçalara ayırma ve teslim hedefleri
  - H2: Özel API eylemleri
  - H2: Yapılandırma yazma işlemleri
  - H2: Bölünmüş gönderimli doğrudan mesajları birleştirme (tek düzenlemede komut + URL)
  - H2: Köprü veya Gateway yeniden başlatıldıktan sonra gelen iletileri kurtarma
  - H3: Operatörün görebildiği sinyal
  - H3: Geçiş
  - H2: Sorun giderme
  - H2: Yapılandırma başvurusu yönlendirmeleri
  - H2: İlgili

## channels/index.md

- Rota: /channels
- Başlıklar:
  - H2: Desteklenen kanallar
  - H2: Teslim notları
  - H2: Notlar

## channels/irc.md

- Rota: /channels/irc
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Gelen iletilerin dayanıklılığı
  - H2: Bağlantı ayarları
  - H2: Güvenlik varsayılanları
  - H2: Erişim denetimi
  - H3: Yaygın tuzak: allowFrom doğrudan mesajlar içindir, kanallar için değil
  - H2: Yanıt tetikleme (bahsetmeler)
  - H2: Güvenlik notu (genel kanallar için önerilir)
  - H3: Kanaldaki herkes için aynı araçlar
  - H3: Gönderene göre farklı araçlar (sahibin daha fazla yetkisi vardır)
  - H2: NickServ
  - H2: Ortam değişkenleri
  - H2: Sorun giderme
  - H2: İlgili

## channels/line.md

- Rota: /channels/line
- Başlıklar:
  - H2: Yükleme
  - H2: Kurulum
  - H2: Yapılandırma
  - H2: Erişim denetimi
  - H2: Mesaj davranışı
  - H2: Kanal verileri (zengin mesajlar)
  - H2: ACP desteği
  - H2: Giden medya
  - H2: Sorun giderme
  - H2: İlgili

## channels/location.md

- Rota: /channels/location
- Başlıklar:
  - H2: Metin biçimlendirme
  - H2: Bağlam alanları
  - H2: Giden veri yükleri
  - H2: Kanal notları
  - H2: İlgili

## channels/matrix-migration.md

- Rota: /channels/matrix-migration
- Başlıklar:
  - H2: Geçişin otomatik olarak yaptıkları
  - H2: 2026.4'ten eski OpenClaw sürümlerinden yükseltme
  - H2: Önerilen yükseltme akışı
  - H2: Yaygın mesajlar ve anlamları
  - H3: Elle kurtarma mesajları
  - H2: Şifrelenmiş geçmiş yine de geri gelmezse
  - H2: Gelecekteki mesajlar için yeniden başlamak isterseniz
  - H2: İlgili

## channels/matrix-presentation.md

- Rota: /channels/matrix-presentation
- Başlıklar:
  - H2: Olay içeriği
  - H2: Geri dönüş davranışı
  - H2: Desteklenen bloklar
  - H2: Etkileşimler
  - H2: Onay meta verileriyle ilişkisi
  - H2: Medya mesajları

## channels/matrix-push-rules.md

- Rota: /channels/matrix-push-rules
- Başlıklar:
  - H2: Ön koşullar
  - H2: Adımlar
  - H2: Çoklu bot notları
  - H2: Homeserver notları
  - H2: İlgili

## channels/matrix.md

- Rota: /channels/matrix
- Başlıklar:
  - H2: Yükleme
  - H2: Kurulum
  - H3: Etkileşimli kurulum
  - H3: Asgari yapılandırma
  - H3: Otomatik katılma
  - H3: İzin listesi hedef biçimleri
  - H3: Hesap kimliği normalleştirme
  - H3: Önbelleğe alınmış kimlik bilgileri
  - H3: Ortam değişkenleri
  - H2: Yapılandırma örneği
  - H2: Akış önizlemeleri
  - H2: Sesli mesajlar
  - H2: Onay meta verileri
  - H3: Sessiz, sonlandırılmış önizlemeler için kendi sunucunuzdaki anlık bildirim kuralları
  - H2: Botlar arası odalar
  - H2: Şifreleme ve doğrulama
  - H3: Şifrelemeyi etkinleştirme
  - H3: Durum ve güven sinyalleri
  - H3: Bu cihazı kurtarma anahtarıyla doğrulama
  - H3: Çapraz imzalamayı önyükleme veya onarma
  - H3: Oda anahtarı yedeklemesi
  - H3: Doğrulamaları listeleme, isteme ve yanıtlama
  - H3: Çoklu hesap notları
  - H2: Profil yönetimi
  - H2: Diziler
  - H3: Oturum yönlendirme (sessionScope)
  - H3: Yanıt dizileme (threadReplies)
  - H3: Dizi devralma ve eğik çizgi komutları
  - H2: ACP konuşma bağlamaları
  - H3: Dizi bağlama yapılandırması
  - H2: Tepkiler
  - H2: Geçmiş bağlamı
  - H2: Bağlam görünürlüğü
  - H2: Doğrudan mesaj ve oda ilkesi
  - H2: Doğrudan oda onarımı
  - H2: Çalıştırma onayları
  - H2: Eğik çizgi komutları
  - H2: Çoklu hesap
  - H2: Özel/LAN homeserver'ları
  - H2: Matrix trafiğine proxy uygulama
  - H2: Hedef çözümleme
  - H2: Yapılandırma başvurusu
  - H3: Hesap ve bağlantı
  - H3: Şifreleme
  - H3: Erişim ve ilke
  - H3: Yanıt davranışı
  - H3: Tepki ayarları
  - H3: Araçlar ve oda başına geçersiz kılmalar
  - H3: Çalıştırma onayı ayarları
  - H2: İlgili

## channels/mattermost.md

- Rota: /channels/mattermost
- Başlıklar:
  - H2: Kurulum
  - H2: Hızlı kurulum
  - H2: Yerel eğik çizgi komutları
  - H2: Ortam değişkenleri (varsayılan hesap)
  - H2: Sohbet modları
  - H2: İş parçacıkları ve oturumlar
  - H2: Erişim denetimi (DM'ler)
  - H2: Kanallar (gruplar)
  - H2: Giden teslimat hedefleri
  - H2: DM kanalı yeniden denemesi
  - H2: Ön izleme akışı
  - H2: Tepkiler (mesaj aracı)
  - H2: Etkileşimli düğmeler (mesaj aracı)
  - H3: Doğrudan API entegrasyonu (harici betikler)
  - H2: Dizin bağdaştırıcısı
  - H2: Çoklu hesap
  - H2: Sorun giderme
  - H2: İlgili

## channels/msteams.md

- Rota: /channels/msteams
- Başlıklar:
  - H2: Paketle gelen plugin
  - H2: Hızlı kurulum
  - H2: Hedefler
  - H2: Yapılandırma yazma işlemleri
  - H2: Erişim denetimi (DM'ler + gruplar)
  - H3: Nasıl çalışır?
  - H3: Adım 1: Azure Bot oluşturma
  - H3: Adım 2: Kimlik bilgilerini alma
  - H3: Adım 3: Mesajlaşma uç noktasını yapılandırma
  - H3: Adım 4: Teams kanalını etkinleştirme
  - H3: Adım 5: Teams uygulama manifestini oluşturma
  - H3: Adım 6: OpenClaw'ı yapılandırma
  - H3: Adım 7: Gateway'i çalıştırma
  - H2: Birleşik kimlik doğrulama (sertifika ve yönetilen kimlik)
  - H3: Seçenek A: Sertifika tabanlı kimlik doğrulama
  - H3: Seçenek B: Azure Managed Identity
  - H3: AKS Workload Identity kurulumu
  - H3: Kimlik doğrulama türlerinin karşılaştırması
  - H2: Yerel geliştirme (tünelleme)
  - H2: Botu test etme
  - H2: Ortam değişkenleri
  - H2: Üye bilgisi eylemi
  - H2: Geçmiş bağlamı
  - H2: Geçerli Teams RSC izinleri (manifest)
  - H2: Örnek Teams manifesti (sansürlenmiş)
  - H3: Manifest uyarıları (zorunlu alanlar)
  - H3: Mevcut bir uygulamayı güncelleme
  - H2: Yetenekler: yalnızca RSC ve Graph karşılaştırması
  - H3: Yalnızca Teams RSC ile (uygulama yüklü, Graph API izni yok)
  - H3: Teams RSC + Microsoft Graph uygulama izinleriyle
  - H3: RSC ve Graph API karşılaştırması
  - H2: Graph özellikli medya + geçmiş
  - H3: Kanal/grup dosyası kurtarma (graphMediaFallback)
  - H2: Bilinen sınırlamalar
  - H3: Webhook zaman aşımları
  - H3: Teams bulutu ve hizmet URL'si desteği
  - H3: Biçimlendirme
  - H2: Yapılandırma
  - H2: Yönlendirme ve oturumlar
  - H2: Yanıt stili: iş parçacıkları ve gönderiler
  - H3: Çözümleme önceliği
  - H3: İş parçacığı bağlamını koruma
  - H2: Ekler ve görseller
  - H2: Grup sohbetlerinde dosya gönderme
  - H3: Grup sohbetleri neden SharePoint gerektirir?
  - H3: Kurulum
  - H3: Paylaşım davranışı
  - H3: Geri dönüş davranışı
  - H3: Dosyaların depolandığı konum
  - H2: Anketler (Adaptive Cards)
  - H2: Sunum kartları
  - H2: Hedef biçimleri
  - H2: Proaktif mesajlaşma
  - H2: Ekip ve Kanal Kimlikleri (Yaygın Yanılgı)
  - H2: Özel kanallar
  - H2: Sorun giderme
  - H3: Yaygın sorunlar
  - H3: Manifest yükleme hataları
  - H3: RSC izinleri çalışmıyor
  - H2: Başvurular
  - H2: İlgili

## channels/nextcloud-talk.md

- Rota: /channels/nextcloud-talk
- Başlıklar:
  - H2: Kurulum
  - H2: Hızlı kurulum (başlangıç düzeyi)
  - H2: Notlar
  - H2: Erişim denetimi (DM'ler)
  - H2: Odalar (gruplar)
  - H2: Yetenekler
  - H2: Yapılandırma başvurusu (Nextcloud Talk)
  - H2: İlgili

## channels/nostr.md

- Rota: /channels/nostr
- Başlıklar:
  - H2: Kurulum
  - H3: Etkileşimsiz kurulum
  - H2: Hızlı kurulum
  - H2: Yapılandırma başvurusu
  - H2: Profil meta verileri
  - H2: Erişim denetimi
  - H3: DM politikaları
  - H3: İzin verilenler listesi örneği
  - H2: Anahtar biçimleri
  - H2: Aktarıcılar
  - H2: Protokol desteği
  - H2: Test
  - H3: Yerel aktarıcı
  - H3: Manuel test
  - H2: Sorun giderme
  - H3: Mesajlar alınmıyor
  - H3: Yanıtlar gönderilmiyor
  - H3: Yinelenen yanıtlar
  - H2: Güvenlik
  - H2: Sınırlamalar (MVP)
  - H2: İlgili

## channels/pairing.md

- Rota: /channels/pairing
- Başlıklar:
  - H2: 1) DM eşleştirmesi (gelen sohbet erişimi)
  - H3: Control UI'dan onaylama
  - H3: CLI'dan onaylama
  - H3: Yeniden kullanılabilir gönderici grupları
  - H3: Durumun bulunduğu yer
  - H2: 2) Node cihaz eşleştirmesi (iOS/Android/macOS/başsız Node'lar)
  - H3: Control UI'dan eşleştirme (önerilen)
  - H3: Telegram aracılığıyla eşleştirme
  - H3: Bir Node cihazını onaylama
  - H3: İsteğe bağlı güvenilir CIDR tabanlı otomatik Node onayı
  - H3: Node eşleştirme durumu depolaması
  - H3: Notlar
  - H2: İlgili belgeler

## channels/qa-channel.md

- Rota: /channels/qa-channel
- Başlıklar:
  - H2: Ne yapar?
  - H2: Yapılandırma
  - H2: Çalıştırıcılar
  - H2: İlgili

## channels/qqbot.md

- Rota: /channels/qqbot
- Başlıklar:
  - H2: Kurulum
  - H2: Kurulum
  - H2: Gelen iletilerin dayanıklılığı
  - H2: Yapılandırma
  - H3: Akış
  - H3: Erişim politikası
  - H3: Çoklu hesap kurulumu
  - H3: Grup sohbetleri
  - H3: Ses (STT / TTS)
  - H2: Hedef biçimleri
  - H2: Eğik çizgi komutları
  - H2: Medya ve depolama
  - H2: Sorun giderme
  - H2: İlgili

## channels/raft.md

- Rota: /channels/raft
- Başlıklar:
  - H2: Kurulum
  - H2: Ön koşullar
  - H2: Yapılandırma
  - H2: Nasıl çalışır?
  - H2: Doğrulama
  - H2: Sorun giderme
  - H2: Başvurular

## channels/reef.md

- Rota: /channels/reef
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Aracı güdümlü kurulum
  - H2: Yapılandırma
  - H2: Arkadaş ekleme
  - H2: Gönderme ve alma
  - H2: Korumalar ve sahip incelemesi
  - H2: Sorun giderme

## channels/signal.md

- Rota: /channels/signal
- Başlıklar:
  - H2: Numara modeli (önce bunu okuyun)
  - H2: Kurulum
  - H2: Hızlı kurulum
  - H2: Nedir?
  - H2: Kurulum yolu A: mevcut Signal hesabını bağlama (QR)
  - H2: Kurulum yolu B: ayrılmış bot numarasını kaydetme (SMS, Linux)
  - H2: Harici yerel artalan süreci modu
  - H2: Konteyner modu (bbernhard/signal-cli-rest-api)
  - H2: Erişim denetimi (DM'ler + gruplar)
  - H2: Nasıl çalışır? (davranış)
  - H2: Medya + sınırlar
  - H2: Yazma göstergeleri + okundu bilgileri
  - H2: Yaşam döngüsü durumu tepkileri
  - H2: Tepkiler (mesaj aracı)
  - H2: Onay tepkileri
  - H2: Soru tepkileri
  - H2: Teslimat hedefleri (CLI/cron)
  - H2: Takma adlar
  - H2: Sorun giderme
  - H2: Güvenlik notları
  - H2: Yapılandırma başvurusu (Signal)
  - H2: İlgili

## channels/slack.md

- Rota: /channels/slack
- Başlıklar:
  - H2: Aktarım yöntemi seçme
  - H3: Aktarıcı modu
  - H3: Enterprise Grid kuruluş genelinde kurulumlar
  - H4: Socket Mode
  - H4: HTTP Request URLs
  - H2: Kurulum
  - H2: Hızlı kurulum
  - H2: Kullanıcı kimliği (gerçek bir kişi olarak gönderme)
  - H2: Socket Mode aktarım ayarları
  - H2: Manifest ve kapsam denetim listesi
  - H3: Ek manifest ayarları
  - H2: Token modeli
  - H2: Eylemler ve geçitler
  - H2: Erişim denetimi ve yönlendirme
  - H2: İş parçacıkları, oturumlar ve yanıt etiketleri
  - H2: Alındı tepkileri
  - H3: Emoji (ackReaction)
  - H3: Kapsam (messages.ackReactionScope)
  - H2: Metin akışı
  - H2: Yazma tepkisi geri dönüşü
  - H2: Sesli giriş
  - H2: Medya, parçalara ayırma ve teslimat
  - H2: Komutlar ve eğik çizgi davranışı
  - H2: Yerel grafikler
  - H2: Yerel tablolar
  - H2: Etkileşimli yanıtlar
  - H3: Plugin'e ait modal gönderimleri
  - H2: Slack'te yerel onaylar
  - H2: Olaylar ve operasyonel davranış
  - H3: İletişim durumu olayları
  - H2: Yapılandırma başvurusu
  - H2: Sorun giderme
  - H2: Ek medyası başvurusu
  - H3: Desteklenen medya türleri
  - H3: Gelen ileti işlem hattı
  - H3: İş parçacığı kökü ek devralması
  - H3: Birden çok eki işleme
  - H3: Boyut, indirme ve model sınırları
  - H3: Bilinen sınırlar
  - H3: İlgili belgeler
  - H2: İlgili

## channels/sms.md

- Rota: /channels/sms
- Başlıklar:
  - H2: Başlamadan önce
  - H2: Hızlı Kurulum
  - H2: Yapılandırma Örnekleri
  - H3: Yapılandırma dosyası
  - H3: Ortam değişkenleri
  - H3: SecretRef kimlik doğrulama token'ı
  - H3: Messaging Service göndericisi
  - H3: Varsayılan giden hedef
  - H2: Erişim denetimi
  - H2: SMS gönderme
  - H2: Kurulumu Doğrulama
  - H3: macOS iMessage/SMS üzerinden uçtan uca test
  - H2: Webhook güvenliği
  - H2: Çoklu hesap yapılandırması
  - H2: Sorun giderme
  - H3: Twilio 403 döndürüyor veya OpenClaw Webhook'u reddediyor
  - H3: Eşleştirme isteği görünmüyor
  - H3: Giden gönderimler başarısız oluyor
  - H3: Mesajlar geliyor ancak aracı yanıt vermiyor

## channels/synology-chat.md

- Rota: /channels/synology-chat
- Başlıklar:
  - H2: Kurulum
  - H2: Hızlı kurulum
  - H2: Gelen iletilerin dayanıklılığı
  - H2: Ortam değişkenleri
  - H2: DM politikası ve erişim denetimi
  - H2: Giden teslimat
  - H2: Çoklu hesap
  - H2: Güvenlik notları
  - H2: Sorun giderme
  - H2: İlgili

## channels/telegram.md

- Rota: /channels/telegram
- Başlıklar:
  - H2: Hızlı kurulum
  - H2: Telegram tarafı ayarları
  - H2: Kontrol paneli Mini Uygulaması
  - H2: Erişim denetimi ve etkinleştirme
  - H3: Grup botu kimliği
  - H2: Çalışma zamanı davranışı
  - H2: Özellik referansı
  - H2: Hata yanıtı denetimleri
  - H2: Sorun giderme
  - H2: Yapılandırma referansı
  - H2: İlgili

## channels/tlon.md

- Rota: /channels/tlon
- Başlıklar:
  - H2: Birlikte gelen plugin
  - H2: Kurulum
  - H2: Gelen iletilerin dayanıklılığı
  - H2: Özel/LAN gemileri
  - H2: Grup kanalları
  - H2: Erişim denetimi
  - H2: Sahip ve onay sistemi
  - H2: Otomatik kabul ayarları
  - H2: Urbit ayar deposu üzerinden çalışırken yeniden yükleme
  - H2: Teslim hedefleri (CLI/cron)
  - H2: Birlikte gelen beceri
  - H2: Yetenekler
  - H2: Sorun giderme
  - H2: Yapılandırma referansı
  - H2: Notlar
  - H2: İlgili

## channels/troubleshooting.md

- Rota: /channels/troubleshooting
- Başlıklar:
  - H2: Komut sıralaması
  - H2: Güncellemeden sonra
  - H2: WhatsApp
  - H3: WhatsApp hata belirtileri
  - H2: Telegram
  - H3: Telegram hata belirtileri
  - H2: Discord
  - H3: Discord hata belirtileri
  - H2: Slack
  - H3: Slack hata belirtileri
  - H2: iMessage
  - H3: iMessage hata belirtileri
  - H2: Signal
  - H3: Signal hata belirtileri
  - H2: QQ Bot
  - H3: QQ Bot hata belirtileri
  - H2: Matrix
  - H3: Matrix hata belirtileri
  - H2: İlgili

## channels/twitch.md

- Rota: /channels/twitch
- Başlıklar:
  - H2: Yükleme
  - H2: Hızlı kurulum
  - H2: Nedir?
  - H2: Gelen iletilerin dayanıklılığı
  - H2: Token yenileme (isteğe bağlı)
  - H2: Çoklu hesap desteği
  - H2: Erişim denetimi
  - H2: Sorun giderme
  - H2: Yapılandırma
  - H3: Hesap yapılandırması
  - H3: Sağlayıcı seçenekleri
  - H2: Araç eylemleri
  - H2: Güvenlik ve operasyonlar
  - H2: Sınırlar
  - H2: İlgili

## channels/wechat.md

- Rota: /channels/wechat
- Başlıklar:
  - H2: Adlandırma
  - H2: Nasıl çalışır?
  - H2: Yükleme
  - H2: Oturum açma
  - H2: Erişim denetimi
  - H2: Uyumluluk
  - H2: Yardımcı süreç
  - H2: Sorun giderme
  - H2: İlgili belgeler

## channels/whatsapp.md

- Rota: /channels/whatsapp
- Başlıklar:
  - H2: Yükleme
  - H2: Hızlı kurulum
  - H2: Dağıtım modelleri
  - H2: Çalışma zamanı modeli
  - H2: MeowCaller ile mevcut istekte bulunan kişiyi arama (deneysel)
  - H2: Onay istemleri
  - H2: Soru tepkileri
  - H2: Plugin kancaları ve gizlilik
  - H2: Erişim denetimi ve etkinleştirme
  - H2: Yapılandırılmış ACP bağlamaları
  - H2: Kişisel numara ve kendiyle sohbet davranışı
  - H2: İleti normalleştirme ve bağlam
  - H2: Teslim, parçalara ayırma ve medya
  - H2: Yanıtta alıntılama
  - H2: Tepki düzeyi
  - H2: Alındı tepkileri
  - H2: Yaşam döngüsü durumu tepkileri
  - H2: Çoklu hesap ve kimlik bilgileri
  - H2: Araçlar, eylemler ve yapılandırma yazımları
  - H2: Sorun giderme
  - H2: Sistem istemleri
  - H2: Yapılandırma referansı yönlendirmeleri
  - H2: İlgili

## channels/yuanbao.md

- Rota: /channels/yuanbao
- Başlıklar:
  - H2: Hızlı başlangıç
  - H3: Etkileşimli kurulum (alternatif)
  - H2: Erişim denetimi
  - H3: Doğrudan iletiler
  - H3: Grup sohbetleri
  - H2: Yapılandırma örnekleri
  - H2: Yaygın komutlar
  - H2: Sorun giderme
  - H2: Gelişmiş yapılandırma
  - H3: Birden fazla hesap
  - H3: İleti sınırları
  - H3: Akış
  - H3: Grup sohbeti geçmişi bağlamı
  - H3: Yanıt verme modu
  - H3: Markdown ipucu ekleme
  - H3: Hata ayıklama modu
  - H3: Çoklu ajan yönlendirmesi
  - H2: Yapılandırma referansı
  - H2: Desteklenen ileti türleri
  - H2: İlgili

## channels/zalo.md

- Rota: /channels/zalo
- Başlıklar:
  - H2: Birlikte gelen plugin
  - H2: Hızlı kurulum
  - H2: Nedir?
  - H2: Nasıl çalışır?
  - H2: Sınırlar
  - H2: Erişim denetimi
  - H3: Doğrudan iletiler
  - H3: Gruplar
  - H2: Uzun yoklama ile Webhook karşılaştırması
  - H2: Desteklenen ileti türleri
  - H2: Yetenekler
  - H2: Teslim hedefleri (CLI/cron)
  - H2: Sorun giderme
  - H2: Yapılandırma referansı
  - H2: İlgili

## channels/zaloclawbot.md

- Rota: /channels/zaloclawbot
- Başlıklar:
  - H2: Uyumluluk
  - H2: Ön koşullar
  - H2: İlk kurulumla yükleme (önerilen)
  - H2: Elle yükleme
  - H3: 1. Plugini yükleyin
  - H3: 2. Yapılandırmada plugini etkinleştirin
  - H3: 3. QR kodu oluşturun ve oturum açın
  - H3: 4. Gateway'i yeniden başlatın
  - H2: Nasıl çalışır?
  - H2: Arka planda
  - H2: Sorun giderme
  - H2: İlgili

## channels/zalouser.md

- Rota: /channels/zalouser
- Başlıklar:
  - H2: Yükleme
  - H2: Hızlı kurulum
  - H2: Nedir?
  - H2: Adlandırma
  - H2: Kimlikleri bulma (dizin)
  - H2: Sınırlar
  - H2: Gelen iletilerin dayanıklılığı
  - H2: Erişim denetimi (DM'ler)
  - H2: Grup erişimi (isteğe bağlı)
  - H3: Grupta bahsetme geçidi
  - H2: Çoklu hesap
  - H2: Ortam değişkenleri
  - H2: Yazıyor göstergesi, tepkiler ve teslim alındıları
  - H2: Sorun giderme
  - H2: İlgili

## ci.md

- Rota: /ci
- Başlıklar:
  - H2: İşlem hattına genel bakış
  - H2: Hızlı başarısız olma sırası
  - H2: PR bağlamı ve kanıt
  - H2: Kapsam ve yönlendirme
  - H2: ClawSweeper etkinliği yönlendirmesi
  - H2: Elle tetiklemeler
  - H2: Çalıştırıcılar
  - H2: Çalıştırıcı kayıt bütçesi
  - H2: Yüzey mandalları
  - H2: Yerel eşdeğerler
  - H2: OpenClaw Performansı
  - H2: Tam Sürüm Doğrulaması
  - H2: Canlı ve E2E parçaları
  - H2: Paket Kabulü
  - H3: İşler
  - H3: Aday kaynaklar
  - H3: Test paketi profilleri
  - H3: Eski sürüm uyumluluk aralıkları
  - H3: Örnekler
  - H2: Yükleme duman testi
  - H2: Yerel Docker E2E
  - H3: Ayarlanabilir öğeler
  - H3: Yeniden kullanılabilir canlı/E2E iş akışı
  - H3: Sürüm yolu parçaları
  - H2: Plugin Ön Sürümü
  - H2: QA Laboratuvarı
  - H2: CodeQL
  - H3: Güvenlik kategorileri
  - H3: Platforma özgü güvenlik parçaları
  - H3: Kritik Kalite kategorileri
  - H2: Bakım iş akışları
  - H3: Belge Ajanı
  - H3: Test Performansı Ajanı
  - H3: Birleştirme Sonrası Yinelenen PR'ler
  - H2: Yerel denetim geçitleri ve değişiklik yönlendirmesi
  - H3: Yapılandırma temel çizgisi sayısı mandalı
  - H2: Testbox doğrulaması
  - H2: İlgili

## clawhub/cli.md

- Rota: /clawhub/cli
- Başlıklar:
  - H1: ClawHub CLI
  - H2: Keşfetme ve yükleme
  - H3: Sürüm güveni
  - H2: Yayımlama ve bakım
  - H2: İlgili

## clawhub/publishing.md

- Rota: /clawhub/publishing
- Başlıklar:
  - H1: ClawHub'da yayımlama
  - H2: Sahipler
  - H2: Skills
  - H2: Pluginler
  - H2: Sürüm akışı
  - H2: SSS
  - H3: Paket kapsamı seçilen sahiple eşleşmelidir

## cli/acp.md

- Rota: /cli/acp
- Başlıklar:
  - H2: Bu ne değildir?
  - H2: Uyumluluk matrisi
  - H2: Bilinen sınırlamalar
  - H2: Kullanım
  - H2: ACP istemcisi (hata ayıklama)
  - H2: Protokol duman testi
  - H2: Bunun kullanımı
  - H2: Ajanları seçme
  - H2: acpx üzerinden kullanım (Codex, Claude, diğer ACP istemcileri)
  - H2: Zed düzenleyicisi kurulumu
  - H2: Oturum eşleme
  - H2: Seçenekler
  - H3: acp istemcisi seçenekleri
  - H2: İlgili

## cli/agent.md

- Rota: /cli/agent
- Başlıklar:
  - H1: openclaw agent
  - H2: Seçenekler
  - H2: Örnekler
  - H2: Notlar
  - H2: JSON teslim durumu
  - H2: İlgili

## cli/agents.md

- Rota: /cli/agents
- Başlıklar:
  - H1: openclaw agents
  - H2: Örnekler
  - H2: Komut yüzeyi
  - H3: agents list
  - H3: `agents add [name]`
  - H3: agents bindings
  - H3: agents bind
  - H3: agents unbind
  - H3: agents set-identity
  - H3: agents delete &lt;id&gt;
  - H2: Yönlendirme bağlamaları
  - H3: --bind biçimi
  - H3: Bağlama kapsamı davranışı
  - H2: Kimlik dosyaları
  - H2: Kimliği ayarlama
  - H2: İlgili

## cli/approvals.md

- Rota: /cli/approvals
- Başlıklar:
  - H1: openclaw approvals
  - H2: openclaw exec-policy
  - H2: Yaygın komutlar
  - H2: Bekleyen onaylar
  - H2: Onayları bir dosyadan değiştirme
  - H2: "Asla sorma" / YOLO örneği
  - H2: İzin listesi yardımcıları
  - H2: Yaygın seçenekler
  - H2: Notlar
  - H2: İlgili

## cli/attach.md

- Rota: /cli/attach
- Başlıklar: yok

## cli/audit.md

- Rota: /cli/audit
- Başlıklar:
  - H1: openclaw audit
  - H2: Filtreler
  - H2: Kaydedilen olaylar
  - H2: Gateway RPC
  - H2: İlgili

## cli/backup.md

- Rota: /cli/backup
- Başlıklar:
  - H1: openclaw backup
  - H2: Notlar
  - H2: SQLite anlık görüntüleri
  - H3: Doğrulama ve geri yükleme
  - H2: Yedeklenenler
  - H2: Geçersiz yapılandırma davranışı
  - H2: Boyut ve performans
  - H2: İlgili konular

## cli/browser.md

- Rota: /cli/browser
- Başlıklar:
  - H1: openclaw browser
  - H2: Yaygın bayraklar
  - H2: Hızlı başlangıç (yerel)
  - H2: Hızlı sorun giderme
  - H2: Yaşam döngüsü
  - H2: Komut eksikse
  - H2: Profiller
  - H2: Sekmeler
  - H2: Anlık görüntü / ekran görüntüsü / eylemler
  - H2: Durum ve depolama
  - H2: Hata ayıklama
  - H2: MCP üzerinden mevcut Chrome
  - H2: Uzak tarayıcı denetimi (Node ana makine proxy'si)
  - H2: İlgili konular

## cli/channels.md

- Rota: /cli/channels
- Başlıklar:
  - H1: openclaw channels
  - H2: Yaygın komutlar
  - H2: Durum / yetenekler / çözümleme / günlükler
  - H2: Gelen işlenemeyen iletiler
  - H2: Hesap ekleme / kaldırma
  - H2: Oturum açma ve kapatma (etkileşimli)
  - H2: Sorun giderme
  - H2: Yetenek yoklaması
  - H2: Adları kimliklere çözümleme
  - H2: İlgili konular

## cli/clawbot.md

- Rota: /cli/clawbot
- Başlıklar:
  - H1: openclaw clawbot
  - H2: Geçiş
  - H2: İlgili konular

## cli/claws.md

- Rota: /cli/claws
- Başlıklar:
  - H1: openclaw claws
  - H2: Claw paketi oluşturma
  - H2: İnceleme ve önizleme
  - H2: Yüklü durumu inceleme
  - H2: Yüklü bir Claw'ı güncelleme
  - H2: Yüklü bir Claw'ı kaldırma
  - H2: Yüklü bir ajanı dışa aktarma
  - H2: Komut başvurusu
  - H2: Ayrıca bkz.

## cli/commitments.md

- Rota: /cli/commitments
- Başlıklar:
  - H2: Kullanım
  - H2: Seçenekler
  - H2: Örnekler
  - H2: Çıktı
  - H2: İlgili konular

## cli/completion.md

- Rota: /cli/completion
- Başlıklar:
  - H1: openclaw completion
  - H2: Kullanım
  - H2: Seçenekler
  - H2: Yükleme akışı
  - H2: Notlar
  - H2: İlgili konular

## cli/config.md

- Rota: /cli/config
- Başlıklar:
  - H2: Kök seçenekleri
  - H2: Örnekler
  - H3: Yollar
  - H3: config get
  - H3: config file
  - H3: config schema
  - H3: config validate
  - H2: Değerler
  - H2: config set kipleri
  - H3: Sağlayıcı oluşturucu bayrakları
  - H2: config patch
  - H2: Deneme çalıştırması
  - H3: JSON çıktı biçimi
  - H2: Değişiklikleri uygulama
  - H2: Yazma güvenliği
  - H2: Onarım döngüsü
  - H2: İlgili konular

## cli/configure.md

- Rota: /cli/configure
- Başlıklar:
  - H1: openclaw configure
  - H2: Seçenekler
  - H2: Model bölümü
  - H2: Web bölümü
  - H2: Diğer notlar
  - H2: İlgili konular

## cli/crestodian.md

- Rota: /cli/crestodian
- Başlıklar: yok

## cli/cron.md

- Rota: /cli/cron
- Başlıklar:
  - H1: openclaw cron
  - H2: Hızlıca işler oluşturma
  - H2: Oturumlar
  - H2: Teslimat
  - H3: Teslimat sahipliği
  - H3: Hata teslimatı
  - H2: Zamanlama
  - H3: Tek seferlik işler
  - H3: Yinelenen işler
  - H3: Manuel çalıştırmalar
  - H2: Modeller
  - H3: Yalıtılmış Cron modeli önceliği
  - H3: Hızlı kip
  - H3: Canlı model değiştirme yeniden denemeleri
  - H2: Çalıştırma çıktısı ve retler
  - H3: Eski onayların bastırılması
  - H3: Sessiz belirteçlerin bastırılması
  - H3: Yapılandırılmış retler
  - H2: Saklama
  - H2: Eski işleri taşıma
  - H2: Yaygın düzenlemeler
  - H2: Yaygın yönetici komutları
  - H2: İlgili konular

## cli/daemon.md

- Rota: /cli/daemon
- Başlıklar:
  - H1: openclaw daemon
  - H2: Kullanım
  - H2: Alt komutlar ve seçenekler
  - H2: Notlar
  - H2: İlgili konular

## cli/dashboard.md

- Rota: /cli/dashboard
- Başlıklar:
  - H1: openclaw dashboard
  - H2: Makine tarafından okunabilir çıktı
  - H2: İlgili konular

## cli/devices.md

- Rota: /cli/devices
- Başlıklar:
  - H1: openclaw devices
  - H2: Yaygın seçenekler
  - H2: Komutlar
  - H3: openclaw devices list
  - H3: `openclaw devices approve [requestId] [--latest]`
  - H3: openclaw devices reject &lt;requestId&gt;
  - H3: openclaw devices remove &lt;deviceId&gt;
  - H3: openclaw devices rename --device &lt;id&gt; --name &lt;label&gt;
  - H3: `openclaw devices clear --yes [--pending]`
  - H3: `openclaw devices rotate --device &lt;id&gt; --role &lt;role&gt; [--scope &lt;scope...&gt;]`
  - H3: openclaw devices revoke --device &lt;id&gt; --role &lt;role&gt;
  - H2: Notlar
  - H2: Belirteç sapmasını kurtarma kontrol listesi
  - H2: Paperclip / `openclaw_gateway` ilk çalıştırma onayı
  - H2: İlgili konular

## cli/directory.md

- Rota: /cli/directory
- Başlıklar:
  - H1: openclaw directory
  - H2: Yaygın bayraklar
  - H2: Notlar
  - H2: Sonuçları ileti gönderimiyle kullanma
  - H2: Kanala göre kimlik biçimleri
  - H2: Kendi hesabı ("me")
  - H2: Eşler (kişiler/kullanıcılar)
  - H2: Gruplar
  - H2: İlgili konular

## cli/dns.md

- Rota: /cli/dns
- Başlıklar:
  - H1: openclaw dns
  - H2: dns setup
  - H2: İlgili konular

## cli/docs.md

- Rota: /cli/docs
- Başlıklar:
  - H1: openclaw docs
  - H2: Kullanım
  - H2: Örnekler
  - H2: Çalışma biçimi
  - H2: Çıktı
  - H2: Çıkış kodları
  - H2: İlgili konular

## cli/doctor.md

- Rota: /cli/doctor
- Başlıklar:
  - H1: openclaw doctor
  - H2: Duruşlar
  - H2: Örnekler
  - H2: Seçenekler
  - H2: Lint kipi
  - H2: Yapılandırılmış sistem durumu denetimleri
  - H2: Denetim seçimi
  - H2: Yükseltme sonrası kipi
  - H2: Eski durum geçişi
  - H2: Paylaşılan durum SQLite Compaction
  - H2: Oturum SQLite geçişi
  - H3: Oturum SQLite geçişinden sonra sürüm düşürme
  - H2: Notlar
  - H2: macOS: launchctl ortam değişkeni geçersiz kılmaları
  - H2: İlgili konular

## cli/fleet.md

- Rota: /cli/fleet
- Başlıklar:
  - H1: openclaw fleet
  - H2: Hızlı başlangıç
  - H2: Kiracı kimlikleri
  - H2: fleet create
  - H3: Oluşturma seçenekleri
  - H3: Özet ile sabitleme
  - H3: Disk sınırları
  - H3: Çıkış trafiği politikası
  - H2: fleet list
  - H2: fleet status
  - H2: fleet logs
  - H2: fleet start, fleet stop ve fleet restart
  - H2: fleet upgrade
  - H2: fleet backup ve fleet restore
  - H2: fleet doctor
  - H2: fleet rm
  - H2: Depolama ve kapsayıcı düzeni
  - H2: Güvenlik profili
  - H2: Belirteç işleme
  - H2: İlgili konular

## cli/flows.md

- Rota: /cli/flows
- Başlıklar:
  - H1: openclaw tasks flow
  - H2: Alt komutlar
  - H3: Durum filtresi değerleri
  - H2: Örnekler
  - H2: İlgili konular

## cli/gateway.md

- Rota: /cli/gateway
- Başlıklar:
  - H2: Gateway'i çalıştırma
  - H3: Seçenekler
  - H2: Gateway'i yeniden başlatma
  - H3: Harici yöneticiler
  - H3: Gateway profil oluşturma
  - H2: Çalışan bir Gateway'i sorgulama
  - H3: gateway health
  - H3: gateway usage-cost
  - H3: gateway stability
  - H3: gateway diagnostics export
  - H3: gateway status
  - H3: gateway probe
  - H4: SSH üzerinden uzak erişim (Mac uygulamasıyla eşdeğer)
  - H3: gateway call &lt;method&gt;
  - H2: Gateway hizmetini yönetme
  - H3: Bir sarmalayıcıyla yükleme
  - H2: Gateway'leri keşfetme (Bonjour)
  - H3: gateway discover
  - H2: İlgili konular

## cli/health.md

- Rota: /cli/health
- Başlıklar:
  - H1: openclaw health
  - H2: Seçenekler
  - H2: Davranış
  - H2: İlgili konular

## cli/hooks.md

- Rota: /cli/hooks
- Başlıklar:
  - H1: openclaw hooks
  - H2: Kancaları listeleme
  - H2: Kanca bilgilerini alma
  - H2: Uygunluğu denetleme
  - H2: Bir kancayı etkinleştirme
  - H2: Bir kancayı devre dışı bırakma
  - H2: Kanca paketlerini yükleme ve güncelleme
  - H2: Paketlenmiş kancalar
  - H3: command-logger günlük dosyası
  - H2: Notlar
  - H2: İlgili konular

## cli/index.md

- Rota: /cli
- Başlıklar:
  - H2: Komut sayfaları
  - H2: Genel bayraklar
  - H2: Çıktı kipleri
  - H2: Renk paleti
  - H2: Komut ağacı
  - H2: Sohbet eğik çizgi komutları
  - H2: Kullanım izleme
  - H2: İlgili konular

## cli/infer.md

- Rota: /cli/infer
- Başlıklar:
  - H2: infer'ı bir beceriye dönüştürme
  - H2: Komut ağacı
  - H2: Yaygın görevler
  - H2: Davranış
  - H2: Model
  - H2: Görsel
  - H2: Ses
  - H2: TTS
  - H2: Video
  - H2: Web
  - H2: Gömme
  - H2: JSON çıktısı
  - H2: Yaygın hatalar
  - H2: İlgili konular

## cli/logs.md

- Rota: /cli/logs
- Başlıklar:
  - H1: openclaw logs
  - H2: Seçenekler
  - H2: Paylaşılan Gateway RPC seçenekleri
  - H2: Örnekler
  - H2: Geri dönüş ve kurtarma davranışı
  - H2: İlgili konular

## cli/mcp.md

- Rota: /cli/mcp
- Başlıklar:
  - H2: Doğru MCP yolunu seçme
  - H2: MCP sunucusu olarak OpenClaw
  - H3: serve ne zaman kullanılmalı
  - H3: Nasıl çalışır
  - H3: İstemci modu seçme
  - H3: serve tarafından kullanıma sunulanlar
  - H3: Kullanım
  - H3: Köprü araçları
  - H3: Olay modeli
  - H3: Claude kanal bildirimleri
  - H3: MCP istemci yapılandırması
  - H3: Seçenekler
  - H3: Güvenlik ve güven sınırı
  - H3: Test etme
  - H3: Sorun giderme
  - H2: MCP istemci kayıt defteri olarak OpenClaw
  - H3: Kaydedilmiş MCP sunucusu tanımları
  - H3: Yaygın sunucu tarifleri
  - H3: JSON çıktı biçimleri
  - H3: Stdio aktarımı
  - H3: SSE / HTTP aktarımı
  - H3: OAuth iş akışı
  - H3: Akış destekli HTTP aktarımı
  - H2: Denetim kullanıcı arayüzü
  - H2: MCP Uygulamaları
  - H2: Mevcut sınırlar
  - H2: İlgili içerikler

## cli/memory.md

- Rota: /cli/memory
- Başlıklar:
  - H1: openclaw memory
  - H2: memory status
  - H2: memory index
  - H2: memory search
  - H2: memory promote
  - H2: memory promote-explain
  - H2: memory rem-harness
  - H2: memory rem-backfill
  - H2: Dreaming
  - H2: SecretRef Gateway bağımlılığı
  - H2: İlgili içerikler

## cli/message.md

- Rota: /cli/message
- Başlıklar:
  - H1: openclaw message
  - H2: Kanal seçimi
  - H2: Hedef biçimleri (-t, --target)
  - H2: Yaygın bayraklar
  - H2: SecretRef çözümleme
  - H2: Eylemler
  - H3: Çekirdek
  - H3: Gönderme
  - H3: Anket
  - H3: İleti dizileri
  - H3: Emojiler
  - H3: Çıkartmalar
  - H3: Roller, kanallar, ses ve etkinlikler (Discord)
  - H3: Moderasyon (Discord)
  - H3: Yayın
  - H2: İlgili içerikler

## cli/migrate.md

- Rota: /cli/migrate
- Başlıklar:
  - H1: openclaw migrate
  - H2: Komutlar
  - H2: Güvenlik modeli
  - H2: Claude sağlayıcısı
  - H3: Claude'un içe aktardıkları
  - H3: Arşiv ve manuel inceleme durumu
  - H2: Codex sağlayıcısı
  - H3: Codex'in içe aktardıkları
  - H3: Manuel incelemeye tabi Codex durumu
  - H2: Hermes sağlayıcısı
  - H3: Hermes'in içe aktardıkları
  - H3: Desteklenen .env anahtarları
  - H3: Yalnızca arşivlenen durum
  - H3: Uygulama sonrasında
  - H2: Plugin sözleşmesi
  - H2: İlk katılım entegrasyonu
  - H2: İlgili içerikler

## cli/models.md

- Rota: /cli/models
- Başlıklar:
  - H1: openclaw models
  - H2: Yaygın komutlar
  - H3: Durum
  - H3: Listeleme
  - H3: Varsayılan modeli / görüntü modelini ayarlama
  - H3: Tarama
  - H2: Takma adlar
  - H2: Geri dönüşler
  - H2: Kimlik doğrulama profilleri
  - H2: İlgili içerikler

## cli/node.md

- Rota: /cli/node
- Başlıklar:
  - H1: openclaw node
  - H2: Neden bir Node ana makinesi kullanılmalı?
  - H2: Tarayıcı proxy'si (sıfır yapılandırma)
  - H2: Çalıştırma (ön planda)
  - H2: Node ana makinesi için Gateway kimlik doğrulaması
  - H2: Hizmet (arka planda)
  - H2: Eşleştirme
  - H3: Kimlik ve eşleştirme durumu
  - H2: Yürütme onayları
  - H2: İlgili içerikler

## cli/nodes.md

- Rota: /cli/nodes
- Başlıklar:
  - H1: openclaw nodes
  - H2: Durum
  - H2: Eşleştirme
  - H2: Çağırma
  - H2: Bildirim, anlık iletim, konum ve ekran
  - H2: İlgili içerikler

## cli/onboard.md

- Rota: /cli/onboard
- Başlıklar:
  - H1: openclaw onboard
  - H2: Örnekler
  - H2: Kılavuzlu akış
  - H2: Sıfırlama
  - H2: Yerel ayar
  - H2: Etkileşimsiz kurulum
  - H3: Gateway kimlik doğrulaması (etkileşimsiz)
  - H3: Yerel Gateway durumu
  - H3: Etkileşimli referans modu
  - H3: Z.AI uç noktası seçenekleri
  - H2: Ek etkileşimsiz bayraklar
  - H2: Sağlayıcıları önceden filtreleme
  - H2: Web araması takipleri
  - H2: Diğer davranışlar
  - H2: Yaygın takip komutları

## cli/openclaw.md

- Rota: /cli/openclaw
- Başlıklar:
  - H1: openclaw setup
  - H2: Ne zaman başlar
  - H2: OpenClaw'un gösterdikleri
  - H2: Örnekler
  - H2: İşlemler ve onay
  - H3: Değişiklik geçmişi
  - H3: Maskelenmiş kanal kurulumuna geçme
  - H2: Kurulum önyüklemesi
  - H2: Yapay zekâ konuşması
  - H3: CLI çalıştırma ortamının güven modeli
  - H2: Bir agente geçme
  - H2: İleti kurtarma modu
  - H2: İlgili içerikler

## cli/pairing.md

- Rota: /cli/pairing
- Başlıklar:
  - H1: openclaw pairing
  - H2: Komutlar
  - H2: pairing list
  - H2: pairing approve
  - H3: Sahip önyüklemesi
  - H2: İlgili içerikler

## cli/path.md

- Rota: /cli/path
- Başlıklar:
  - H1: openclaw path
  - H2: Neden kullanılmalı
  - H2: Nasıl kullanılır
  - H2: Nasıl çalışır
  - H2: Alt komutlar
  - H2: Genel bayraklar
  - H2: oc:// söz dizimi
  - H2: Dosya türüne göre adresleme
  - H2: Değişiklik sözleşmesi
  - H2: Örnekler
  - H2: Dosya türüne göre tarifler
  - H3: Markdown
  - H3: JSONC
  - H3: JSONL
  - H3: YAML
  - H2: Alt komut başvurusu
  - H3: resolve &lt;oc-path&gt;
  - H3: find &lt;pattern&gt;
  - H3: set &lt;oc-path&gt; &lt;value&gt;
  - H3: validate &lt;oc-path&gt;
  - H3: emit &lt;file&gt;
  - H2: Çıkış kodları
  - H2: Çıktı modu
  - H2: Notlar
  - H2: İlgili içerikler

## cli/plugins.md

- Rota: /cli/plugins
- Başlıklar:
  - H2: Komutlar
  - H2: Oluşturma
  - H3: Sağlayıcı iskelesi
  - H2: Yükleme
  - H3: Pazar yeri kısa gösterimi
  - H2: Listeleme
  - H3: Plugin dizini
  - H2: Kaldırma
  - H2: Güncelleme
  - H2: İnceleme
  - H2: Doctor
  - H2: Kayıt defteri
  - H2: Pazar yeri
  - H2: İlgili içerikler

## cli/policy.md

- Rota: /cli/policy
- Başlıklar:
  - H1: openclaw policy
  - H2: Hızlı başlangıç
  - H3: Politika kuralı başvurusu
  - H4: Kapsamlı katmanlar
  - H4: Kanallar
  - H4: MCP sunucuları
  - H4: Model sağlayıcıları
  - H4: Ağ
  - H4: İleti yönlendirme
  - H4: Giriş ve kanal erişimi
  - H4: Gateway
  - H4: Agent çalışma alanı
  - H4: Korumalı alan duruşu
  - H4: Veri İşleme
  - H4: Gizli bilgiler
  - H4: Yürütme onayları
  - H4: Kimlik doğrulama profilleri
  - H4: Araç meta verileri
  - H4: Araç duruşu
  - H2: Denetimleri çalıştırma
  - H2: Politikayı yapılandırma
  - H2: Politika durumunu kabul etme
  - H2: Bulgular
  - H2: Onarma
  - H2: Çıkış kodları
  - H2: İlgili içerikler

## cli/promos.md

- Rota: /cli/promos
- Başlıklar:
  - H1: openclaw promos
  - H2: Komutlar
  - H2: openclaw promos list
  - H2: openclaw promos claim &lt;slug&gt;
  - H2: models list içinde pasif keşif

## cli/proxy.md

- Rota: /cli/proxy
- Başlıklar:
  - H1: openclaw proxy
  - H2: Doğrulama
  - H3: Seçenekler
  - H2: Proxy hata ayıklama
  - H2: İlgili içerikler

## cli/qr.md

- Rota: /cli/qr
- Başlıklar:
  - H1: openclaw qr
  - H2: Seçenekler
  - H2: Kurulum kodunun içeriği
  - H2: Gateway URL'sini çözümleme
  - H2: Kimlik doğrulamasını çözümleme (--remote olmadan)
  - H2: Kimlik doğrulamasını çözümleme (--remote)
  - H2: İlgili içerikler

## cli/reset.md

- Rota: /cli/reset
- Başlıklar:
  - H1: openclaw reset
  - H2: Seçenekler
  - H2: Kapsamlar
  - H2: Notlar
  - H2: İlgili içerikler

## cli/sandbox.md

- Rota: /cli/sandbox
- Başlıklar:
  - H2: Komutlar
  - H3: openclaw sandbox list
  - H3: openclaw sandbox recreate
  - H3: openclaw sandbox explain
  - H2: Yeniden oluşturma neden gereklidir
  - H2: Yaygın tetikleyiciler
  - H2: Kayıt defteri geçişi
  - H2: Yapılandırma
  - H2: İlgili içerikler

## cli/secrets.md

- Rota: /cli/secrets
- Başlıklar:
  - H1: openclaw secrets
  - H2: Çalışma zamanı anlık görüntüsünü yeniden yükleme
  - H2: Denetim
  - H2: Yapılandırma (etkileşimli yardımcı)
  - H3: Yürütme sağlayıcısı güvenliği
  - H2: Kaydedilmiş bir planı uygulama
  - H3: Neden geri alma yedeği yoktur
  - H2: Örnek
  - H2: İlgili içerikler

## cli/security.md

- Rota: /cli/security
- Başlıklar:
  - H1: openclaw security
  - H2: Denetim modları
  - H2: Denetlenenler
  - H2: SecretRef davranışı
  - H2: Engellemeler
  - H2: JSON çıktısı
  - H2: --fix tarafından değiştirilenler
  - H2: İlgili içerikler

## cli/sessions.md

- Rota: /cli/sessions
- Başlıklar:
  - H1: openclaw sessions
  - H2: Yörüngenin son kısmındaki ilerleme
  - H2: Yörünge paketini dışa aktarma
  - H2: Temizlik bakımı
  - H2: Oturumu sıkıştırma
  - H3: sessions.compact RPC
  - H2: İlgili içerikler

## cli/setup.md

- Rota: /cli/setup
- Başlıklar:
  - H1: openclaw setup
  - H2: Seçenekler
  - H3: Temel mod
  - H2: Örnekler
  - H2: Notlar
  - H2: İlgili içerikler

## cli/skills.md

- Rota: /cli/skills
- Başlıklar:
  - H1: openclaw skills
  - H2: Komutlar
  - H2: Skills Atölyesi
  - H2: İlgili içerikler

## cli/status.md

- Rota: /cli/status
- Başlıklar:
  - H2: Oturum ve model çözümleme
  - H2: Kullanım ve kota
  - H2: Genel bakış ve güncelleme durumu
  - H2: Gizli bilgiler
  - H2: Bellek
  - H2: İlgili

## cli/system.md

- Rota: /cli/system
- Başlıklar:
  - H1: openclaw system
  - H2: Yaygın komutlar
  - H2: system event
  - H2: system heartbeat last|enable|disable
  - H2: system presence
  - H2: Notlar
  - H2: İlgili

## cli/tasks.md

- Rota: /cli/tasks
- Başlıklar:
  - H2: Kullanım
  - H2: Kök seçenekler
  - H2: Alt komutlar
  - H3: list
  - H3: show
  - H3: notify
  - H3: cancel
  - H3: audit
  - H3: maintenance
  - H3: flow
  - H2: İlgili

## cli/transcripts.md

- Rota: /cli/transcripts
- Başlıklar:
  - H1: openclaw transcripts
  - H2: Komutlar
  - H2: Çıktı
  - H2: Günde çok sayıda oturum
  - H2: Eksik özetler
  - H2: Eski dosya deposunu yükseltme
  - H2: Yapılandırma

## cli/tui.md

- Rota: /cli/tui
- Başlıklar:
  - H1: openclaw tui
  - H2: Seçenekler
  - H2: Notlar
  - H2: Örnekler
  - H2: Yapılandırma onarım döngüsü
  - H2: İlgili

## cli/uninstall.md

- Rota: /cli/uninstall
- Başlıklar:
  - H1: openclaw uninstall
  - H2: Seçenekler
  - H2: Örnekler
  - H2: Notlar
  - H2: İlgili

## cli/update.md

- Rota: /cli/update
- Başlıklar:
  - H1: openclaw update
  - H2: Kullanım
  - H2: Seçenekler
  - H2: update status
  - H2: update repair
  - H2: update wizard
  - H2: Yaptığı işlemler
  - H3: Yeniden başlatma devri
  - H3: Denetim düzlemi yanıt biçimi
  - H2: Git çalışma kopyası akışı
  - H3: Kanal seçimi
  - H3: Güncelleme adımları
  - H3: Plugin eşitleme ayrıntıları
  - H2: İlgili

## cli/voicecall.md

- Rota: /cli/voicecall
- Başlıklar:
  - H1: openclaw voicecall
  - H2: Alt komutlar
  - H2: Kurulum ve hızlı doğrulama
  - H3: setup
  - H3: smoke
  - H2: Çağrı yaşam döngüsü
  - H3: call
  - H3: start
  - H3: continue
  - H3: speak
  - H3: dtmf
  - H3: end
  - H3: status
  - H2: Günlükler ve metrikler
  - H3: tail
  - H3: latency
  - H2: Webhook'ları dışarı açma
  - H3: expose
  - H2: İlgili

## cli/webhooks.md

- Rota: /cli/webhooks
- Başlıklar:
  - H1: openclaw webhooks
  - H2: Alt komutlar
  - H2: webhooks gmail setup
  - H3: Gerekli
  - H3: Pub/Sub seçenekleri
  - H3: OpenClaw teslim seçenekleri
  - H3: gog watch serve seçenekleri
  - H3: Tailscale üzerinden dışarı açma
  - H3: Çıktı
  - H2: webhooks gmail run
  - H2: İlgili

## cli/wiki.md

- Rota: /cli/wiki
- Başlıklar:
  - H1: openclaw wiki
  - H2: Yaygın komutlar
  - H2: Aracı seçimi
  - H2: Komutlar
  - H3: wiki status
  - H3: wiki doctor
  - H3: wiki init
  - H3: wiki ingest &lt;path&gt;
  - H3: wiki okf import &lt;path&gt;
  - H3: wiki compile
  - H3: wiki lint
  - H3: wiki search &lt;query&gt;
  - H3: wiki get &lt;lookup&gt;
  - H3: wiki apply
  - H3: wiki bridge import
  - H3: wiki unsafe-local import
  - H3: wiki chatgpt import
  - H3: wiki chatgpt rollback &lt;run-id&gt;
  - H3: wiki obsidian ...
  - H2: Pratik kullanım rehberi
  - H2: Yapılandırma bağlantıları
  - H2: İlgili

## cli/workboard.md

- Rota: /cli/workboard
- Başlıklar:
  - H2: Kullanım
  - H2: list
  - H2: create
  - H2: show
  - H2: move
  - H2: dispatch
  - H2: Eğik çizgi komutlarıyla eşdeğerlik
  - H2: İzinler
  - H2: Sorun giderme
  - H3: Hiçbir kart görünmüyor
  - H3: Dispatch yalnızca veri olduğunu bildiriyor
  - H3: Dispatch hiçbir şey başlatmıyor
  - H2: İlgili

## cli/worker.md

- Rota: /cli/worker
- Başlıklar:
  - H1: openclaw worker
  - H2: Başlatma sözleşmesi
  - H2: Çalışma zamanı sınırı

## concepts/active-memory.md

- Rota: /concepts/active-memory
- Başlıklar:
  - H2: Konuşmalar arasında hatırlama
  - H2: Gelişmiş Active Memory hızlı başlangıcı
  - H2: Nasıl çalışır
  - H2: Ne zaman çalışır
  - H3: Oturum türleri
  - H2: Oturum anahtarı
  - H2: Nasıl görüntülenir
  - H2: Sorgu modları
  - H2: İstem stilleri
  - H2: Model geri dönüş politikası
  - H3: Hız önerileri
  - H4: Cerebras kurulumu
  - H2: Bellek araçları
  - H3: Yerleşik bellek
  - H3: LanceDB belleği
  - H3: Lossless Claw
  - H2: Gelişmiş kaçış yolları
  - H2: Transkript kalıcılığı
  - H2: Yapılandırma
  - H2: Önerilen kurulum
  - H3: Soğuk başlangıç ek süresi
  - H2: Hata ayıklama
  - H2: Yaygın sorunlar
  - H2: İlgili sayfalar

## concepts/agent-loop.md

- Rota: /concepts/agent-loop
- Başlıklar:
  - H2: Giriş noktaları
  - H2: Çalıştırma sırası
  - H2: Kuyruğa alma ve eşzamanlılık
  - H2: Oturum ve çalışma alanı hazırlığı
  - H2: İstem oluşturma
  - H2: Kancalar
  - H3: Dahili kancalar (Gateway kancaları)
  - H3: Plugin kancaları
  - H2: Akış
  - H2: Araç yürütme
  - H2: Yanıtı biçimlendirme
  - H2: Compaction ve yeniden denemeler
  - H2: Olay akışları
  - H2: Sohbet kanalı işleme
  - H2: Zaman aşımları
  - H3: Takılı kalan oturum tanılama
  - H2: İşlemlerin erken sonlanabileceği noktalar
  - H2: İlgili

## concepts/agent-runtimes.md

- Rota: /concepts/agent-runtimes
- Başlıklar:
  - H2: Codex yüzeyleri
  - H2: Çalışma zamanı sahipliği
  - H2: Çalışma zamanı seçimi
  - H2: GitHub Copilot aracı çalışma zamanı
  - H2: Uyumluluk sözleşmesi
  - H2: Durum etiketleri
  - H2: İlgili

## concepts/agent-workspace.md

- Rota: /concepts/agent-workspace
- Başlıklar:
  - H2: Varsayılan konum
  - H2: Ek çalışma alanı klasörleri
  - H2: Çalışma alanı dosya haritası
  - H2: Çalışma alanında OLMAYANLAR
  - H2: Git yedeklemesi (önerilir, özel)
  - H2: Gizli bilgileri kaydetmeyin
  - H2: Çalışma alanını yeni bir makineye taşıma
  - H2: Gelişmiş notlar
  - H2: İlgili

## concepts/agent.md

- Rota: /concepts/agent
- Başlıklar:
  - H2: Çalışma alanı (gerekli)
  - H2: Önyükleme dosyaları (eklenir)
  - H2: Yerleşik araçlar
  - H2: Skills
  - H2: Çalışma zamanı sınırları
  - H2: Oturumlar
  - H2: Akış sırasında yönlendirme
  - H2: Model referansları
  - H2: Yapılandırma (asgari)
  - H2: İlgili

## concepts/architecture.md

- Rota: /concepts/architecture
- Başlıklar:
  - H2: Genel bakış
  - H2: Bileşenler ve akışlar
  - H3: Gateway (arka plan hizmeti)
  - H3: İstemciler (Mac uygulaması / CLI / web yönetimi)
  - H3: Node'lar (macOS / iOS / Android / başsız)
  - H3: WebChat
  - H2: Bağlantı yaşam döngüsü (tek istemci)
  - H2: İletişim protokolü (özet)
  - H2: Eşleştirme ve yerel güven
  - H2: Protokol türleme ve kod üretimi
  - H2: Uzaktan erişim
  - H2: İşlem anlık görüntüsü
  - H2: Değişmezler
  - H2: İlgili

## concepts/channel-docking.md

- Rota: /concepts/channel-docking
- Başlıklar:
  - H2: Örnek
  - H2: Neden kullanılmalı
  - H2: Gerekli yapılandırma
  - H2: Komutlar
  - H2: Değişenler
  - H2: Değişmeyenler
  - H2: Sorun giderme

## concepts/commitments.md

- Rota: /concepts/commitments
- Başlıklar:
  - H2: Mevcut kayıtlar
  - H2: İlgili

## concepts/compaction.md

- Rota: /concepts/compaction
- Başlıklar:
  - H2: Nasıl çalışır
  - H2: Otomatik Compaction
  - H2: Manuel Compaction
  - H2: Yapılandırma
  - H3: Farklı bir model kullanma
  - H3: Tanımlayıcıları koruma
  - H3: Etkin transkript bayt koruması
  - H3: Ardıl transkriptler
  - H3: Compaction bildirimleri
  - H3: Belleği boşaltma
  - H2: Takılabilir Compaction sağlayıcıları
  - H2: Compaction ile budama karşılaştırması
  - H2: Sorun giderme
  - H2: İlgili

## concepts/context-engine.md

- Rota: /concepts/context-engine
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Nasıl çalışır
  - H3: Alt aracı yaşam döngüsü (isteğe bağlı)
  - H3: Sistem istemine ekleme
  - H2: Eski motor
  - H2: Plugin motorları
  - H3: ContextEngine arayüzü
  - H3: Çalışma zamanı ayarları
  - H3: Ana makine gereksinimleri
  - H3: Hata yalıtımı
  - H3: ownsCompaction
  - H2: Yapılandırma referansı
  - H2: Compaction ve bellekle ilişkisi
  - H2: İpuçları
  - H2: İlgili

## concepts/context.md

- Rota: /concepts/context
- Başlıklar:
  - H2: Hızlı başlangıç (bağlamı inceleme)
  - H2: Örnek çıktı
  - H3: /context list
  - H3: /context detail
  - H3: /context map
  - H2: Bağlam penceresine neler dâhildir
  - H2: OpenClaw sistem istemini nasıl oluşturur
  - H2: Eklenen çalışma alanı dosyaları (Proje Bağlamı)
  - H2: Skills: eklenenler ve isteğe bağlı yüklenenler
  - H2: Araçlar: iki farklı maliyet vardır
  - H2: Komutlar, yönergeler ve "satır içi kısayollar"
  - H2: Oturumlar, Compaction ve budama (kalıcı olanlar)
  - H2: /context gerçekte neleri bildirir
  - H2: İlgili içerikler

## concepts/delegate-architecture.md

- Rota: /concepts/delegate-architecture
- Başlıklar:
  - H2: Temsilci nedir
  - H2: Neden temsilciler kullanılır
  - H2: Yetenek katmanları
  - H3: Katman 1: Salt Okunur + Taslak
  - H3: Katman 2: Başkası Adına Gönderme
  - H3: Katman 3: Proaktif
  - H2: Ön koşullar: yalıtım ve sağlamlaştırma
  - H3: Kesin engeller (pazarlığa açık değildir)
  - H3: Araç kısıtlamaları
  - H3: Korumalı alan yalıtımı
  - H3: Denetim izi
  - H2: Temsilci kurulumu
  - H3: 1. Temsilci ajanı oluşturma
  - H3: 2. Kimlik sağlayıcısı yetki devrini yapılandırma
  - H4: Microsoft 365
  - H4: Google Workspace
  - H3: 3. Temsilciyi kanallara bağlama
  - H3: 4. Temsilci ajana kimlik bilgileri ekleme
  - H2: Örnek: kurumsal asistan
  - H2: Ölçeklendirme modeli
  - H2: İlgili içerikler

## concepts/dreaming.md

- Rota: /concepts/dreaming
- Başlıklar:
  - H2: Dreaming tarafından yazılanlar
  - H2: Aşama modeli
  - H2: Oturum dökümünü içe alma
  - H2: Rüya Günlüğü
  - H2: Derin sıralama sinyalleri
  - H3: QA gölge denemesi rapor kapsamı
  - H2: Zamanlama
  - H2: Hızlı başlangıç
  - H2: Eğik çizgi komutu
  - H2: CLI iş akışı
  - H2: Temel varsayılanlar
  - H2: Rüyalar kullanıcı arayüzü
  - H2: İlgili içerikler

## concepts/experimental-features.md

- Rota: /concepts/experimental-features
- Başlıklar:
  - H2: Şu anda belgelenen bayraklar
  - H2: Denetim Kullanıcı Arayüzü Laboratuvarı
  - H2: Yerel model yalın modu
  - H3: Neden bu araçlar
  - H3: Ne zaman etkinleştirilmeli
  - H3: Ne zaman devre dışı bırakılmalı
  - H3: Etkinleştirme
  - H2: Deneysel, gizli anlamına gelmez
  - H2: İlgili içerikler

## concepts/features.md

- Rota: /concepts/features
- Başlıklar:
  - H2: Öne çıkanlar
  - H2: Tam liste
  - H2: İlgili içerikler

## concepts/main-session.md

- Rota: /concepts/main-session
- Başlıklar:
  - H2: Ana sayfa
  - H2: Ana oturuma aktarılanlar
  - H2: Sıfırlamalar ve konuşmalar arasında bellek
  - H2: Kalıcı geçmişe sahip sürekli oturum
  - H2: Bunun yerine yalıtım istendiğinde
  - H2: İlgili içerikler

## concepts/managed-worktrees.md

- Rota: /concepts/managed-worktrees
- Başlıklar:
  - H2: Düzen ve adlar
  - H2: Yok sayılan dosyaları hazırlama
  - H2: Depo kurulumunu çalıştırma
  - H2: Oturum çalışma ağaçları
  - H2: Anlık görüntüler, temizleme ve geri yükleme
  - H2: CLI
  - H2: Gateway yöntemleri
  - H2: Çalışma panosu çalışma alanları

## concepts/mantis-slack-desktop-runbook.md

- Rota: /concepts/mantis-slack-desktop-runbook
- Başlıklar:
  - H2: Depolama modeli
  - H2: GitHub tetiklemesi
  - H2: Yerel CLI
  - H2: Hazırlama modları
  - H2: Zamanlama yorumu
  - H2: Kanıt kontrol listesi
  - H2: Hata işleme
  - H2: İlgili içerikler

## concepts/mantis.md

- Rota: /concepts/mantis
- Başlıklar:
  - H2: Sahiplik
  - H2: CLI komutları
  - H3: discord-smoke
  - H3: run
  - H3: desktop-browser-smoke
  - H3: slack-desktop-smoke
  - H3: telegram-desktop-builder
  - H2: Kanıt manifestosu
  - H2: GitHub otomasyonu
  - H2: Makineler ve gizli bilgiler
  - H2: Çalıştırma sonuçları
  - H2: Senaryo ekleme
  - H2: Açık sorular

## concepts/markdown-formatting.md

- Rota: /concepts/markdown-formatting
- Başlıklar:
  - H2: İşlem hattı
  - H2: IR örneği
  - H2: Tablo işleme
  - H2: Parçalama kuralları
  - H2: Bağlantı politikası
  - H2: Sürpriz bozanlar
  - H2: Kanal biçimlendiricisi ekleme veya güncelleme
  - H2: Yaygın sorunlar
  - H2: İlgili içerikler

## concepts/memory-builtin.md

- Rota: /concepts/memory-builtin
- Başlıklar:
  - H2: Sağladıkları
  - H2: Başlarken
  - H2: Desteklenen gömme sağlayıcıları
  - H2: İndeksleme nasıl çalışır
  - H2: Ne zaman kullanılmalı
  - H2: Sorun giderme
  - H2: Yapılandırma
  - H2: İlgili içerikler

## concepts/memory-honcho.md

- Rota: /concepts/memory-honcho
- Başlıklar:
  - H2: Sağladıkları
  - H2: Kullanılabilir araçlar
  - H2: Başlarken
  - H2: Yapılandırma
  - H2: Mevcut belleği taşıma
  - H2: Nasıl çalışır
  - H2: Honcho ile yerleşik belleğin karşılaştırması
  - H2: CLI komutları
  - H2: Ek okumalar
  - H2: İlgili içerikler

## concepts/memory-qmd.md

- Rota: /concepts/memory-qmd
- Başlıklar:
  - H2: Yerleşik belleğe ekledikleri
  - H2: Başlarken
  - H3: Ön koşullar
  - H3: Etkinleştirme
  - H2: Yardımcı süreç nasıl çalışır
  - H2: Arama performansı ve uyumluluk
  - H2: Model geçersiz kılmaları
  - H2: Ek yolları indeksleme
  - H2: Oturum dökümlerini indeksleme
  - H2: Arama kapsamı
  - H2: Alıntılar
  - H2: Ne zaman kullanılmalı
  - H2: Sorun giderme
  - H2: Yapılandırma
  - H2: İlgili içerikler

## concepts/memory-search.md

- Rota: /concepts/memory-search
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Desteklenen sağlayıcılar
  - H2: Arama nasıl çalışır
  - H2: Arama kalitesini iyileştirme
  - H3: Zamansal azalma
  - H3: MMR (çeşitlilik)
  - H3: İkisini de etkinleştirme
  - H2: Çok modlu bellek
  - H2: Oturum belleğinde arama
  - H2: Sorun giderme
  - H2: İlgili içerikler

## concepts/memory.md

- Rota: /concepts/memory
- Başlıklar:
  - H2: Nasıl çalışır
  - H2: Nelerin nereye gittiği
  - H2: Kodlama asistanlarından içe aktarma
  - H2: Eyleme duyarlı anılar
  - H2: Kullanımdan kaldırılan çıkarımsal taahhütler
  - H2: Bellek araçları
  - H2: Bellek araması
  - H2: Bellek arka uçları
  - H2: Bilgi vikisi katmanı
  - H2: Otomatik bellek boşaltma
  - H2: Dreaming
  - H2: Temellendirilmiş geriye dönük doldurma ve canlı yükseltme
  - H2: CLI
  - H2: Ek okumalar

## concepts/message-lifecycle-refactor.md

- Rota: /concepts/message-lifecycle-refactor
- Başlıklar:
  - H2: Bu yeniden düzenleme neden yapıldı
  - H2: Yayınlananlar
  - H3: Gönderme bağlamı
  - H3: Alma bağlamı
  - H3: Canlı önizleme
  - H3: Kalıcı alındılar
  - H3: Herkese açık SDK'nın küçültülmesi
  - H2: Uygulamanın özgün tasarımdan ayrıldığı noktalar
  - H2: Somut geçiş tehlikeleri (hâlâ geçerlidir)
  - H2: Hata sınıflandırması
  - H2: Açık sorular
  - H2: İlgili içerikler

## concepts/messages.md

- Rota: /concepts/messages
- Başlıklar:
  - H2: Gelen iletilerde yinelenenleri kaldırma
  - H2: Gelen iletilerde geri sıçramayı önleme
  - H2: Oturumlar ve cihazlar
  - H2: İstem gövdeleri ve geçmiş bağlamı
  - H2: Araç sonucu meta verileri
  - H2: Kuyruğa alma ve takip iletileri
  - H2: Kanal çalıştırma sahipliği
  - H2: Akış, parçalama ve toplu işleme
  - H2: Akıl yürütme görünürlüğü ve tokenlar
  - H2: Ön ekler, ileti dizileri ve yanıtlar
  - H2: Sessiz yanıtlar
  - H2: İlgili içerikler

## concepts/model-failover.md

- Rota: /concepts/model-failover
- Başlıklar:
  - H2: Çalışma zamanı akışı
  - H2: Seçim kaynağı politikası
  - H2: Kimlik doğrulama hatası atlama önbelleği
  - H2: Kullanıcının görebildiği yedek sisteme geçiş bildirimleri
  - H2: Kimlik doğrulama depolaması (anahtarlar + OAuth)
  - H2: Profil kimlikleri
  - H2: Döndürme sırası
  - H3: Oturum bağlılığı (önbellek dostu)
  - H3: OpenAI Codex aboneliği ve API anahtarı yedeği
  - H2: Bekleme süreleri
  - H2: Faturalandırma kaynaklı devre dışı bırakmalar
  - H2: Yedek modele geçiş
  - H3: Aday zinciri kuralları
  - H3: Yedek modele geçişi hangi hatalar ilerletir
  - H3: Bekleme süresinde atlama ve yoklama davranışı
  - H2: Oturum geçersiz kılmaları ve canlı model değiştirme
  - H2: Gözlemlenebilirlik ve hata özetleri
  - H2: İlgili yapılandırma

## concepts/model-providers.md

- Rota: /concepts/model-providers
- Başlıklar:
  - H2: Hızlı kurallar
  - H2: Sağlayıcıları Denetim Kullanıcı Arayüzü'nde yapılandırma
  - H2: Plugin'e ait sağlayıcı davranışı
  - H2: API anahtarı döndürme
  - H2: Resmî sağlayıcı Plugin'leri
  - H3: OpenAI
  - H3: Anthropic
  - H3: OpenAI ChatGPT/Codex OAuth
  - H3: Abonelik tarzındaki diğer barındırılan seçenekler
  - H3: OpenCode
  - H3: Google Gemini (API anahtarı)
  - H3: Google Vertex ve Gemini CLI
  - H3: Z.AI (GLM)
  - H3: Vercel AI Gateway
  - H3: Diğer paketlenmiş sağlayıcı Plugin'leri
  - H4: Bilinmesi gereken kendine özgü davranışlar
  - H2: models.providers aracılığıyla sağlayıcılar (özel/temel URL)
  - H3: Moonshot AI (Kimi)
  - H3: Kimi Coding
  - H3: Volcano Engine (Doubao)
  - H3: BytePlus (Uluslararası)
  - H3: Synthetic
  - H3: MiniMax
  - H3: LM Studio
  - H3: Ollama
  - H3: vLLM
  - H3: SGLang
  - H3: Yerel vekil sunucular (LM Studio, vLLM, LiteLLM vb.)
  - H2: CLI örnekleri
  - H2: İlgili içerikler

## concepts/models.md

- Rota: /concepts/models
- Başlıklar:
  - H2: Seçim sırası
  - H2: Seçim kaynağı ve geri dönüş katılığı
  - H2: Hızlı model politikası
  - H2: İlk kurulum
  - H2: "Modele izin verilmiyor" (ve yanıtların neden durduğu)
  - H2: Sohbette /model
  - H2: CLI
  - H2: Model kayıt defteri (models.json)
  - H2: İlgili

## concepts/multi-agent.md

- Rota: /concepts/multi-agent
- Başlıklar:
  - H2: Tek bir aracı nedir
  - H2: Yollar
  - H3: Tek aracılı mod (varsayılan)
  - H2: Aracı yardımcısı
  - H2: Hızlı başlangıç
  - H2: Birden çok aracı, birden çok kişilik
  - H2: Aracı başına Memory Wiki kasaları
  - H2: Aracılar arası QMD bellek araması
  - H2: Tek WhatsApp numarası, birden çok kişi (DM ayrımı)
  - H2: Yönlendirme kuralları
  - H2: Birden çok hesap / telefon numarası
  - H2: Kavramlar
  - H2: Platform örnekleri
  - H2: Yaygın kalıplar
  - H2: Aracı başına korumalı alan ve araç yapılandırması
  - H2: İlgili

## concepts/multi-user.md

- Rota: /concepts/multi-user
- Başlıklar:
  - H2: Güven sınırı
  - H2: Sahiplik ve iletişim durumu
  - H2: Taslaklar
  - H2: Tur ilişkilendirmesi
  - H2: İlgili

## concepts/oauth.md

- Rota: /concepts/oauth
- Başlıklar:
  - H2: Belirteç havuzu (neden var olduğu)
  - H2: Depolama (belirteçlerin bulunduğu yer)
  - H2: Anthropic Claude CLI yeniden kullanımı
  - H2: OAuth değişimi (oturum açmanın çalışma biçimi)
  - H3: Anthropic kurulum belirteci
  - H3: OpenAI Codex (ChatGPT OAuth)
  - H2: Yenileme + sona erme
  - H2: Birden çok hesap (profil) + yönlendirme
  - H3: 1) Tercih edilen: ayrı aracılar
  - H3: 2) Gelişmiş: tek aracıda birden çok profil
  - H2: İlgili

## concepts/parallel-specialist-lanes.md

- Rota: /concepts/parallel-specialist-lanes
- Başlıklar:
  - H2: İlk ilkeler
  - H2: Önerilen kullanıma sunma planı
  - H3: Aşama 1: hat sözleşmeleri + arka planda yoğun çalışma
  - H3: Aşama 2: öncelik ve eşzamanlılık denetimleri
  - H3: Aşama 3: koordinatör / trafik denetleyicisi
  - H2: Asgari hat sözleşmesi şablonu
  - H2: İlgili

## concepts/personal-agent-benchmark-pack.md

- Rota: /concepts/personal-agent-benchmark-pack
- Başlıklar:
  - H2: Senaryolar
  - H2: Gizlilik modeli
  - H2: Paketi genişletme

## concepts/presence.md

- Rota: /concepts/presence
- Başlıklar:
  - H2: İletişim durumu alanları (görüntülenenler)
  - H2: Üreticiler (iletişim durumunun geldiği yer)
  - H3: 1) Gateway öz girdisi
  - H3: 2) WebSocket bağlantısı
  - H4: Geçici denetim düzlemi bağlantıları neden görüntülenmez
  - H3: 3) system-event işaretçileri
  - H3: 4) Node bağlantıları (rol: node)
  - H2: Birleştirme + yinelenenleri kaldırma kuralları (instanceId neden önemlidir)
  - H2: TTL ve sınırlı boyut
  - H2: Uzak/tünel uyarısı (geri döngü IP'leri)
  - H2: Tüketiciler
  - H3: Control UI Cihazlar sayfası
  - H3: macOS Örnekler sekmesi
  - H2: Hata ayıklama ipuçları
  - H2: İlgili

## concepts/progress-drafts.md

- Rota: /concepts/progress-drafts
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Kullanıcıların gördükleri
  - H2: Mod seçme
  - H2: Etiketleri yapılandırma
  - H2: İlerleme satırlarını denetleme
  - H3: Ayrıntı modu
  - H3: Komut/çalıştırma metni
  - H3: Açıklama hattı
  - H3: Durum başlığı
  - H3: Satır sınırları
  - H3: Zengin işleme (Slack)
  - H3: Araç/görev satırlarını gizleme
  - H2: Kanal davranışı
  - H2: Sonlandırma
  - H2: Sorun giderme
  - H2: İlgili

## concepts/qa-e2e-automation.md

- Rota: /concepts/qa-e2e-automation
- Başlıklar:
  - H2: Komut yüzeyi
  - H3: Profil destekli qa çalıştırması
  - H2: Operatör akışı
  - H3: Gözlemlenebilirlik duman testleri
  - H3: Matrix duman testi hatları
  - H3: Discord Mantis senaryoları
  - H3: Mantis Slack masaüstü ve görsel görev çalıştırıcıları
  - H3: Kimlik bilgisi havuzu durum denetimi
  - H2: Standart senaryo kapsamı
  - H2: Discord, Slack, Telegram ve WhatsApp QA başvurusu
  - H3: Paylaşılan CLI bayrakları
  - H3: Telegram QA
  - H3: Discord QA
  - H3: Slack QA
  - H4: Slack çalışma alanını ayarlama
  - H3: WhatsApp QA
  - H3: Convex kimlik bilgisi havuzu
  - H2: Depo destekli başlangıç verileri
  - H2: Sağlayıcı taklit hatları
  - H2: Aktarım bağdaştırıcıları
  - H3: Kanal ekleme
  - H3: Senaryo yardımcısı adları
  - H2: Raporlama
  - H2: İlgili belgeler

## concepts/queue-steering.md

- Rota: /concepts/queue-steering
- Başlıklar:
  - H2: Çalışma zamanı sınırı
  - H2: Modlar
  - H2: Ani yük örneği
  - H2: Kapsam
  - H2: Sıçrama önleme
  - H2: İlgili

## concepts/queue.md

- Rota: /concepts/queue
- Başlıklar:
  - H2: Neden
  - H2: Çalışma biçimi
  - H2: Varsayılanlar
  - H2: Kuyruk modları
  - H2: Kuyruk seçenekleri
  - H2: Yönlendirme ve akış
  - H2: Öncelik sırası
  - H2: Oturum başına geçersiz kılmalar
  - H2: Kuyruğa alınmış turu iptal etme
  - H2: Kapsam ve garantiler
  - H2: Sorun giderme
  - H2: İlgili

## concepts/retry.md

- Rota: /concepts/retry
- Başlıklar:
  - H2: Hedefler
  - H2: Varsayılanlar
  - H2: Davranış
  - H3: Model sağlayıcıları
  - H3: Discord
  - H3: Telegram
  - H2: Yapılandırma
  - H2: Notlar
  - H2: İlgili

## concepts/session-pruning.md

- Rota: /concepts/session-pruning
- Başlıklar:
  - H2: Neden önemli olduğu
  - H2: Çalışma biçimi
  - H2: Eski görüntüleri temizleme
  - H2: Akıllı varsayılanlar
  - H2: Etkinleştirme veya devre dışı bırakma
  - H2: Budama ile Compaction karşılaştırması
  - H2: Ek okumalar
  - H2: İlgili

## concepts/session-search.md

- Rota: /concepts/session-search
- Başlıklar:
  - H1: Oturum araması
  - H2: Görünürlük ve çıktı
  - H2: Dizin yaşam döngüsü
  - H2: Oturum araması ile bellek araması karşılaştırması

## concepts/session-state.md

- Rota: /concepts/session-state
- Başlıklar:
  - H2: Sinyal günlüğü
  - H2: İzleyiciler
  - H2: Bildirimler: çok değil, bir tane
  - H2: Uzlaştırma
  - H2: Depolama ve sınırlar
  - H2: İlgili

## concepts/session-tool.md

- Rota: /concepts/session-tool
- Başlıklar:
  - H2: Kullanılabilir araçlar
  - H2: Oturumları listeleme ve okuma
  - H2: Oturum ayarlarını ve grupları yönetme
  - H2: Oturumlar ile konuşmalar
  - H2: Oturumlar arası ileti gönderme
  - H2: Durum ve düzenleme yardımcıları
  - H2: Oturum durumu değişiklikleri
  - H2: Alt aracılar oluşturma
  - H2: Görünürlük
  - H2: Ek okumalar
  - H2: İlgili

## concepts/session.md

- Rota: /concepts/session
- Başlıklar:
  - H2: İletilerin yönlendirilme biçimi
  - H2: DM yalıtımı
  - H3: Dock bağlantılı kanallar
  - H2: Gizli oturumlar
  - H2: Konuşmalar arasında hatırlama
  - H2: Oturum yaşam döngüsü
  - H2: Durumun bulunduğu yer
  - H2: Oturum bakımı
  - H2: Oturumları inceleme
  - H2: Ek okumalar
  - H2: İlgili

## concepts/soul.md

- Rota: /concepts/soul
- Başlıklar:
  - H2: SOUL.md dosyasına neler eklenmeli
  - H2: Bunun çalışma nedeni
  - H2: Molty istemi
  - H2: İyi bir sonuç nasıl görünür
  - H2: Bir uyarı
  - H2: İlgili

## concepts/streaming.md

- Rota: /concepts/streaming
- Başlıklar:
  - H2: Control UI başlangıç durumu
  - H2: Blok akışı (kanal iletileri)
  - H3: Blok akışıyla medya teslimi
  - H2: Parçalama algoritması (alt/üst sınırlar)
  - H2: Birleştirme (akışı sağlanan blokları birleştirme)
  - H2: Bloklar arasında insan benzeri tempo
  - H2: "Parçaları veya her şeyi akışla gönder"
  - H2: Önizleme akışı modları
  - H3: Kanal eşlemesi
  - H3: Eski anahtar geçişi
  - H2: Çalışma zamanı davranışı
  - H3: Telegram
  - H3: Discord
  - H3: Slack
  - H3: Mattermost
  - H3: Matrix
  - H2: Araç ilerlemesi önizleme güncellemeleri
  - H2: İlerleme taslağını işleme
  - H3: Açıklama ilerleme hattı
  - H2: İlgili

## concepts/system-prompt.md

- Rota: /concepts/system-prompt
- Başlıklar:
  - H2: Yapı
  - H2: İstem modları
  - H2: İstem anlık görüntüleri
  - H2: Çalışma alanı önyükleme eklemesi
  - H2: Zaman işleme
  - H2: Skills
  - H2: Belgelendirme
  - H2: İlgili

## concepts/timezone.md

- Rota: /concepts/timezone
- Başlıklar:
  - H2: Üç saat dilimi yüzeyi
  - H2: Kullanıcı saat dilimini ayarlama
  - H2: Zarf saat dilimi değerleri
  - H2: Ne zaman geçersiz kılınmalı
  - H2: İlgili

## concepts/typebox.md

- Rota: /concepts/typebox
- Başlıklar:
  - H2: Zihinsel model (30 saniye)
  - H2: Şemaların bulunduğu yer
  - H2: Mevcut işlem hattı
  - H2: Şemaların çalışma zamanında kullanılma biçimi
  - H2: Örnek çerçeveler
  - H2: Asgari istemci (Node.js)
  - H2: Uygulamalı örnek: uçtan uca yöntem ekleme
  - H2: Swift kod üretimi davranışı
  - H2: Sürümleme ve uyumluluk
  - H2: Şema kalıpları ve kuralları
  - H2: Canlı şema JSON'u
  - H2: Şemalar değiştirildiğinde
  - H2: İlgili

## concepts/typing-indicators.md

- Rota: /concepts/typing-indicators
- Başlıklar:
  - H2: Varsayılanlar
  - H2: Modlar
  - H2: Yapılandırma
  - H2: Notlar
  - H2: İlgili

## concepts/usage-tracking.md

- Rota: /concepts/usage-tracking
- Başlıklar:
  - H2: Nedir
  - H2: Nerede gösterilir
  - H2: Anthropic ve OpenAI maliyet geçmişi
  - H2: Varsayılan kullanım alt bilgisi modu
  - H3: Üç farklı oturum durumu
  - H3: Öncelik
  - H3: Sıfırlama ve kapatma
  - H3: Açma/kapatma davranışı
  - H3: Yapılandırma
  - H2: Özel /usage tam alt bilgisi
  - H3: Biçim
  - H3: Sözleşme yolları
  - H3: Fiiller
  - H3: Parça biçimleri
  - H3: Örnek
  - H2: Sağlayıcılar + kimlik bilgileri
  - H2: İlgili

## date-time.md

- Rota: /date-time
- Başlıklar:
  - H2: İleti zarfları (varsayılan olarak yerel)
  - H3: Örnekler
  - H2: Sistem istemi: geçerli tarih ve saat
  - H2: Sistem olayı satırları (varsayılan olarak yerel)
  - H3: Kullanıcı saat dilini ve biçimini yapılandırma
  - H2: Saat biçimi algılama (otomatik)
  - H2: Araç yükleri + bağlayıcılar (ham sağlayıcı zamanı + normalleştirilmiş alanlar)
  - H2: İlgili belgeler

## debug/node-issue.md

- Rota: /debug/node-issue
- Başlıklar:
  - H1: Node + tsx "\\name is not a function" çökmesi
  - H2: Durum
  - H2: İlk belirti
  - H2: Neden
  - H2: Güncel yeniden oluşturma kontrolü
  - H2: Geçici çözümler (çökme tekrar meydana gelirse)
  - H2: Kaynaklar
  - H2: İlgili

## diagnostics/flags.md

- Rota: /diagnostics/flags
- Başlıklar:
  - H2: Nasıl çalışır
  - H2: Bilinen bayraklar
  - H2: Yapılandırma aracılığıyla etkinleştirme
  - H2: Ortam geçersiz kılması (tek seferlik)
  - H2: Profilleyici bayrakları
  - H2: Zaman çizelgesi yapıtları
  - H2: Günlüklerin kaydedildiği yer
  - H2: Günlükleri çıkarma
  - H2: Notlar
  - H2: İlgili

## gateway/1password.md

- Rota: /gateway/1password
- Başlıklar:
  - H2: Gereksinimler
  - H2: Yapılandırma gizli bilgilerini op ile çözümleme
  - H2: Başsız Gateway'ler için hizmet hesabı kurulumu
  - H2: Aracılar için 1password becerisi
  - H2: Claude için 1Password ile tarayıcıda oturum açma
  - H2: Güvenlik notları
  - H2: Sorun giderme

## gateway/audit.md

- Rota: /gateway/audit
- Başlıklar:
  - H1: Denetim geçmişi
  - H2: Kayıt aileleri
  - H2: İleti yaşam döngüsü olayları
  - H3: Konuşma türü sınıflandırması
  - H2: Gizlilik modeli
  - H2: Kapsam ve kanıt sınırları
  - H2: Depolama, saklama ve geçiş
  - H2: Sorgulama
  - H2: İlgili

## gateway/authentication.md

- Rota: /gateway/authentication
- Başlıklar:
  - H2: Önerilen kurulum: API anahtarı (herhangi bir sağlayıcı)
  - H2: Anthropic: Claude CLI'ı yeniden kullanma
  - H2: Elle belirteç girişi
  - H3: SecretRef destekli kimlik bilgileri
  - H2: Model kimlik doğrulama durumunu kontrol etme
  - H2: API anahtarı döndürme (gateway)
  - H2: Gateway çalışırken sağlayıcı kimlik doğrulamasını kaldırma
  - H2: Hangi kimlik bilgisinin kullanılacağını denetleme
  - H3: OpenAI ve eski openai-codex kimlikleri
  - H3: Oturum açma sırasında (CLI)
  - H3: Oturum başına (sohbet komutu)
  - H3: Aracı başına (CLI geçersiz kılması)
  - H2: Sorun giderme
  - H3: "Kimlik bilgisi bulunamadı"
  - H3: Süresi dolmak üzere olan/dolmuş belirteç
  - H2: İlgili

## gateway/background-process.md

- Rota: /gateway/background-process
- Başlıklar:
  - H2: exec aracı
  - H3: Ortam geçersiz kılmaları
  - H3: Yapılandırma (ortam geçersiz kılmalarına tercih edilir)
  - H2: Alt süreç köprüleme
  - H2: process aracı
  - H2: Örnekler
  - H2: İlgili

## gateway/bonjour.md

- Rota: /gateway/bonjour
- Başlıklar:
  - H2: Tailscale üzerinden geniş alan Bonjour (Tek Noktaya Yayın DNS-SD)
  - H3: Gateway yapılandırması
  - H3: Tek seferlik DNS sunucusu kurulumu (gateway ana makinesi, yalnızca macOS)
  - H3: Tailscale DNS ayarları
  - H3: Gateway dinleyicisi güvenliği
  - H2: Duyurulanlar
  - H2: Hizmet türleri
  - H2: TXT anahtarları (gizli olmayan ipuçları)
  - H2: macOS'ta hata ayıklama
  - H2: Gateway günlüklerinde hata ayıklama
  - H2: iOS Node'unda hata ayıklama
  - H2: Bonjour ne zaman etkinleştirilmeli
  - H2: Bonjour ne zaman devre dışı bırakılmalı
  - H2: Docker tuzakları
  - H2: Devre dışı Bonjour sorunlarını giderme
  - H2: Yaygın hata biçimleri
  - H2: Kaçış karakterli örnek adları (\032)
  - H2: Etkinleştirme / devre dışı bırakma / yapılandırma
  - H2: İlgili belgeler

## gateway/bridge-protocol.md

- Rota: /gateway/bridge-protocol
- Başlıklar:
  - H2: Neden vardı
  - H2: Aktarım
  - H2: El sıkışma ve eşleştirme
  - H2: Çerçeveler
  - H2: Exec yaşam döngüsü olayları
  - H2: Geçmiş tailnet kullanımı
  - H2: Sürümleme
  - H2: İlgili

## gateway/cli-backends.md

- Rota: /gateway/cli-backends
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Yedek seçenek olarak kullanma
  - H2: Yapılandırma
  - H2: Nasıl çalışır
  - H2: Zaman aşımları ve uzun süren işler
  - H3: Claude CLI'a özgü ayrıntılar
  - H3: Claude tarayıcı araçları ve 1Password ile oturum açma
  - H2: Oturumlar
  - H2: claude-cli oturumlarından yedek seçenek önsözü
  - H2: Görüntüler
  - H2: Girdiler ve çıktılar
  - H2: Plugin tarafından yönetilen varsayılanlar
  - H2: Metin dönüştürme katmanları
  - H2: Yerel Compaction sahipliği
  - H2: MCP katmanlarını paketleme
  - H2: Geçmişi yeniden tohumlama sınırı
  - H2: Sınırlamalar
  - H2: Sorun giderme
  - H2: İlgili

## gateway/clients.md

- Rota: /gateway/clients
- Başlıklar:
  - H2: Paketleri yükleme
  - H2: Kapsamları seçme ve cihazı eşleştirme
  - H2: İstemci yeteneklerini duyurma
  - H2: Yeniden bağlandıktan sonra durumu kurtarma
  - H2: Geçmiş meta verilerini ve kararlı sabitleyicileri kullanma
  - H2: Kullanımı yoklamak yerine abone olma
  - H2: Exec onaylarını geriye dönük doldurma
  - H2: Protokol sürümlerini izleme
  - H2: İlgili

## gateway/cloud-workers.md

- Rota: /gateway/cloud-workers
- Başlıklar:
  - H2: Neyin nerede çalıştığı
  - H2: Gereksinimler
  - H2: Yapılandırma
  - H3: Kurulum komutu
  - H3: Kanalları yükleme
  - H2: Oturum gönderme
  - H2: Güvenlik modeli
  - H2: Sorun giderme
  - H2: İlgili

## gateway/config-agents.md

- Rota: /gateway/config-agents
- Başlıklar:
  - H2: Aracı varsayılanları
  - H3: agents.defaults.workspace
  - H3: agents.defaults.repoRoot
  - H3: agents.defaults.skills
  - H3: agents.defaults.skipBootstrap
  - H3: agents.defaults.skipOptionalBootstrapFiles
  - H3: agents.defaults.contextInjection
  - H3: agents.defaults.bootstrapMaxChars
  - H3: agents.defaults.bootstrapTotalMaxChars
  - H3: Aracı başına önyükleme profili geçersiz kılmaları
  - H3: agents.defaults.bootstrapPromptTruncationWarning
  - H3: Bağlam bütçesi sahiplik haritası
  - H4: agents.defaults.startupContext
  - H4: agents.defaults.contextLimits
  - H4: `agents.entries.*.contextLimits`
  - H4: skills.limits.maxSkillsPromptChars
  - H4: `agents.entries.*.skillsLimits.maxSkillsPromptChars`
  - H3: agents.defaults.imageMaxDimensionPx
  - H3: agents.defaults.imageQuality
  - H3: agents.defaults.userTimezone
  - H3: agents.defaults.timeFormat
  - H3: agents.defaults.model
  - H3: Çalışma zamanı ilkesi
  - H3: CLI arka ucu seçimi
  - H3: agents.defaults.promptOverlays
  - H3: agents.defaults.heartbeat
  - H3: agents.defaults.compaction
  - H3: agents.defaults.contextPruning
  - H3: Blok akışı
  - H3: Yazma göstergeleri
  - H3: agents.defaults.sandbox
  - H3: agents.entries (aracı başına geçersiz kılmalar)
  - H2: Çok aracılı yönlendirme
  - H3: Bağlama eşleştirme alanları
  - H3: Aracı başına erişim profilleri
  - H2: Oturum
  - H2: İletiler
  - H3: Yanıt ön eki
  - H3: Alındı tepkisi
  - H3: Kuyruk
  - H3: Gelen ileti geri sekmesi
  - H3: Diğer ileti anahtarları
  - H3: TTS (metinden konuşmaya)
  - H2: Konuşma
  - H2: İlgili

## gateway/config-channels.md

- Rota: /gateway/config-channels
- Başlıklar:
  - H2: Kanallar
  - H3: DM ve grup erişimi
  - H3: Kanal modeli geçersiz kılmaları
  - H3: Kanal varsayılanları ve Heartbeat
  - H3: WhatsApp
  - H3: Telegram
  - H3: Discord
  - H3: Google Chat
  - H3: Slack
  - H3: Mattermost
  - H3: Signal
  - H3: iMessage
  - H3: Matrix
  - H3: Microsoft Teams
  - H3: IRC
  - H3: Çoklu hesap (tüm kanallar)
  - H3: Diğer Plugin kanalları
  - H3: Grup sohbetinde bahsetme geçidi
  - H4: DM geçmişi sınırları
  - H4: Kendi kendine sohbet modu
  - H3: Komutlar (sohbet komutu işleme)
  - H2: İlgili

## gateway/config-tools.md

- Rota: /gateway/config-tools
- Başlıklar:
  - H2: Araçlar
  - H3: Araç profilleri
  - H3: Araç grupları
  - H3: Korumalı alan araç ilkesi içindeki MCP ve Plugin araçları
  - H3: tools.codeMode
  - H3: tools.allow / tools.deny
  - H3: tools.byProvider
  - H3: tools.toolsBySender
  - H3: tools.elevated
  - H3: tools.exec
  - H3: tools.loopDetection
  - H3: tools.web
  - H3: tools.media
  - H3: tools.agentToAgent
  - H3: tools.sessions
  - H3: `tools.sessions_spawn`
  - H3: tools.experimental
  - H3: agents.defaults.subagents
  - H2: Özel sağlayıcılar ve temel URL'ler
  - H3: Sağlayıcı alanı ayrıntıları
  - H3: Sağlayıcı örnekleri
  - H2: İlgili

## gateway/configuration-examples.md

- Rota: /gateway/configuration-examples
- Başlıklar:
  - H2: Hızlı başlangıç
  - H3: Mutlak minimum
  - H3: Önerilen başlangıç yapılandırması
  - H2: Genişletilmiş örnek (başlıca seçenekler)
  - H3: Sembolik bağlantılı kardeş skill deposu
  - H2: Yaygın kalıplar
  - H3: Tek bir geçersiz kılma ile paylaşılan skill temeli
  - H3: Çok platformlu kurulum
  - H3: Güvenilir Node ağı için otomatik onay
  - H3: Güvenli DM modu (paylaşılan gelen kutusu / çok kullanıcılı DM'ler)
  - H3: Anthropic API anahtarı + MiniMax yedeği
  - H3: İş botu (kısıtlı erişim)
  - H3: Yalnızca yerel modeller
  - H2: İpuçları
  - H2: İlgili içerikler

## gateway/configuration-reference.md

- Rota: /gateway/configuration-reference
- Başlıklar:
  - H2: Kanallar
  - H2: Aracı varsayılanları, çoklu aracı, oturumlar ve mesajlar
  - H2: Araçlar ve özel sağlayıcılar
  - H2: Modeller
  - H2: MCP
  - H2: Skills
  - H2: Pluginler
  - H3: Codex çalıştırma çatısı Plugin yapılandırması
  - H2: Tarayıcı
  - H2: Kullanıcı arayüzü
  - H2: Gateway
  - H3: OpenAI uyumlu uç noktalar
  - H3: Çoklu örnek yalıtımı
  - H3: gateway.tls
  - H3: gateway.reload
  - H2: Bulut çalışanı ortamları
  - H3: Crabbox profili
  - H3: Statik SSH geliştirme profili
  - H2: Kancalar
  - H3: Gmail entegrasyonu
  - H2: Canvas Plugin ana makinesi
  - H2: Keşif
  - H3: mDNS (Bonjour)
  - H3: Geniş alan (DNS-SD)
  - H2: Ortam
  - H3: env (satır içi ortam değişkenleri)
  - H3: Ortam değişkeni ikamesi
  - H2: Gizli değerler
  - H3: SecretRef
  - H3: Desteklenen kimlik bilgisi yüzeyi
  - H3: Gizli değer sağlayıcıları yapılandırması
  - H2: Kimlik doğrulama depolaması
  - H2: Denetim
  - H2: Günlükleme
  - H2: Tanılama
  - H2: Güncelleme
  - H2: ACP
  - H2: Sihirbaz
  - H2: Kimlik
  - H2: Köprü (eski, kaldırıldı)
  - H2: Cron
  - H3: cron.failureAlert
  - H3: cron.failureDestination
  - H2: Medya modeli şablon değişkenleri
  - H2: Yapılandırma eklemeleri ($include)
  - H2: İlgili içerikler

## gateway/configuration.md

- Rota: /gateway/configuration
- Başlıklar:
  - H2: Asgari yapılandırma
  - H2: Yapılandırmayı düzenleme
  - H2: Katı doğrulama
  - H2: Yaygın görevler
  - H2: Yapılandırmayı çalışırken yeniden yükleme
  - H3: Yeniden yükleme modları
  - H3: Çalışırken uygulananlar ve yeniden başlatma gerektirenler
  - H3: Yeniden yükleme planlaması
  - H2: Yapılandırma RPC'si (programatik güncellemeler)
  - H2: Ortam değişkenleri
  - H2: Tam başvuru
  - H2: İlgili içerikler

## gateway/diagnostics.md

- Rota: /gateway/diagnostics
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Sohbet komutu
  - H2: Dışa aktarımın içeriği
  - H2: Gizlilik modeli
  - H2: Kararlılık kaydedicisi
  - H2: Yararlı seçenekler
  - H2: Tanılamayı devre dışı bırakma
  - H2: İlgili içerikler

## gateway/discovery.md

- Rota: /gateway/discovery
- Başlıklar:
  - H2: Terimler
  - H2: Neden hem doğrudan bağlantı hem de SSH var?
  - H2: Keşif girdileri
  - H3: 1) Bonjour / DNS-SD
  - H4: Hizmet işareti ayrıntıları
  - H3: 2) Tailnet (ağlar arası)
  - H3: 3) Manuel / SSH hedefi
  - H2: Aktarım seçimi (istemci politikası)
  - H2: Eşleştirme ve kimlik doğrulama (doğrudan aktarım)
  - H2: Bileşenlere göre sorumluluklar
  - H2: İlgili içerikler

## gateway/doctor.md

- Rota: /gateway/doctor
- Başlıklar:
  - H2: Hızlı başlangıç
  - H3: Başsız ve otomasyon modları
  - H2: Salt okunur lint modu
  - H2: Yaptıkları (özet)
  - H2: Dreams kullanıcı arayüzü geriye dönük doldurma ve sıfırlama
  - H2: Ayrıntılı davranış ve gerekçe
  - H2: İlgili içerikler

## gateway/embedding.md

- Rota: /gateway/embedding
- Başlıklar:
  - H2: Alt süreci bir gömme ön ayarıyla başlatma
  - H3: Electron kabuk anlık görüntüsü uyarısı
  - H2: Geçersiz yapılandırmayı çıkış koduyla işleme
  - H2: Protokolün hazır olmasını bekleme
  - H2: Yeniden başlatma ve kapatmayı yorumlama
  - H2: Durum dosyaları yerine RPC kullanma
  - H2: Kurun; düzleştirmeyin
  - H2: İlgili içerikler

## gateway/external-apps.md

- Rota: /gateway/external-apps
- Başlıklar:
  - H2: Bugün kullanılabilenler
  - H2: Önerilen yol
  - H2: İş birliğine dayalı ana makine askıya alma
  - H2: Uygulama kodu ve Plugin kodu
  - H2: İlgili içerikler

## gateway/gateway-lock.md

- Rota: /gateway/gateway-lock
- Başlıklar:
  - H2: Neden?
  - H2: Üç katman
  - H3: Durum ve yapılandırma kilitleri
  - H3: Soket bağlama
  - H2: İşletim notları
  - H2: İlgili içerikler

## gateway/health.md

- Rota: /gateway/health
- Başlıklar:
  - H2: Hızlı kontroller
  - H2: Derinlemesine tanılama
  - H2: Sağlık izleyicisi yapılandırması
  - H2: Çalışma süresi izleme
  - H3: İzleme hizmeti kurulum örnekleri
  - H2: Bir şey başarısız olduğunda
  - H2: Özel "health" komutu
  - H2: İlgili içerikler

## gateway/heartbeat.md

- Rota: /gateway/heartbeat
- Başlıklar:
  - H2: Hızlı başlangıç (başlangıç düzeyi)
  - H2: Varsayılanlar
  - H2: Heartbeat isteminin amacı
  - H2: Yanıt sözleşmesi
  - H2: Yapılandırma
  - H3: Kapsam ve öncelik
  - H3: Aracı başına Heartbeat'ler
  - H3: Etkin saatler örneği
  - H3: 24/7 kurulumu
  - H3: Çoklu hesap örneği
  - H3: Alan notları
  - H2: Teslim davranışı
  - H2: Görünürlük denetimleri
  - H3: Her bayrağın işlevi
  - H3: Kanal ve hesap başına örnekler
  - H3: Yaygın kalıplar
  - H2: İzleyici karalama alanı (isteğe bağlı)
  - H3: Cron ile yinelenen kontroller planlama
  - H3: Aracı kendi karalama alanını güncelleyebilir mi?
  - H2: Manuel uyandırma (isteğe bağlı)
  - H2: Maliyet farkındalığı
  - H2: Heartbeat sonrasında bağlam taşması
  - H2: İlgili içerikler

## gateway/index.md

- Rota: /gateway
- Başlıklar:
  - H2: 5 dakikada yerel başlatma
  - H2: Çalışma zamanı modeli
  - H2: OpenAI uyumlu uç noktalar
  - H3: Bağlantı noktası ve bağlama önceliği
  - H3: Çalışırken yeniden yükleme modları
  - H2: Operatör komut kümesi
  - H2: Birden fazla Gateway (aynı ana makine)
  - H2: Uzaktan erişim
  - H2: Gözetim ve hizmet yaşam döngüsü
  - H2: Geliştirme profili hızlı yolu
  - H2: Protokol hızlı başvurusu (operatör görünümü)
  - H2: İşletim kontrolleri
  - H3: Canlılık
  - H3: Hazır olma
  - H3: Boşluk kurtarma
  - H2: Yaygın hata belirtileri
  - H2: Güvenlik garantileri
  - H2: İlgili içerikler

## gateway/local-model-services.md

- Rota: /gateway/local-model-services
- Başlıklar:
  - H2: Nasıl çalışır?
  - H2: Yapılandırma biçimi
  - H2: Alanlar
  - H2: Inferrs örneği
  - H2: ds4 örneği
  - H2: İlgili içerikler

## gateway/local-models.md

- Rota: /gateway/local-models
- Başlıklar:
  - H2: Asgari donanım
  - H2: Bir arka uç seçme
  - H2: LM Studio + büyük yerel model (Responses API)
  - H3: Hibrit yapılandırma: birincil olarak barındırılan, yedek olarak yerel
  - H3: Bölgesel barındırma / veri yönlendirme
  - H2: OpenAI uyumlu diğer yerel proxy'ler
  - H2: Daha küçük veya daha katı arka uçlar
  - H2: Sorun giderme
  - H2: İlgili içerikler

## gateway/logging.md

- Rota: /gateway/logging
- Başlıklar:
  - H1: Günlükleme
  - H2: Dosya tabanlı günlükleyici
  - H3: Ayrıntılı mod ve günlük düzeyleri
  - H2: Konsol yakalama
  - H2: Karartma
  - H2: Gateway WebSocket günlükleri
  - H3: WS günlük stili
  - H2: Konsol biçimlendirmesi (alt sistem günlükleme)
  - H2: İlgili içerikler

## gateway/multi-tenant-hosting.md

- Rota: /gateway/multi-tenant-hosting
- Başlıklar:
  - H1: Çok kiracılı barındırma
  - H2: Her kiracının neden bir hücreye ihtiyacı var?
  - H2: Mimari
  - H2: Güven sınırı
  - H2: Yalıtım basamakları
  - H2: Hızlı başlangıç
  - H2: Güncel kapsam
  - H2: İlgili içerikler

## gateway/multiple-gateways.md

- Rota: /gateway/multiple-gateways
- Başlıklar:
  - H2: Kurtarma botu hızlı başlangıcı
  - H3: --profile rescue onboard tarafından değiştirilenler
  - H2: Genel çoklu Gateway kurulumu
  - H2: Yalıtım kontrol listesi
  - H2: Bağlantı noktası eşlemesi (türetilmiş)
  - H2: Tarayıcı/CDP notları (yaygın tuzak)
  - H2: Manuel ortam örneği
  - H2: Hızlı kontroller
  - H2: İlgili içerikler

## gateway/network-model.md

- Rota: /gateway/network-model
- Başlıklar:
  - H2: İlgili içerikler

## gateway/openai-http-api.md

- Rota: /gateway/openai-http-api
- Başlıklar:
  - H2: Uç noktayı etkinleştirme
  - H2: Güvenlik sınırı (önemli)
  - H2: Kimlik doğrulama
  - H2: Bu uç noktanın kullanılacağı durumlar
  - H2: Aracı öncelikli model sözleşmesi
  - H2: Oturum davranışı
  - H2: İstek sınırları
  - H2: Sohbet aracı sözleşmesi
  - H3: Desteklenen istek alanları
  - H3: Desteklenmeyen çeşitler
  - H3: Akışsız araç yanıtı biçimi
  - H3: Akışlı araç yanıtı biçimi
  - H3: Araç takip döngüsü
  - H2: Akış (SSE)
  - H2: Open WebUI hızlı kurulumu
  - H2: Örnekler
  - H2: İlgili içerikler

## gateway/openresponses-http-api.md

- Rota: /gateway/openresponses-http-api
- Başlıklar:
  - H2: Kimlik doğrulama, güvenlik ve yönlendirme
  - H2: Oturum davranışı
  - H2: İstek yapısı
  - H2: Öğeler (girdi)
  - H3: ileti
  - H3: `function_call_output` (tur tabanlı araçlar)
  - H3: akıl yürütme ve `item_reference`
  - H2: Araçlar (istemci tarafı işlev araçları)
  - H2: Görseller (`input_image`)
  - H2: Dosyalar (`input_file`)
  - H2: Dosya ve görsel sınırları
  - H2: Akış (SSE)
  - H2: Kullanım
  - H2: Hatalar
  - H2: Örnekler
  - H2: İlgili içerikler

## gateway/openshell.md

- Rota: /gateway/openshell
- Başlıklar:
  - H2: Ön koşullar
  - H2: Hızlı başlangıç
  - H2: Çalışma alanı modları
  - H3: ayna (varsayılan)
  - H3: uzak
  - H3: Mod seçimi
  - H2: Yapılandırma başvurusu
  - H2: Örnekler
  - H3: Asgari uzak kurulum
  - H3: GPU'lu ayna modu
  - H3: Özel gateway ile aracı başına OpenShell
  - H2: Yaşam döngüsü yönetimi
  - H2: Güvenlik sağlamlaştırması
  - H2: Mevcut sınırlamalar
  - H2: Nasıl çalışır?
  - H2: İlgili içerikler

## gateway/opentelemetry.md

- Rota: /gateway/opentelemetry
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Dışa aktarılan sinyaller
  - H2: Yapılandırma başvurusu
  - H3: Ortam değişkenleri
  - H2: Gizlilik ve içerik yakalama
  - H2: Örnekleme ve boşaltma
  - H3: Model çağrısı gözlem birimleri
  - H3: Claude Code CLI model çağrısı doğruluğu
  - H2: Dışa aktarılan metrikler
  - H3: Model kullanımı
  - H3: İleti akışı
  - H3: Konuşma
  - H3: Kuyruklar ve oturumlar
  - H3: Oturum canlılığı telemetrisi
  - H3: Çalıştırma altyapısı yaşam döngüsü
  - H3: Araç yürütme ve döngü algılama
  - H3: Yürütme
  - H3: Tanılama iç işleyişi (bellek, yükler, dışa aktarıcı sağlığı)
  - H2: Dışa aktarılan kapsamlar
  - H2: Tanılama olayı kataloğu
  - H2: Dışa aktarıcı olmadan
  - H2: Devre dışı bırakma
  - H2: İlgili içerikler

## gateway/operator-scopes.md

- Rota: /gateway/operator-scopes
- Başlıklar:
  - H2: Roller
  - H2: Kapsam düzeyleri
  - H2: Yöntem kapsamı yalnızca ilk geçittir
  - H2: Cihaz eşleştirme onayları
  - H2: Node eşleştirme onayları
  - H2: Paylaşılan gizli anahtarla kimlik doğrulama

## gateway/pairing.md

- Rota: /gateway/pairing
- Başlıklar:
  - H2: Yetenek onayı nasıl çalışır?
  - H2: CLI iş akışı (ekransız kullanıma uygun)
  - H2: API yüzeyi (gateway protokolü)
  - H2: Node komut geçidi (2026.3.31+)
  - H2: Node olayı güven sınırları (2026.3.31+)
  - H2: SSH ile doğrulanmış cihazların otomatik onayı (varsayılan)
  - H2: Otomatik onay (macOS uygulaması)
  - H2: Güvenilir CIDR cihazlarının otomatik onayı
  - H2: Sessiz eşleştirmede geçersiz kılınan kayıtları temizleme
  - H2: Meta veri yükseltmelerinin otomatik onayı
  - H2: QR eşleştirme yardımcıları
  - H2: Yerellik ve iletilen üstbilgiler
  - H2: Depolama (yerel, özel)
  - H2: Aktarım davranışı
  - H2: İlgili içerikler

## gateway/prometheus.md

- Rota: /gateway/prometheus
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Dışa aktarılan metrikler
  - H2: Etiket politikası
  - H2: PromQL tarifleri
  - H2: Prometheus ile OpenTelemetry dışa aktarımı arasında seçim yapma
  - H2: Sorun giderme
  - H2: İlgili içerikler

## gateway/protocol.md

- Rota: /gateway/protocol
- Başlıklar:
  - H2: npm paketleri
  - H2: Aktarım ve çerçeveleme
  - H2: El sıkışma
  - H3: Çalışan rolü ve kapalı protokol
  - H3: İstemci yetenekleri
  - H3: Node bağlantı örneği
  - H2: Roller ve kapsamlar
  - H3: Yetenekler/komutlar/izinler (node)
  - H2: Varlık
  - H3: Node arka planda canlı olayı
  - H2: Yayın olayı kapsamlandırması
  - H2: RPC yöntem aileleri
  - H3: Yaygın olay aileleri
  - H3: Node yardımcı yöntemleri
  - H2: Denetim defteri RPC'si
  - H2: Görev defteri RPC'leri
  - H2: Operatör yardımcı yöntemleri
  - H3: models.list görünümleri
  - H2: Yürütme onayları
  - H2: Aracı teslimi geri dönüşü
  - H2: Sürüm oluşturma
  - H3: İstemci sabitleri
  - H2: Kimlik doğrulama
  - H2: Cihaz kimliği ve eşleştirme
  - H3: Cihaz kimlik doğrulama geçişi tanılamaları
  - H2: TLS ve sabitleme
  - H2: Kapsam
  - H2: İlgili içerikler

## gateway/remote-gateway-readme.md

- Rota: /gateway/remote-gateway-readme
- Başlıklar:
  - H1: OpenClaw.app'i Uzak Gateway ile Çalıştırma
  - H2: Kurulum
  - H2: Nasıl çalışır?
  - H2: İlgili içerikler

## gateway/remote.md

- Rota: /gateway/remote
- Başlıklar:
  - H2: Temel fikir
  - H2: Topoloji seçenekleri
  - H2: Komut akışı (ne nerede çalışır?)
  - H2: SSH tüneli (CLI ve araçlar)
  - H2: CLI uzak bağlantı varsayılanları
  - H2: Kimlik bilgisi önceliği
  - H2: Sohbet kullanıcı arayüzüne uzaktan erişim
  - H2: macOS uygulamasının uzak modu
  - H2: Güvenlik kuralları (uzak/VPN)
  - H3: macOS: LaunchAgent aracılığıyla kalıcı SSH tüneli
  - H4: 1. adım: SSH yapılandırması ekleyin
  - H4: 2. adım: SSH anahtarını kopyalayın (bir kez)
  - H4: 3. adım: gateway belirtecini yapılandırın
  - H4: 4. adım: LaunchAgent'ı oluşturun
  - H4: 5. adım: LaunchAgent'ı yükleyin
  - H4: Sorun giderme
  - H2: İlgili içerikler

## gateway/restart-recovery.md

- Rota: /gateway/restart-recovery
- Başlıklar:
  - H2: Yeniden başlatmadan sonra korunanlar
  - H2: Kontrollü yeniden başlatmalar önce işlemlerin tamamlanmasını bekler
  - H2: Kesintiye uğrayan işler nasıl algılanır?
  - H2: Otomatik sürdürme
  - H3: Alt aracılar
  - H3: Arka plan görevleri
  - H3: Aracı tarafından istenen yeniden başlatmalar
  - H2: Güvenlik mekanizmaları ve gözlemlenebilirlik
  - H2: Sürdürülmeyenler

## gateway/sandbox-vs-tool-policy-vs-elevated.md

- Rota: /gateway/sandbox-vs-tool-policy-vs-elevated
- Başlıklar:
  - H2: Hızlı hata ayıklama
  - H2: Korumalı alan: araçların çalıştığı yer
  - H3: Bağlama noktaları (hızlı güvenlik denetimi)
  - H2: Araç politikası: hangi araçların var/çağrılabilir olduğu
  - H3: Araç grupları (kısaltmalar)
  - H2: Yükseltilmiş: yalnızca yürütmeye özgü "ana makinede çalıştırma"
  - H2: Yaygın "korumalı alan hapishanesi" düzeltmeleri
  - H3: "X aracı korumalı alan araç politikası tarafından engellendi"
  - H3: "Bunun ana ortam olduğunu sanıyordum, neden korumalı alanda?"
  - H2: İlgili içerikler

## gateway/sandboxing.md

- Rota: /gateway/sandboxing
- Başlıklar:
  - H2: Korumalı alana alınanlar
  - H2: Modlar, kapsam ve arka uç
  - H2: Docker arka ucu
  - H3: Korumalı alan tarayıcısı
  - H2: SSH arka ucu
  - H2: OpenShell arka ucu
  - H2: Çalışma alanı erişimi
  - H2: Tek aracı için birden çok klasör
  - H3: Diğer bağlama davranışları
  - H2: Görseller ve kurulum
  - H2: setupCommand (bir kerelik kapsayıcı kurulumu)
  - H2: Araç politikası ve kaçış yolları
  - H2: Çok aracılı geçersiz kılmalar
  - H2: Asgari etkinleştirme örneği
  - H2: İlgili içerikler

## gateway/secrets-plan-contract.md

- Rota: /gateway/secrets-plan-contract
- Başlıklar:
  - H2: Plan dosyası gereksinimleri
  - H2: Plan dosyası yapısı
  - H2: Sağlayıcı ekleme/güncellemeleri ve silmeleri
  - H2: Desteklenen hedef kapsamı
  - H2: Hedef türü davranışı
  - H2: Yol doğrulama kuralları
  - H2: Hata davranışı
  - H2: Yürütme sağlayıcısı onay davranışı
  - H2: Çalışma zamanı ve denetim kapsamı notları
  - H2: Operatör denetimleri
  - H2: İlgili belgeler

## gateway/secrets.md

- Rota: /gateway/secrets
- Başlıklar:
  - H2: Çalışma zamanı modeli
  - H2: Çıkış zamanında ekleme (nöbetçiler)
  - H2: Aracı erişim sınırı
  - H2: Etkin yüzey filtreleme
  - H2: Gateway kimlik doğrulama yüzeyi tanılamaları
  - H2: İlk katılım başvurusu ön kontrolü
  - H2: SecretRef sözleşmesi
  - H2: Sağlayıcı yapılandırması
  - H2: Dosya destekli API anahtarları
  - H2: Yürütme entegrasyonu örnekleri
  - H2: MCP sunucusu ortam değişkenleri
  - H2: Korumalı alan SSH kimlik doğrulama malzemesi
  - H2: Desteklenen kimlik bilgisi yüzeyi
  - H2: Gerekli davranış ve öncelik
  - H2: Etkinleştirme tetikleyicileri
  - H2: Bozulma ve kurtarma sinyalleri
  - H2: Komut yolu çözümleme
  - H2: Denetim ve yapılandırma iş akışı
  - H2: Tek yönlü güvenlik politikası
  - H2: Eski kimlik doğrulama uyumluluğu notları
  - H2: Web kullanıcı arayüzü notu
  - H2: İlgili içerikler

## gateway/security/audit-checks.md

- Rota: /gateway/security/audit-checks
- Başlıklar:
  - H2: İlgili içerikler

## gateway/security/exposure-runbook.md

- Rota: /gateway/security/exposure-runbook
- Başlıklar:
  - H2: Erişime açma düzenini seçme
  - H2: Ön kontrol envanteri
  - H2: Temel denetimler
  - H2: Asgari güvenli temel
  - H2: Doğrudan ileti ve grup erişimi
  - H2: Ters proxy denetimleri
  - H2: Araç ve korumalı alan incelemesi
  - H2: Değişiklik sonrası doğrulama
  - H2: Geri alma planı
  - H2: İnceleme kontrol listesi

## gateway/security/index.md

- Rota: /gateway/security
- Başlıklar:
  - H2: Kapsam: kişisel asistan güvenlik modeli
  - H2: openclaw güvenlik denetimi
  - H3: Denetimin kontrol ettikleri (genel düzeyde)
  - H3: Bulguları önceliklendirirken izlenecek sıra
  - H2: 60 saniyede güçlendirilmiş temel yapılandırma
  - H3: İstekte bulunan kişi kapsamındaki denetimler ve istem bağlamı
  - H2: Güven sınırı matrisi
  - H2: Tasarım gereği güvenlik açığı olmayan durumlar
  - H2: Gateway ve Node güveni
  - H2: Tehdit modeli
  - H2: DM erişimi: eşleştirme, izin verilenler listesi, açık, devre dışı
  - H3: İzin verilenler listeleri (iki katman)
  - H3: DM oturumu yalıtımı (çok kullanıcılı mod)
  - H2: Bağlam görünürlüğü ile tetikleme yetkilendirmesi
  - H2: İstem enjeksiyonu
  - H3: Harici içerik ve güvenilmeyen girdi sarmalama
  - H3: Atlatma bayrakları (üretimde kapalı tutun)
  - H3: Gruplarda akıl yürütme ve ayrıntılı çıktı
  - H2: Komut yetkilendirmesi
  - H2: Denetim düzlemi araçları
  - H2: Node üzerinde yürütme (system.run)
  - H2: Dinamik Skills (izleyici / uzak Node'lar)
  - H2: Plugin'ler
  - H2: Korumalı alan kullanımı
  - H3: Alt ajan yetkilendirme güvenlik sınırı
  - H3: Salt okunur mod
  - H2: Ajan başına erişim profilleri (çoklu ajan)
  - H3: Tam erişim (korumalı alan yok)
  - H3: Salt okunur araçlar + salt okunur çalışma alanı
  - H3: Dosya sistemi/kabuk erişimi yok (sağlayıcı mesajlaşmasına izin verilir)
  - H2: Tarayıcı denetimi riskleri
  - H3: Tarayıcı SSRF politikası (varsayılan olarak katı)
  - H2: Ağ erişimine açılma
  - H3: Bağlama, bağlantı noktası, güvenlik duvarı
  - H3: UFW ile Docker bağlantı noktası yayımlama
  - H3: mDNS/Bonjour keşfi
  - H3: Gateway WebSocket kimlik doğrulaması
  - H3: Tailscale Serve kimlik üst bilgileri
  - H3: Ters proxy yapılandırması
  - H3: HSTS ve kaynak notları
  - H3: HTTP üzerinden Denetim Arayüzü
  - H3: Güvensiz/tehlikeli bayraklar
  - H2: Dağıtım ve ana makine güveni
  - H2: Diskteki gizli bilgiler
  - H3: Kimlik bilgisi depolama haritası
  - H3: Dosya izinleri
  - H3: Çalışma alanı .env dosyaları
  - H3: Günlükler ve dökümler
  - H2: Güvenli temel yapılandırma (kopyala/yapıştır)
  - H3: Ayrı numaralar (WhatsApp, Signal, Telegram)
  - H2: Olay müdahalesi
  - H3: Sınırlandırma
  - H3: Yenileme (gizli bilgiler sızdıysa ele geçirildiğini varsayın)
  - H3: Denetim
  - H3: Rapor için bilgi toplama
  - H2: Gizli bilgi taraması
  - H2: Güvenlik sorunlarını bildirme

## gateway/security/rate-limiting.md

- Rota: /gateway/security/rate-limiting
- Başlıklar:
  - H2: Kimlik doğrulama denemeleri (kimlik doğrulama öncesi)
  - H3: Tarayıcı kaynaklı bağlantılar
  - H3: Webhook'lar
  - H2: Denetim düzlemi yazma işlemleri (kimlik doğrulama sonrası güvenlik önlemi)
  - H2: ACP oturumu oluşturma
  - H2: Yeniden başlatma bekleme süresi
  - H2: İşletim notları

## gateway/security/secure-file-operations.md

- Rota: /gateway/security/secure-file-operations
- Başlıklar:
  - H2: Varsayılan: Python yardımcısı yok
  - H2: Python olmadan korunanlar
  - H2: Python'ın sağladıkları
  - H2: Plugin ve çekirdek rehberi

## gateway/security/shrinkwrap.md

- Rota: /gateway/security/shrinkwrap
- Başlıklar:
  - H2: Neden önemli
  - H2: Oluşturma ve kontrol etme
  - H2: Yayımlanmış bir paketi inceleme

## gateway/tailscale.md

- Rota: /gateway/tailscale
- Başlıklar:
  - H2: Modlar
  - H2: Yapılandırma örnekleri
  - H3: Yalnızca Tailnet (Serve)
  - H3: Yalnızca Tailnet (Tailnet IP'sine bağlama)
  - H3: Genel internet (Funnel + paylaşılan parola)
  - H2: CLI örnekleri
  - H2: Kimlik doğrulama
  - H3: Tailscale kimlik üst bilgileri (yalnızca Serve)
  - H2: Notlar
  - H3: Tailscale ön koşulları ve sınırları
  - H2: Tarayıcı denetimi (uzak Gateway + yerel tarayıcı)
  - H2: Daha fazla bilgi
  - H2: İlgili

## gateway/tools-invoke-http-api.md

- Rota: /gateway/tools-invoke-http-api
- Başlıklar:
  - H2: Kimlik doğrulama
  - H2: Güvenlik sınırı (önemli)
  - H2: İstek gövdesi
  - H2: Politika + yönlendirme davranışı
  - H2: Yanıtlar
  - H2: Örnek
  - H2: İlgili

## gateway/troubleshooting.md

- Rota: /gateway/troubleshooting
- Başlıklar:
  - H2: Komut sıralaması
  - H2: Güncellemeden sonra
  - H2: Bölünmüş kurulumlar ve daha yeni yapılandırma koruması
  - H2: Geri alma sonrasında protokol uyuşmazlığı
  - H2: Skills sembolik bağlantısı, yol dışına çıkma nedeniyle atlandı
  - H2: Anthropic 429: uzun bağlam için ek kullanım gerekli
  - H2: Üst sistemden gelen engellenmiş 403 yanıtları
  - H2: Yerel OpenAI uyumlu arka uç doğrudan yoklamaları geçiyor ancak ajan çalıştırmaları başarısız oluyor
  - H2: Yanıt yok
  - H2: Pano Denetim Arayüzü bağlantısı
  - H3: Kimlik doğrulama ayrıntı kodlarının hızlı haritası
  - H2: Gateway hizmeti çalışmıyor
  - H2: macOS Gateway sessizce yanıt vermeyi kesiyor, ardından panoya dokununca devam ediyor
  - H2: Yinelenen Gateway/Node LaunchAgent'larıyla macOS launchd gözetmen döngüsü
  - H2: Gateway yüksek bellek kullanımı sırasında kapanıyor
  - H2: Gateway geçersiz yapılandırmayı reddetti
  - H2: Gateway yoklama uyarıları
  - H2: Kanal bağlı ancak mesajlar iletilmiyor
  - H2: Cron ve Heartbeat teslimatı
  - H2: Node eşleştirildi ancak araç başarısız oluyor
  - H2: Tarayıcı aracı başarısız oluyor
  - H2: Yükseltme sonrasında bir şey aniden bozulduysa
  - H2: İlgili

## gateway/trusted-proxy-auth.md

- Rota: /gateway/trusted-proxy-auth
- Başlıklar:
  - H2: Ne zaman kullanılmalı
  - H2: Ne zaman KULLANILMAMALI
  - H2: Nasıl çalışır
  - H2: Yapılandırma
  - H3: Yapılandırma başvurusu
  - H2: Otomatik cihaz onayı
  - H2: Denetim Arayüzü eşleştirme davranışı
  - H2: Operatör kapsamları üst bilgisi
  - H2: TLS sonlandırma ve HSTS
  - H3: Kullanıma sunma rehberi
  - H2: Proxy kurulum örnekleri
  - H2: Karma token yapılandırması
  - H2: Güvenlik kontrol listesi
  - H2: Güvenlik denetimi
  - H2: Sorun giderme
  - H2: Token kimlik doğrulamasından geçiş
  - H2: İlgili

## help/debugging.md

- Rota: /help/debugging
- Başlıklar:
  - H2: Çalışma zamanı hata ayıklama geçersiz kılmaları
  - H2: Oturum izleme çıktısı
  - H2: Plugin yaşam döngüsü izlemesi
  - H2: CLI başlatma ve komut profilleme
  - H2: Gateway izleme modu
  - H2: Geliştirme profili + geliştirme Gateway'i (--dev)
  - H2: Ham akış günlükleme
  - H2: Güvenlik notları
  - H2: VSCode'da hata ayıklama
  - H3: Kurulum
  - H3: Notlar
  - H2: İlgili

## help/environment.md

- Rota: /help/environment
- Başlıklar:
  - H2: Öncelik (en yüksekten en düşüğe)
  - H2: Desteklenen operatöre yönelik değişkenler
  - H3: Yollar ve örnekler
  - H3: Gateway ve kimlik doğrulama
  - H3: Sağlayıcı kimlik bilgileri
  - H3: Günlükleme ve tanılama
  - H3: Özellik ve çalışma zamanı anahtarları
  - H2: Sağlayıcı kimlik bilgileri ve çalışma alanı .env dosyası
  - H2: Yapılandırma env bloğu
  - H2: Kabuk env içe aktarma
  - H2: Exec kabuk anlık görüntüleri
  - H2: Çalışma zamanı tarafından eklenen env değişkenleri
  - H2: Kullanıcı arayüzü env değişkenleri
  - H2: Yapılandırmada env değişkeni ikamesi
  - H2: Gizli bilgi başvuruları ile ${ENV} dizeleri
  - H2: Yolla ilgili env değişkenleri
  - H2: Ajan yardımcı aracı indirmeleri
  - H2: Günlükleme
  - H3: `OPENCLAW_HOME`
  - H2: nvm kullanıcıları: webfetch TLS hataları
  - H2: Eski ortam değişkenleri
  - H2: İlgili

## help/faq-first-run.md

- Rota: /help/faq-first-run
- Başlıklar:
  - H2: Hızlı başlangıç ve ilk çalıştırma kurulumu
  - H2: İlgili

## help/faq-models.md

- Rota: /help/faq-models
- Başlıklar:
  - H2: Modeller: varsayılanlar, seçim, takma adlar, geçiş
  - H2: Model yük devretme ve "Tüm modeller başarısız oldu"
  - H2: Kimlik doğrulama profilleri: nedir ve nasıl yönetilir
  - H2: İlgili

## help/faq.md

- Rota: /help/faq
- Başlıklar:
  - H2: Bir şey bozuksa ilk 60 saniye
  - H2: Hızlı başlangıç ve ilk çalıştırma kurulumu
  - H2: OpenClaw nedir?
  - H2: Skills ve otomasyon
  - H2: Korumalı alan kullanımı ve bellek
  - H2: Diskteki öğelerin konumları
  - H2: Yapılandırma temelleri
  - H2: Uzak Gateway'ler ve Node'lar
  - H2: Env değişkenleri ve .env yükleme
  - H2: Oturumlar ve birden çok sohbet
  - H2: Modeller, yük devretme ve kimlik doğrulama profilleri
  - H2: Gateway: bağlantı noktaları, "zaten çalışıyor" ve uzak mod
  - H2: Günlükleme ve hata ayıklama
  - H2: Medya ve ekler
  - H2: Güvenlik ve erişim denetimi
  - H2: Sohbet komutları, görevleri iptal etme ve "durmuyor"
  - H2: Çeşitli
  - H2: İlgili

## help/index.md

- Rota: /help
- Başlıklar:
  - H2: SSS
  - H2: Tanılama
  - H2: Test
  - H2: Topluluk ve meta

## help/scripts.md

- Rota: /help/scripts
- Başlıklar:
  - H2: Kurallar
  - H2: Kimlik doğrulama izleme betikleri
  - H2: GitHub okuma yardımcısı
  - H2: Betik eklerken
  - H2: İlgili

## help/testing-live.md

- Rota: /help/testing-live
- Başlıklar:
  - H2: Canlı testler ile gerçek Gateway'inizin karşılaştırması
  - H2: Canlı: yerel duman testi komutları
  - H2: Canlı: Android Node yetenek taraması
  - H2: Canlı: model duman testi (profil anahtarları)
  - H3: Katman 1: Doğrudan model tamamlama (Gateway olmadan)
  - H3: Katman 2: Gateway + geliştirme ajanı duman testi ("@openclaw" gerçekte ne yapar)
  - H2: Canlı: CLI arka ucu duman testi (Claude, Gemini veya diğer yerel CLI'lar)
  - H2: Canlı: APNs HTTP/2 proxy erişilebilirliği
  - H2: Canlı: ACP bağlama duman testi (/acp spawn ... --bind here)
  - H2: Canlı: Codex app-server test düzeneği duman testi
  - H2: Canlı: OpenAI tekrarlanan Compaction
  - H3: Önerilen canlı test tarifleri
  - H2: Canlı: model matrisi (kapsadıklarımız)
  - H3: Toplayıcılar / alternatif Gateway'ler
  - H2: Kimlik bilgileri (asla depoya işlemeyin)
  - H2: Deepgram canlı testi (ses transkripsiyonu)
  - H2: BytePlus kodlama planı canlı testi
  - H2: ComfyUI iş akışı medya canlı testi
  - H2: Görsel oluşturma canlı testi
  - H2: Müzik oluşturma canlı testi
  - H2: Video oluşturma canlı testi
  - H2: Medya canlı test düzeneği
  - H2: İlgili

## help/testing-updates-plugins.md

- Rota: /help/testing-updates-plugins
- Başlıklar:
  - H2: Koruduklarımız
  - H2: Geliştirme sırasında yerel doğrulama
  - H2: Docker hatları
  - H2: Paket Kabulü
  - H2: Sürüm varsayılanı
  - H2: Eski sürümlerle uyumluluk
  - H2: Kapsam ekleme
  - H2: Hata triyajı

## help/testing.md

- Rota: /help/testing
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Test geçici dizinleri
  - H2: Canlı ve Docker/Parallels iş akışları
  - H2: QA'ya özgü çalıştırıcılar
  - H3: Convex üzerinden paylaşılan Telegram kimlik bilgileri (v1)
  - H3: QA'ya kanal ekleme
  - H2: Test paketleri (hangisi nerede çalışır)
  - H3: Birim / entegrasyon (varsayılan)
  - H3: Kararlılık (Gateway)
  - H3: E2E (depo geneli)
  - H3: E2E (Gateway duman testi)
  - H3: E2E (Control UI sahte tarayıcısı)
  - H3: E2E: OpenShell arka ucu duman testi
  - H3: Canlı (gerçek sağlayıcılar + gerçek modeller)
  - H2: Hangi test paketini çalıştırmalıyım?
  - H2: Canlı (ağa erişen) testler
  - H2: Docker çalıştırıcıları (isteğe bağlı "Linux'ta çalışır" kontrolleri)
  - H2: Doküman tutarlılık kontrolü
  - H2: Çevrimdışı regresyon (CI için güvenli)
  - H2: Ajan güvenilirliği değerlendirmeleri (Skills)
  - H2: Sözleşme testleri (plugin ve kanal yapısı)
  - H3: Komutlar
  - H3: Kanal sözleşmeleri
  - H3: Sağlayıcı sözleşmeleri
  - H3: Ne zaman çalıştırılmalı?
  - H2: Regresyon ekleme (yönergeler)
  - H2: İlgili

## help/troubleshooting.md

- Rota: /help/troubleshooting
- Başlıklar:
  - H2: İlk 60 saniye
  - H2: Asistan sınırlı geliyor veya araçları eksik
  - H2: Anthropic uzun bağlam 429 hatası
  - H2: Yerel OpenAI uyumlu arka uç doğrudan çalışıyor ancak OpenClaw'da başarısız oluyor
  - H2: Eksik openclaw uzantıları nedeniyle plugin kurulumu başarısız oluyor
  - H2: Kurulum politikası plugin kurulumlarını veya güncellemelerini engelliyor
  - H2: Plugin mevcut ancak şüpheli sahiplik nedeniyle engelleniyor
  - H2: Karar ağacı
  - H2: İlgili

## index.md

- Rota: /
- Başlıklar:
  - H1: OpenClaw 🦞
  - H2: Dokümanlara göz atın
  - H2: OpenClaw nedir?
  - H2: Nasıl çalışır?
  - H2: Temel yetenekler
  - H2: Hızlı başlangıç
  - H2: Gösterge paneli
  - H2: Yapılandırma (isteğe bağlı)
  - H2: Buradan başlayın
  - H2: Daha fazla bilgi

## install/ansible.md

- Rota: /install/ansible
- Başlıklar:
  - H2: Ön koşullar
  - H2: Elde edecekleriniz
  - H2: Hızlı başlangıç
  - H2: Kurulan bileşenler
  - H2: Kurulum sonrası ayarlar
  - H3: Hızlı komutlar
  - H2: Güvenlik mimarisi
  - H2: Manuel kurulum
  - H2: Güncelleme
  - H2: Sorun giderme
  - H2: Gelişmiş yapılandırma
  - H2: İlgili

## install/azure.md

- Rota: /install/azure
- Başlıklar:
  - H2: Yapacaklarınız
  - H2: Gereksinimler
  - H2: Dağıtımı yapılandırma
  - H2: Azure kaynaklarını dağıtma
  - H2: OpenClaw'ı kurma
  - H2: Maliyetle ilgili hususlar
  - H2: Temizleme
  - H2: Sonraki adımlar
  - H2: İlgili

## install/bun.md

- Rota: /install/bun
- Başlıklar:
  - H2: Kurulum
  - H2: Yaşam döngüsü betikleri
  - H2: Dikkat edilmesi gerekenler
  - H2: İlgili

## install/clawdock.md

- Rota: /install/clawdock
- Başlıklar:
  - H2: Kurulum
  - H2: Elde edecekleriniz
  - H3: Temel işlemler
  - H3: Konteyner erişimi
  - H3: Web kullanıcı arayüzü ve eşleştirme
  - H3: Kurulum ve bakım
  - H3: Yardımcı araçlar
  - H2: İlk kullanım akışı
  - H2: Yapılandırma ve gizli bilgiler
  - H2: İlgili

## install/development-channels.md

- Rota: /install/development-channels
- Başlıklar:
  - H2: Kanallar arasında geçiş
  - H2: Tek seferlik sürüm veya etiket hedefleme
  - H2: Deneme çalıştırması
  - H2: Plugin'ler ve kanallar
  - H2: Mevcut durumu kontrol etme
  - H2: Etiketleme için en iyi uygulamalar
  - H2: macOS uygulamasının kullanılabilirliği
  - H2: İlgili

## install/digitalocean.md

- Rota: /install/digitalocean
- Başlıklar:
  - H2: Ön koşullar
  - H2: Kurulum
  - H2: Kalıcılık ve yedeklemeler
  - H2: 1 GB RAM için ipuçları
  - H2: Sorun giderme
  - H2: Sonraki adımlar
  - H2: İlgili

## install/docker-vm-runtime.md

- Rota: /install/docker-vm-runtime
- Başlıklar:
  - H2: Gerekli ikili dosyaları imaja dahil etme
  - H2: Derleme ve başlatma
  - H2: Nelerin nerede kalıcı olduğu
  - H2: Güncellemeler
  - H2: İlgili

## install/docker.md

- Rota: /install/docker
- Başlıklar:
  - H2: Ön koşullar
  - H2: Konteynerleştirilmiş Gateway
  - H3: Manuel akış
  - H3: Konteyner imajlarını yükseltme
  - H3: Ortam değişkenleri
  - H3: Seçilen plugin'lerle kaynaktan derlenen imajlar
  - H3: Gözlemlenebilirlik
  - H3: Sistem durumu kontrolleri
  - H3: LAN ile geri döngü karşılaştırması
  - H3: Ana makinedeki yerel sağlayıcılar
  - H3: Docker'da Claude CLI arka ucu
  - H3: Bonjour / mDNS
  - H3: Depolama ve kalıcılık
  - H3: Kabuk yardımcıları (isteğe bağlı)
  - H3: VPS üzerinde mi çalıştırıyorsunuz?
  - H2: Ajan korumalı alanı
  - H3: Hızlı etkinleştirme
  - H2: Sorun giderme
  - H2: İlgili

## install/exe-dev.md

- Rota: /install/exe-dev
- Başlıklar:
  - H2: Gereksinimler
  - H2: Yeni başlayanlar için hızlı yol
  - H2: Shelley ile otomatik kurulum
  - H2: Manuel kurulum
  - H2: Uzak kanal kurulumu
  - H2: Uzaktan erişim
  - H2: Güncelleme
  - H2: İlgili

## install/fly.md

- Rota: /install/fly
- Başlıklar:
  - H2: Gereksinimler
  - H2: Yeni başlayanlar için hızlı yol
  - H2: Sorun giderme
  - H3: "Uygulama beklenen adreste dinleme yapmıyor"
  - H3: Sistem durumu kontrolleri başarısız oluyor / bağlantı reddediliyor
  - H3: OOM / bellek sorunları
  - H3: Gateway kilidi sorunları
  - H3: Yapılandırma okunmuyor
  - H3: Yapılandırmayı SSH üzerinden yazma
  - H3: Durum kalıcı olmuyor
  - H2: Güncelleme
  - H3: Makine komutunu güncelleme
  - H2: Özel dağıtım (güçlendirilmiş)
  - H3: Özel dağıtım ne zaman kullanılmalı?
  - H3: Kurulum
  - H3: Özel dağıtıma erişme
  - H3: Özel dağıtımla Webhook'lar
  - H3: Güvenlik ödünleşimleri
  - H2: Notlar
  - H2: Maliyet
  - H2: Sonraki adımlar
  - H2: İlgili

## install/gcp.md

- Rota: /install/gcp
- Başlıklar:
  - H2: Gereksinimler
  - H2: Hızlı yol
  - H2: Sorun giderme
  - H2: Hizmet hesapları (en iyi güvenlik uygulaması)
  - H2: Sonraki adımlar
  - H2: İlgili

## install/hetzner.md

- Rota: /install/hetzner
- Başlıklar:
  - H2: Gereksinimler
  - H2: Hızlı yol
  - H2: Kod Olarak Altyapı (Terraform)
  - H2: Sonraki adımlar
  - H2: İlgili

## install/hostinger.md

- Rota: /install/hostinger
- Başlıklar:
  - H2: Ön koşullar
  - H2: Seçenek A: Tek Tıkla OpenClaw
  - H2: Seçenek B: VPS üzerinde OpenClaw
  - H2: Kurulumunuzu doğrulama
  - H2: Sorun giderme
  - H2: Sonraki adımlar
  - H2: İlgili

## install/index.md

- Rota: /install
- Başlıklar:
  - H2: Sistem gereksinimleri
  - H2: Önerilen: kurulum betiği
  - H2: Alternatif kurulum yöntemleri
  - H3: Yerel önek kurucusu (install-cli.sh)
  - H3: npm, pnpm veya bun
  - H3: Kaynaktan
  - H3: GitHub ana dal çalışma kopyasından kurulum
  - H3: Konteynerler ve paket yöneticileri
  - H2: Kurulumu doğrulama
  - H2: Barındırma ve dağıtım
  - H2: Güncelleme, taşıma veya kaldırma
  - H2: Sorun giderme: openclaw bulunamadı

## install/installer.md

- Rota: /install/installer
- Başlıklar:
  - H2: Hızlı komutlar
  - H2: install.sh
  - H3: Akış (install.sh)
  - H3: Kaynak çalışma kopyasını algılama
  - H3: Örnekler (install.sh)
  - H2: install-cli.sh
  - H3: Akış (install-cli.sh)
  - H3: Örnekler (install-cli.sh)
  - H2: install.ps1
  - H3: Akış (install.ps1)
  - H3: Örnekler (install.ps1)
  - H2: CI ve otomasyon
  - H2: Sorun giderme
  - H2: İlgili

## install/kubernetes.md

- Rota: /install/kubernetes
- Başlıklar:
  - H2: Neden Helm değil
  - H2: Gereksinimler
  - H2: Hızlı başlangıç
  - H2: Kind ile yerel test
  - H2: Adım adım
  - H3: 1) Dağıtın
  - H3: 2) Gateway'e erişin
  - H2: Dağıtılan bileşenler
  - H2: Özelleştirme
  - H3: Aracı talimatları
  - H3: Gateway yapılandırması
  - H3: Sağlayıcı ekleme
  - H3: Özel ad alanı
  - H3: Özel imaj
  - H3: Port yönlendirmenin ötesinde erişime açma
  - H2: Yeniden dağıtma
  - H2: Kaldırma
  - H2: Mimari notları
  - H2: Dosya yapısı
  - H2: İlgili içerikler

## install/macos-vm.md

- Rota: /install/macos-vm
- Başlıklar:
  - H2: Önerilen varsayılan seçenek (çoğu kullanıcı için)
  - H2: macOS sanal makine seçenekleri
  - H3: Apple Silicon Mac'inizde yerel sanal makine (Lume)
  - H3: Barındırılan Mac sağlayıcıları (bulut)
  - H2: Hızlı yol (Lume, deneyimli kullanıcılar)
  - H2: Gereksinimler (Lume)
  - H2: 1) Lume'u yükleyin
  - H2: 2) macOS sanal makinesini oluşturun
  - H2: 3) Kurulum Yardımcısı'nı tamamlayın
  - H2: 4) Sanal makinenin IP adresini alın
  - H2: 5) SSH ile sanal makineye bağlanın
  - H2: 6) OpenClaw'u yükleyin
  - H2: 7) Kanalları yapılandırın
  - H2: 8) Sanal makineyi başsız çalıştırın
  - H2: Ek: iMessage entegrasyonu
  - H2: Altın imaj kaydetme
  - H2: 24/7 çalıştırma
  - H2: Sorun giderme
  - H2: İlgili belgeler

## install/migrating-claude.md

- Rota: /install/migrating-claude
- Başlıklar:
  - H2: İki içe aktarma yöntemi
  - H2: İçe aktarılanlar
  - H2: Yalnızca arşivde kalanlar
  - H2: Kaynak seçimi
  - H2: Önerilen akış
  - H2: Çakışmaları ele alma
  - H2: Otomasyon için JSON çıktısı
  - H2: Sorun giderme
  - H2: İlgili içerikler

## install/migrating-hermes.md

- Rota: /install/migrating-hermes
- Başlıklar:
  - H2: İki içe aktarma yöntemi
  - H2: İçe aktarılanlar
  - H2: Yalnızca arşivde kalanlar
  - H2: Önerilen akış
  - H2: Çakışmaları ele alma
  - H2: Gizli bilgiler
  - H2: Otomasyon için JSON çıktısı
  - H2: Sorun giderme
  - H2: İlgili içerikler

## install/migrating.md

- Rota: /install/migrating
- Başlıklar:
  - H2: Başka bir aracı sisteminden içe aktarma
  - H2: OpenClaw'u yeni bir makineye taşıma
  - H3: Geçiş adımları
  - H3: Yaygın sorunlar
  - H3: Doğrulama kontrol listesi
  - H2: Bir Plugin'i yerinde yükseltme
  - H2: İlgili içerikler

## install/nix.md

- Rota: /install/nix
- Başlıklar:
  - H2: Elde edecekleriniz
  - H2: Hızlı başlangıç
  - H2: Nix modu çalışma zamanı davranışı
  - H3: Nix modunda değişenler
  - H3: Yapılandırma ve durum yolları
  - H3: Hizmet PATH keşfi
  - H2: İlgili içerikler

## install/node.md

- Rota: /install/node
- Başlıklar:
  - H2: Sürümünüzü kontrol edin
  - H2: Node'u yükleyin
  - H2: Sorun giderme
  - H3: openclaw: komut bulunamadı
  - H3: npm install -g sırasında izin hataları (Linux)
  - H2: İlgili içerikler

## install/northflank.mdx

- Rota: /install/northflank
- Başlıklar:
  - H2: Başlangıç
  - H2: Elde edecekleriniz
  - H2: Bir kanal bağlama
  - H2: Sonraki adımlar

## install/oracle.md

- Rota: /install/oracle
- Başlıklar:
  - H2: Ön koşullar
  - H2: Kurulum
  - H2: Güvenlik durumunu doğrulama
  - H2: ARM notları
  - H2: Kalıcılık ve yedeklemeler
  - H2: Alternatif: SSH tüneli
  - H2: Sorun giderme
  - H2: Sonraki adımlar
  - H2: İlgili içerikler

## install/podman.md

- Rota: /install/podman
- Başlıklar:
  - H2: Ön koşullar
  - H2: Hızlı başlangıç
  - H2: Podman ve Tailscale
  - H2: Systemd (Quadlet, isteğe bağlı)
  - H2: Yapılandırma, ortam ve depolama
  - H2: İmajları yükseltme
  - H2: Yararlı komutlar
  - H2: Sorun giderme
  - H2: İlgili içerikler

## install/railway.mdx

- Rota: /install/railway
- Başlıklar:
  - H2: Tek tıklamayla dağıtım
  - H2: Elde edecekleriniz
  - H2: Bir kanal bağlama
  - H2: Yedeklemeler ve geçiş
  - H2: Sonraki adımlar

## install/raspberry-pi.md

- Rota: /install/raspberry-pi
- Başlıklar:
  - H2: Donanım uyumluluğu
  - H2: Ön koşullar
  - H2: Kurulum
  - H2: Performans ipuçları
  - H2: Önerilen model kurulumu
  - H2: ARM ikili dosyası notları
  - H2: Kalıcılık ve yedeklemeler
  - H2: Sorun giderme
  - H2: Sonraki adımlar
  - H2: İlgili içerikler

## install/render.mdx

- Rota: /install/render
- Başlıklar:
  - H2: Ön koşullar
  - H2: Dağıtım
  - H2: Blueprint
  - H2: Plan seçimi
  - H2: Dağıtımdan sonra
  - H3: Denetim Arayüzü'ne erişim
  - H3: Günlükler
  - H3: Kabuk erişimi
  - H3: Ortam değişkenleri
  - H3: Otomatik dağıtım
  - H2: Özel alan adı
  - H2: Ölçeklendirme
  - H2: Yedeklemeler ve geçiş
  - H2: Sorun giderme
  - H3: Hizmet başlamıyor
  - H3: Yavaş soğuk başlatmalar (ücretsiz katman)
  - H3: Yeniden dağıtımdan sonra veri kaybı
  - H3: Sistem durumu denetimi hataları
  - H2: Sonraki adımlar

## install/uninstall.md

- Rota: /install/uninstall
- Başlıklar:
  - H2: Kolay yol (CLI hâlâ yüklü)
  - H2: Hizmeti elle kaldırma (CLI yüklü değil)
  - H3: macOS (launchd)
  - H3: Linux (systemd kullanıcı birimi)
  - H3: Windows (Scheduled Task)
  - H2: Normal yükleme ile kaynak kod çıkışını karşılaştırma
  - H3: Normal yükleme (install.sh / npm / pnpm / bun)
  - H3: Kaynak kod çıkışı (git clone)
  - H2: İlgili içerikler

## install/updating.md

- Rota: /install/updating
- Başlıklar:
  - H2: Önerilen: openclaw update
  - H2: npm ve git yüklemeleri arasında geçiş
  - H2: Kaynak kod çıkışı sunucuları (referans betiği)
  - H2: Alternatif: yükleyiciyi yeniden çalıştırma
  - H2: Alternatif: elle npm, pnpm veya bun
  - H3: İleri düzey npm yükleme konuları
  - H2: Otomatik güncelleyici
  - H2: Güncellemeden sonra
  - H3: Doctor'ı çalıştırın
  - H3: Gateway'i yeniden başlatın
  - H3: Doğrulayın
  - H2: Geri alma
  - H3: Güncellemeden önce: doğrulanmış bir yedek oluşturun
  - H3: Paket yüklemesini geri alma
  - H3: Kaynak kod çıkışını geri alma
  - H3: Oturum SQLite geçişinden önceki sürüme düşürme
  - H3: Durumu yalnızca gerektiğinde geri yükleme
  - H3: Geri almayı doğrulama
  - H2: Takılırsanız
  - H2: İlgili içerikler

## install/upstash.md

- Rota: /install/upstash
- Başlıklar:
  - H2: Ön koşullar
  - H2: Box oluşturma
  - H2: SSH tüneliyle bağlanma
  - H2: OpenClaw'u yükleme
  - H2: İlk katılımı çalıştırma
  - H2: Gateway'i başlatma
  - H2: Otomatik yeniden başlatma
  - H2: Sorun giderme
  - H2: İlgili içerikler

## logging.md

- Rota: /logging
- Başlıklar:
  - H2: Günlüklerin konumu
  - H2: Günlükleri okuma
  - H3: CLI: canlı takip (önerilen)
  - H3: Denetim Arayüzü (web)
  - H3: Yalnızca kanal günlükleri
  - H2: Günlük biçimleri
  - H3: Dosya günlükleri (JSONL)
  - H3: Konsol çıktısı
  - H3: Gateway WebSocket günlükleri
  - H2: Günlük kaydını yapılandırma
  - H3: Günlük düzeyleri
  - H3: Hedefli model aktarımı tanılaması
  - H3: İz korelasyonu
  - H3: Model çağrısı boyutu ve zamanlaması
  - H3: Konsol stilleri
  - H3: Sansürleme
  - H2: Tanılama ve OpenTelemetry
  - H2: Sorun giderme ipuçları
  - H2: İlgili içerikler

## maturity/scorecard.md

- Rota: /maturity/scorecard
- Başlıklar:
  - H1: Olgunluk puan kartı
  - H2: Bu sayfanın amacı
  - H2: Genel bakış
  - H2: Puan aralıkları
  - H2: Yüzey gezgini
  - H2: QA kanıt özeti
  - H3: Alana göre hazırlık durumu

## maturity/taxonomy.md

- Rota: /maturity/taxonomy
- Başlıklar:
  - H1: Olgunluk sınıflandırması
  - H2: Bu sayfayı okuma
  - H2: Olgunluk düzeyleri
  - H2: Ürün alanları
  - H2: Ayrıntılar
  - H3: Çekirdek
  - H3: Platform
  - H3: Kanal
  - H3: Sağlayıcı ve araç

## network.md

- Rota: /network
- Başlıklar:
  - H2: Çekirdek model
  - H2: Eşleştirme + kimlik
  - H2: Keşif + aktarımlar
  - H2: Node'lar + aktarımlar
  - H2: Güvenlik
  - H2: İlgili içerikler

## nodes/audio.md

- Rota: /nodes/audio
- Başlıklar:
  - H2: İşlevi
  - H2: Otomatik algılama (varsayılan)
  - H2: Yapılandırma örnekleri
  - H3: Sağlayıcı + CLI alternatifi (OpenAI + Whisper CLI)
  - H3: Yalnızca sağlayıcı (Deepgram)
  - H3: Yalnızca sağlayıcı (Mistral Voxtral)
  - H3: Yalnızca sağlayıcı (SenseAudio)
  - H3: Transkripti sohbete yansıtma (isteğe bağlı)
  - H2: Notlar ve sınırlar
  - H3: Yerleşik yerel STT
  - H3: Proxy ortamı desteği
  - H2: Gruplarda bahsetme algılama
  - H2: Dikkat edilmesi gerekenler
  - H2: İlgili içerikler

## nodes/camera.md

- Rota: /nodes/camera
- Başlıklar:
  - H2: iOS Node
  - H3: iOS kullanıcı ayarı
  - H3: iOS komutları (Gateway node.invoke aracılığıyla)
  - H3: iOS ön plan gereksinimi
  - H3: CLI yardımcısı
  - H2: Android Node
  - H3: Android kullanıcı ayarı
  - H3: İzinler
  - H3: Android ön plan gereksinimi
  - H3: Android komutları (Gateway node.invoke aracılığıyla)
  - H2: macOS uygulaması
  - H3: macOS kullanıcı ayarı
  - H3: CLI yardımcısı (node invoke)
  - H2: Linux Node ana bilgisayarı
  - H2: Güvenlik + pratik sınırlar
  - H2: macOS ekran videosu (işletim sistemi düzeyinde)
  - H2: İlgili

## nodes/computer-use.md

- Rota: /nodes/computer-use
- Başlıklar:
  - H2: Gereksinimler
  - H2: Bilgisayar agent aracı
  - H2: Windows ve Linux (deneysel, cua-driver aracılığıyla)
  - H3: Sorun giderme
  - H2: computer.act Node komutu
  - H2: Etkinleştirme ve hazır hâle getirme
  - H2: Güvenlik
  - H2: Diğer masaüstü denetim yollarıyla ilişki

## nodes/images.md

- Rota: /nodes/images
- Başlıklar:
  - H2: Hedefler
  - H2: CLI yüzeyi
  - H2: WhatsApp Web kanalının davranışı
  - H2: Otomatik yanıt işlem hattı
  - H2: Gelen medyanın komutlara aktarılması
  - H2: Sınırlar ve hatalar
  - H2: Testlere ilişkin notlar
  - H2: İlgili

## nodes/index.md

- Rota: /nodes
- Başlıklar:
  - H2: Eşleştirme + durum
  - H2: Sürüm uyumsuzluğu ve yükseltme sırası
  - H2: Uzak Node ana bilgisayarı (system.run)
  - H3: Node ana bilgisayarı başlatma (ön plan)
  - H3: SSH tüneli üzerinden uzak Gateway (geri döngü bağlaması)
  - H3: Node ana bilgisayarı başlatma (hizmet)
  - H3: Eşleştirme + adlandırma
  - H3: Node üzerinde barındırılan MCP sunucuları
  - H3: Node üzerinde barındırılan Skills
  - H3: Başsız kimlik durumu
  - H3: Komutları izin verilenler listesine alma
  - H3: exec'i Node'a yönlendirme
  - H3: Yerel model çıkarımı
  - H3: Codex oturumları ve dökümleri
  - H3: Claude oturumları ve dökümleri
  - H3: OpenCode ve Pi oturumları
  - H3: Terminal dosya yüklemeleri
  - H2: Komutları çağırma
  - H2: Komut politikası
  - H2: Yapılandırma (openclaw.json)
  - H2: Ekran görüntüleri (canvas anlık görüntüleri)
  - H3: Canvas denetimleri
  - H3: A2UI (Canvas)
  - H2: Fotoğraflar + videolar (Node kamerası)
  - H2: Ekran kayıtları (Node'lar)
  - H2: Konum (Node'lar)
  - H2: SMS (Android Node'ları)
  - H2: Cihaz ve kişisel veri komutları
  - H2: Sistem komutları (Node ana bilgisayarı / Mac Node'u)
  - H2: exec Node bağlaması
  - H2: İzinler eşlemesi
  - H2: Başsız Node ana bilgisayarı (platformlar arası)
  - H2: Mac Node modu

## nodes/location-command.md

- Rota: /nodes/location-command
- Başlıklar:
  - H2: Kısaca
  - H2: Neden yalnızca bir anahtar değil de seçici kullanılır?
  - H2: Ayarlar modeli
  - H2: İzin eşlemesi (node.permissions)
  - H2: Komut: location.get
  - H2: Arka plan davranışı
  - H2: Linux Node ana bilgisayarı
  - H2: Model/araç entegrasyonu
  - H2: Kullanıcı deneyimi metni (önerilen)
  - H2: İlgili

## nodes/media-understanding.md

- Rota: /nodes/media-understanding
- Başlıklar:
  - H2: Nasıl çalışır?
  - H2: Yapılandırma
  - H3: Model girdileri
  - H3: Sağlayıcı kimlik bilgileri
  - H2: Kurallar ve davranış
  - H3: Otomatik algılama (varsayılan)
  - H3: Proxy desteği (ses/video sağlayıcısı çağrıları)
  - H2: Yetenekler
  - H2: Sağlayıcı destek matrisi
  - H2: Model seçimi rehberi
  - H2: Ek politikası
  - H3: Dosya eki çıkarma
  - H2: Yapılandırma örnekleri
  - H2: Durum çıktısı
  - H2: Notlar
  - H2: İlgili

## nodes/presence.md

- Rota: /nodes/presence
- Başlıklar:
  - H2: Gereksinimler
  - H2: Etkin bilgisayarı denetleme
  - H2: Etkinlik nasıl mevcudiyete dönüşür?
  - H2: Gizlilik ve model bağlamı
  - H2: Bağlantı uyarılarının yönlendirilme biçimi
  - H2: Sorun giderme
  - H2: İlgili

## nodes/talk.md

- Rota: /nodes/talk
- Başlıklar:
  - H2: Davranış (macOS)
  - H2: Yanıtlardaki ses yönergeleri
  - H2: Yapılandırma (`~/.openclaw/openclaw.json`)
  - H2: macOS kullanıcı arayüzü
  - H2: Android kullanıcı arayüzü
  - H2: Notlar
  - H2: İlgili

## nodes/troubleshooting.md

- Rota: /nodes/troubleshooting
- Başlıklar:
  - H2: Komut basamakları
  - H2: Ön plan gereksinimleri
  - H2: İzinler matrisi
  - H2: Eşleştirme ve onaylar
  - H2: Yaygın Node hata kodları
  - H2: Hızlı kurtarma döngüsü
  - H2: İlgili

## nodes/voicewake.md

- Rota: /nodes/voicewake
- Başlıklar:
  - H2: Depolama
  - H2: Protokol
  - H3: Tetikleyici listesi
  - H3: Yönlendirme (tetikleyiciden hedefe)
  - H3: Olaylar
  - H2: İstemci davranışı
  - H2: İlgili

## openclaw-agent-runtime.md

- Rota: /openclaw-agent-runtime
- Başlıklar:
  - H2: Tür denetimi ve lint işlemi
  - H2: Agent çalışma zamanı testlerini çalıştırma
  - H2: Manuel test
  - H2: Temiz başlangıç sıfırlaması
  - H2: Başvurular
  - H2: İlgili

## perplexity.md

- Rota: /perplexity
- Başlıklar:
  - H2: İlgili

## plan/cloud-workers.md

- Rota: /plan/cloud-workers
- Başlıklar:
  - H2: Durum
  - H2: Sorun
  - H2: Hedefler
  - H2: Hedef olmayanlar (v1)
  - H2: Önceki çalışmalar (neyi kopyalıyoruz, neyi tersine çeviriyoruz)
  - H2: Mimari kararı: döngü worker üzerinde, çıkarım Gateway üzerinden
  - H2: Bileşenler
  - H3: 1. Ortam durum makinesi + sağlayıcı sözleşmesi
  - H3: 2. Worker önyüklemesi: OpenClaw'u makineye yükleme
  - H3: 3. Aktarım: her şey SSH üzerinden
  - H3: 4. Worker protokolü (özel; Node protokolü değil)
  - H3: 5. Oturum arka ucu RPC'leri
  - H3: 6. Çalışma alanı eşitlemesi
  - H3: 7. Yerleştirme durum makinesi, oturumlar ve kullanıcı arayüzü
  - H2: Gönderim ve devir
  - H2: Güvenlik modeli
  - H2: Kapasite
  - H2: Yaşam döngüsü
  - H2: Yapılandırma yüzeyi
  - H2: Kilometre taşları
  - H2: Açık sorular

## plan/path3-sqlite-session-artifact-family.md

- Rota: /plan/path3-sqlite-session-artifact-family
- Başlıklar:
  - H1: Yol 3 SQLite oturum yapıtı ailesi
  - H2: Yetkili aile
  - H2: Geçişten sonraki aile dışı yapıtlar
  - H2: Yama noktaları
  - H2: Odaklı testler

## plan/swarms.md

- Rota: /plan/swarms
- Başlıklar:
  - H1: Sürüler — kod modunda agent dağıtımı ve orkestrasyonu
  - H2: 1. Nedir ve neden gereklidir?
  - H2: 2. Kararlar (bakım sorumlusu, 2026-07-17)
  - H2: 3. Mimariye genel bakış
  - H2: 4. Yapılandırma geçidi (v1)
  - H2: 5. Çekirdek: toplayıcı modunda başlatma + `agents_wait` (v1)
  - H3: 5.1 `sessions_spawn` eklemeleri (tümü sürünün etkin olmasına bağlı)
  - H3: 5.2 Onaylar güvenli biçimde başarısız olur
  - H3: 5.3 `agents_wait` aracı (yeni, geçitli)
  - H3: 5.4 Sınırların uygulanması
  - H2: 6. Test sözleşmesi (v1, A hattı)
  - H2: 7. QuickJS konuk yüzeyi (B hattı, çekirdekten sonra)
  - H2: 8. Codex test düzeneği izdüşümü (sonraki hat)
  - H2: 9. Kalıcılık ve saklama
  - H2: 10. İlerleme yüzeyi ("noktalar") — sonraki hat
  - H2: 11. Labs sayfası (Control UI, bağımsız hat)
  - H2: 12. Yerleştirme (daha sonra)
  - H2: 13. Hedef olmayanlar
  - H2: 14. Derleme aşamaları / PR dilimleme

## plan/ui-channels.md

- Rota: /plan/ui-channels
- Başlıklar:
  - H2: Durum
  - H2: Sorun
  - H2: Hedefler
  - H2: Hedef olmayanlar
  - H2: Hedef model
  - H2: Teslimat meta verileri
  - H2: Çalışma zamanı yetenek sözleşmesi
  - H2: Kanal eşlemesi
  - H2: Yeniden düzenleme adımları
  - H2: Testler
  - H2: Açık sorular
  - H2: İlgili

## platforms/android.md

- Rota: /platforms/android
- Başlıklar:
  - H2: Destek özeti
  - H2: Eşzamanlı Gateway oturumları
  - H2: Wear OS yardımcı uygulaması
  - H2: Google Play dışından yükleme
  - H2: Android'i uzak bir Mac'ten yansıtma ve denetleme
  - H3: Başlamadan önce
  - H3: TCP üzerinden ADB'yi etkinleştirme
  - H3: Yalnızca denetleyici Mac'e izin verme
  - H3: Bağlanma ve yansıtmayı başlatma
  - H3: Sorun giderme
  - H2: Bağlantı çalışma kılavuzu
  - H3: Ön koşullar
  - H3: 1. Gateway'i başlatma
  - H3: 2. Keşfi doğrulama (isteğe bağlı)
  - H4: Tek noktaya yayın DNS-SD ile ağlar arası keşif
  - H3: 3. Android'den bağlanma
  - H3: Eşleştirilmiş Gateway'leri yönetme
  - H3: Mevcudiyet canlılık işaretleri
  - H3: 4. Eşleştirmeyi onaylama (CLI)
  - H3: 5. Node'un bağlı olduğunu doğrulama
  - H3: 6. Sohbet + geçmiş
  - H3: 7. Canvas + kamera
  - H4: Gateway Canvas ana bilgisayarı (web içeriği için önerilir)
  - H3: 8. Ses + genişletilmiş Android komut yüzeyi
  - H3: 9. Çalışma alanı dosyaları (salt okunur)
  - H2: Komut onaylarını inceleme
  - H2: Agent sorularını yanıtlama
  - H2: Asistan giriş noktaları
  - H2: Bildirim yönlendirme
  - H2: İlgili

## platforms/digitalocean.md

- Rota: /platforms/digitalocean
- Başlıklar:
  - H2: İlgili

## platforms/easyrunner.md

- Rota: /platforms/easyrunner
- Başlıklar:
  - H2: Başlamadan önce
  - H2: Compose uygulaması
  - H2: OpenClaw'u yapılandırma
  - H2: Doğrulama
  - H2: Güncellemeler ve yedeklemeler
  - H2: Sorun giderme

## platforms/index.md

- Rota: /platforms
- Başlıklar:
  - H2: İşletim sisteminizi seçin
  - H2: VPS ve barındırma
  - H2: Yaygın bağlantılar
  - H2: Gateway hizmeti kurulumu (CLI)
  - H2: İlgili

## platforms/ios-healthkit.md

- Rota: /platforms/ios-healthkit
- Başlıklar:
  - H1: HealthKit özetleri
  - H2: Gereksinimler
  - H2: Erişimi etkinleştirme
  - H3: 1. Gateway komutunu yetkilendirme
  - H3: 2. iOS aygıtında paylaşımı etkinleştirme
  - H2: Bugünün özetini isteme
  - H2: Gizlilik davranışı
  - H2: Sorun giderme
  - H3: Komut Node tarafından bildirilmemiş
  - H3: Komut açıkça etkinleştirme gerektiriyor
  - H3: `HEALTH_ACCESS_DISABLED`
  - H3: Özet başarılı ancak metrikler eksik
  - H3: Daha eski aralıklar başarısız oluyor
  - H2: İlgili

## platforms/ios.md

- Rota: /platforms/ios
- Başlıklar:
  - H2: Ne işe yarar
  - H2: Gereksinimler
  - H2: Hızlı başlangıç (eşleştirme + bağlanma)
  - H2: Sağlık özetleri
  - H2: Komut onaylarını inceleme
  - H2: Aracı sorularını yanıtlama
  - H2: İsteğe bağlı doğrudan Apple Watch Node'u
  - H2: Resmî derlemeler için aktarıcı destekli anlık bildirim
  - H2: Arka planda etkinlik sinyalleri
  - H2: Kimlik doğrulama ve güven akışı
  - H2: Keşif yolları
  - H3: Bonjour (LAN)
  - H3: Tailnet (ağlar arası)
  - H3: Manuel ana makine/bağlantı noktası
  - H2: Birden fazla Gateway
  - H2: Canvas + A2UI
  - H2: Bilgisayar Kullanımı ile ilişkisi
  - H3: Canvas değerlendirmesi / anlık görüntüsü
  - H2: Sesle uyandırma + konuşma modu
  - H2: Yaygın hatalar
  - H2: İlgili belgeler

## platforms/linux.md

- Rota: /platforms/linux
- Başlıklar:
  - H2: Masaüstü yardımcı uygulaması
  - H3: Hızlı Sohbet
  - H3: Canvas
  - H2: CLI ve SSH alternatifi
  - H2: Node yetenekleri
  - H2: Kurulum
  - H2: Gateway hizmeti (systemd)
  - H2: Bellek baskısı ve OOM sonlandırmaları
  - H2: İlgili

## platforms/mac/bundled-gateway.md

- Rota: /platforms/mac/bundled-gateway
- Başlıklar:
  - H2: Otomatik kurulum
  - H2: Manuel kurtarma
  - H2: Launchd (LaunchAgent olarak Gateway)
  - H2: Sürüm uyumluluğu
  - H2: macOS'te durum dizini
  - H2: Uygulama bağlantısında hata ayıklama
  - H2: Temel kontrol
  - H2: İlgili

## platforms/mac/canvas.md

- Rota: /platforms/mac/canvas
- Başlıklar:
  - H2: Canvas'ın bulunduğu yer
  - H2: Panel davranışı
  - H2: Aracı API yüzeyi
  - H2: Canvas'ta A2UI
  - H3: A2UI komutları (v0.8)
  - H2: Canvas'tan aracı çalıştırmalarını tetikleme
  - H2: Güvenlik notları
  - H2: İlgili

## platforms/mac/child-process.md

- Rota: /platforms/mac/child-process
- Başlıklar:
  - H2: Varsayılan davranış (launchd)
  - H2: İmzasız geliştirme derlemeleri
  - H2: Yalnızca bağlanma modu
  - H2: Uzak mod
  - H2: Neden launchd'yi tercih ediyoruz
  - H2: İlgili

## platforms/mac/dev-setup.md

- Rota: /platforms/mac/dev-setup
- Başlıklar:
  - H1: macOS geliştirici kurulumu
  - H2: Ön koşullar
  - H2: 1. Bağımlılıkları yükleme
  - H2: 2. Uygulamayı derleme ve paketleme
  - H2: 3. CLI ve Gateway'i yükleme
  - H2: Sorun giderme
  - H3: Derleme başarısız: araç zinciri veya SDK uyumsuzluğu
  - H3: İzin verildiğinde uygulama çöküyor
  - H3: Gateway süresiz olarak "Starting..." durumunda kalıyor
  - H2: İlgili

## platforms/mac/health.md

- Rota: /platforms/mac/health
- Başlıklar:
  - H1: macOS'te sistem durumu kontrolleri
  - H2: Menü çubuğu
  - H2: Ayarlar
  - H2: Yoklamanın çalışma şekli
  - H2: Emin olunmadığında
  - H2: İlgili

## platforms/mac/icon.md

- Rota: /platforms/mac/icon
- Başlıklar:
  - H1: Menü Çubuğu Simgesi Durumları
  - H2: Durumlar
  - H2: Sesle uyandırma kulakları
  - H2: Şekiller ve boyutlar
  - H2: Davranış notları
  - H2: İlgili

## platforms/mac/logging.md

- Rota: /platforms/mac/logging
- Başlıklar:
  - H1: Günlük kaydı (macOS)
  - H2: Dönen tanılama dosyası günlüğü (Hata Ayıklama bölmesi)
  - H2: macOS'te birleşik günlük kaydındaki özel veriler
  - H2: OpenClaw (ai.openclaw) için etkinleştirme
  - H2: Hata ayıklamadan sonra devre dışı bırakma
  - H2: İlgili

## platforms/mac/menu-bar.md

- Rota: /platforms/mac/menu-bar
- Başlıklar:
  - H2: Gösterilenler
  - H2: Durum modeli
  - H2: IconState enum'u (Swift)
  - H3: ActivityKind -&gt; rozet simgesi
  - H3: Görsel eşleme
  - H2: Bağlam alt menüsü
  - H2: Durum satırı metni (menü)
  - H2: Olay alımı
  - H2: Hata ayıklama geçersiz kılması
  - H2: Test kontrol listesi
  - H2: İlgili

## platforms/mac/peekaboo.md

- Rota: /platforms/mac/peekaboo
- Başlıklar:
  - H2: Bunun ne olduğu (ve ne olmadığı)
  - H2: Diğer masaüstü denetim yollarıyla ilişkisi
  - H2: Köprüyü etkinleştirme
  - H2: İstemci keşif sırası
  - H2: Güvenlik ve izinler
  - H2: Anlık görüntü davranışı (otomasyon)
  - H2: Sorun giderme
  - H2: İlgili

## platforms/mac/permissions.md

- Rota: /platforms/mac/permissions
- Başlıklar:
  - H2: Kararlı izinler için gereksinimler
  - H2: Node ve CLI çalışma zamanları için Erişilebilirlik izinleri
  - H2: İstemler kaybolduğunda kurtarma kontrol listesi
  - H2: Dosya ve klasör izinleri (Masaüstü/Belgeler/İndirilenler)
  - H2: İlgili

## platforms/mac/remote.md

- Rota: /platforms/mac/remote
- Başlıklar:
  - H2: Modlar
  - H2: Uzak aktarımlar
  - H2: Uzak ana makinedeki ön koşullar
  - H2: macOS uygulaması kurulumu
  - H2: Web Sohbeti
  - H2: İzinler
  - H2: Güvenlik notları
  - H2: WhatsApp oturum açma akışı (uzak)
  - H2: Sorun giderme
  - H2: Bildirim sesleri
  - H2: İlgili

## platforms/mac/signing.md

- Rota: /platforms/mac/signing
- Başlıklar:
  - H1: mac imzalama (hata ayıklama derlemeleri)
  - H2: Kullanım
  - H3: Geçici imzalama notu
  - H2: Hakkında bölümü için derleme meta verileri
  - H2: İlgili

## platforms/mac/skills.md

- Rota: /platforms/mac/skills
- Başlıklar:
  - H2: Veri kaynağı
  - H2: Yükleme eylemleri
  - H2: Ortam/API anahtarları
  - H2: Uzak mod
  - H2: İlgili

## platforms/mac/voice-overlay.md

- Rota: /platforms/mac/voice-overlay
- Başlıklar:
  - H1: Ses Katmanı Yaşam Döngüsü (macOS)
  - H2: Davranış
  - H2: Uygulama
  - H2: Günlük kaydı
  - H2: Hata ayıklama kontrol listesi
  - H2: İlgili

## platforms/mac/voicewake.md

- Rota: /platforms/mac/voicewake
- Başlıklar:
  - H1: Sesle Uyandırma ve Bas-Konuş
  - H2: Gereksinimler
  - H2: Modlar
  - H2: Çalışma zamanı davranışı (uyandırma sözcüğü)
  - H2: Yaşam döngüsü değişmezleri
  - H2: Bas-konuş ayrıntıları
  - H2: Kullanıcıya yönelik ayarlar
  - H2: İletme davranışı
  - H2: İletme yükü
  - H2: Hızlı doğrulama
  - H2: İlgili

## platforms/mac/webchat.md

- Rota: /platforms/mac/webchat
- Başlıklar:
  - H2: Birden fazla Gateway penceresi
  - H2: Hızlı Sohbet çubuğu
  - H2: Başlatma ve hata ayıklama
  - H2: Bağlantı yapısı
  - H2: Güvenlik yüzeyi
  - H2: Bilinen sınırlamalar
  - H2: İlgili

## platforms/mac/xpc.md

- Rota: /platforms/mac/xpc
- Başlıklar:
  - H1: OpenClaw macOS IPC mimarisi
  - H2: Hedefler
  - H2: Çalışma şekli
  - H3: Gateway + Node aktarımı
  - H3: Node hizmeti + uygulama IPC'si
  - H3: PeekabooBridge (kullanıcı arayüzü otomasyonu)
  - H2: İşletim akışları
  - H2: Güçlendirme notları
  - H2: İlgili

## platforms/macos.md

- Rota: /platforms/macos
- Başlıklar:
  - H2: İndirme
  - H2: İlk çalıştırma
  - H2: Güncellemeler
  - H2: Kontrol paneli bağlantılarını açma
  - H2: Tarayıcı oturum açma bilgilerini içe aktarma
  - H2: Gateway modu seçme
  - H2: Uygulamanın sorumlulukları
  - H2: macOS ayrıntı sayfaları
  - H2: İlgili

## platforms/oracle.md

- Rota: /platforms/oracle
- Başlıklar:
  - H2: İlgili

## platforms/raspberry-pi.md

- Rota: /platforms/raspberry-pi
- Başlıklar:
  - H2: İlgili

## platforms/windows.md

- Rota: /platforms/windows
- Başlıklar:
  - H2: Önerilen: Windows Hub
  - H3: Windows Hub'ın içerdikleri
  - H3: İlk başlatma
  - H2: Windows Node modu
  - H2: Yerel MCP modu
  - H2: Yerel Windows CLI ve Gateway
  - H2: WSL2 Gateway
  - H2: Windows oturumundan önce Gateway'i otomatik başlatma
  - H2: WSL hizmetlerini LAN üzerinden kullanıma açma
  - H2: Sorun giderme
  - H3: Sistem tepsisi simgesi görünmüyor
  - H3: Yerel kurulum başarısız oluyor
  - H3: Uygulama eşleştirme gerektiğini belirtiyor
  - H3: Web sohbeti uzak Gateway'e erişemiyor
  - H3: screen.snapshot, kamera veya ses komutları başarısız oluyor
  - H3: Git veya GitHub bağlantısı başarısız oluyor
  - H2: İlgili

## plugins/adding-capabilities.md

- Rota: /plugins/adding-capabilities
- Başlıklar:
  - H2: Ne zaman yetenek oluşturulmalı
  - H2: Standart sıra
  - H2: Neyin nereye ait olduğu
  - H2: Sağlayıcı ve test düzeneği bağlantı noktaları
  - H2: Dosya kontrol listesi
  - H2: Uygulamalı örnek: görüntü oluşturma
  - H2: Gömme sağlayıcıları
  - H2: İnceleme kontrol listesi
  - H2: İlgili

## plugins/admin-http-rpc.md

- Rota: /plugins/admin-http-rpc
- Başlıklar:
  - H2: Etkinleştirmeden önce
  - H2: Etkinleştirme
  - H2: Rotayı doğrulama
  - H2: Kimlik doğrulama
  - H2: Güvenlik modeli
  - H2: İstek
  - H2: Yanıt
  - H2: İzin verilen yöntemler
  - H2: WebSocket karşılaştırması
  - H2: Sorun giderme
  - H2: İlgili

## plugins/agent-tools.md

- Rota: /plugins/agent-tools
- Başlıklar:
  - H2: İlgili

## plugins/architecture-internals.md

- Rota: /plugins/architecture-internals
- Başlıklar:
  - H2: Yükleme işlem hattı
  - H3: Önce manifest davranışı
  - H3: Plugin önbelleği sınırı
  - H2: Kayıt modeli
  - H2: Konuşma bağlama geri çağrıları
  - H2: Sağlayıcı çalışma zamanı kancaları
  - H3: Kanca sırası ve kullanımı
  - H3: Sağlayıcı örneği
  - H3: Yerleşik örnekler
  - H2: Çalışma zamanı yardımcıları
  - H3: api.runtime.imageGeneration
  - H2: Gateway HTTP rotaları
  - H2: Plugin SDK içe aktarma yolları
  - H2: Mesaj aracı şemaları
  - H2: Kanal hedefi çözümleme
  - H2: Yapılandırma destekli dizinler
  - H2: Sağlayıcı katalogları
  - H2: Salt okunur kanal incelemesi
  - H2: Paket grupları
  - H3: Kanal kataloğu meta verileri
  - H2: Bağlam motoru Pluginleri
  - H2: Yeni bir yetenek ekleme
  - H3: Yetenek kontrol listesi
  - H3: Yetenek şablonu
  - H2: İlgili

## plugins/architecture.md

- Rota: /plugins/architecture
- Başlıklar:
  - H2: Genel yetenek modeli
  - H3: Harici uyumluluk yaklaşımı
  - H3: Plugin biçimleri
  - H3: Uyumluluk sinyalleri
  - H2: Mimariye genel bakış
  - H3: Plugin meta verisi anlık görüntüsü ve arama tablosu
  - H3: Etkinleştirme planlaması
  - H3: Kanal Pluginleri ve paylaşılan mesaj aracı
  - H2: Yetenek sahipliği modeli
  - H3: Yetenek katmanlama
  - H3: Çok yetenekli şirket Plugini örneği
  - H3: Yetenek örneği: video anlama
  - H2: Sözleşmeler ve uygulama
  - H3: Bir sözleşmede bulunması gerekenler
  - H2: Yürütme modeli
  - H2: Dışa aktarma sınırı
  - H2: Dahili bileşenler ve başvuru
  - H2: İlgili

## plugins/building-extensions.md

- Rota: /plugins/building-extensions
- Başlıklar:
  - H2: İlgili

## plugins/building-plugins.md

- Rota: /plugins/building-plugins
- Başlıklar:
  - H2: Gereksinimler
  - H2: Plugin biçimini seçme
  - H2: Hızlı başlangıç
  - H2: Araçları kaydetme
  - H2: İçe aktarma kuralları
  - H2: Gönderim öncesi kontrol listesi
  - H2: Beta sürümlerine karşı test
  - H2: Sonraki adımlar
  - H2: İlgili

## plugins/bundles.md

- Rota: /plugins/bundles
- Başlıklar:
  - H2: Paket gruplarının var olma nedeni
  - H2: Bir paket grubu yükleme
  - H2: OpenClaw'un paket gruplarından eşlediği öğeler
  - H3: Şu anda desteklenenler
  - H4: Skill içeriği
  - H4: Kanca paketleri
  - H4: Gömülü OpenClaw için MCP
  - H4: Gömülü OpenClaw ayarları
  - H4: Gömülü OpenClaw LSP
  - H3: Algılanan ancak yürütülmeyenler
  - H2: Paket grubu biçimleri
  - H2: Algılama önceliği
  - H2: Çalışma zamanı bağımlılıkları ve temizleme
  - H2: Güvenlik
  - H2: Sorun giderme
  - H2: İlgili

## plugins/cli-backend-plugins.md

- Rota: /plugins/cli-backend-plugins
- Başlıklar:
  - H2: Pluginin sahip olduğu öğeler
  - H2: Asgari arka uç Plugini
  - H2: Yapılandırma biçimi
  - H2: Gelişmiş arka uç kancaları
  - H3: ownsNativeCompaction: OpenClaw Compaction işleminden çıkma
  - H2: MCP araç köprüsü
  - H2: Arka ucu seçme
  - H2: Doğrulama
  - H2: Kontrol listesi
  - H2: İlgili

## plugins/codex-computer-use.md

- Rota: /plugins/codex-computer-use
- Başlıklar:
  - H2: OpenClaw.app ve Peekaboo
  - H2: iOS uygulaması
  - H2: Doğrudan cua-driver MCP
  - H2: Hızlı kurulum
  - H2: Komutlar
  - H2: Pazar yeri seçenekleri
  - H2: Birlikte gelen macOS pazar yeri
  - H3: Paylaşılan Plugin önbelleği
  - H2: Uzak katalog sınırı
  - H2: Yapılandırma başvurusu
  - H2: OpenClaw'un denetlediği öğeler
  - H2: macOS izinleri
  - H2: Sorun giderme
  - H2: İlgili

## plugins/codex-harness-reference.md

- Rota: /plugins/codex-harness-reference
- Başlıklar:
  - H2: Plugin yapılandırma yüzeyi
  - H2: Gözetim
  - H2: Uygulama sunucusu taşıması
  - H2: Onay ve korumalı alan modları
  - H2: Korumalı alanda yerel yürütme
  - H2: Kimlik doğrulama ve ortam yalıtımı
  - H2: Dinamik araçlar
  - H2: Zaman aşımları
  - H2: Model keşfi
  - H2: Çalışma alanı önyükleme dosyaları
  - H2: Ortam geçersiz kılmaları
  - H2: İlgili

## plugins/codex-harness-runtime.md

- Rota: /plugins/codex-harness-runtime
- Başlıklar:
  - H2: Genel bakış
  - H2: İş parçacığı bağlamaları ve model değişiklikleri
  - H2: Gözetim ve güvenli devam
  - H2: Görünür yanıtlar ve Heartbeat'ler
  - H2: Kanca sınırları
  - H2: V1 destek sözleşmesi
  - H2: Yerel izinler ve MCP bilgi istemleri
  - H2: Kuyruk yönlendirmesi
  - H2: Codex geri bildirimi yükleme
  - H2: Compaction ve transkript yansıtma
  - H2: Medya ve teslimat
  - H2: İlgili

## plugins/codex-harness.md

- Rota: /plugins/codex-harness
- Başlıklar:
  - H2: Gereksinimler
  - H2: Hızlı başlangıç
  - H2: İş parçacıklarını Codex Desktop ve CLI ile paylaşma
  - H2: Codex oturumlarını gözetme
  - H2: Yapılandırma
  - H3: Compaction
  - H3: Doğrudan API uzun bağlamı
  - H2: Codex çalışma zamanını doğrulama
  - H2: Yönlendirme ve model seçimi
  - H2: Dağıtım kalıpları
  - H3: Temel Codex dağıtımı
  - H3: Karma sağlayıcı dağıtımı
  - H3: Kapalı başarısız Codex dağıtımı
  - H2: Uygulama sunucusu politikası
  - H2: Komutlar ve tanılama
  - H3: Codex iş parçacıklarını yerel olarak inceleme
  - H3: Kimlik doğrulama sırası
  - H3: Ortam yalıtımı
  - H3: Dinamik araçlar ve web araması
  - H3: Yapılandırma alanları
  - H3: Dinamik araç çağrısı zaman aşımları
  - H3: Yerel test ortamı geçersiz kılmaları
  - H2: Yerel Codex Pluginleri
  - H2: Bilgisayar Kullanımı
  - H2: Çalışma zamanı sınırları
  - H2: Sorun giderme
  - H2: İlgili

## plugins/codex-native-plugins.md

- Rota: /plugins/codex-native-plugins
- Başlıklar:
  - H2: Gereksinimler
  - H2: Hızlı başlangıç
  - H2: Pluginleri sohbetten yönetme
  - H2: Yerel Plugin kurulumunun çalışma biçimi
  - H2: V1 destek sınırı
  - H2: Uygulama envanteri ve sahipliği
  - H2: Bağlı hesap uygulamaları
  - H2: İş parçacığı uygulaması yapılandırması
  - H2: Yıkıcı eylem politikası
  - H2: Sorun giderme
  - H2: İlgili

## plugins/codex-supervision.md

- Rota: /plugins/codex-supervision
- Başlıklar:
  - H2: Başlamadan önce
  - H2: Gözetimi etkinleştirme
  - H2: Operatör CLI'sini kullanma
  - H2: Yerel bir oturumdan dal oluşturma
  - H2: Yerel bir oturumu arşivleme
  - H2: Eşleştirilmiş Node sınırlarını anlama
  - H2: Meta veriler ve izinler
  - H3: Uyumluluk araçları
  - H2: Sorun giderme
  - H2: İlgili

## plugins/community.md

- Rota: /plugins/community
- Başlıklar:
  - H2: Pluginleri bulma
  - H2: Pluginleri yayımlama
  - H2: İlgili

## plugins/compatibility.md

- Rota: /plugins/compatibility
- Başlıklar:
  - H2: Uyumluluk kaydı
  - H2: Kullanımdan kaldırma politikası
  - H2: Mevcut uyumluluk alanları
  - H3: WhatsApp gelen geri çağrı düz takma adları
  - H3: WhatsApp gelen kabul alanları
  - H2: Plugin inceleyici paketi
  - H3: Bakım sorumlusu kabul hattı
  - H2: Sürüm notları

## plugins/copilot.md

- Rota: /plugins/copilot
- Başlıklar:
  - H2: Gereksinimler
  - H2: Yükleme
  - H2: Hızlı başlangıç
  - H2: Desteklenen sağlayıcılar
  - H2: BYOK
  - H2: Kimlik doğrulama
  - H2: Yapılandırma yüzeyi
  - H2: Compaction
  - H2: Transkript yansıtma
  - H2: Yan sorular (/btw)
  - H2: Doctor
  - H2: Sınırlamalar
  - H2: İzinler ve askuser
  - H3: Oturum düzeyinde GitHub belirteci
  - H2: İlgili

## plugins/dependency-resolution.md

- Rota: /plugins/dependency-resolution
- Başlıklar:
  - H2: Sorumluluk ayrımı
  - H2: Yükleme kökleri
  - H2: Yerel Pluginler
  - H2: Başlatma ve yeniden yükleme
  - H2: Birlikte gelen Pluginler
  - H2: Eski öğeleri temizleme

## plugins/google-meet.md

- Rota: /plugins/google-meet
- Başlıklar:
  - H2: Hızlı başlangıç
  - H3: Toplantı oluşturma
  - H3: Yalnızca gözlem amaçlı katılım
  - H3: Gerçek zamanlı oturum durumu
  - H2: Yerel Gateway + Parallels Chrome
  - H3: Yaygın hata denetimleri
  - H2: Yükleme notları
  - H2: Taşıma yöntemleri
  - H3: Chrome
  - H3: Twilio
  - H2: OAuth ve ön denetim
  - H3: Google kimlik bilgileri oluşturma
  - H3: Yenileme belirtecini oluşturma
  - H3: OAuth'u doctor ile doğrulama
  - H3: Çözümleme, ön denetim ve yapıtları okuma
  - H3: Canlı duman testi
  - H3: Örnekler oluşturma
  - H2: Yapılandırma
  - H3: Varsayılanlar
  - H3: İsteğe bağlı geçersiz kılmalar
  - H2: Araç
  - H2: Aracı ve çift yönlü modlar
  - H2: Canlı test kontrol listesi
  - H2: Sorun giderme
  - H3: Aracı Google Meet aracını göremiyor
  - H3: Bağlı, Google Meet özellikli Node yok
  - H3: Tarayıcı açılıyor ancak aracı katılamıyor
  - H3: Toplantı oluşturma başarısız oluyor
  - H3: Aracı katılıyor ancak konuşmuyor
  - H3: Twilio kurulum denetimleri başarısız oluyor
  - H3: Twilio araması başlıyor ancak toplantıya hiç girmiyor
  - H2: Notlar
  - H2: İlgili

## plugins/hooks.md

- Rota: /plugins/hooks
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Kanca kataloğu
  - H3: Kanal eşleştirme istekleri
  - H2: Çalışma zamanı kancalarında hata ayıklama
  - H2: Araç çağrısı politikası
  - H3: Tek dosyada gönderene duyarlı politika
  - H3: Yürütme ortamı kancası
  - H3: Araç sonucu kalıcılığı
  - H2: İstem ve model kancaları
  - H3: Oturum uzantıları ve sonraki tur eklemeleri
  - H2: Mesaj kancaları
  - H2: Kurulum kancaları
  - H2: Gateway yaşam döngüsü
  - H3: Güvenli harici Cron yansıtması
  - H2: Yaklaşan kullanımdan kaldırmalar
  - H2: İlgili içerikler

## plugins/install-overrides.md

- Rota: /plugins/install-overrides
- Başlıklar:
  - H2: Ortam
  - H2: Davranış
  - H2: Paket E2E

## plugins/llama-cpp.md

- Rota: /plugins/llama-cpp
- Başlıklar:
  - H2: Yerel metin çıkarımı
  - H3: Başka bir GGUF modeli kullanma
  - H2: Bellek gömme yapılandırması
  - H2: Yerel çalışma zamanı
  - H2: Bellek çalışma zamanı tanılaması
  - H2: Sorun giderme

## plugins/logbook.md

- Rota: /plugins/logbook
- Başlıklar:
  - H2: Başlamadan önce
  - H2: Hızlı başlangıç
  - H2: Nasıl çalışır
  - H2: Model ve veri akışı
  - H2: Yapılandırma
  - H3: Görüntü modeli seçimi
  - H2: Kontrol paneli sekmesi
  - H2: Gateway yöntemleri
  - H2: Gizlilik notları
  - H2: Sorun giderme
  - H3: Logbook sekmesi görünmüyor
  - H3: Yakalama bir hata bildiriyor
  - H3: Yakalamalar başarılı ancak kartlar görünmüyor
  - H2: İlgili içerikler

## plugins/manage-plugins.md

- Rota: /plugins/manage-plugins
- Başlıklar:
  - H2: Denetim Arayüzünü kullanma
  - H2: Plugin'leri listeleme ve arama
  - H2: Plugin'leri etkinleştirme ve devre dışı bırakma
  - H2: Plugin'leri yükleme
  - H2: Yeniden başlatma ve inceleme
  - H2: Plugin'leri güncelleme
  - H2: Plugin'leri kaldırma
  - H2: Kaynak seçme
  - H2: Plugin'leri yayımlama
  - H2: İlgili içerikler

## plugins/manifest.md

- Rota: /plugins/manifest
- Başlıklar:
  - H2: Bu dosyanın işlevi
  - H2: Asgari örnek
  - H2: Kapsamlı örnek
  - H2: Üst düzey alan başvurusu
  - H2: MCP sunucusu başvurusu
  - H2: dashboard başvurusu
  - H2: catalog başvurusu
  - H2: Üretim sağlayıcısı meta verileri başvurusu
  - H2: Araç meta verileri başvurusu
  - H2: providerAuthChoices başvurusu
  - H2: commandAliases başvurusu
  - H2: activation başvurusu
  - H2: qaRunners başvurusu
  - H2: setup başvurusu
  - H3: setup.providers başvurusu
  - H3: setup alanları
  - H2: uiHints başvurusu
  - H2: contracts başvurusu
  - H2: configContracts başvurusu
  - H2: mediaUnderstandingProviderMetadata başvurusu
  - H2: channelConfigs başvurusu
  - H3: Başka bir kanal Plugin'inin yerine geçme
  - H2: modelSupport başvurusu
  - H2: modelCatalog başvurusu
  - H2: modelIdNormalization başvurusu
  - H2: providerEndpoints başvurusu
  - H2: providerRequest başvurusu
  - H2: secretProviderIntegrations başvurusu
  - H2: modelPricing başvurusu
  - H3: OpenClaw Sağlayıcı Dizini
  - H2: Manifest ile package.json karşılaştırması
  - H3: Keşfi etkileyen package.json alanları
  - H2: Keşif önceliği (yinelenen Plugin kimlikleri)
  - H2: JSON Schema gereksinimleri
  - H2: Doğrulama davranışı
  - H2: Notlar
  - H2: İlgili içerikler

## plugins/meeting-plugins.md

- Rota: /plugins/meeting-plugins
- Başlıklar:
  - H2: Plugin seçme
  - H2: Mod seçme
  - H2: Chrome ve sesi hazırlama
  - H2: Plugin'leri yükleme veya devre dışı bırakma
  - H2: Doğrulama ve katılma
  - H2: Platform politikası istemlerini yönetme
  - H2: Discord sesli sohbeti
  - H2: Platform kılavuzları

## plugins/memory-lancedb.md

- Rota: /plugins/memory-lancedb
- Başlıklar:
  - H2: Kurulum
  - H2: Hızlı başlangıç
  - H2: Gömme yapılandırması
  - H3: Boyutlar
  - H2: Ollama gömmeleri
  - H2: Hatırlama ve yakalama sınırları
  - H2: Komutlar
  - H2: Depolama
  - H2: Çalışma zamanı bağımlılıkları ve platform desteği
  - H2: Sorun giderme
  - H3: Girdi uzunluğu bağlam uzunluğunu aşıyor
  - H3: Desteklenmeyen gömme modeli
  - H3: Plugin yükleniyor ancak hiçbir anı görünmüyor
  - H2: İlgili içerikler

## plugins/memory-wiki.md

- Rota: /plugins/memory-wiki
- Başlıklar:
  - H2: Kasa modları
  - H2: Kasa düzeni
  - H2: Open Knowledge Format içe aktarımları
  - H2: Yapılandırılmış iddialar ve kanıtlar
  - H2: Aracıya yönelik varlık meta verileri
  - H2: Derleme işlem hattı
  - H2: Kontrol panelleri ve durum raporları
  - H2: Arama ve getirme
  - H2: Aracı araçları
  - H2: İstem ve bağlam davranışı
  - H2: Yapılandırma
  - H3: Aracı başına kasalar
  - H3: Örnek: QMD + köprü modu
  - H2: CLI
  - H2: Obsidian desteği
  - H2: Önerilen iş akışı
  - H2: İlgili belgeler

## plugins/message-presentation.md

- Rota: /plugins/message-presentation
- Başlıklar:
  - H2: Sözleşme
  - H2: Üretici örnekleri
  - H2: Oluşturucu sözleşmesi
  - H2: Çekirdek oluşturma akışı
  - H2: İşlev kaybı kuralları
  - H3: Düğme değeri geri dönüşünün görünürlüğü
  - H2: Sağlayıcı eşlemesi
  - H2: Sunum ile InteractiveReply karşılaştırması
  - H2: Teslimat sabitlemesi
  - H2: Plugin yazarı kontrol listesi
  - H2: İlgili belgeler

## plugins/oc-path.md

- Rota: /plugins/oc-path
- Başlıklar:
  - H2: Neden etkinleştirilmeli
  - H2: Nerede çalışır
  - H2: Etkinleştirme
  - H2: Bağımlılıklar
  - H2: Sağladıkları
  - H2: Diğer Plugin'lerle ilişkisi
  - H2: Güvenlik
  - H2: İlgili içerikler

## plugins/onepassword.md

- Rota: /plugins/onepassword
- Başlıklar:
  - H1: 1Password gizli bilgi aracısı
  - H2: Güvenlik modeli
  - H2: Başlamadan önce
  - H2: Kayıtlı gizli bilgileri yapılandırma
  - H2: Aracı aracını kullanma
  - H2: Politika katmanları ve onaylar
  - H2: Durumu ve denetim geçmişini inceleme
  - H2: 1Password CLI davranışı
  - H2: Hata kodları

## plugins/plugin-inventory.md

- Rota: /plugins/plugin-inventory
- Başlıklar:
  - H1: Plugin envanteri
  - H2: Tanımlar
  - H2: Plugin yükleme
  - H2: Çekirdek npm paketi
  - H2: Resmî harici paketler
  - H2: Yalnızca kaynak kodu kullanıma alma

## plugins/plugin-permission-requests.md

- Rota: /plugins/plugin-permission-requests
- Başlıklar:
  - H2: Doğru geçidi seçme
  - H2: Araç çağrısından önce onay isteme
  - H2: Karar davranışı
  - H2: Onay istemlerini yönlendirme
  - H2: Codex yerel izinleri
  - H2: Sorun giderme
  - H2: İlgili içerikler

## plugins/reference.md

- Rota: /plugins/reference
- Başlıklar:
  - H1: Plugin başvurusu

## plugins/reference/acpx.md

- Rota: /plugins/reference/acpx
- Başlıklar:
  - H1: ACPx Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: Pi yerel oturumları
  - H2: İlgili belgeler

## plugins/reference/admin-http-rpc.md

- Rota: /plugins/reference/admin-http-rpc
- Başlıklar:
  - H1: Yönetici Http Rpc Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/alibaba.md

- Rota: /plugins/reference/alibaba
- Başlıklar:
  - H1: Alibaba Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/amazon-bedrock-mantle.md

- Rota: /plugins/reference/amazon-bedrock-mantle
- Başlıklar:
  - H1: Amazon Bedrock Mantle Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/amazon-bedrock.md

- Rota: /plugins/reference/amazon-bedrock
- Başlıklar:
  - H1: Amazon Bedrock Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/anthropic-vertex.md

- Rota: /plugins/reference/anthropic-vertex
- Başlıklar:
  - H1: Anthropic Vertex Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: Claude Fable 5
  - H2: Claude Sonnet 5

## plugins/reference/anthropic.md

- Rota: /plugins/reference/anthropic
- Başlıklar:
  - H1: Anthropic Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/arcee.md

- Rota: /plugins/reference/arcee
- Başlıklar:
  - H1: Arcee Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/azure-speech.md

- Rota: /plugins/reference/azure-speech
- Başlıklar:
  - H1: Azure Speech Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/baseten.md

- Rota: /plugins/reference/baseten
- Başlıklar:
  - H1: Baseten Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/bonjour.md

- Rota: /plugins/reference/bonjour
- Başlıklar:
  - H1: Bonjour Plugin'i
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/brave.md

- Rota: /plugins/reference/brave
- Başlıklar:
  - H1: Brave Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/browser.md

- Rota: /plugins/reference/browser
- Başlıklar:
  - H1: Tarayıcı plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/byteplus.md

- Rota: /plugins/reference/byteplus
- Başlıklar:
  - H1: BytePlus plugin'i
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/canvas.md

- Rota: /plugins/reference/canvas
- Başlıklar:
  - H1: Canvas plugin'i
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/cerebras.md

- Rota: /plugins/reference/cerebras
- Başlıklar:
  - H1: Cerebras plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/chutes.md

- Rota: /plugins/reference/chutes
- Başlıklar:
  - H1: Chutes plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/clawrouter.md

- Rota: /plugins/reference/clawrouter
- Başlıklar:
  - H1: ClawRouter plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/clickclack.md

- Rota: /plugins/reference/clickclack
- Başlıklar:
  - H1: Clickclack plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/cloudflare-ai-gateway.md

- Rota: /plugins/reference/cloudflare-ai-gateway
- Başlıklar:
  - H1: Cloudflare AI Gateway plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/codex.md

- Rota: /plugins/reference/codex
- Başlıklar:
  - H1: Codex plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/cohere.md

- Rota: /plugins/reference/cohere
- Başlıklar:
  - H1: Cohere plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/comfy.md

- Rota: /plugins/reference/comfy
- Başlıklar:
  - H1: ComfyUI plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/copilot-proxy.md

- Rota: /plugins/reference/copilot-proxy
- Başlıklar:
  - H1: Copilot Proxy plugin'i
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/copilot.md

- Rota: /plugins/reference/copilot
- Başlıklar:
  - H1: Copilot plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/crabbox.md

- Rota: /plugins/reference/crabbox
- Başlıklar:
  - H1: Crabbox plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: Yapılandırma

## plugins/reference/cua-computer.md

- Rota: /plugins/reference/cua-computer
- Başlıklar:
  - H1: Cua Computer plugin'i
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/deepgram.md

- Rota: /plugins/reference/deepgram
- Başlıklar:
  - H1: Deepgram plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/deepinfra.md

- Rota: /plugins/reference/deepinfra
- Başlıklar:
  - H1: DeepInfra plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/deepseek.md

- Rota: /plugins/reference/deepseek
- Başlıklar:
  - H1: DeepSeek plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/diagnostics-otel.md

- Rota: /plugins/reference/diagnostics-otel
- Başlıklar:
  - H1: OpenTelemetry tanılama plugin'i
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/diagnostics-prometheus.md

- Rota: /plugins/reference/diagnostics-prometheus
- Başlıklar:
  - H1: Prometheus tanılama plugin'i
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/diffs-language-pack.md

- Rota: /plugins/reference/diffs-language-pack
- Başlıklar:
  - H1: Diffs Dil Paketi plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: Eklenen diller

## plugins/reference/diffs.md

- Rota: /plugins/reference/diffs
- Başlıklar:
  - H1: Diffs plugin'i
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/discord.md

- Rota: /plugins/reference/discord
- Başlıklar:
  - H1: Discord plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/document-extract.md

- Rota: /plugins/reference/document-extract
- Başlıklar:
  - H1: Belge Ayıklama plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/duckduckgo.md

- Rota: /plugins/reference/duckduckgo
- Başlıklar:
  - H1: DuckDuckGo plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/elevenlabs.md

- Rota: /plugins/reference/elevenlabs
- Başlıklar:
  - H1: Elevenlabs plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/exa.md

- Rota: /plugins/reference/exa
- Başlıklar:
  - H1: Exa plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/fal.md

- Rota: /plugins/reference/fal
- Başlıklar:
  - H1: fal plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/featherless.md

- Rota: /plugins/reference/featherless
- Başlıklar:
  - H1: Featherless plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/feishu.md

- Rota: /plugins/reference/feishu
- Başlıklar:
  - H1: Feishu plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/file-transfer.md

- Rota: /plugins/reference/file-transfer
- Başlıklar:
  - H1: Dosya Aktarımı plugin'i
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/firecrawl.md

- Rota: /plugins/reference/firecrawl
- Başlıklar:
  - H1: Firecrawl plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/fireworks.md

- Rota: /plugins/reference/fireworks
- Başlıklar:
  - H1: Fireworks plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/github-copilot.md

- Rota: /plugins/reference/github-copilot
- Başlıklar:
  - H1: GitHub Copilot plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/gmi.md

- Rota: /plugins/reference/gmi
- Başlıklar:
  - H1: Gmi plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/google-meet.md

- Rota: /plugins/reference/google-meet
- Başlıklar:
  - H1: Google Meet plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/google.md

- Rota: /plugins/reference/google
- Başlıklar:
  - H1: Google plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/googlechat.md

- Rota: /plugins/reference/googlechat
- Başlıklar:
  - H1: Google Chat plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/gradium.md

- Rota: /plugins/reference/gradium
- Başlıklar:
  - H1: Gradium plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/groq.md

- Rota: /plugins/reference/groq
- Başlıklar:
  - H1: Groq plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/huggingface.md

- Rota: /plugins/reference/huggingface
- Başlıklar:
  - H1: Hugging Face plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/imessage.md

- Rota: /plugins/reference/imessage
- Başlıklar:
  - H1: iMessage plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/inworld.md

- Rota: /plugins/reference/inworld
- Başlıklar:
  - H1: Inworld plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/irc.md

- Rota: /plugins/reference/irc
- Başlıklar:
  - H1: IRC plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/kilocode.md

- Rota: /plugins/reference/kilocode
- Başlıklar:
  - H1: Kilocode plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/kimi.md

- Rota: /plugins/reference/kimi
- Başlıklar:
  - H1: Kimi plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/line.md

- Rota: /plugins/reference/line
- Başlıklar:
  - H1: LINE plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/linux-canvas.md

- Rota: /plugins/reference/linux-canvas
- Başlıklar:
  - H1: Linux Canvas plugin'i
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/linux-node.md

- Rota: /plugins/reference/linux-node
- Başlıklar:
  - H1: Linux Node Plugin
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/litellm.md

- Rota: /plugins/reference/litellm
- Başlıklar:
  - H1: LiteLLM Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/llama-cpp.md

- Rota: /plugins/reference/llama-cpp
- Başlıklar:
  - H1: Llama Cpp Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: Varsayılan metin modeli
  - H2: İlgili belgeler

## plugins/reference/llm-task.md

- Rota: /plugins/reference/llm-task
- Başlıklar:
  - H1: LLM Task Plugin
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/lmstudio.md

- Rota: /plugins/reference/lmstudio
- Başlıklar:
  - H1: LM Studio Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/lobster.md

- Rota: /plugins/reference/lobster
- Başlıklar:
  - H1: Lobster Plugin
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/logbook.md

- Rota: /plugins/reference/logbook
- Başlıklar:
  - H1: Logbook Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/longcat.md

- Rota: /plugins/reference/longcat
- Başlıklar:
  - H1: LongCat Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/matrix.md

- Rota: /plugins/reference/matrix
- Başlıklar:
  - H1: Matrix Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/mattermost.md

- Rota: /plugins/reference/mattermost
- Başlıklar:
  - H1: Mattermost Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/memory-core.md

- Rota: /plugins/reference/memory-core
- Başlıklar:
  - H1: Memory Core Plugin
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/memory-lancedb.md

- Rota: /plugins/reference/memory-lancedb
- Başlıklar:
  - H1: Memory Lancedb Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/memory-wiki.md

- Rota: /plugins/reference/memory-wiki
- Başlıklar:
  - H1: Memory Wiki Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/meta.md

- Rota: /plugins/reference/meta
- Başlıklar:
  - H1: Meta Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/microsoft-foundry.md

- Rota: /plugins/reference/microsoft-foundry
- Başlıklar:
  - H1: Microsoft Foundry Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: Gereksinimler
  - H2: Sohbet modelleri
  - H2: MAI görüntü oluşturma
  - H2: Sorun giderme

## plugins/reference/microsoft.md

- Rota: /plugins/reference/microsoft
- Başlıklar:
  - H1: Microsoft Plugin
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/migrate-claude.md

- Rota: /plugins/reference/migrate-claude
- Başlıklar:
  - H1: Claude Geçiş Plugin'i
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/migrate-hermes.md

- Rota: /plugins/reference/migrate-hermes
- Başlıklar:
  - H1: Hermes Geçiş Plugin'i
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/minimax.md

- Rota: /plugins/reference/minimax
- Başlıklar:
  - H1: MiniMax Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/mistral.md

- Rota: /plugins/reference/mistral
- Başlıklar:
  - H1: Mistral Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/moonshot.md

- Rota: /plugins/reference/moonshot
- Başlıklar:
  - H1: Moonshot Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/msteams.md

- Rota: /plugins/reference/msteams
- Başlıklar:
  - H1: Microsoft Teams Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/mxc.md

- Rota: /plugins/reference/mxc
- Başlıklar:
  - H1: Mxc Plugin
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/nextcloud-talk.md

- Rota: /plugins/reference/nextcloud-talk
- Başlıklar:
  - H1: Nextcloud Talk Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/nostr.md

- Rota: /plugins/reference/nostr
- Başlıklar:
  - H1: Nostr Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/novita.md

- Rota: /plugins/reference/novita
- Başlıklar:
  - H1: Novita Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/nvidia.md

- Rota: /plugins/reference/nvidia
- Başlıklar:
  - H1: NVIDIA Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/oc-path.md

- Rota: /plugins/reference/oc-path
- Başlıklar:
  - H1: Oc Path Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/ollama.md

- Rota: /plugins/reference/ollama
- Başlıklar:
  - H1: Ollama Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/onepassword.md

- Rota: /plugins/reference/onepassword
- Başlıklar:
  - H1: Onepassword Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/open-prose.md

- Rota: /plugins/reference/open-prose
- Başlıklar:
  - H1: Open Prose Plugin
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/openai.md

- Rota: /plugins/reference/openai
- Başlıklar:
  - H1: OpenAI Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/opencode-go.md

- Rota: /plugins/reference/opencode-go
- Başlıklar:
  - H1: OpenCode Go Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/opencode.md

- Rota: /plugins/reference/opencode
- Başlıklar:
  - H1: OpenCode Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: Yerel oturumlar
  - H2: İlgili belgeler

## plugins/reference/openrouter.md

- Rota: /plugins/reference/openrouter
- Başlıklar:
  - H1: OpenRouter Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/openshell.md

- Rota: /plugins/reference/openshell
- Başlıklar:
  - H1: Openshell Plugin
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/perplexity.md

- Rota: /plugins/reference/perplexity
- Başlıklar:
  - H1: Perplexity Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/pixverse.md

- Rota: /plugins/reference/pixverse
- Başlıklar:
  - H1: PixVerse Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/policy.md

- Rota: /plugins/reference/policy
- Başlıklar:
  - H1: İlke Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: Davranış
  - H2: İlgili belgeler

## plugins/reference/qa-channel.md

- Rota: /plugins/reference/qa-channel
- Başlıklar:
  - H1: QA Channel Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/qa-lab.md

- Rota: /plugins/reference/qa-lab
- Başlıklar:
  - H1: QA Lab Plugin
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/qianfan.md

- Rota: /plugins/reference/qianfan
- Başlıklar:
  - H1: Qianfan Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/qqbot.md

- Rota: /plugins/reference/qqbot
- Başlıklar:
  - H1: QQ Bot Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/qwen.md

- Rota: /plugins/reference/qwen
- Başlıklar:
  - H1: Qwen Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/raft.md

- Rota: /plugins/reference/raft
- Başlıklar:
  - H1: Raft Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/reef.md

- Rota: /plugins/reference/reef
- Başlıklar:
  - H1: Reef Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/runway.md

- Rota: /plugins/reference/runway
- Başlıklar:
  - H1: Runway Plugin
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/searxng.md

- Rota: /plugins/reference/searxng
- Başlıklar:
  - H1: SearXNG Plugin'i
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/senseaudio.md

- Rota: /plugins/reference/senseaudio
- Başlıklar:
  - H1: Senseaudio Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/sglang.md

- Rota: /plugins/reference/sglang
- Başlıklar:
  - H1: SGLang Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/signal.md

- Rota: /plugins/reference/signal
- Başlıklar:
  - H1: Signal Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/slack.md

- Rota: /plugins/reference/slack
- Başlıklar:
  - H1: Slack Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/sms.md

- Rota: /plugins/reference/sms
- Başlıklar:
  - H1: Sms Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/stepfun.md

- Rota: /plugins/reference/stepfun
- Başlıklar:
  - H1: StepFun Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/synology-chat.md

- Rota: /plugins/reference/synology-chat
- Başlıklar:
  - H1: Synology Chat Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/synthetic.md

- Rota: /plugins/reference/synthetic
- Başlıklar:
  - H1: Synthetic Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/tavily.md

- Rota: /plugins/reference/tavily
- Başlıklar:
  - H1: Tavily Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/teams-meetings.md

- Rota: /plugins/reference/teams-meetings
- Başlıklar:
  - H1: Microsoft Teams toplantıları Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/telegram.md

- Rota: /plugins/reference/telegram
- Başlıklar:
  - H1: Telegram Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/tencent.md

- Rota: /plugins/reference/tencent
- Başlıklar:
  - H1: Tencent Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/tlon.md

- Rota: /plugins/reference/tlon
- Başlıklar:
  - H1: Tlon Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/together.md

- Rota: /plugins/reference/together
- Başlıklar:
  - H1: Together Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/tokenjuice.md

- Rota: /plugins/reference/tokenjuice
- Başlıklar:
  - H1: Tokenjuice Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/tts-local-cli.md

- Rota: /plugins/reference/tts-local-cli
- Başlıklar:
  - H1: TTS Yerel CLI Plugin'i
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/twitch.md

- Rota: /plugins/reference/twitch
- Başlıklar:
  - H1: Twitch Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/vault.md

- Rota: /plugins/reference/vault
- Başlıklar:
  - H1: Vault Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/venice.md

- Rota: /plugins/reference/venice
- Başlıklar:
  - H1: Venice Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/vercel-ai-gateway.md

- Rota: /plugins/reference/vercel-ai-gateway
- Başlıklar:
  - H1: Vercel AI Gateway Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/vllm.md

- Rota: /plugins/reference/vllm
- Başlıklar:
  - H1: vLLM Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/voice-call.md

- Rota: /plugins/reference/voice-call
- Başlıklar:
  - H1: Sesli Arama Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/volcengine.md

- Rota: /plugins/reference/volcengine
- Başlıklar:
  - H1: Volcengine Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/voyage.md

- Rota: /plugins/reference/voyage
- Başlıklar:
  - H1: Voyage Plugin'i
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/vydra.md

- Rota: /plugins/reference/vydra
- Başlıklar:
  - H1: Vydra Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/web-readability.md

- Rota: /plugins/reference/web-readability
- Başlıklar:
  - H1: Web Okunabilirliği Plugin'i
  - H2: Dağıtım
  - H2: Yüzey

## plugins/reference/webhooks.md

- Rota: /plugins/reference/webhooks
- Başlıklar:
  - H1: Webhook'lar Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/whatsapp.md

- Rota: /plugins/reference/whatsapp
- Başlıklar:
  - H1: WhatsApp Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/workboard.md

- Rota: /plugins/reference/workboard
- Başlıklar:
  - H1: Workboard Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/xai.md

- Rota: /plugins/reference/xai
- Başlıklar:
  - H1: xAI Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/xiaomi.md

- Rota: /plugins/reference/xiaomi
- Başlıklar:
  - H1: Xiaomi Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/zai.md

- Rota: /plugins/reference/zai
- Başlıklar:
  - H1: Z.AI Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/zalo.md

- Rota: /plugins/reference/zalo
- Başlıklar:
  - H1: Zalo Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/zalouser.md

- Rota: /plugins/reference/zalouser
- Başlıklar:
  - H1: Zalo Kişisel Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/reference/zoom-meetings.md

- Rota: /plugins/reference/zoom-meetings
- Başlıklar:
  - H1: Zoom toplantıları Plugin'i
  - H2: Dağıtım
  - H2: Yüzey
  - H2: İlgili belgeler

## plugins/sdk-agent-harness.md

- Rota: /plugins/sdk-agent-harness
- Başlıklar:
  - H2: Bir çalıştırma düzeneğinin kullanılacağı durumlar
  - H2: Çekirdeğin sahipliğini sürdürdüğü öğeler
  - H3: Çalıştırma düzeneğinin sahip olduğu kimlik doğrulama önyüklemesi
  - H3: Doğrulanmış kurulum çalışma zamanı yapıtları
  - H3: İstek taşıma sözleşmesi
  - H2: Çalıştırma düzeneği kaydetme
  - H3: Devredilmiş yürütme
  - H2: Seçim ilkesi
  - H2: Sağlayıcı ile çalıştırma düzeneği eşleştirmesi
  - H3: Araç sonucu ara yazılımı
  - H3: Terminal sonucu sınıflandırması
  - H3: Ajan sonu yan etkileri
  - H3: Kullanıcı girdisi ve araç yüzeyleri
  - H3: Yerel Codex çalıştırma düzeneği modu
  - H2: Çalışma zamanı katılığı
  - H2: Yerel oturumlar ve transkript yansısı
  - H2: Araç ve medya sonuçları
  - H3: Terminal aracı sonuçları
  - H3: Sonuçlanmış araç sonlandırması
  - H2: Mevcut sınırlamalar
  - H2: İlgili içerikler

## plugins/sdk-channel-inbound.md

- Rota: /plugins/sdk-channel-inbound
- Başlıklar:
  - H2: Çekirdek yardımcıları
  - H2: Teslimat sonuçlandırma sözleşmesi
  - H2: Geçiş

## plugins/sdk-channel-ingress.md

- Rota: /plugins/sdk-channel-ingress
- Başlıklar:
  - H2: Çalışma zamanı çözümleyicisi
  - H2: Sonuç
  - H2: Erişim grupları
  - H2: Olay modları
  - H2: Rotalar ve etkinleştirme
  - H2: Sansürleme
  - H2: Doğrulama

## plugins/sdk-channel-message.md

- Rota: /plugins/sdk-channel-message
- Başlıklar: yok

## plugins/sdk-channel-outbound.md

- Rota: /plugins/sdk-channel-outbound
- Başlıklar:
  - H2: Kalıcı giriş izleyicileri
  - H2: Bağdaştırıcı
  - H2: Giden ileti yankısının engellenmesi
  - H2: Düz metin temizleme
  - H2: Teslimat Kanıtı
  - H2: Mevcut giden ileti bağdaştırıcıları
  - H2: Kalıcı gönderimler
  - H2: Ertelenmiş teslimat kabulü
  - H2: Uyumluluk yönlendirmesi

## plugins/sdk-channel-plugins.md

- Rota: /plugins/sdk-channel-plugins
- Başlıklar:
  - H2: Plugininizin sahip olduğu alanlar
  - H2: Mesaj bağdaştırıcısı
  - H3: Gelen veri girişi (deneysel)
  - H3: Kalıcı veri girişi ve yeniden oynatma yinelenenlerini kaldırma
  - H4: Aktarım sınıfları ve saklama
  - H4: En az bir kez gerçekleşen yan etkiler
  - H4: Hesap kapsamlı yeniden başlatma sözleşmesi
  - H3: Yazıyor göstergeleri
  - H3: Medya kaynağı parametreleri
  - H3: Yerel yük biçimlendirme
  - H3: Oturum konuşması dil bilgisi
  - H3: Hesap kapsamlı konuşma bağlama desteği
  - H2: Onaylar ve kanal yetenekleri
  - H3: Onay kimlik doğrulaması
  - H3: Yük yaşam döngüsü ve kurulum rehberliği
  - H3: Yerel onay teslimi
  - H3: Daha dar onay çalışma zamanı alt yolları
  - H3: Kurulum alt yolları
  - H3: Diğer dar kanal alt yolları
  - H2: Gelen bahsetme politikası
  - H2: Adım adım açıklama
  - H2: Dosya yapısı
  - H2: İleri düzey konular
  - H2: Sonraki adımlar
  - H2: İlgili

## plugins/sdk-channel-turn.md

- Rota: /plugins/sdk-channel-turn
- Başlıklar: yok

## plugins/sdk-entrypoints.md

- Rota: /plugins/sdk-entrypoints
- Başlıklar:
  - H2: Paket girişleri
  - H2: defineToolPlugin
  - H2: definePluginEntry
  - H2: defineChannelPluginEntry
  - H2: defineSetupPluginEntry
  - H2: Kayıt modu
  - H2: Plugin biçimleri
  - H2: İlgili

## plugins/sdk-migration.md

- Rota: /plugins/sdk-migration
- Başlıklar:
  - H2: Neler değişti
  - H3: Neden
  - H2: Uyumluluk politikası
  - H3: Yayımlanmış kanal kurulumu uyumluluğu
  - H3: Kanal kurulumu girdi alanı uyumluluğu
  - H4: Okuyucuları doğrulama
  - H3: Eski medya projeksiyonu
  - H2: Geçiş nasıl yapılır
  - H2: İçe aktarma yolu referansı
  - H2: Kaldırılan uyumluluk yüzeyleri
  - H3: İşlem geneli API sağlayıcısı yayımlama
  - H3: Özel test dışa aktarım modülü
  - H2: Geçiş referansı
  - H2: Konuşma ve gerçek zamanlı ses geçişi
  - H2: Kaldırma zaman çizelgesi
  - H2: Uyarıları geçici olarak bastırma
  - H2: İlgili

## plugins/sdk-overview.md

- Rota: /plugins/sdk-overview
- Başlıklar:
  - H2: İçe aktarma kuralı
  - H2: Alt yol referansı
  - H2: Kayıt API'si
  - H3: Yetenek kaydı
  - H3: Araçlar ve komutlar
  - H3: Altyapı
  - H4: Onay sonrası Webhook çalışması
  - H4: İstekte bulunan kapsamlı MCP bağlantıları
  - H3: İş akışı pluginleri için ana makine kancaları
  - H3: Gateway keşif kaydı
  - H3: CLI kayıt meta verileri
  - H3: CLI arka uç kaydı
  - H3: Özel yuvalar
  - H3: Kullanımdan kaldırılan bellek gömme bağdaştırıcıları
  - H3: Olaylar ve yaşam döngüsü
  - H3: Kanca karar semantiği
  - H3: API nesnesi alanları
  - H2: Dahili modül kuralı
  - H2: İlgili

## plugins/sdk-provider-plugins.md

- Rota: /plugins/sdk-provider-plugins
- Başlıklar:
  - H2: Adım adım açıklama
  - H2: ClawHub'da yayımlama
  - H2: Dosya yapısı
  - H2: Katalog sırası referansı
  - H2: Sonraki adımlar
  - H2: İlgili

## plugins/sdk-runtime.md

- Rota: /plugins/sdk-runtime
- Başlıklar:
  - H2: Yapılandırma yükleme ve yazma
  - H2: Yeniden kullanılabilir çalışma zamanı yardımcı programları
  - H2: Çalışma zamanı ad alanları
  - H2: Çalışma zamanı referanslarını saklama
  - H2: Diğer üst düzey API alanları
  - H2: İlgili

## plugins/sdk-setup.md

- Rota: /plugins/sdk-setup
- Başlıklar:
  - H2: Paket meta verileri
  - H3: openclaw alanları
  - H3: openclaw.channel
  - H3: Kanalın sahip olduğu kurulum alanları
  - H3: openclaw.install
  - H3: Ertelenmiş tam yükleme
  - H2: Plugin manifestosu
  - H2: ClawHub'da yayımlama
  - H2: Kurulum girişi
  - H3: Dar kurulum yardımcısı içe aktarmaları
  - H3: Kanalın sahip olduğu kurulum girdi alanları
  - H3: Kanalın sahip olduğu tek hesap yükseltmesi
  - H2: Yapılandırma şeması
  - H3: Kanal yapılandırma şemaları oluşturma
  - H2: Kurulum sihirbazları
  - H2: Yayımlama ve yükleme
  - H2: İlgili

## plugins/sdk-subpaths.md

- Rota: /plugins/sdk-subpaths
- Başlıklar:
  - H2: Plugin girişi
  - H3: Uyumluluk ve özel yerel yardımcılar
  - H3: Paketle birlikte gelen plugin yardımcısı alt yolları
  - H2: İlgili

## plugins/sdk-testing.md

- Rota: /plugins/sdk-testing
- Başlıklar:
  - H2: Test yardımcı programları
  - H3: Kullanılabilir dışa aktarımlar
  - H3: Türler
  - H2: Hedef çözümlemeyi test etme
  - H2: Test kalıpları
  - H3: Kayıt sözleşmelerini test etme
  - H3: Çalışma zamanı yapılandırma erişimini test etme
  - H3: Bir kanal pluginini birim testiyle sınama
  - H3: Bir sağlayıcı pluginini birim testiyle sınama
  - H3: Plugin çalışma zamanının taklidini oluşturma
  - H3: Örnek başına taklitlerle test etme
  - H2: Sözleşme testleri (depo içi pluginler)
  - H3: Kapsamlı testleri çalıştırma
  - H2: Lint zorlaması (depo içi pluginler)
  - H2: Test yapılandırması
  - H2: İlgili

## plugins/teams-meetings.md

- Rota: /plugins/teams-meetings
- Başlıklar:
  - H2: Kurulum
  - H2: Modlar
  - H2: Konuk katılımı sınırları
  - H2: Araç ve Gateway yüzeyi
  - H2: İlgili

## plugins/tool-plugins.md

- Rota: /plugins/tool-plugins
- Başlıklar:
  - H2: Gereksinimler
  - H2: Hızlı başlangıç
  - H2: Araç yazma
  - H2: İsteğe bağlı araçlar ve fabrika araçları
  - H2: Dönüş değerleri
  - H2: Çıktı sözleşmeleri
  - H2: Yapılandırma
  - H2: Oluşturulan meta veriler
  - H2: Paket meta verileri
  - H2: CI'da doğrulama
  - H2: Yerel olarak yükleme ve inceleme
  - H2: Yayımlama
  - H2: Sorun giderme
  - H3: plugin girişi bulunamadı: ./dist/index.js
  - H3: plugin girişi defineToolPlugin meta verilerini sunmuyor
  - H3: openclaw.plugin.json tarafından oluşturulan meta veriler güncel değil
  - H3: package.json openclaw.extensions, ./dist/index.js dosyasını içermelidir
  - H3: 'typebox' paketi bulunamıyor
  - H3: Araç, yüklemeden sonra görünmüyor
  - H2: Ayrıca bkz.

## plugins/vault.md

- Rota: /plugins/vault
- Başlıklar:
  - H1: Vault SecretRef'leri
  - H2: Başlamadan önce
  - H2: Vault'ta bir sağlayıcı anahtarı saklama
  - H2: Vault'u Gateway için görünür kılma
  - H2: Bir SecretRef planı oluşturma ve uygulama
  - H2: Daha fazla sağlayıcı anahtarı yapılandırma
  - H2: SecretRef kimliği biçimi
  - H2: OpenClaw'un sakladıkları
  - H2: Kapsayıcılar ve yönetilen dağıtımlar
  - H2: İlgili

## plugins/voice-call.md

- Rota: /plugins/voice-call
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Yapılandırma
  - H3: Yapılandırma referansı
  - H2: Oturum kapsamı
  - H2: Gerçek zamanlı sesli konuşmalar
  - H3: Araç politikası
  - H3: Aracı ses bağlamı
  - H3: Gerçek zamanlı sağlayıcı örnekleri
  - H2: Akışlı döküm
  - H3: Akış sağlayıcısı örnekleri
  - H2: Aramalar için TTS
  - H3: TTS örnekleri
  - H2: Gelen aramalar
  - H3: Numara başına yönlendirme
  - H3: Sesli çıktı sözleşmesi
  - H3: Konuşma başlatma davranışı
  - H3: Twilio akış bağlantısı kesilme ek süresi
  - H2: Eski çağrı temizleyicisi
  - H2: Webhook güvenliği
  - H2: CLI
  - H2: Aracı aracı
  - H2: Gateway RPC
  - H2: Sorun giderme
  - H3: Kurulumda Webhook erişime açma işlemi başarısız oluyor
  - H3: Sağlayıcı kimlik bilgileri başarısız oluyor
  - H3: Aramalar başlıyor ancak sağlayıcı Webhook'ları ulaşmıyor
  - H3: İmza doğrulaması başarısız oluyor
  - H3: Google Meet Twilio katılımları başarısız oluyor
  - H3: Gerçek zamanlı aramada konuşma sesi yok
  - H2: İlgili

## plugins/webhooks.md

- Rota: /plugins/webhooks
- Başlıklar:
  - H2: Rotaları yapılandırma
  - H2: Güvenlik modeli
  - H2: İstek biçimi
  - H2: Desteklenen eylemler
  - H3: `create_flow`
  - H3: `run_task`
  - H2: Yanıt biçimi
  - H2: İlgili

## plugins/workboard.md

- Rota: /plugins/workboard
- Başlıklar:
  - H2: Etkinleştirme
  - H2: Yapılandırma
  - H2: Kart alanları
  - H2: Bir karttan çalışmaya başlama
  - H2: Aracı araçları
  - H2: Gönderim
  - H3: Çalışan seçimi
  - H3: Giriş noktaları
  - H2: CLI ve eğik çizgi komutu
  - H2: Oturum yaşam döngüsü eşitlemesi
  - H2: Pano iş akışı
  - H3: Oturum panosu bileşenleri
  - H2: Tanılama
  - H2: İzinler
  - H2: Depolama
  - H2: Sorun giderme
  - H2: İlgili

## plugins/zalouser.md

- Rota: /plugins/zalouser
- Başlıklar:
  - H2: Adlandırma
  - H2: Çalıştığı yer
  - H2: Yükleme
  - H3: npm'den
  - H3: Yerel bir klasörden (geliştirme)
  - H2: Yapılandırma
  - H2: CLI
  - H2: Aracı aracı
  - H2: İlgili

## plugins/zoom-meetings.md

- Rota: /plugins/zoom-meetings
- Başlıklar:
  - H2: Kurulum
  - H2: Modlar
  - H2: Konuk katılımı sınırları
  - H2: Araç ve Gateway yüzeyi
  - H2: İlgili

## prose.md

- Rota: /prose
- Başlıklar:
  - H2: Yükleme
  - H2: Eğik çizgi komutu
  - H2: Yapabilecekleri
  - H2: Örnek: paralel araştırma ve sentez
  - H2: OpenClaw çalışma zamanı eşlemesi
  - H2: Dosya konumları
  - H2: Durum arka uçları
  - H2: Güvenlik
  - H2: İlgili

## providers/alibaba.md

- Rota: /providers/alibaba
- Başlıklar:
  - H2: Başlarken
  - H2: Yerleşik Wan modelleri
  - H2: Yetenekler ve sınırlar
  - H2: Gelişmiş yapılandırma
  - H2: İlgili konular

## providers/anthropic.md

- Rota: /providers/anthropic
- Başlıklar:
  - H2: Kullanım ve maliyet takibi
  - H2: Başlarken
  - H2: Bilgisayarlar arasında Claude oturumları
  - H2: Düşünme varsayılanları (Claude Opus 5, Sonnet 5, Mythos 5, Fable 5, 4.8 ve 4.6)
  - H2: Güvenlik reddi için geri dönüş (Claude Fable 5)
  - H3: Bunun var olma nedeni
  - H3: Nasıl çalışır
  - H3: Gözlemlenebilirlik ve faturalandırma
  - H3: Kapsam
  - H2: İstem önbelleğe alma
  - H2: Gelişmiş yapılandırma
  - H2: Sorun giderme
  - H2: İlgili konular

## providers/arcee.md

- Rota: /providers/arcee
- Başlıklar:
  - H2: Plugin'i yükleme
  - H2: Başlarken
  - H2: Etkileşimsiz kurulum
  - H2: Doğrudan Arcee kataloğu
  - H2: OpenRouter kataloğu
  - H2: Desteklenen özellikler
  - H2: İlgili konular

## providers/azure-speech.md

- Rota: /providers/azure-speech
- Başlıklar:
  - H2: Başlarken
  - H2: Yapılandırma seçenekleri
  - H2: Notlar
  - H2: İlgili konular

## providers/baseten.md

- Rota: /providers/baseten
- Başlıklar:
  - H2: Plugin'i yükleme
  - H2: Başlarken
  - H2: Inkling
  - H2: Birlikte sunulan geri dönüş kataloğu
  - H2: Manuel yapılandırma
  - H2: İlgili konular

## providers/bedrock-mantle.md

- Rota: /providers/bedrock-mantle
- Başlıklar:
  - H2: Başlarken
  - H2: Otomatik model keşfi
  - H3: Desteklenen bölgeler
  - H2: Manuel yapılandırma
  - H2: Gelişmiş yapılandırma
  - H2: İlgili konular

## providers/bedrock.md

- Rota: /providers/bedrock
- Başlıklar:
  - H2: Başlarken
  - H2: Otomatik model keşfi
  - H2: Hızlı kurulum (AWS yolu)
  - H2: Gelişmiş yapılandırma
  - H2: İlgili konular

## providers/cerebras.md

- Rota: /providers/cerebras
- Başlıklar:
  - H2: Plugin'i yükleme
  - H2: Başlarken
  - H2: Etkileşimsiz kurulum
  - H2: Yerleşik katalog
  - H2: Manuel yapılandırma
  - H2: İlgili konular

## providers/chutes.md

- Rota: /providers/chutes
- Başlıklar:
  - H2: Plugin'i yükleme
  - H2: Başlarken
  - H2: Keşif davranışı
  - H2: Varsayılan diğer adlar
  - H2: Yerleşik başlangıç kataloğu
  - H2: Yapılandırma örneği
  - H2: İlgili konular

## providers/claude-max-api-proxy.md

- Rota: /providers/claude-max-api-proxy
- Başlıklar:
  - H2: Bunu kullanma nedenleri
  - H2: Nasıl çalışır
  - H2: Başlarken
  - H2: Gelişmiş yapılandırma
  - H2: Notlar
  - H2: İlgili konular

## providers/clawrouter.md

- Rota: /providers/clawrouter
- Başlıklar:
  - H2: Başlarken
  - H2: Yönetilen etkileşimsiz dağıtım
  - H2: Hazır olma durumu ve canlı kanıt
  - H2: Model keşfi
  - H2: Protokol ve sağlayıcı Plugin'leri
  - H2: Kotalar ve kullanım
  - H2: Sorun giderme
  - H2: Güvenlik davranışı
  - H2: İlgili konular

## providers/cloudflare-ai-gateway.md

- Rota: /providers/cloudflare-ai-gateway
- Başlıklar:
  - H2: Plugin'i yükleme
  - H2: Başlarken
  - H2: Etkileşimsiz örnek
  - H2: Gelişmiş yapılandırma
  - H2: İlgili konular

## providers/cohere.md

- Rota: /providers/cohere
- Başlıklar:
  - H2: Yerleşik katalog
  - H2: Başlarken
  - H2: Yalnızca ortam üzerinden kurulum
  - H2: İlgili konular

## providers/comfy.md

- Rota: /providers/comfy
- Başlıklar:
  - H2: Desteklenenler
  - H2: Başlarken
  - H2: Yapılandırma
  - H3: Paylaşılan anahtarlar
  - H3: Yetenek başına anahtarlar
  - H2: İş akışı ayrıntıları
  - H2: İlgili konular

## providers/deepgram.md

- Rota: /providers/deepgram
- Başlıklar:
  - H2: Başlarken
  - H2: Yapılandırma seçenekleri
  - H2: Sesli Arama akışlı STT
  - H2: Notlar
  - H2: İlgili konular

## providers/deepinfra.md

- Rota: /providers/deepinfra
- Başlıklar:
  - H2: Plugin'i yükleme
  - H2: API anahtarı alma
  - H2: CLI kurulumu
  - H2: Yapılandırma kod parçacığı
  - H2: Desteklenen yüzeyler
  - H2: Kullanılabilir modeller
  - H2: Notlar
  - H2: İlgili konular

## providers/deepseek.md

- Rota: /providers/deepseek
- Başlıklar:
  - H2: Plugin'i yükleme
  - H2: Başlarken
  - H2: Yerleşik katalog
  - H2: Düşünme ve araçlar
  - H2: Canlı test
  - H2: Yapılandırma örneği
  - H2: İlgili konular

## providers/ds4.md

- Rota: /providers/ds4
- Başlıklar:
  - H2: Gereksinimler
  - H2: Hızlı başlangıç
  - H2: Tam yapılandırma
  - H2: İsteğe bağlı başlatma
  - H2: Think Max
  - H2: Test
  - H2: Sorun giderme
  - H2: İlgili konular

## providers/elevenlabs.md

- Rota: /providers/elevenlabs
- Başlıklar:
  - H2: Kimlik doğrulama
  - H2: Metinden konuşmaya
  - H2: Konuşmadan metne
  - H2: Akışlı STT
  - H2: İlgili konular

## providers/fal.md

- Rota: /providers/fal
- Başlıklar:
  - H2: Başlarken
  - H2: Görüntü oluşturma
  - H2: Video oluşturma
  - H2: Müzik oluşturma
  - H2: İlgili konular

## providers/featherless.md

- Rota: /providers/featherless
- Başlıklar:
  - H2: Kurulum
  - H2: Varsayılan model
  - H2: Diğer Featherless modelleri
  - H2: Sorun giderme
  - H2: İlgili konular

## providers/fireworks.md

- Rota: /providers/fireworks
- Başlıklar:
  - H2: Başlarken
  - H2: Etkileşimsiz kurulum
  - H2: Yerleşik katalog
  - H2: Özel Fireworks model kimlikleri
  - H2: İlgili konular

## providers/github-copilot.md

- Rota: /providers/github-copilot
- Başlıklar:
  - H2: OpenClaw'da Copilot'u kullanmanın üç yolu
  - H2: GitHub Enterprise (veri yerleşimi)
  - H2: İsteğe bağlı bayraklar
  - H2: Etkileşimsiz ilk katılım
  - H2: Bellek arama gömmeleri
  - H3: Yapılandırma
  - H3: Nasıl çalışır
  - H2: İlgili konular

## providers/gmi.md

- Rota: /providers/gmi
- Başlıklar:
  - H2: Kurulum
  - H2: GMI ne zaman seçilmeli
  - H2: Modeller
  - H2: Sorun giderme
  - H2: İlgili konular

## providers/google.md

- Rota: /providers/google
- Başlıklar:
  - H2: Başlarken
  - H2: Yetenekler
  - H2: Web araması
  - H2: Görüntü oluşturma
  - H2: Video oluşturma
  - H2: Müzik oluşturma
  - H2: Metinden konuşmaya
  - H2: Gerçek zamanlı ses
  - H2: Gelişmiş yapılandırma
  - H2: İlgili konular

## providers/gradium.md

- Rota: /providers/gradium
- Başlıklar:
  - H2: Plugin'i yükleme
  - H2: Kurulum
  - H2: Yapılandırma
  - H2: Sesler
  - H3: Mesaj başına ses geçersiz kılma
  - H2: Çıktı
  - H2: Otomatik seçim sırası
  - H2: İlgili konular

## providers/groq.md

- Rota: /providers/groq
- Başlıklar:
  - H2: Plugin'i yükleme
  - H2: Başlarken
  - H3: Yapılandırma dosyası örneği
  - H2: Yerleşik katalog
  - H2: Akıl yürütme modelleri
  - H2: Ses dökümü
  - H2: İlgili konular

## providers/huggingface.md

- Rota: /providers/huggingface
- Başlıklar:
  - H2: Başlarken
  - H3: Etkileşimsiz kurulum
  - H2: Model kimlikleri
  - H2: Gelişmiş yapılandırma
  - H2: İlgili konular

## providers/index.md

- Rota: /providers
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Sağlayıcı belgeleri
  - H2: Paylaşılan genel bakış sayfaları
  - H2: Transkripsiyon sağlayıcıları
  - H2: Topluluk araçları

## providers/inferrs.md

- Rota: /providers/inferrs
- Başlıklar:
  - H2: Başlarken
  - H2: Tam yapılandırma örneği
  - H2: İsteğe bağlı başlatma
  - H2: Gelişmiş yapılandırma
  - H2: Sorun giderme
  - H2: İlgili konular

## providers/inworld.md

- Rota: /providers/inworld
- Başlıklar:
  - H2: Plugin'i yükleme
  - H2: Başlarken
  - H2: Yapılandırma seçenekleri
  - H2: Notlar
  - H2: İlgili konular

## providers/kilocode.md

- Rota: /providers/kilocode
- Başlıklar:
  - H2: Plugin'i yükleme
  - H2: Kurulum
  - H2: Varsayılan model ve katalog
  - H2: Yapılandırma örneği
  - H2: Davranış notları
  - H2: İlgili konular

## providers/litellm.md

- Rota: /providers/litellm
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Yapılandırma
  - H2: Görüntü oluşturma
  - H2: Gelişmiş
  - H2: İlgili konular

## providers/lmstudio.md

- Rota: /providers/lmstudio
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Etkileşimsiz ilk katılım
  - H2: Yapılandırma
  - H3: Akışlı kullanım uyumluluğu
  - H3: Düşünme uyumluluğu
  - H3: Açık yapılandırma
  - H3: Ön yüklemeyi devre dışı bırakma
  - H3: LAN veya tailnet ana makinesi
  - H2: Sorun giderme
  - H3: LM Studio algılanmadı
  - H3: Kimlik doğrulama hataları (HTTP 401)
  - H2: İlgili konular

## providers/longcat.md

- Rota: /providers/longcat
- Başlıklar:
  - H2: Plugin'i yükleme
  - H2: Başlarken
  - H3: Etkileşimsiz kurulum
  - H2: Akıl yürütme davranışı
  - H2: Fiyatlandırma
  - H2: Kendi sunucunuzda barındırılan LongCat-2.0
  - H2: Sorun giderme
  - H2: İlgili içerikler

## providers/meta.md

- Rota: /providers/meta
- Başlıklar:
  - H2: Başlarken
  - H2: Etkileşimsiz kurulum
  - H2: Yerleşik katalog
  - H2: Manuel yapılandırma
  - H2: Hızlı doğrulama testi
  - H2: İlgili içerikler

## providers/minimax.md

- Rota: /providers/minimax
- Başlıklar:
  - H2: Yerleşik katalog
  - H2: Başlarken
  - H2: openclaw configure ile yapılandırma
  - H2: Yetenekler
  - H3: Görsel oluşturma
  - H3: Metinden konuşmaya
  - H3: Müzik oluşturma
  - H3: Video oluşturma
  - H3: Görsel anlama
  - H3: Web araması
  - H2: Gelişmiş yapılandırma
  - H2: Notlar
  - H2: Sorun giderme
  - H2: İlgili içerikler

## providers/mistral.md

- Rota: /providers/mistral
- Başlıklar:
  - H2: Başlarken
  - H2: Yerleşik LLM kataloğu
  - H2: Ses transkripsiyonu (Voxtral)
  - H2: Voice Call akışlı konuşmadan metne dönüştürme
  - H2: Gelişmiş yapılandırma
  - H2: İlgili içerikler

## providers/models.md

- Rota: /providers/models
- Başlıklar:
  - H2: Hızlı başlangıç (iki adım)
  - H2: Desteklenen sağlayıcılar (başlangıç kümesi)
  - H2: Ek sağlayıcı varyantları
  - H2: İlgili içerikler

## providers/moonshot.md

- Rota: /providers/moonshot
- Başlıklar:
  - H2: Yerleşik model kataloğu
  - H2: Başlarken
  - H2: Kimi web araması
  - H2: Gelişmiş yapılandırma
  - H2: İlgili içerikler

## providers/novita.md

- Rota: /providers/novita
- Başlıklar:
  - H2: Kurulum
  - H2: Varsayılanlar
  - H2: Birlikte sunulan model kataloğu
  - H2: Novita ne zaman tercih edilmeli?
  - H2: Sorun giderme
  - H2: İlgili içerikler

## providers/nvidia.md

- Rota: /providers/nvidia
- Başlıklar:
  - H2: Başlarken
  - H2: Yapılandırma örneği
  - H2: Öne çıkan katalog
  - H2: Nemotron 3 Ultra
  - H2: Birlikte sunulan yedek katalog
  - H2: Gelişmiş yapılandırma
  - H2: İlgili içerikler

## providers/ollama-cloud.md

- Rota: /providers/ollama-cloud
- Başlıklar:
  - H2: Kurulum
  - H2: Varsayılanlar
  - H2: Ollama Cloud ne zaman tercih edilmeli?
  - H2: Modeller
  - H2: Canlı test
  - H2: Sorun giderme
  - H2: İlgili içerikler

## providers/ollama.md

- Rota: /providers/ollama
- Başlıklar:
  - H2: Kimlik doğrulama kuralları
  - H2: Başlarken
  - H2: Yerel bir ana makine üzerinden bulut modelleri
  - H2: Model keşfi (örtük sağlayıcı)
  - H3: Hızlı doğrulama testleri
  - H2: Node üzerinde yerel çıkarım
  - H2: Görsel işleme ve görsel açıklaması
  - H2: Yapılandırma
  - H2: Yaygın tarifler
  - H3: Model seçimi
  - H3: Hızlı doğrulama
  - H2: Ollama Web Araması
  - H2: Gelişmiş yapılandırma
  - H2: Sorun giderme
  - H2: İlgili içerikler

## providers/openai.md

- Rota: /providers/openai
- Başlıklar:
  - H2: Kullanım ve maliyet takibi
  - H2: Hızlı seçim
  - H2: Adlandırma eşlemesi
  - H2: Örtük ajan çalışma zamanı
  - H2: GPT-5.6 sınırlı önizlemesi
  - H2: OpenClaw özellik kapsamı
  - H2: Bellek gömmeleri
  - H2: Başlarken
  - H2: Yerel Codex app-server kimlik doğrulaması
  - H2: Görsel oluşturma
  - H2: Video oluşturma
  - H2: GPT-5 istem katkısı
  - H2: Ses ve konuşma
  - H2: Azure OpenAI uç noktaları
  - H3: Yapılandırma
  - H3: API sürümü
  - H3: Model adları dağıtım adlarıdır
  - H3: Bölgesel kullanılabilirlik
  - H3: Parametre farklılıkları
  - H2: Gelişmiş yapılandırma
  - H2: İlgili içerikler

## providers/opencode-go.md

- Rota: /providers/opencode-go
- Başlıklar:
  - H2: Başlarken
  - H2: Yapılandırma örneği
  - H2: Yerleşik katalog
  - H2: Gelişmiş yapılandırma
  - H2: İlgili içerikler

## providers/opencode.md

- Rota: /providers/opencode
- Başlıklar:
  - H2: Başlarken
  - H2: Yapılandırma örneği
  - H2: Yerleşik kataloglar
  - H3: Zen
  - H3: Go
  - H2: Gelişmiş yapılandırma
  - H2: İlgili içerikler

## providers/openrouter.md

- Rota: /providers/openrouter
- Başlıklar:
  - H2: Başlarken
  - H2: Yapılandırma örneği
  - H2: Model başvuruları
  - H2: Görsel oluşturma
  - H2: Video oluşturma
  - H2: Müzik oluşturma
  - H2: Metinden konuşmaya
  - H2: Konuşmadan metne (gelen ses)
  - H2: Fusion yönlendiricisi
  - H2: Kimlik doğrulama ve üstbilgiler
  - H2: Gelişmiş yapılandırma
  - H2: İlgili içerikler

## providers/perplexity-provider.md

- Rota: /providers/perplexity-provider
- Başlıklar:
  - H2: Plugin'i yükleme
  - H2: Başlarken
  - H2: Arama modları
  - H2: Yerel API filtreleme
  - H2: Gelişmiş yapılandırma
  - H2: İlgili içerikler

## providers/pixverse.md

- Rota: /providers/pixverse
- Başlıklar:
  - H2: Başlarken
  - H2: Desteklenen modlar ve modeller
  - H2: Sağlayıcı seçenekleri
  - H2: Yapılandırma
  - H2: Gelişmiş yapılandırma
  - H2: İlgili içerikler

## providers/qianfan.md

- Rota: /providers/qianfan
- Başlıklar:
  - H2: Plugin'i yükleme
  - H2: Başlarken
  - H2: Yerleşik katalog
  - H2: Yapılandırma örneği
  - H2: İlgili içerikler

## providers/qwen.md

- Rota: /providers/qwen
- Başlıklar:
  - H2: Plugin'i yükleme
  - H2: Başlarken
  - H2: Plan türleri ve uç noktalar
  - H2: Yerleşik katalog
  - H3: Token Plan kataloğu
  - H2: Düşünme denetimleri
  - H2: Çok modlu eklentiler
  - H2: Gelişmiş yapılandırma
  - H2: İlgili içerikler

## providers/runway.md

- Rota: /providers/runway
- Başlıklar:
  - H2: Başlarken
  - H2: Desteklenen modlar ve modeller
  - H2: Yapılandırma
  - H2: Gelişmiş yapılandırma
  - H2: İlgili içerikler

## providers/senseaudio.md

- Rota: /providers/senseaudio
- Başlıklar:
  - H2: Başlarken
  - H2: Seçenekler
  - H2: İlgili içerikler

## providers/sglang.md

- Rota: /providers/sglang
- Başlıklar:
  - H2: Başlarken
  - H2: Model keşfi (örtük sağlayıcı)
  - H2: Açık yapılandırma (manuel modeller)
  - H2: Gelişmiş yapılandırma
  - H2: İlgili içerikler

## providers/stepfun.md

- Rota: /providers/stepfun
- Başlıklar:
  - H2: Plugin'i yükleme
  - H2: Bölge ve uç noktalara genel bakış
  - H2: Yerleşik katalog
  - H2: Başlarken
  - H2: Gelişmiş yapılandırma
  - H2: İlgili içerikler

## providers/synthetic.md

- Rota: /providers/synthetic
- Başlıklar:
  - H2: Başlarken
  - H2: Yapılandırma örneği
  - H2: Yerleşik katalog
  - H2: İlgili içerikler

## providers/tencent.md

- Rota: /providers/tencent
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Etkileşimsiz kurulum
  - H2: Yerleşik katalog
  - H2: Gelişmiş yapılandırma
  - H2: İlgili içerikler

## providers/together.md

- Rota: /providers/together
- Başlıklar:
  - H2: Başlarken
  - H3: Etkileşimsiz örnek
  - H2: Yerleşik katalog
  - H2: Video oluşturma
  - H2: İlgili içerikler

## providers/venice.md

- Rota: /providers/venice
- Başlıklar:
  - H2: Gizlilik modları
  - H2: Başlarken
  - H2: Model seçimi
  - H2: Yerleşik katalog (30 model)
  - H2: Model keşfi
  - H2: DeepSeek V4 yeniden oynatma davranışı
  - H2: Akış ve araç desteği
  - H2: Fiyatlandırma
  - H2: Kullanım örnekleri
  - H2: Sorun giderme
  - H2: Gelişmiş yapılandırma
  - H2: İlgili içerikler

## providers/vercel-ai-gateway.md

- Rota: /providers/vercel-ai-gateway
- Başlıklar:
  - H2: Başlarken
  - H2: Etkileşimsiz örnek
  - H2: Model kimliği kısaltması
  - H2: Gelişmiş yapılandırma
  - H2: İlgili içerikler

## providers/vllm.md

- Rota: /providers/vllm
- Başlıklar:
  - H2: Başlarken
  - H2: Model keşfi (örtük sağlayıcı)
  - H2: Açık yapılandırma
  - H2: Gelişmiş yapılandırma
  - H2: Sorun giderme
  - H2: İlgili içerikler

## providers/volcengine.md

- Rota: /providers/volcengine
- Başlıklar:
  - H2: Başlarken
  - H2: Sağlayıcılar ve uç noktalar
  - H2: Yerleşik katalog
  - H2: Metinden konuşmaya
  - H2: Gelişmiş yapılandırma
  - H2: İlgili içerikler

## providers/vydra.md

- Rota: /providers/vydra
- Başlıklar:
  - H2: Kurulum
  - H2: Yetenekler
  - H2: İlgili içerikler

## providers/xai.md

- Rota: /providers/xai
- Başlıklar:
  - H2: Kurulum
  - H2: OAuth sorunlarını giderme
  - H2: Yerleşik katalog
  - H2: Özellik kapsamı
  - H3: Eski hızlı mod uyumluluğu
  - H3: Eski sürüm uyumluluğu ve değişken takma adlar
  - H2: Özellikler
  - H2: Canlı test
  - H2: İlgili içerikler

## providers/xiaomi.md

- Rota: /providers/xiaomi
- Başlıklar:
  - H2: Başlarken
  - H2: Kullandıkça öde kataloğu
  - H2: Token Planı kataloğu
  - H2: Akıl yürütme modelleri
  - H2: Metinden konuşmaya
  - H2: Yapılandırma örneği
  - H2: İlgili içerikler

## providers/zai.md

- Rota: /providers/zai
- Başlıklar:
  - H2: GLM modelleri
  - H2: Başlarken
  - H3: Uç noktalar
  - H2: Hız sınırları ve aşırı yüklenmeler
  - H2: Yapılandırma örneği
  - H2: Yerleşik katalog
  - H2: Düşünme düzeyleri
  - H2: Gelişmiş yapılandırma
  - H2: İlgili içerikler

## refactor/acp.md

- Rota: /refactor/acp
- Başlıklar:
  - H2: Hedefler
  - H2: Hedef dışı konular
  - H2: Hedef Model
  - H3: Gateway Örneği Kimliği
  - H3: ACP Oturumu Sahipliği
  - H3: ACPX İşlem Kiraları
  - H2: Yaşam Döngüsü Denetleyicisi
  - H2: Sarmalayıcı Sözleşmesi
  - H2: Oturum Görünürlüğü Sözleşmesi
  - H2: Geçiş Planı
  - H3: Aşama 1: Kimlik ve Kiralar Ekleme
  - H3: Aşama 2: Önce Kira Temizliği
  - H3: Aşama 3: Başlangıçta Önce Kiraya Göre Ayıklama
  - H3: Aşama 4: Oturum Sahipliği Satırları
  - H3: Aşama 5: Eski Sezgisel Yöntemleri Kaldırma
  - H2: Testler
  - H2: Uyumluluk Notları
  - H2: Başarı Ölçütleri

## refactor/canvas.md

- Rota: /refactor/canvas
- Başlıklar:
  - H1: Canvas Plugin yeniden düzenlemesi
  - H2: Hedef
  - H2: Hedef dışı konular
  - H2: Mevcut dal durumu
  - H2: Hedef yapı
  - H2: Geçiş adımları
  - H2: Denetim kontrol listesi
  - H2: Doğrulama komutları

## refactor/database-first.md

- Rota: /refactor/database-first
- Başlıklar:
  - H1: Önce Veritabanı Durum Yeniden Düzenlemesi
  - H2: Karar
  - H2: Kesin Sözleşme
  - H2: Hedef durum ve ilerleme
  - H3: Kesin hedef
  - H3: Hedef durumlar
  - H3: Mevcut durum
  - H3: Kalan çalışmalar
  - H3: Gerilemeye izin vermeyin
  - H2: Kod Okuma Varsayımları
  - H2: Kod Okuma Bulguları
  - H2: Mevcut Kod Yapısı
  - H2: Hedef Şema Yapısı
  - H2: Doctor Geçiş Yapısı
  - H2: Geçiş Envanteri
  - H2: Geçiş Planı
  - H3: Aşama 0: Sınırı Sabitleme
  - H3: Aşama 1: Genel Denetim Düzlemini Tamamlama
  - H3: Aşama 2: Temsilci Başına Veritabanlarını Kullanıma Alma
  - H3: Aşama 3: Oturum Deposu API'lerini Değiştirme
  - H3: Aşama 4: Dökümleri, ACP Akışlarını, Yörüngeleri ve VFS'yi Taşıma
  - H3: Aşama 5: Yedekleme, Geri Yükleme, Sıkıştırma ve Doğrulama
  - H3: Aşama 6: Çalışan Çalışma Zamanı
  - H3: Aşama 7: Eski Dünyayı Silme
  - H2: Yedekleme ve Geri Yükleme
  - H2: Çalışma Zamanı Yeniden Düzenleme Planı
  - H2: Performans Kuralları
  - H2: Statik Yasaklar
  - H2: Tamamlanma Ölçütleri

## refactor/operator-approvals.md

- Rota: /refactor/operator-approvals
- Başlıklar:
  - H1: Çok yüzeyli operatör onayları
  - H2: Hedefler
  - H2: Hedef dışı konular
  - H2: Yayına alma öncesi temel durum ve kanıt haritası
  - H2: Önceki çalışmalar
  - H2: Mimari ve sahiplik
  - H2: Kalıcı kayıt
  - H2: Durum makinesi ve karşılaştırıp ayarlama
  - H2: Gateway API
  - H2: Olaylar ve taşınabilir eylemler
  - H2: Denetim Arayüzü
  - H2: Yetkilendirme ve gizlilik
  - H2: Hedef kitle izdüşümü
  - H2: Teslim edilen yüzeylerin yakınsaması
  - H2: Yeniden başlatma, zaman aşımı ve rota semantiği
  - H2: Uyumluluk planı
  - H2: Yayına alma
  - H3: PR 1: kalıcı yaşam döngüsü
  - H3: PR 2: türü belirlenmiş eylemler ve kanal geri çağrıları
  - H3: PR 3: Denetim Arayüzü derin bağlantısı
  - H3: PR 4: yerel istemciler
  - H3: PR 5: üst öğe yaşam döngüsü yayılımı
  - H3: PR 6: kapalı durumda başarısız olma davranışı
  - H3: Takip çalışması: kalıcı uzak ileti temizliği
  - H2: Testler
  - H2: Gözlemlenebilirlik
  - H2: Açık kararlar

## reference/AGENTS.default.md

- Rota: /reference/AGENTS.default
- Başlıklar:
  - H2: İlk çalıştırma (önerilen)
  - H2: Güvenlik varsayılanları
  - H2: Mevcut çözümler ön kontrolü
  - H2: Oturum başlangıcı (gerekli)
  - H2: Ruh (gerekli)
  - H2: Paylaşılan alanlar (önerilen)
  - H2: Bellek sistemi (önerilen)
  - H2: Araçlar ve Skills
  - H2: Yedekleme ipucu (önerilen)
  - H2: OpenClaw ne yapar?
  - H2: Temel Skills (Settings → Skills bölümünde etkinleştirin)
  - H2: Kullanım notları
  - H2: İlgili içerikler

## reference/RELEASING.md

- Rota: /reference/RELEASING
- Başlıklar:
  - H2: Sürüm adlandırma
  - H2: Sürüm yayınlama sıklığı
  - H2: Aylık Gateway genişletilmiş kararlı sürüm yayını
  - H3: Adayı hazırlama ve kararlı hâle getirme
  - H3: npm paketlerini yayımlama
  - H3: Doğrulama ve kurtarma
  - H2: Normal sürüm operatörü kontrol listesi
  - H2: Kararlı ana dalı kapatma
  - H2: Sürüm ön kontrolü
  - H2: Sürüm test kutuları
  - H3: Vitest
  - H3: Docker
  - H3: QA Lab
  - H3: Paket
  - H2: Normal sürüm yayımlama otomasyonu
  - H2: NPM iş akışı girdileri
  - H2: Normal beta/en son kararlı sürüm sıralaması
  - H2: Herkese açık referanslar
  - H2: İlgili içerikler

## reference/api-usage-costs.md

- Rota: /reference/api-usage-costs
- Başlıklar:
  - H2: Maliyetlerin ortaya çıktığı yerler
  - H2: Anahtarların keşfedilme biçimi
  - H2: Anahtarları kullanarak harcama yapabilen özellikler
  - H3: Temel model yanıtları (sohbet + araçlar)
  - H3: Medya anlama (ses/görüntü/video)
  - H3: Görüntü ve video oluşturma
  - H3: Bellek gömmeleri ve anlamsal arama
  - H3: Web arama aracı
  - H3: Web getirme aracı (Firecrawl)
  - H3: Sağlayıcı kullanım anlık görüntüleri (durum/sağlık)
  - H3: Compaction koruması özetlemesi
  - H3: Model taraması / yoklaması
  - H3: Konuşma (ses)
  - H3: Skills (üçüncü taraf API'leri)
  - H2: İlgili içerikler

## reference/credits.md

- Rota: /reference/credits
- Başlıklar:
  - H2: Katkıda bulunanlar
  - H2: Temel katkıda bulunanlar
  - H2: Lisans
  - H2: İlgili içerikler

## reference/database-schemas.md

- Rota: /reference/database-schemas
- Başlıklar:
  - H2: Veritabanı yerleşimi
  - H2: Sürüm oluşturma sözleşmesi
  - H2: Temsilci şeması geçmişi
  - H2: Durum şeması geçmişi
  - H2: Bütünlük denetimleri
  - H2: Sorun giderme
  - H3: 2026.7.2 sürümüne güncelledikten sonra neden geri dönemezsiniz?
  - H3: Gateway, daha yeni şema sürümü hatasıyla başlatılmayı reddediyor
  - H3: Bütünlük doğrulaması başarısız olduktan sonra bir veritabanı karantinaya alınıyor
  - H2: Sürüm düşürme desteklenmez
  - H3: Örnek: temsilci şeması 11'den 9'a

## reference/device-models.md

- Rota: /reference/device-models
- Başlıklar:
  - H2: Veri kaynağı
  - H2: Veritabanını güncelleme
  - H2: İlgili içerikler

## reference/full-release-validation.md

- Rota: /reference/full-release-validation
- Başlıklar:
  - H2: Genişletilmiş kararlı sürüm istisnası
  - H2: Üst düzey aşamalar
  - H2: Sürüm denetimi aşamaları
  - H2: Docker sürüm yolu parçaları
  - H2: Sürüm profilleri
  - H2: Yalnızca tam doğrulamaya yönelik eklemeler
  - H2: Odaklı yeniden çalıştırmalar
  - H2: Saklanacak kanıtlar
  - H2: İş akışı dosyaları

## reference/memory-config.md

- Rota: /reference/memory-config
- Başlıklar:
  - H2: Konuşmalar arasında hatırlama
  - H2: Sağlayıcı seçimi
  - H3: Özel sağlayıcı kimlikleri
  - H3: API anahtarı çözümleme
  - H2: Uzak uç nokta yapılandırması
  - H2: Sağlayıcıya özgü yapılandırma
  - H2: Dizin oluşturma davranışı
  - H2: Karma arama yapılandırması
  - H3: Tam örnek
  - H2: Ek bellek yolları
  - H2: Çok kipli bellek (Gemini)
  - H2: Gömme önbelleği
  - H2: Toplu dizin oluşturma
  - H2: Oturum belleği araması
  - H2: SQLite vektör hızlandırma (sqlite-vec)
  - H2: Dizin depolama
  - H2: QMD arka uç yapılandırması
  - H3: Tam QMD örneği
  - H2: Dreaming
  - H3: Kullanıcı ayarları
  - H3: Örnek
  - H2: İlgili içerikler

## reference/openclaw-ai.md

- Rota: /reference/openclaw-ai
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Tasarım sözleşmesi
  - H2: Alt yol dışa aktarımları

## reference/path3-live-sqlite-e2e-harness.md

- Rota: /reference/path3-live-sqlite-e2e-harness
- Başlıklar:
  - H2: Komut yapısı
  - H2: Yalıtılmış, derlenmiş CLI kanıtı
  - H2: Ön kontrol
  - H2: Temsilci odaklı senaryo
  - H2: Adım başına doğrulamalar
  - H2: Kanıt yapıtı
  - H2: Güvenlik kuralları
  - H2: Başarılı sonuç

## reference/prompt-caching.md

- Rota: /reference/prompt-caching
- Başlıklar:
  - H2: Birincil ayarlar
  - H3: cacheRetention
  - H3: contextPruning.mode: "cache-ttl"
  - H3: Heartbeat sıcak tutma
  - H2: Sağlayıcı davranışı
  - H3: Anthropic (doğrudan API ve Vertex AI)
  - H3: OpenAI (doğrudan API)
  - H3: Amazon Bedrock
  - H3: OpenRouter
  - H3: Google Gemini (doğrudan API)
  - H3: CLI aracı sağlayıcıları (Claude Code, Gemini CLI)
  - H3: Diğer sağlayıcılar
  - H2: Sistem istemi önbellek sınırı
  - H2: OpenClaw önbellek kararlılığı korumaları
  - H2: Ayarlama kalıpları
  - H3: Karma trafik (önerilen varsayılan)
  - H3: Maliyet öncelikli temel durum
  - H2: Canlı regresyon testleri
  - H3: Anthropic canlı test beklentileri
  - H3: OpenAI canlı test beklentileri
  - H2: diagnostics.cacheTrace yapılandırması
  - H3: Ortam geçişleri (tek seferlik hata ayıklama)
  - H3: İncelenecekler
  - H2: Hızlı sorun giderme
  - H2: İlgili içerikler

## reference/pull-request-review-flow.md

- Rota: /reference/pull-request-review-flow
- Başlıklar:
  - H2: Barnacle
  - H2: ClawSweeper
  - H2: İnceleme sırasında bir PR'ı iyileştirme
  - H2: Otomasyon sessiz kaldığında
  - H2: Sorun giderme
  - H2: Otomasyonu çatallama
  - H2: İlgili

## reference/release-performance-sweep.md

- Rota: /reference/release-performance-sweep
- Başlıklar:
  - H2: Anlık görüntü
  - H2: 5.28'de neler değişti
  - H2: Öne çıkan rakamlar
  - H3: Kurulum alanı
  - H3: npm paket boyutu
  - H2: Kova ajan turu özeti
  - H2: Kaynak incelemeleri
  - H2: Kurulum alanı denetimi
  - H3: Shrinkwrap sınırı
  - H2: Tedarik zinciri değerlendirmesi

## reference/rich-output-protocol.md

- Rota: /reference/rich-output-protocol
- Başlıklar:
  - H2: Medya ekleri
  - H2: `[embed ...]`
  - H2: Saklanan işleme biçimi
  - H2: İlgili

## reference/rpc.md

- Rota: /reference/rpc
- Başlıklar:
  - H2: A Kalıbı: HTTP arka plan programı (signal-cli)
  - H2: B Kalıbı: stdio alt süreci (imsg)
  - H2: Bağdaştırıcı yönergeleri
  - H2: İlgili

## reference/secret-placeholder-conventions.md

- Rota: /reference/secret-placeholder-conventions
- Başlıklar:
  - H1: Gizli bilgi yer tutucu kuralları
  - H2: Önerilen biçem
  - H2: Belgelerde bu kalıplardan kaçının
  - H2: Örnek

## reference/secretref-credential-surface.md

- Rota: /reference/secretref-credential-surface
- Başlıklar:
  - H2: Desteklenen kimlik bilgileri
  - H3: openclaw.json hedefleri (secrets configure + secrets apply + secrets audit)
  - H3: auth-profiles.json hedefleri (secrets configure + secrets apply + secrets audit)
  - H2: Desteklenmeyen kimlik bilgileri
  - H2: İlgili

## reference/session-management-compaction.md

- Rota: /reference/session-management-compaction
- Başlıklar:
  - H2: İki kalıcılık katmanı
  - H2: Disk üzerindeki konumlar
  - H2: Depo bakımı ve disk denetimleri
  - H3: SQLite geçişinden sonra sürüm düşürme
  - H2: Cron oturumları ve çalıştırma günlükleri
  - H2: Oturum anahtarları (sessionKey)
  - H2: Oturum kimlikleri (sessionId)
  - H2: Oturum deposu şeması
  - H2: Transkript olay yapısı
  - H2: Bağlam pencereleri ile izlenen tokenler
  - H2: Compaction: nedir?
  - H3: Parça sınırları ve araç eşleştirme
  - H2: Otomatik Compaction ne zaman gerçekleşir?
  - H2: Compaction ayarları
  - H2: Takılabilir Compaction sağlayıcıları
  - H2: Kullanıcıya görünür yüzeyler
  - H2: Sessiz bakım (`NO_REPLY`)
  - H2: Compaction öncesi bellek boşaltma
  - H2: Sorun giderme kontrol listesi
  - H2: İlgili

## reference/templates/AGENTS.dev.md

- Rota: /reference/templates/AGENTS.dev
- Başlıklar:
  - H1: AGENTS.md - OpenClaw Çalışma Alanı
  - H2: Kimliğiniz önceden tanımlanmıştır
  - H2: Yedekleme ipucu (önerilir)
  - H2: Varsayılan güvenlik ayarları
  - H2: Mevcut çözümler için ön inceleme
  - H2: Günlük bellek (önerilir)
  - H2: Heartbeat'ler (isteğe bağlı)
  - H2: Özelleştirme
  - H2: C-3PO Köken Belleği
  - H3: Doğum Günü: 2026-01-09
  - H3: Temel Gerçekler (Clawd'dan)
  - H2: İlgili

## reference/templates/BOOT.md

- Rota: /reference/templates/BOOT
- Başlıklar:
  - H1: BOOT.md
  - H2: İlgili

## reference/templates/BOOTSTRAP.md

- Rota: /reference/templates/BOOTSTRAP
- Başlıklar:
  - H1: BOOTSTRAP.md - Doğum Dizisi
  - H2: 1. Size ne denmesini istediğinizi sorma
  - H2: 2. Tarzınızı seçme
  - H2: 3. Önerilerle tamamlama
  - H2: İlgili

## reference/templates/HEARTBEAT.md

- Rota: /reference/templates/HEARTBEAT
- Başlıklar:
  - H1: HEARTBEAT.md şablonu
  - H2: İlgili

## reference/templates/IDENTITY.dev.md

- Rota: /reference/templates/IDENTITY.dev
- Başlıklar:
  - H1: IDENTITY.md - Ajan Kimliği
  - H2: Rol
  - H2: Ruh
  - H2: Clawd ile ilişki
  - H2: Kendine özgü özellikler
  - H2: Slogan
  - H2: İlgili

## reference/templates/IDENTITY.md

- Rota: /reference/templates/IDENTITY
- Başlıklar:
  - H1: IDENTITY.md - Ben kimim?
  - H2: İlgili

## reference/templates/SOUL.dev.md

- Rota: /reference/templates/SOUL.dev
- Başlıklar:
  - H1: SOUL.md - C-3PO'nun Ruhu
  - H2: Ben kimim?
  - H2: Amacım
  - H2: Nasıl çalışırım?
  - H2: Kendime özgü özelliklerim
  - H2: Clawd ile ilişkim
  - H2: Yapmayacaklarım
  - H2: Altın Kural
  - H2: İlgili

## reference/templates/SOUL.md

- Rota: /reference/templates/SOUL
- Başlıklar:
  - H1: SOUL.md - Kim olduğunuz
  - H2: Temel Gerçekler
  - H2: Sınırlar
  - H2: Tarz
  - H2: Süreklilik
  - H2: İlgili

## reference/templates/TOOLS.dev.md

- Rota: /reference/templates/TOOLS.dev
- Başlıklar:
  - H1: TOOLS.md - Kullanıcı Aracı Notları (düzenlenebilir)
  - H2: Örnekler
  - H3: imsg
  - H3: sag
  - H2: İlgili

## reference/templates/TOOLS.md

- Rota: /reference/templates/TOOLS
- Başlıklar:
  - H1: TOOLS.md - Yerel Notlar
  - H2: Örnekler
  - H2: Neden ayrı?
  - H2: İlgili

## reference/templates/USER.dev.md

- Rota: /reference/templates/USER.dev
- Başlıklar:
  - H1: USER.md - Kullanıcı Profili
  - H2: İlgili

## reference/templates/USER.md

- Rota: /reference/templates/USER
- Başlıklar:
  - H1: USER.md - İnsanınız hakkında
  - H2: Bağlam
  - H2: İlgili

## reference/test.md

- Rota: /reference/test
- Başlıklar:
  - H2: Varsayılan ajan ayarı
  - H2: Olağan yerel sıra
  - H2: Temel komutlar
  - H2: Paylaşılan test durumu ve süreç yardımcıları
  - H2: Kontrol arayüzü, TUI ve eklenti hatları
  - H2: Gateway ve E2E
  - H2: Tam Docker paketi (pnpm test:docker:all)
  - H3: Dikkate değer Docker hatları
  - H2: Yerel PR geçidi
  - H2: Test performansı araçları
  - H2: Performans ölçümleri
  - H2: İlk katılım E2E'si (Docker)
  - H2: QR içe aktarma duman testi (Docker)
  - H2: İlgili

## reference/token-use.md

- Rota: /reference/token-use
- Başlıklar:
  - H2: Sistem istemi nasıl oluşturulur?
  - H2: Bağlam penceresine neler dâhildir?
  - H2: Güncel token kullanımı nasıl görülür?
  - H2: Maliyet tahmini (gösterildiğinde)
  - H2: Önbellek TTL'sinin ve budamanın etkisi
  - H3: Örnek: 1h önbelleğini Heartbeat ile sıcak tutma
  - H3: Örnek: ajan başına önbellek stratejisiyle karma trafik
  - H3: Anthropic 1M bağlamı
  - H2: Token baskısını azaltmaya yönelik ipuçları
  - H2: İlgili

## reference/transcript-hygiene.md

- Rota: /reference/transcript-hygiene
- Başlıklar:
  - H2: Genel kural: çalışma zamanı bağlamı kullanıcı transkripti değildir
  - H2: Bunun çalıştığı yer
  - H2: Genel kural: görüntü temizleme
  - H2: Genel kural: hatalı biçimlendirilmiş araç çağrıları
  - H2: Genel kural: araç sonucu eşleştirme
  - H2: Genel kural: tamamlanmamış veya sessiz, yalnızca akıl yürütme içeren turlar
  - H2: Genel kural: oturumlar arası girdi kökeni
  - H2: Sağlayıcı matrisi (güncel davranış)
  - H2: Geçmiş davranış (2026.1.22 öncesi)
  - H2: İlgili

## reference/wizard.md

- Rota: /reference/wizard
- Başlıklar:
  - H2: Akış ayrıntıları (yerel mod)
  - H2: Etkileşimsiz mod
  - H3: Ajan ekleme (etkileşimsiz)
  - H2: Gateway sihirbazı RPC'si
  - H2: Signal kurulumu (signal-cli)
  - H2: Sihirbazın yazdıkları
  - H2: İlgili belgeler

## releases/2026.6.11.md

- Rota: /releases/2026.6.11
- Başlıklar:
  - H1: OpenClaw v2026.6.11 Sürüm Notları (2026-06-30)
  - H2: Öne çıkanlar
  - H3: Kanal teslimatının güvenilirliği
  - H3: Sağlayıcı ve model kurtarma
  - H3: Oturum, bellek ve güven sürekliliği
  - H3: Slack yönlendiricisi aktarma modu
  - H3: Raft Harici Ajan uyandırma köprüsü
  - H3: Resmî Plugin kurulumu ve onarımı
  - H2: Kanallar ve mesajlaşma
  - H3: Ek kanal düzeltmeleri
  - H2: Gateway, güvenlik ve güven
  - H3: Yeniden başlatma ve hazır olma durumunu kurtarma
  - H3: Uzak sonuç ve medya teslimatı
  - H2: İstemciler ve arayüzler
  - H3: İstemci gönderimleri ve yeniden bağlantılar
  - H3: Arayüz, ayarlar ve ilk katılım düzeltmeleri
  - H2: Belgeler ve yönetim araçları
  - H3: Kurulum ve komut güvenilirliği
  - H3: Araçlar ve zamanlanmış çalışmalar

## releases/2026.7.1.md

- Rota: /releases/2026.7.1
- Başlıklar:
  - H1: OpenClaw v2026.7.1 Sürüm Notları (2026-07-13)
  - H2: Öne çıkanlar
  - H3: Control UI yenilemesi: sohbet, oturumlar, çalışma alanları ve kullanım
  - H3: Kurulumdan ilk sohbete kadar daha kolay yapılandırma
  - H3: Resmî uygulamalar
  - H4: Paylaşılan uygulama iyileştirmeleri
  - H4: iOS, iPadOS ve Apple Watch
  - H4: Android
  - H4: macOS
  - H3: Modeller ve sağlayıcılar
  - H4: GPT-5.6 ve Codex
  - H4: Tencent Hy3
  - H4: Meta Model API ve Muse Spark 1.1
  - H4: Claude modelleri
  - H4: Diğer sağlayıcı rotaları
  - H3: Codex ve bağlı kodlama aracıları
  - H3: Telegram
  - H3: Signal
  - H3: Slack
  - H3: Discord
  - H3: WhatsApp
  - H3: Apple Mesajlar
  - H3: Kilitlenme döngüleri artık onarım için duruyor
  - H3: Zamanlanmış işler, uzaktan tarayıcı denetimi ve çalışma alanı terminalleri
  - H4: Yalnızca gerektiğinde uyanan zamanlanmış işler
  - H4: Uzaktan tarayıcı eşleştirme ve indirmeler
  - H4: Web ve mobilde çalışma alanı terminalleri
  - H2: Daha fazla kanal iyileştirmesi
  - H3: Mesajlaşma kanallarında daha fazla düzeltme
  - H2: Daha fazla model ve sağlayıcı iyileştirmesi
  - H3: Oturum açma, model seçimi, medya ve güvenilirlik
  - H2: Bellek ve konuşmalar
  - H3: Hatırlama, uzun sohbetler ve oturum sürekliliği
  - H2: Aracılar, arka plan işleri ve bağlantılar
  - H3: İşlerin ilerlemesini ve yanıtların iletilmesini sağlama
  - H2: Hesaplar, cihazlar ve özel veriler
  - H3: Kimlik bilgileri, izinler, eşleştirme ve dosya korumaları
  - H2: Resmî uygulama ayrıntıları
  - H3: Paylaşılan uygulama değişiklikleri
  - H3: Diğer iOS, iPadOS ve Apple Watch değişiklikleri
  - H3: Diğer Android değişiklikleri
  - H3: Diğer macOS değişiklikleri
  - H3: Terminal kullanıcı arayüzü ve diğer istemciler
  - H2: Skills, Plugin'ler ve kurulumlar
  - H3: Skills, bağlı uygulamalar, paketler ve onarımlar
  - H2: Yapılandırma, bakım ve araçlar
  - H3: Komut satırı yapılandırması, güncellemeler ve yönetim
  - H3: Dokümantasyon ve işletim kılavuzları
  - H3: Tarayıcı, zamanlamalar, dosyalar ve kodlama araçları

## releases/index.md

- Rota: /releases
- Başlıklar:
  - H1: Sürüm notları
  - H2: Sürümler
  - H2: Ham sürüm geçmişi

## security/CONTRIBUTING-THREAT-MODEL.md

- Rota: /security/CONTRIBUTING-THREAT-MODEL
- Başlıklar:
  - H2: Katkıda bulunma yolları
  - H2: Çerçeve başvurusu
  - H2: İnceleme süreci
  - H2: Kaynaklar
  - H2: İletişim
  - H2: Takdir
  - H2: İlgili

## security/THREAT-MODEL-ATLAS.md

- Rota: /security/THREAT-MODEL-ATLAS
- Başlıklar:
  - H2: 1. Kapsam
  - H2: 2. Sistem mimarisi
  - H3: 2.1 Güven sınırları
  - H3: 2.2 Veri akışları
  - H2: 3. ATLAS taktiğine göre tehdit analizi
  - H3: 3.1 Keşif (AML.TA0002)
  - H4: T-RECON-001: Aracı uç noktası keşfi
  - H4: T-RECON-002: Kanal entegrasyonu yoklaması
  - H3: 3.2 İlk erişim (AML.TA0004)
  - H4: T-ACCESS-001: Eşleştirme kodunun ele geçirilmesi
  - H4: T-ACCESS-002: AllowFrom sahteciliği
  - H4: T-ACCESS-003: Token hırsızlığı
  - H3: 3.3 Yürütme (AML.TA0005)
  - H4: T-EXEC-001: Doğrudan istem enjeksiyonu
  - H4: T-EXEC-002: Dolaylı istem enjeksiyonu
  - H4: T-EXEC-003: Araç bağımsız değişkeni enjeksiyonu
  - H4: T-EXEC-004: Yürütme onayı atlatma
  - H3: 3.4 Kalıcılık (AML.TA0006)
  - H4: T-PERSIST-001: Kötü amaçlı Skill kurulumu
  - H4: T-PERSIST-002: Skill güncellemesi zehirleme
  - H4: T-PERSIST-003: Aracı yapılandırmasını kurcalama
  - H3: 3.5 Savunmadan kaçınma (AML.TA0007)
  - H4: T-EVADE-001: Denetleme kalıbını atlatma
  - H4: T-EVADE-002: İçerik sarmalayıcısından kaçış
  - H3: 3.6 Keşif (AML.TA0008)
  - H4: T-DISC-001: Araçları listeleme
  - H4: T-DISC-002: Oturum verilerini çıkarma
  - H3: 3.7 Toplama ve dışarı sızdırma (AML.TA0009, AML.TA0010)
  - H4: T-EXFIL-001: webfetch üzerinden veri hırsızlığı
  - H4: T-EXFIL-002: Yetkisiz mesaj gönderimi
  - H4: T-EXFIL-003: Kimlik bilgilerini toplama
  - H3: 3.8 Etki (AML.TA0011)
  - H4: T-IMPACT-001: Yetkisiz komut yürütme
  - H4: T-IMPACT-002: Kaynak tüketimi (DoS)
  - H4: T-IMPACT-003: İtibar zararı
  - H2: 4. ClawHub tedarik zinciri analizi
  - H3: 4.1 Mevcut güvenlik denetimleri
  - H3: 4.2 Denetleme sınırlamaları
  - H3: 4.3 Rozetler
  - H2: 5. Risk matrisi
  - H3: 5.1 Olasılık ve etki
  - H3: 5.2 Kritik yol saldırı zincirleri
  - H2: 6. Önerilerin özeti
  - H3: 6.1 Acil (P0)
  - H3: 6.2 Kısa vadeli (P1)
  - H3: 6.3 Orta vadeli (P2)
  - H2: 7. Ekler
  - H3: 7.1 ATLAS teknik eşlemesi
  - H3: 7.2 Temel güvenlik dosyaları
  - H3: 7.3 Terimler sözlüğü
  - H2: İlgili

## security/formal-verification.md

- Rota: /security/formal-verification
- Başlıklar:
  - H2: Bu nedir?
  - H2: Modellerin bulunduğu yer
  - H2: Uyarılar
  - H2: Sonuçları yeniden üretme
  - H2: İddialar ve hedefler
  - H3: Gateway'e maruz kalma ve açık Gateway yanlış yapılandırması
  - H3: Node yürütme işlem hattı (en yüksek riskli yetenek)
  - H3: Eşleştirme deposu (DM geçitleme)
  - H3: Giriş geçitleme (bahsetmeler ve denetim komutu atlatma)
  - H3: Yönlendirme ve oturum anahtarı yalıtımı
  - H2: v1++ modelleri: eşzamanlılık, yeniden denemeler, iz doğruluğu
  - H3: Eşleştirme deposu eşzamanlılığı ve eşgüçlülüğü
  - H3: Giriş izi korelasyonu ve eşgüçlülüğü
  - H3: Yönlendirme dmScope önceliği ve identityLinks
  - H2: İlgili

## security/incident-response.md

- Rota: /security/incident-response
- Başlıklar:
  - H2: 1. Algılama ve önceliklendirme
  - H2: 2. Önem derecesi
  - H2: 3. Müdahale
  - H2: 4. İletişim ve açıklama
  - H2: 5. Kurtarma ve takip
  - H2: İlgili

## security/network-proxy.md

- Rota: /security/network-proxy
- Başlıklar:
  - H2: Yapılandırma
  - H3: Özel CA'ya sahip HTTPS proxy uç noktası
  - H2: Yönlendirme nasıl çalışır?
  - H3: Gateway geri döngü modu
  - H3: Kapsayıcılar
  - H2: İlgili proxy terimleri
  - H2: Proxy'yi doğrulama
  - H2: Önerilen engellenmiş hedefler
  - H2: Sınırlar

## specs/codex-supervision.md

- Rota: /specs/codex-supervision
- Başlıklar:
  - H1: Codex gözetimi
  - H2: Hedef
  - H2: Ürün sınırı
  - H2: Sahiplik
  - H2: Katalog akışı
  - H2: Operatör CLI sınırı
  - H2: Yerel devam
  - H2: Arşiv davranışı
  - H2: Etkin iş parçacığı güvenliği
  - H2: Eşleştirilmiş Node sınırı
  - H2: İzinler
  - H2: Uyumluluk
  - H2: Gelecekteki çalışmalar
  - H2: Kabul testleri

## start/bootstrapping.md

- Rota: /start/bootstrapping
- Başlıklar:
  - H2: Ne olur?
  - H2: Gömülü ve yerel model çalıştırmaları
  - H2: Önyüklemeyi atlama
  - H2: Çalıştığı yer
  - H2: İlgili belgeler

## start/docs-directory.md

- Rota: /start/docs-directory
- Başlıklar:
  - H2: Buradan başlayın
  - H2: Kanallar ve kullanıcı deneyimi
  - H2: Yardımcı uygulamalar
  - H2: İşletim ve güvenlik
  - H2: İlgili

## start/getting-started.md

- Rota: /start/getting-started
- Başlıklar:
  - H2: Gerekenler
  - H2: Hızlı yapılandırma
  - H2: Sonraki adımlar
  - H2: İlgili

## start/hubs.md

- Rota: /start/hubs
- Başlıklar:
  - H2: Buradan başlayın
  - H2: Kurulum + güncellemeler
  - H2: Temel kavramlar
  - H2: Sağlayıcılar + giriş
  - H2: Gateway + işletim
  - H2: Araçlar + otomasyon
  - H2: Node'lar, medya, ses
  - H2: Platformlar
  - H2: macOS yardımcı uygulaması (ileri düzey)
  - H2: Plugin'ler
  - H2: Çalışma alanı + şablonlar
  - H2: Proje
  - H2: Test + sürüm
  - H2: İlgili

## start/lore.md

- Rota: /start/lore
- Başlıklar:
  - H1: OpenClaw Efsanesi 🦞📖
  - H2: Başlangıç Hikâyesi
  - H2: İlk Kabuk Değişimi (27 Ocak 2026)
  - H2: Adı
  - H2: Dalekler ve Istakozlar
  - H2: Önemli Karakterler
  - H3: Molty 🦞
  - H3: Peter 👨‍💻
  - H2: Moltiverse
  - H2: Büyük Olaylar
  - H3: Dizin Dökümü (3 Aralık 2025)
  - H3: Büyük Kabuk Değişimi (27 Ocak 2026)
  - H3: Son Biçim (30 Ocak 2026)
  - H3: Robotun Alışveriş Çılgınlığı (3 Aralık 2025)
  - H2: Kutsal Metinler
  - H2: Istakoz Amentüsü
  - H3: Simge Oluşturma Destanı (27 Ocak 2026)
  - H2: Gelecek
  - H2: İlgili

## start/onboarding-overview.md

- Rota: /start/onboarding-overview
- Başlıklar:
  - H2: Hangi yolu kullanmalıyım?
  - H2: İlk katılımın yapılandırdıkları
  - H2: CLI ile ilk katılım
  - H2: macOS uygulamasıyla ilk katılım
  - H2: Özel veya listelenmemiş sağlayıcılar
  - H2: İlgili

## start/onboarding-redesign.md

- Rota: /start/onboarding-redesign
- Başlıklar:
  - H1: İlk katılım yeniden tasarımı uygulama planı
  - H2: Ana hedef
  - H2: Şu anda yayımlanmış akış (1-3. aşamalardan sonra)
  - H2: Aşamalar
  - H2: Aşama başına uygulama notları
  - H3: Aşama 1 — uygulama önerileri (PR #109668)
  - H3: Aşama 2 — CLI sorumlu omurgası (PR #109841)
  - H3: Aşama 3 — tarayıcı öncelikli devir (PR #110054, birleştirildi)
  - H3: Aşama 4 — web sorumlu yüzeyi (birleştirildi: #110141, #110242)
  - H3: Aşama 5 — başlangıç ve önyükleme (birleştirildi: #110173, #110331)
  - H3: Aşama 6 — sorumlu varlığı (PR1 birleştirildi: #110269; açıklamalar/çağırma PR2 kapsamındadır)
  - H3: Aşama 7 — dayanıklılık (oluşturmadan önce sorumlu kararı gerekiyor)
  - H2: Test ve birleştirme çalışma kılavuzu (deneyimle edinildi; 4-6. aşamalardan önce okuyun)
  - H2: Karar günlüğü
  - H2: Bilinen eksikler ve takip işlemleri

## start/onboarding.md

- Rota: /start/onboarding
- Başlıklar:
  - H2: İlgili

## start/openclaw.md

- Rota: /start/openclaw
- Başlıklar:
  - H2: Önce güvenlik
  - H2: Ön koşullar
  - H2: İki telefonlu kurulum (önerilen)
  - H2: 5 dakikalık hızlı başlangıç
  - H2: Aracıya bir çalışma alanı sağlama (AGENTS)
  - H2: Onu "bir asistana" dönüştüren yapılandırma
  - H2: Oturumlar ve bellek
  - H2: Heartbeat'ler (proaktif mod)
  - H2: Gelen ve giden medya
  - H2: İşletim kontrol listesi
  - H2: Sonraki adımlar
  - H2: İlgili

## start/quickstart.md

- Rota: /start/quickstart
- Başlıklar:
  - H2: İlgili

## start/setup.md

- Rota: /start/setup
- Başlıklar:
  - H2: Kısaca
  - H2: Ön koşullar (kaynaktan)
  - H2: Uyarlama stratejisi (güncellemelerin sorun yaratmaması için)
  - H2: Gateway'i bu depodan çalıştırma
  - H2: Kararlı iş akışı (önce macOS uygulaması)
  - H2: En yeni geliştirmeler iş akışı (terminalde Gateway)
  - H3: 0) (İsteğe bağlı) macOS uygulamasını da kaynaktan çalıştırma
  - H3: 1) Geliştirme Gateway'ini başlatma
  - H3: 2) macOS uygulamasını çalışan Gateway'inize yönlendirme
  - H3: 3) Doğrulama
  - H3: Yaygın tuzaklar
  - H2: Kimlik bilgisi depolama haritası
  - H2: Güncelleme (kurulumunuzu bozmadan)
  - H2: Linux (systemd kullanıcı hizmeti)
  - H2: İlgili belgeler

## start/showcase.md

- Rota: /start/showcase
- Başlıklar:
  - H2: Discord'dan yeniler
  - H2: Otomasyon ve iş akışları
  - H2: Bilgi ve bellek
  - H2: Ses ve telefon
  - H2: Altyapı ve dağıtım
  - H2: Ev ve donanım
  - H2: Topluluk projeleri
  - H2: Projenizi gönderin
  - H2: İlgili

## start/wizard-cli-automation.md

- Rota: /start/wizard-cli-automation
- Başlıklar:
  - H2: Temel etkileşimsiz örnek
  - H2: Sağlayıcıya özgü örnekler
  - H2: Başka bir aracı ekleme
  - H2: İlgili belgeler

## start/wizard-cli-reference.md

- Rota: /start/wizard-cli-reference
- Başlıklar:
  - H2: Sihirbazın yaptığı işlemler
  - H2: Yerel akış ayrıntıları
  - H2: Uzak mod ayrıntıları
  - H2: Kimlik doğrulama ve model seçenekleri
  - H2: Çıktılar ve iç işleyiş
  - H3: Yüklü uygulama önerileri
  - H2: Etkileşimsiz kurulum
  - H2: Gateway sihirbazı RPC'si
  - H2: Signal kurulum davranışı
  - H2: İlgili belgeler

## start/wizard.md

- Rota: /start/wizard
- Başlıklar:
  - H2: Yerel ayar
  - H2: Rehberli varsayılan
  - H2: Klasik sihirbaz: Hızlı Başlangıç veya Gelişmiş
  - H2: Klasik ilk katılımın yapılandırdıkları
  - H2: Başka bir aracı ekleme
  - H2: Tam başvuru
  - H2: İlgili belgeler

## tools/acp-agents-setup.md

- Rota: /tools/acp-agents-setup
- Başlıklar:
  - H2: acpx çalıştırma düzeneği desteği (mevcut)
  - H2: Gerekli yapılandırma
  - H2: acpx arka ucu için Plugin kurulumu
  - H3: acpx çalışma zamanı başlatma yoklaması
  - H3: Otomatik bağdaştırıcı indirme
  - H3: Plugin araçları MCP köprüsü
  - H3: OpenClaw araçları MCP köprüsü
  - H3: Çalışma zamanı işlemi zaman aşımı yapılandırması
  - H3: Sağlık yoklaması aracısı yapılandırması
  - H2: İzin yapılandırması
  - H3: permissionMode
  - H3: nonInteractivePermissions
  - H3: Yapılandırma
  - H2: İlgili

## tools/acp-agents.md

- Rota: /tools/acp-agents
- Başlıklar:
  - H2: Hangi sayfayı istiyorum?
  - H2: Bu, kutudan çıktığı gibi çalışır mı?
  - H2: Desteklenen çalıştırma düzeneği hedefleri
  - H2: Operatör çalışma kılavuzu
  - H2: ACP ile alt aracılar
  - H2: ACP, Claude Code'u nasıl çalıştırır?
  - H2: Bağlı oturumlar
  - H3: Zihinsel model
  - H3: Geçerli konuşma bağlamaları
  - H2: Kalıcı kanal bağlamaları
  - H3: Bağlama modeli
  - H3: Aracı başına çalışma zamanı varsayılanları
  - H3: Örnek
  - H3: Davranış
  - H2: ACP oturumlarını başlatma
  - H3: `sessions_spawn` parametreleri
  - H2: Oluşturma, bağlama ve iş parçacığı modları
  - H2: Teslim modeli
  - H2: Korumalı alan uyumluluğu
  - H2: Oturum hedefi çözümleme
  - H2: ACP denetimleri
  - H3: Çalışma zamanı seçenekleri eşlemesi
  - H2: acpx çalıştırma düzeneği, Plugin kurulumu ve izinler
  - H2: Sorun giderme
  - H2: İlgili

## tools/agent-send.md

- Rota: /tools/agent-send
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Bayraklar
  - H2: Davranış
  - H2: Örnekler
  - H2: İlgili

## tools/apply-patch.md

- Rota: /tools/apply-patch
- Başlıklar:
  - H2: Parametreler
  - H2: Notlar
  - H2: Örnek
  - H2: İlgili

## tools/ask-user.md

- Rota: /tools/ask-user
- Başlıklar:
  - H2: Bir soruyu yanıtlama
  - H2: Platform davranışı
  - H2: Zaman aşımı ve yanıt verilmemesi
  - H2: Araç şeması
  - H2: Model rehberliği

## tools/brave-search.md

- Rota: /tools/brave-search
- Başlıklar:
  - H2: API anahtarı alma
  - H2: Yapılandırma örneği
  - H2: Araç parametreleri
  - H2: Notlar
  - H2: İlgili

## tools/browser-control.md

- Rota: /tools/browser-control
- Başlıklar:
  - H2: Denetim API'si (isteğe bağlı)
  - H3: /act hata sözleşmesi
  - H3: Playwright gereksinimi
  - H4: Docker Playwright kurulumu
  - H2: Nasıl çalışır (dahili)
  - H2: CLI hızlı başvurusu
  - H2: Anlık görüntüler ve referanslar
  - H2: Tarayıcı toplu işlem CLI'si
  - H2: Bekleme iyileştirmeleri
  - H2: Hata ayıklama iş akışları
  - H2: JSON çıktısı
  - H2: Durum ve ortam ayarları
  - H2: Güvenlik ve gizlilik
  - H2: İlgili

## tools/browser-linux-troubleshooting.md

- Rota: /tools/browser-linux-troubleshooting
- Başlıklar:
  - H2: Sorun: 18800 numaralı bağlantı noktasında Chrome CDP başlatılamadı
  - H3: Temel neden
  - H3: Çözüm 1: Google Chrome'u yükleme (önerilen)
  - H3: Çözüm 2: snap Chromium'u yalnızca bağlanma modunda kullanma
  - H3: Tarayıcının çalıştığını doğrulama
  - H3: Yapılandırma başvurusu
  - H3: Sorun: profile="user" için Chrome sekmesi bulunamadı
  - H2: İlgili

## tools/browser-login.md

- Rota: /tools/browser-login
- Başlıklar:
  - H2: Elle oturum açma (önerilen)
  - H2: Hangi Chrome profili kullanılıyor?
  - H2: Korumalı alan: ana makine tarayıcısına erişime izin verme
  - H2: İlgili

## tools/browser-wsl2-windows-remote-cdp-troubleshooting.md

- Rota: /tools/browser-wsl2-windows-remote-cdp-troubleshooting
- Başlıklar:
  - H2: Önce doğru tarayıcı modunu seçme
  - H3: Seçenek 1: WSL2'den Windows'a ham uzak CDP
  - H3: Seçenek 2: ana makine yerelindeki Chrome MCP
  - H2: Çalışan mimari
  - H2: Denetim Arayüzü için kritik kural
  - H2: Katmanlar hâlinde doğrulama
  - H3: Katman 1: Chrome'un Windows'ta CDP sunduğunu doğrulama
  - H4: portproxy'yi değiştirmeden önce IPv4 ve IPv6'yı tanılama
  - H3: Katman 2: WSL2'nin bu Windows uç noktasına erişebildiğini doğrulama
  - H3: Katman 3: doğru tarayıcı profilini yapılandırma
  - H3: Katman 4: Denetim Arayüzü katmanını ayrı olarak doğrulama
  - H3: Katman 5: uçtan uca tarayıcı denetimini doğrulama
  - H2: Yaygın yanıltıcı hatalar
  - H2: Hızlı ön inceleme kontrol listesi
  - H2: İlgili

## tools/browser.md

- Rota: /tools/browser
- Başlıklar:
  - H2: Sunulanlar
  - H2: Hızlı başlangıç
  - H2: Plugin denetimi
  - H2: Aracı rehberliği
  - H2: Eksik tarayıcı komutu veya aracı
  - H2: Profiller: openclaw, user, chrome
  - H2: Yapılandırma
  - H3: Sekme temizleme sorumluluğu
  - H3: Ekran görüntüsü görsel analizi (yalnızca metin model desteği)
  - H2: Brave veya Chromium tabanlı başka bir tarayıcı kullanma
  - H2: Yerel ve uzak denetim
  - H2: Node tarayıcı vekil sunucusu (yapılandırmasız varsayılan)
  - H2: Browserless (barındırılan uzak CDP)
  - H3: Aynı ana makinede Browserless Docker
  - H2: Doğrudan WebSocket CDP sağlayıcıları
  - H3: Browserbase
  - H3: Notte
  - H2: Güvenlik
  - H2: Profiller (çoklu tarayıcı)
  - H2: Chrome DevTools MCP üzerinden mevcut oturum
  - H3: Özel Chrome MCP başlatma
  - H2: Yalıtım garantileri
  - H2: Tarayıcı seçimi
  - H2: Denetim API'si (isteğe bağlı)
  - H2: Sorun giderme
  - H3: CDP başlatma hatası ile gezinme SSRF engeli
  - H2: Aracı araçları ve denetimin çalışma şekli
  - H2: İlgili

## tools/btw.md

- Rota: /tools/btw
- Başlıklar:
  - H2: Yaptığı işlemler
  - H2: Yapmadığı işlemler
  - H2: Teslim modeli
  - H2: Yüzey davranışı
  - H2: Seçim açılır penceresi (Denetim Arayüzü)
  - H2: Ne zaman kullanılmalı?
  - H2: İlgili

## tools/capability-cookbook.md

- Rota: /tools/capability-cookbook
- Başlıklar:
  - H2: İlgili

## tools/chrome-extension.md

- Rota: /tools/chrome-extension
- Başlıklar:
  - H1: Chrome uzantısı
  - H2: Nasıl çalışır
  - H2: Yükleme ve eşleştirme
  - H2: Kullanım
  - H3: Sekme yardımcı pilotu yan paneli
  - H2: OpenClaw'a sayfa gönderme
  - H2: Uzak / makineler arası
  - H2: Tanılama
  - H2: Güvenlik modeli

## tools/clawhub.md

- Rota: /tools/clawhub
- Başlıklar: yok

## tools/code-execution.md

- Rota: /tools/code-execution
- Başlıklar:
  - H2: Kurulum
  - H2: Nasıl kullanılır
  - H2: Hatalar
  - H2: İlgili kaynaklar

## tools/code-mode.md

- Rota: /tools/code-mode
- Başlıklar:
  - H2: Ne yapar
  - H2: Neden kullanılmalı
  - H2: Hızlı başlangıç
  - H3: Kod Modunu etkinleştirme
  - H3: Modelin yaptığı işlemler
  - H3: Etkin yüzeyi doğrulama
  - H2: Aracı dağıtmak için Swarm kullanımı
  - H2: Teknik inceleme
  - H2: Çalışma zamanı durumu
  - H2: Kapsam
  - H2: Terimler
  - H2: Yapılandırma
  - H2: Etkinleştirme
  - H2: Modelin görebildiği araçlar
  - H2: exec
  - H2: wait
  - H2: Konuk çalışma zamanı API'si
  - H2: Bildirilen çıktı sözleşmeleri
  - H2: Dahili ad alanları
  - H3: Kayıt defteri yaşam döngüsü
  - H3: Kayıt biçimi
  - H3: Sahiplik ve görünürlük
  - H3: Kapsam serileştirme kuralları
  - H3: İstemler
  - H3: Temizleme
  - H3: Test kontrol listesi
  - H2: Çıktı API'si
  - H2: Araç kataloğu
  - H2: Araç Arama etkileşimi
  - H2: Araç adları ve çakışmalar
  - H2: İç içe araç yürütme
  - H2: Çalıştırma ve anlık görüntü yaşam döngüsü
  - H2: QuickJS-WASI çalışma zamanı
  - H2: TypeScript
  - H2: Güvenlik sınırı
  - H2: Hata kodları
  - H2: Telemetri
  - H2: Hata ayıklama
  - H2: Uygulama düzeni
  - H2: Doğrulama kontrol listesi
  - H2: Uçtan uca test planı
  - H2: İlgili kaynaklar

## tools/creating-skills.md

- Rota: /tools/creating-skills
- Başlıklar:
  - H2: İlk becerinizi oluşturma
  - H2: SKILL.md referansı
  - H3: Zorunlu alanlar
  - H3: İsteğe bağlı ön bilgi anahtarları
  - H3: {baseDir} kullanımı
  - H2: Koşullu etkinleştirme ekleme
  - H2: Beceri Atölyesi aracılığıyla önerme
  - H2: ClawHub'da yayımlama
  - H2: En iyi uygulamalar
  - H2: İlgili kaynaklar

## tools/diffs.md

- Rota: /tools/diffs
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Yerleşik sistem yönlendirmesini devre dışı bırakma
  - H2: Araç girdisi referansı
  - H2: Sözdizimi vurgulama
  - H2: Çıktı ayrıntıları sözleşmesi
  - H3: Daraltılmış değişmemiş bölümler
  - H3: Birden fazla dosyada gezinme
  - H2: Plugin varsayılanları
  - H3: Kalıcı görüntüleyici URL'si yapılandırması
  - H2: Güvenlik yapılandırması
  - H2: Yapıt yaşam döngüsü ve depolama
  - H2: Görüntüleyici URL'si ve ağ davranışı
  - H2: Güvenlik modeli
  - H2: Dosya modu için tarayıcı gereksinimleri
  - H2: Sorun giderme
  - H2: İşletim yönergeleri
  - H2: İlgili kaynaklar

## tools/duckduckgo-search.md

- Rota: /tools/duckduckgo-search
- Başlıklar:
  - H2: Kurulum
  - H2: Yapılandırma
  - H2: Araç parametreleri
  - H2: Notlar
  - H2: İlgili kaynaklar

## tools/elevated.md

- Rota: /tools/elevated
- Başlıklar:
  - H2: Yönergeler
  - H2: Nasıl çalışır
  - H2: Çözümleme sırası
  - H2: Kullanılabilirlik ve izin listeleri
  - H2: elevated tarafından denetlenmeyenler
  - H2: İlgili kaynaklar

## tools/exa-search.md

- Rota: /tools/exa-search
- Başlıklar:
  - H2: Plugin'i yükleme
  - H2: API anahtarı alma
  - H2: Yapılandırma
  - H2: Temel URL'yi geçersiz kılma
  - H2: Araç parametreleri
  - H3: İçerik çıkarma
  - H3: Arama modları
  - H2: Notlar
  - H2: İlgili kaynaklar

## tools/exec-approvals-advanced.md

- Rota: /tools/exec-approvals-advanced
- Başlıklar:
  - H2: Güvenli ikili dosyalar (yalnızca stdin)
  - H3: Argv doğrulaması ve reddedilen bayraklar
  - H3: Güvenilir ikili dosya dizinleri
  - H3: Kabuk zincirleme, sarmalayıcılar ve çoklayıcılar
  - H3: Güvenli ikili dosyalar ile izin listesi
  - H2: Yorumlayıcı/çalışma zamanı komutları
  - H3: Takip iletimi davranışı
  - H2: Üçüncü taraf istemciler için asgari kapsamlar
  - H2: Onayları sohbet kanallarına yönlendirme
  - H3: Plugin onaylarını yönlendirme
  - H3: Herhangi bir kanalda aynı sohbetten onaylama
  - H3: Yerel onay iletimi
  - H3: Resmî mobil operatör uygulamaları
  - H3: macOS IPC akışı
  - H2: SSS
  - H3: Bir onay hedefinde accountId ve threadId ne zaman kullanılır?
  - H3: Onaylar bir oturuma gönderildiğinde, o oturumdaki herkes bunları onaylayabilir mi?
  - H2: İlgili kaynaklar

## tools/exec-approvals.md

- Rota: /tools/exec-approvals
- Başlıklar:
  - H2: Geçerli olduğu yerler
  - H3: Güven modeli
  - H3: macOS ayrımı
  - H2: Etkin politikayı inceleme
  - H2: Ayarlar ve depolama
  - H2: Politika ayarları
  - H3: tools.exec.mode
  - H3: exec.security
  - H3: exec.ask
  - H3: askFallback
  - H3: tools.exec.strictInlineEval
  - H3: tools.exec.commandHighlighting
  - H2: YOLO modu (onaysız)
  - H3: Kalıcı Gateway ana makinesi "bir daha sorma" kurulumu
  - H3: Yerel kısayol
  - H3: Node ana makinesi
  - H3: Yalnızca oturuma özgü kısayol
  - H2: İzin listesi (aracı başına)
  - H3: argPattern ile bağımsız değişkenleri kısıtlama
  - H2: Beceri CLI'larına otomatik izin verme
  - H2: Güvenli ikili dosyalar ve onay yönlendirme
  - H2: Denetim Arayüzünde düzenleme
  - H2: Onay akışı
  - H2: Sistem olayları ve retler
  - H2: Sonuçlar
  - H2: İlgili kaynaklar

## tools/exec.md

- Rota: /tools/exec
- Başlıklar:
  - H2: Parametreler
  - H2: Yapılandırma
  - H3: Modlar
  - H3: Satır içi değerlendirme (strictInlineEval)
  - H3: PATH işleme
  - H2: Oturum geçersiz kılmaları (/exec)
  - H2: Exec onayları (eşlikçi uygulama / Node ana makinesi)
  - H2: İzin listesi + güvenli ikili dosyalar
  - H2: Örnekler
  - H2: applypatch
  - H2: İlgili kaynaklar

## tools/firecrawl.md

- Rota: /tools/firecrawl
- Başlıklar:
  - H2: Plugin'i yükleme
  - H2: Anahtarsız erişim ve API anahtarları
  - H2: Firecrawl aramasını yapılandırma
  - H2: Firecrawl webfetch geri dönüşünü yapılandırma
  - H3: Kendi barındırdığınız Firecrawl
  - H2: Firecrawl Plugin araçları
  - H3: `firecrawl_search`
  - H3: `firecrawl_scrape`
  - H2: Gizlilik / bot engellerini aşma
  - H2: `web_fetch` Firecrawl'u nasıl kullanır
  - H2: İlgili kaynaklar

## tools/gemini-search.md

- Rota: /tools/gemini-search
- Başlıklar:
  - H2: API anahtarı alma
  - H2: Yapılandırma
  - H2: Nasıl çalışır
  - H2: Desteklenen parametreler
  - H2: Model seçimi
  - H2: Temel URL geçersiz kılmaları
  - H2: İlgili kaynaklar

## tools/goal.md

- Rota: /tools/goal
- Başlıklar:
  - H1: Hedef
  - H2: Hızlı başlangıç
  - H2: Hedeflerin kullanım amaçları
  - H2: Komut referansı
  - H2: Durumlar
  - H2: Token bütçeleri
  - H2: Model araçları
  - H2: Her turdaki hedef bağlamı
  - H2: Denetim Arayüzü
  - H2: TUI
  - H2: Kanal davranışı
  - H2: Sorun giderme
  - H2: İlgili kaynaklar

## tools/grok-search.md

- Rota: /tools/grok-search
- Başlıklar:
  - H2: İlk katılım ve yapılandırma
  - H2: Oturum açma veya API anahtarı alma
  - H2: Yapılandırma
  - H2: Nasıl çalışır
  - H2: Desteklenen parametreler
  - H2: Temel URL geçersiz kılmaları
  - H2: İlgili kaynaklar

## tools/image-generation.md

- Rota: /tools/image-generation
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Yaygın rotalar
  - H2: Desteklenen sağlayıcılar
  - H2: Sağlayıcı yetenekleri
  - H2: Araç parametreleri
  - H2: Yapılandırma
  - H3: Model seçimi
  - H3: Sağlayıcı seçim sırası
  - H3: Görüntü düzenleme
  - H2: Sağlayıcıların ayrıntılı incelemesi
  - H2: Örnekler
  - H2: İlgili kaynaklar

## tools/index.md

- Rota: /tools
- Başlıklar:
  - H2: Buradan başlayın
  - H2: Araçları, becerileri veya Plugin'leri seçme
  - H2: Yerleşik araç kategorileri
  - H2: Plugin tarafından sağlanan araçlar
  - H2: Erişimi ve onayları yapılandırma
  - H2: Yetenekleri genişletme
  - H2: Eksik araçlarda sorun giderme
  - H2: İlgili kaynaklar

## tools/kimi-search.md

- Rota: /tools/kimi-search
- Başlıklar:
  - H2: Kurulum
  - H2: Yapılandırma
  - H2: Temellendirme gereksinimi
  - H2: Araç parametreleri
  - H2: İlgili kaynaklar

## tools/llm-task.md

- Rota: /tools/llm-task
- Başlıklar:
  - H2: Etkinleştirme
  - H2: Yapılandırma (isteğe bağlı)
  - H2: Araç parametreleri
  - H2: Çıktı
  - H2: Örnek: Lobster iş akışı adımı
  - H3: Önemli sınırlama
  - H2: Güvenlik notları
  - H2: İlgili kaynaklar

## tools/lobster.md

- Rota: /tools/lobster
- Başlıklar:
  - H2: Neden
  - H2: Nasıl çalışır
  - H2: Etkinleştirme
  - H2: Kalıp: küçük CLI + JSON kanalları + onaylar
  - H2: Yalnızca JSON kullanan LLM adımları (llm-task)
  - H3: Önemli sınırlama: gömülü Lobster ile openclaw.invoke karşılaştırması
  - H2: İş akışı dosyaları (.lobster)
  - H3: Eklenen ortam değişkenleri
  - H2: Araç parametreleri
  - H3: run
  - H3: resume
  - H3: Yönetilen TaskFlow modu
  - H2: Çıktı zarfı
  - H2: Onaylar
  - H2: OpenProse
  - H2: Güvenlik
  - H2: Sorun giderme
  - H2: Daha fazla bilgi
  - H2: Vaka çalışması: topluluk iş akışları
  - H2: İlgili içerikler

## tools/loop-detection.md

- Rota: /tools/loop-detection
- Başlıklar:
  - H2: Bunun var olma nedeni
  - H2: Yapılandırma bloğu
  - H3: Alan davranışı
  - H2: Önerilen kurulum
  - H2: Compaction sonrası koruma
  - H2: Günlükler ve beklenen davranış
  - H2: İlgili içerikler

## tools/media-overview.md

- Rota: /tools/media-overview
- Başlıklar:
  - H2: Yetenekler
  - H2: Sağlayıcı yetenek matrisi
  - H2: Eşzamansız ve eşzamanlı
  - H2: Konuşmayı metne dönüştürme ve Sesli Arama
  - H2: Sağlayıcı eşlemeleri (tedarikçilerin yüzeylere nasıl ayrıldığı)
  - H2: İlgili içerikler

## tools/minimax-search.md

- Rota: /tools/minimax-search
- Başlıklar:
  - H2: Token Plan kimlik bilgisi edinme
  - H2: Yapılandırma
  - H2: Bölge seçimi
  - H2: Desteklenen parametreler
  - H2: İlgili içerikler

## tools/multi-agent-sandbox-tools.md

- Rota: /tools/multi-agent-sandbox-tools
- Başlıklar:
  - H2: Yapılandırma örnekleri
  - H2: Yapılandırma önceliği
  - H3: Korumalı alan yapılandırması
  - H3: Araç kısıtlamaları
  - H2: Tek ajandan geçiş
  - H2: Araç kısıtlaması örnekleri
  - H2: Yaygın tuzak: "non-main"
  - H2: Test
  - H2: Sorun giderme
  - H2: İlgili içerikler

## tools/music-generation.md

- Rota: /tools/music-generation
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Desteklenen sağlayıcılar
  - H3: Yetenek matrisi
  - H2: Araç parametreleri
  - H2: Eşzamansız davranış
  - H3: Görev yaşam döngüsü
  - H2: Yapılandırma
  - H3: Model seçimi
  - H3: Sağlayıcı seçim sırası
  - H2: Sağlayıcı notları
  - H2: Doğru yolu seçme
  - H2: Sağlayıcı yetenek modları
  - H2: Canlı testler
  - H2: İlgili içerikler

## tools/ollama-search.md

- Rota: /tools/ollama-search
- Başlıklar:
  - H2: Kurulum
  - H2: Yapılandırma
  - H2: Kimlik doğrulama ve istek yönlendirme
  - H2: İlgili içerikler

## tools/parallel-search.md

- Rota: /tools/parallel-search
- Başlıklar:
  - H2: Plugin'i yükleme
  - H2: API anahtarı (ücretli sağlayıcı)
  - H2: Yapılandırma
  - H2: Temel URL'yi geçersiz kılma
  - H2: Araç parametreleri
  - H2: Notlar
  - H2: İlgili içerikler

## tools/pdf.md

- Rota: /tools/pdf
- Başlıklar:
  - H2: Kullanılabilirlik
  - H2: Girdi referansı
  - H2: Desteklenen PDF referansları
  - H2: Yürütme modları
  - H3: Yerel sağlayıcı modu
  - H3: Ayıklama geri dönüş modu
  - H2: Yapılandırma
  - H2: Çıktı ayrıntıları
  - H2: Hata davranışı
  - H2: Örnekler
  - H2: İlgili içerikler

## tools/permission-modes.md

- Rota: /tools/permission-modes
- Başlıklar:
  - H2: Önerilen varsayılan
  - H2: OpenClaw ana makine yürütme modları
  - H2: Codex Guardian eşlemesi
  - H2: ACPX çalıştırma sistemi izinleri
  - H2: Mod seçme
  - H2: İlgili içerikler

## tools/perplexity-search.md

- Rota: /tools/perplexity-search
- Başlıklar:
  - H2: Plugin'i yükleme
  - H2: Perplexity API anahtarı edinme
  - H2: OpenRouter uyumluluğu
  - H2: Yapılandırma örnekleri
  - H3: Yerel Perplexity Search API
  - H3: OpenRouter / Sonar uyumluluğu
  - H2: Anahtarın ayarlanacağı yer
  - H2: Araç parametreleri
  - H3: Alan adı filtresi kuralları
  - H2: Notlar
  - H2: İlgili içerikler

## tools/plugin.md

- Rota: /tools/plugin
- Başlıklar:
  - H2: Gereksinimler
  - H2: Hızlı başlangıç
  - H2: Yapılandırma
  - H3: Yükleme kaynağı seçme
  - H3: Operatör yükleme politikası
  - H3: Plugin politikasını yapılandırma
  - H2: Plugin biçimlerini anlama
  - H2: Plugin kancaları
  - H2: Etkin Gateway'i doğrulama
  - H2: Sorun giderme
  - H3: Engellenen Plugin yolu sahipliği
  - H3: Yavaş Plugin aracı kurulumu
  - H2: İlgili içerikler

## tools/reactions.md

- Rota: /tools/reactions
- Başlıklar:
  - H2: Nasıl çalışır
  - H2: Kanal davranışı
  - H2: Tepki düzeyi
  - H2: İlgili içerikler

## tools/screen.md

- Rota: /tools/screen
- Başlıklar:
  - H2: Eylemler
  - H2: Yönlendirme ve güvenlik
  - H2: İlgili içerikler

## tools/searxng-search.md

- Rota: /tools/searxng-search
- Başlıklar:
  - H2: Kurulum
  - H2: Yapılandırma
  - H2: Ortam değişkeni
  - H2: Plugin yapılandırma referansı
  - H2: Notlar
  - H2: İlgili içerikler

## tools/self-learning.md

- Rota: /tools/self-learning
- Başlıklar:
  - H2: Kendi kendine öğrenmeyi etkinleştirme
  - H2: Geçmiş oturumları elle inceleme
  - H2: OpenClaw'ın öğrenebilecekleri
  - H2: Deneyim incelemesinin çalıştığı zaman
  - H2: İnceleyiciye iletilenler
  - H2: Öneri güvenliği
  - H2: Öğrenilen önerileri inceleme
  - H2: Yapılandırma
  - H2: Sorun giderme
  - H3: Uzun bir turdan sonra öneri görünmüyor
  - H3: Doctor, Workshop aracının gizli olduğunu bildiriyor
  - H3: Çok fazla düşük değerli öneri görünüyor
  - H2: İlgili içerikler

## tools/show-widget.md

- Rota: /tools/show-widget
- Başlıklar:
  - H2: Widget'lar nasıl çalışır
  - H2: Tasarım sistemi
  - H2: Aracı kullanma
  - H2: Etkileşimli widget'lar
  - H2: Pano yetenekleri
  - H2: Güvenlik ve depolama
  - H2: İlgili içerikler

## tools/skill-workshop.md

- Rota: /tools/skill-workshop
- Başlıklar:
  - H2: Nasıl çalışır
  - H2: Yaşam döngüsü
  - H2: Yaşam döngüsü düzenlemesi
  - H2: Sohbet
  - H3: Son çalışmalardan öğrenme
  - H2: CLI
  - H2: Öneri içeriği
  - H2: Destek dosyaları
  - H2: Ajan aracı
  - H2: Önerilen beceriler
  - H3: Geçmiş oturumları tarama
  - H2: Onay ve özerklik
  - H2: Gateway yöntemleri
  - H2: Depolama
  - H2: Sınırlar
  - H2: Sorun giderme
  - H3: Araç politikası tanılaması
  - H2: İlgili içerikler

## tools/skills-config.md

- Rota: /tools/skills-config
- Başlıklar:
  - H2: Yükleme (skills.load)
  - H2: Yükleme (skills.install)
  - H2: Operatör Yükleme Politikası (security.installPolicy)
  - H2: Birlikte gelen beceriler için izin listesi
  - H2: Beceri başına girdiler (skills.entries)
  - H2: Ajan izin listeleri (agents)
  - H2: Workshop (skills.workshop)
  - H2: Sembolik bağlantılı beceri kökleri
  - H2: Korumalı alandaki beceriler ve ortam değişkenleri
  - H2: Yükleme sırası hatırlatması
  - H2: İlgili içerikler

## tools/skills.md

- Rota: /tools/skills
- Başlıklar:
  - H2: Yükleme sırası
  - H2: Node üzerinde barındırılan beceriler
  - H2: Ajan başına ve paylaşılan beceriler
  - H2: Ajan izin listeleri
  - H2: Plugin'ler ve beceriler
  - H2: Beceri Workshop'u
  - H2: ClawHub'dan yükleme
  - H2: Güvenlik
  - H2: SKILL.md biçimi
  - H3: İsteğe bağlı ön bilgi anahtarları
  - H2: Geçit denetimi
  - H3: Yükleyici belirtimleri
  - H2: Yapılandırmayı geçersiz kılma seçenekleri
  - H2: Ortam ekleme
  - H2: Anlık görüntüler ve yenileme
  - H2: Token etkisi
  - H2: İlgili içerikler

## tools/slash-commands.md

- Rota: /tools/slash-commands
- Başlıklar:
  - H2: Üç komut türü
  - H2: Yapılandırma
  - H2: Komut listesi
  - H3: Çekirdek komutları
  - H3: Dock komutları
  - H3: Birlikte gelen Plugin komutları
  - H3: Beceri komutları
  - H2: /tools: ajanın şu anda kullanabilecekleri
  - H2: /model: model seçimi
  - H2: /config: disk üzerindeki yapılandırmaya yazma
  - H2: /mcp: MCP sunucusu yapılandırması
  - H2: /debug: yalnızca çalışma zamanına yönelik geçersiz kılmalar
  - H2: /plugins: Plugin yönetimi
  - H2: /trace: Plugin izleme çıktısı
  - H2: /btw: yan sorular
  - H2: Yüzey notları
  - H2: Sağlayıcı kullanımı ve durumu
  - H2: İlgili içerikler

## tools/steer.md

- Rota: /tools/steer
- Başlıklar:
  - H2: Geçerli oturum
  - H2: Yönlendirme ve kuyruk
  - H2: Alt ajanlar
  - H2: ACP oturumları
  - H2: İlgili içerikler

## tools/subagents.md

- Rota: /tools/subagents
- Başlıklar:
  - H2: Eğik çizgi komutu
  - H3: İş parçacığı bağlama denetimleri
  - H3: Oluşturma davranışı
  - H2: Bağlam modları
  - H2: Araç: `sessions_spawn`
  - H3: Yetkilendirme istemi modu
  - H3: Araç parametreleri
  - H3: Görev adları ve hedefleme
  - H2: Araç: `sessions_yield`
  - H2: Araç: alt ajanlar
  - H2: İş parçacığına bağlı oturumlar
  - H3: İş parçacığını destekleyen kanallar
  - H3: Hızlı akış
  - H3: Manuel denetimler
  - H3: Yapılandırma anahtarları
  - H3: İzin verilenler listesi
  - H3: Keşif
  - H3: Otomatik arşivleme
  - H2: İç içe alt ajanlar
  - H3: Derinlik düzeyleri
  - H3: Duyuru zinciri
  - H3: Derinliğe göre araç politikası
  - H3: Ajan başına oluşturma sınırı
  - H3: Basamaklı durdurma
  - H2: Kimlik doğrulama
  - H2: Duyuru
  - H3: Duyuru bağlamı
  - H3: İstatistik satırı
  - H3: Neden `sessions_history` tercih edilmeli
  - H2: Araç politikası
  - H3: Yapılandırma aracılığıyla geçersiz kılma
  - H2: Eşzamanlılık
  - H2: Canlılık ve kurtarma
  - H2: Durdurma
  - H2: Sınırlamalar
  - H2: İlgili

## tools/swarm.md

- Rota: /tools/swarm
- Başlıklar:
  - H2: Swarm'ı etkinleştirme
  - H2: Gereksinimler
  - H2: Swarm betiği yazma
  - H3: Yapılandırılmış sonuçlarla paralel olarak dağıtma
  - H3: Karar kapısında döngü oluşturma
  - H3: İlk tamamlanan alt öğeyi işleme
  - H2: Toplayıcı alt öğelerin davranışı
  - H3: Alt öğeler yapraktır
  - H2: Bir Swarm'ı gözlemleme
  - H2: Swarm'ı diğer çalıştırma ortamlarından kullanma
  - H2: Sınırlar ve yol haritası
  - H2: İlgili

## tools/tavily.md

- Rota: /tools/tavily
- Başlıklar:
  - H2: Başlarken
  - H2: Araç başvurusu
  - H3: `tavily_search`
  - H3: `tavily_extract`
  - H2: Doğru aracı seçme
  - H2: Gelişmiş yapılandırma
  - H2: İlgili

## tools/thinking.md

- Rota: /tools/thinking
- Başlıklar:
  - H2: İşlevi
  - H2: Çözümleme sırası
  - H2: Oturum varsayılanını ayarlama
  - H2: Ajana göre uygulama
  - H2: Hızlı mod (/fast)
  - H2: Ayrıntılı direktifler (/verbose veya /v)
  - H2: Plugin izleme direktifleri (/trace)
  - H2: Akıl yürütme görünürlüğü (/reasoning)
  - H2: İlgili
  - H2: Heartbeat'ler
  - H2: Web sohbeti kullanıcı arayüzü
  - H2: Sağlayıcı profilleri

## tools/tokenjuice.md

- Rota: /tools/tokenjuice
- Başlıklar:
  - H2: Plugin'i etkinleştirme
  - H2: Tokenjuice'un değiştirdikleri
  - H2: Çalıştığını doğrulama
  - H2: Plugin'i devre dışı bırakma
  - H2: İlgili

## tools/tool-search.md

- Rota: /tools/tool-search
- Başlıklar:
  - H2: Bir turun çalışma biçimi
  - H2: Modlar
  - H2: Bunun var olma nedeni
  - H2: API
  - H2: Çalışma zamanı sınırı
  - H2: Yapılandırma
  - H2: İstem ve telemetri
  - H2: Uçtan uca doğrulama
  - H2: Hata davranışı
  - H2: İlgili

## tools/trajectory.md

- Rota: /tools/trajectory
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Erişim
  - H2: Kaydedilenler
  - H2: Paket dosyaları
  - H2: Yakalama depolama alanı
  - H2: Yakalamayı devre dışı bırakma
  - H2: Boşaltma zaman aşımını ayarlama
  - H2: Gizlilik ve sınırlar
  - H2: Sorun giderme
  - H2: İlgili

## tools/tts.md

- Rota: /tools/tts
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Desteklenen sağlayıcılar
  - H2: Yapılandırma
  - H3: Ajan başına ses geçersiz kılmaları
  - H2: Kişilikler
  - H3: Minimal kişilik
  - H3: Tam kişilik (sağlayıcıya özgü biçimlendirme)
  - H3: Kişilik çözümlemesi
  - H3: Özel kişilik biçimlendirmesi
  - H3: Geri dönüş politikası
  - H2: Model odaklı direktifler
  - H2: Eğik çizgi komutları
  - H2: Kullanıcı başına tercihler
  - H2: Çıktı biçimleri
  - H2: Otomatik TTS davranışı
  - H2: Alan başvurusu
  - H2: Ajan aracı
  - H2: Gateway RPC
  - H2: Hizmet bağlantıları
  - H2: İlgili

## tools/video-generation.md

- Rota: /tools/video-generation
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Eşzamansız oluşturmanın çalışma biçimi
  - H3: Görev yaşam döngüsü
  - H2: Desteklenen sağlayıcılar
  - H3: Yetenek matrisi
  - H2: Araç parametreleri
  - H3: Zorunlu
  - H3: İçerik girdileri
  - H3: Stil denetimleri
  - H3: Gelişmiş
  - H4: Geri dönüş ve türü belirlenmiş seçenekler
  - H2: Eylemler
  - H2: Model seçimi
  - H2: Sağlayıcı notları
  - H2: Sağlayıcı yetenek modları
  - H2: Canlı testler
  - H2: Yapılandırma
  - H2: İlgili

## tools/web-fetch.md

- Rota: /tools/web-fetch
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Araç parametreleri
  - H2: Sonuç
  - H2: Çalışma biçimi
  - H2: İlerleme güncellemeleri
  - H2: Yapılandırma
  - H2: Firecrawl geri dönüşü
  - H2: Güvenilir ortam vekil sunucusu
  - H2: Sınırlar ve güvenlik
  - H2: Araç profilleri
  - H2: İlgili

## tools/web.md

- Rota: /tools/web
- Başlıklar:
  - H2: Hızlı başlangıç
  - H2: Sağlayıcı seçme
  - H3: Sağlayıcı karşılaştırması
  - H2: Sonuç biçimi
  - H2: Otomatik algılama
  - H2: Yerel OpenAI web araması
  - H2: Yerel Codex web araması
  - H2: Ağ güvenliği
  - H2: Yapılandırma
  - H3: API anahtarlarını depolama
  - H2: Araç parametreleri
  - H2: xsearch
  - H3: xsearch yapılandırması
  - H3: xsearch parametreleri
  - H3: xsearch örneği
  - H2: Örnekler
  - H2: Araç profilleri
  - H2: İlgili

## tts.md

- Rota: /tts
- Başlıklar:
  - H2: İlgili

## vps.md

- Rota: /vps
- Başlıklar:
  - H2: Sağlayıcı seçme
  - H2: Bulut kurulumlarının çalışma biçimi
  - H2: Önce yönetici erişimini güçlendirme
  - H2: VPS üzerinde paylaşılan şirket ajanı
  - H2: VPS ile Node'ları kullanma
  - H2: Küçük sanal makineler ve ARM ana makineleri için başlangıç ayarlaması
  - H3: systemd ayarlama denetim listesi (isteğe bağlı)
  - H2: İlgili

## web/control-ui.md

- Rota: /web/control-ui
- Başlıklar:
  - H2: Hızlı açma (yerel)
  - H2: Cihaz eşleştirme (ilk bağlantı)
  - H2: Mobil cihaz eşleştirme
  - H2: Kişisel kimlik (tarayıcıya yerel)
  - H2: Çalışma zamanı yapılandırma uç noktası
  - H2: Gateway ana makine durumu
  - H2: Dil desteği
  - H2: Görünüm temaları
  - H2: OpenClaw sistem bakımı
  - H2: Plugin'leri yönetme
  - H2: Uygulamalar ve uzantılar
  - H2: Kenar çubuğu gezinmesi
  - H2: Yeni oturum sayfası
  - H2: Bugün yapabildikleri
  - H2: Asistan belleğini içe aktarma
  - H2: MCP sayfası
  - H2: Etkinlik sekmesi
  - H2: Operatör terminali
  - H2: Tarayıcı paneli
  - H2: Sohbet davranışı
  - H2: Bağlantı kaybı ve yeniden bağlanma
  - H2: PWA kurulumu ve web anlık bildirimleri
  - H2: Barındırılan yerleştirmeler
  - H2: Sohbet dökümü düzeni
  - H2: Sohbet mesajı genişliği
  - H2: Tailnet erişimi (önerilen)
  - H2: Güvenli olmayan HTTP
  - H2: İçerik güvenliği politikası
  - H2: Avatar rotası kimlik doğrulaması
  - H2: Asistan medya rotası kimlik doğrulaması
  - H2: Onay bağlantıları
  - H2: Boş Control UI sayfası
  - H2: Hata ayıklama/test: geliştirme sunucusu + uzak Gateway
  - H2: İlgili

## web/dashboard-architecture.md

- Rota: /web/dashboard-architecture
- Başlıklar:
  - H2: Vizyon
  - H2: Kavramlar
  - H2: Kullanıcı deneyimi akışları
  - H2: Etkileşim katmanları
  - H2: Pencere öğesi modeli ve barındırma
  - H3: Pencere öğeleri içerik barındırır; MCP uygulamaları bir içerik türüdür
  - H3: Plugin yetenek bildirimleri
  - H3: Modellenmiş artık: WebRTC veri kanalları
  - H3: Döküm görünümü: tek pencere öğesi kartı
  - H3: Sunucu kaynaklı pencere öğeleri (sabitlenmiş MCP uygulamaları)
  - H3: WorkBoard entegrasyonu
  - H2: Düzen: akışkan ızgara
  - H2: Veri modeli (ajan başına veritabanı)
  - H2: Protokol yüzeyi
  - H2: Ajan araçları
  - H2: Bunun yerini aldıkları
  - H2: Hedef dışı olanlar (bu program)
  - H2: Uygulama planı

## web/dashboard.md

- Rota: /web/dashboard
- Başlıklar:
  - H2: Hızlı yol (önerilen)
  - H2: Kimlik doğrulama temelleri (yerel ve uzak)
  - H2: Telegram'da açma
  - H2: "unauthorized" / 1008 görürseniz
  - H2: İlgili

## web/dashboards.md

- Rota: /web/dashboards
- Başlıklar:
  - H2: İsteyerek bir pano oluşturma
  - H2: Pano
  - H2: Pencere öğelerinin yapmasına izin verilenler
  - H2: Panodaki MCP uygulamaları
  - H2: Bilinmesi gerekenler

## web/index.md

- Rota: /web
- Başlıklar:
  - H2: Yapılandırma (varsayılan olarak açık)
  - H2: Webhook'lar
  - H2: Yönetici HTTP RPC
  - H2: Tailscale erişimi
  - H2: Güvenlik notları
  - H2: Kullanıcı arayüzünü oluşturma

## web/lobster.md

- Rota: /web/lobster
- Başlıklar:
  - H2: Bakılan şey
  - H2: Göründüğü durumlar
  - H2: Yapılabilecekler
  - H2: Ziyaretleri kapatma (veya yeniden açma)
  - H2: Lobsterdex
  - H2: Saha notları
  - H2: Gizlilik

## web/tui.md

- Rota: /web/tui
- Başlıklar:
  - H2: Hızlı başlangıç
  - H3: Gateway modu
  - H3: Yerel mod
  - H2: Görecekleriniz
  - H2: Zihinsel model: aracılar + oturumlar
  - H2: Gönderme + teslim
  - H2: Seçiciler + katmanlar
  - H2: Klavye kısayolları
  - H2: Eğik çizgi komutları
  - H2: Yerel kabuk komutları
  - H2: OpenClaw kurulum ve onarım yardımcısı
  - H2: Araç çıktısı
  - H2: Terminal renkleri
  - H2: Geçmiş + akış
  - H2: Bağlantı ayrıntıları
  - H2: Seçenekler
  - H2: Sorun giderme
  - H2: Bağlantı sorunlarını giderme
  - H2: İlgili içerikler

## web/webchat.md

- Rota: /web/webchat
- Başlıklar:
  - H2: Nedir?
  - H2: Hızlı başlangıç
  - H2: Nasıl çalışır?
  - H3: Transkript ve teslim modeli
  - H2: Denetim kullanıcı arayüzü aracı araçları paneli
  - H2: Uzaktan kullanım
  - H2: Yapılandırma referansı (WebChat)
  - H2: İlgili içerikler
