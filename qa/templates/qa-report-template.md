# QA Raporu: {APP_NAME}

| Alan | Değer |
|------|-------|
| **Tarih** | {DATE} |
| **URL** | {URL} |
| **Dal** | {BRANCH} |
| **Commit** | {COMMIT_SHA} ({COMMIT_DATE}) |
| **PR** | {PR_NUMBER} ({PR_URL}) veya "—" |
| **Katman** | Hızlı / Standart / Kapsamlı |
| **Kapsam** | {SCOPE} veya "Tam uygulama" |
| **Süre** | {DURATION} |
| **Ziyaret edilen sayfalar** | {COUNT} |
| **Ekran görüntüleri** | {COUNT} |
| **Çerçeve** | {DETECTED} veya "Bilinmiyor" |
| **Dizin** | [Tüm QA çalışmaları](./index.md) |

## Sağlık Puanı: {SCORE}/100

| Kategori | Puan |
|----------|------|
| Konsol | {0-100} |
| Bağlantılar | {0-100} |
| Görsel | {0-100} |
| İşlevsel | {0-100} |
| Kullanıcı Deneyimi | {0-100} |
| Performans | {0-100} |
| Erişilebilirlik | {0-100} |

## Düzeltilmesi Gereken İlk 3 Şey

1. **{ISSUE-NNN}: {başlık}** — {tek satır açıklama}
2. **{ISSUE-NNN}: {başlık}** — {tek satır açıklama}
3. **{ISSUE-NNN}: {başlık}** — {tek satır açıklama}

## Konsol Sağlığı

| Hata | Sayı | İlk görülme |
|------|------|-------------|
| {hata mesajı} | {N} | {URL} |

## Özet

| Ciddiyet | Sayı |
|----------|------|
| Kritik | 0 |
| Yüksek | 0 |
| Orta | 0 |
| Düşük | 0 |
| **Toplam** | **0** |

## Sorunlar

### ISSUE-001: {Kısa başlık}

| Alan | Değer |
|------|-------|
| **Ciddiyet** | kritik / yüksek / orta / düşük |
| **Kategori** | görsel / işlevsel / kullanıcı deneyimi / içerik / performans / konsol / erişilebilirlik |
| **URL** | {sayfa URL'si} |

**Açıklama:** {Neyin yanlış olduğu, beklenen vs gerçekleşen.}

**Yeniden Üretim Adımları:**

1. {URL} adresine git
   ![Adım 1](screenshots/issue-001-step-1.png)
2. {Eylem}
   ![Adım 2](screenshots/issue-001-step-2.png)
3. **Gözlemle:** {neyin yanlış gittiği}
   ![Sonuç](screenshots/issue-001-result.png)

---

## Uygulanan Düzeltmeler (varsa)

| Sorun | Düzeltme Durumu | Commit | Değiştirilen Dosyalar |
|-------|----------------|-------|----------------------|
| ISSUE-NNN | doğrulandı / en iyi çaba / geri alındı / ertelendi | {SHA} | {dosyalar} |

### Önce/Sonra Kanıtı

#### ISSUE-NNN: {başlık}
**Önce:** ![Önce](screenshots/issue-NNN-before.png)
**Sonra:** ![Sonra](screenshots/issue-NNN-after.png)

---

## Regresyon Testleri

| Sorun | Test Dosyası | Durum | Açıklama |
|-------|-------------|-------|----------|
| ISSUE-NNN | path/to/test | commit edildi / ertelendi / atlandı | açıklama |

### Ertelenen Testler

#### ISSUE-NNN: {başlık}
**Önkoşul:** {hatayı tetikleyen kurulum durumu}
**Eylem:** {kullanıcının yaptığı şey}
**Beklenen:** {doğru davranış}
**Neden ertelendi:** {neden}

---

## Yayına Hazırlık

| Metrik | Değer |
|--------|-------|
| Sağlık puanı | {önce} → {sonra} ({fark}) |
| Bulunan sorunlar | N |
| Uygulanan düzeltmeler | N (doğrulanan: X, en iyi çaba: Y, geri alman: Z) |
| Ertelenen | N |

**PR Özeti:** "QA N sorun buldu, M'sini düzeltti, sağlık puanı X → Y."

---

## Regresyon (varsa)

| Metrik | Temel | Güncel | Fark |
|--------|-------|--------|------|
| Sağlık puanı | {N} | {N} | {+/-N} |
| Sorunlar | {N} | {N} | {+/-N} |

**Temel değerden beri düzeltilen:** {liste}
**Temel değerden beri yeni:** {liste}