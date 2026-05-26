# gstack geliştirme

## Komutlar

```bash
bun install          # bağımlılıkları yükle
bun test             # ücretsiz testleri çalıştır (browse + snapshot + skill doğrulama)
bun run test:evals   # ücretli eval'leri çalıştır: LLM judge + E2E (diff tabanlı, ~$4/run max)
bun run test:evals:all  # diff'den bağımsız TÜM ücretli eval'leri çalıştır
bun run test:gate    # yalnızca gate-tier testlerini çalıştır (CI varsayılanı, merge'i engeller)
bun run test:periodic  # yalnızca periodic-tier testlerini çalıştır (haftalık cron / manuel)
bun run test:e2e     # yalnızca E2E testlerini çalıştır (diff tabanlı, ~$3.85/run max)
bun run test:e2e:all # diff'den bağımsız TÜM E2E testlerini çalıştır
bun run eval:select  # mevcut diff'e göre hangi testlerin çalışacağını göster
bun run dev <cmd>    # CLI'yi dev modunda çalıştır, ör. bun run dev goto https://example.com
bun run build        # dokümanları oluştur + binary'leri derle
bun run gen:skill-docs  # SKILL.md dosyalarını şablonlardan yeniden oluştur
bun run skill:check  # tüm skill'ler için sağlık paneli
bun run dev:skill    # watch modu: değişiklikte otomatik yeniden oluşturma + doğrulama
bun run eval:list    # ~/.gstack-dev/evals/ içindeki tüm eval çalıştırmalarını listele
bun run eval:compare # iki eval çalıştırmasını karşılaştır (en yenileri otomatik seçilir)
bun run eval:summary # tüm eval çalıştırmaları genelinde istatistikleri birleştir
bun run slop          # tam slop-scan raporu (tüm dosyalar)
bun run slop:diff     # yalnızca bu branch'te değişen dosyalardaki slop bulguları
```

`test:evals` için `ANTHROPIC_API_KEY` gereklidir. Codex E2E testleri (`test/codex-e2e.test.ts`)
Codex'in kendi auth'unu `~/.codex/` yapılandırmasından kullanır; `OPENAI_API_KEY` ortam değişkenine
gerek yoktur.

**Conductor çalışma alanlarındaki ortam değişkenleri.** `GSTACK_*` ortam-shim'i (v1.39.2.0+,
`lib/conductor-env-shim.ts`), `GSTACK_ANTHROPIC_API_KEY` /
`GSTACK_OPENAI_API_KEY` değerlerini gstack'in TS binary'leri içindeki kanonik isimlerine
promote eder. gstack giriş noktaları üzerinden çalışan testler bu promote işlemini otomatik olarak devralır.
Anahtar değerini stdout'a, loglara veya shell geçmişine yazmayın. Bir testin
Agent SDK'sine geçirirken, `runAgentSdkTest`'e `env: {...}` iletmeyin; SDK'nın
auth pipeline'ı anahtarı ortam değişkeni nesnesi olarak verildiğinde aynı şekilde algılamaz
(doğrulanmış hata modu). Çağrıdan önce `process.env.ANTHROPIC_API_KEY`'i
ortama doğrudan mutate edin ve `finally` içinde eski haline getirin.

E2E testleri ilerlemeyi gerçek zamanlı olarak akışla bildirir (araç-aracı, `--output-format stream-json
--verbose` ile). Sonuçlar, önceki çalıştırmayla otomatik karşılaştırma ile birlikte
`~/.gstack-dev/evals/` dizinine kalıcı olarak yazılır.

**Diff tabanlı test seçimi:** `test:evals` ve `test:e2e`, temel branch'a karşı `git diff`
temelinde testleri otomatik seçer. Her test dosya bağımlılıklarını
`test/helpers/touchfiles.ts` içinde bildirir. Genel touchfile'larda yapılan değişiklikler
(session-runner, eval-store, touchfiles.ts'in kendisi) tüm testleri tetikler.
Tüm testleri zorlamak için `EVALS_ALL=1` veya `:all` script varyantlarını kullanın.
Hangi testlerin çalışacağını önizlemek için `eval:select` komutunu çalıştırın.

**İki katmanlı sistem:** Testler `E2E_TIERS` içinde (`test/helpers/touchfiles.ts`
dosyasında) `gate` veya `periodic` olarak sınıflandırılır. CI yalnızca gate testlerini
çalıştırır (`EVALS_TIER=gate`); periodic testleri haftalık cron ile veya manuel olarak
çalıştırılır. Filtrelemek için `EVALS_TIER=gate` veya `EVALS_TIER=periodic` kullanın.
Yeni E2E testleri eklerken, onları sınıflandırın:
1. Güvenlik guardrail'i veya deterministik işlevsel test mi? -> `gate`
2. Kalite kıyaslaması, Opus model testi veya deterministik olmayan mı? -> `periodic`
3. Harici hizmet gerektiriyor mu (Codex, Gemini)? -> `periodic`

## Test

```bash
bun test             # her commit'ten önce çalıştır — ücretsiz, <2s
bun run test:evals   # göndermeden önce çalıştır — ücretli, diff tabanlı (~$4/run max)
```

`bun test` skill doğrulaması, gen-skill-docs kalite kontrolleri ve browse
entegrasyon testlerini çalıştırır. `bun run test:evals`, LLM-judge kalite eval'lerini ve E2E
testlerini `claude -p` üzerinden çalıştırır. PR oluşturmadan önce ikisinin de geçmesi gerekir.

## Proje yapısı

```
gstack/
├── browse/          # Headless tarayıcı CLI'si (Playwright)
│   ├── src/         # CLI + sunucu + komutlar
│   │   ├── commands.ts  # Komut kayıt defteri (tek doğru kaynak)
│   │   └── snapshot.ts  # SNAPSHOT_FLAGS metadata dizisi
│   ├── test/        # Entegrasyon testleri + fixture'lar
│   └── dist/        # Derlenmiş binary
├── hosts/           # Tiplenmiş host yapılandırmaları (her AI ajanı için bir tane)
│   ├── claude.ts    # Birincil host yapılandırması
│   ├── codex.ts, factory.ts, kiro.ts  # Mevcut host'lar
│   ├── opencode.ts, slate.ts, cursor.ts, openclaw.ts  # IDE host'ları
│   ├── hermes.ts, gbrain.ts  # Ajan çalışma zamanı host'ları
│   └── index.ts     # Kayıt defteri: tümünü dışa aktarır, Host tipini türetir
├── scripts/         # Build + DX araçları
│   ├── gen-skill-docs.ts  # Şablondan SKILL.md oluşturucu (yapılandırma güdümlü)
│   ├── host-config.ts     # HostConfig arayüzü + doğrulayıcı
│   ├── host-config-export.ts  # Kurulum script'i için shell köprüsü
│   ├── host-adapters/     # Host'a özel bağdaştırıcılar (OpenClaw araç eşlemesi)
│   ├── resolvers/   # Şablo çözücü modülleri (preamble, design, review, gbrain vb.)
│   ├── skill-check.ts     # Sağlık paneli
│   └── dev-skill.ts       # Watch modu
├── test/            # Skill doğrulama + eval testleri
│   ├── helpers/     # skill-parser.ts, session-runner.ts, llm-judge.ts, eval-store.ts
│   ├── fixtures/    # Gerçek değer JSON'ları, yerleştirilmiş hata fixture'ları, eval referans değerleri
│   ├── skill-validation.test.ts  # Katman 1: statik doğrulama (ücretsiz, <1s)
│   ├── gen-skill-docs.test.ts    # Katman 1: oluşturucu kalitesi (ücretsiz, <1s)
│   ├── skill-llm-eval.test.ts   # Katman 3: LLM-as-judge (~$0.15/run)
│   └── skill-e2e-*.test.ts       # Katman 2: claude -p ile E2E (~$3.85/run, kategoriye göre ayrılmış)
├── qa-only/         # /qa-only skill'i (yalnızca raporlayan QA, düzeltme yok)
├── plan-design-review/  # /plan-design-review skill'i (yalnızca raporlayan tasarım denetimi)
├── design-review/    # /design-review skill'i (tasarım denetimi + düzeltme döngüsü)
├── ship/            # Ship iş akışı skill'i
├── review/          # PR inceleme skill'i
├── plan-ceo-review/ # /plan-ceo-review skill'i
├── plan-eng-review/ # /plan-eng-review skill'i
├── autoplan/        # /autoplan skill'i (otomatik inceleme pipeline'ı: CEO → tasarım → mühendislik)
├── benchmark/       # /benchmark skill'i (performans gerileme tespiti)
├── canary/          # /canary skill'i (deploy sonrası izleme döngüsü)
├── codex/           # /codex skill'i (OpenAI Codex CLI üzerinden çoklu-AI ikinci görüş)
├── land-and-deploy/ # /land-and-deploy skill'i (merge → deploy → canary doğrulama)
├── office-hours/    # /office-hours skill'i (YC Ofis Saatleri — startup teşhisi + yapıcı beyin fırtınası)
├── investigate/     # /investigate skill'i (sistematik kök-neden hata ayıklama)
├── retro/           # Retrospektif skill'i (/retro global çapraz-proje modunu içerir)
├── bin/             # CLI araçları (gstack-repo-mode, gstack-slug, gstack-config vb.)
├── document-release/ # /document-release skill'i (ship sonrası doküman güncellemeleri + Diataxis kapsama haritası)
├── document-generate/ # /document-generate skill'i (Diataxis doküman oluşturucu: eğitim/nasıl yapılır/referans/açıklama)
├── cso/             # /cso skill'i (OWASP Top 10 + STRIDE güvenlik denetimi)
├── design-consultation/ # /design-consultation skill'i (sıfırdan tasarım sistemi)
├── design-shotgun/  # /design-shotgun skill'i (görsel tasarım keşfi)
├── open-gstack-browser/  # /open-gstack-browser skill'i (GStack Tarayıcı'yı başlat)
├── connect-chrome/  # symlink → open-gstack-browser (geriye dönük uyumluluk)
├── design/          # Tasarım binary CLI'si (GPT Image API)
│   ├── src/         # CLI + komutlar (oluştur, varyantlar, karşılaştır, serve vb.)
│   ├── test/        # Entegrasyon testleri
│   └── dist/        # Derlenmiş binary
├── extension/       # Chrome uzantısı (yan panel + etkinlik akışı + CSS denetçisi)
├── lib/             # Paylaşılan kütüphaneler (worktree.ts)
├── docs/designs/    # Tasarım dokümanları
├── setup-deploy/    # /setup-deploy skill'i (tek seferlik deploy yapılandırması)
├── .github/         # CI iş akışları + Docker imajı
│   ├── workflows/   # evals.yml (Ubicloud üzerinde E2E), skill-docs.yml, actionlint.yml
│   └── docker/      # Dockerfile.ci (önceden hazırlanmış araç zinciri + Playwright/Chromium)
├── contrib/         # Yalnızca katkıda bulunanlar için araçlar (kullanıcılar için asla kurulmaz)
│   └── add-host/    # /gstack-contrib-add-host skill'i
├── setup            # Tek seferlik kurulum: binary oluştur + skill'leri symlink et
├── SKILL.md         # SKILL.md.tmpl'den oluşturulur (doğrudan düzenlemeyin)
├── SKILL.md.tmpl    # Şablon: bunu düzenleyin, gen:skill-docs çalıştırın
├── ETHOS.md         # Yapıcı felsefesi (Boil the Lake, Search Before Building)
└── package.json     # Browse için build script'leri
```

## SKILL.md iş akışı

SKILL.md dosyaları `.tmpl` şablonlarından **oluşturulur**. Dokümanları güncellemek için:

1. `.tmpl` dosyasını düzenleyin (örn. `SKILL.md.tmpl` veya `browse/SKILL.md.tmpl`)
2. `bun run gen:skill-docs` komutunu çalıştırın (veya bunu otomatik yapan `bun run build` komutunu)
3. Hem `.tmpl` hem de oluşturulan `.md` dosyalarını commit edin

Yeni bir browse komutu eklemek için: `browse/src/commands.ts` dosyasına ekleyin ve yeniden oluşturun.
Yeni bir snapshot bayrağı eklemek için: `browse/src/snapshot.ts` dosyasındaki `SNAPSHOT_FLAGS` dizisine ekleyin ve yeniden oluşturun.

**Token sınırı:** Oluşturulan SKILL.md dosyaları 160KB (~40K token) üzerinde bir uyarı tetikler.
Bu "özellik şişirmesine dikkat" koruma rail'idir, sert bir kapı değil. Modern amiral
modelleri 200K-1M bağlam penceresine sahiptir, bu nedenle 40K pencerenin %4-20'sidir ve prompt önbellekleme
daha büyük skill'lerin marjinal maliyetini küçük yapar. Sınır, kontrolsüz
preamble/resolver büyümesini yakalamak için var, özenle ayarlanmış büyük skill'leri
sıkıştırmaya zorlamak için değil (`ship`, `plan-ceo-review`, `office-hours` meşru olarak
25-35K token davranış paketler). 40K'yi aşarsanız, doğru düzeltme genellikle şudur:
(1) NEyin büyüdüğüne bakın, (2) tek bir resolver tek bir PR'da 10K+ eklediyse, bunun
satır içi mi yoksa referans dokümanı olarak mı yer alması gerektiğini sorgulayın,
(3) özenle ayarlanmış düzyazıyı sıkıştırmayı ancak son çare olarak yapın; kapsam
denetimi, inceleme ordusu veya ses yönergesindeki kesimlerin gerçek kalite maliyeti vardır.

**SKILL.md dosyalarında merge çakışmaları:** Oluşturulan SKILL.md dosyalarındaki çakışmaları
ASLA herhangi bir tarafı kabul ederek çözün. Bunun yerine: (1) çakışmaları `.tmpl`
şablonlarında ve `scripts/gen-skill-docs.ts` dosyasında (doğruluk kaynakları) çözün,
(2) tüm SKILL.md dosyalarını yeniden oluşturmak için `bun run gen:skill-docs` çalıştırın,
(3) yeniden oluşturulan dosyaları stage edin. Bir tarafın oluşturulan çıktısını kabul etmek,
diğer tarafın şablon değişikliklerini sessizce düşürür.

## Platformdan bağımsız tasarım

Skill'ler çerçeveye özgü komutları, dosya kalıplarını veya dizin
yapılarını ASLA sabit kodlamamalıdır. Bunun yerine:

1. **CLAUDE.md dosyasını okuyun**; projeye özgü yapılandırma için (test komutları, eval komutları vb.)
2. **Eksikse AskUserQuestion** — kullanıcıya bildirin veya gstack'in repoyu aramasına izin verin
3. **Yanıtı CLAUDE.md dosyasına kalıcı olarak yazın** böylece bir daha sormak zorunda kalmazsınız

Bu, test komutları, eval komutları, deploy komutları ve diğer tüm
projeye özgü davranışlar için geçerlidir. Proje yapılandırmasının sahibi kendisidir; gstack onu okur.

## SKILL şablonları yazma

SKILL.md.tmpl dosyaları **Claude tarafından okunan prompt şablonlarıdır**, bash script'leri değildir.
Her bash kod bloğu ayrı bir shell'de çalışır; değişkenler bloklar arasında kalıcı değildir.

Kurallar:
- **Mantık ve durum için doğal dil kullanın.** Kod blokları arasında durum taşımak için
  shell değişkenleri kullanmayın. Bunun yerine, Claude'a neyi hatırlaması gerektiğini söyleyin ve
  düzyazıda referans verin (örn., "0. Adımda algılanan temel branch").
- **Branch isimlerini sabit kodlamayın.** `main`/`master`/vb.'yi `gh pr view` veya
  `gh repo view` ile dinamik olarak algılayın. PR hedefleyen skill'ler için
  `{{BASE_BRANCH_DETECT}}` kullanın. Düzyazıda "temel branch", kod bloğu yer tutucularında `<base>` kullanın.
- **Bash bloklarını kendi başına yetkin tutun.** Her kod bloğu bağımsız olarak çalışabilmelidir.
  Bir bloğun önceki bir adımdan bağlama ihtiyacı varsa, bunu yukarıdaki düzyazıda yeniden ifade edin.
- **Koşulları İngilizce olarak ifade edin.** Bash'te iç içe `if/elif/else` yerine,
  numaralandırılmış karar adımları yazın: "1. X ise, Y yap. 2. Aksi takdirde, Z yap."

## Yazım tarzı (V1)

Katman-≥2 her skill'in varsayılan çıktısı `scripts/resolvers/preamble.ts` dosyasındaki
Yazım Tarzı bölümünü takip eder: ilk kullanımda jargon açıklanır (seçkili liste
`scripts/jargon-list.json`, gen-skill-docs zamanında gömülür), sorular
sonuç terimleriyle çerçevelenir ("kullanıcılarınız için ne bozulur..."), uygulama terimleriyle değil,
kısa cümleler, kararlar kullanıcı etkisine yakın. Daha sıkı V0 düzyazısı isteyen
güçlü kullanıcılar `gstack-config set explain_level terse` ayarını yapar (ikili anahtar,
orta mod yok). Tam tasarım gerekçesi için `docs/designs/PLAN_TUNING_V1.md` dosyasına bakın.
Yazım tarzıyla birlikte gitmeye çalışan inceleme hızlandırma yeniden düzenlemesi
V1.1'e ayrıldı; `docs/designs/PACING_UPDATES_V0.md` dosyasına bakın.

## Tarayıcı etkileşimi

Bir tarayıcıyla etkileşim kurmanız gerektiğinde (QA, dogfooding, çerez kurulumu),
`/browse` skill'ini kullanın veya browse binary'sini doğrudan `$B <komut>` ile çalıştırın.
ASLA `mcp__claude-in-chrome__*` araçlarını kullanmayın; onlar yavaş, güvenilmez ve
bu projenin kullandığı şey değil.

**Yan panel mimarisi:** `sidepanel.js`, `background.js`,
`content.js`, `terminal-agent.ts` veya yan panelle ilgili sunucu uç noktalarını
değiştirmeden önce `docs/designs/SIDEBAR_MESSAGE_FLOW.md` dosyasını okuyun. Yan panelin bir
birincil yüzeyi vardır: **Terminal** bölmesi (etkileşimli `claude` PTY) ve bunun
yanında Alıştırmalar / Referanslar / Denetçi, altbilginin `debug` anahtarı arkasında
hata ayıklama katmanları olarak yer alır. Sohrot kuyruğu yolu, PTY kanıtladıktan sonra
kaldırıldı; `sidebar-agent.ts` ve `/sidebar-command` / `/sidebar-chat` /
`/sidebar-agent/event` uç noktaları kaldırılmıştır. Doküman, WS auth
akışını, çift-token modelini ve tehdit-model sınırını kapsar; sessiz hatalar
genellikle bileşenler arası akışın anlaşılmamasından kaynaklanır.

**Gömücü terminal-agent sahipliği** (v1.42.1.0+, kimlik tabanlı sonlandırma v1.44.0.0+).
`browse/src/server.ts` dosyasındaki `buildFetchHandler`, `ServerConfig.ownsTerminalAgent?:
boolean` değerini kabul eder (varsayılan `true`). `true` olduğunda, fabrika kapatılması
tam sökümü çalıştırır: `browse/src/terminal-agent-control.ts` dosyasından
`killAgentByRecord(readAgentRecord(stateDir))` ile kimlik tabanlı sonlandırma ve
`<stateDir>/terminal-port`, `<stateDir>/terminal-internal-token` ve
`<stateDir>/terminal-agent-pid` üzerinde `safeUnlinkQuiet` (v1.44'de tanıtılan,
önyükleme başına ajan kaydı). Kendi PTY
sunucusunu önceden başlatan gömücüler (örn. gbrowser phoenix katmanı), gstack söküm
döngülerinde kendi keşif dosyalarının hayatta kalması için `false` geçmelidir.
Bayrak, `ServerConfig` içindeki üçüncü arayan-sahibi söküm kapısıdır
(`xvfb?` ve `proxyBridge?` yanında); polarite ters çevrilmiştir (varlık yerine açık bool)
ve alanın JSDoc'unda belgelenmiştir. CLI `start()` her zaman açıkça `true` geçer;
`browse/test/server-embedder-terminal-port.test.ts` dosyasındaki statik-grep test'i,
bir yeniden düzenleme bunu düşürürse CI'da başarısız olur. v1.44 öncesi `pkill -f terminal-agent\.ts`
(regex eşleşmesi) kullanıyordu ve aynı host'taki kardeş gstack oturumlarını öldürebilirdi;
yeni `browse/test/terminal-agent-pid-identity.test.ts` statik-grep tripwire'ı, herhangi bir
kaynak dosyası `pkill ... terminal-agent` veya `spawnSync('pkill', ...)` yeniden tanıtırsa
CI'ı başarısız kılar.

**WebSocket auth çerezleri değil Sec-WebSocket-Protocol kullanır.** Tarayıcılar
bir WebSocket yükseltmesinde `Authorization` başlığını ayarlayamaz, ancak
`new WebSocket(url, [token])` ile `Sec-WebSocket-Protocol` ayarlayabilir. Ajan
bunu okur, `validTokens` ile doğrular ve yükseltme yanıtında protokolü geri
yansıtmak ZORUNDADIR; yansıtma olmadan Chromium bağlantıyı hemen kapatır.
`Set-Cookie: gstack_pty=...`, tarayıcı dışı arayanlar için geri dönüş olarak
tutulur (çapraz-port `SameSite=Strict` çerez yolu, chrome-extension kökeninden
kurtulamaz).

**Çapraz-bölme PTY enjeksiyonu.** Araç çubuğunun Temizle düğmesi ve
Denetçi'nin "Koda Gönder" eylemi, her ikisi de `sidepanel-terminal.js` tarafından
açığa çıkarılan `window.gstackInjectToTerminal(text)` üzerinden canlı claude
PTY'sine metin gönderir. `/sidebar-command` POST yok; canlı REPL artık
yan paneldeki tek yürütme yüzeyidir.

**`/health` HERHANGİ bir shell-grant token'ını ortaya çıkarmamalıdır.** Başlıklı modda
zaten `AUTH_TOKEN`'ı localhost arayanlarına sızdırmaktadır (v1.1+ TODO'su). PTY
oturum token'ını da oraya ekleyerek bunu daha kötüleştirmeyin. PTY auth yalnızca
`POST /pty-session` üzerinden akar.

**Aktarım katmanı güvenliği** (v1.6.0.0+). `pair-agent` bir ngrok tüneli başlattığında,
daemon iki HTTP dinleyici bağlar: yerel dinleyici (127.0.0.1, tam komut
yüzeyi, asla yönlendirilmez) ve tünel dinleyicisi (kilitli izin listesi: `/connect`,
kapsamlı token + 26 komutluk tarayıcı-sürme izin listesi ile `/command`,
`/sidebar-chat`). ngrok yalnızca tünel portunu yönlendirir. Tünel üzerinden root token'lar
403 döndürür. SSE uç noktaları `POST /sse-session` üzerinden basılan 30 dakikalık HttpOnly
`gstack_sse` çerezi kullanır (`/command` için asla geçerli değildir). Tünel yüzeyi
reddetmeleri `tunnel-denial-log.ts` üzerinden `~/.gstack/security/attempts.jsonl`
dosyasına gider. `server.ts`, `sse-session-cookie.ts` veya `tunnel-denial-log.ts`
dosyalarını düzenlemeden önce [ARCHITECTURE.md](ARCHITECTURE.md#dual-listener-tunnel-architecture-v1600)
okuyun; modül sınırı (`token-registry.ts`'den `sse-session-cookie.ts`'e içe aktarma yok)
kapsam izolasyonu için yapısal öneme sahiptir.

**Sunucu çıkışında Unicode arıtma** (v1.38.0.0+). Sayfa içeriğinden türetilen dizeleri
gönderen her sunucu çıkışı, nesne yükleri için `JSON.stringify(payload,
sanitizeReplacer)` veya metin gövdeleri için `sanitizeLoneSurrogates(body)` üzerinden
geçmek ZORUNDADIR. Aksi takdirde CDP sayfa içeriğindeki yalın UTF-16 vekil yarımları
Anthropic API'ye `\uD800` tarzı kaçışlar olarak ulaşır ve 400 hatası tetikler. Bugün
dört çıkış noktasına bağlanmıştır: `handleCommandInternal` (HTTP + sanitize sarmalayıcısı
üzerinden batch, `handleCommandInternalImpl` etrafında) ve her iki SSE üreticisi
(`/activity/stream`, `/inspector/events`). Stringify sonrası regex hiçbir şey yapmaz;
`JSON.stringify` vekili regex eşleştirmeden önce kaçırılmıştır, bu nedenle replacer
kodlama pipeline'ı içinde çalışmak zorundadır. `server.ts` dosyasında yeni bir
SSE/WebSocket yazıcı veya HTTP yanıt eklemeden önce
[ARCHITECTURE.md](ARCHITECTURE.md#unicode-sanitization-at-server-egress-v13800)
okuyun. `browse/test/server-sanitize-surrogates.test.ts` değişmez testlerle
bağlantıyı sabitler, bu nedenle baypaslar CI'ı başarısız kılar.

**Kurulum symlink sağlamlaştırma** (v1.38.0.0+). `setup` içindeki her bağlantı yeri,
`IS_WINDOWS` algılamasının yanındaki `_link_or_copy SRC DST` yardımcısı üzerinden
yönlendirilmek ZORUNDADIR. Geliştirici Modu olmayan Windows'ta, düz `ln -snf`
donmuş dosya kopyaları üretir ve `git pull` sonrası yenilenmez; bu her host
bağdaştırıcısında sessiz eskime anlamına gelir. Yardımcı Unix'te `ln -snf`'i korur
ve Windows'ta `cp -R` / `cp -f`'e geçer. `test/setup-windows-fallback.test.ts`
statik bir değişmez uygular: yardımcı gövdesi dışında tek bir ham `ln` çağrısı
CI'ı başarısız kılar. Windows kullanıcıları her `git pull` sonrası `./setup`'ı
yeniden çalıştırmayı hatırlatan `_print_windows_copy_note_once`'tan bir satırlık not alır.

**Yan panel güvenlik yığını** (prompt enjeksiyonuna karşı katmanlı savunma):

| Katman | Modül | Yaşadığı yer |
|-------|--------|----------|
| L1-L3 | `content-security.ts` | hem sunucu hem ajan — veri işaretleme, gizli öğe çıkarma, ARIA regex, URL engelleme listesi, zarf sarmalama |
| L4 | `security-classifier.ts` (TestSavantAI ONNX) | **yalnızca sidebar-agent** |
| L4b | `security-classifier.ts` (Claude Haiku transkript) | **yalnızca sidebar-agent** |
| L5 | `security.ts` (canary) | her ikisi — derlenmiş içine enjekte, ajanda kontrol |
| L6 | `security.ts` (combineVerdict topluluk) | her ikisi |

**Kritik kısıtlama:** `security-classifier.ts` derlenmiş browse binary'sinden içe aktarılamaz.
`@huggingface/transformers` v4, `onnxruntime-node` gerektirir ve bu da
Bun compile'ın geçici çıkarma dizininden `dlopen` yapamaz. Yalnızca `security.ts`
(saf dize işlemleri — canary, karar birleştirici, saldırı günlüğü, durum) `server.ts`
için güvenlidir. Tam mimari karar için `~/.gstack/projects/garrytan-gstack/ceo-plans/2026-04-19-prompt-injection-guard.md`
§"Pre-Impl Gate 1 Outcome" bölümüne bakın.

**Eşikler** (`security.ts` içinde):
- `BLOCK: 0.85` — çapraz doğrulanırsa BLOCK'a neden olacak tek katmanlı skor
- `WARN: 0.75` — çapraz doğrulama eşiği. L4 VE L4b her ikisi >= 0.75 olduğunda → BLOCK
- `LOG_ONLY: 0.40` — transkript sınıflandırıcısını geçitler (tüm katmanlar < 0.40 olduğunda Haiku'yu atla)
- `SOLO_CONTENT_BLOCK: 0.92` — etiketsiz içerik sınıflandırıcıları için tek katmanlı eşik
  (testsavant, deberta). Kasıtlı olarak `BLOCK`'tan yüksektir çünkü bu katmanlar "bu bir
  enjeksiyon" ile "bu kullanıcıya yönelik oltalama gibi görünüyor" arasındaki farkı
  ayırt edemez. Transkript sınıflandırıcısı `BLOCK` (0.85) eşiğinde ayrı, etiket-geçitli
  tek başına yol tutar.

**Topluluk kuralı:** BLOCK yalnızca ML içerik sınıflandırıcısı VE transkript
sınıflandırıcısının her ikisi >= WARN bildirmesi durumunda gerçekleşir. Tek katmanlı
yüksek güven WARN'e düşürülür; bu, Stack Overflow talimat-yazma FP azaltma önlemidir.
Canary sızıntısı her zaman BLOCK'lar (deterministik).

**Ortam değişkeni anahtarları:**
- `GSTACK_SECURITY_OFF=1` — acil kapatma anahtarı. Sınıflandırıcı ısınmış olsa bile kapalı kalır.
  Canary hala enjekte edilir; yalnızca ML taraması atlanır.
- `GSTACK_SECURITY_ENSEMBLE=deberta` — katılımci DeBERTa-v3 topluluğu. L4c sınıflandırıcı olarak
  ProtectAI DeBERTa-v3-base-injection-onnx ekler ve çapraz-model anlaşması sağlar.
  İlk çalıştırmada 721MB indirme. Topluluk etkinken, BLOCK için 2/3 ML sınıflandırıcının
  >= WARN'de anlaşması gerekir (testsavant, deberta, transkript). Topluluk olmadan
  (varsayılan), BLOCK için testsavant + transkript'in >= WARN'de olması gerekir.
- Sınıflandırıcı model önbelleği: `~/.gstack/models/testsavant-small/` (112MB, yalnızca ilk çalıştırma)
  artı `~/.gstack/models/deberta-v3-injection/` (721MB, yalnızca topluluk etkinken)
- Saldırı günlüğü: `~/.gstack/security/attempts.jsonl` (tuzlanmış sha256 + yalnızca alan adı,
  10MB'da döndürülür, 5 nesil)
- Cihaz başına tuz: `~/.gstack/security/device-salt` (0600)
- Oturum durumu: `~/.gstack/security/session-state.json` (çapraz-süreç, atomik)

## Geliştirme symlink farkındalığı

gstack geliştirilirken, `.claude/skills/gstack` bu çalışma dizinine geri
symlink olabilir (gitignore edilmiş). Bu, skill değişikliklerinin **anında canlı**
olduğu anlamına gelir; hızlı yineleme için harika, ancak yarı yazılmış skill'lerin
eşzamanlı olarak gstack kullanan diğer Claude Code oturumlarını bozabileceği
büyük yeniden düzenlemeler sırasında risklidir.

**Oturum başına bir kez kontrol edin:** Symlink mi yoksa gerçek kopya mı görmek için
`ls -la .claude/skills/gstack` komutunu çalıştırın. Çalışma dizininize bir symlink ise,
şunların farkında olun:
- Şablon değişiklikleri + `bun run gen:skill-docs` tüm gstack çağrılarını anında etkiler
- SKILL.md.tmpl dosyalarındaki bozucu değişiklikler eşzamanlı gstack oturumlarını bozabilir
- Büyük yeniden düzenlemeler sırasında, global kurulumun `~/.claude/skills/gstack/`
  kullanılması için symlink'i kaldırın (`rm .claude/skills/gstack`)

**Önek ayarı:** Kurulum, üst düzeyde (gstack/ altında değil) içinde bir SKILL.md
symlink'i olan gerçek dizinler (symlink'ler değil) oluşturur (örn., `qa/SKILL.md -> gstack/qa/SKILL.md`).
Bu, Claude'un bunları gstack/ altında iç içe değil, üst düzey skill'ler olarak keşfetmesini sağlar.
İsimler ya kısadır (`qa`) ya da ad alanlıdır (`gstack-qa`), `~/.gstack/config.yaml`
dosyasındaki `skill_prefix` tarafından kontrol edilir. İnteraktif istemi atlamak için
`--no-prefix` veya `--prefix` geçirin.

**Not:** gstack'i bir projenin reposuna vendor olarak eklemek kullanım dışıdır. Bunun yerine
global kurulum + `./setup --team` kullanın. Takım modu talimatları için README.md dosyasına bakın.

**Plan incelemeleri için:** Skill şablonlarını veya gen-skill-docs pipeline'ını
değiştiren planları incelerken, değişikliklerin canlıya çıkmadan önce izole olarak
test edilmesi gerekip gerekmediğini düşünün (özellikle kullanıcı gstack'i diğer
pencerelerde aktif olarak kullanıyorsa).

**Yükseltme geçişleri:** Bir değişiklik disk üzerindeki durumu (dizin yapısı,
yapılandırma formatı, eski dosyalar) mevcut kullanıcı kurulumlarını bozabilecek
şekilde değiştirdiğinde, `gstack-upgrade/migrations/` dizinine bir geçiş script'i ekleyin.
Format ve test gereksinimleri için CONTRIBUTING.md'nin "Upgrade migrations" bölümünü okuyun.
Yükseltme skill'i bunları `/gstack-upgrade` sırasında `./setup`'tan sonra otomatik olarak çalıştırır.

## Derlenmiş binary'ler — browse/dist/ veya design/dist/ ASLA commit etmeyin

`browse/dist/` ve `design/dist/` dizinleri derlenmiş Bun binary'leri içerir
(`browse`, `find-browse`, `design`, her biri ~58MB). Bunlar yalnızca Mach-O arm64'tür;
Linux, Windows veya Intel Mac'lerde çalışmazlar. `./setup` script'i zaten her
platform için kaynaktan oluşturur, bu nedenle izlenen binary'ler gereksizdir. Bunlar
geçmiş bir hata nedeniyle git tarafından izlenmektedir ve sonunda `git rm --cached`
ile kaldırılmalıdır.

**Bu dosyaları ASLA stage etmeyin veya commit etmeyin.** `.gitignore` olmalarına rağmen
izlendikleri için `git status`'ta değiştirilmiş olarak görünürler; onları yok sayın.
Dosyaları stage ederken, her zaman belirli dosya adları kullanın (`git add dosya1 dosya2`);
asla `git add .` veya `git add -A` kullanmayın, çünkü bu binary'leri yanlışlıkla dahil eder.

## Commit tarzı

**Her zaman bisect commit yapın.** Her commit tek bir mantıksal değişiklik olmalıdır.
Birden fazla değişiklik yaptığınızda (örn., bir yeniden adlandırma + yeniden yazma + yeni testler),
push etmeden önce bunları ayrı commit'lara bölün. Her commit bağımsız olarak anlaşılabilir
ve geri alınabilir olmalıdır.

İyi bisect örnekleri:
- Yeniden adlandırma/taşıma, davranış değişikliklerinden ayrı
- Test altyapısı (touchfiles, yardımcılar), test uygulamalarından ayrı
- Şablon değişiklikleri, oluşturulan dosyaların yeniden oluşturulmasından ayrı
- Mekanik yeniden düzenlemeler, yeni özelliklerden ayrı

Kullanıcı "bisect commit" veya "bisect and push" dediğinde, stage edilmiş/stage
edilmemiş değişiklikleri mantıksal commit'lara bölün ve push edin.

## Slop-scan: AI kod kalitesi, AI kod gizleme değil

AI tarafından üretilen kodun insanın yazacağından gerçekten daha kötü olduğu
kalıpları yakalamak için [slop-scan](https://github.com/benvinegar/slop-scan) kullanıyoruz.
İnsan kodu gibi geçmeye ÇALIŞMIYORUZ. AI ile kodladığımız ve bununla gurur duyuyoruz.
Amaç kod kalitesidir.

```bash
npx slop-scan scan .          # insan tarafından okunabilir rapor
npx slop-scan scan . --json   # makine tarafından okunabilir, diff için
```

Yapılandırma: Repo kökündeki `slop-scan.config.json` (şu anda `**/vendor/**` hariç tutuyor).

### Düzelenecekler (gerçek kalite iyileştirmeleri)

- **Dosya işlemleri etrafında boş catch'ler** — `safeUnlink()` kullanın (ENOENT'yi yok sayar, EPERM/EIO'yu yeniden fırlatır). Temizlikte yutulan bir EPERM sessiz veri kaybı anlamına gelir.
- **Süreç sonlandırmaları etrafında boş catch'ler** — `safeKill()` kullanın (ESRCH'yi yok sayar, EPERM'yi yeniden fırlatır). Yutulan bir EPERM, bir şeyi öldürdüğünüzü sandığınız ama öldüremediğiniz anlamına gelir.
- **Gereksiz `return await`** — sarmalayan try bloğu yoksa kaldırın. Bir microtask tasarruf eder, niyeti belirtir.
- **Tiplenmiş istisna yakalamaları** — `catch (err) { if (!(err instanceof TypeError)) throw err }`,
  try bloğu URL ayrıştırma veya DOM işlemi yaptığında `catch {}`'ten gerçekten daha iyidir.
  Hangi hatayı beklediğinizi biliyorsunuz, o halde söyleyin.

### Düzeltilmeyecekler (linter oyunu, kalite değil)

- **Hata mesajlarında dize eşleştirme** — `err.message.includes('closed')` kırılgandır.
  Playwright/Chrome her zaman ifadeleri değiştirebilir. Bir fire-and-forget işlemi HERHANGİ bir
  nedenle başarısız olabiliyorsa ve umursamıyorsanız, `catch {}` doğru kalıptır.
- **Geçiş sarmalayıcılarını muaf tutmak için yorum eklemek** — slop-scan'in muafiyet kuralını
  tetiklemek için bir metodun üstündeki "alias for active session" yorumu, dokümantasyon değil
  gürültüdür.
- **Uzantı catch-and-log'u seçici yeniden fırlatmaya dönüştürmek** — Chrome uzantıları
  yakalanmamış hatalarda tamamen çöker. Catch logluyor ve devam ediyorsa, bu uzantı kodu
  için DOĞRU kalıptır. Throw yaptırmayın.
- **En iyi çaba temizlik yollarını sıkılaştırmak** — kapatma, acil durum temizliği ve bağlantı kesme
  kodu `safeUnlinkQuiet()` kullanmalıdır (TÜM hataları yutar). EPERM'de throw yapan bir
  temizlik yolu, geri kalan temizliğin çalışmaması anlamına gelir. Bu daha kötüdür.

### `browse/src/error-handling.ts` içindeki yardımcılar

| İşlev | Kullanım durumu | Davranış |
|----------|----------|----------|
| `safeUnlink(path)` | Normal dosya silme | ENOENT'yi yok sayar, diğerlerini yeniden fırlatır |
| `safeUnlinkQuiet(path)` | Kapatma/acil durum temizliği | Tüm hataları yutar |
| `safeKill(pid, signal)` | Sinyal gönderme | ESRCH'yi yok sayar, diğerlerini yeniden fırlatır |
| `isProcessAlive(pid)` | Boolean süreç kontrolleri | true/false döndürür, asla throw yapmaz |

### Skor takibi

Başlangıç (2026-04-09, temizlik öncesi): 100 bulgu, 432.8 skor, 2.38 skor/dosya.
Temizlik sonrası: 90 bulgu, 358.1 skor, 1.96 skor/dosya.

Sayının peşine düşmeyin. Gerçek kod kalitesi sorunlarını temsil eden kalıpları düzeltin.
"Dağınık" kalıbın doğru mühendislik tercihi olduğu bulguları kabul edin.

## Topluluk PR koruma rayları

Topluluk PR'larını incelerken veya birleştirirken, şu commit'leri kabul etmeden önce
**her zaman AskUserQuestion** yapın:

1. **ETHOS.md'ye dokunuyor** — bu dosya Garry'nin kişisel yapıcı felsefesidir. Dış katkıda
   bulananlardan veya AI ajanlarından düzenleme yok, istisnasız.
2. **Promosyon materyallerini kaldırıyor veya yumuşatıyor** — YC referansları, kurucu perspektifi
   ve ürün sesi kasıtlıdır. Bunları "gereksiz" veya "çok promosyonel" olarak çerçeveleyen
   PR'lar reddedilmelidir.
3. **Garry'nin sesini değiştiriyor** — skill şablonlarındaki, CHANGELOG'daki ve dokümanlardaki
   ton, mizah, doğrudanlık ve perspektif genel değildir. Sesi daha "tarafsız" veya "profesyonel"
   olacak şekilde yeniden yazan PR'lar reddedilmelidir.

Ajan bir değişikliğin projeyi iyileştirdiğine güçlü bir şekilde inansa bile, bu üç kategori
AskUserQuestion aracılığıyla açık kullanıcı onayı gerektirir. İstisna yok.
Otomatik birleştirme yok. "Bunu ben hallederim" yok.

## garrytan-agents'ten PR'ları kontrol etme

Kullanıcı "check out <PR bağlantısı>" dediğinde ve PR `garrytan-agents/gstack`'ten
(veya `garrytan/gstack` üzerinde işbirlikçi olmayan başka bir fork'tan) geliyorsa, sadece
`gh pr checkout` yapmayın. Fork PR'leri temel repo sırlarını (`ANTHROPIC_API_KEY`,
`OPENAI_API_KEY` vb.) alamaz, bu nedenle eval/E2E CI işleri temel repoda ne ayarlanmış
olursa olsun boş-ortam auth hatalarıyla başarısız olur.

**İş akışı:** Branch'ı `garrytan/gstack`'e (temel repo) push edin ve PR'ı oradan yeniden hedefleyin.

Somut olarak, `gh pr checkout <N>` komutundan sonra:

1. Orijinal PR numarasını ve head branch adını not edin.
2. Aynı branch'ı temel repoya push edin: `git push origin HEAD:<branch-adı>`
   (origin = `garrytan/gstack`, worktree bu remote ile kurulduğu için).
3. Fork PR'ı kapatın (`gh pr close <N> --comment "moving to base-repo branch for secret access"`).
4. Temel repo branch'ından yeni bir PR açın: `gh pr create --base main --head <branch-adı>`.
5. Yeni PR'nin iş akışları sırları otomatik olarak alacaktır.

Neden fork tarafında düzeltmeyesiniz? `garrytan-agents`, `garrytan/gstack` üzerinde
işbirlikçi değil. İşbirlikçi olarak eklemek (seçenek A) veya repo genelinde "sırları
fork PR'lerine gönder" anahtarını çevirmek (seçenek B), sırların herhangi birinin
fork PR'lerine ulaşmasını sağlar; bu, sadece bu bir branch'ı taşımaktan daha geniş
etki alanıdır. Seçenek C (bu bölüm) gizli dağıtım kapsamını sıkı tutar.

Kullanıcı taşımayı atlamayı istiyorsa (örn., "fork PR olarak bırak"), buna
saygı gösterin; eval CI boş-ortam auth ile başarısız olur, ancak check-freshness,
workflow-lint ve windows-tests fork PR'ında hala geçecektir.

## CHANGELOG + VERSION tarzı

**Sürümleme değişmezi (workspace-farkında ship).** VERSION monotonik sıralı bir
yayın tanımlayıcısdır, katı bir semver taahhüdü değil. Yükseltme seviyesi
(major/minor/patch/micro), ship zamanındaki niyeti ifade eder. Aynı yükseltme
seviyesi içinde önceden talep edilmiş bir sürümü sıraya ilerletmek açıkça izinlidir;
A branch v1.7.0.0'ı MINOR olarak talep ederse ve B branch da MINOR ise, B v1.8.0.0'da
ulaşır (hala main'e göre bir MINOR). Aşağı akış tüketiciler, "MINOR = yalnızca özellik,
PATCH = yalnızca düzeltme" konusunda katı bir sözleşme olarak DAYANMAMALIDIR. Bu yüzden
`bin/gstack-next-version`, çarpışmalar olduğunda yükseltme seviyesini yeniden seçmek
yerine seçilen yükseltme seviyesi içinde ilerler.

**Ölçek-farkında yükseltmeler — sağduyu kullanın.** Diff büyükse, PATCH değil
MINOR (veya MAJOR) yükseltin. PATCH hata düzeltmeleri ve küçük eklemeler içindir;
MINOR önemli yeni yetenek veya önemli azalma içindir; MAJOR bozucu değişiklikler içindir.
Kaba kılavuzlar (kurallar olarak değil, koku kontrolleri olarak düşünün):

- **PATCH (X.Y.Z+1.0)**: hata düzeltme, doküman ince ayarı, küçük artan değişiklik, tek
  test/dosya eklendi. Net diff ~500 satırın altında, yeni kullanıcıya dönük yetenek yok.
- **MINOR (X.Y+1.0.0)**: yeni yetenek ship edildi (skill, harness, komut, büyük
  yeniden düzenleme), önemli kod azaltma (sıkıştırma, geçiş) veya koordineli
  çoklu dosya değişikliği. Net diff ~2000 satırın üzerinde eklendi/kaldırıldı, VEYA
  bir tweet'te paylaşacağınız kullanıcıya görünür bir özellik.
- **MAJOR (X+1.0.0.0)**: genel yüzeysel bozucu değişiklik (CLI bayrak yeniden adlandırma,
  skill kaldırma, yapılandırma formatı değişti), VEYA bir blog yazısının manşeti olacak
  kadar büyük bir yayın.

"10K eklenen + 24K kaldırılan gerçekten PATCH mı?" diye tartışıyorsanız — değildir.
MINOR yükseltün. "Bu 6 yeni E2E testi + yardımcı araçlarla tamamen yeni bir test harness
ekliyor" için de aynıı — MINOR. Yükseltme seviyesi, kullanıcıya bu yayının ne tür
olduğuna dair iletişimdir; onu düşük satmayın.

origin/main'i birleştirmek daha yüksek bir VERSION getirdiğinde, yükseltme seviyesini
sadece main'in ileriye taşındığına göre değil, branch'ınızın çalışmasının ÖLÇEĞİNE
göre yeniden değerlendirin. Main MINOR yükseltildiyse ve branch'ınız da önemli bir
değişiklikse, üzerine tekrar MINOR yükseltin (örn., main v1.14.0.0'da, branch'ınız v1.15.0.0'da ulaşır).

**VERSION ve CHANGELOG branch kapsamlıdır.** Ship eden her özellik branch'ının kendi
sürüm yükseltmesi ve CHANGELOG girdisi vardır. Girdi, BU branch'ın eklediğini açıklar;
main'de zaten ne olduğundan değil.

**CHANGELOG girdisi, main ile shipping branch arasındaki farktır; kullanıcılar
yükselttiğinde ne elde ettikleridir. Branch'in oraya nasıl ulaştığı DEĞİL.** Bir okuyucu
girdiye geldiğinde, daha önce yapamadığı ama şimdi yapabildiği şeyi öğrenmelidir;
branch'ın iç sürüm yükseltmelerini, branch geliştirme sırasında yakaladığımız ve
düzelttiğimiz hataları, çalıştırdığımız plan incelemelerini veya ezediğimiz commit'leri
öğrenmemelidir. Bu, branch geliştirme anlatısıdır. PR açıklamalarına ve commit mesajlarına
aittir, CHANGELOG'a değil.

**Branch-içi sürümlere CHANGELOG girdisinde ASLA referans vermeyin.** Branch'ınız
geliştirme sırasında VERSION'u v1.5.0.0 → v1.5.1.0 → v1.6.0.0 olarak yükselttiyse
ve yalnızca son v1.6.0.0 main'e ship olduysa, girdi v1.5.1.0'ın hiç var olmamış gibi
okunmalıdır. Somut olarak, ASLA şunu yazmayın:
- "v1.5.1.0'da v1.6.0.0'ın düzelttiği bir hata vardı" — okuyucular v1.5.1.0'ı bilmiyor;
  bu bir branch-içi eser.
- "v1.5.1.0'ın ship manşeti bozuk çünkü..." — aynı neden. Main'in perspektifinden,
  v1.5.1.0 hiç yayınlanmadı.
- "Düzeltme öncesi testler bozuk davranışı kodladı" — bu katkıda bulunanın zafer turu,
  kullanıcı yararı değil.
- "İki cerrahi düzenleme, ikisi de dispatch yolunda" — yamanın mikro anlatısı.

Bunun yerine, ship edilen sistemi tanımlayın: "Tarayıcı-skill'leri beklenen sekme erişim
anlamlarıyla uçtan uca çalışır." Ship edilen sistemin bir özelliği dikkate değer ise
(örn., "skill spawn'ları izin verilen sekme erişimi alır; pair-agent tünel token'ları
sahiplik gerektirir"), bunu bir düzeltme olarak değil bir özellik olarak belgeleyin.
Ship edilen sistem kullanıcının elde ettiği şeydir; o sisteme giden yol onlar için
görünmezdir.

**CHANGELOG girdisi ne zaman yazılır:**
- `/ship` zamanında (13. Adım), geliştirme sırasında veya branch ortasında değil.
- Girdi, bu branch üzerindeki temel branch'a karşı TÜM commit'leri kapsar.
- Main'de zaten bulunan önceki bir sürümün CHANGELOG girdisine asla yeni iş katmayın.
  Main v0.10.0.0'daysa ve branch'ınız özellikler ekliyorsa, yeni bir girdiyle v0.10.1.0'a
  yükseltin; v0.10.0.0 girdisini düzenlemeyin.

**Yazmadan önce temel sorular:**
1. Hangi branch'tayım? BU branch ne değiştirdi?
2. Temel branch sürümü zaten yayınlandı mı? (Evetse, yükselt ve yeni girdi oluştur.)
3. Bu branch'ta mevcut bir girdi daha önceki çalışmayı kapsıyor mu? (Evetse, son sürüm
   için birleşik bir girdiyle değiştirin.)

**Main'i birleştirmek, main'in sürümünü benimsemek DEĞİLDİR.** origin/main'i bir
özellik branch'ına birleştirdiğinizde, main yeni CHANGELOG girdileri ve daha yüksek
bir VERSION getirebilir. Branch'ınızın hala üzerinde kendi sürüm yükseltmesine
ihtiyacı var. Main v0.13.8.0'daysa ve branch'ınız özellikler ekliyorsa, yeni bir
girdiyle v0.13.9.0'a yükseltin. Değişikliklerinizi main'de zaten bulunan bir girdiye
sıkıştırmayın. Branch'ınız sonra ulaştığı için girdiniz en üstte yer alır.

**Main'i birleştirdikten sonra, her zaman kontrol edin:**
- CHANGELOG'da branch'ınızın kendi girdisi, main'in girdilerinden ayrı mı?
- VERSION, main'in VERSION'undan yüksek mü?
- Girdiniz CHANGELOG'daki en üstteki girdi mi (main'in en yenisinin üstünde)?
Herhangi bir yanıt hayırsa, devam etmeden önce düzeltin.

**CHANGELOG girdilerini taşıyan, ekleyen veya kaldıran her CHANGELOG düzenlemesinden sonra,**
hemen `grep "^## \[" CHANGELOG.md` komutunu çalıştırarak yinelenen olmadığını ve mantıklı bir
ters kronolojik sıra olduğunu doğrulayın. Sürüm numaraları arasındaki boşluklar sorun değil.
v1.5.2.0 veya v1.5.3.0 girdisi olmadan v1.6.4.0'ta ship olan bir branch doğrudur;
bunlar main'de hiç yer almayan branch-içi sürüm numaralarıydı. Boşlukları yer tutucu
girdilerle geri doldurmayın.

**Branch-içi sürümleri asla yetim bırakmayın.** Branch'ınız geliştirme sırasında
VERSION'u birkaç kez yükselttiyse (örn., v1.5.1.0 → v1.5.2.0 → v1.6.4.0) ve bu önceki
girdiler hiçbir zaman main'e yayınlanmadıysa, son ship TÜMÜNÜ son sürümde (v1.6.4.0)
tek bir girdide birleştirir. Eski girdileri silin ve içeriklerini son girdiye taşıyın,
sürüm tablosu sütunlarını buna göre güncelleyin. Okuyucular bir yayın görür,
branch günlüğü değil. Boşluklar sorun değil (main'de v1.5.x arası olmadan v1.6.3.0 → v1.6.4.0
doğrudur).

CHANGELOG.md **kullanıcılar içindir**, katkıda bulunanlar için değil. Ürün sürüm notları
gibi yazın:

- Kullanıcının şimdi yapabildiği ama daha önce yapamadığı şeyle başlayın. Özelliği satın.
- Düz yazı kullanın, uygulama detaylarını değil. "Artık yapabilirsiniz..." "Yeniden düzenlendi..." değil.
- **ASLA TODOS.md, iç izleme, eval altyapısı veya katkıda bulunan-yönelik detaylardan bahsetmeyin.**
  Bunlar kullanıcılar için görünmez ve onlar için anlamsızdır.
- Katkıda bulunan/iç değişiklikleri altta ayrı bir "Katkıda bulunanlar için" bölümüne koyun.
- Her girdi birinin "oh güzel, bunu denemek istiyorum" düşünmesini sağlamalıdır.
- Jargon yok: "Her soru artık hangi projede ve hangi branch'ta olduğunuzu söylüyor" deyin,
  "AskUserQuestion formatı preamble resolver üzerinden skill şablonlarında standartlaştırıldı" demeyin.

**Yalnızca main ile bu değişiklik arasında ship edilenleri belgeleyin.** Okuyucular
orasına nasıl geldiğimizi umursamıyor. CHANGELOG'dan her zaman çıkarın:

- Branch yeniden senkronizasyonları, main ile merge commit'leri, rebase etkinliği.
- Plan onayları, inceleme sonuçları (CEO / mühendislik / tasarım / dış-ses / codex bulguları),
  AskUserQuestion kararları, kapsam müzakereleri.
- "İş sıraya alındı", "plan onaylandı", "devam ediyor", "daha sonra ship edilecek" — CHANGELOG
  ship EDİLENİ belgeler, ship EDİLEBİLECEĞİni değil.
- Kullanıcıya dönük hiçbir iş gerçekten yerleşmediğinde sürüm yükseltme evrak işi.

Temel branch sürümü ile bu sürüm arasındaki diff'te kullanıcıya dönük değişiklik yoksa
(yalnızca merge'ler, yalnızca CHANGELOG düzenlemeleri, yalnızca yer tutucu iş), dürüst
girdi bir cümledir: "Branch-önde disiplini için sürüm yükseltme. Henüz kullanıcıya
dönük değişiklik yok." Orada durun. Doldurmayın. Sonunda ship edecek planı açıklamayın.
Branch'ın geçmişini anlatmayın. Gerçek iş yerleştiğinde, girdi /ship zamanında bunun
yerini alacaktır.

### Yayın-özet formatı (her `## [X.Y.Z]` girdisi)

`CHANGELOG.md`'deki her sürüm girdisi, GStack/Garry sesinde bir yayın-özet bölümüyle
başlamak ZORUNDADIR; bir görüş penceresi değerinde düzyazı + tablolar, pazarlama değil
bir karar gibi gelmelidir. Maddelik CHANGELOG (alt bölümler, madde işaretleri, dosyalar)
bunun ALTINDA gelir, bir `### Maddelik değişiklikler` başlığıyla ayrılır.

Yayın-özet bölümü insanlar, otomatik güncelleme ajanı ve yükseltip yükseltmeyeceğine
karar veren herkes tarafından okunur. Maddelik liste, tam olarak neyin değiştiğini
bilmesi gereken ajanlar içindir.

Her `## [X.Y.Z]` girdisinin en üstündeki yapı:

1. **İki satırlık kalın manşet** (toplam 10-14 kelime). Bir karar gibi gelmeli,
   pazarlama değil. Bugün ship eden ve çalışıp çalışmadığını önemseyen biri gibi konuşun.
2. **Başlangıç paragrafı** (3-5 cümle). Ne ship edildi, kullanıcı için ne değişti.
   Somut, belirgin, AI kelime dağarcığı yok, em dash yok, abartı yok.
3. **"Önemli olan X sayı" bölümü** ile:
   - Sayıların kaynağını belirten kısa bir kurulum paragrafı (gerçek üretim
     dağıtımı VEYA yeniden üretilebilir kıyaslama, dosya/komut adını belirtin).
   - ÖNCE / SONRA / Δ sütunlarıyla 3-6 temel ölçüm tablosu.
   - İlgiliyse kategori başına döküm için isteğe bağlı ikinci tablo.
   - En çarpıcı sayıyı somut kullanıcı terimleriyle yorumlayan 1-2 cümle.
4. **"Bu [izleyici] için ne anlama gelir" kapanış paragrafı** (2-4 cümle)
   ölçümleri gerçek bir iş akışı kaymasına bağlar. Ne yapılması gerektiğiyle bitirin.

Yayın özeti için ses kuralları:
- Em dash yok (virgül, nokta, "..." kullanın).
- AI kelime dağarcığı (delve, robust, comprehensive, nuanced, fundamental vb.) veya
  yasaklı ifadeler ("işin can alıcı noktası", "alt çizgi" vb.) yok.
- Gerçek sayılar, gerçek dosya adları, gerçek komutlar. "Hızlı" değil, "30K sayfada ~30s."
- Kısa paragraflar, tek cümlelik vurguları 2-3 cümlelik akışlarla karıştırın.
- Kullanıcı sonuçlarına bağlayın: "ajan ~3x daha az okuyor", "iyileştirilmiş hassasiyet"ten daha iyi.
- Kalite hakkında doğrudan olun. "İyi tasarlanmış" veya "bu bir karmaşa." Dans etmeyin.

Kaynak malzeme:
- Önceki bağlam için CHANGELOG önceki girdisi.
- Başlık sayıları için kıyaslama dosyaları veya `/retro` çıktısı.
- Ne ship edildiğini görmek için son commit'ler (`git log <önceki-sürüm>..HEAD --oneline`).
- Sayı uydurmayın. Bir ölçüm kıyaslama veya üretim verisinde yoksa,
  dahil etmeyin. Sorulursa "henüz ölçüm yok" deyin.

Hedef uzunluk: Özet için ~250-350 kelime. Bir görüş penceresi olarak render olmalıdır.

### Maddelik değişiklikler (yayın özetinin altında)

`### Maddelik değişiklikler` yazın ve detaylı alt bölümlerle devam edin (Eklendi,
Değiştirildi, Düzeltildi, Katkıda bulunanlar için). Yukarıdaki kullanıcıya dönük ses
yönergeleriyle aynı kurallar, artı:

- **Topluluk katkılarına her zaman kredi verin.** Bir girdi bir topluluk PR'ından
  iş içeriyorsa, katkıda bulunanı `Contributed by @username` ile adlandırın. Katkıda
  bulunanlar gerçek iş yaptı. Onları her zaman her durumda açıkça teşekkür edin, istisnasız.

## AI çaba sıkıştırma

Çabayı tahmin ederken veya tartışırken, her zaman insan ekibi ve CC+gstack süresini gösterin:

| Görev türü | İnsan ekibi | CC+gstack | Sıkıştırma |
|-----------|-----------|-----------|-------------|
| Boilerplate / iskelet | 2 gün | 15 dk | ~100x |
| Test yazma | 1 gün | 15 dk | ~50x |
| Özellik uygulama | 1 hafta | 30 dk | ~30x |
| Hata düzeltme + gerileme testi | 4 saat | 15 dk | ~20x |
| Mimari / tasarım | 2 gün | 4 saat | ~5x |
| Araştırma / keşif | 1 gün | 3 saat | ~3x |

Tamamlılık ucuzdur. Tam uygulama bir "göl" (başarılabilir) olduğunda, "okyanus"
(çok-çeyrek geçiş) değil, kısayollar önermeyin. Tam felsefe için skill preamble'ındaki
Tamamlılık İlkesine bakın.

## Yapmadan önce araştır

Eşzamanlılık, yabancı kalıplar, altyapı veya çalışma zamanı/çerçevenin yerleşik bir
özelliği olabileceği herhangi bir şey içeren bir çözüm tasarlamadan önce:

1. "{çalışma zamanı} {şey} yerleşik" araştırın
2. "{şey} en iyi uygulama {geçerli yıl}" araştırın
3. Resmi çalışma zamanı/çerçeve dokümanlarını kontrol edin

Bilginin üç katmanı: denenmiş-ve-doğru (Katman 1), yeni-ve-popüler (Katman 2),
ilk-ilkeler (Katman 3). Her şeyden çok Katman 3'ü ödüllendirin. Tam yapıcı felsefe için
ETHOS.md dosyasına bakın.

## Yerel planlar

Katkıda bulunanlar uzun vadeli vizyon dokümanlarını ve tasarım dokümanlarını
`~/.gstack-dev/plans/` dizininde saklayabilir. Bunlar yalnızca yereldir (işlenmez).
TODOS.md'yi incelerken, TODO'lara terfi etmeye veya uygulamaya hazır olabilecek
adaylar için `plans/` dizinini kontrol edin.

## E2E eval başarısızlığı suç protokolü

`/ship` veya başka bir iş akışı sırasında bir E2E eval başarısız olduğunda, **"değişikliklerimizle
ilişkili değil" iddia etmeden önce bunu kanıtlayın.** Bu sistemlerin görünmez
bağlantıları vardır; bir preamble metin değişikliği ajan davranışını etkiler,
yeni bir yardımcı zamanlamayı değiştirir, yeniden oluşturulan bir SKILL.md prompt
bağlamını kaydırır.

**"Önceden var olan" olarak bir başarısızlığı atfetmeden önce gerekli:**
1. Aynı eval'i main'de (veya temel branch'ta) çalıştırın ve orada da başarısız olduğunu gösterin
2. Main'de geçer ama branch'ta başarısız olursa — bu SİZİN değişikliğinizdir. Suçu izleyin.
3. Main'de çalışamıyorsanız, "doğrulanmamış — ilgili olabilir veya olmayabilir" deyin ve
   PR gövdesinde bir risk olarak işaretleyin

Makbuzsuz "önceden var olan" tembel bir iddiadır. Kanıtlayın veya söylemeyin.

## Uzun süre çalışan görevler: pes etmeyin

Eval'ler, E2E testleri veya herhangi bir uzun süre çalışan arka plan görevi çalıştırırken,
**tamamlanana kadar yoklayın**. Her 3 dakikada bir `sleep 180 && echo "ready"` + `TaskOutput`
kullanın. Yoklama zaman aşımına uğradığında asla engellemeli moda geçmeyin ve pes etmeyin.
Asla "tamamlandığında bildirileceğim" demeyin ve kontrol etmeyi bırakmayın; görev
bitene veya kullanıcı durmamı söyleyene kadar döngüye devam edin.

Tam E2E seti 30-45 dakika sürebilir. Bu 10-15 yoklama döngüsüdür. Hepsini yapın.
Her kontrolde ilerlemeyi raporlayın (hangi testler geçti, hangileri çalışıyor, şimdiye kadar
hangi başarısızlıklar var). Kullanıcı çalışmanın tamamlanmasını görmek ister, daha sonra
kontrol edeceğime dair bir söz değil.

## E2E test fixture'ları: kopyalayın, değil kopyalamayın

**Tam bir SKILL.md dosyasını ASLA bir E2E test fixture'ına kopyalamayın.** SKILL.md dosyaları
1500-2000 satırdır. `claude -p` o kadar büyük bir dosyayı okuduğunda, bağlam şişirmesi
zaman aşımlarına, dalgalı dönüş sınırlarına ve gerekenden 5-10x daha uzun süren testlere
neden olur.

Bunun yerine, yalnızca testin gerçekten ihtiyaç duyduğu bölümü çıkarın:

```typescript
// KÖTÜ — ajan 1900 satır okur, ilgili olmayan bölümlerde token harcar
fs.copyFileSync(path.join(ROOT, 'ship', 'SKILL.md'), path.join(dir, 'ship-SKILL.md'));

// İYİ — ajan ~60 satır okur, zaman aşımı yerine 38 saniyede tamamlanır
const full = fs.readFileSync(path.join(ROOT, 'ship', 'SKILL.md'), 'utf-8');
const start = full.indexOf('## Review Readiness Dashboard');
const end = full.indexOf('\n---\n', start);
fs.writeFileSync(path.join(dir, 'ship-SKILL.md'), full.slice(start, end > start ? end : undefined));
```

Ayrıca başarısızlıkları ayıklamak için hedeflenmiş E2E testleri çalıştırırken:
- **Ön planda** çalıştırın (`bun test ...`), `&` ve `tee` ile arka planda değil
- Çalışan eval süreçlerini asla `pkill` yapıp yeniden başlatmayın — sonuçları kaybedersiniz ve para israf edersiniz
- Bir temiz çalıştırma, üç öldürülüp-yeniden başlatılan çalıştırmadan iyidir

## Yerel OpenClaw skill'lerini ClawHub'ta yayınlama

Yerel OpenClaw skill'leri `openclaw/skills/gstack-openclaw-*/SKILL.md` içinde yer alır. Bunlar
ClawHub'ta herhangi bir OpenClaw kullanıcısının kurabilmesi için yayınlanan el yapımı
metodoloji skill'leridir (pipeline tarafından oluşturulmamış).

**Yayınlama:** Komut `clawhub publish`'dir (`clawhub skill publish` DEĞİL):

```bash
clawhub publish openclaw/skills/gstack-openclaw-office-hours \
  --slug gstack-openclaw-office-hours --name "gstack Office Hours" \
  --version 1.0.0 --changelog "değişikliklerin açıklaması"
```

Her skill için tekrarlayın: `gstack-openclaw-ceo-review`, `gstack-openclaw-investigate`,
`gstack-openclaw-retro`. Her güncellemede `--version`'ı yükseltin.

**Kimlik doğrulama:** `clawhub login` (GitHub auth için tarayıcı açar). Doğrulamak için `clawhub whoami`.

**Güncelleme:** Daha yüksek `--version` ve `--changelog` ile aynı `clawhub publish` komutu.

**Doğrulama:** Canlı olduklarını onaylamak için `clawhub search gstack`.

## Aktif skill'e deploy etme

Aktif skill `~/.claude/skills/gstack/` dizininde yer alır. Değişiklikler yaptıktan sonra:

1. Branch'ınızı push edin
2. Skill dizininde getirin ve sıfırlayın: `cd ~/.claude/skills/gstack && git fetch origin && git reset --hard origin/main`
3. Yeniden oluşturun: `cd ~/.claude/skills/gstack && bun run build`

Veya binary'leri doğrudan kopyalayın:
- `cp browse/dist/browse ~/.claude/skills/gstack/browse/dist/browse`
- `cp design/dist/design ~/.claude/skills/gstack/design/dist/design`

## Skill yönlendirme

Kullanıcının isteği kullanılabilir bir skill ile eşleştiğinde, Skill aracı aracılığıyla çağırın.
Şüpheye düştüğünüzde, skill'i çağırın.

Temel yönlendirme kuralları:
- Ürün fikirleri/beyin fırtınası → /office-hours
- Strateji/kapsam → /plan-ceo-review
- Mimari → /plan-eng-review
- Tasarım sistemi/plan incelemesi → /design-consultation veya /plan-design-review
- Tam inceleme pipeline'ı → /autoplan
- Hatalar/hatalar → /investigate
- QA/site davranışını test etme → /qa veya /qa-only
- Kod incelemesi/diff kontrolü → /review
- Görselcilik → /design-review
- Ship/deploy/PR → /ship veya /land-and-deploy
- İlerlemeyi kaydet → /context-save
- Bağlamı devam ettir → /context-restore

## GBrain Arama Kılavuzu (/sync-gbrain tarafından yapılandırılır)
<!-- gstack-gbrain-search-guidance:start -->

GBrain bu makinede kuruludur ve senkronize edilmiştir. Ajan, soru anlamsal olduğunda veya
henüz tam tanımlayıcıyı bilmediğinde Grep yerine gbrain'ı tercih etmelidir.

**Bu worktree, repo kökündeki `.gbrain-source` dosyası aracılığıyla (kubectl tarzı bağlam)
worktree-kapsamlı bir kod kaynağına sabitlenmiştir.** Bu worktree altındaki herhangi bir
yerden yapılan `gbrain code-def`, `code-refs`, `code-callers`, `code-callees` veya `query`
çağrısı varsayılan olarak o kaynağa yönlendirilir; `--source` bayrağı gerekmez. Aynı repo
Conductor kardeş worktree'lerinin her birinin kendi sabitlemesi ve kendi dizinli sayfaları vardır,
bu nedenle anlamsal sonuçlar bu worktree'deki diskteki gerçek kodla eşleşir.

`gbrain` CLI aracılığıyla kullanılabilen iki dizinli kaynak:
- Bu worktree'nin kodu (`.gbrain-source` aracılığıyla otomatik sabitlenmiş).
- `~/.gstack/` seçkili bellek (mevcut federasyon pipeline'ı üzerinden
  `gstack-brain-<user>` kaynağı olarak kayıtlı).

gbrain'ı tercih edin:
- "X nerede işleniyor?" / anlamsal niyet, henüz tam dize yok:
    `gbrain search "<terimler>"` veya `gbrain query "<soru>"`
- "Y sembolü nerede tanımlanmış?" / sembol tabanlı kod soruları:
    `gbrain code-def <sembol>` veya `gbrain code-refs <sembol>`
- "Y'yi ne çağırıyor?" / "Y neye bağımlı?":
    `gbrain code-callers <sembol>` / `gbrain code-callees <sembol>`
- "Geçen sefer ne karar verdik?" / geçmiş planlar, retrospektifler, öğrenimler:
    `gbrain search "<terimler>" --source gstack-brain-<user>`

Grep, bilinen tam dizeler, regex, çok satırlı kalıplar ve dosya globları için hala doğrudur.
Anlamlı kod değişikliklerinden sonra `/sync-gbrain` çalıştırın; tüm worktree'lerde sürekli
otomatik senkronizasyon için makine başına bir kez `gbrain autopilot --install` çalıştırın;
gbrain'ın daemon'u bir planlamada artımlı yenileme gerçekleştirir.

<!-- gstack-gbrain-search-guidance:end -->