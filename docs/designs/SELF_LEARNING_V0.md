# Tasarım: GStack Kendi Kendini Öğrenme Altyapısı

/office-hours + /plan-ceo-review + /plan-eng-review tarafından 2026-03-28 tarihinde oluşturuldu
Güncelleme: 2026-04-01 (Oturum Zekasından sonra, Codex tarafından gözden geçirildi)
Dal: garrytan/ce-features
Repo: gstack
Durum: AKTİF
Mod: Açık Kaynak / Topluluk

## Sorun Bildirimi

GStack oturumlar arasında 30+ yetenek çalıştırıyor ama aralarında hiçbir şey öğrenmiyor. Bir /review oturumu bir N+1 sorgu kalıbını yakalıyor ve aynı kod tabanında bir sonraki /review sıfırdan başlıyor. Bir /ship çalışması test komutunu keşfediyor ve gelecekteki her /ship bunu yeniden keşfediyor. Bir /investigate zorlu bir yarış durumu buluyor ve hiçbir gelecek oturum bunu bilmiyor.

Her AI kodlama aracında bu sorun var. Cursor'ın kullanıcı başına belleği var. Claude Code'un CLAUDE.md'si var. Windsurf'un kalıcı bağlamı var. Ama hiçbiri bileşikleşmiyor. Hiçbiri öğrendiklerini yapılandırmıyor. Hiçbiri bilgileri yetenekler arasında paylaşmıyor.

## Ne İnşa Ediyoruz

Oturumlar ve yetenekler arasında bileşikleşen proje bazlı kurumsal bilgi. Yapılandırılmış, türlü, güven puanlı öğrenmeler ki her gstack yeteneği okuyup yazabiliyor. Hedef: aynı kod tabanında 20 oturumdan sonra, gstack her mimari kararı, her geçmiş hata kalıbını ve her yanlış olduğu anı biliyor.

## Kuzey Yıldızı

/autoship (Sürüm 5). Tek bir komutta tam mühendislik ekibi. Bir özellik tanımlayın, planı onaylayın, diğer her şey otomatik. /autoship öğrenmeler (R1), inceleme kalitesi (R2), oturum sürekliliği (R3) ve uyarlanır seremoni (R4) olmadan çalışamaz. Sürümler 1-4, /autoship'in gerçekten çalışmasını sağlayan altyapıdır.

## Hedef Kitle

YC kurucuları AI ile inşa ediyor. GStack'i gerçek kod tabanlarında haftada 20+ kez çalıştıran ve aynı soruyu iki kez sorduğunu fark eden kişiler.

## Farklılaştırma

| Araç | Bellek modeli | Kapsam | Yapı |
|------|-------------|-------|-----------|
| Cursor | Kullanıcı başına sohbet belleği | Oturum başına | Yapılandırılmamış |
| CLAUDE.md | Statik dosya | Proje başına | Manuel |
| Windsurf | Kalıcı bağlam | Oturum başına | Yapılandırılmamış |
| **GStack** | **Proje başına JSONL** | **Oturumlar arası, yetenekler arası** | **Türlü, puanlı, azalan** |

---

## Durum Sistemleri

gstack dört farklı kalıcılık katmanına sahiptir. Depolama kalıplarını paylaşıyorlar (`~/.gstack/projects/$SLUG/` içinde JSONL) ama farklı amaçlara hizmet ediyorlar:

| Sistem | Dosya | Ne saklar | Yazan | Okuyan |
|--------|------|---------------|------------|---------|
| **Öğrenmeler** | `learnings.jsonl` | Kurumsal bilgi (tuzaklar, kalıplar, tercihler) | Tüm yetenekler | Tüm yetenekler (preamble) |
| **Zaman Çizelgesi** | `timeline.jsonl` | Olay geçmişi (yetenek başlangıç/bitiş, dal, sonuç) | Preamble (otomatik) | /retro, preamble bağlam kurtarma |
| **Kontrol Noktaları** | `checkpoints/*.md` | Çalışma durumu anlık görüntüleri (kararlar, kalan iş, dosyalar) | /checkpoint, /ship, /investigate | Preamble bağlam kurtarma, /checkpoint devam |
| **Sağlık** | `health-history.jsonl` | Zaman içinde kod kalite puanları (araç başına, bileşik) | /health | /retro, /ship (geçit), /health (trendler) |

Bunlar örtüşmüyor. Öğrenmeler = bildiklerin. Zaman çizelgesi = ne oldu. Kontrol noktaları = neredesin. Sağlık = kod ne kadar iyi. Her biri farklı bir soruyu yanıtlıyor.

---

## Sürüm Yol Haritası

### Sürüm 1: "GStack Öğreniyor" (v0.13-0.14) — GÖNDERİLDİ

**Başlık:** Her oturum bir sonrakini daha akıllı yapar.

Gönderilenler:
- `~/.gstack/projects/{slug}/learnings.jsonl` konumunda öğrenmeler kalıcılığı
- Manuel inceleme, arama, budama, dışa aktarma için `/learn` yeteneği
- Tüm inceleme bulgularında güven puanı kalibrasyonu (1-10 puanlar ve görüntüleme kuralları)
- Gözlemlenen/çıkarılan öğrenmeler için güven azalması (1puan/30gün)
- Çapraz proje öğrenmeleri keşfi (isteğe bağlı, AskUserQuestion onayı)
- İncelemeler geçmiş öğrenmelerle eşleştiğinde "Öğrenme uygulandı" çağrıları
- /review, /ship, /plan-*, /office-hours, /investigate, /retro entegrasyonu

Şema:
```json
{
  "ts": "2026-03-28T12:00:00Z",
  "skill": "review",
  "type": "pitfall",
  "key": "n-plus-one-activerecord",
  "insight": "list endpoint'lerinde her zaman includes() kontrolü yapın",
  "confidence": 8,
  "source": "observed",
  "branch": "feature-x",
  "commit": "abc1234",
  "files": ["app/models/user.rb"]
}
```

Türler: `pattern` | `pitfall` | `preference` | `architecture` | `tool`
Kaynaklar: `observed` | `user-stated` | `inferred` | `cross-model`

Mimari: Sadece ekleme JSONL. Yinelenenler okuma zamanında çözülür ("en son kazanan", anahtar+tür başına). Yazma zamanında mutasyon yok, yarış durumu yok.

### Sürüm 2: "İnceleme Ordusu" (v0.14.3-0.14.4) — GÖNDERİLDİ

**Başlık:** Her PR'da 10 uzman inceleyici.

Gönderilenler:
- 7 paralel uzman alt ajans: her zaman açık (test, bakım yapılabilirlik) + koşullu (güvenlik, performans, veri geçişi, API sözleşmesi, tasarım) + kırmızı takım (büyük diff'ler / kritik bulgular)
- Güven puanları + parmak izi yinelenen kaldırma ile JSON yapılandırılmış bulgular
- PR kalite puanı (0-10) inceleme başına kaydedildi + /retro trendleme
- Öğrenme-bilgilendirilmiş uzman promptları, geçmiş tuzaklar alan başına enjekte edildi
- Çoklu-uzman fikir birliği vurgulama, onaylanan bulgular yükseltildi
- PLAN_COMPLETION_AUDIT aracılığıyla geliştirilmiş Teslimat Bütünlüğü
- Kontrol listesi yeniden yapılandırıldı: KRİTİK kategoriler ana geçişte kalır, uzman kategorileri review/specialists/ odaklı kontrol listelerine çıkarıldı

### Sürüm 2.5: "İnceleme Ordusu Genişletmeleri" — HENÜZ GÖNDERİLMEDİ

**Başlık:** R2 kararlı kanıtlandıktan sonra gönder. Temel döngünün nasıl performans gösterdiğini kontrol et.

Ön kontrol: R2 kalite metriklerini incele (PR kalite puanları, uzman bulgu oranları, yanlış pozitif oranları, Uçtan uca test kararlılığı). Temel döngüde sorunlar varsa, önce onları düzelt.

Gönderilecekler:
- E1: Uyarlanır uzman geçitlendirme, 0-bulgu kayıt geçmişine sahip uzmanları otomatik atla. Proje başına bulgu oranlarını gstack-learnings-log ile sakla. Kullanıcı --security vb. ile zorlayabilir.
- E3: Test taslağı oluşturma, her uzman bulguların yanı sıra TEST_STUB çıktısı. Çerçeve projeden algılanır (Jest/Vitest/RSpec/pytest/Go test). Fix-First'a akar: AUTO-FIX düzeltmeyi uygular + test dosyası oluşturur.
- E5: Çapraz inceleme bulgu yinelenen kaldırma, önceki inceleme girdileri için gstack-review-read oku. Önceki kullanıcı tarafından atlanmış bulguyla eşleşen bulguları bastır.
- E7: Uzman performans takibi, uzman başına metrikleri gstack-review-log ile kaydet. Zaman çizelgesi entegrasyonu: uzman çalışmaları /retro trendleme için timeline.jsonl'de görünür.

### Sürüm 3: "Oturum Zekası" (v0.15.0) — GÖNDERİLDİ

**Başlık:** AI oturumlarınız ne olduğunu hatırlıyor.

Gönderilenler:
- Oturum zaman çizelgesi: her yetenek başlangıç/bitiş olaylarını otomatik olarak `~/.gstack/projects/$SLUG/timeline.jsonl` dosyasına kaydeder. Sadece yerel, hiçbir yere gönderilmiyor, telemetri ayarından bağımsız her zaman açık.
- Bağlam kurtarma: sıkıştırma veya oturum başlangıcından sonra, preamble en son CEO planlarını, kontrol noktalarını ve incelemeleri listeler. Aracı en yeniyi okuyarak bağlamı kurtarır.
- Oturumlar arası enjeksiyon: preamble geçerli dal için LAST_SESSION ve LATEST_CHECKPOINT yazdırır. Bir şey yazmadan önce nerede kaldığınızı görürsünüz.
- Öngörücü yetenek önerisi: son 3 oturumunuz bir kalıp izliyorsa (review, ship, review), gstack muhtemelen istediğiniz şeyi önerir.
- Oturum başlangıcında "Tekrar hoş geldiniz" sentezlenmiş bağlam mesajı.
- `/checkpoint` yeteneği: çalışma durumu anlık görüntülerini kaydet/sürdür/listele. Ajanslar arası Conductor çalışma alanı devri için dallar arası listeleme.
- `/health` yeteneği: proje araçlarını saran kod kalite puanlayıcı (tsc, biome, knip, shellcheck, testler). Bileşik 0-10 puan, trend takibi, puanlar düştüğünde iyileştirme önerileri.
- Zaman çizelgesi ikili dosyaları: `bin/gstack-timeline-log` ve `bin/gstack-timeline-read`.
- Yönlendirme kuralları: /checkpoint ve /health preamble yetenek yönlendirmesine eklendi.

Tasarım belgesi: `docs/designs/SESSION_INTELLIGENCE.md`

### Sürüm 4: "Uyarlanır Seremoni" — HENÜZ GÖNDERİLMEDİ

**Başlık:** GStack güvenliğinizden ödün vermeden zamanınıza saygı duyar.

Seremoni ve güven farklı kaygılardır. Seremoni = bir PR'nin geçtiği inceleme/test/QA adımları kümesi. Güven = hangi seremoni seviyesinin geçerli olduğunu belirleyen bir politika motoru. Etkileşirler ama birleşmezler.

Gönderilecekler:

**Seremoni seviyeleri:**
- FULL: tüm uzmanlar, çelişkili, Codex yapılandırılmış inceleme, kapsama denetimi, plan tamamlama. Büyük diff'ler, yeni özellikler, geçişler, auth değişiklikleri için.
- STANDARD: çelişkili + Codex, kapsama denetimi, plan tamamlama. Orta diff'ler, tipik özellik çalışması için.
- FAST: sadece çelişkili. Güvenilen projelerde küçük, iyi test edilmiş değişiklikler için.

**Güven politika motoru:**
- Kapsam duyarlı güven. Güven küresel olarak değil, değişiklik sınıfı başına kazanılır. Sadece doküman PR'lerinde temiz geçmiş, geçiş PR'lerinde güven satın almaz.
- Değişiklik sınıfı algılama: dokümanlar, testler, config, frontend, backend, geçişler, auth, altyapı. Her sınıfın kendi güven eşiği var.
- Güven sinyalleri: ardışık temiz incelemeler (sınıf başına), /health puanı kararlılığı, regresyon sıklığı, test kapsama trendleri.
- Güven asla hızlı takip yapmaz: geçişler, auth/izin değişiklikleri, yeni API endpoint'leri, altyapı değişiklikleri. Bunlar güven seviyesinden bağımsız her zaman FULL seremoni alır.
- Aşamalı bozulma, ikili sıfırlama değil. Tek bir regresyon tüm güveni sıfırlamaz. O değişiklik sınıfı için güveni bir seviye düşürür.

**Kapsam değerlendirmesi:**
- /review, /ship, /autoplan içinde TINY/SMALL/MEDIUM/LARGE sınıflandırma diff boyutuna, dokunulan dosyalara ve değişiklik sınıfına dayalı.
- Seremoni seviyesi = f(kapsam, güven, değişiklik sınıfı).

**TODO yaşam döngüsü:**
- Gelen TODO'ların interaktif onayı için /triage
- Paralel ajanslar aracılığıyla toplu çözüm için /resolve

### Sürüm 5: "/autoship — Tek Komut, Tam Özellik" — HENÜZ GÖNDERİLMEDİ

**Başlık:** Bir özellik tanımlayın. Planı onaylayın. Geri kalan her şey otomatik.

/autoship devam ettirilebilir bir durum makinesidir, doğrusal bir boru hattı değil. İnceleme ve QA işi build/fix'e geri gönderebilir. Sıkıştırma herhangi bir aşamayı kesebilir. Sistem zarif bir şekilde kurtarabilmelidir.

```
                    ┌──────────┐
                    │  START   │
                    └────┬─────┘
                         │
                    ┌────▼─────┐
                    │ /office- │
                    │  hours   │
                    └────┬─────┘
                         │
                    ┌────▼─────┐
                    │/autoplan │ ◄── tek onay geçidi
                    └────┬─────┘
                         │
              ┌──────────▼──────────┐
              │       BUILD         │ ◄── /checkpoint otomatik kaydet
              └──────────┬──────────┘
                         │
              ┌──────────▼──────────┐
              │      /health        │ ◄── kalite geçidi
              │   (score >= 7.0)    │
              └──────────┬──────────┘
                         │ başarısız → BUILD'e geri
              ┌──────────▼──────────┐
              │      /review        │
              └──────────┬──────────┘
                         │ ASK öğeleri → BUILD'e geri
              ┌──────────▼──────────┐
              │        /qa          │
              └──────────┬──────────┘
                         │ hatalar bulundu → BUILD'e geri
              ┌──────────▼──────────┐
              │       /ship         │
              └──────────┬──────────┘
                         │
              ┌──────────▼──────────┐
              │ /checkpoint archive │ ◄── koru, yok etme
              └─────────────────────┘
```

Gönderilecekler:
- Yukarıdaki durum makinesi ile /autoship otonom boru hattı.
  Her aşama timeline.jsonl'e yazar. Kontrol noktaları her aşama öncesi otomatik kaydeder.
  Sıkıştırma kurtarma: kontrol noktası + zaman çizelgesini okur, son tamamlanan aşamada devam eder.
- Tamamlanmada kontrol noktası arşivleme (silme değil). Kurtarma durumu başarısız autoship çalışmaları için hata ayıklama amacıyla korunur.
- /ideate beyin fırtınası yeteneği (paralel ıraksak ajanslar + çelişkili filtreleme)
- /plan-eng-review içinde araştırma ajansları (kod tabanı analisti, geçmiş analisti, en iyi uygulamalar araştırmacısı, öğrenmeler araştırmacısı)

Bağımlılıklar: R1 (araştırma ajansları için öğrenmeler), R2 (kalite için inceleme ordusu), R3 (süreklilik için oturum zekası), R4 (hız için uyarlanır seremoni).

### Sürüm 6: "Yürütme Stüdyosu" — HENÜZ GÖNDERİLMEDİ

**Başlık:** Paralel yürütme altyapısı.

Gönderilecekler:
- Swarm orkestrasyonu: çoklu-worktree paralel derlemeler. /checkpoint'ten (R3) Conductor çalışma alanı devri üzerine inşa eder. Bir orkestratör yeteneği bağımsız iş akışlarını paralel ajanslara gönderir, her biri kendi worktree'si ile.
- Codex derleme delegasyonu: görev türüne göre Codex CLI'ye uygulama delegasyonunu otomatik algıla (boilerplate, test oluşturma, mekanik yeniden düzenlemeler).
- PR geri bildirim çözümü: inceleme platformları arası paralel yorum çözücüsü.
- /onboard: kod tabanı analizinden otomatik oluşturulmuş katılımcı rehberi.
- /triage-prs: bakımcılar için toplu PR triyajı.

### Sürüm 7: "Tasarım & Medya" — HENÜZ GÖNDERİLMEDİ

**Başlık:** Görsel tasarım entegrasyonu.

Gönderilecekler:
- Figma tasarım senkronizasyonu (piksel eşleştirme yineleme döngüsü)
- Özellik video kaydı (otomatik oluşturulmuş PR demoları)
- Çapraz platform taşınabilirliği (Copilot, Kiro, Windsurf çıktısı)

---

## Risk Kaydı

### İncelemeyi atlamak için izin olarak vekil sinyaller
(Codex incelemesi tarafından tanımlandı, 2026-04-01)

/health puanları, temiz inceleme geçmişi ve zaman çizelgesi kalıpları kullanışlı sinyallerdir. Güvenlik kanıtı değillerdir. Bu sinyaller seremoni azaltmasını VE /autoship'i beslerse, başarısızlık modu nadir, sessiz, yüksek şiddetli hatalardır. Azaltıcı önlemler:
- Belirli değişiklik sınıfları asla hızlı takip yapmaz (geçişler, auth, altyapı, yeni endpoint'ler).
- Güven aşamalı olarak bozulur, ikili sıfırlama değil.
- /autoship proje başına ilk çalışmasında her zaman FULL seremoni çalıştırır. Güven kazanılır.

### Bayat bağlam kurtarma
(Codex incelemesi tarafından tanımlandı, 2026-04-01)

Bağlam kurtarma yanlış dal durumu, eski planlar veya geçersiz kontrol noktaları enjekte edebilir. Azaltıcı önlemler:
- Kontrol noktaları YAML ön bilgisinde dal adı içerir. Bağlam kurtarma geçerli dala göre filtreler.
- Zaman çizelgesi grep LAST_SESSION göstermeden önce dala göre filtreler.
- Bayat yapıt algılama: kontrol noktası >7 günlükse, muhtemelen bayat olarak şu anki olarak sunmak yerine not olarak işaretle.

### Doğrulama metrikleri gerekli
(Codex incelemesi tarafından tanımlandı, 2026-04-01)

R4 (Uyarlanır Seremoni) gönderilmeden önce ölçün:
- Öngörücü öneri doğruluğu (kullanıcı önerilen yeteneği çalıştırdı mı?)
- Güven politikası yanlış-atlama oranı (hızlı takip yapılan PR'larda birleştirme sonrası sorunlar oldu mu?)
- Bağlam kurtarma doğruluğu (kurtarılan bağlam gerçek durumuyla eşleşti mi?)
- /health puanı korelasyonu gerçek kod kalitesi ile (yüksek puanlar daha az üretim hatası öngörüyor mu?)

Bu metrikler R3 kullanımı sırasında toplanmalı ve R4 gönderilmeden önce incelenmelidir.

---

## Teşekkür Edilen İlham

Kendi kendini öğrenme yol haritası, Nico Bailon'un [Compound Engineering](https://github.com/nicobailon/compound-engineering) projesindeki fikirlerden ilham aldı. Öğrenmeler kalıcılığı, paralel inceleme ajansları ve otonom boru hatları konusundaki keşifleri, GStack'in yaklaşımının tasarımını katalize etti. Her kavramı GStack'in şablon sistemine, sesine ve mimarisine uyacak şekilde uyarladık, doğrudan taşımadık.