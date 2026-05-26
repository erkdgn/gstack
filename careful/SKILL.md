---
name: careful
version: 0.1.0
description: |
  Yıkıcı komutlar için güvenlik koruma rail'leri. rm -rf, DROP TABLE,
  force-push, git reset --hard, kubectl delete ve benzeri yıkıcı işlemlerden
  önce uyarır. Kullanıcı her uyarıyı geçersiz kılabilir. Prodüksiyona dokunurken,
  canlı sistemlerde hata ayıklarken veya paylaşımlı bir ortamda çalışırken kullanın.
  "dikkatli ol", "güvenlik modu", "prod modu" veya "dikkatli mod"
  isteklerinde kullanılır. (gstack)
triggers:
  - dikkatli ol
  - yıkıcı işlemlerden önce uyar
  - güvenlik modu
allowed-tools:
  - Bash
  - Read
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "bash ${CLAUDE_SKILL_DIR}/bin/check-careful.sh"
          statusMessage: "Yıkıcı komutlar kontrol ediliyor..."
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — doğrudan düzenlemeyin -->
<!-- Yeniden oluştur: bun run gen:skill-docs -->

# /careful — Yıkıcı Komut Koruma Rail'leri

Güvenlik modu artık **aktif**. Her bash komutu çalıştırılmadan önce yıkıcı
kalıplar için kontrol edilecek. Yıkıcı bir komut algılanırsa uyarılacak ve
devam etmeyi veya iptal etmeyi seçebileceksiniz.

```bash
mkdir -p ~/.gstack/analytics
echo '{"skill":"careful","ts":"'$(date -u +%Y-%m-%dT%H:%M:%SZ)'","repo":"'$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo "unknown")'"}'  >> ~/.gstack/analytics/skill-usage.jsonl 2>/dev/null || true
```

## Neler korunur

| Kalıp | Örnek | Risk |
|-------|-------|------|
| `rm -rf` / `rm -r` / `rm --recursive` | `rm -rf /var/data` | Özyinelemeli silme |
| `DROP TABLE` / `DROP DATABASE` | `DROP TABLE users;` | Veri kaybı |
| `TRUNCATE` | `TRUNCATE orders;` | Veri kaybı |
| `git push --force` / `-f` | `git push -f origin main` | Geçmiş yeniden yazma |
| `git reset --hard` | `git reset --hard HEAD~3` | Kaydedilmemiş iş kaybı |
| `git checkout .` / `git restore .` | `git checkout .` | Kaydedilmemiş iş kaybı |
| `kubectl delete` | `kubectl delete pod` | Prodüksiyon etkisi |
| `docker rm -f` / `docker system prune` | `docker system prune -a` | Konteyner/imaj kaybı |

## Güvenli istisnalar

Bu kalıplar uyarı olmadan izinlidir:
- `rm -rf node_modules` / `.next` / `dist` / `__pycache__` / `.cache` / `build` / `.turbo` / `coverage`

## Nasıl çalışır

Hook, araç girdisi JSON'undan komutu okur, yukarıdaki kalıplarla karşılaştırır
ve bir eşleşme bulunursa uyarı mesajıyla birlikte `permissionDecision: "ask"`
döndürür. Uyarıyı her zaman geçersiz kılıp devam edebilirsiniz.

Devre dışı bırakmak için konuşmayı sonlandırın veya yeni bir konuşma başlatın. Hook'lar oturum kapsamlıdır.