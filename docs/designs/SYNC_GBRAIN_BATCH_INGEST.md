# /sync-gbrain toplu ingest geçişi

**Durum:** garrytan/dublin-v1 üzerinde uygulandı (D1-D8 kararları bu PR'da yerleşir)
**Dal:** garrytan/dublin-v1
**Sahip:** Garry Tan
**Tetikleyen:** /investigate çalışması, 2026-05-09
**Tahmini efor:** insan ~3 gün / CC+gstack ~2 saat
**Dokunulan dosyalar:** 4 kaynak + 1 test = 5 toplam (tahminin altında)

## Kararlar (inceleme sonrası)

Bu belge orijinal mimariyi yakalar. Son mimari, `/Users/garrytan/.claude/plans/purrfect-tumbling-quiche.md` konumundaki 8 inceleme kararına göre yerleşir:

- **D1** hiyerarşik hazırlık dizini (slug segment'i başına mkdir -p) — korundu
- **D2** aynı PR'de kes ve eskiyi sil ( `--legacy-ingest` bayrağı yok) — korundu
- **D3** önce kaynak dosyayı tara, sadece temiz olanları hazırla — korundu
- **D4** ~~üç durumlı OK/DEGRADED/ERR kararı~~ Codex bulgusu 7'ye göre ÇÖKERTİLDİ, OK/ERR olarak (gbrain content_hash idempotensi üçüncü durumu gereksiz kılar)
- **D5** ~~durum şemasında skip_reason alanı~~ Codex bulgusu 7'ye göre DÜŞÜRÜLDÜ (yeniden çalıştırmalar ucuz; kalıcı atlama takibi gerekmez)
- **D6** gbrain'in content_hash idempotensine güven; muhasebe iskelet iniş at (skip_reason, üç-durum, SIGTERM kontrol noktası)
- **D7** `~/.gbrain/sync-failures.jsonl` üzerinden dosya başına hata algılama (bayt-ofset anlık görüntüsü + sadece eklemeli okuma)
- **D8** 3 kapsam içi önceden var olan düzeltmeyi paketle: F6 atomik saveState (tmp+rename), F8 izole hazırlık kıyaslama, F9 tam dosya sha256 hash (1MB sınırı yok)

## gbrain kaynağından doğrulandı

`~/git/gbrain/src/` okunarak doğrulanan üç özellik:

- **Idempotensi** `core/import-file.ts:242-243, :478` konumunda — content_hash kontrolü, değişmediyse atla, değiştiyse üzerine yaz.
- **Ön bilgi eşliği** `core/import-file.ts:228, 297, 410-422` konumunda — başlık/tür/etiketler onurlandı; ön bilgi sadece ön bilgi olmadığında çıkarım yapılır.
- **Yol-otorite slug'ı** `core/sync.ts:260` konumunda (`slugifyPath`), `core/import-file.ts:429` konumunda uygulanır.
- **Dosya başına hatalar yüzeye çıkar** `commands/import.ts:308-310` konumunda, `:28` yorumunda: "çağıranlar durum ilerlemelerini geçitleyebilir" — D7'nin kullandığı şey için kasıtlı API.

## Performans: planlanan vs ölçülen (2026-05-10 perf incelemesi sonrası)

| Metrik | Plan hedefi | Ölçülen | Karar |
|---|---|---|---|
| 5135 dosya üzerinde hazırlık aşaması | — | <10s | HIZLI |
| 5135 dosya üzerinde `gbrain import` | — | >10 dk | gbrain tarafı perf sorunu, dosyalandı |
| Döngü / askıda kalma (orijinal hata) | hiçbir zaman | hiçbir zaman | DÜZELTİLDİ |
| Bellek ingest null üzerinde SIGTERM'den çıkar | hayır | hayır — durum yazmaları başarılı; alt gbrain üst süreçle birlikte ölür | DÜZELTİLDİ |
| FILE_TOO_LARGE last_commit'i engeller | hayır | hayır — başarısız yollar D7 ile dışlandı | DÜZELTİLDİ |

**İlk perf kaçırmayı + düzeltme.** İlk soğuk çalıştırma ölçümü (~12 dk) 1841 sıralı gitleaks alt süreç başlatması tarafından domine edildi, her biri ~256ms — gereksiz bir güvenlik geçidi. Makineler arası sızdırma sınırı `gstack-brain-sync`'tir (hazırlanmış diff üzerinde `git commit` öncesinde regex tabanlı gizli taraması, bin/gstack-brain-sync:78-110). Yerel bir PGLite'ye ingestten önce her kaynak dosyayı taramak maruziyeti değiştirmez — gizli zaten diskte düz metin olarak var. Gitleaks'i dosya başı `--scan-secrets` ile opsiyonel yaptık. Varsayılan kapalı. Bu, hazırlık aşamasını ~12 dk'dan 10 saniyenin altına indirdi.

Kalan soğuk çalıştırma maliyeti `gbrain import`'un kendisidir, büyük hazırlık dizilerinde doğrusaldan daha kötü ölçeklenir (501 dosya için 10s; 5031 için >10 dk). Bu bir gbrain tarafı perf sorunu, gstack mimarisi değil. Olarak TODO dosyalandı; düzeltme muhtemelen gbrain'in content_hash kontrol döngüsünde veya auto-link mutabakat aşamasında.

## F9 hash geçişi (tek seferlik uçurum)

F9, `fileSha256`'yı 1MB sınırlandırılmış hash'ten tam dosya hash'ine geçirdi. Bu değişiklikten önceki mevcut durum girdileri eski 1MB sınırlandırılmış hash'i taşır. mtime'u değişmeyen herhangi bir dosya için, `fileChangedSinceState` mtime kontrolünde false döndürür ve yeni hash hiç hesaplanmaz — bu nedenle değişmeyen dosyalar aynı şekilde davranır. Yükseltmeden sonra mtime'u DEĞİŞEN herhangi bir dosya için, tam dosya hash'i yeniden hesaplanır ve (doğru olarak) değiştirilmiş olarak kabul edilir, ardından yeniden içe aktarılır. `gbrain doctor` prob raporunun `updated_count` değeri, yükseltme sonrası ilk çalıştırmada her dokunulan dosya algoritma sınırını geçtiği için şişmiş gösterebilir. Veri kaybı yok, ama bilmeye değer.

## Takip çalışmalar (TODO olarak dosyalandı)

1. **Büyük dizilerde gbrain import perf** — 501 dosya 10s sürerken 5031 dosyanın >10 dk sürmesinin nedenini araştır. Olası suçlular: `getPage(slug)` content_hash kontrolü için N+1 SQL, sayfa başına auto-link mutabakatı, toplu işlemsiz FTS dizin güncellemeleri. gstack'de değil, gbrain'de.
2. **Opsiyonel: kaynak dosya değişiklik algılama önbelleği** — hazırlık aşaması hızlı olsa bile, 5031 dosyayı taramak biraz zaman alır. "Başarılı ingestten bu yana değişiklik yok" durumunu dosya başına değil toplu iş düzeyinde önbelleğe almak, hiçbir şey yapılmayan artımlı bir çalıştırmada hazırlık aşamasını tamamen atlar.

## Sorun

`/sync-gbrain` bellek hazırlık aşaması taze bir PGLite üzerinde 35 dakika sürüyor ve null çıkıyor, tüm ilerlemeyi kaybediyor. Sonraki çalışmalar aynı 35 dakikayı yeniden yapıyor. İki ardışık çalıştırmada gözlemlendi (gbrain 0.30.0 bozuk-postgres çalıştırması: 712s null-çıkış; gbrain 0.31.2 PGLite çalıştırması: 2100s null-çıkış, 501 sayfa gerçekten kalıcı).

## Kök nedeni (/investigate'ten)

`bin/gstack-memory-ingest.ts` içinde iki birleşen hata:

1. **Dosya başına alt süreç mimarisi.** 911. satırdaki ingest döngüsü `~/.gstack/projects/` içinde 1,841 dosyayı taramakta ve dosya başına iki alt süreç başlatmaktadır:
   - `gitleaks detect --no-git --source <path>` — 46ms soğuk başlangıç (`lib/gstack-memory-helpers.ts:157`)
   - `gbrain put <slug>` — 329ms soğuk başlangıç (`bin/gstack-memory-ingest.ts:823`)
   - Dosya başına alt sınır: 375ms × 1841 = 690s (11.5 dk) gerçek iş başlamadan önce sadece alt süreç başlatma.

2. **Öldür-kaydetme zaman aşımı.** `bin/gstack-gbrain-sync.ts:442` konumundaki orkestratör 35 dakikalık bir zaman aşımı uygular. Tetiklendiğinde, `spawnSync` `result.status === null` döndürür, alt süreç SIGTERM alır ve bellek içi ingest durumu asla `~/.gstack/.transcript-ingest-state.json` dosyasına floşlanmaz. Sonraki çalıştırma aynı ilerlememiş durumdan başlar — her şeyi yeniden yapma kalıbını açıklar.

## Alandaki sayılar

| Metrik | Değer | Kaynak |
|---|---|---|
| walkAllSources'taki dosyalar | 1,841 | `find ~/.gstack/projects -type f \( -name "*.md" -o -name "*.jsonl" \)` |
| `gbrain put` soğuk başlangıç | 329ms | `time (echo "test" \| gbrain put _bench)` |
| `gitleaks detect` soğuk başlangıç | 46ms | `time gitleaks detect --no-git --source <küçük-dosya>` |
| Teorik alt sınır (sadece alt süreç) | 690s / 11.5 dk | 375ms × 1841 |
| Gözlemlenen çalıştırma süresi | 2100s / 35 dk | orkestratör zaman aşımıyla tam olarak eşleşir |
| Gerçekten kalıcı olan sayfalar | 501 | gbrain sources list page_count |
| Çalıştırma sırasında PGLite büyümesi | 290 → 386 MB | `du -sh ~/.gbrain/brain.pglite` |

## Önerilen mimari

Dosya başı alt süreç döngüsünü **hazırla-sonra-toplu** boru hattı ile değiştir:

```
walkAllSources(ctx)
  → prepareStage (süreç içi, hızlı):
       transkriptleri/yapıtları ayrıştır
       özel YAML ön bilgisi ile PageRecord oluştur
       gitleaks taraması (hazırlık dizininde tek alt süreç)
       hazırlanan .md dosyalarını hazırlık dizinine yaz
  → gbrain import <hazırlık-dizini> --no-embed (tek alt süreç)
  → tüm başarılarla durum dosyasını floşla
  → hazırlık dizinini temizle
```

### Neden `gbrain import <dir>` doğru toplu iş yoludur

- zaten gbrain CLI'da gönderildi (doğrulandı: `gbrain --help` `import <dir> [--no-embed]` gösterir).
- Dizini gbrain'in kendi çalışma zamanı içinde süreç içi taramak — alt süreç yayılma yok.
- gbrain'in toplu iş boyutu ve gömme-toplu iş ayarını onurlandırır.
- gbrain v0.31.2 import, gözlemlenen çalıştırmada 501 sayfa + 2906 parçayı 10 saniyede yaptı; yavaş kısım onun üzerindeki dosya başına `gbrain put` döngümüz oldu.

### Mevcut kodun doğru yaptığı şeyi saklıyoruz

- **Özel YAML ön bilgi enjeksiyonu** (başlık, tür, etiketler) — hazırlanan .md dosyalarını ön bilgi ile hazırlık dizinine yazarak korundu.
- **Gizli tarama** — korundu, ama hazırlıktan sonra, import öncesi TEK `gitleaks detect --source <hazırlık-dizini>` çağrısına taşındı. Bulguları olan dosyalar sansürlenir veya dışlanır; hazırlık dizini gitleaks'e sadece hazırlanmış içeriği gösterir, iç gbrain durumunu değil.
- **Kısmi transkript algılama** — hazırlık aşamasında korundu; kısmi dosyalar ön bilgisinde hâlâ bir `partial: true` alanı alır.
- **Atfedilmemiş transkript filtreleme** — hazırlık aşamasında korundu.
- **Dosya başına mtime + sha256 durum takibi** — korundu; hazırlık aşaması neyin hazırlıklandığını kaydeder, import başarısı neyin indiğini kaydeder.
- **Artımlı mod** — `fileChangedSinceState` kontrolü hazırlık döngüsünün üstünde kalır.

## Geçiş adımları

### Adım 1: mevcut ingest döngüsünden `preparePages` çıkar

`ingestPass`'taki ( `bin/gstack-memory-ingest.ts` satırları 899-988) tarama ile `gbrainPutPage` çağrısı arasındaki her şeyi alın. `preparePages(args, ctx, state) → { staged: PreparedPage[], skipped, failed }` yeni işlevine taşıyın.

Çıktı: `{ slug, body, source_path, mtime_ns, sha256, partial }` listesi, `body` ön bilgi dahil tam markdown.

### Adım 2: hazırlık dizini yazıcısı ekle

Saf işlev: `writeStaged(prepared, stagingDir) → { written, errors }`.
Dosya adı: `${slug}.md`. Etkisiz üzerine yazma.

Hazırlık dizini yaşam döngüsü:
- `~/.gstack/.staging-ingest-${pid}-${ts}/` konumunda oluşturulur
- `finally` bloğunda temizlenir, SIGTERM'de bile
- Her ingest geçişi için bir hazırlık dizini — çalışmalar arasında asla yeniden kullanılmaz

### Adım 3: tek gitleaks geçişi

Dosya başı `secretScanFile(path)` çağrılarını hazırlıktan sonra tek çağrı ile değiştir:
`gitleaks detect --no-git --source <hazırlık-dizini> --report-format json --report-path -`.

JSON çıktısını ayrıştır, `Map<slug, findings[]>` oluştur. Bulguları olan dosyalar import öncesi hazırlık dizininden kaldırılır (veya mevcut sansürleme politikasına göre yerinde sansürlenir `lib/gstack-memory-helpers.ts`).

### Adım 4: `gbrainPutPage` döngüsünü tek import çağrısıyla değiştir

```typescript
const importResult = spawnSync("gbrain", ["import", stagingDir], {
  stdio: ["ignore", "inherit", "inherit"],
  timeout: 30 * 60 * 1000, // bol; tüm toplu iş
});
```

`Import complete` satırı ve `failed` sayısı için stdout'u ayrıştır.

### Adım 5: kısmi başarı durumunda durum sakla

gbrain import `imported=N, failed=M` raporluyorsa, N başarılı slug'lar için durum sakla (hepsi için değil). Başarısız olanlar bir sonraki çalıştırmada yeniden denenir, ama başarılı olanlar yeniden yapılmaz.

### Adım 6: `gstack-memory-ingest.ts` içinde SIGTERM işleyicisi

`main()`'i şu şekilde sar:
```typescript
let interrupted = false;
const flush = () => {
  if (interrupted) return;
  interrupted = true;
  saveState(state); // biriken her neyse en iyi çabayla floşla
  cleanupStagingDir();
  process.exit(143);
};
process.on("SIGTERM", flush);
process.on("SIGINT", flush);
```

Bu, öldür-kaydetme hatasını bağımsız olarak engeller — toplu iş import orkestratör zaman aşımını aşsa bile, hazırlık aşamasından durum hayatta kalır.

### Adım 7: orkestratör güncellemesi

`bin/gstack-gbrain-sync.ts:444` konumunda:
- `result.status === 0`'ı `result.status === 0 || (parsedSummary.imported > 0 && parsedSummary.imported >= parsedSummary.skipped + parsedSummary.failed)` olarak değiştir.
  Kısmi başarıyı (çoğu sayfa içe aktarıldı) ERR değil, OK olarak ele al.
  - `failed_count` ve `partial_blockers`'ı aşama özetinde yüzeye çıkar, böylece kullanıcı `ERR exited null` yerine `Memory ... OK 487/501 imported (14 FILE_TOO_LARGE)` görür.

### Adım 8: FILE_TOO_LARGE'ı özel olarak ele al

gbrain FILE_TOO_LARGE raporladığında, bir sonraki hazırlık aşamasının o dosyayı tamamen atlaması için yeni bir `~/.gstack/.ingest-skip-list.json` dosyasına kaydet. Her zaman başarısız olacak bir dosyayı yeniden hazırlamaktan kaçınır. Kullanıcı yeni bir `gstack-memory-ingest --skip-list` bayrağı ile atlama listesini inceleyebilir.

## Test planı

1. **Birim (ücretsiz, `bun test` içinde çalışır):**
   - 50 dosyalık fixture corpusuna karşı `preparePages`: YAML doğru, kısmi algılama çalışır, atfedilmemiş filtrelenmiş iddiası.
   - `writeStaged` üzerine yazma etkisizliği.
   - Alt süreç test donanımı kullanan SIGTERM işleyicisi floş davranışı.

2. **Entegrasyon (ücretsiz, `bun test` içinde çalışır):**
   - Uçtan uca: hazırla → gitleaks → geçici PGLite üzerinde gbrain import, sayfa sayısının içe aktarılan sayıyla eşleştiği iddiası.
   - Kısmi başarı yolu: bilerek FILE_TOO_LARGE enjekte et; başarıların durumlandığını, hatanın atlama listesine kaydedildiğini iddiası.
   - SIGTEMİ arasında durum korunumu: ingest başlat, ortada öldür, yeniden başlat, devam edilen durum iddiası.

3. **Kıyaslama geçidi (periyodik, ücretli):**
   - 1841 dosyalık fixture üzerinde soğuk çalıştırma: 8 dk altında iddiası.
   - Artımlı çalıştırma (değişiklik yok): 60 saniye altında iddiası.
   - Test fixture'ı: tekrarlanabilir zamanlama için `~/.gstack/projects/` anlık görüntüsünün kopyası.

## Geri alma stratejisi

- `gstack-memory-ingest` üzerinde yeni `--legacy-ingest` bayrağı, eski dosya başı yolunu bir sürüm döngüsü için çağrılabilir tutar.
- Toplu iş yolu gerçek bir corpus'ta geriye giderse, yeniden dağıtım olmadan geri almak için `gstack-config set memory_ingest_path legacy` ayarlayın.
- Toplu işin kararlı olduğunu onayladıktan bir küçük sürüm sonra bayrak + eski yolu kaldırın.

## Riskler ve plan-eng-review için açık sorular

1. **Örtüşen slug'larda gbrain import idempotensi.** Önceki bir çalıştırma slug X'i eski içerikle PGLite'e yazdıysa, güncellenmiş X'in `gbrain import`'u üzerine yazar mı yoksa kopyalar mı? Buna güvenmeden önce test etmek gerekir.

2. **`gbrain import` ayrıştırıcısı içinde ön bilgi enjeksiyonu.** Mevcut kod varolan ön bilgi bloklarına başlık/tür/etiketleri nasıl enjekte edeceğini biliyor (satır 794-821). `gbrain import` bu alanları `gbrain put` ile aynı şekilde onurlandırıyor mu? Birim testinde doğrulayın.

3. **Hazırlık dizini disk baskısı.** 1841 dosya × ortalama ~50KB = ~92MB hazırlık .md içeriği. Geliştirici makinelerinde kabul edilebilir ama bilmeye değer.
   Alternatif: hazırlanmış içeriği gbrain'e bir tar boru hattı ile akıtmak (gbrain destekliyorsa) — muhtemelen hayır, V1 için göz ardı et.

4. **Çapraz-worktree eşzamanlılık.** `~/.gstack/.staging-ingest-${pid}-${ts}/` pid ad alanlıdır, bu nedenle iki eşzamanlı /sync-gbrain çalışması çarpışmaz.
   Ama orkestratör zaten `~/.gstack/.sync-gbrain.lock` konumunda bir kilit tutuyor, bu nedenle bu kemer+askı. Saklayın.

5. **"Bellek ingest null çıktı" mesajı.** Bu değişiklikten sonra, orkestratör gerçek OOM öldürmelerinde veya SIGKILL'de hâlâ status=null görebilir.
   Karar bloğu daha dürüst olmalı mı? Örn.,
   `ERR memory: killed by signal SIGTERM at 35:00 (timeout)`.

6. **Bellek için `gbrain put`'u kullanımdan kaldırmalı mıyız?** Eski yol V1.5'in `put_file` geçiş planı için var. Toplu iş import çalışırken, tek sayfalı put'a hâlâ orkestratör dışında tetiklenen `~/.gstack/.transcript-ingest-state.json` güncellemeleri için bir geri dönüş olarak ihtiyacımız var mı? Muhtemelen evet, ama doğrulanmaya değer.

## Bu ne değil

- Bir gbrain CLI değişikliği değil. Tüm çalışma gstack'de.
- Bir CLAUDE.md ses/UX değişikliği değil.
- Yeni bir kullanıcıya yönelik özellik değil. CHANGELOG girdisi şöyle okuyacak: "Bellek ingest soğuk çalıştırmalarda ~10 kat daha hızlı ve kesintiye dayanıklı."

## Kabul kriterleri

- 1841 dosya üzerinde soğuk `/sync-gbrain` 8 dakikanın altında tamamlanır.
- Artımlı `/sync-gbrain` (dosya değişikliği yok) 60 saniyenin altında tamamlanır.
- Ortadaki SIGTERM durumu floşlar; bir sonraki çalıştırma başarılı şekilde içe aktarılan dosyaları yeniden yapmaz.
- FILE_TOO_LARGE başarısızlıkları sync.last_commit ilerlemesini engellemez.
- Tüm mevcut test fixture'ları (transkriptler, öğrenmeler, tasarım belgeleri, ceo planları) tam ön bilgi ile doğru şekilde içe aktarılır.
- Kısmi transkript veya atfedilmemiş transkrict işlemede gerileme yok.