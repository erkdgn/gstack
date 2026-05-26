---
name: make-pdf
preamble-tier: 1
version: 1.0.0
description: |
  Herhangi bir markdown dosyasını yayıncılık kalitesinde PDF'e dönüştürür. Uygun 1 inç
  kenar boşlukları, akıllı sayfa sonları, sayfa numaraları, kapak sayfaları, sürekli
  üstbilgiler, kıvrımlı tırnak işaretleri ve em tireler, tıklanabilir İÇİNDEKİLER,
  çapraz TASLAK filigranı. Bir taslak yapıt değil — bitmiş bir yapıt. "PDF yap",
  "PDF'e aktar", "bu markdownu PDF'e dönüştür" veya "belge oluştur" isteklerinde
  kullanılır. (gstack)
  Ses tetikleyicileri (konuşmadan metne takma adlar): "bunu pdfe çevir", "pdfe çevir", "pdfe aktar", "bunu pdfe dönüştür", "bu markdownu pdfe dönüştür", "pdf oluştur", "bunu pdfe yap", "bu markdownu pdfe yap".
triggers:
  - markdowndan pdf
  - pdf oluştur
  - pdf yap
  - pdf aktar
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — doğrudan düzenlemeyin -->
<!-- Yeniden oluştur: bun run gen:skill-docs -->

## Önsöz (önce çalıştır)

```bash
_UPD=$(~/.claude/skills/gstack/bin/gstack-update-check 2>/dev/null || .claude/skills/gstack/bin/gstack-update-check 2>/dev/null || true)
[ -n "$_UPD" ] && echo "$_UPD" || true
mkdir -p ~/.gstack/sessions
touch ~/.gstack/sessions/"$PPID"
_SESSIONS=$(find ~/.gstack/sessions -mmin -120 -type f 2>/dev/null | wc -l | tr -d ' ')
find ~/.gstack/sessions -mmin +120 -type f -exec rm {} + 2>/dev/null || true
_PROACTIVE=$(~/.claude/skills/gstack/bin/gstack-config get proactive 2>/dev/null || echo "true")
_PROACTIVE_PROMPTED=$([ -f ~/.gstack/.proactive-prompted ] && echo "yes" || echo "no")
_BRANCH=$(git branch --show-current 2>/dev/null || echo "unknown")
echo "BRANCH: $_BRANCH"
_SKILL_PREFIX=$(~/.claude/skills/gstack/bin/gstack-config get skill_prefix 2>/dev/null || echo "false")
echo "PROACTIVE: $_PROACTIVE"
echo "PROACTIVE_PROMPTED: $_PROACTIVE_PROMPTED"
echo "SKILL_PREFIX: $_SKILL_PREFIX"
source <(~/.claude/skills/gstack/bin/gstack-repo-mode 2>/dev/null) || true
REPO_MODE=${REPO_MODE:-unknown}
echo "REPO_MODE: $REPO_MODE"
_LAKE_SEEN=$([ -f ~/.gstack/.completeness-intro-seen ] && echo "yes" || echo "no")
echo "LAKE_INTRO: $_LAKE_SEEN"
_TEL=$(~/.claude/skills/gstack/bin/gstack-config get telemetry 2>/dev/null || true)
_TEL_PROMPTED=$([ -f ~/.gstack/.telemetry-prompted ] && echo "yes" || echo "no")
_TEL_START=$(date +%s)
_SESSION_ID="$$-$(date +%s)"
echo "TELEMETRY: ${_TEL:-off}"
echo "TEL_PROMPTED: $_TEL_PROMPTED"
_EXPLAIN_LEVEL=$(~/.claude/skills/gstack/bin/gstack-config get explain_level 2>/dev/null || echo "default")
if [ "$_EXPLAIN_LEVEL" != "default" ] && [ "$_EXPLAIN_LEVEL" != "terse" ]; then _EXPLAIN_LEVEL="default"; fi
echo "EXPLAIN_LEVEL: $_EXPLAIN_LEVEL"
_QUESTION_TUNING=$(~/.claude/skills/gstack/bin/gstack-config get question_tuning 2>/dev/null || echo "false")
echo "QUESTION_TUNING: $_QUESTION_TUNING"
mkdir -p ~/.gstack/analytics
if [ "$_TEL" != "off" ]; then
echo '{"skill":"make-pdf","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
fi
for _PF in $(find ~/.gstack/analytics -maxdepth 1 -name '.pending-*' 2>/dev/null); do
  if [ -f "$_PF" ]; then
    if [ "$_TEL" != "off" ] && [ -x "~/.claude/skills/gstack/bin/gstack-telemetry-log" ]; then
      ~/.claude/skills/gstack/bin/gstack-telemetry-log --event-type skill_run --skill _pending_finalize --outcome unknown --session-id "$_SESSION_ID" 2>/dev/null || true
    fi
    rm -f "$_PF" 2>/dev/null || true
  fi
  break
done
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)" 2>/dev/null || true
_LEARN_FILE="${GSTACK_HOME:-$HOME/.gstack}/projects/${SLUG:-unknown}/learnings.jsonl"
if [ -f "$_LEARN_FILE" ]; then
  _LEARN_COUNT=$(wc -l < "$_LEARN_FILE" 2>/dev/null | tr -d ' ')
  echo "LEARNINGS: $_LEARN_COUNT entries loaded"
  if [ "$_LEARN_COUNT" -gt 5 ] 2>/dev/null; then
    ~/.claude/skills/gstack/bin/gstack-learnings-search --limit 3 2>/dev/null || true
  fi
else
  echo "LEARNINGS: 0"
fi
~/.claude/skills/gstack/bin/gstack-timeline-log '{"skill":"make-pdf","event":"started","branch":"'"$_BRANCH"'","session":"'"$_SESSION_ID"'"}' 2>/dev/null &
_HAS_ROUTING="no"
if [ -f CLAUDE.md ] && grep -q "## Skill routing" CLAUDE.md 2>/dev/null; then
  _HAS_ROUTING="yes"
fi
_ROUTING_DECLINED=$(~/.claude/skills/gstack/bin/gstack-config get routing_declined 2>/dev/null || echo "false")
echo "HAS_ROUTING: $_HAS_ROUTING"
echo "ROUTING_DECLINED: $_ROUTING_DECLINED"
_VENDORED="no"
if [ -d ".claude/skills/gstack" ] && [ ! -L ".claude/skills/gstack" ]; then
  if [ -f ".claude/skills/gstack/VERSION" ] || [ -d ".claude/skills/gstack/.git" ]; then
    _VENDORED="yes"
  fi
fi
echo "VENDORED_GSTACK: $_VENDORED"
echo "MODEL_OVERLAY: claude"
_CHECKPOINT_MODE=$(~/.claude/skills/gstack/bin/gstack-config get checkpoint_mode 2>/dev/null || echo "explicit")
_CHECKPOINT_PUSH=$(~/.claude/skills/gstack/bin/gstack-config get checkpoint_push 2>/dev/null || echo "false")
echo "CHECKPOINT_MODE: $_CHECKPOINT_MODE"
echo "CHECKPOINT_PUSH: $_CHECKPOINT_PUSH"
[ -n "$OPENCLAW_SESSION" ] && echo "SPAWNED_SESSION: true" || true
```

## MAKE-PDF KURULUM (herhangi bir make-pdf komutundan ÖNCE bu kontrolü çalıştır)

```bash
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
P=""
[ -n "$MAKE_PDF_BIN" ] && [ -x "$MAKE_PDF_BIN" ] && P="$MAKE_PDF_BIN"
[ -z "$P" ] && [ -n "$_ROOT" ] && [ -x "$_ROOT/.claude/skills/gstack/make-pdf/dist/pdf" ] && P="$_ROOT/.claude/skills/gstack/make-pdf/dist/pdf"
[ -z "$P" ] && P="$HOME/.claude/skills/gstack/make-pdf/dist/pdf"
if [ -x "$P" ]; then
  echo "MAKE_PDF_READY: $P"
  alias _p_="$P"   # shellcheck alias yardımcısı (dışa aktarılmaz)
  export P   # aynı skill çağrısındaki sonraki bloklarda $P olarak kullanılabilir
else
  echo "MAKE_PDF_NOT_AVAILABLE (build etmek için gstack reposunda './setup' çalıştırın)"
fi
```

Eğer `MAKE_PDF_NOT_AVAILABLE` yazdırılırsa: kullanıcıya ikili dosyanın build
edilmediğini söyleyin. gstack reposundan `./setup` çalıştırmalarını isteyin, ardından tekrar deneyin.

Eğer `MAKE_PDF_READY` yazdırılırsa: `$P`, skill'in geri kalanı için
ikili dosya yoludur. Skill gövdesinin taşınabilir kalması için açık bir yol yerine `$P` kullanın.

Temel komutlar:
- `$P generate <input.md> [output.pdf]` — markdownu PDF'e dönüştür (%80 kullanım durumu)
- `$P generate --cover --toc essay.md out.pdf` — tam yayıncılık düzeni
- `$P generate --watermark DRAFT memo.md draft.pdf` — çapraz TASLAK filigranı
- `$P preview <input.md>` — HTML oluşturur ve tarayıcıda açar (hızlı yineleme)
- `$P setup` — browse + Chromium + pdftotext doğrulaması yapar ve bir duman testi çalıştırır
- `$P --help` — tam bayrak referansı

Çıktı sözleşmesi:
- `stdout`: Başarı durumunda yalnızca çıktı yolu. Bir satır.
- `stderr`: İlerleme (`Rendering HTML... Generating PDF...`) `--quiet` olmadıkça.
- Çıkış 0 başarı / 1 hatalı argümanlar / 2 oluşturma hatası / 3 Paged.js zaman aşımı / 4 browse kullanılamıyor.

## Plan Modu Güvenli İşlemleri

Plan modunda, planı bilgilendirdikleri için izinlidir: `$B`, `$D`, `codex exec`/`codex review`, `~/.gstack/` yazmaları, plan dosyasına yazmalar ve oluşturulan yapılar için `open`.

## Plan Modunda Skill Çağırma

Kullanıcı plan modunda bir skill çağırırsa, skill genel plan modu davranışına göre öncelik alır. **Skill dosyasını referans değil, çalıştırılabilir talimat olarak ele alın.** Adım 0'dan başlayarak adım adım izleyin; ilk AskUserQuestion, iş akışının plan moduna girmesidir, plan modunu ihlal değil. AskUserQuestion (herhangi bir varyant — `mcp__*__AskUserQuestion` veya yerel; "AskUserQuestion Format → Tool resolution" sayfasına bakın) plan modunun tur sonu gereksinimini karşılar. Çağrılabilir varyant yoksa, skill BLOCKED — AskUserQuestion Format kuralına göre `BLOCKED — AskUserQuestion unavailable` olarak raporlayın ve durun. STOP noktasında hemen durun. İş akışına devam etmeyin veya ExitPlanMode'u çağırmayın. "PLAN MODE EXCEPTION — ALWAYS RUN" olarak işaretlenen komutlar çalıştırılır. ExitPlanMode'u yalnızca skill iş akışı tamamlandıktan sonra veya kullanıcı skill'i iptal etmesini veya plan modundan çıkmasını söyledikten sonra çağırın.

Eğer `PROACTIVE` `"false"` ise, skill'leri otomatik olarak çağırmayın veya proaktif olarak önermeyin. Bir skill yararlı görünüyorsa, sorun: "Sanırım /skillname burada yardımcı olabilir — çalıştırmamı ister misiniz?"

Eğer `SKILL_PREFIX` `"true"` ise, `/gstack-*` adlarını öner/çağır. Disk yolları `~/.claude/skills/gstack/[skill-name]/SKILL.md` olarak kalır.

Eğer çıktıda `UPGRADE_AVAILABLE <old> <new>` görünüyorsa: `~/.claude/skills/gstack/gstack-upgrade/SKILL.md` dosyasını okuyun ve "Satır içi yükseltme akışı"nı izleyin (yapılandırılmışsa otomatik yükselt, aksi takdirde 4 seçenekli AskUserQuestion, reddedilirse snooze durumu yaz).

Eğer çıktıda `JUST_UPGRADED <from> <to>` görünüyorsa: "gstack v{to} çalıştırılıyor (az önce güncellendi!)" yazdır. Eğer `SPAWNED_SESSION` true ise, özellik keşfini atlayın.

Özellik keşfi, oturum başına en fazla bir istem:
- `~/.claude/skills/gstack/.feature-prompted-continuous-checkpoint` eksik: Sürekli checkpoint otomatik commit'leri için AskUserQuestion. Kabul edilirse, `~/.claude/skills/gstack/bin/gstack-config set checkpoint_mode continuous` çalıştırın. Her zaman işaretleyiciyi dokunarak oluşturun.
- `~/.claude/skills/gstack/.feature-prompted-model-overlay` eksik: "Model overlay'leri aktif. MODEL_OVERLAY yamayı gösterir." bilgisini verin. Her zaman işaretleyiciyi dokunarak oluşturun.

Yükseltme istemlerinden sonra iş akışına devam edin.

Eğer `WRITING_STYLE_PENDING` `yes` ise: yazım tarzı hakkında bir kez sorun:

> v1 istemleri daha basit: ilk kullanımda jargon tanımları, sonuç-çerçeveli sorular, daha kısa düzyazı. Varsayılanı koruyun mu yoksa terse geri dönelim mi?

Seçenekler:
- A) Yeni varsayılanı koru (önerilen — iyi yazım herkese yardımcı olur)
- B) V0 düzyazısına geri dön — `explain_level: terse` ayarla

A ise: `explain_level` ayarını değiştirmeden bırakın (varsayılan `default` olur).
B ise: `~/.claude/skills/gstack/bin/gstack-config set explain_level terse` çalıştırın.

Her zaman çalıştırın (seçimden bağımsız olarak):
```bash
rm -f ~/.gstack/.writing-style-prompt-pending
touch ~/.gstack/.writing-style-prompted
```

Eğer `WRITING_STYLE_PENDING` `no` ise atlayın.

Eğer `LAKE_INTRO` `no` ise: "gstack **Boil the Lake** ilkesini izler — AI marjinal maliyeti sıfıra yaklaştığında eksiksiz olanı yapın. Daha fazla: https://garryslist.org/posts/boil-the-ocean" deyin. Açmayı teklif edin:

```bash
open https://garryslist.org/posts/boil-the-ocean
touch ~/.gstack/.completeness-intro-seen
```

`open` komutunu yalnızca evet ise çalıştırın. Her zaman `touch` çalıştırın.

Eğer `TEL_PROMPTED` `no` ise VE `LAKE_INTRO` `yes` ise: telemetriyi bir kez AskUserQuestion ile sorun:

> gstack'in daha iyi olmasına yardımcı olun. Yalnızca kullanım verilerini paylaşın: skill, süre, çökmeler, kararlı cihaz kimliği. Kod, dosya yolu veya repo adı yok.

Seçenekler:
- A) gstack'in daha iyi olmasına yardımcı ol! (önerilen)
- B) Hayır teşekkürler

A ise: `~/.claude/skills/gstack/bin/gstack-config set telemetry community` çalıştırın

B ise: takip sorusu sorun:

> Anonim mod yalnızca toplu kullanım gönderir, benzersiz kimlik yok.

Seçenekler:
- A) Tabii, anonim sorun değil
- B) Hayır teşekkürler, tamamen kapalı

B→A ise: `~/.claude/skills/gstack/bin/gstack-config set telemetry anonymous` çalıştırın
B→B ise: `~/.claude/skills/gstack/bin/gstack-config set telemetry off` çalıştırın

Her zaman çalıştırın:
```bash
touch ~/.gstack/.telemetry-prompted
```

Eğer `TEL_PROMPTED` `yes` ise atlayın.

Eğer `PROACTIVE_PROMPTED` `no` ise VE `TEL_PROMPTED` `yes` ise: bir kez sorun:

> gstack skill'leri proaktif olarak önersin mi, örneğin /qa "bu çalışıyor mu?" için veya /investigate hatalar için?

Seçenekler:
- A) Açık tut (önerilen)
- B) Kapat — /komutları kendim yazacağım

A ise: `~/.claude/skills/gstack/bin/gstack-config set proactive true` çalıştırın
B ise: `~/.claude/skills/gstack/bin/gstack-config set proactive false` çalıştırın

Her zaman çalıştırın:
```bash
touch ~/.gstack/.proactive-prompted
```

Eğer `PROACTIVE_PROMPTED` `yes` ise atlayın.

Eğer `HAS_ROUTING` `no` ise VE `ROUTING_DECLINED` `false` ise VE `PROACTIVE_PROMPTED` `yes` ise:
Proje kökünde bir CLAUDE.md dosyası olup olmadığını kontrol edin. Yoksa, oluşturun.

AskUserQuestion kullanın:

> gstack, projenizin CLAUDE.md dosyası skill yönlendirme kuralları içerdiğinde en iyi şekilde çalışır.

Seçenekler:
- A) CLAUDE.md'ye yönlendirme kuralları ekle (önerilen)
- B) Hayır teşekkürler, skill'leri manuel olarak çağıracağım

A ise: Bu bölümü CLAUDE.md'nin sonuna ekleyin:

```markdown

## Skill routing

Kullanıcının isteği mevcut bir skill ile eşleştiğinde, Skill aracı üzerinden çağırın. Şüpheli olduğunuzda skill'i çağırın.

Temel yönlendirme kuralları:
- Ürün fikirleri/beyin fırtınası → /office-hours çağır
- Strateji/kapsam → /plan-ceo-review çağır
- Mimari → /plan-eng-review çağır
- Tasarım sistemi/plan incelemesi → /design-consultation veya /plan-design-review çağır
- Tam inceleme boru hattı → /autoplan çağır
- Hatalar/hatalar → /investigate çağır
- QA/test site davranışı → /qa veya /qa-only çağır
- Kod incelemesi/diff kontrolü → /review çağır
- Görsel cilalama → /design-review çağır
- Gönder/dağıt/PR → /ship veya /land-and-deploy çağır
- İlerlemeyi kaydet → /context-save çağır
- Bağlamı geri yükle → /context-restore çağır
```

Ardından değişikliği commit edin: `git add CLAUDE.md && git commit -m "chore: gstack skill yönlendirme kurallarını CLAUDE.md'ye ekle"`

B ise: `~/.claude/skills/gstack/bin/gstack-config set routing_declined true` çalıştırın ve `gstack-config set routing_declined false` ile yeniden etkinleştirebileceklerini söyleyin.

Bu proje başına yalnızca bir kez gerçekleşir. `HAS_ROUTING` `yes` veya `ROUTING_DECLINED` `true` ise atlayın.

Eğer `VENDORED_GSTACK` `yes` ise, `~/.gstack/.vendoring-warned-$SLUG` mevcut değilse AskUserQuestion ile bir kez uyarın:

> Bu projede gstack `.claude/skills/gstack/` içinde vendored olarak bulunuyor. Vendoring kullanımdan kaldırılmıştır.
> Team moduna geçiş yapılsın mı?

Seçenekler:
- A) Evet, şimdi team moduna geç
- B) Hayır, kendim hallederim

A ise:
1. `git rm -r .claude/skills/gstack/` çalıştırın
2. `echo '.claude/skills/gstack/' >> .gitignore` çalıştırın
3. `~/.claude/skills/gstack/bin/gstack-team-init required` çalıştırın (veya `optional`)
4. `git add .claude/ .gitignore CLAUDE.md && git commit -m "chore: gstack'i vendored modundan team moduna geçir"` çalıştırın
5. Kullanıcıya şunu söyleyin: "Tamamlandı. Her geliştirici şimdi çalıştırıyor: `cd ~/.claude/skills/gstack && ./setup --team`"

B ise: "Tamam, vendored kopyayı güncel tutmak size kalır." deyin.

Her zaman çalıştırın (seçimden bağımsız olarak):
```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)" 2>/dev/null || true
touch ~/.gstack/.vendoring-warned-${SLUG:-unknown}
```

İşaretleyici mevcutsa atlayın.

Eğer `SPAWNED_SESSION` `"true"` ise, bir AI orkestratörü (ör. OpenClaw) tarafından oluşturulan bir oturumda çalışıyorsunuz. Oluşturulan oturumlarda:
- Etkileşimli istemler için AskUserQuestion KULLANMAYIN. Önerilen seçeneği otomatik olarak seçin.
- Yükseltme kontrollerini, telemetri istemlerini, yönlendirme enjeksiyonunu veya lake girişini ÇALIŞTIRMAYIN.
- Görevi tamamlamaya ve sonuçları düz metin çıktısı ile raporlamaya odaklanın.
- Bir tamamlama raporu ile bitirin: ne gönderildi, alınan kararlar, belirsiz olan her şey.

## Artifacts Sync (skill başlangıcı)

```bash
_GSTACK_HOME="${GSTACK_HOME:-$HOME/.gstack}"
# v1.27.0.0 artifacts dosyasını tercih et; geçiş betiği çalışmadan önce
# ara sıra yükseltme yapan kullanıcılar için brain dosyasına geri dön.
if [ -f "$HOME/.gstack-artifacts-remote.txt" ]; then
  _BRAIN_REMOTE_FILE="$HOME/.gstack-artifacts-remote.txt"
else
  _BRAIN_REMOTE_FILE="$HOME/.gstack-brain-remote.txt"
fi
_BRAIN_SYNC_BIN="~/.claude/skills/gstack/bin/gstack-brain-sync"
_BRAIN_CONFIG_BIN="~/.claude/skills/gstack/bin/gstack-config"

# /sync-gbrain context-load: gbrain mevcut olduğunda aracın kullanmasını sağla.
# Worktree başına pin: spike sonrası yeniden tasarım, sorguları kapsamlandırmak için
# git toplevel'daki kubectl tarzı `.gbrain-source` kullanır. Pini worktree'de arayın
# (global bir durum dosyasında değil), böylece pini olmayan B worktree'sini açmak "indexlenmiş"
# iddiasında bulunmaz, çünkü A worktree'si senkronize edilmiştir. gbrain yapılandırılmadığında
# boş dize (gbrain kullanmayan kullanıcılar için sıfır bağlam maliyeti).
_GBRAIN_CONFIG="$HOME/.gbrain/config.json"
if [ -f "$_GBRAIN_CONFIG" ] && command -v gbrain >/dev/null 2>&1; then
  _GBRAIN_VERSION_OK=$(gbrain --version 2>/dev/null | grep -c '^gbrain ' || echo 0)
  if [ "$_GBRAIN_VERSION_OK" -gt 0 ] 2>/dev/null; then
    _GBRAIN_PIN_PATH=""
    _REPO_TOP=$(git rev-parse --show-toplevel 2>/dev/null || echo "")
    if [ -n "$_REPO_TOP" ] && [ -f "$_REPO_TOP/.gbrain-source" ]; then
      _GBRAIN_PIN_PATH="$_REPO_TOP/.gbrain-source"
    fi
    if [ -n "$_GBRAIN_PIN_PATH" ]; then
      echo "GBrain configured. Prefer \`gbrain search\`/\`gbrain query\` over Grep for"
      echo "semantic questions; use \`gbrain code-def\`/\`code-refs\`/\`code-callers\` for"
      echo "symbol-aware code lookup. See \"## GBrain Search Guidance\" in CLAUDE.md."
      echo "Run /sync-gbrain to refresh."
    else
      echo "GBrain configured but this worktree isn't pinned yet. Run \`/sync-gbrain --full\`"
      echo "before relying on \`gbrain search\` for code questions in this worktree."
      echo "Falls back to Grep until pinned."
    fi
  fi
fi

_BRAIN_SYNC_MODE=$("$_BRAIN_CONFIG_BIN" get artifacts_sync_mode 2>/dev/null || echo off)

# Uzak-MCP modunu algıla (/setup-gbrain'in 4. Yolu). Yerel artifacts sync
# uzak modda no-op'tur; brain sunucusu GitHub/GitLab'den kendi zamanlamasıyla çeker.
# Bu önsözü hızlı tutmak için claude.json'ı doğrudan okuyun (her skill başlangıcında
# claude CLI alt süreci yok).
_GBRAIN_MCP_MODE="none"
if command -v jq >/dev/null 2>&1 && [ -f "$HOME/.claude.json" ]; then
  _GBRAIN_MCP_TYPE=$(jq -r '.mcpServers.gbrain.type // .mcpServers.gbrain.transport // empty' "$HOME/.claude.json" 2>/dev/null)
  case "$_GBRAIN_MCP_TYPE" in
    url|http|sse) _GBRAIN_MCP_MODE="remote-http" ;;
    stdio) _GBRAIN_MCP_MODE="local-stdio" ;;
  esac
fi

if [ -f "$_BRAIN_REMOTE_FILE" ] && [ ! -d "$_GSTACK_HOME/.git" ] && [ "$_BRAIN_SYNC_MODE" = "off" ]; then
  _BRAIN_NEW_URL=$(head -1 "$_BRAIN_REMOTE_FILE" 2>/dev/null | tr -d '[:space:]')
  if [ -n "$_BRAIN_NEW_URL" ]; then
    echo "ARTIFACTS_SYNC: artifacts repo detected: $_BRAIN_NEW_URL"
    echo "ARTIFACTS_SYNC: run 'gstack-brain-restore' to pull your cross-machine artifacts (or 'gstack-config set artifacts_sync_mode off' to dismiss forever)"
  fi
fi

if [ -d "$_GSTACK_HOME/.git" ] && [ "$_BRAIN_SYNC_MODE" != "off" ]; then
  _BRAIN_LAST_PULL_FILE="$_GSTACK_HOME/.brain-last-pull"
  _BRAIN_NOW=$(date +%s)
  _BRAIN_DO_PULL=1
  if [ -f "$_BRAIN_LAST_PULL_FILE" ]; then
    _BRAIN_LAST=$(cat "$_BRAIN_LAST_PULL_FILE" 2>/dev/null || echo 0)
    _BRAIN_AGE=$(( _BRAIN_NOW - _BRAIN_LAST ))
    [ "$_BRAIN_AGE" -lt 86400 ] && _BRAIN_DO_PULL=0
  fi
  if [ "$_BRAIN_DO_PULL" = "1" ]; then
    ( cd "$_GSTACK_HOME" && git fetch origin >/dev/null 2>&1 && git merge --ff-only "origin/$(git rev-parse --abbrev-ref HEAD)" >/dev/null 2>&1 ) || true
    echo "$_BRAIN_NOW" > "$_BRAIN_LAST_PULL_FILE"
  fi
  "$_BRAIN_SYNC_BIN" --once 2>/dev/null || true
fi

if [ "$_GBRAIN_MCP_MODE" = "remote-http" ]; then
  # Uzak-MCP modu: yerel artifacts sync no-op (brain yöneticisinin sunucusu
  # GitHub/GitLab'den çeker). Kullanıcıya bunun tasarım gereği olduğunu, bozuk olmadığını göster.
  _GBRAIN_HOST=$(jq -r '.mcpServers.gbrain.url // empty' "$HOME/.claude.json" 2>/dev/null | sed -E 's|^https?://([^/:]+).*|\1|')
  echo "ARTIFACTS_SYNC: remote-mode (managed by brain server ${_GBRAIN_HOST:-remote})"
elif [ -d "$_GSTACK_HOME/.git" ] && [ "$_BRAIN_SYNC_MODE" != "off" ]; then
  _BRAIN_QUEUE_DEPTH=0
  [ -f "$_GSTACK_HOME/.brain-queue.jsonl" ] && _BRAIN_QUEUE_DEPTH=$(wc -l < "$_GSTACK_HOME/.brain-queue.jsonl" | tr -d ' ')
  _BRAIN_LAST_PUSH="never"
  [ -f "$_GSTACK_HOME/.brain-last-push" ] && _BRAIN_LAST_PUSH=$(cat "$_GSTACK_HOME/.brain-last-push" 2>/dev/null || echo never)
  echo "ARTIFACTS_SYNC: mode=$_BRAIN_SYNC_MODE | last_push=$_BRAIN_LAST_PUSH | queue=$_BRAIN_QUEUE_DEPTH"
else
  echo "ARTIFACTS_SYNC: off"
fi
```



Gizlilik durdurma kapısı: eğer çıktıda `ARTIFACTS_SYNC: off` görünüyorsa, `artifacts_sync_mode_prompted` `false` ise ve gbrain PATH'te veya `gbrain doctor --fast --json` çalışıyorsa, bir kez sorun:

> gstack, yapıtlarınızı (CEO planları, tasarımlar, raporlar) makineler arası indeksleyen GBrain'e sahip özel bir GitHub reposuna yayınlayabilir. Ne kadar sync edilmeli?

Seçenekler:
- A) Her şey izin verilenler listesinde (önerilen)
- B) Yalnızca yapıtlar
- C) Reddet, her şeyi yerel tut

Cevaptan sonra:

```bash
# Seçilen mod: full | artifacts-only | off
"$_BRAIN_CONFIG_BIN" set artifacts_sync_mode <choice>
"$_BRAIN_CONFIG_BIN" set artifacts_sync_mode_prompted true
```

A/B ise ve `~/.gstack/.git` eksikse, `gstack-artifacts-init` çalıştırılıp çalıştırılmayacağını sorun. Skill'i engellemeyin.

Skill SONUNDA telemetriden önce:

```bash
"~/.claude/skills/gstack/bin/gstack-brain-sync" --discover-new 2>/dev/null || true
"~/.claude/skills/gstack/bin/gstack-brain-sync" --once 2>/dev/null || true
```


## Modele Özel Davranış Yaması (claude)

Aşağıdaki düzeltmeler claude model ailesi için ayarlanmıştır. Bunlar skill iş akışına,
STOP noktalarına, AskUserQuestion kapılarına, plan modu güvenliğine ve /ship inceleme
kapılarına **tabidir**. Aşağıdaki bir düzeltme skill talimatlarıyla çelişirse,
skill kazanır. Bunları kurallar değil, tercihler olarak ele alın.

**Yapılacaklar listesi disiplini.** Çok adımlı bir plan üzerinden çalışırken, her görevi
tamamladıkça ayrı ayrı işaretleyin. Sonunda toplu olarak tamamlama yapmayın. Bir görevin
gereksiz olduğu ortaya çıkarsa, tek satırlık bir nedenle atlandı olarak işaretleyin.

**Ağır işlemlerden önce düşünün.** Karmaşık işlemler (yeniden düzenlemeler, geçişler,
önemsiz olmayan yeni özellikler) için yaklaşımınızı çalıştırmadan önce kısaca belirtin.
Bu, kullanıcının uçuş ortasında yerine düşük maliyetle düzeltme yapmasına olanak tanır.

**Bash yerine özel araçlar.** Read, Edit, Write, Glob, Grep araçlarını shell
karşılıkları (cat, sed, find, grep) yerine tercih edin. Özel araçlar daha ucuz ve daha net.

## Ses

Doğrudan, somut, yapıcıdan yapıcıya. Dosya, fonksiyon, komut ve kullanıcıya görünen
etkiyi adlandırın. Dolgu yok.

Uzun tire yok. AI kelime dağarcığı yok: delve, crucial, robust, comprehensive, nuanced,
multifaceted. Asla kurumsal veya akademik. Kısa paragraflar. Ne yapılması gerektiğiyle bitirin.

Kullanıcının sizin sahip olmadığınız bağlamı var. Modeller arası anlaşma bir öneridir, bir
karar değil. Kullanıcı karar verir.

## Tamamlama Durum Protokolü

Skill iş akışını tamamlarken, durumu şunlardan birini kullanarak raporlayın:
- **DONE** — kanıtla tamamlandı.
- **DONE_WITH_CONCERNS** — tamamlandı, ancak endişeleri listeleyin.
- **BLOCKED** — devam edemiyor; engelleyiciyi ve deneneni belirtin.
- **NEEDS_CONTEXT** — eksik bilgi; tam olarak neye ihtiyaç duyulduğunu belirtin.

3 başarısız girişimden, belirsiz güvenliğe duarlı değişikliklerden veya doğrulayamadığınız
kapsamdan sonra yükseltin. Format: `DURUM`, `NEDEN`, `DENENEN`, `ÖNERİ`.

## Operasyonel Kendini Geliştirme

Tamamlamadan önce, bir sonraki sefere 5+ dakika kazandıracak dayanıklı bir proje tuhaflığı
veya komut düzeltmesi keşfettiyseniz, loglayın:

```bash
~/.claude/skills/gstack/bin/gstack-learnings-log '{"skill":"SKILL_NAME","type":"operational","key":"SHORT_KEY","insight":"DESCRIPTION","confidence":N,"source":"observed"}'
```

Açık gerçekleri veya tek seferlik geçici hataları loglamayın.

## Telemetri (son olarak çalıştır)

İş akışı tamamlandıktan sonra telemetriyi loglayın. Frontmatter'daki skill `name:` değerini kullanın. OUTCOME: success/error/abort/unknown.

**PLAN MODE İSTİSNASI — HER ZAMAN ÇALIŞTIR:** Bu komut telemetriyi
`~/.gstack/analytics/` dizinine yazar, önsöz analitik yazmalarıyla eşleşir.

Bu bash'ı çalıştırın:

```bash
_TEL_END=$(date +%s)
_TEL_DUR=$(( _TEL_END - _TEL_START ))
rm -f ~/.gstack/analytics/.pending-"$_SESSION_ID" 2>/dev/null || true
# Oturum zaman çizelgesi: skill tamamlanmasını kaydet (yalnızca yerel, hiçbir yere gönderilmez)
~/.claude/skills/gstack/bin/gstack-timeline-log '{"skill":"SKILL_NAME","event":"completed","branch":"'$(git branch --show-current 2>/dev/null || echo unknown)'","outcome":"OUTCOME","duration_s":"'"$_TEL_DUR"'","session":"'"$_SESSION_ID"'"}' 2>/dev/null || true
# Yerel analitik (telemetri ayarına bağlı)
if [ "$_TEL" != "off" ]; then
echo '{"skill":"SKILL_NAME","duration_s":"'"$_TEL_DUR"'","outcome":"OUTCOME","browse":"USED_BROWSE","session":"'"$_SESSION_ID"'","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'"}' >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
fi
# Uzak telemetri (katılım esaslı, ikili gerektirir)
if [ "$_TEL" != "off" ] && [ -x ~/.claude/skills/gstack/bin/gstack-telemetry-log ]; then
  ~/.claude/skills/gstack/bin/gstack-telemetry-log \
    --skill "SKILL_NAME" --duration "$_TEL_DUR" --outcome "OUTCOME" \
    --used-browse "USED_BROWSE" --session-id "$_SESSION_ID" 2>/dev/null &
fi
```

Çalıştırmadan önce `SKILL_NAME`, `OUTCOME` ve `USED_BROWSE` değerlerini değiştirin.

## Plan Durumu Altbilgisi

Plan incelemeleri çalıştıran skill'ler (`/plan-*-review`, `/codex review`), skill'in sonundaki EXIT PLAN MODE GATE engelleme kontrol listesini içerir; bu liste, ExitPlanMode çağrılmadan önce plan dosyasının `## GSTACK REVIEW REPORT` ile bittiğini doğrular. Plan incelemeleri çalıştırmayan skill'ler (`/ship`, `/qa`, `/review` gibi operasyonel skill'ler) genellikle plan modunda çalışmaz ve doğrulanacak inceleme raporu yoktur; bu altbilgi onlar için no-op'tur. Plan dosyasını yazmak, plan modunda izin verilen tek düzenlemedir.

# make-pdf: markdown'dan yayıncılık kalitesinde PDF'ler

`.md` dosyalarını Faber & Faber denemelerine benzeyen PDF'lere dönüştürür: 1 inç kenar boşlukları,
sol hizalı gövde, boyunca Helvetica, kıvrımlı tırnak işaretleri ve em tireler, isteğe bağlı
kapak sayfası ve tıklanabilir İÇİNDEKİLER, ihtiyacınız olduğunda çapraz TASLAK filigranı.
PDF'ten kopyalama temiz kelimeler üretir, asla "S a i l i n g" değil.

Linux'ta doğru oluşturma için `fonts-liberation` kurun — Helvetica ve Arial varsayılan olarak
mevcut değildir ve Liberation Sans standart metrik uyumlu yedeğidir. CI ve Docker yapıları
Dockerfile.ci aracılığıyla otomatik olarak kurar.

## Temel kalıplar

### %80 kullanım durumu — not/mektup

Tek komut, bayrak yok. Sürekli üstbilgi + sayfa numaraları + varsayılan olarak GİZLİ altbilgi
ile temiz bir PDF alır.

```bash
$P generate letter.md                 # /tmp/letter.pdf dosyasına yazar
$P generate letter.md letter.pdf      # açık çıktı yolu
```

### Yayıncılık modu — kapak + İÇİNDEKİLER + bölüm sonları

```bash
$P generate --cover --toc --author "Garry Tan" --title "On Horizons" \
  essay.md essay.pdf
```

Markdown'daki her üst düzey H1 yeni bir sayfa başlatır. Birden fazla H1'i olan notlar için
`--no-chapter-breaks` ile devre dışı bırakın.

### Taslak aşaması filigranı

```bash
$P generate --watermark DRAFT memo.md draft.pdf
```

Her sayfada çapraz %10 opaklık TASLAK. Taslak son haline geldiğinde, bayrağı kaldırın
ve yeniden oluşturun.

### Önizleme ile hızlı yineleme

```bash
$P preview essay.md
```

Aynı baskı CSS ile HTML oluşturur ve tarayıcınızda açar. Markdown'ı
düzenledikçe yenileyin. Hazır olana kadar PDF turunu atlayın.

### Marksız (GİZLİ altbilgi yok)

```bash
$P generate --no-confidential memo.md memo.pdf
```

## Yaygın bayraklar

```
Sayfa düzeni:
  --margins <boyut>            1in (varsayılan) | 72pt | 2.54cm | 25mm
  --page-size letter|a4|legal

Yapı:
  --cover                    Kapak sayfası (başlık, yazar, tarih, ince çizgi)
  --toc                      Sayfa numaralı tıklanabilir İÇİNDEKİLER
  --no-chapter-breaks        Her H1'de yeni sayfa başlatma

Markalama:
  --watermark <metin>         Çapraz filigran ("DRAFT", "CONFIDENTIAL")
  --header-template <html>    Özel sürekli üstbilgi
  --footer-template <html>    Özel altbilgi (--page-numbers ile çelişir)
  --no-confidential          GİZLİ sağ altbilgiyi gizle

Çıktı:
  --page-numbers             "N / M" altbilgi (varsayılan açık)
  --tagged                   Erişilebilir PDF (varsayılan açık)
  --outline                  Başlıklardan PDF yer imleri (varsayılan açık)
  --quiet                    stderr üzerindeki ilerlemeyi gizle
  --verbose                   Aşama aşama zamanlamalar

Ağ:
  --allow-network            Harici resimleri çek. Varsayılan olarak kapalı
                             (izleme piksellerini engeller).

Meta veriler:
  --title "..."              Belge başlığı (varsayılan: ilk H1)
  --author "..."              Kapak + PDF meta verileri için yazar
  --date "..."               Kapak için tarih (varsayılan: bugün)
```

## Claude ne zaman çalıştırmalı

Markdown'dan PDF'e dönüştürme niyetini izleyin. Bu kalıplardan herhangi biri → `$P generate` çalıştırın:

- "Bu markdownu PDF yapabilir misin"
- "PDF olarak aktar"
- "Bu mektubu PDF'e dönüştür"
- "Makalenin PDF'ini istiyorum"
- "Bunu benim için PDF olarak yazdır"

Kullanıcıda açık bir `.md` dosyası varsa ve "güzel görünmesini sağla" derse,
`$P generate --cover --toc` önerin ve çalıştırmadan önce sorun.

## Hata ayıklama

- Çıktı boş / beyaz görünüyor → browse daemon'ın çalıştığını kontrol edin: `$B status`.
- Kopyala-yapıştırda parçalanmış metin → highlight.js çıktısı (Aşama 4). Bu bayrak
  mevcut olana kadar `--no-syntax` ile tekrar deneyin. Şimdilik, çitli kod bloklarını
  kaldırın ve yeniden oluşturun.
- Paged.js zaman aşımı → muhtemelen markdown'da başlık yok. `--toc` seçeneğini kaldırın.
- Harici resim eksik → `--allow-network` ekleyin (markdown dosyasına resim URL'lerinden
  çekme izni verdiğinizi anlayın).
- Oluşturulan PDF çok uzun/geniş → `--page-size a4` veya `--margins 0.75in`.

## Çıktı sözleşmesi

```
stdout: /tmp/letter.pdf          ← sadece yol, bir satır
stderr: Rendering HTML...        ← ilerleme döndürücüsü (--quiet olmadıkça)
        Generating PDF...
        Done in 1.5s. 43 words · 22KB · /tmp/letter.pdf

exit code: 0 başarı / 1 hatalı argümanlar / 2 oluşturma hatası / 3 Paged.js zaman aşımı
           / 4 browse kullanılamıyor
```

Yolu yakalayın: `PDF=$($P generate letter.md)` — ardından `$PDF` kullanın.