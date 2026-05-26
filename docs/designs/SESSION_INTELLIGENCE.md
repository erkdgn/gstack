# Oturum Zeka Katmanı

## Sorun

Claude Code'un bağlam penceresi geçici. Her oturum taze başlıyor. Otomatik sıkıştırma ~167K token'da tetiklendiğinde, genel bir özet saklar ama dosya okumalarını, akıl yürütme zincirlerini ve ara kararları yok eder.

gstack zaten diskte hayatta kalan değerli yapıtlar üretiyor: CEO planları, mühendislik incelemeleri, tasarım incelemeleri, QA raporları, öğrenmeler. Bu dosyalar mevcut çalışmayı şekillendiren kararları, kısıtlamaları ve bağlamı içeriyor. Ama Claude bunların varlığını bilmiyor. Sıkıştırmadan sonra, her kararı bilgilendiren planlar ve incelemeler sessizce bağlamdan kayboluyor.

Ekosistem bunun üzerinde çalışıyor. claude-mem (9K+ yıldız) araç kullanımını yakalar ve gelecek oturumlara bağlam enjekte eder. Claude HUD gerçek zamanlı ajans durumu gösterir. Anthropic'in kendi `claude-progress.txt` kalıbı, ajansların her oturumun başında okuduğu bir ilerleme dosyası kullanır.

Kimse **yetenek tarafından üretilen yapıtların** sıkıştırmayı yaşatması için özel sorunu çözmüyor. Çünkü başka kimsenin gstack'in yapıt mimarisi yok.

## İçgörü

gstack zaten `~/.gstack/projects/$SLUG/` konumuna yapılandırılmış yapıtlar yazıyor:
- CEO planları: `ceo-plans/`
- Tasarım incelemeleri: `design-reviews/`
- Mühendislik incelemeleri: `eng-reviews/`
- Öğrenmeler: `learnings.jsonl`
- Yetenek kullanımı: `../analytics/skill-usage.jsonl`

Eksik parça depolama değil. Farkındalık. Preamble'ın aracıya söylemesi gerekiyor: "Bu dosyalar var. Zaten verdiğiniz kararları içeriyor. Sıkıştırmadan sonra, onları yeniden okuyun."

## Mimari

```
                   ┌─────────────────────────────────────┐
                   │        Claude Bağlam Penceresi        │
                   │   (geçici, ~167K token sınırı)       │
                   │                                      │
                   │   Sıkıştırma tetiklendi ──► sadece özet   │
                   └──────────────┬──────────────────────┘
                                  │
                          başlangıçta / sıkıştırmadan sonra okur
                                  │
                   ┌──────────────▼──────────────────────┐
                   │    ~/.gstack/projects/$SLUG/         │
                   │    (kalıcı, her şeyi yaşatır) │
                   │                                      │
                   │  ceo-plans/         ← /plan-ceo-review
                   │  eng-reviews/       ← /plan-eng-review
                   │  design-reviews/    ← /plan-design-review
                   │  checkpoints/       ← /checkpoint (yeni)
                   │  timeline.jsonl     ← her yetenek (yeni)
                   │  learnings.jsonl    ← /learn
                   └─────────────────────────────────────┘
                                  │
                          haftalık olarak özetlenir
                                  │
                   ┌──────────────▼──────────────────────┐
                   │           /retro                      │
                   │  Zaman Çizelgesi: 3 /review, 2 /ship, ...   │
                   │  Sağlık trendleri: compile 8/10 (↑2)     │
                   │  Uygulanan öğrenmeler: bu hafta 4       │
                   └─────────────────────────────────────┘
```

## Özellikler

### Katman 1: Bağlam Kurtarma (preamble, tüm yetenekler)
Preamble'da ~10 satır düzyazı. Sıkıştırma veya bağlam bozulmasından sonra, aracı `~/.gstack/projects/$SLUG/` konumundaki son planları, incelemeleri ve kontrol noktalarını kontrol eder. Dizini listeler, en son dosyayı okur.

Maliyet: sıfıra yakın. Fayda: her yeteneğin planları/incelemeleri sıkıştırmayı yaşatır.

### Katman 2: Oturum Zaman Çizelgesi (preamble, tüm yetenekler)
Her yetenek `timeline.jsonl` dosyasına tek satırlık bir JSONL girdisi ekler: zaman damgası, yetenek adı, dal, ana sonuç. `/retro` bunu işler.

Projenin AI destekli çalışma geçmişini görünür kılar. "Bu hafta: 3 /review, 2 /ship, 1 /investigate, feature-auth ve fix-billing dallarında."

### Katman 3: Oturumlar Arası Enjeksiyon (preamble, tüm yetenekler)
Yeni bir oturum, yakın yapıtları olan bir dalda başladığında, preamble tek satırlık bir mesaj yazdırır: "Son oturum: JWT auth uygulandı, 3/5 görev tamamlandı. Plan: ~/.gstack/projects/$SLUG/checkpoints/latest.md"

Aracı herhangi bir dosya okumadan önce nerede kaldığınızı bilir.

### Katman 4: /checkpoint (isteğe bağlı yetenek)
Çalışma durumunun manuel anlık görüntüsü: ne yapılıyor, hangi dosyalar düzenleniyor, verilen kararlar, ne kaldı. Uzaklaşmadan önce, karmaşık işlemlerden önce, çalışma alanı devirleri için veya günler sonra geri dönmeden önce kullanışlı.

### Katman 5: /health (isteğe bağlı yetenek)
Kod kalite panosu: tür kontrolü, lint, test takımı, ölü kod taraması. Bileşik 0-10 puan. Zaman içinde takip eder. `/retro` trendleri gösterir. `/ship` yapılandırılabilir eşikte geçit sağlar.

## Bileşik Etki

Her özellik bağımsız olarak kullanışlıdır. Birlikte, bileşikleşen bir şey yaratırlar:

Oturum 1: /plan-ceo-review bir plan üretir. Diske kaydedilir.
Oturum 2: Arşı planı preamble'dan sonra okur. Kararları yeniden sormaz.
Oturum 3: /checkpoint ilerlemeyi kaydeder. Zaman çizelgesi 2 /review, 1 /ship gösterir.
Oturum 4: Sıkıştırma yeniden düzenleme sırasında tetiklenir. Arşı kontrol noktasını yeniden okur.
           Anahtar kararları, türleri, kalan işi kurtarır. Devam eder.
Oturum 5: /retro haftayı özetler. Sağlık trendi: 6/10 → 8/10.
           Zaman çizelgesi 3 dalda 12 yetenek çağrısı gösterir.

Projenin AI geçmişi artık geçici değil. Kalıcıdır, bileşikleşir ve her gelecek oturumu daha akıllı yapar. Oturum zeka katmanı budur.

## Bu Ne Değildir

- Claude'un yerleşik sıkıştırmasının yerine geçmez (oturum durumunu o işler; gstack yapıtlarını biz işleriz)
- claude-mem gibi tam bir bellek sistemi değildir (SQLite üzerinden oturumlar arası belleği o işler; yapılandırılmış yetenek yapıtlarını biz işleriz)
- Bir veritabanı veya servis değildir (diskte sadece markdown dosyaları)

## Araştırma Kaynakları

- [Anthropic: Uzun süre çalışan ajanslar için etkili donanımlar](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents)
- [Anthropic: Etkili bağlam mühendisliği](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)
- [claude-mem](https://github.com/thedotmack/claude-mem)
- [Claude HUD](https://github.com/jarrodwatts/claude-hud)
- [CodeScene: Agentic AI kodlama en iyi uygulamaları](https://codescene.com/blog/agentic-ai-coding-best-practice-patterns-for-speed-with-quality)
- [Sıkıştırma sonrası kurtarma, git ile kalıcı durum (Beads)](https://dev.to/jeremy_longshore/building-post-compaction-recovery-for-ai-agent-workflows-with-beads-207l)