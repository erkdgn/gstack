---
name: guard
version: 0.1.0
description: |
  Tam güvenlik modu: yıkıcı komut uyarıları + dizin kapsamlı düzenleme kısıtlamaları.
  /careful (rm -rf, DROP TABLE, force-push vb. önce uyarır) ile /freeze
  (belirli bir dizin dışındaki düzenlemeleri engeller) birleştirmesidir. Prodüksiyona
  dokunurken veya canlı sistemlerde hata ayıklarken azami güvenlik için kullanın.
  "guard modu", "tam güvenlik", "kilitle" veya "azami güvenlik" isteklerinde
  kullanılır. (gstack)
triggers:
  - tam güvenlik modu
  - hatalara karşı koru
  - azami güvenlik
allowed-tools:
  - Bash
  - Read
  - AskUserQuestion
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/../careful/bin/check-careful.sh"
          statusMessage: "Yıkıcı komutlar kontrol ediliyor..."
    - matcher: "Edit"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/../freeze/bin/check-freeze.sh"
          statusMessage: "Freeze sınırı kontrol ediliyor..."
    - matcher: "Write"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/../freeze/bin/check-freeze.sh"
          statusMessage: "Freeze sınırı kontrol ediliyor..."
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — doğrudan düzenlemeyin -->
<!-- Yeniden oluştur: bun run gen:skill-docs -->

# /guard — Tam Güvenlik Modu

Hem yıkıcı komut uyarılarını hem de dizin kapsamlı düzenleme kısıtlamalarını
aktifleştirir. Bu, `/careful` + `/freeze` birleşiminin tek bir komutta sunulmasıdır.

**Bağımlılık notu:** Bu skill, kardeş `/careful` ve `/freeze` skill dizinlerindeki
hook betiklerine referans verir. İkisinin de kurulu olması gerekir (gstack kurulum
betiği tarafından birlikte kurulurlar).

```bash
mkdir -p ~/.gstack/analytics
echo '{"skill":"guard","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
```

## Kurulum

Kullanıcıya düzenlemelerin hangi dizinle sınırlandırılacağını sorun. AskUserQuestion kullanın:

- Soru: "Guard modu: düzenlemeler hangi dizinle sınırlandırılsın? Yıkıcı komut uyarıları her zaman aktiftir. Seçilen yol dışındaki dosyalar düzenlenmekten engellenecektir."
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

Kullanıcıya şunu söyleyin:
- "**Guard modu aktif.** İki koruma şimdi çalışıyor:"
- "1. **Yıkıcı komut uyarıları** — rm -rf, DROP TABLE, force-push vb. çalıştırılmadan önce uyarı verir (geçersiz kılabilirsiniz)"
- "2. **Düzenleme sınırı** — dosya düzenlemeleri `<path>/` ile sınırlandırıldı. Bu dizin dışındaki düzenlemeler engellenir."
- "Düzenleme sınırını kaldırmak için `/unfreeze` çalıştırın. Her şeyi devre dışı bırakmak için oturumu sonlandırın."

## Neler korunur

Yıkıcı komut kalıplarının ve güvenli istisnaların tam listesi için `/careful` sayfasına bakın.
Düzenleme sınırı zorlamanın nasıl çalıştığını görmek için `/freeze` sayfasına bakın.