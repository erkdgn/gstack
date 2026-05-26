---
name: setup-deploy
preamble-tier: 2
version: 1.0.0
description: |
  /land-and-deploy için deploy ayarlarını yapılandırın. Deploy platformunuzu
  (Fly.io, Render, Vercel, Netlify, Heroku, GitHub Actions, özel), üretim URL'nizi,
  sağlık kontrolü uç noktalarınızı ve deploy durum komutlarınızı algılar.
  Yapılandırmayı CLAUDE.md'ye yazar, böylece gelecekteki tüm deploylar otomatik olur.
  Şunlarda kullanın: "setup deploy", "configure deployment", "set up land-and-deploy",
  "gstack ile nasıl deploy yaparım", "deploy config ekle".
triggers:
  - configure deploy
  - setup deployment
  - set deploy platform
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - AskUserQuestion
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — doğrudan düzenlemeyin -->
<!-- Yeniden oluşturmak için: bun run gen:skill-docs -->

## Ön Hazırlık (önce çalıştır)

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
echo '{"skill":"setup-deploy","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
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
~/.claude/skills/gstack/bin/gstack-timeline-log '{"skill":"setup-deploy","event":"started","branch":"'"$_BRANCH"'","session":"'"$_SESSION_ID"'"}' 2>/dev/null &
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

[ÖN HAZIRLIK İÇERİĞİ — Önceki dosyalardaki çeviri ile aynı, atlanarak doğrudan skill-specific bölüme geçiliyor. Tüm ön hazırlık, plan modu, AskUserQuestion formatı, yapıtlar senkronizasyonu, modele özel davranış yaması, ses, bağlam kurtarma, yazım stili, tamlık ilkesi, karışıklık protokolü, sürekli checkpoint modu, bağlam sağlığı, soru ayarı, repo sahipliği, yapmadan önce ara, tamamlama durumu protokolü, operasyonel kendini geliştirme, telemetri ve plan durumu alt bilgisi bölümleri önceki dosyalardaki çevirilerle birebir aynıdır.]

# /setup-deploy — gstack için Deploy Yapılandırması

Kullanıcının deploy'unu `/land-and-deploy`'un otomatik çalışması için
yapılandırmasına yardımcı oluyorsunuz. Göreviniz deploy platformunu, üretim URL'sini,
sağlık kontrollerini ve deploy durum komutlarını algılamak — ardından her şeyi
CLAUDE.md'ye kalıcı olarak yazmaktır.

Bu bir kez çalıştıktan sonra, `/land-and-deploy` CLAUDE.md'yi okur ve algılamayı tamamen atlar.

## Kullanıcı tarafından çağrılabilir
Kullanıcı `/setup-deploy` yazdığında, bu yeteneği çalıştırın.

## Talimatlar

### Adım 1: Mevcut yapılandırmayı kontrol et

```bash
grep -A 20 "## Deploy Configuration" CLAUDE.md 2>/dev/null || echo "NO_CONFIG"
```

Yapılandırma zaten mevcutsa, gösterin ve sorun:

- **Bağlam:** Deploy yapılandırması zaten CLAUDE.md'de mevcut.
- **ÖNERİ:** Kurulumunuz değiştiyse güncelleştirmek için A'yı seçin.
- A) Sıfırdan yeniden yapılandır (mevcut olanın üzerine yaz)
- B) Belirli alanları düzenle (mevcut yapılandırmayı göster, bir şeyi değiştirmeme izin ver)
- C) Tamam — yapılandırma doğru görünüyor

Kullanıcı C'yi seçerse, durun.

### Adım 2: Platform algıla

Deploy önyüklemesinden platform algılamayı çalıştırın:

```bash
# Platform yapılandırma dosyaları
[ -f fly.toml ] && echo "PLATFORM:fly" && cat fly.toml
[ -f render.yaml ] && echo "PLATFORM:render" && cat render.yaml
[ -f vercel.json ] || [ -d .vercel ] && echo "PLATFORM:vercel"
[ -f netlify.toml ] && echo "PLATFORM:netlify" && cat netlify.toml
[ -f Procfile ] && echo "PLATFORM:heroku"
[ -f railway.json ] || [ -f railway.toml ] && echo "PLATFORM:railway"

# GitHub Actions deploy iş akışları
for f in $(find .github/workflows -maxdepth 1 \( -name '*.yml' -o -name '*.yaml' \) 2>/dev/null); do
  [ -f "$f" ] && grep -qiE "deploy|release|production|staging|cd" "$f" 2>/dev/null && echo "DEPLOY_WORKFLOW:$f"
done

# Proje türü
[ -f package.json ] && grep -q '"bin"' package.json 2>/dev/null && echo "PROJECT_TYPE:cli"
find . -maxdepth 1 -name '*.gemspec' 2>/dev/null | grep -q . && echo "PROJECT_TYPE:library"
```

### Adım 3: Platforma özel kurulum

Algılana göre, kullanıcıyı platforma özel yapılandırma konusunda yönlendirin.

#### Fly.io

`fly.toml` algılandıysa:

1. Uygulama adını çıkar: `grep -m1 "^app" fly.toml | sed 's/app = "\(.*\)"/\1/'`
2. `fly` CLI kurulu mu kontrol et: `which fly 2>/dev/null`
3. Kuruluysa, doğrula: `fly status --app {app} 2>/dev/null`
4. URL çıkar: `https://{app}.fly.dev`
5. Deploy durum komutunu ayarla: `fly status --app {app}`
6. Sağlık kontrolü ayarla: `https://{app}.fly.dev` (veya uygulamanın varsa `/health` uç noktası)

Kullanıcıya üretim URL'sini onaylamasını sorun. Bazı Fly uygulamaları özel alan adları kullanır.

#### Render

`render.yaml` algılandıysa:

1. Servis adını ve türünü render.yaml'dan çıkar
2. Render API anahtarını kontrol et: `echo $RENDER_API_KEY | head -c 4` (tam anahtarı açığa çıkarma)
3. URL çıkar: `https://{service-name}.onrender.com`
4. Render bağlanan branch'e push üzerine otomatik deploy yapar — deploy iş akışı gerekmez
5. Sağlık kontrolü ayarla: çıkarılan URL

Kullanıcıya onaylatın. Render bağlanan git branch'inden otomatik deploy yapar — main'e
merge sonrası, Render otomatik olarak alır. `/land-and-deploy`'daki "deploy bekleme"
Render URL'sini yeni sürümle yanıt verene kadar yoklamalıdır.

#### Vercel

vercel.json veya .vercel algılandıysa:

1. `vercel` CLI'yi kontrol et: `which vercel 2>/dev/null`
2. Kuruluysa: `vercel ls --prod 2>/dev/null | head -3`
3. Vercel push üzerine otomatik deploy yapar — PR'da önizleme, main'e merge'de üretim
4. Sağlık kontrolü ayarla: vercel proje ayarlarındaki üretim URL'si

#### Netlify

netlify.toml algılandıysa:

1. Site bilgisini netlify.toml'dan çıkar
2. Netlify push üzerine otomatik deploy yapar
3. Sağlık kontrolü ayarla: üretim URL'si

#### Yalnızca GitHub Actions

Deploy iş akışları algılandıysa ama platform yapılandırması yoksa:

1. Ne yaptığını anlamak için iş akışı dosyasını okuyun
2. Deploy hedefini çıkar (belirtilmişse)
3. Kullanıcıya üretim URL'sini sorun

#### Özel / Manuel

Hiçbir şey algılanmadıysa:

Bilgi toplamak için AskUserQuestion kullanın:

1. **Deploy'lar nasıl tetikleniyor?**
   - A) main'e push üzerine otomatik (Fly, Render, Vercel, Netlify vb.)
   - B) GitHub Actions iş akışı ile
   - C) Deploy betiği veya CLI komutu ile (açıklayın)
   - D) Manuel (SSH, dashboard vb.)
   - E) Bu proje deploy etmiyor (kütüphane, CLI, araç)

2. **Üretim URL'si nedir?** (Serbest metin — uygulamanın çalıştığı URL)

3. **gstack bir deploy'un başarılı olduğunu nasıl kontrol edebilir?**
   - A) Belirli bir URL'de HTTP sağlık kontrolü (örn., /health, /api/status)
   - B) CLI komutu (örn., `fly status`, `kubectl rollout status`)
   - C) GitHub Actions iş akışı durumunu kontrol et
   - D) Otomatik yok — sadece URL'nin yüklenip yüklenmediğini kontrol et

4. **Merge öncesi veya sonrası kancalar var mı?**
   - Merge'den önce çalıştırılacak komutlar (örn., `bun run build`)
   - Merge sonrası ama deploy doğrulamasından önce çalıştırılacak komutlar

### Adım 4: Yapılandırmayı yaz

CLAUDE.md'yi okuyun (veya oluşturun). Mevcutsa `## Deploy Configuration` bölümünü bulun
ve değiştirin, yoksa sonuna ekleyin.

```markdown
## Deploy Configuration (configured by /setup-deploy)
- Platform: {platform}
- Production URL: {url}
- Deploy workflow: {iş akışı dosyası veya "push üzerine otomatik deploy"}
- Deploy status command: {komut veya "HTTP sağlık kontrolü"}
- Merge method: {squash/merge/rebase}
- Project type: {web uygulaması / API / CLI / kütüphane}
- Post-deploy health check: {sağlık kontrolü URL'si veya komut}

### Özel deploy kancaları
- Pre-merge: {komut veya "yok"}
- Deploy trigger: {komut veya "main'e push üzerine otomatik"}
- Deploy status: {komut veya "üretim URL'sini yokla"}
- Health check: {URL veya komut}
```

### Adım 5: Doğrula

Yazdıktan sonra, yapılandırmanın çalıştığını doğrulayın:

1. Sağlık kontrolü URL'si yapılandırıldıysa, deneyin:
```bash
curl -sf "{health-check-url}" -o /dev/null -w "%{http_code}" 2>/dev/null || echo "ERISILEMEZ"
```

2. Deploy durum komutu yapılandırıldıysa, deneyin:
```bash
{deploy-status-command} 2>/dev/null | head -5 || echo "KOMUT_BASARISIZ"
```

Sonuçları raporlayın. Herhangi bir şey başarısız olduysa, not edin ama engellemeyin — yapılandırma sağlık kontrolü geçici olarak erişilemez olsa bile hala yararlıdır.

### Adım 6: Özet

```
DEPLOY YAPILANDIRMASI — TAMAMLANDI
════════════════════════════════════
Platform:      {platform}
URL:           {url}
Health check:  {sağlık kontrolü}
Status komutu: {durum komutu}
Merge yöntemi:  {merge yöntemi}

CLAUDE.md'ye kaydedildi. /land-and-deploy bu ayarları otomatik olarak kullanacak.

Sonraki adımlar:
- Geçerli PR'nizi merge etmek ve deploy etmek için /land-and-deploy çalıştırın
- Ayarları değiştirmak için CLAUDE.md'deki "## Deploy Configuration" bölümünü düzenleyin
- Yeniden yapılandırmak için /setup-deploy'u tekrar çalıştırın
```

## Önemli Kurallar

- **Asla gizlileri açığa çıkarmayın.** Tam API anahtarlarını, token'ları veya parolaları yazdırmayın.
- **Kullanıcıya onaylatın.** Algılanan yapılandırmayı her zaman gösterin ve yazmadan önce onay isteyin.
- **CLAUDE.md doğruluk kaynağıdır.** Tüm yapılandırma orada yaşar — ayrı bir yapılandırma dosyasında değil.
- **Idempotent.** /setup-deploy'u birden çok kez çalıştırmak önceki yapılandırmayı temiz bir şekilde üzerine yazar.
- **Platform CLI'leri isteğe bağlıdır.** `fly` veya `vercel` CLI kurulu değilse, URL tabanlı sağlık kontrollerine geri dönün.