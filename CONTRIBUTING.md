# gstack'e Katkıda Bulunma

gstack'i daha iyi hale getirmek istediğiniz için teşekkürler. İster bir skill prompt'undaki yazım hatasını düzeltiyor olun, ister tamamen yeni bir workflow inşa ediyor olun, bu rehber sizi hızla başlatacaktır.

## Hızlı başlangıç

gstack skill'leri, Claude Code'un bir `skills/` dizininden keşfettiği Markdown dosyalarıdır. Normalde `~/.claude/skills/gstack/` konumundadır (global kurulumunuz). Ancak gstack'in kendisini geliştirirken, Claude Code'un working tree'nizdeki skill'leri kullanmasını istersiniz — böylece düzenlemeler kopyalama veya deploy olmadan anında etkili olur.

Bu, dev modunun yaptığıdır. Repo'nuzu yerel `.claude/skills/` dizinine sembolik bağla bağlar, böylece Claude Code skill'leri doğrudan checkout'unuzdan okur.

```bash
git clone https://github.com/garrytan/gstack.git && cd gstack
bun install                    # bağımlılıkları kur
bin/dev-setup                  # dev modunu aktifleştir
```

> **Tam klon vs sığ.** README'nin kullanıcıya yönelik kurulumu hız için `--depth 1` kullanır. Katkıda bulunan olarak, tam bir klon kullanın (`--depth` flag'i yok) — `git log`, `git blame`, `git bisect` ve önceki versiyonlara karşı PR'ları incelemek için geçmişe ihtiyacınız olacak. README'yi takip ederek zaten `--depth 1` klonu yaptıysanız, `git fetch --unshallow` ile tam klona yükseltin.

Artık herhangi bir `SKILL.md`'yi düzenleyin, Claude Code'da çağırın (örn. `/review`) ve değişikliklerinizi canlı görün. Geliştirmeyi bitirdiğinizde:

```bash
bin/dev-teardown               # devre dışı bırak — global kurulumunuza geri dönün
```

## Operasyonel kendi kendini geliştirme

gstack başarısızlıklardan otomatik olarak öğrenir. Her skill oturumunun sonunda, ajan
neyin yanlış gittiğini (CLI hataları, yanlış yaklaşımlar, proje özellikleri) yansıtır ve
operasyonel öğrenmeleri `~/.gstack/projects/{slug}/learnings.jsonl` dosyasına kaydeder. Gelecek
oturumlar bu öğrenmeleri otomatik olarak ortaya çıkarır, böylece gstack kod tabanınız üzerinde
zamanla daha akıllı hale gelir.

Kurulum gerekmez. Öğrenmeler otomatik olarak kaydedilir. `/learn` ile görüntüleyin.

### Katkıda bulunan workflow'u

1. **gstack'i normal şekilde kullanın** — operasyonel öğrenmeler otomatik olarak yakalanır
2. **Öğrenmelerinizi kontrol edin:** `/learn` veya `ls ~/.gstack/projects/*/learnings.jsonl`
3. **gstack'i fork edin ve klonlayın** (henüz yapmadıysanız)
4. **Fork'unuzu bug'ı yaşadığınız projeye sembolik bağla bağlayın:**
   ```bash
   # Çekirdek projenizde (gstack'in sizi rahatsız ettiği proje)
   ln -sfn /path/to/your/gstack-fork .claude/skills/gstack
   cd .claude/skills/gstack && bun install && bun run build && ./setup
   ```
   Setup, içinde SKILL.md sembolik bağları bulunan skill başına dizinler oluşturur (`qa/SKILL.md -> gstack/qa/SKILL.md`)
   ve önek tercih sorar. İstemeyi atlamak ve kısa isimler kullanmak için `--no-prefix` geçirin.
5. **Sorunu düzeltin** — değişiklikleriniz bu projede anında canlı
6. **Gerçekten gstack'i kullanarak test edin** — sizi rahatsız eden şeyi yapın, düzeltildiğini doğrulayın
7. **Fork'unuzdan bir PR açın**

Bu, katkıda bulunmanın en iyi yoludur: gerçek işinizi yaparken, sorunu gerçekten hissettiğiniz
projede gstack'i düzeltin.

### Oturum farkındalığı

Aynı anda 3+ gstack oturumu açtığınızda, her soru size hangi proje, hangi branch ve neler
olduğunu söyler. Bir soruya bakıp "bekle, bu hangi pencereydi?" diye düşünmek yok. Format tüm
skill'lerde tutarlıdır.

## gstack repo'su içinde gstack üzerinde çalışmak

gstack skill'lerini düzenlerken ve onları aynı repo'da gerçekten gstack'i kullanarak test
etmek istediğinizde, `bin/dev-setup` bunu ayarlar. Working tree'nize geri işaret eden
`.claude/skills/` sembolik bağları oluşturur (gitignore edilmiş), böylece Claude Code global
kurulum yerine yerel düzenlemelerinizi kullanır.

```
gstack/                          <- working tree'niz
├── .claude/skills/              <- dev-setup tarafından oluşturuldu (gitignore)
│   ├── gstack -> ../../         <- repo köküne sembolik bağ
│   ├── review/                  <- gerçek dizin (kısa isim, varsayılan)
│   │   └── SKILL.md -> gstack/review/SKILL.md
│   ├── ship/                    <- veya --prefix ile gstack-review/, gstack-ship/
│   │   └── SKILL.md -> gstack/ship/SKILL.md
│   └── ...                      <- skill başına bir dizin
├── review/
│   └── SKILL.md                 <- bunu düzenleyin, /review ile test edin
├── ship/
│   └── SKILL.md
├── browse/
│   ├── src/                     <- TypeScript kaynağı
│   └── dist/                    <- derlenmiş binary (gitignore)
└── ...
```

Setup, üst düzeyde gerçek dizinler (sembolik bağ değil) oluşturur, içinde bir SKILL.md
sembolik bağıyla birlikte. Bu, Claude'un onları `gstack/` altında değil, üst düzey skill'ler
olarak keşfetmesini sağlar. İsimler önek ayarınıza bağlıdır (`~/.gstack/config.yaml`).
Kısa isimler (`/review`, `/ship`) varsayılandır. İsim alanlı isimler (`/gstack-review`,
`/gstack-ship`) tercih ediyorsanız `./setup --prefix` çalıştırın.

## Günlük workflow

```bash
# 1. Dev moduna girin
bin/dev-setup

# 2. Bir skill düzenleyin
vim review/SKILL.md

# 3. Claude Code'da test edin — değişiklikler canlı
#    > /review

# 4. Browse kaynağını mı düzenliyorsunuz? Binary'yi yeniden derleyin
bun run build

# 5. Gün bitti? Kapatın
bin/dev-teardown
```

## Test & eval'ler

### Kurulum

```bash
# 1. .env.example'i kopyalayın ve API anahtarınızı ekleyin
cp .env.example .env
# .env dosyasını düzenleyin → ANTHROPIC_API_KEY=sk-ant-... ayarlayın

# 2. Bağımlılıkları kurun (henüz yapmadıysanız)
bun install
```

Bun `.env`'i otomatik yükler — ekstra yapılandırma gerekmez. Conductor workspace'leri `.env`'i ana worktree'den otomatik olarak miras alır (aşağıda "Conductor workspace'leri" bölümüne bakın).

### Test katmanları

| Katman | Komut | Maliyet | Ne test eder |
|--------|-------|---------|--------------|
| 1 — Statik | `bun test` | Ücretsiz | Komut doğrulama, snapshot flag'leri, SKILL.md doğruluğu, TODOS-format.md referansları, gözlemlenebilirlik birim testleri |
| 2 — Uçtan uca | `bun run test:e2e` | ~$3.85 | `claude -p` alt süreci aracılığıyla tam skill çalıştırması |
| 3 — LLM eval | `bun run test:evals` | ~$0.15 bağımsız | Üretilen SKILL.md dokümanlarının LLM-as-judge puanlaması |
| 2+3 | `bun run test:evals` | ~$4 birleşik | Uçtan uca + LLM-as-judge (her ikisini de çalıştırır) |

```bash
bun test                     # Yalnızca Katman 1 (her commit'te çalışır, <5s)
bun run test:e2e             # Yalnızca Katman 2: Uçtan uca (EVALS=1 gerektirir, Claude Code içinde çalışamaz)
bun run test:evals           # Katman 2 + 3 birleşik (~$4/çalıştırma)
```

### Katman 1: Statik doğrulama (ücretsiz)

`bun test` ile otomatik olarak çalışır. API anahtarı gerekmez.

- **Skill parser testleri** (`test/skill-parser.test.ts`) — SKILL.md bash kod bloklarındaki her `$B` komutunu çıkarır ve `browse/src/commands.ts` içindeki komut kayıt defterine karşı doğrular. Yazım hataları, kaldırılmış komutlar ve geçersiz snapshot flag'leri yakalar.
- **Skill doğrulama testleri** (`test/skill-validation.test.ts`) — SKILL.md dosyalarının yalnızca gerçek komutlara ve flag'lere referans gösterdiğini ve komut açıklamalarının kalite eşiklerini karşıladığını doğrular.
- **Üretici testler** (`test/gen-skill-docs.test.ts`) — Şablon sistemini test eder: yer tutucuların doğru şekilde çözüldüğünü, çıktının flag'ler için değer ipuçları içerdiğini (örn. sadece `-d` değil `-d <N>`), anahtar komutlar için zenginleştirilmiş açıklamalar olduğunu (örn. `is` geçerli durumları listeler, `press` tuş örnekleri listeler) doğrular.

### Katman 2: `claude -p` aracılığıyla uçtan uca (~$3.85/çalıştırma)

`claude -p`'yi `--output-format stream-json --verbose` ile bir alt süreç olarak başlatır, gerçek zamanlı ilerleme için NDJSON akışı sağlar ve browse hataları için tarar. Bu, "bu skill gerçekten uçtan uca çalışıyor mu?" sorusuna en yakın şeydir.

```bash
# Düz bir terminalden çalıştırılmalı — Claude Code veya Conductor içine iç içe konamaz
EVALS=1 bun test test/skill-e2e-*.test.ts
```

- `EVALS=1` ortam değişkeni ile sınırlandırılır (yanlışlıkla pahalı çalıştırmaları önler)
- Claude Code içinde çalışıyorsa otomatik atlar (`claude -p` iç içe konamaz)
- API bağlantı ön kontrolü — bütçe harcamadan ConnectionRefused durumunda hızlıca başarısız olur
- stderr'a gerçek zamanlı ilerleme: `[Ns] turn T tool #C: Name(...)`
- Hata ayıklama için tam NDJSON transkriptleri ve başarısızlık JSON'ları kaydeder
- Testler `test/skill-e2e-*.test.ts` içinde yaşar (kategoriye göre ayrılmış), çalıştırıcı mantığı `test/helpers/session-runner.ts` içinde

### Uçtan uca gözlemlenebilirlik

Uçtan uca testler çalıştırıldığında, `~/.gstack-dev/` içinde makine tarafından okunabilir artifact'ler üretir:

| Artifact | Yol | Amacı |
|----------|------|---------|
| Kalp atışı | `e2e-live.json` | Mevcut test durumu (araç çağrısı başına güncellenir) |
| Kısmi sonuçlar | `evals/_partial-e2e.json` | Tamamlanmış testler (kill'lerden kurtulur) |
| İlerleme günlüğü | `e2e-runs/{runId}/progress.log` | Yalnızca ekleme metin günlüğü |
| NDJSON transkriptleri | `e2e-runs/{runId}/{test}.ndjson` | Test başına ham `claude -p` çıktısı |
| Başarısızlık JSON'ı | `e2e-runs/{runId}/{test}-failure.json` | Başarısızlıkta tanı verisi |

**Canlı dashboard:** Tamamlanan testleri, şu anda çalışan testi ve maliyeti gösteren canlı bir dashboard görmek için ikinci bir terminalde `bun run eval:watch` çalıştırın. Son 10 ilerleme.log satırını da görmek için `--tail` kullanın.

**Eval geçmiş araçları:**

```bash
bun run eval:list            # tüm eval çalıştırmalarını listele (tur, süre, çalıştırma başına maliyet)
bun run eval:compare         # iki çalıştırmayı karşılaştır — test başına deltalar + Yorum bölümünü gösterir
bun run eval:summary         # çalıştırmalar arası toplu istatistikler + test başına verimlilik ortalamaları
```

**Eval karşılaştırma yorumu:** `eval:compare`, çalıştırmalar arasındaki değişiklikleri yorumlayan doğal dil Yorum bölümleri üretir — regresyonları işaretler, iyileştirmeleri not eder, verimlilik kazanımlarını vurgular (daha az tur, daha hızlı, daha ucuz) ve genel bir özet üretir. Bu, `eval-store.ts` içindeki `generateCommentary()` tarafından yönetilir.

Artifact'ler asla temizlenmez — otopsi hata ayıklama ve trend analizi için `~/.gstack-dev/` içinde birikir.

### Katman 3: LLM-as-judge (~$0.15/çalıştırma)

Üretilen SKILL.md dokümanlarını üç boyutta puanlamak için Claude Sonnet kullanır:

- **Netlik** — Bir AI ajanı talimatları belirsizlik olmadan anlayabilir mi?
- **Bütünlük** — Tüm komutlar, flag'ler ve kullanım pattern'leri dokümante edilmiş mi?
- **Uygulanabilirlik** — Ajan, yalnızca dokümandaki bilgileri kullanarak görevleri yürütebilir mi?

Her boyut 1-5 puanlanır. Eşik: her boyut **≥ 4** puan almalıdır. Ayrıca üretilen dokümanları `origin/main`'den el ile korunmuş temel çizgiye karşı karşılaştıran bir regresyon testi vardır — üretilenler eşit veya daha yüksek puan almalıdır.

```bash
# .env dosyasında ANTHROPIC_API_KEY gerektirir — bun run test:evals içinde dahil edilir
```

- Puanlama kararlılığı için `claude-sonnet-4-6` kullanır
- Testler `test/skill-llm-eval.test.ts` içinde yaşar
- Anthropic API'yi doğrudan çağırır (`claude -p` değil), bu yüzden Claude Code içinde dahil her yerden çalışır

### CI

Bir GitHub Action (`.github/workflows/skill-docs.yml`) her push ve PR'da `bun run gen:skill-docs --dry-run` çalıştırır. Üretilen SKILL.md dosyaları commitlenenlerden farklıysa, CI başarısız olur. Bu, eski dokümanların merge edilmeden önce yakalanmasını sağlar.

Testler browse binary'sine doğrudan karşı çalışır — dev modu gerektirmez.

## SKILL.md dosyalarını düzenleme

SKILL.md dosyaları `.tmpl` şablonlarından **üretilir**. `.md` dosyasını doğrudan düzenlemeyin — değişiklikleriniz bir sonraki build'de üzerine yazılacaktır.

```bash
# 1. Şablonu düzenleyin
vim SKILL.md.tmpl              # veya browse/SKILL.md.tmpl

# 2. Tüm host'lar için yeniden üretin
bun run gen:skill-docs --host all

# 3. Sağlığı kontrol edin (tüm host'ları raporlar)
bun run skill:check

# Veya watch modu kullanın — kaydetme üzerine otomatik yeniden üretir
bun run dev:skill
```

Şablon yazarlığı en iyi uygulamaları (bash-ism'ler yerine doğal dil, dinamik branch algılama, `{{BASE_BRANCH_DETECT}}` kullanımı) için CLAUDE.md'nin "SKILL şablonları yazma" bölümüne bakın.

Bir browse komutu eklemek için `browse/src/commands.ts` dosyasına ekleyin. Bir snapshot flag'i eklemek için `browse/src/snapshot.ts` dosyasındaki `SNAPSHOT_FLAGS`'e ekleyin. Ardından yeniden derleyin.

## Jargon listesi (V1 yazım tarzı)

gstack'in Yazım Tarzı bölümü (katman ≥2 olan her skill'in başlangıcına eklenir),
her skill çağrısında ilk kullanımda teknik terimleri açıklar. Açıklamaya hak kazanan
terimlerin listesi `scripts/jargon-list.json` dosyasında yaşar — ~50 seçilmiş yüksek
frekanslı terim (idempotent, race condition, N+1, backpressure vb.).
Listede olmayan terimler yeterince düz-İngilizce kabul edilir.

**Terim ekleme veya kaldırma:** `scripts/jargon-list.json` dosyasını düzenleyerek bir PR açın.
Düzenlemeden sonra `bun run gen:skill-docs` çalıştırın — terimler üretilen her
SKILL.md'ye üretim zamanında dahil edilir, bu yüzden değişiklikler yalnızca yeniden üretmeden
sonra etkili olur. Runtime yüklemesi yoktur; kullanıcı tarafında override yoktur. Repo listesi
gerçeklik kaynağıdır.

Ekleme için iyi adaylar: teknik olmayan kullanıcıların bağlam olmadan review çıktısında
karşılaştığı yüksek frekanslı terimler (yaygın veritabanı/eşzamanlılık terminolojisi, güvenlik
jargonu, frontend framework kavramları). Yalnızca bir veya iki niş skill'de görünen terimleri
eklemeyin — maliyetten değere oran, review ek yüküne değmez.

## Çoklu-host geliştirme

gstack, bir dizi `.tmpl` şablonundan 8 host için SKILL.md dosyaları üretir.
Her host, `hosts/*.ts` içinde tiplenmiş bir yapılandırmadır. Üretici, host'a uygun çıktı
üretmek için bu yapılandırmaları okur (farklı frontmatter, yollar, araç isimleri).

**Desteklenen host'lar:** Claude (birincil), Codex, Factory, Kiro, OpenCode, Slate, Cursor, OpenClaw.

### Tüm host'lar için üretme

```bash
# Belirli bir host için üret
bun run gen:skill-docs                    # Claude (varsayılan)
bun run gen:skill-docs --host codex       # Codex
bun run gen:skill-docs --host opencode    # OpenCode
bun run gen:skill-docs --host all         # Tüm 8 host

# Veya build kullanın, tüm host'lar + binary'leri derler
bun run build
```

### Host'lar arası ne değişir

Her host yapılandırması (`hosts/*.ts`) şunları kontrol eder:

| Yön | Örnek (Claude vs Codex) |
|--------|---------------------------|
| Çıktı dizini | `{skill}/SKILL.md` vs `.agents/skills/gstack-{skill}/SKILL.md` |
| Frontmatter | Tam (isim, açıklama, hook'lar, versiyon) vs minimal (isim + açıklama) |
| Yollar | `~/.claude/skills/gstack` vs `$GSTACK_ROOT` |
| Araç isimleri | "Bash aracını kullanın" vs aynı (Factory "bu komutu çalıştır" olarak yeniden yazar) |
| Hook skill'leri | `hooks:` frontmatter vs satır içi güvenlik danışmanlık metni |
| Bastırılan bölümler | Yok vs Codex kendi-çağırma bölümleri çıkarılmış |

Tam `HostConfig` arayüzü için `scripts/host-config.ts` dosyasına bakın.

### Host çıktısını test etme

```bash
# Tüm statik testleri çalıştır (tüm host'lar için parametreli smoke testleri içerir)
bun test

# Tüm host'lar için tazelik kontrolü
bun run gen:skill-docs --host all --dry-run

# Sağlık dashboard'u tüm host'ları kapsar
bun run skill:check
```

### Yeni bir host ekleme

Tam rehber için [docs/ADDING_A_HOST.md](docs/ADDING_A_HOST.md) dosyasına bakın. Kısa versiyon:

1. `hosts/myhost.ts` oluşturun (`hosts/opencode.ts` dosyasından kopyalayın)
2. `hosts/index.ts`'ye ekleyin
3. `.gitignore`'a `.myhost/` ekleyin
4. `bun run gen:skill-docs --host myhost` çalıştırın
5. `bun test` çalıştırın (parametreli testler otomatik olarak kapsar)

Sıfır üretici, kurulum veya araç kodu değişikliği gerekmez.

### Yeni bir skill ekleme

Yeni bir skill şablonu eklediğinizde, tüm host'lar otomatik olarak onu alır:
1. `{skill}/SKILL.md.tmpl` oluşturun
2. `bun run gen:skill-docs --host all` çalıştırın
3. Dinamik şablon keşfi onu alır, güncellenecek statik liste yok
4. `{skill}/SKILL.md` commit edin, dış host çıktısı kurulum zamanında üretilir ve gitignore edilir

## Conductor workspace'leri

Paralel olarak birden fazla Claude Code oturumu çalıştırmak için [Conductor](https://conductor.build) kullanıyorsanız, `conductor.json` workspace yaşam döngüsünü otomatik olarak bağlar:

| Hook | Betik | Ne yapar |
|------|--------|-------------|
| `setup` | `bin/dev-setup` | Ana worktree'den `.env` kopyalar, bağımlılıkları kurar, skill'leri sembolik bağla bağlar |
| `archive` | `bin/dev-teardown` | Skill sembolik bağlarını kaldırır, `.claude/` dizinini temizler |

Conductor yeni bir workspace oluşturduğunda, `bin/dev-setup` otomatik olarak çalışır. Ana worktree'yi algılar (`git worktree list` aracılığıyla), API anahtarlarınızın taşınması için `.env` dosyanızı kopyalar ve dev modunu ayarlar — manuel adım gerekmez.

**İlk kurulum:** `ANTHROPIC_API_KEY`'nizi ana repo'daki `.env` dosyasına koyun (`.env.example` dosyasına bakın). Her Conductor workspace'i onu otomatik olarak miras alır.

**`GSTACK_*` ortam öneki (Conductor-enjekte edilen anahtarlar).** Conductor, her workspace'in süreç ortamından `ANTHROPIC_API_KEY` ve `OPENAI_API_KEY`'yi açıkça kaldırır. `.env` kopyalama yolu bunları geri yüklemiyor — kaldırma işlemi ortam mirasından sonra gerçekleşir. Ücretli eval'lerin, `/sync-gbrain` embedding'lerinin veya `claude-agent-sdk` çağrılarının bir Conductor workspace'inde çalışmasını isteyen kullanıcılar, Conductor'un workspace ortam yapılandırmasında `GSTACK_ANTHROPIC_API_KEY` ve `GSTACK_OPENAI_API_KEY` ayarlamalıdır; Conductor bunları dokunulmadan geçirir. gstack tarafında, TS giriş noktaları yan etki olarak `lib/conductor-env-shim.ts` dosyasını import eder; bu, kanonik isim boş olduğunda `GSTACK_FOO_API_KEY`'yi `FOO_API_KEY`'ye yükseltir. Ücretli bir API'ye dokunan yeni bir TS giriş noktası eklerseniz, dosyanın en üstüne `import "../lib/conductor-env-shim";` ekleyin. Bugün shim şu dosyalardan import edilmiştir: `bin/gstack-gbrain-sync.ts`, `bin/gstack-model-benchmark`, `scripts/preflight-agent-sdk.ts` ve `test/helpers/e2e-helpers.ts`.

## Bilinmesi gerekenler

- **SKILL.md dosyaları üretilir.** `.md` dosyasını değil, `.tmpl` şablonunu düzenleyin. Yeniden üretmek için `bun run gen:skill-docs` çalıştırın.
- **TODOS.md birleşik backloğdur.** P0-P4 öncelikleriyle skill/bileşen bazında organize edilir. `/ship` tamamlanan öğeleri otomatik algılar. Tüm planlama/review/retro skill'leri bağlam için onu okur.
- **Browse kaynak değişiklikleri yeniden derleme gerektirir.** `browse/src/*.ts` dosyasına dokunursanız, `bun run build` çalıştırın.
- **Dev modu global kurulumunuzu gölgeler.** Proje-yerel skill'ler `~/.claude/skills/gstack`'e öncelik alır. `bin/dev-teardown` global olanı geri yükler.
- **Conductor workspace'leri bağımsızdır.** Her workspace kendi git worktree'sidir. `bin/dev-setup` `conductor.json` aracılığıyla otomatik olarak çalışır.
- **`.env` worktree'ler arasında yayılır.** Ana repoda bir kez ayarlayın, tüm Conductor workspace'leri onu alır.
- **`.claude/skills/` gitignore edilir.** Sembolik bağlar asla commit edilmez.
- **`setup` içinde asla ham `ln -snf` yazmayın.** `setup` içindeki her bağlantı noktası, `IS_WINDOWS` algılama yakınındaki `_link_or_copy SRC DST` yardımcısı üzerinden ZORUNLU olarak yönlendirilmelidir. Yardımcı Unix'te `ln -snf`'i korur ve Developer Mode olmadan Windows'ta `cp -R` / `cp -f`'e geçer; burada düz `ln -snf`, `git pull` üzerine yenilenmeyen dondurulmuş dosya kopyaları üretir. `test/setup-windows-fallback.test.ts` bunu statik bir değişmez ile uygular — yardımcı gövdesi dışında tek bir ham `ln` çağrısı CI'yi başarısız kılar.

## Değişikliklerinizi gerçek bir projede test etme

**Bu, gstack geliştirmenin önerilen yoludur.** gstack checkout'unuzu gerçekten kullandığınız
projeye sembolik bağla bağlayın, böylece gerçek iş yaparken değişiklikleriniz canlı olsun.

### Adım 1: Checkout'unuzu sembolik bağla bağlayın

```bash
# Çekirdek projenizde (gstack repo'sunda değil)
ln -sfn /path/to/your/gstack-checkout .claude/skills/gstack
```

### Adım 2: Skill başına sembolik bağları oluşturmak için setup çalıştırın

Sadece `gstack` sembolik bağı yeterli değildir. Claude Code skill'leri
bireysel üst düzey dizinler (`qa/SKILL.md`, `ship/SKILL.md` vb.) aracılığıyla keşfeder,
`gstack/` dizininin kendisi aracılığıyla değil. Onları oluşturmak için `./setup` çalıştırın:

```bash
cd .claude/skills/gstack && bun install && bun run build && ./setup
```

Setup, kısa isimler (`/qa`) mi yoksa isim alanlı mı (`/gstack-qa`) istediğinizi soracak.
Seçiminiz `~/.gstack/config.yaml`'a kaydedilir ve gelecekteki çalıştırmalar için hatırlanır.
İstemeyi atlamak için `--no-prefix` (kısa isimler) veya `--prefix` (isim alanlı) geçirin.

### Adım 3: Geliştirin

Bir şablonu düzenleyin, `bun run gen:skill-docs` çalıştırın ve bir sonraki `/review` veya
`/qa` çağrısı onu hemen alır. Yeniden başlatma gerekmez.

### Kararlı global kurulumuna geri dönme

Proje-yerel sembolik bağı kaldırın. Claude Code `~/.claude/skills/gstack/`'e geri döner:

```bash
rm .claude/skills/gstack
```

Skill başına dizinler (`qa/`, `ship/` vb.) `gstack/...`'a işaret eden SKILL.md sembolik
bağları içerir, bu yüzden otomatik olarak global kurulumu çözerler.

### Önek modunu değiştirme

gstack'i bir önek ayarıyla kurduysanız ve değiştirmek istiyorsanız:

```bash
cd .claude/skills/gstack && ./setup --no-prefix   # /qa, /ship'e geç
cd .claude/skills/gstack && ./setup --prefix       # /gstack-qa, /gstack-ship'e geç
```

Setup eski sembolik bağları otomatik olarak temizler. Manuel temizlik gerekmez.

### Alternatif: global kurulumunuzu bir branch'a yönlendirme

Proje başına sembolik bağlar istemiyorsanız, global kurulumu değiştirebilirsiniz:

```bash
cd ~/.claude/skills/gstack
git fetch origin
git checkout origin/<branch>
bun install && bun run build && ./setup
```

Bu, tüm projeleri etkiler. Geri almak için: `git checkout main && git pull && bun run build && ./setup`.

## Topluluk PR triyajı (dalga süreci)

Topluluk PR'ları biriktiğinde, onları temalı dalgalara gruplayın:

1. **Kategorize edin** — temaya göre gruplayın (güvenlik, özellikler, altyapı, dokümanlar)
2. **Tekilleştirin** — iki PR aynı şeyi düzeltiyorsa, daha az satır değiştireni seçin. Diğerini kazananı işaret eden bir notla kapatın.
3. **Toplayıcı branch** — `pr-wave-N` oluşturun, temiz PR'ları merge edin, kirli olanlar için
   çakışmaları çözün, `bun test && bun run build` ile doğrulayın
4. **Bağlamla kapatın** — kapatılan her PR, nedenini ve neyin (varsa) yerine geçtiğini açıklayan bir yorum alır. Katkıda bulunanlar gerçek iş yaptı; buna net iletişimle saygı gösterin.
5. **Tek PR olarak ship edin** — merge commit'lerinde tüm atıfların korunduğu ana kola tek PR. Neyin merge edildiğini ve neyin kapatıldığını özetleyen bir tablo ekleyin.

Örnek olarak [PR #205](../../pull/205) (v0.8.3) dosyasına bakın.

## Yükseltme migrasyonları

Bir release disk üzerinde durumu (dizin yapısı, yapılandırma formatı, eski
dosyalar) `./setup` tek başına düzeltemeyecek şekilde değiştirdiğinde, mevcut
kullanıcıların temiz bir yükseltme alması için bir migrasyon betiği ekleyin.

### Ne zaman migrasyon eklenmeli

- Skill dizinlerinin oluşturulma şekli değiştirildiğinde (sembolik bağlar vs gerçek dizinler)
- `~/.gstack/config.yaml` içindeki yapılandırma anahtarları yeniden adlandırıldığında veya taşındığında
- Önceki bir versiyondan artık dosyalar silinmesi gerektiğinde
- `~/.gstack/` durum dosyalarının formatı değiştirildiğinde

Migrasyon eklenmemeli: yeni özellikler (kullanıcılar otomatik olarak alır), yeni
skill'ler (setup keşfeder), veya yalnızca kod değişiklikleri (disk üzerinde durum yok).

### Nasıl eklenir

1. `gstack-upgrade/migrations/v{VERSION}.sh` oluşturun; `{VERSION}`, düzeltmeye
   ihtiyaç duyan release'in VERSION dosyasıyla eşleşir.
2. Çalıştırılabilir yapın: `chmod +x gstack-upgrade/migrations/v{VERSION}.sh`
3. Betik **idempotent** (birden fazla kez çalıştırılması güvenli) ve
   **ölümcül olmayan** (başarısızlıklar günlüklenir ama yükseltmeyi engellemez) olmalıdır.
4. Ne değiştiğini, migrasyonun neden gerekli olduğunu ve hangi kullanıcıların
   etkilendiğini açıklayan en üstte bir yorum bloğu ekleyin.

Örnek:

```bash
#!/usr/bin/env bash
# Migrasyon: v0.15.2.0 — Skill dizin yapısını düzelt
# Etkilenen: v0.15.2.0 öncesi --no-prefix ile kurulum yapan kullanıcılar
set -euo pipefail
SCRIPT_DIR="$(cd "$(dirname "$0")/../.." && pwd)"
"$SCRIPT_DIR/bin/gstack-relink" 2>/dev/null || true
```

### Nasıl çalışır

`/gstack-upgrade` sırasında, `./setup` tamamlandıktan sonra (Adım 4.75), yükseltme
skill'i `gstack-upgrade/migrations/` dizinini tarar ve kullanıcının eski versiyonundan
daha yeni olan her `v*.sh` betiğini çalıştırır. Betikler versiyon sırasına göre çalışır.
Başarısızlıklar günlüklenir ama asla yükseltmeyi engellemez.

### Migrasyonları test etme

Migrasyonlar `bun test` kapsamında test edilir (katman 1, ücretsiz). Test paketi,
`gstack-upgrade/migrations/` içindeki tüm migrasyon betiklerinin çalıştırılabilir olduğunu
ve sözdizimi hataları olmadan ayrıştırıldığını doğrular.

## Değişikliklerinizi ship etme

Skill düzenlemelerinizden memnun olduğunuzda:

```bash
/ship
```

Bu, test çalıştırır, diff'i inceler, Greptile yorumlarını triyaj eder (2 katmanlı eskalasyon ile), TODOS.md'yi yönetir, versiyonu yükseltir ve bir PR açar. Tam workflow için `ship/SKILL.md` dosyasına bakın.