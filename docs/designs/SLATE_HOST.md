# Slate Host Entegrasyonu — Araştırma & Tasarım Belgesi

**Tarih:** 2026-04-02
**Dal:** garrytan/slate-agent-support
**Durum:** Araştırma tamamlandı, host config yeniden düzenlemesine bağlı
**Yerine geçer:** Yok

## Slate Nedir

Slate, Random Labs'ten tescilli bir kodlama ajansı CLI'sidir.
Kurulum: `npm i -g @randomlabs/slate` veya `brew install anthropic/tap/slate`.
Lisans: Tescil. 85MB derlenmiş Bun ikili dosyası (arm64/x64, darwin/linux/windows).
npm paketi: `@randomlabs/slate@1.0.25` (ince 8.8KB başlatıcı + platforma özel isteğe bağlı bağımlılıklar).

Çoklu model: Claude Sonnet/Opus/Haiku ve diğer modelleri dinamik olarak seçer.
"Swarm orkestrasyonu" ve genişletilmiş çok saatlik oturumlar için inşa edilmiştir.

## Slate bir OpenCode çatalıdır

85MB Mach-O arm64 ikili dosyasının **ikili dize analizi ile doğrulandı:**

- İç ad: `name: "opencode"` (ikili dosyada kelimesi kelimesine dize)
- Tüm `OPENCODE_*` ortam değişkenleri `SLATE_*` karşılıklarıyla birlikte mevcut
- OpenCode'un araç/yetenek mimarisini, LSP entegrasyonunu, terminal yönetimini paylaşıyor
- Kendi markalaşması, API endpoint'leri (`api.randomlabs.ai`, `agent-worker-prod.randomlabs.workers.dev`) ve config yolları

Bu entegrasyon için önemli: OpenCode kuralları çoğunlukla geçerlidir, ancak Slate kendi yollarını ve ortam değişkenlerini ekler.

## Yetenek Keşfi (ikili dosyadan doğrulanmış)

Slate yetenekler için DÜRT dizin ailesinin hepsini tarar. İkili dosyadaki hata mesajları doğrular:

```
"failed .slate directory scan for skills"
"failed .claude directory scan for skills"
"failed .agents directory scan for skills"
"failed .opencode directory scan for skills"
```

**Keşif yolları (Slate belgelerinden öncelik sırası):**

1. `.slate/skills/<name>/SKILL.md` — proje düzeyinde, en yüksek öncelik
2. `~/.slate/skills/<name>/SKILL.md` — global
3. `.opencode/skills/`, `.agents/skills/` — uyumluluk geri dönüşü
4. `.claude/skills/` — Claude Code uyumluluk geri dönüşü (en düşük)
5. `slate.json` aracılığıyla özel yollar

**Glob kalıpları:** `**/SKILL.md` ve `{skill,skills}/**/SKILL.md`

**Komutlar:** Aynı dizin yapısı ama `commands/` alt dizinleri altında:
`/.slate/commands/`, `/.claude/commands/`, `/.agents/commands/`, `/.opencode/commands/`

**Yetenek ön bilgisinde:** `name` ve `description` alanları ile YAML (Slate belgelerine göre).
Her iki alan için belgelenmiş uzunluk sınırı yok.

## Proje Talimatları

Slate proje talimatları için `CLAUDE.md` ve `AGENTS.md`'yi okur.
Her iki kelimesi kelimesine dize de ikili dosyada doğrulanmış. Mevcut gstack projelerinde değişiklik gerekmiyor... CLAUDE.md olduğu gibi çalışıyor.

## Yapılandırma

**Config dosyası:** `slate.json` / `slate.jsonc` (opencode.json DEĞİL)

**Config seçenekleri (Slate belgelerinden):**
- `privacy` (boolean) — telemetri/günlüklemeyi devre dışı bırakır
- İzinler: araç başına `allow`, `ask`, `deny` (`read`, `edit`, `bash`, `grep`, `webfetch`, `websearch`, `*`)
- Model yuvaları: `models.main`, `models.subagents`, `models.search`, `models.reasoning`
- MCP sunucuları: özel komutlar ve başlıklarla yerel veya uzak
- Özel komutlar: şablonlarla `/commands`

Kurulum betiği `slate.json` oluşturmamalıdır. Kullanıcılar kendi izinlerini yapılandırır.

## CLI Bayrakları (Başsız Mod)

```
--stream-json / --output-format stream-json  — JSONL çıktısı, "Anthropic Claude Code SDK ile uyumlu"
--dangerously-skip-permissions               — tüm izin kontrollerini atla (CI/otomasyon)
--input-format stream-json                   — programlı giriş
-q                                           — etkileşimli olmayan mod
-w <dir>                                     — çalışma alanı dizini
--output-format text                         — düz metin çıktısı (varsayılan)
```

**Stream-JSON formatı:** Slate belgeleri "Anthropic Claude Code SDK ile uyumlu" iddia ediyor.
Henüz ampirik olarak doğrulanmadı. OpenCode mirası göz önüne alındığında, muhtemelen Claude Code'un
NDJSON olay şemasıyla eşleşiyor (type: "assistant", type: "tool_result", type: "result").

**Doğrulanması gereken:** Oturum çalıştırıcı ayrıştırıcısını inşa etmeden önce `slate -q "hello" --stream-json` komutunu geçerli kredilerle çalıştırın ve gerçek JSONL olaylarını yakalayın.

## Ortam Değişkenleri (ikili dizilerden)

### Slate'e özgü
```
SLATE_API_KEY                              — API anahtarı
SLATE_AGENT                                — ajans seçimi
SLATE_AUTO_SHARE                           — otomatik paylaşım ayarı
SLATE_CLIENT                               — istemci tanımlayıcısı
SLATE_CONFIG                               — config geçersiz kılma
SLATE_CONFIG_CONTENT                       — satır içi config
SLATE_CONFIG_DIR                           — config dizini
SLATE_DANGEROUSLY_SKIP_PERMISSIONS         — izinleri atla
SLATE_DIR                                  — veri dizini geçersiz kılma
SLATE_DISABLE_AUTOUPDATE                   — otomatik güncellemeyi devre dışı bırak
SLATE_DISABLE_CLAUDE_CODE                  — Claude Code entegrasyonunu tamamen devre dışı bırak
SLATE_DISABLE_CLAUDE_CODE_PROMPT           — Claude Code prompt yüklemeyi devre dışı bırak
SLATE_DISABLE_CLAUDE_CODE_SKILLS           — .claude/skills/ yüklemeyi devre dışı bırak
SLATE_DISABLE_DEFAULT_PLUGINS              — varsayılan eklentileri devre dışı bırak
SLATE_DISABLE_FILETIME_CHECK               — dosya zaman kontrollerini devre dışı bırak
SLATE_DISABLE_LSP_DOWNLOAD                 — LSP otomatik indirmeyi devre dışı bırak
SLATE_DISABLE_MODELS_FETCH                 — modeller config getirme devre dışı bırak
SLATE_DISABLE_PROJECT_CONFIG               — proje düzeyi config devre dışı bırak
SLATE_DISABLE_PRUNE                        — oturum budamayı devre dışı bırak
SLATE_DISABLE_TERMINAL_TITLE               — terminal başlık güncellemelerini devre dışı bırak
SLATE_ENABLE_EXA                           — Exa aramayı etkinleştir
SLATE_ENABLE_EXPERIMENTAL_MODELS           — deneysel modelleri etkinleştir
SLATE_EXPERIMENTAL                         — deneysel özellikleri etkinleştir
SLATE_EXPERIMENTAL_BASH_DEFAULT_TIMEOUT_MS — bash zaman aşımı geçersiz kılma
SLATE_EXPERIMENTAL_DISABLE_COPY_ON_SELECT  — seçimde kopyalamayı devre dışı bırak
SLATE_EXPERIMENTAL_DISABLE_FILEWATCHER     — dosya izleyiciyi devre dışı bırak
SLATE_EXPERIMENTAL_EXA                     — Exa arama (alternatif bayrak)
SLATE_EXPERIMENTAL_FILEWATCHER             — dosya izleyiciyi etkinleştir
SLATE_EXPERIMENTAL_ICON_DISCOVERY          — ikon keşfi
SLATE_EXPERIMENTAL_LSP_TOOL               — LSP aracı
SLATE_EXPERIMENTAL_LSP_TY                 — LSP tür kontrolü
SLATE_EXPERIMENTAL_MARKDOWN               — markdown modu
SLATE_EXPERIMENTAL_OUTPUT_TOKEN_MAX       — çıktı token sınırı
SLATE_EXPERIMENTAL_OXFMT                  — oxfmt entegrasyonu
SLATE_EXPERIMENTAL_PLAN_MODE              — plan modu
SLATE_FAKE_VCS                            — test için sahte VCS
SLATE_GIT_BASH_PATH                       — git bash yolu (Windows)
SLATE_MODELS_URL                          — modeller config URL'si
SLATE_PERMISSION                          — izin geçersiz kılma
SLATE_SERVER_PASSWORD                     — sunucu auth
SLATE_SERVER_USERNAME                     — sunucu auth
SLATE_TELEMETRY_DISABLED                  — telemetriyi devre dışı bırak
SLATE_TEST_HOME                           — test ana dizini
SLATE_TOKEN_DIR                           — token depolama dizini
```

### OpenCode mirası (hâlâ işlevsel)
```
OPENCODE_DISABLE_LSP_DOWNLOAD
OPENCODE_EXPERIMENTAL_DISABLE_FILEWATCHER
OPENCODE_EXPERIMENTAL_FILEWATCHER
OPENCODE_EXPERIMENTAL_ICON_DISCOVERY
OPENCODE_EXPERIMENTAL_LSP_TY
OPENCODE_EXPERIMENTAL_OXFMT
OPENCODE_FAKE_VCS
OPENCODE_GIT_BASH_PATH
OPENCODE_LIBC
OPENCODE_TERMINAL
```

### gstack entegrasyonu için kritik ortam değişkenleri

**`SLATE_DISABLE_CLAUDE_CODE_SKILLS`** — Ayarlandığında, `.claude/skills/` yüklemesi devre dışı bırakılır.
Bu, `.slate/skills/` yayınlamayı sadece bir optimizasyondan daha fazla, kritik hale getirir.
Yerel `.slate/` yayınlama olmadan, bu bayrak ayarlandığında gstack yetenekleri kaybolur.

**`SLATE_TEST_HOME`** — Uçtan uca testler için kullanışlı. Slate'in ana dizinini yalıtılmış bir geçici dizine yönlendirebilir, Codex testlerinin geçici HOME kullanmasına benzer şekilde.

**`SLATE_DANGEROUSLY_SKIP_PERMISSIONS`** — Başsız uçtan uca testler için gerekli.

## Model Referansları (ikili dosyadan)

```
anthropic/claude-sonnet-4.6
anthropic/claude-opus-4
anthropic/claude-haiku-4
anthropic/slate              — Slate'in kendi model yönlendirmesi
openai/gpt-5.3-codex
google/nano-banana
randomlabs/fast-default-alpha
```

## API Endpoint'leri (ikili dosyadan)

```
https://api.randomlabs.ai                          — ana API
https://api.randomlabs.ai/exaproxy                 — Exa arama proxy'si
https://agent-worker-prod.randomlabs.workers.dev   — üretim çalışanı
https://agent-worker-dev.randomlabs.workers.dev    — geliştirme çalışanı
https://dashboard.randomlabs.ai                    — pano
https://docs.randomlabs.ai                         — belgeler
https://randomlabs.ai/config.json                  — uzaktan config
```

Brew tap: `anthropic/tap/slate` (dikkat çekici: Random Labs'ın değil, Anthropic'ın tap'ı altında)

## npm Paket Yapısı

```
@randomlabs/slate (8.8 kB, ince başlatıcı)
├── bin/slate           — Node.js başlatıcı (platform ikili dosyasını node_modules içinde bulur)
├── bin/slate1          — Bun başlatıcı (aynı mantık, import.meta.filename)
├── postinstall.mjs     — Platform ikili dosyasının varlığını doğrular, gerekirse sembolik bağ oluşturur
└── package.json        — Tüm platformlar için optionalDependencies bildirir

Platform paketleri (her biri 85MB):
├── @randomlabs/slate-darwin-arm64
├── @randomlabs/slate-darwin-x64
├── @randomlabs/slate-linux-arm64
├── @randomlabs/slate-linux-x64
├── @randomlabs/slate-linux-x64-musl
├── @randomlabs/slate-linux-arm64-musl
├── @randomlabs/slate-linux-x64-baseline
├── @randomlabs/slate-linux-x64-baseline-musl
├── @randomlabs/slate-darwin-x64-baseline
├── @randomlabs/slate-windows-x64
└── @randomlabs/slate-windows-x64-baseline
```

İkili dosya geçersiz kılma: `SLATE_BIN_PATH` ortam değişkeni tüm keşfi atlar ve belirtilen ikili dosyayı doğrudan çalıştırır.

## Bugün Zaten Çalışan Şey

gstack yetenekleri `.claude/skills/` geri dönüş yolu üzerinden Slate'te zaten çalışıyor.
Temel işlevsellik için değişiklik gerekmiyor. Claude Code için gstack yükleyen ve ayrıca Slate kullanan kullanıcılar, yeteneklerini her iki ajansında da mevcut bulacaktır.

## Birinci Sınıf Destek Ne Ekler

1. **Güvenilirlik** — `.slate/skills/` Slate'in en yüksek öncelikli yoludur. `SLATE_DISABLE_CLAUDE_CODE_SKILLS`'ten etkilenmez.
2. **Optimize edilmiş ön bilgi** — Slate'in kullanmadığı Claude'a özgü alanları (allowed-tools, hooks, version) çıkarır. Sadece `name` ve `description` saklanır.
3. **Kurulum betiği** — `slate` ikili dosyasını otomatik algılar, yetenekleri `~/.slate/skills/` konumuna kurar.
4. **Uçtan uca testler** — Yeteneklerin Slate tarafından doğrudan çağrıldığında çalıştığını doğrular.

## Bağlı: Host Config Yeniden Düzenlemesi

Codex'in dış ses incelemesi, Slate'i 4. host (Claude, Codex, Factory'den sonra) olarak eklemenin "bir yol diğer adı için host patlaması" olduğunu belirledi. Mevcut mimari şunlara sahip:

- `type Host = 'claude' | 'codex' | 'factory` içinde sabit kodlu host adları
- Yakın kopya mantıkla `transformFrontmatter()`
- Benzer kalıplarla `EXTERNAL_HOST_CONFIG` içinde host başına config
- Kurulum betiğinde host başına işlevler (`create_codex_runtime_root`, `link_codex_skill_dirs`)
- `bin/gstack-platform-detect`, `bin/gstack-uninstall`, `bin/dev-setup` içinde çoğaltılan host adları

Slate eklemek, bu kalıpların hepsinin kopyalanması demek. Hostları veri güdümlü (if/else dalları yerine config nesneleri) yapan bir yeniden düzenleme, Slate entegrasyonunu önemsiz hale getirir VE gelecekteki hostları (herhangi bir yeni OpenCode çatalı, herhangi bir yeni ajans) sıfır çabayla yapar.

### Plandan eksik (Codex tarafından tanımlandı)

- `lib/worktree.ts` sadece `.agents/` kopyalar, `.slate/` değil — worktree'lerdeki uçtan uca testlerde Slate yetenekleri olmayacak
- `bin/gstack-uninstall` `.slate/` bilmiyor
- `bin/dev-setup` katılımcı geliştirme modu için `.slate/` bağlamıyor
- `bin/gstack-platform-detect` Slate'i algılamıyor
- Uçtan uca testler `.slate/` yolunun gerçekten çalıştığını kanıtlamak için `SLATE_DISABLE_CLAUDE_CODE_SKILLS=1` ayarlamalıdır (sadece `.claude/`'a geri dönmek değil)

## Oturum Çalıştırıcı Tasarımı (daha sonra)

JSONL formatı doğrulandığında, oturum çalıştırıcısı şunları yapmalıdır:

- Başlat: `slate -q "<prompt>" --stream-json --dangerously-skip-permissions -w <dir>`
- Ayrıştır: Claude Code SDK uyumlu NDJSON (varsayılan, doğrulanması gerekiyor)
- Yetenekler: Test fixture'ında `.slate/skills/` konumuna kur (`.claude/skills/` değil)
- Auth: `SLATE_API_KEY` veya mevcut `~/.slate/` kimlik bilgilerini kullan
- İzolasyon: Ana dizin izolasyonu için `SLATE_TEST_HOME` kullan
- Zaman aşımı: 300s varsayılan (Codex ile aynı)

```typescript
export interface SlateResult {
  output: string;
  toolCalls: string[];
  tokens: number;
  exitCode: number;
  durationMs: number;
  sessionId: string | null;
  rawLines: string[];
  stderr: string;
}
```

## Belgeler Referansları

- Slate belgeleri: https://docs.randomlabs.ai
- Hızlı başlangıç: https://docs.randomlabs.ai/en/getting-started/quickstart
- Yetenekler: https://docs.randomlabs.ai/en/using-slate/skills
- Yapılandırma: https://docs.randomlabs.ai/en/using-slate/configuration
- Kısayol tuşları: https://docs.randomlabs.ai/en/using-slate/hotkey_reference