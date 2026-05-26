# TODOS.md Format Referansı

Kanonik TODOS.md formatı için paylaşılan referans. Tutarlı TODO öğe yapısını sağlamak için `/ship` (Adım 5.5) ve `/plan-ceo-review` (TODOS.md güncellemeleri bölümü) tarafından referans alınır.

---

## Dosya Yapısı

```markdown
# TODOS

## <Skill/Bileşen>     ← örn., ## Browse, ## Ship, ## Review, ## Infrastructure
<öğeler önce P0, sonra P1, P2, P3, P4 olarak sıralanmış>

## Tamamlandı
<bitiş açıklaması ile tamamlanmış öğeler>
```

**Bölümler:** Skill veya bileşene göre düzenleyin (`## Browse`, `## Ship`, `## Review`, `## QA`, `## Retro`, `## Infrastructure`). Her bölümde öğeleri önceliğe göre sıralayın (en üstte P0).

---

## TODO Öğe Formatı

Her öğe bölümü altında bir H3'tür:

```markdown
### <Başlık>

**Ne:** İşin tek satırlık açıklaması.

**Neden:** Çözdüğü somut sorun veya açtığı değer.

**Bağlam:** Birisi bunu 3 ay sonra ele aldığında motivasyonu, mevcut durumu ve nereden başlayacağını anlayacak kadar ayrıntı.

**Çaba:** S / M / L / XL
**Öncelik:** P0 / P1 / P2 / P3 / P4
**Bağlıdır:** <önkoşullar veya "Yok">
```

**Gerekli alanlar:** Ne, Neden, Bağlam, Çaba, Öncelik
**İsteğe bağlı alanlar:** Bağlıdır, Engelleyen

---

## Öncelik Tanımları

- **P0** — Engelleyici: bir sonraki sürümden önce yapılmalı
- **P1** — Kritik: bu döngüde yapılmalı
- **P2** — Önemli: P0/P1 bittiğinde yapılmalı
- **P3** — Güzel-olur: benimsenme/kullanım verisinden sonra yeniden değerlendir
- **P4** — Bir gün: iyi fikir, aciliyet yok

---

## Tamamlanmış Öğe Formatı

Bir öğe tamamlandığığda, orijinal içeriğini koruyarak `## Tamamlandı` bölümüne taşıyın ve şunu ekleyin:

```markdown
**Tamamlandı:** vX.Y.Z (YYYY-AA-GG)
```