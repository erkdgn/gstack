---
name: setup-browser-cookies
preamble-tier: 1
version: 1.0.0
description: |
  Gerçek Chromium tarayıcınızdan çerezleri headless browse oturumuna aktarır.
  Hangi çerez alan adlarını aktaracağınızı seçtiğiniz etkileşimli bir seçici
  arayüzü açar. Kimlik doğrulaması yapılan sayfaları QA testinden önce kullanın.
  "çerezleri aktar", "siteye giriş yap" veya "tarayıcıyı doğrula" isteklerinde
  kullanılır. (gstack)
  Ses tetikleyicileri (konuşmadan metne takma adlar): "bunu pdfe çevir", "pdfe çevir", "pdfe aktar", "bunu pdfe dönüştür", "bu markdownu pdfe dönüştür", "pdf oluştur", "bunu pdfe yap".
triggers:
  - tarayıcı çerezlerini aktar
  - test sitesine giriş yap
  - kimlik doğrulamalı oturum kur
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
echo '{"skill":"setup-browser-cookies","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
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
~/.claude/skills/gstack/bin/gstack-timeline-log '{"skill":"setup-browser-cookies","event":"started","branch":"'"$_BRANCH"'","session":"'"$_SESSION_ID"'"}' 2>/dev/null &
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

3 başarısız girişimden, belirsiz güvenliğe duyarlı değişikliklerden veya doğrulayamadığınız
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

# Tarayıcı Çerezlerini Kur

Gerçek Chromium tarayıcınızdan oturum açma bilgilerini headless browse oturumuna aktarın.

## CDP mod kontrolü

Önce browse'un kullanıcının gerçek tarayıcısına bağlı olup olmadığını kontrol edin:
```bash
$B status 2>/dev/null | grep -q "Mode: cdp" && echo "CDP_MODE=true" || echo "CDP_MODE=false"
```
Eğer `CDP_MODE=true` ise: kullanıcıya "Gerekli değil — CDP üzerinden gerçek tarayıcınıza bağlısınız. Çerezleriniz ve oturumlarınız zaten mevcut." deyin ve durun. Çerez aktarımı gerekli değil.

## Nasıl çalışır

1. Browse ikili dosyasını bulun
2. Yüklü tarayıcıları algılamak ve seçici arayüzü açmak için `cookie-import-browser` çalıştırın
3. Kullanıcı, tarayıcısında hangi çerez alan adlarını aktaracağını seçer
4. Çerezler şifre çözülür ve Playwright oturumuna yüklenir

## Adımlar

### 1. Browse ikili dosyasını bulun

## KURULUM (herhangi bir browse komutundan ÖNCE bu kontrolü çalıştır)

```bash
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
B=""
[ -n "$_ROOT" ] && [ -x "$_ROOT/.claude/skills/gstack/browse/dist/browse" ] && B="$_ROOT/.claude/skills/gstack/browse/dist/browse"
[ -z "$B" ] && B="$HOME/.claude/skills/gstack/browse/dist/browse"
if [ -x "$B" ]; then
  echo "READY: $B"
else
  echo "NEEDS_SETUP"
fi
```

Eğer `NEEDS_SETUP` ise:
1. Kullanıcıya şunu söyleyin: "gstack browse tek seferlik bir kurulum gerektiriyor (~10 saniye). Devam edeyim mi?" Ardından DURUN ve bekleyin.
2. Çalıştırın: `cd <SKILL_DIR> && ./setup`
3. Eğer `bun` kurulu değilse:
   ```bash
   if ! command -v bun >/dev/null 2>&1; then
     BUN_VERSION="1.3.10"
     BUN_INSTALL_SHA="bab8acfb046aac8c72407bdcce903957665d655d7acaa3e11c7c4616beae68dd"
     tmpfile=$(mktemp)
     curl -fsSL "https://bun.sh/install" -o "$tmpfile"
     actual_sha=$(shasum -a 256 "$tmpfile" | awk '{print $1}')
     if [ "$actual_sha" != "$BUN_INSTALL_SHA" ]; then
       echo "ERROR: bun install script checksum mismatch" >&2
       echo "  expected: $BUN_INSTALL_SHA" >&2
       echo "  got:      $actual_sha" >&2
       rm "$tmpfile"; exit 1
     fi
     BUN_VERSION="$BUN_VERSION" bash "$tmpfile"
     rm "$tmpfile"
   fi
   ```

### 2. Çerez seçiciyi aç

```bash
$B cookie-import-browser
```

Bu, yüklü Chromium tarayıcılarını otomatik olarak algılar ve varsayılan tarayıcınızda
şunları yapabileceğiniz etkileşimli bir seçici arayüzü açar:
- Yüklü tarayıcılar arasında geçiş yapma
- Alan adları arama
- Bir alan adının çerezlerini aktarmak için "+" tıklama
- Aktarılan çerezleri kaldırmak için çöp kutusuna tıklama

Kullanıcıya şunu söyleyin: **"Çerez seçici açıldı — tarayıcınızda aktarmak istediğiniz alan adlarını seçin, ardından bittiğinizde bana söyleyin."**

### 3. Doğrudan aktarım (alternatif)

Kullanıcı bir alan adını doğrudan belirtirse (örn. `/setup-browser-cookies github.com`), arayüzü atlayın:

```bash
$B cookie-import-browser comet --domain github.com
```

Belirtilmişse uygun tarayıcı ile `comet` değiştirin.

### 4. Doğrula

Kullanıcı işleminin bittiğini onayladıktan sonra:

```bash
$B cookies
```

Kullanıcıya aktarılan çerezlerin bir özetini gösterin (alan adı sayıları).

## Notlar

- macOS'ta, tarayıcı başına ilk aktarım bir Anahtarlık iletişim kutusu tetikleyebilir — "İzin Ver" / "Her Zaman İzin Ver" tıklayın
- Linux'ta, `v11` çerezleri `secret-tool`/libsecret erişimi gerektirebilir; `v10` çerezleri Chromium'ın standart yedek anahtarını kullanır
- Çerez seçici, browse sunucusuyla aynı portta sunulur (ekstra süreç yok)
- Arayüzde yalnızca alan adları ve çerez sayıları gösterilir — çerez değerleri açığa çıkarılmaz
- Browse oturumu çerezleri komutlar arasında korur, bu nedenle aktarılan çerezler hemen çalışır