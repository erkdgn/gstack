# Düzeltme #1671: `/office-hours` her zaman SESSION_COUNT: 0 raporluyor

**Durum:** GÖNDERİLDİ
**Dal:** fix-1671-profile-migration
**Tarih:** 2026-05-23
**Sorun:** https://github.com/garrytan/gstack/issues/1671
**Hatayı tanıtan orijinal PR:** garrytan/gstack#1039 / commit `0a803f9` / v1.0.0.0 / 2026-04-18

## Sorun

`/office-hours`, skill'i birçok kez çalıştırmış kullanıcılar için bile her çağrıda `SESSION_COUNT: 0` ve `TIER: introduction` raporluyor. Geri dönen kullanıcılar için kapanış sunumunu atlamak amacıyla var olan `welcome_back` katmanı (`bin/gstack-developer-profile:165-169`) erişilemez değil. v1.0.0.0'dan bu yana tüm taze-`$HOME` kullanıcılarında ~5 haftadır canlı.

## Kök neden

v1.0.0.0 göçü, okuma yolunu `~/.gstack/developer-profile.json`'a taşıdı ama yazıcıyı `office-hours/SKILL.md.tmpl` içinde eski `~/.gstack/builder-profile.jsonl` dosyasına yazmaya bıraktı. İlk okumada oluşturulan `ensure_profile` koçanının `sessions: []` değeri var; sonraki yazımlar okuyucunun hiç tekrar okumadığı bir dosyaya gidiyor. Okuyucu ve yazıcı depolama konusunda anlaşmıyor.

Tam kök neden analizi (RC2/RC3 takipleri dahil): https://github.com/garrytan/gstack/issues/1671

## Düzeltme

Yazıcının okuyucunun kullandığı aynı dosyayı kullanmasını sağla.

### Değişiklikler

1. **`bin/gstack-developer-profile`** — `--log-session '<json>'` alt komutunu ekle:
   - Zorunlu alanları doğrular (`date`, `mode`), geçersiz girdide sessizce atlar (`bin/gstack-timeline-log:22-26` ile eşleşir).
   - `bun -e` ile mevcut `developer-profile.json`'u okur.
   - `sessions[]` dizisine giriş ekler. `signals_accumulated`'ı günceller (her-sinyal-dizesi artışı, `do_migrate:67-69` ile aynı), `resources_shown` ve `topics`'ı birleştirir.
   - Atomik mktemp+mv yazımı (54. satırdaki mevcut desenle eşleşir).
   - Yazımdan sonra `gstack-brain-enqueue "developer-profile.json"` çağırır, `bin/gstack-timeline-log:40` ile yansılar.

2. **`bin/gstack-developer-profile:do_read`** — LAST_PROJECT / LAST_ASSIGNMENT / LAST_DESIGN_TITLE / CROSS_PROJECT / DESIGN_* seçerken `mode:"resources"` girişlerini filtrele. Aşama 6 kaynakları, aynı /office-hours çağrısında gerçek oturumdan sonra otomatik eklenir; filtre olmadan bu kaynak girişi, kullanıcının bir sonraki oturumu için gerçek oturum durumunun üzerine yazar. Bozuk yazıcı tarafından maskelenen gizli hata; düzeltmeyle ortaya çıktı.

3. **`office-hours/SKILL.md.tmpl`** — 490. ve 893. satırlardaki yazıcıları değiştir:
   - Önceki: `echo '{...}' >> "$GSTACK_STATE_ROOT/builder-profile.jsonl"`
   - Sonraki: `~/.claude/skills/gstack/bin/gstack-developer-profile --log-session '{...}' 2>/dev/null || true`
   - `office-hours/SKILL.md`'yi yeniden oluşturmak için `bun run gen:skill-docs` çalıştır.

### Düzeltmede kasıtlı olarak YOK

- **Yeni ikili dosya yok.** `developer-profile.json`'un sahibi ikili dosyası `gstack-developer-profile`; yazıcı oraya bir alt komut olarak ait. `--log-session`, ikili dosyanın mevcut `--migrate` / `--derive` yazma tarafı alt komut sınırına katılır, `gstack-*-log` olay yazıcı ailesine değil. Fiil adı yine de `gstack-*-log` ile eşleşiyor.
- **mkdir-kilitleri yok.** Eşzamanlı /office-hours çağrıları `developer-profile.json` üzerinde oku-değiştir-yaz yarışı içeriyor. Kod tabanı aynı yarışı `gstack-config`'te kabul ediyor (YAML üzerinde r-m-w, kilit yok). Bu düzeltmeyle tanıtılmadı; kapsam dışı.
- **Şema yükseltmesi yok.** Şema `schema_version: 1` olarak kalıyor. Düzeltme şemayı değiştirmiyor, sadece yazıcının onu kullanmasını sağlıyor.
- **Etkilenen kullanıcılar için otomatik uzlaştırma yok.** Mahsur kalmış `builder-profile.jsonl` girişlerine sahip mevcut kullanıcıların geçmiş verileri `developer-profile.json`'a otomatik olarak birleştirilmez. Bir sonraki /office-hours çalıştırmalarında ilk yeni oturum `welcome_back`'e düşer; geçmiş veriler eski dosyada kalır (kullanım dışı bırakma süresince diğer araçlar tarafından hala okunabilir). Çoğu etkilenen kullanıcının yalızca birkaç mahsur kalmış oturumu var, bu yüzden kayıp çoğunlukla estetik. Tek-sürüm uzlaştırma yolu, net gürültü olarak değerlendirilip bırakıldı — Garry'nin "hak edilmiş diff" yaklaşımı.
- **Autoplan zaman çizelgesi özeti yok (RC2).** Ayrı concern, ayrı PR.
- **Proje kapsamı katılımı yok (RC3).** Ayrı concern, ayrı PR.
- **gbrain glob değişikliği yok.** office-hours manifestosu hala bağlam için `~/.gstack/builder-profile.jsonl`'ı globluyor; yeni yazımlar oraya düşmeyi bıraktığında, anlık görüntü soğuyor. Bir UX sorunu haline gelirse takip güncellemesinde ele al.

### Testler (tümü gate-tier, ücretsiz, deterministik)

1. **Regresyon testi** `test/gstack-developer-profile.test.ts` içinde:
   - Taze `$HOME`.
   - /office-hours önyazısını çalıştır: gstack-developer-profile boş koçan oluşturur.
   - Bir başlatma modu JSON'u ile `--log-session` çağır.
   - `--read`'i tekrar çalıştır. `SESSION_COUNT: 1`, `TIER: welcome_back` olduğunu doğrula.
   - Mevcut main dalında başarısız olur (alt komut mevcut değil). Düzeltmeyle geçer.

2. **`do_read` mod filtre testi:** bir başlatma oturumu ve ardından bir kaynak girişi kaydettikten sonra, `--read` gerçek oturumdan LAST_PROJECT / LAST_ASSIGNMENT / LAST_DESIGN_TITLE döndürür, kaynak girişinden değil. RESOURCES_SHOWN hala doğru şekilde birleştirir.

3. **Doğrulama + birleştirme testleri:** `--log-session` geçersiz JSON / eksik zorunlu alanları sessizce atlar, eksikse `ts` enjekte eder, kullanıcı tarafından ayarlanmış `ts`'yi korur, sinyalleri/kaynakları/konuları birden fazla oturumda doğru şekilde birleştirir.

4. **Statik-grep değişmezi** `test/static-no-legacy-writes.test.ts` içinde (yeni): her skill dizinini gezin, izin verilenler listesindeki okuyucular (`gstack-developer-profile`, `gstack-memory-ingest.ts`, `gstack-artifacts-init`, doc dosyaları) dışında hiçbir üretim kodu yolunun `builder-profile.jsonl`'a yazmadığını doğrular. Gelecekteki yazıcıların eski dosyaya geri dönmesini engeller.

### Kabul kriterleri

- Taze bir `$HOME` üzerinde ikinci `/office-hours` çağrısı `TIER: welcome_back` döndürür.
- `bun test` dokunulan dosyalarda izole olarak geçer.
- `bun run gen:skill-docs` `.tmpl` düzenlemeleriyle eşleşen temiz diff üretir.

### Dağıtım

- Tek commit. CHANGELOG stil kılavuzuna göre YAMA sürüm artışı.
- `/ship` tarafından yazılan CHANGELOG girdisi. Kullanıcıya yönelik ses: kullanıcıların şimdi sahip oldukları ve önce sahip olmadıkları şeylerle başlayın (welcome_back katmanı ikinci ziyarette devreye girer).

## Takip Yapılacaklar

- `builder-profile.jsonl`'ı tamamen kullanım dışı bırak (yazıcı + shim + memory-ingest türü) bir sürüm sonra.
- RC2'yi düzelt (autoplan satır içi alt-skill'ler, zaman çizelgesi günlüğü önyazılarını atlıyor).
- Birden fazla ajan kimliğine sahip güçlü kullanıcılar için `GSTACK_PROFILE_SCOPE` katılımı ekle (RC3).
- `/plan-tune` şu anda `--derive` çağırmıyor, bu yüzden `inferred`/`gap` kayabilir (önceden var olan, #1671 ile ilgisi yok).
- `mode:"resources"` girişleri mevcut katman toplayıcısı altında SESSION_COUNT'u şişirir (önceden var olan, #1671 kök nedeniyle ilgisi yok).