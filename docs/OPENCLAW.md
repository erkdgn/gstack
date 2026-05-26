# gstack x OpenClaw Entegrasyonu

gstack, OpenClaw ile taşınmış bir kod tabanı olarak değil, bir metodoloji kaynağı olarak
entegre olur. OpenClaw'ın ACP çalışma zamanı, Claude Code oturumlarını yerel olarak
başlatır. gstack, bu oturumları daha iyi kılan planlama disiplinini ve metodolojiyi sağlar.

Bu, komut istemi metni olarak kodlanmış hafif bir protokoldür. Artalan süreci yok. JSON-RPC
yok. Uyumluluk matrisleri yok. Komut istemi köprüdür.

## Mimari

```
  OpenClaw                               gstack deposu
  ─────────────────────                    ──────────────
  Orchestratör: mesajlaşma,                 Metodoloji + planlama için
  takvim, bellek, EA                         gerçeklik kaynağı
       │                                        │
       ├── Yerel yetenekler (konuşmacı)          ├── gen-skill-docs ardışık düzeni
       │   office-hours, ceo-review,            │   aracılığıyla yerel yetenekler üretir
       │   investigate, retro                   │
       │                                        ├── gstack-lite üretir
       ├── sessions_spawn(runtime: "acp")       │   (planlama disiplini)
       │       │                                │
       │       └── Claude Code                  ├── gstack-full üretir
       │           └── gstack şu konumda kurulu │   (tamamlanmış ardışık düzen)
       │               ~/.claude/skills/gstack  │
       │                                        └── docs/OPENCLAW.md (bu dosya)
       └── Gönderim yönlendirmesi (AGENTS.md)
```

## Gönderim Yönlendirmesi

OpenClaw, başlatma zamanında hangi gstack destek katmanını kullanacağına karar verir:

| Katman | Ne zaman | Komut istemi öneki |
|------|------|---------------|
| **Basit** | Tek dosya düzenlemeler, yazım hataları, yapılandırma değişiklikleri | gstack bağlamı enjekte edilmez |
| **Orta** | Çok dosyalı özellikler, yeniden düzenlemeler | gstack-lite CLAUDE.md eklenir |
| **Ağır** | Belirli gstack yeteneği gerekli | "gstack'i yükle. /X çalıştır" |
| **Tam** | Tam özellikler, hedefler, projeler | gstack-full ardışık düzen eklenir |
| **Plan** | "Bana bir Claude Code projesi planlamama yardım et" | gstack-plan ardışık düzen eklenir |

### Karar sezgisi

- <10 satır kodda yapılabilir mi? -> **Basit**
- Birden fazla dosyaya dokunuyor ama yaklaşım bariz mi? -> **Orta**
- Kullanıcı belirli bir yetenek adlandırıyor mu (/cso, /review, /qa)? -> **Ağır**
- Bir özellik, proje veya hedef mi (görev değil)? -> **Tam**
- Kullanıcı henüz uygulamadan Claude Code için bir şey PLANLAMAK mı istiyor? -> **Plan**

### Gönderim yönlendirme kılavuzu (AGENTS.md için)

Tamamı kopyalanmaya hazır bölüm `openclaw/agents-gstack-section.md` dosyasında yer alır.
OpenClaw AGENTS.md dosyanıza kopyalayın.

Temel davranış kuralları (bunlar gönderim katmanlarının ÜSTÜNE gelir):

1. **Her zaman başlat, asla yönlendirme.** Kullanıcı HERHANGİ bir gstack yeteneği kullanmayı
   istediğinde, HER ZAMAN bir Claude Code oturumu başlatın. Kullanıcıya asla Claude Code'u
   açmasını söylemeyin.
2. **Depoyu çözün.** Kullanıcı bir depo adlandırırsa, çalışma dizinini ayarlayın. Bilinmiyorsa,
   hangi depo olduğunu sorun.
3. **Otomatik plan baştan sona çalışır.** Başlatın, tam ardışık düzeni çalıştırın, sohbette
   rapor edin. Kullanıcının Telegram'dan ayrılması gerekmemeli.

### CLAUDE.md çakışma yönetimi

Zaten bir CLAUDE.md dosyası olan bir depoda Claude Code başlatırken, gstack-lite/full'i
yeni bir bölüm olarak EKLEYİN. Deponun mevcut talimatlarını değiştirmeyin.

## gstack OpenClaw için ne üretir

Tüm yapıtlar `openclaw/` dizininde bulunur ve `bun run gen:skill-docs --host openclaw`
tarafından üretilir:

### gstack-lite (Orta katman)
`openclaw/gstack-lite-CLAUDE.md` — ~15 satırlık planlama disiplini:
1. Değiştirmeden önce her dosyayı oku
2. 5 satırlık bir plan yaz: ne, neden, hangi dosyalar, test durumu, risk
3. Karar ilkellerini kullanarak belirsizliği çöz
4. Tamamlandırmadan önce kendi kendini incele
5. Tamamlanma raporu: ne gönderildi, verilen kararlar, belirsiz olan her şey

A/B test edildi: 2 kat zaman, anlamına gelen şekilde daha iyi çıktı.

### gstack-full (Tam katman)
`openclaw/gstack-full-CLAUDE.md` — mevcut gstack yeteneklerini zincirler:
1. CLAUDE.md'yi oku ve projeyi anla
2. /autoplan çalıştır (CEO + mühendislik + tasarım incelemesi)
3. Onaylanan planı uygula
4. PR oluşturmak için /ship çalıştır
5. PR URL'si ve kararlarla geri rapor et

### gstack-plan (Plan katmanı)
`openclaw/gstack-plan-CLAUDE.md` — tam inceleme geçidi, uygulama yok:
1. Bir tasarım belgesi üretmek için /office-hours çalıştır
2. /autoplan çalıştır (CEO + mühendislik + tasarım + DX incelemeleri + codex sertlik)
3. İncelenen planı `plans/<proje-kısa-adı>-plan-<tarih>.md` konumuna kaydet
4. Geri rapor et: plan yolu, özet, kilit kararlar, önerilen sonraki adım

Orkestratör, plan bağlantısını kendi bellek deposuna (brain deposu, bilgi tabanı veya
AGENTS.md'de yapılandırılmış her neyse) kalıcı hale getirir. Kullanıcı inşa etmeye
hazır olduğunda, kaydedilen plana referans veren bir TAM oturum başlatır.

### Yerel metodoloji yetenekleri
ClawHub'da yayınlanır. `clawhub install` ile kurun:
- `gstack-openclaw-office-hours` — Ürün sorgulaması (6 zorlayıcı soru)
- `gstack-openclaw-ceo-review` — Stratejik meydan okuma (10 bölümlük inceleme, 4 mod)
- `gstack-openclaw-investigate` — Operasyonel hata ayıklama (4 aşamalı metodoloji)
- `gstack-openclaw-retro` — Operasyonel geriye bakış (haftalık inceleme)

Kaynak, gstack deposundaki `openclaw/skills/` dizininde yer alır. Bunlar, OpenClaw'ın
konuşmacı bağlamı için gstack metodolojisinin el ile hazırlanmış uyarlamalarıdır.
gstack altyapısı yok (browse yok, telemetri yok, önsöz yok).

## Başlatılan oturum algılama

Claude Code, OpenClaw tarafından başlatılan bir oturum içinde çalıştığında, `OPENCLAW_SESSION`
ortam değişkeni ayarlanmalıdır. gstack bunu algılar ve ayarlar:
- Etkileşimli komut istemlerini atlar (önerilen seçenekleri otomatik olarak seçer)
- Yükseltme denetimlerini ve telemetri komut istemlerini atlar
- Görev tamamlama ve düzyazı raporlamaya odaklanır

Ortam değişkenini sessions_spawn içinde ayarlayın: `env: { OPENCLAW_SESSION: "1" }`

## Kurulum

OpenClaw kullanıcıları için: OpenClaw aracınıza "openclaw için gstack kur" deyin.

Aracı şunları yapmalıdır:
1. gstack-lite CLAUDE.md dosyasını kodlama oturumu şablonlarına kurun
2. 4 yerel metodoloji yeteneğini kurun
3. Gönderim yönlendirmesini AGENTS.md dosyasına ekleyin
4. Bir test başlatmasıyla doğrulayın

gstack geliştiricileri için: `./setup --host openclaw` bu belgelendirmeyi çıktılar.
Gerçek yapıtlar `bun run gen:skill-docs --host openclaw` tarafından üretilir.

## Yapmadığımız şeyler

- Gönderim artalan süreci yok (ACP oturum başlatmayı yönetir)
- Clawvisor geçişi yok (güvenlik katmanı gerekmez)
- Çift yönlü öğrenmeler köprüsü yok (brain deposu bilgi deposudur)
- JSON şemaları veya protokol sürümleme yok
- gstack'ten SOUL.md yok (OpenClaw'ın kendi SOUL.md'si var)
- Tam yetenek taşıma yok (kodlama yetenekleri Claude Code'a özgü kalır)