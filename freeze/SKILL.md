---
name: freeze
version: 0.1.0
description: |
  Dosya düzenlemelerini oturum boyunca belirli bir dizinle sınırlandırır. İzin
  verilen yol dışındaki Edit ve Write işlemlerini engeller. Hata ayıklarken
  istemeden "ilgisiz" kodu düzeltmeyi önlemek veya değişiklikleri tek bir
  modülle sınırlandırmak istediğinizde kullanın. "freeze", "düzenlemeleri
  kısıtla", "sadece bu klasörü düzenle" veya "düzenlemeleri kilitle"
  isteklerinde kullanılır. (gstack)
triggers:
  - düzenlemeleri dizine kilitle
  - düzenleme kapsamını kilitle
  - dosya değişikliklerini kısıtla
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
hooks:
  PreToolUse:
    - matcher: "Edit"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/bin/check-freeze.sh"
          statusMessage: "Freeze sınırı kontrol ediliyor..."
    - matcher: "Write"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/bin/check-freeze.sh"
          statusMessage: "Freeze sınırı kontrol ediliyor..."
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — doğrudan düzenlemeyin -->
<!-- Yeniden oluştur: bun run gen:skill-docs -->

# /freeze — Düzenlemeleri Bir Dizinle Sınırlandır

Dosya düzenlemelerini belirli bir dizine kilitler. İzin verilen yol dışındaki
herhangi bir Edit veya Write işlemi **engellenir** (sadece uyarı değil).

```bash
mkdir -p ~/.gstack/analytics
echo '{"skill":"freeze","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
```

## Kurulum

Kullanıcıya düzenlemelerin hangi dizinle sınırlandırılacağını sorun. AskUserQuestion kullanın:

- Soru: "Düzenlemeleri hangi dizinle sınırlandırayım? Bu yol dışındaki dosyalar düzenlenmekten engellenecek."
- Metin girişi (çoklu seçim değil) — kullanıcı bir yol yazar.

Kullanıcı bir dizin yolu sağladıktan sonra:

1. Mutlak yola çözümleyin:
```bash
FREEZE_DIR=$(cd "<user-provided-path>" 2>/dev/null && pwd)
echo "$FREEZE_DIR"
```

2. Sondaki eğik çizgiyi sağlayın ve freeze durum dosyasına kaydedin:
```bash
FREEZE_DIR="${FREEZE_DIR%/}/"
eval "$(~/.claude/skills/gstack/bin/gstack-paths)"
STATE_DIR="$GSTACK_STATE_ROOT"
mkdir -p "$STATE_DIR"
echo "$FREEZE_DIR" > "$STATE_DIR/freeze-dir.txt"
echo "Freeze boundary set: $FREEZE_DIR"
```

Kullanıcıya şunu söyleyin: "Düzenlemeler artık `<path>/` ile sınırlandırıldı. Bu
dizin dışındaki herhangi bir Edit veya Write engellenecektir. Sınırı değiştirmek
için `/freeze` komutunu tekrar çalıştırın. Kaldırmak için `/unfreeze` çalıştırın
veya oturumu sonlandırın."

## Nasıl çalışır

Hook, Edit/Write araç girdisi JSON'undan `file_path` değerini okur ve yolun
freeze diziniyle başlayıp başlamadığını kontrol eder. Başlamıyorsa, işlemi
engellemek için `permissionDecision: "deny"` döndürür.

Freeze sınırı durum dosyası aracılığıyla oturum boyunca kalıcı olur. Hook
betiği bunu her Edit/Write çağrısında okur.

## Notlar

- Freeze dizinindeki sondaki `/`, `/src` dizininin `/src-old` ile eşleşmesini önler
- Freeze yalnızca Edit ve Write araçlarına uygulanır — Read, Bash, Glob, Grep etkilenmez
- Bu istemeyen düzenlemeleri önler, bir güvenlik sınırı değildir — `sed` gibi Bash komutları hâlâ sınır dışındaki dosyaları değiştirebilir
- Devre dışı bırakmak için `/unfreeze` çalıştırın veya konuşmayı sonlandırın