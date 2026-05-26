# İniş Öncesi İnceleme Kontrol Listesi

## Talimatlar

Aşağıda listelenen sorunlar için `git diff origin/main` çıktısını inceleyin. Belirli olun — `file:line` belirtin ve düzeltmeler önerin. İyi olan her şeyi atlayın. Yalnızca gerçek sorunları işaretleyin.

**İki geçitli inceleme:**
- **Geçiş 1 (KRİTİK):** Önce SQL & Veri Güvenliği, Yarış Durumları, LLM Çıktı Güven Sınırı, Shell Enjeksiyonu ve Enum Tamlığı'nı çalıştırın. En yüksek şiddet.
- **Geçiş 2 (BİLGİSEL):** Aşağıdaki kalan kategorileri çalıştırın. Daha düşük şiddet ama yine de işlem gören.
- **Uzman kategorileri (paralel alt ajanlar tarafından işlenir, BU kontrol listesi tarafından DEĞİL):** Test Boşlukları, Ölü Kod, Sihirli Sayılar, Koşullu Yan Etkiler, Performans & Paket Etkisi, Kripto & Entropi. Bunlar için `review/specialists/` dosyasına bakın.

Tüm bulgular Düzeltme-Önce İnceleme ile işlem görür: açık mekanik düzeltmeler otomatik olarak uygulanır,
gerçekten belirsiz sorunlar tek bir kullanıcı sorusunda toplanır.

**Çıktı formatı:**

```
İniş Öncesi İnceleme: N sorun (X kritik, Y bilgilendirici)

**OTOMATİK-DÜZELTİLDİ:**
- [dosya:satır] Sorun → düzeltme uygulandı

**GİRDİ-GEREKLİ:**
- [dosya:satır] Sorun açıklaması
  Önerilen düzeltme: önerilen düzeltme
```

Sorun bulunamazsa: `İniş Öncesi İnceleme: Sorun bulunamadı.`

Kısa olun. Her sorun için: sorunu açıklayan bir satır, düzeltmeyi belirten bir satır. Önsöz yok, özet yok, "genel olarak iyi görünüyor" yok.

---

## İnceleme Kategorileri

### Geçiş 1 — KRİTİK

#### SQL & Veri Güvenliği
- SQL'de dize interpolasyonu (değerler `.to_i`/`.to_f` olsa bile — parametreli sorgular kullanın (Rails: sanitize_sql_array/Arel; Node: prepared statements; Python: parametreli sorgular))
- TOCTOU yarış durumları: atomik `WHERE` + `update_all` olması gereken kontrol-sonra-ayarla desenleri
- Doğrudan DB yazıları için model doğrulamalarını atlama (Rails: update_column; Django: QuerySet.update(); Prisma: raw sorgular)
- N+1 sorguları: Döngüler/görünümlerde kullanılan ilişkiler için eksik istekli yükleme (Rails: .includes(); SQLAlchemy: joinedload(); Prisma: include)

#### Yarış Durumları & Eşzamanlılık
- Benzersizlik kısıtlaması veya yinelenen anahtar hatasını yakalama ve yeniden deneme olmaksızın okuma-kontrol-yazma (örn., `where(hash:).first` ardından eşzamanlı ekleme işleme olmadan `save!`)
- Benzersiz DB dizini olmaksızın bul-veya-oluştur — eşzamanlı çağrılar yinelenenler oluşturabilir
- Atomik `WHERE old_status = ? UPDATE SET new_status` kullanmayan durum geçişleri — eşzamanlı güncellemeler geçişleri atlayabilir veya iki kez uygulayabilir
- Kullanıcı kontrollü verilerde güvensiz HTML oluşturma (Rails: .html_safe/raw(); React: dangerouslySetInnerHTML; Vue: v-html; Django: |safe/mark_safe) (XSS)

#### LLM Çıktı Güven Sınırı
- Format doğrulaması olmadan DB'ye yazılan veya posta göndericilerine geçirilen LLM tarafından oluşturulan değerler (e-postalar, URL'ler, isimler). Kalıcı kılmadan önce hafif koruyucular ekleyin (`EMAIL_REGEXP`, `URI.parse`, `.strip`).
- Veritabanı yazımlarından önce tür/şekil kontrolü olmaksızın kabul edilen yapılandırılmış aruç çıktısı (diziler, karmalar).
- İzin listesi olmaksızın getirilen LLM tarafından oluşturulan URL'ler — URL dahili ağa işaret ediyorsa SSRF riski (Python: `urllib.parse.urlparse` → `requests.get`/`httpx.get` çağrısından önce ana bilgisayarı kara listeye karşı kontrol edin)
- Sanitizasyon olmaksızın bilgi tabanlarına veya vektör DB'lerine kaydedilen LLM çıktısı — depolanmış prompt enjeksiyonu riski

#### Shell Enjeksiyonu (Python'a özgü)
- `shell=True` VE komut dizesinde f-string/`.format()` interpolasyonu ile `subprocess.run()` / `subprocess.call()` / `subprocess.Popen()` — bunun yerine argüman dizileri kullanın
- Değişken interpolasyonu ile `os.system()` — argüman dizileri kullanarak `subprocess.run()` ile değiştirin
- Kum havuzu olmaksızın LLM tarafından oluşturulan kodda `eval()` / `exec()`

#### Enum & Değer Tamlığı
- Diff yeni bir enum değeri, durum dizesi, katman adı veya tür sabiti eklediğinde:
- **Her tüketiciden izleyin.** Bu değere göre dallanan, filtreleyen veya görüntüleyen her dosyayı okuyun (sadece grep yapmayın — OKUYUN). Herhangi bir tüketici yeni değeri işlemiyorsa, işaretleyin. Yaygın kaçırma: ön uç açılır menüsüne değer eklemek ancak arka uç modeli/hesaplama yöntemi onu kalıcı kılmıyorsa.
- **İzin listelerini/filtre dizilerini kontrol edin.** Kardeş değerleri içeren diziler veya `%w[]` listeleri arayın (örn., katmanlara "revize" ekliyorsanız, her `%w[quick lfg mega]` bulun ve "revize"in gerektiği yerde dahil olduğunu doğrulayın).
- **`case`/`if-elsif` zincirlerini kontrol edin.** Mevcut kod enum üzerinde dallanıyorsa, yeni değer yanlış bir varsayılanın içine mi düşüyor?
Bunu yapmak için: Kardeş değerlere tüm referansları bulmak için Grep kullanın (örn., tüm katman tüketicilerini bulmak için "lfg" veya "mega" grep'leyin). Her eşleşmeyi okuyun. Bu adım diff DIŞINDAKİ kodu okumayı gerektirir.

### Geçiş 2 — BİLGİSEL

#### Async/Sync Karışımı (Python'a özgü)
- `async def` uç noktaları içinde senkron `subprocess.run()`, `open()`, `requests.get()` — olay döngüsünü engeller. Bunun yerine `asyncio.to_thread()`, `aiofiles` veya `httpx.AsyncClient` kullanın.
- `async` fonksiyonlar içinde `time.sleep()` — `asyncio.sleep()` kullanın
- `run_in_executor()` sarmalaması olmaksızın async bağlamda senkron DB çağrıları

#### Sütun/Alan Adı Güvenliği
- ORM sorgularındaki sütun adlarını gerçek DB şemasına karşı doğrulayın (`.select()`, `.eq()`, `.gte()`, `.order()`) — yanlış sütun adları sessizce boş sonuçlar döndürür veya yutulan hatalar atar
- Sorgu sonuçlarındaki `.get()` çağrılarının gerçekte seçilen sütun adını kullandığını kontrol edin
- Mevcut olduğunda şema belgeleriyle çapraz referans yapın

#### Ölü Kod & Tutarlılık (yalnızca sürüm/değişiklik günlüğü — diğer öğeler bakım uzmanı tarafından işlenir)
- PR başlığı ile VERSION/CHANGELOG dosyaları arasındaki sürüm uyuşmazlığı
- Değişiklikleri yanlış açıklayan CHANGELOG girdileri (örn., "X'den Y'ye değiştirildi" ancak X hiç mevcut değilken)

#### LLM Prompt Sorunları
- Prompt'larda 0-indeksli listeler (LLM'ler güvenilir şekilde 1-indeksli döndürür)
- Aslında `tool_classes`/`tools` dizisinde bağlananlarla eşleşmeyen prompt metninde listelenen mevcut araçlar/yetenekler
- Birden fazla yerde belirtilen ve sürüklenebilen kelime/jeton sınırları

#### Tamlık Boşlukları
- Tam sürümün <30 dk CC süresi mal olacağı kısayol uygulamaları (örn., kısmi enum işleme, eksik hata yolları, açık sınır durumları olan düz kenar testleri)
- Yalnızca insan-takım çaba tahminleriyle sunulan seçenekler — hem insan hem de CC+gstack süresini göstermeli
- Eksik test kapsamı boşlukları (mutlu yol yapısını yansıtan eksik negatif yol testleri, eksik sınır durumu testleri ekleme)
- %80-90'ta uygulanan özellikler, %100'ün mütevazı ek kodla ulaşılabilir olduğu yerlerde

#### Zaman Penceresi Güvenliği
- "Bugün"ün 24 saat kapsadığını varsayan tarih-anahtarlı aramalar — saat 8'da PT'de rapor yalnızca bugünün anahtarı altında gece yarısı→8'ı görür
- İlgili özellikler arasında uyuşmayan zaman pencereleri — biri saatlik kovalar, diğeri aynı veri için günlük anahtarlar kullanıyor

#### Sınırda Tür Dönüştürme
- Türün değişebileceği Ruby→JSON→JS sınırlarını geçen değerler (sayısal vs dize) — karma/özet girdileri türleri normalleştirmelidir
- Serileştirmeden önce `.to_s` veya eşdeğerini çağırmayan karma/özet girdileri — `{ cores: 8 }` vs `{ cores: "8" }` farklı karmalar üretir

#### Görünüm/Ön Uç
- Kısmi görünümlerde satır içi `<style>` blokları (her oluşturmada yeniden ayrıştırılır)
- Görünümlerde O(n*m) aramalar (`index_by` karması yerine bir döngüde `Array#find`)
- Bir `WHERE` yancesi olabilecek DB sonuçlarında Ruby tarafı `.select{}` filtrelemesi (önde gelen joker karakter `LIKE`'ten kasıtlı kaçınma dışında)

#### Dağıtım & CI/CD Hattı
- CI/CD iş akışı değişiklikleri (`.github/workflows/`): derleme aracı sürümlerinin proje gereksinimleriyle eşleştiğini, yapı ismi/yollarının doğru olduğunu, gizli anahtarların `${{ secrets.X }}` kullandığını sabit kodlanmış değerler yerine doğrulayın
- Yeni yapı türleri (CLI binary, kütüphane, paket): bir yayımlama/sürüm iş akışının var olduğunu ve doğru platformları hedeflediğini doğrulayın
- Çapraz platform derlemeleri: CI matrisinin tüm hedef OS/mimari kombinasyonlarını kapsadığını veya hangilerinin test edilmediğini belgelediğini doğrulayın
- Sürüm etiketi format tutarlılığı: `v1.2.3` vs `1.2.3` — VERSION dosyası, git etiketleri ve yayımlama betikleri arasında eşleşmeli
- Yayımlama adımı eşkuvvetliliği: yayımlama iş akışını yeniden çalıştırmak başarısız olmamalıdır (örn., `gh release delete` önce `gh release create`)

**İŞARETLEMEYİN:**
- Mevcut otomatik dağıtım hatlarına sahip web hizmetleri (Docker derlemesi + K8s dağıtımı)
- Takım dışına dağıtılmayan dahili araçlar
- Yalnızca test CI değişiklikleri (yayımlama adımları değil, test adımları ekleme)

---

## Şiddet Sınıflandırması

```
KRİTİK (en yüksek şiddet):      BİLGİSEL (ana ajan):      UZMAN (paralel alt ajanlar):
├─ SQL & Veri Güvenliği          ├─ Async/Sync Karışımı             ├─ Test uzmanı
├─ Yarış Durumları & Eşzamanlılık  ├─ Sütun/Alan Adı Güvenliği      ├─ Bakım uzmanı
├─ LLM Çıktı Güven Sınırı      ├─ Ölü Kod (yalnızca sürüm)      ├─ Güvenlik uzmanı
├─ Shell Enjeksiyonu                ├─ LLM Prompt Sorunları             ├─ Performans uzmanı
└─ Enum & Değer Tamlığı      ├─ Tamlık Boşlukları             ├─ Veri Göç uzmanı
                                   ├─ Zaman Penceresi Güvenliği            ├─ API Sözleşmesi uzmanı
                                   ├─ Sınırda Tür Dönüştürme   └─ Kırmızı Takım (koşullu)
                                   ├─ Görünüm/Ön Uç
                                   └─ Dağıtım & CI/CD Hattı

Tüm bulgular Düzeltme-Önce İnceleme ile işlem görür. Şiddet, sunum sırasını
ve OTOMATİK-DÜZELT vs SOR arasında sınıflandırmayı belirler — kritik bulgular
SOR'a meyillidir (daha risklidirler), bilgilendirici bulgular OTOMATİK-DÜZELT'e
meyillidir (daha mekaniktir).
```

---

## Düzeltme-Önce Heuristiği

Bu heuristik hem `/review` hem de `/ship` tarafından referans alınır. Ajan'ın
bir bulguyu otomatik düzeltip düzeltmediğini veya kullanıcıya sorup sormadığını belirler.

```
OTOMATİK-DÜZELT (ajan sormadan düzeltir):     SOR (insan yargısı gerekir):
├─ Ölü kod / kullanılmayan değişkenler            ├─ Güvenlik (yetkilendirme, XSS, enjeksiyon)
├─ N+1 sorguları (eksik istekli yükleme)      ├─ Yarış durumları
├─ Kodu çelişen eski yorumlar       ├─ Tasarım kararları
├─ Sihirli sayılar → adlandırılmış sabitler         ├─ Büyük düzeltmeler (>20 satır)
├─ Eksik LLM çıktı doğrulama           ├─ Enum tamlığı
├─ Sürüm/yol uyuşmazlıkları                 ├─ İşlevselliği kaldırma
└─ Satır içi stiller, O(n*m) görünüm aramaları        └─ Kullanıcı tarafından görülebilir davranışı
      değiştiren herhangi bir şey
```

**Kural:** Düzeltme mekanikse ve kıdemli bir mühendis tartışmasız uygulayacaksa, OTOMATİK-DÜZELT'tir. Makul mühendisler düzeltme konusunda anlaşamıyorsa, SOR'dur.

**Kritik bulgular varsayılan olarak SOR'a meyillidir** (doğası gereği daha risklidirler).
**Bilgilendirici bulgular varsayılan olarak OTOMATİK-DÜZELT'e meyillidir** (daha mekaniktirler).

---

## Baskılar — Bunları işaretlemeyin

- "X, Y ile gereksiz" olduğunda gereksizlik zararsızsa ve okunabilirliğe yardımcı oluyorsa (örn., `length > 20` ile gereksiz `present?`)
- "Bu eşik/sabitin neden seçildiğini açıklayan bir yorum ekleyin" — eşikler ayarlama sırasında değişir, yorumlar eskir
- "Bu iddia daha sıkı olabilir" olduğunda iddia zaten davranışı kapsıyorsa
- Yalnızca tutarlılık için değişiklik önermek (bir değeri, başka bir sabitin korunma şekliyle eşleşmesi için bir koşula sarmak)
- "Regex X sınır durumunu ele almıyor" olduğunda girdi kısıtlıysa ve X pratikte asla oluşmuyorsa
- "Test birden fazla koruyucuyu aynı anda sınıyor" — sorun yok, testlerin her koruyucuyu izole etmesi gerekmez
- Eşik değişikliklerini değerlendirme (max_actionable, minimum puanlar) — bunlar ampirik olarak ayarlanır ve sürekli değişir
- Zararsız no-op'lar (örn., dizide hiçbir zaman bulunmayan bir eleman üzerinde `.reject`)
- İncelediğiniz dif'te zaten ele alınmış HERHANGİ BİR ŞEY — yorum yapmadan ÖNCE TAM dif'i okuyun