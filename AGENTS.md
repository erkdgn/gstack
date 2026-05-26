# gstack — AI Mühendislik Workflow'u

gstack, AI ajanlarına yazılım geliştirme için yapılandırılmış roller veren SKILL.md dosyalarından
oluşan bir koleksiyondur. Her skill bir uzmandır: CEO reviewer, mühendislik yöneticisi, tasarımcı,
QA lideri, release mühendisi, debugger ve daha fazlası.

## Mevcut skill'ler

Skill'ler `.agents/skills/` dizininde bulunur (veya Claude Code'da `~/.claude/skills/gstack/`).
İsme göre çağırın (örn. `/office-hours`).

### Plan-modu incelemeleri

| Skill | Ne yapar |
|-------|----------|
| `/office-hours` | Buradan başlayın. Kod yazmadan önce ürün fikrinizi yeniden çerçevelendirir. |
| `/plan-ceo-review` | CEO seviyesinde inceleme: istekte 10 yıldızlı ürünü bulun. |
| `/plan-eng-review` | Mimari, veri akışı, edge case'ler ve testleri kilitler. |
| `/plan-design-review` | Her tasarım boyutunu 0-10 puanlar, 10'un nasıl göründüğünü açıklar. |
| `/plan-devex-review` | DX-modu inceleme: TTHW, büyülü anlar, sürtünme noktaları, persona izleri. |
| `/plan-tune` | AskUserQuestion hassasiyetini soru bazında kendi kendine ayarlar. |
| `/autoplan` | Tek komutla CEO → tasarım → eng → DX incelemesi çalıştırır. |
| `/design-consultation` | Sıfırdan eksiksiz bir tasarım sistemi inşa eder. |

### Implementasyon + inceleme

| Skill | Ne yapar |
|-------|----------|
| `/review` | Landing öncesi PR incelemesi. CI'dan geçen ama prod'da bozulan bug'ları bulur. |
| `/codex` | OpenAI Codex üzerinden ikinci görüş. Review, challenge veya consult modları. |
| `/investigate` | Sistematik kök-neden debugging. Araştırma olmadan fix yok. |
| `/design-review` | Canlı site görsel denetimi + atomik commit'lerle fix döngüsü. |
| `/design-shotgun` | Birden fazla AI tasarım varyantı üretir, karşılaştırma panosu, iterasyon yapar. |
| `/design-html` | Prodüksiyon kalitesinde Pretext-native HTML/CSS üretir. |
| `/devex-review` | Canlı geliştirici deneyimi denetimi (gerçek akışa karşı ölçülen TTHW). |
| `/qa` | Gerçek bir tarayıcı açar, bug'ları bulur, düzeltir, yeniden doğrular. |
| `/qa-only` | /qa ile aynı metodoloji ama sadece rapor — kod değişikliği yok. |
| `/scrape` | Bir web sayfasından veri çeker. İlk çağrı prototip; kodlanmış çağrı ~200ms'de çalışır. |
| `/skillify` | En son başarılı `/scrape` akışını kalıcı bir browser-skill olarak kodlar. |

### Release + deploy

| Skill | Ne yapar |
|-------|----------|
| `/ship` | Test çalıştırır, inceler, push eder, PR açar. Workspace-aware versiyon kuyruğu. |
| `/land-and-deploy` | PR'ı merge eder, CI ve deploy'u bekler, prodüksiyon sağlığını doğrular. |
| `/canary` | Deploy sonrası browse daemon kullanarak izleme döngüsü. |
| `/landing-report` | Workspace-aware ship kuyruğu için salt-okunur dashboard. |
| `/document-release` | Az önce ship ettiğinizle eşleşmesi için tüm dokümanları günceller. |
| `/document-generate` | Koddan Diataxis dokümanları üretir (eğitim / nasıl yapılır / referans / açıklama). |
| `/setup-deploy` | Tek seferlik deploy yapılandırma tespiti (Fly.io, Render, Vercel vb.). |
| `/gstack-upgrade` | gstack'i en son versiyona günceller. |

### Operasyonel + bellek

| Skill | Ne yapar |
|-------|----------|
| `/context-save` | Çalışma bağlamını kaydeder (git durumu, kararlar, kalan iş). |
| `/context-restore` | Kaydedilmiş bir bağlamdan devam eder, Conductor workspace'leri arası bile. |
| `/learn` | gstack'in oturumlar arası öğrendiklerini yönetir. |
| `/retro` | Kişi başına döküm ve shipping streak'leri ile haftalık retrospektif. |
| `/health` | Kod kalitesi dashboard'u (type checker, linter, testler, ölü kod). |
| `/benchmark` | Performans regresyon tespiti (sayfa yükü, Core Web Vitals). |
| `/benchmark-models` | Skill'ler için çapraz-model benchmark (Claude, GPT, Gemini yan yana). |
| `/cso` | OWASP Top 10 + STRIDE güvenlik denetimi. |
| `/setup-gbrain` | Çapraz-makine oturum bellek senkronizasyonu için gbrain kurulumu. |
| `/sync-gbrain` | gbrain'i bu repo'nun koduyla güncel tutar; ajan arama rehberini CLAUDE.md'de yeniler. |

### Tarayıcı + ajan entegrasyonu

| Skill | Ne yapar |
|-------|----------|
| `/browse` | Headless tarayıcı — gerçek Chromium, gerçek tıklamalar, ~100ms/komut. |
| `/open-gstack-browser` | Kenar çubuğu + stealth ile görünür GStack Browser'ı başlatır. |
| `/setup-browser-cookies` | Kimlik doğrulamalı test için gerçek tarayıcınızdan çerezleri içe aktarır. |
| `/pair-agent` | Uzak bir AI ajanını (OpenClaw, Codex vb.) tarayıcınızla eşleştirir. |

### iOS QA — USB veya Tailscale üzerinden gerçek iPhone'ları sürün (v1.43.0.0+)

| Skill | Ne yapar |
|-------|----------|
| `/ios-qa` | USB CoreDevice tüneli + gömülü StateServer üzerinden canlı cihaz iOS QA. İsteğe bağlı olarak cihazı Tailscale üzerinden açar, böylece uzak ajanlar sürebilir. |
| `/ios-fix` | Regresyon anlık görüntüsü yakalama ile otonom iOS bug fixleyici. |
| `/ios-design-review` | Gerçek bir iPhone'da tasarımcı gözüyle QA — 10 boyutlu Apple HIG rubrikası. |
| `/ios-clean` | Kolaylık: Release build öncesi DebugBridge + #if DEBUG kablolamasını temizler. |
| `/ios-sync` | iOS debug köprüsünü en son upstream şablonlarına karşı yeniden üretir. |

Yardımcı CLI'lar (cihaza takılı Mac üzerinde çalışır):

| Komut | Ne yapar |
|-------|----------|
| `gstack-ios-qa-daemon` | Mac tarafı aracı. Varsayılan olarak loopback; `--tailnet`, yetenek katmanları ve denetim günlüklemesi ile Tailscale'e bakan bir dinleyici ekler. |
| `gstack-ios-qa-mint` | Tailnet izin listesi için owner-grant CLI (`grant`/`revoke`/`list`). |

Uçtan uca kılavuz: [docs/howto-ios-testing-with-gstack.md](docs/howto-ios-testing-with-gstack.md).

### Güvenlik + kapsam

| Skill | Ne yapar |
|-------|----------|
| `/careful` | Yıkıcı komutlar öncesi uyarır (rm -rf, DROP TABLE, force-push). |
| `/freeze` | Düzenlemeleri bir dizine kilitler. Sadece uyarı değil, sert engel. |
| `/guard` | careful + freeze'ı aynı anda aktifleştirir. |
| `/unfreeze` | Dizin düzenleme kısıtlamalarını kaldırır. |
| `/make-pdf` | Herhangi bir markdown dosyasını yayınculuk kalitesinde PDF'e dönüştürür. |

## Build komutları

```bash
bun install              # bağımlılıkları kur
bun test                 # ücretsiz testleri çalıştır (API harcaması yok)
bun run test:windows     # seçilmiş Windows-güvenli alt küme (windows-latest üzerinde çalışır)
bun run build            # dokümanları oluştur + binary'leri derle
bun run gen:skill-docs   # SKILL.md dosyalarını şablonlardan yeniden oluştur
bun run skill:check      # tüm skill'ler için sağlık dashboard'u
```

## Platform desteği

- **macOS** + **Linux**: tam test paketi desteklenir.
- **Windows**: seçilmiş Windows-güvenli alt küme, `windows-free-tests` CI job'ı ile
  `windows-latest` üzerinde çalışır. Kurulum betiği (`./setup`) bugün Git Bash veya
  MSYS gerektirir; yerel PowerShell desteği gelecekteki bir genişlemedir. `bin/gstack-paths`
  yardımcısı, durum köklerini `CLAUDE_PLUGIN_DATA` / `GSTACK_HOME` üzerinden çözer, böylece
  eklenti kurulumları her platformda çalışır.

## Temel konvansiyonlar

- SKILL.md dosyaları `.tmpl` şablonlarından **üretilir**. Çıktıyı değil, şablonu düzenleyin.
- Codex'e özel çıktıyı yeniden üretmek için `bun run gen:skill-docs --host codex` çalıştırın.
- Browse binary'si headless tarayıcı erişimi sağlar. Skill'lerde `$B <komut>` kullanın.
- Güvenlik skill'leri (careful, freeze, guard) satır içi danışmanlık metni kullanır — yıkıcı işlemlerden önce her zaman onaylayın.
- Durum yolları `bin/gstack-paths` üzerinden çözülür (`eval "$(...)"` ile kaynaklanır). `GSTACK_HOME`, `CLAUDE_PLUGIN_DATA`, `CLAUDE_PLANS_DIR`'i onurlandırır.
- `claude` CLI binary'si `browse/src/claude-bin.ts` üzerinden çözülür (`Bun.which()` + `GSTACK_CLAUDE_BIN` override). Windows'ta Claude'u WSL üzerinden çalıştırmak için `GSTACK_CLAUDE_BIN=wsl` ve `GSTACK_CLAUDE_BIN_ARGS='["claude"]'` ayarlayın.