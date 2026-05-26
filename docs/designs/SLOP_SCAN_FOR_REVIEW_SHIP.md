# Tasarım: /review ve /ship içinde slop-scan entegrasyonu

Durum: ertelendi
Oluşturulma: 2026-04-09
Bağımlılıklar: slop-diff betiği (scripts/slop-diff.ts, zaten yerleşmiş)

## Sorun

slop-scan bulguları sadece `bun run slop:diff` komutunu manuel olarak çalıştırırsanız görünür. Kod incelemesi ve gönderim sırasında otomatik olarak yüzeye çıkmalıdır, SQL güvenliği ve güven sınırı kontrollerinin yaptığı gibi.

## Entegrasyon noktaları

### /review (Adım 4, kontrol listesi geçişinden sonra)

Kritik/bilgilendirici kontrol listesi geçişinden sonra `bun run slop:diff` çalıştırın. Yeni bulguları diğer inceleme çıktısıyla birlikte gösterin:

```
Gönderim Öncesi İnceleme: 3 sorun (1 kritik, 2 bilgilendirici)

AI Slop: +2 yeni bulgu, -0 kaldırılan
  browse/src/new-feature.ts
    defensive.empty-catch: 2 konum
      satır 42: boş catch, boundary=filesystem
      satır 87: boş catch, boundary=process
```

Sınıflandırma: BİLGİLENDİRİCİ (asla birleştirmeyi engellemez, sadece kalıbı gösterir).

Fix-First buluşsal kuralı geçerlidir: bulgu bir dosya işlemi etrafındaki boş bir catchesa, `safeUnlink()` ile otomatik düzelt. Uzantı kodundaki bir catch-and-log ise, atla (CLAUDE.md yönergelerine göre bu doğru kalıptır).

### /ship (Adım 3.5, gönderim öncesi inceleme + PR gövdesi)

/review ile aynı entegrasyon. Ek olarak, PR gövdesinde tek satırlık bir özet göster:

```markdown
## Gönderim Öncesi İnceleme
- 2 sorun otomatik düzeltildi, 0 giriş gerektiriyor
- AI Slop: +0 yeni / -3 kaldırılan ✓
```

### İnceleme Hazırlık Panosu

Satır EKLEMEYİN. Slop, incelemeden bağımsız olarak "çalıştırılan" bir inceleme değil, diff üzerindeki bir tanıdır. Eng Review çıktısının içinde görünür, kendi pano girişi olarak değil.

## Ne otomatik düzeltilmeli vs ne atlanmalı

CLAUDE.md "Slop-scan" bölümünü takip et. Özet:

**Otomatik düzelt (gerçek kalite iyileştirmeleri):**
- `fs.unlinkSync` etrafındaki boş catch → `safeUnlink()` ile değiştir
- `process.kill` etrafındaki boş catch → `safeKill()` ile değiştir
- Kapsayan try olmadan `return await` → `await` kaldır
- URL ayrıştırma etrafındaki türlenmemiş catch → `instanceof TypeError` kontrolü ekle

**Atla (slop-scan'ın bayrakladığı doğru kalıplar):**
- Fire-and-forget browser işlemlerinde `.catch(() => {})` (page.close, bringToFront)
- Chrome uzantı kodunda catch-and-log (yakalanmayan hatalar uzantıları çökertir)
- Kapatma/acil durum yollarında `safeUnlinkQuiet` (tüm hataları yutmak doğrudur)
- Aktif oturuma devreden geçiş sarmalayıcıları (API kararlılık katmanı)

## Uygulama notları

- `scripts/slop-diff.ts` zaten ağır işi halleder (worktree tabanlı temel karşılaştırma, satır numarası duyarsız parmak izi, zarif geri dönüş)
- review/ship yetenekleri bash blokları çalıştırır. Entegrasyon şudur: betiği çalıştır, çıktıyı ayrıştır, inceleme bulgularına dahil et
- slop-scan kurulu değilse (`npx slop-scan` başarısız olursa), sessizce atla
- Betik her zaman 0 çıkar (tanısal, asla geçit yapmaz)

## Efor tahmini

| Görev | İnsan | CC+gstack |
|------|-------|-----------|
| review/SKILL.md.tmpl'e ekle | 2 saat | 10 dk |
| ship/SKILL.md.tmpl'e ekle | 2 saat | 10 dk |
| review/checklist.md'e ekle | 1 saat | 5 dk |
| Gerçek PR'lerle test et | 2 saat | 15 dk |
| SKILL.md dosyalarını yeniden oluştur | — | 1 dk |