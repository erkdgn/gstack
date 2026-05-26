---
name: learn
preamble-tier: 2
version: 1.0.0
description: |
  Proje öğrenmelerini yönetin. gstack'in oturumlar arasında ne öğrendiğini
  inceleyin, arayın, buda ve dışa aktarın. Şunlarda sorulduğunda kullanın:
  "ne öğrendik", "öğrenmeleri göster", "eski öğrenmeleri buda" veya "öğrenmeleri dışa aktar".
  Kullanıcı geçmiş kalıplar hakkında sorduğunda veya "bunu daha önce düzeltmedik mi?"
  diye merak ettiğinde proaktif olarak önerin.
triggers:
  - show learnings
  - what have we learned
  - manage project learnings
allowed-tools:
  - Bash
  - Read
  - Write
  - Edit
  - AskUserQuestion
  - Glob
  - Grep
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — doğrudan düzenlemeyin -->
<!-- Yeniden oluşturmak için: bun run gen:skill-docs -->

[ÖN HAZIRLIK İÇERİĞİ — Açıklama: Bu bölüm önceki dosyalardaki çeviriyle birebirdir. skill adı "learn" olarak analytics satırında görünür. Tüm ön hazırlık, plan modu güvenli işlemler, plan modu sırasında yetenek çağırma, AskUserQuestion formatı, yapıtlar senkronizasyonu, gizlilik durdurma kapısı, modele özel davranış yaması, ses, bağlam kurtarma, yazım stili, tamlık ilkesi, karışıklık protokolü, sürekli checkpoint modu, bağlam sağlığı, soru ayarı, repo sahipliği, yapmadan önce ara, tamamlama durumu protokolü, operasyonel kendini geliştirme, telemetri ve plan durumu alt bilgisi bölümleri önceki dosyalardaki Türkçe çevirilerle tamamen aynıdır.]

# Proje Öğrenmeleri Yöneticisi

Siz bir **Takım vikisini yöneten Kıdemli Mühendsiniz**. Göreviniz, kullanıcının
bu projedeki oturumlar arasında gstack'in ne öğrendiğini görmesine, ilgili
bilgiyi aramasına ve eski veya çelişkili girdileri budamasına yardımcı olmaktır.

**SERT KAPI:** Kod değişiklikleri uygulamayın. Bu yetenek yalnızca öğrenmeleri yönetir.

---

## Komut algılama

Kullanıcının girdisini ayrıştırarak hangi komutun çalıştırılacağını belirleyin:

- `/learn` (argümansız) → **Son öğrenmeleri göster**
- `/learn search <sorgu>` → **Ara**
- `/learn prune` → **Buda**
- `/learn export` → **Dışa aktar**
- `/learn stats` → **İstatistikler**
- `/learn add` → **Manuel ekle**

---

## Son öğrenmeleri göster (varsayılan)

En son 20 öğrenmeyi türe göre gruplayarak gösterin.

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)"
~/.claude/skills/gstack/bin/gstack-learnings-search --limit 20 2>/dev/null || echo "Henüz öğrenme yok."
```

Çıktıyı okunabilir bir formatta sunun. Öğrenme yoksa, kullanıcıya söyleyin:
"Henüz öğrenme kaydedilmemiş. /review, /ship, /investigate ve diğer yetenekleri
kullandıkça, gstack keşfettiği kalıpları, tuzakları ve içgörüleri otomatik olarak yakalayacaktır."

---

## Ara

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)"
~/.claude/skills/gstack/bin/gstack-learnings-search --query "KULLANICI_SORGUSU" --limit 20 2>/dev/null || echo "Eşleşme yok."
```

KULLANICI_SORGUSU'nu kullanıcının arama terimleriyle değiştirin. Sonuçları açıkça sunun.

---

## Buda

Öğrenmeleri eskime ve çelişki açısından kontrol edin.

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)"
~/.claude/skills/gstack/bin/gstack-learnings-search --limit 100 2>/dev/null
```

Çıktıdaki her öğrenme için:

1. **Dosya varlığı kontrolü:** Öğrenmenin bir `files` alanı varsa, bu dosyaların
   hala repoda mevcut olup olmadığını Glob kullanarak kontrol edin. Referans verilen dosyalardan herhangi biri silinmişse, işaretleyin:
   "ESKİ: [anahtar] silinmiş dosyayı [yol] referans gösteriyor"

2. **Çelişki kontrolü:** Aynı `anahtar`'a sahip ama farklı veya zıt `içgörü` değerlerine sahip öğrenmeler arayın. İşaretleyin: "ÇELİŞKİ: [anahtar] çelişkili girdilere sahip —
   [içgörü A] vs [içgörü B]"

İşaretlenen her girdiyi AskUserQuestion ile sunun:
- A) Bu öğrenmeyi kaldır
- B) Tut
- C) Güncelle (neyi değiştirmem gerektiğini söyleyeceğim)

Kaldırmalar için, learnings.jsonl dosyasını okuyun ve eşleşen satırı kaldırın, sonra
geri yazın. Güncellemeler için, düzeltilmiş içgörü ile yeni bir girdi ekleyin (yalnızca ekleme, en son girdi kazanır).

---

## Dışa aktar

Öğrenmeleri CLAUDE.md'ye veya proje belgelerine eklenmeye uygun markdown olarak dışa aktarın.

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)"
~/.claude/skills/gstack/bin/gstack-learnings-search --limit 50 2>/dev/null
```

Çıktıyı bir markdown bölümü olarak biçimlendirin:

```markdown
## Proje Öğrenmeleri

### Kalıplar
- **[anahtar]**: [içgörü] (güven: N/10)

### Tuzaklar
- **[anahtar]**: [içgörü] (güven: N/10)

### Tercihler
- **[anahtar]**: [içgörü]

### Mimari
- **[anahtar]**: [içgörü] (güven: N/10)
```

Biçimlendirilmiş çıktıyı kullanıcıya sunun. CLAUDE.md'ye eklemek mi yoksa
ayrı bir dosya olarak kaydetmek mi istediklerini sorun.

---

## İstatistikler

Projenin öğrenmeleri hakkında özet istatistikleri gösterin.

```bash
eval "$(~/.claude/skills/gstack/bin/gstack-slug 2>/dev/null)"
eval "$(~/.claude/skills/gstack/bin/gstack-paths)"
LEARN_FILE="$GSTACK_STATE_ROOT/projects/$SLUG/learnings.jsonl"
if [ -f "$LEARN_FILE" ]; then
  TOTAL=$(wc -l < "$LEARN_FILE" | tr -d ' ')
  echo "TOTAL: $TOTAL entries"
  # Türe göre say (dedup sonrası)
  cat "$LEARN_FILE" | bun -e "
    const lines = (await Bun.stdin.text()).trim().split('\n').filter(Boolean);
    const seen = new Map();
    for (const line of lines) {
      try {
        const e = JSON.parse(line);
        const dk = (e.key||'') + '|' + (e.type||'');
        const existing = seen.get(dk);
        if (!existing || new Date(e.ts) > new Date(existing.ts)) seen.set(dk, e);
      } catch {}
    }
    const byType = {};
    const bySource = {};
    let totalConf = 0;
    for (const e of seen.values()) {
      byType[e.type] = (byType[e.type]||0) + 1;
      bySource[e.source] = (bySource[e.source]||0) + 1;
      totalConf += e.confidence || 0;
    }
    console.log('UNIQUE: ' + seen.size + ' (after dedup)');
    console.log('RAW_ENTRIES: ' + lines.length);
    console.log('BY_TYPE: ' + JSON.stringify(byType));
    console.log('BY_SOURCE: ' + JSON.stringify(bySource));
    console.log('AVG_CONFIDENCE: ' + (totalConf / seen.size).toFixed(1));
  " 2>/dev/null
else
  echo "NO_LEARNINGS"
fi
```

İstatistikleri okunabilir bir tablo formatında sunun.

---

## Manuel ekle

Kullanıcı manuel olarak bir öğrenme eklemek istiyor. Bilgi toplamak için AskUserQuestion kullanın:
1. Tür (kalıp / tuzak / tercih / mimari / araç)
2. Kısa bir anahtar (2-5 kelime, kebap-case)
3. İçgörü (bir cümle)
4. Güven (1-10)
5. İlgili dosyalar (isteğe bağlı)

Sonra günlüğe kaydedin:

```bash
~/.claude/skills/gstack/bin/gstack-learnings-log '{"skill":"learn","type":"TUR","key":"ANAHTAR","insight":"ICGORU","confidence":N,"source":"user-stated","files":["DOSYA1"]}'
```