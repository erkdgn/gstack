---
name: unfreeze
version: 0.1.0
description: |
  /freeze tarafından ayarlanan freeze sınırını temizler, tüm dizinlerde yeniden
  düzenlemeye izin verir. Oturumu sonlandırmadan düzenleme kapsamını genişletmek
  istediğinizde kullanın. "unfreeze", "düzenlemelerin kilidini aç", "freeze kaldır"
  veya "tüm düzenlemelere izin ver" isteklerinde kullanılır. (gstack)
triggers:
  - düzenleme kilidini aç
  - tüm dizinlerin kilidini aç
  - düzenleme kısıtlamalarını kaldır
allowed-tools:
  - Bash
  - Read
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — doğrudan düzenlemeyin -->
<!-- Yeniden oluştur: bun run gen:skill-docs -->

# /unfreeze — Freeze Sınırını Temizle

`/freeze` tarafından ayarlanan düzenleme kısıtlamasını kaldırır, tüm dizinlerde
düzenlemeye izin verir.

```bash
mkdir -p ~/.gstack/analytics
echo '{"skill":"unfreeze","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
```

## Sınırı temizle

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-paths)"
STATE_DIR="$GSTACK_STATE_ROOT"
if [ -f "$STATE_DIR/freeze-dir.txt" ]; then
  PREV=$(cat "$STATE_DIR/freeze-dir.txt")
  rm -f "$STATE_DIR/freeze-dir.txt"
  echo "Freeze boundary cleared (was: $PREV). Edits are now allowed everywhere."
else
  echo "No freeze boundary was set."
fi
```

Kullanıcıya sonucu bildirin. `/freeze` hook'larının oturum için hâlâ kayıtlı
olduğunu — durum dosyası olmadığından her şeye izin vereceklerini unutmayın.
Yeniden freeze etmek için `/freeze` komutunu tekrar çalıştırın.