# Greptile Yorum Triyajı

GitHub PR'larında Greptile inceleme yorumlarını getirme, filtreleme ve sınıflandırma için paylaşılan referans. Hem `/review` (Adım 2.5) hem de `/ship` (Adım 3.75) bu belgeye referans verir.

---

## Getir

PR'yi algılamak ve yorumları getirmek için bu komutları çalıştırın. Her iki API çağrısı da paralel çalışır.

```bash
REPO=$(gh repo view --json nameWithOwner --jq '.nameWithOwner' 2>/dev/null)
PR_NUMBER=$(gh pr view --json number --jq '.number' 2>/dev/null)
```

**Her ikisi de başarısız olursa veya boş olursa:** Greptile triyajını sessizce atlayın. Bu entegrasyon eklentidir — iş akışı onsuz çalışır.

```bash
# Satır düzeyinde inceleme yorumlarını VE üst düzey PR yorumlarını paralel olarak getir
gh api repos/$REPO/pulls/$PR_NUMBER/comments \
  --jq '.[] | select(.user.login == "greptile-apps[bot]") | select(.position != null) | {id: .id, path: .path, line: .line, body: .body, html_url: .html_url, source: "line-level"}' > /tmp/greptile_line.json &
gh api repos/$REPO/issues/$PR_NUMBER/comments \
  --jq '.[] | select(.user.login == "greptile-apps[bot]") | {id: .id, body: .body, html_url: .html_url, source: "top-level"}' > /tmp/greptile_top.json &
wait
```

**API hataları veya her iki uç noktada sıfır Greptile yorumu olursa:** Sessizce atlayın.

Satır düzeyinde yorumlardaki `position != null` filtresi, zorla itilmiş koddan gelen eski yorumları otomatik olarak atlar.

---

## Baskı Kontrolü

Proje özgü geçmiş yolunu türet:
```bash
REMOTE_SLUG=$(browse/bin/remote-slug 2>/dev/null || ~/.claude/skills/gstack/browse/bin/remote-slug 2>/dev/null || basename "$(git rev-parse --show-toplevel 2>/dev/null || pwd)")
PROJECT_HISTORY="$HOME/.gstack/projects/$REMOTE_SLUG/greptile-history.md"
```

Mevcutsa `$PROJECT_HISTORY` dosyasını okuyun (proje özgü baskılar). Her satır önceki bir triyaj sonucunu kaydeder:

```
<tarih> | <depo> | <tür:fp|fix|already-fixed> | <dosya-deseni> | <kategori>
```

**Kategoriler** (sabit küme): `race-condition`, `null-check`, `error-handling`, `style`, `type-safety`, `security`, `performance`, `correctness`, `other`

Her getirilen yorumu şu kriterlere karşı eşleştirin:
- `tür == fp` (yalnızca bilinen yanlış pozitifleri baskılar, önceden düzeltilmiş gerçek sorunları değil)
- `depo` geçerli depoyla eşleşiyor
- `dosya-deseni` yorumun dosya yoluyla eşleşiyor
- `kategori` yorumdaki sorun türüyle eşleşiyor

Eşleşen yorumları **BASTIRILMIŞ** olarak atlayın.

Geçmiş dosyası mevcut değilse veya çözülemeyen satırlar içeriyorsa, bu satırları atlayın ve devam edin — asla bozuk bir geçmiş dosyasında başarısız olmayın.

---

## Sınıflandır

Her bastırılmamış yorum için:

1. **Satır düzeyinde yorumlar:** Dosyayı belirtilen `path:line` ve çevresel bağlamda (±10 satır) okuyun
2. **Üst düzey yorumlar:** Tam yorum gövdesini okuyun
3. Yorumu tam dif (`git diff origin/main`) ve inceleme kontrol listesiyle çapraz referanslayın
4. Sınıflandır:
   - **GEÇERLİ & İŞLENEBİLİR** — mevcut kodda var olan gerçek bir hata, yarış durumu, güvenlik sorunu veya doğruluk sorunu
   - **GEÇERLİ AMA ZATEN DÜZELTİLDİ** — daldaki sonraki bir commit'te ele alınmış gerçek bir sorun. Düzeltici commit SHA'sını belirleyin.
   - **YANLIŞ POZİTİF** — yorum kodu yanlış anlıyor, başka bir yerde ele alınan bir şeyi işaret ediyor veya stilistik gürültü
   - **BASTIRILMIŞ** — yukarıdaki baskılar kontrolünde zaten filtrelenmiş

---

## Yanıt API'leri

Greptile yorumlarına yanıt verirken, yorum kaynağına göre doğru uç noktayı kullanın:

**Satır düzeyinde yorumlar** (`pulls/$PR/comments`'dan):
```bash
gh api repos/$REPO/pulls/$PR_NUMBER/comments/$COMMENT_ID/replies \
  -f body="<yanıt metni>"
```

**Üst düzey yorumlar** (`issues/$PR/comments`'dan):
```bash
gh api repos/$REPO/issues/$PR_NUMBER/comments \
  -f body="<yanıt metni>"
```

**Bir yanıt POST'u başarısız olursa** (örn., PR kapatıldı, yazma izni yok): uyarın ve devam edin. Başarısız bir yanıt için iş akışını durdurmayın.

---

## Yanıt Şablonları

Her Greptile yanıtında bu şablonları kullanın. Her zaman somut kanıt ekleyin — asla belirsiz yanıtlar göndermeyin.

### Kademe 1 (İlk yanıt) — Dostane, kanıt dahil

**DÜZELTMELER için (kullanıcı sorunu düzeltmeyi seçti):**

```
**Düzeltildi** `<commit-sha>` commit'inde.

\`\`\`diff
- <eski sorunlu satır(lar)>
+ <yeni düzeltilmiş satır(lar)>
\`\`\`

**Neden:** <neyin yanlış olduğu ve düzeltmenin bunu nasıl ele aldığına dair 1 cümlelik açıklama>
```

**ZATEN DÜZELTİLDİ için (sorun daldaki önceki bir commit'te ele alındı):**

```
**Zaten düzeltildi** `<commit-sha>` commit'inde.

**Yapılan:** <mevcut commit'in bu sorunu nasıl ele aldığına dair 1-2 cümlelik açıklama>
```

**YANLIŞ POZİTİF için (yorum yanlış):**

```
**Hata değil.** <bunun neden yanlış olduğu konusunda 1 cümle doğrudan belirtme>

**Kanıt:**
- <desenin güvenli/doğru olduğunu gösteren belirli kod referansı>
- <örn., "Null kontrolü `ActiveRecord::FinderMethods#find` tarafından ele alınıyor, bu nil değil RecordNotFound yükseltiyor">

**Önerilen yeniden-sıralama:** Bu bir `<stil|gürültü|yanlış okuma>` sorunu olarak görünüyor, `<Greptile'in adlandırdığı şey>` değil. Şiddeti düşürmeyi düşünün.
```

### Kademe 2 (Greptile önceki yanıt sonrası yeniden işaretlerse) — Sıkı, ezici kanıt

Eskalasyon algılama (aşağıda) aynı konu üzerinde önceki bir GStack yanıtı belirlediğinde Kademe 2'yi kullanın. Tartışmayı kapatmak için maksimum kanıt ekleyin.

```
**Bu incelendi ve [kasıtlı/zaten-düzeltilmiş/hata-değil] olarak onaylandı.**

\`\`\`diff
<değişikliği veya güvenli deseni gösteren tam ilgili dif>
\`\`\`

**Kanıt zinciri:**
1. <güvenli deseni veya düzeltmeyi gösteren dosya:satır kalıcı bağlantı>
2. <varsa ele alındığı commit SHA'sı>
3. <varsa mimari gerekçe veya tasarım kararı>

**Önerilen yeniden-sıralama:** Lütfen yeniden kalibre edin — bu bir `<gerçek kategori>` sorunu, `<iddia edilen kategori>` değil. [Yararlısa belirli dosya değişikliği kalıcı bağlantısına bağlayın]
```

---

## Eskalasyon Algılama

Yanıt oluşturmadan önce, bu yorum konuğunda önceki bir GStack yanıtının mevcut olup olmadığını kontrol edin:

1. **Satır düzeyinde yorumlar için:** `gh api repos/$REPO/pulls/$PR_NUMBER/comments/$COMMENT_ID/replies` ile yanıtları getirin. Herhangi bir yanıt gövdesinde GStack işaretleri içerip içermediğini kontrol edin: `**Düzeltildi**`, `**Hata değil.**`, `**Zaten düzeltildi**`.

2. **Üst düzey yorumlar için:** Getirilen sorun yorumlarında Greptile yorumundan sonra gönderilen ve GStack işaretlerini içeren yanıtları tarayın.

3. **Önceki bir GStack yanıtı mevcutsa VE Greptile aynı dosya+kategori üzerinde yeniden gönderdiyse:** Kademe 2 (sıkı) şablonlarını kullanın.

4. **Önceki bir GStack yanıtı mevcut değilse:** Kademe 1 (dostane) şablonlarını kullanın.

Eskalasyon algılama başarısız olursa (API hatası, belirsiz konuk): varsayılan olarak Kademe 1. Belirsizlikte asla eskale etmeyin.

---

## Şiddet Değerlendirmesi & Yeniden Sıralama

Yorumları sınıflandırırken, Greptile'in ima edilen şiddetinin gerçeklikle eşleşip eşleşmediğini de değerlendirin:

- Greptile bir şeyi **güvenlik/doğruluk/yarış-durumu** sorunu olarak işaretlerse ama aslında bir **stil/performans** nükranı ise: yanıtta kategorinin düzeltilmesini isteyen `**Önerilen yeniden-sıralama:**` ekleyin.
- Greptile düşük şiddetli bir stil sorununu kritik olarak işaretlerse: yanıtta geri itin.
- Yeniden sıralamanın neden gerekli olduğu hakkında her zaman spesifik olun — görüş değil, kod ve satır numaraları belirtin.

---

## Geçmiş Dosya Yazımları

Yazmadan önce her iki dizinin mevcut olduğundan emin olun:
```bash
REMOTE_SLUG=$(browse/bin/remote-slug 2>/dev/null || ~/.claude/skills/gstack/browse/bin/remote-slug 2>/dev/null || basename "$(git rev-parse --show-toplevel 2>/dev/null || pwd)")
mkdir -p "$HOME/.gstack/projects/$REMOTE_SLUG"
mkdir -p ~/.gstack
```

Her triyaj sonucunu **her iki dosyaya** bir satır ekleyin (baskılar için proje bazında, retrospektif için genel):
- `~/.gstack/projects/$REMOTE_SLUG/greptile-history.md` (proje bazında)
- `~/.gstack/greptile-history.md` (genel toplam)

Format:
```
<YYYY-AA-GG> | <owner/repo> | <tür> | <dosya-deseni> | <kategori>
```

Örnek girdiler:
```
2026-03-13 | garrytan/myapp | fp | app/services/auth_service.rb | race-condition
2026-03-13 | garrytan/myapp | fix | app/models/user.rb | null-check
2026-03-13 | garrytan/myapp | already-fixed | lib/payments.rb | error-handling
```

---

## Çıktı Formatı

Çıktı başlığında bir Greptile özeti ekleyin:
```
+ N Greptile yorumu (X geçerli, Y düzeltildi, Z yanlış pozitif)
```

Her sınıflandırılmış yorum için şunu gösterin:
- Sınıflandırma etiketi: `[GEÇERLİ]`, `[DÜZELTİLDİ]`, `[YANLIŞ POZİTİF]`, `[BASTIRILMIŞ]`
- Dosya:satır referansı (satır düzeyinde için) veya `[üst-düzey]` (üst düzey için)
- Tek satırlık gövde özeti
- Kalıcı bağlantı URL'si (`html_url` alanı)