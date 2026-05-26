# Tasarım: Tasarım Saçması — Tarayıcıdan-Ajana Geri Besleme Döngüsü

2026-03-27 tarihinde oluşturuldu
Dal: garrytan/agent-design-tools
Durum: CANLI BELGE — hatalar bulundukça ve düzeltildikçe güncelleyin

## Bu Özellik Ne Yapıyor

Tasarım Saçması birden fazla AI tasarım mockup'u oluşturur, kullanıcının gerçek
tarayıcısında bir karşılaştırma panosu olarak yan yana açar ve yapılandırılmış geri
besleme toplar (bir favori seçme, alternatifleri derecelendirme, notlar bırakma,
yeniden oluşturma isteme). Geri besleme kodlama ajanına geri akar ve ajan buna göre
hareket eder: ya onaylanan varyantla devam eder ya da yeni varyantlar oluşturur ve
panoyu yeniden yükler.

Kullanıcı hiçbir zaman tarayıcı sekmesinden ayrılmaz. Ajan hiçbir zaman gereksiz
sorular sormaz. Pano geri besleme mekanizmasıdır.

## Temel Sorun: Birbirleriyle Konuşması Gereken İki Dünya

```
  ┌─────────────────────┐          ┌──────────────────────┐
  │   KULLANICININ TARAYICISI    │          │   KODLAMA AJANI       │
  │   (gerçek Chrome)     │          │   (Claude Code /     │
  │                     │          │    Conductor)         │
  │  Karşılaştırma panosu   │          │                      │
  │  butonlar ile:      │   ???    │  Bilmesi gerekenler:      │
  │  - Gönder           │ ──────── │  - Hangisi seçildi   │
  │  - Yeniden oluştur   │          │  - Yıldız derecelendirmeleri      │
  │  - Buna benzer daha   │          │  - Yorumlar          │
  │  - Remiks            │          │  - Yeniden oluşturma istendi mi?  │
  └─────────────────────┘          └──────────────────────┘
```

"Soru işaretleri" zor kısımdır. Kullanıcı Chrome'da bir düğmeye tıklıyor. Terminal'de
çalışan ajanın bunu bilmesi gerekiyor. Bunlar, ortak belleği, ortak etkinlik veri
yolunu, WebSocket bağlantısı olmayan iki tamamen ayrı süreç.

## Mimari: Bağlantı Nasıl Çalışıyor

```
  KULLANICININ TARAYICISI                    $D serve (Bun HTTP)              AJAN
  ════════════════                   ═══════════════════              ═════
       │                                   │                           │
       │  GET /                            │                           │
       │ ◄─────── pano HTML'si sunar ──────►│                           │
       │    (__GSTACK_SERVER_URL ile       │                           │
       │     <head>'a enjekte edilmiş)         │                           │
       │                                   │                           │
       │  [kullanıcı derecelendirir, seçer, yorumlar]    │                           │
       │                                   │                           │
       │  POST /api/feedback               │                           │
       │ ─────── {preferred:"A",...} ─────►│                           │
       │                                   │                           │
       │  ◄── {received:true} ────────────│                           │
       │                                   │── feedback.json yazar ──►│
       │  [girdiler devre dışı,                │   (veya yeniden oluşturma için
       │   "Ajana dön" gösterilir)        │    feedback-pending
       │                                   │    .json)
       │                                   │                           │
       │                                   │                  [ajan her 5sn'de
       │                                   │                   yoklar,
       │                                   │                   dosyayı okur]
```

### Üç Dosya

| Dosya | Ne zaman yazılır | Anlamı | Ajan eylemi |
|------|-------------|-------|-------------|
| `feedback.json` | Kullanıcı Gönder'e tıklar | Nihai seçim, bitti | Oku, devam et |
| `feedback-pending.json` | Kullanıcı Yeniden Oluştur/Buna Benzer Daha Fazla tıklar | Yeni seçenekler istiyor | Oku, sil, yeni varyantlar oluştur, panoyu yeniden yükle |
| `feedback.json` (2. ve sonraki turlar) | Yeniden oluşturmadan sonra kullanıcı Gönder'e tıklar | Yinelemeden sonra nihai seçim | Oku, devam et |

### Durum Makinesi

```
  $D serve başlar
       │
       ▼
  ┌──────────┐
  │ SUNUYOR  │◄──────────────────────────────────────┐
  │          │                                        │
  │ Pano      │  POST /api/feedback                    │
  │ canlı,    │  {regenerated: true}                   │
  │ bekliyor  │──────────────────►┌──────────────┐     │
  │          │                   │ YENİDEN OLUŞTURUYOR │     │
  │          │                   │              │     │
  └────┬─────┘                   │ Ajan 10 dk     │     │
       │                         │ yeni          │     │
       │  POST /api/feedback     │ pano HTML'si    │     │
       │  {regenerated: false}   │ GÖNDERİR       │     │
       │                         └──────┬───────┘     │
       ▼                                │             │
  ┌──────────┐                POST /api/reload        │
  │  BİTTİ    │                {html: "/new/board"}    │
  │          │                          │             │
  │ çıkış 0   │                          ▼             │
  └──────────┘                   ┌──────────────┐     │
                                 │  YENİDEN YÜKLÜYOR   │─────┘
                                 │              │
                                 │ Pano otomatik  │
                                 │ yenilenir    │
                                 │ (aynı sekme)   │
                                 └──────────────┘
```

### Port Keşfi

Ajan `$D serve`'i arka planda çalıştırır ve port için stderr'yi okur:

```
SERVE_STARTED: port=54321 html=/path/to/board.html
SERVE_BROWSER_OPENED: url=http://127.0.0.1:54321
```

Ajan stderr'den `port=XXXXX`'i ayrıştırır. Bu port daha sonra kullanıcı yeniden
oluşturma istediğinde `/api/reload`'u POST etmek için gerekir. Ajan port numarasını
kaybederse, panoyu yeniden yükleyemez.

### Neden 127.0.0.1, localhost Değil

`localhost`, bazı sistemlerde IPv6 `::1`'e çözümlenebilirken Bun.serve() yalnızca
IPv4 üzerinde dinler. Daha da önemlisi, `localhost` geliştiricinin çalıştığı her
etkin alan için tüm geliştirme çerezlerini gönderir. Birçok etkin oturuma sahip
bir makinede, bu Bun'un varsayılan başlık boyutu sınırını aşar (HTTP 431 hatası).
`127.0.0.1` her iki sorunu da önler.

## Her Sınır Durumu ve Tuzak

### 1. Zombi Form Sorunu

**Ne olan:** Kullanıcı geri besleme gönderir, POST başarılı olur, sunucu çıkar. Ama HTML
sayfası hala Chrome'da açık. Etkileşimli görünüyor. Kullanıcı geri bildirimini düzenleyip
Gönder'e tekrar tıklayabilir. Sunucu gittiği için hiçbir şey olmaz.

**Düzeltme:** Başarılı POST'tan sonra, pano JS'si:
- TÜM girdileri devre dışı bırakır (düğmeler, radyolar, metin alanları, yıldız derecelendirmeleri)
- Yeniden Oluştur çubuğunu tamamen gizler
- Gönder düğmesini şununla değiştirir: "Geri bildirim alındı! Kodlama ajanınıza dönün."
- Şunu gösterir: "Daha fazla değişiklik yapmak mı istiyorsunuz? Tekrar `/design-shotgun` çalıştırın."
- Sayfa gönderilenlerin salt-okunur kaydı olur

**Uygulayan:** `compare.ts:showPostSubmitState()` (satır 484)

### 2. Ölü Sunucu Sorunu

**Ne olan:** Sunucu zaman aşımına uğrar (varsayılan 10 dk) veya kullanıcı hala panoyu
açken çöker. Kullanıcı Gönder'e tıklar. fetch() sessizce başarısız olur.

**Düzeltme:** `postFeedback()` işlevinin bir `.catch()` işleyicisi vardır. Ağ başarısızlığında:
- Kırmızı hata banner'ı gösterir: "Bağlantı kaybedildi"
- Toplanan geri besleme JSON'unu kopyalanabilir bir `<pre>` bloğunda gösterir
- Kullanıcı doğrudan kodlama ajanına kopyala-yapıştırabilir

**Uygulayan:** `compare.ts:showPostFailure()` (satır 546)

### 3. Eski Yeniden Oluşturma Döndürücüsü

**Ne olan:** Kullanıcı Yeniden Oluştur'a tıklar. Pano döndürücüyü gösterir ve 2 saniyede
bir `/api/progress`'i yoklar. Ajan çöker veya yeni varyantlar oluşturmak çok uzun sürer.
Döndürücü sonsuza kadar döner.

**Düzeltme:** İlerleme yoklamasının 5 dakikalık sert zaman aşımı vardır (150 yoklama x 2sn aralık).
5 dakika sonra:
- Döndürücü şununla değiştirilir: "Bir şeyler yanlış gitti."
- Şunu gösterir: "Kodlama ajanınızda tekrar `/design-shotgun` çalıştırın."
- Yoklama durur. Sayfa bilgilendirici olur.

**Uygulayan:** `compare.ts:startProgressPolling()` (satır 511)

### 4. file:// URL Sorunu (ORİJİNAL HATA)

**Ne olan:** Beceri şablonu başlangıçta `$B goto file:///path/to/board.html` kullanıyordu.
Ama `browse/src/url-validation.ts:71` güvenlik için `file://` URL'lerini engeller.
Geri dönüş `open file://...` kullanıcının macOS tarayıcısını açar, ancak `$B eval`
Playwright'ın başsız tarayıcısını yoklar (farklı süreç, hiçbir zaman sayfayı yüklememiş).
Ajan sonsuza kadar boş DOM'u yoklar.

**Düzeltme:** `$D serve` HTTP üzerinden sunar. Pano için asla `file://` kullanmayın.
`$D compare`'deki `--serve` bayrağı, pano oluşturma ve HTTP sunumunu tek komutta birleştirir.

**Kanıt:** `.context/attachments/image-v2.png` dosyasına bakın — gerçek bir kullanıcı bu
hatayla karşılaştı Ajan doğru bir şekilde teşhis etti: (1) `$B goto` `file://` URL'lerini
reddeder, (2) tarama daemon'u ile yoklama döngüsü yok.

### 5. Çift Tıklama Yarış Durumu

**Ne olan:** Kullanıcı Gönder'e iki kez hızlıca tıklar. İki POST isteği sunucuya ulaşır.
İlkincisi durumu "bitti" olarak ayarlar ve 100ms içinde exit(0) zamanlar. İkincisi o
100ms penceresi içinde ulaşır.

**Mevcut durum:** Tamamen korunmuyor. `handleFeedback()` işlevi işlemeye başlamadan önce
durumun zaten "bitti" olup olmadığını kontrol etmez. İkinci POST başarılı olur ve ikinci
bir `feedback.json` yazar (zararsız, aynı veri). Çıkış yine de 100ms sonra tetiklenir.

**Risk:** Düşük. Pano ilk başarılı POST yanıtında tüm girdileri devre dışı bırakır,
yani ikinci bir tıklamanın ~1ms içinde ulaşması gerekir. Ve iki yazma da aynı geri
besleme verilerini içerir.

**Olası düzeltme:** `handleFeedback()`'in başına `if (state === 'done') return Response.json({error: 'already submitted'}, {status: 409})` ekleyin.

### 6. Port Koordinasyonu Sorunu

**Ne olan:** Ajan `$D serve`'i arka planda çalıştırır ve stderr'den `port=54321`'i ayrıştırır.
Ajanın bu porta daha sonra yeniden oluşturma sırasında `/api/reload`'u POST etmesi gerekir.
Ajan bağlamı kaybederse (konuşma sıkıştırır, bağlam penceresi dolar), portu hatırlamayabilir.

**Mevcut durum:** Port stderr'ye bir kez yazdırılır. Ajanın onu hatırlaması gerekir.
Diske yazılmış bir port dosyası yoktur.

**Olası düzeltme:** Başlangıçta pano HTML'sinin yanına bir `serve.pid` veya `serve.port`
dosyası yazın. Ajan her zaman okuyabilir:
```bash
cat "$_DESIGN_DIR/serve.port"  # → 54321
```

### 7. Geri Besleme Dosyası Temizleme Sorunu

**Ne olan:** Yeniden oluşturma turundan `feedback-pending.json` diskte kalır. Ajan
okumadan önce çökerse, bir sonraki `$D serve` oturumu eski bir dosya bulur.

**Mevcut durum:** Resolver şablonundaki yoklama döngüsü, `feedback-pending.json`'u
okuduktan sonra silmesini söyler. Ama bu, ajanın talimatlara mükemmel şekilde uymasına
bağlıdır. Eski dosyalar yeni bir oturumu karıştırabilir.

**Olası düzeltme:** `$D serve` başlangıçta eski geri besleme dosyalarını kontrol edip
silebilir. Veya: dosyaları zaman damgalarıyla adlandırın (`feedback-pending-1711555200.json`).

### 8. Sıralı Oluşturma Kuralı

**Ne olan:** Temel OpenAI GPT Image API eşzamanlı görüntü oluşturma isteklerini
hız sınırlandırır. 3 `$D generate` çağrısı paralel çalıştığında, 1'i başarılı olur
ve 2'si iptal edilir.

**Düzeltme:** Beceri şablonu açıkça şunu söylemelidir: "Mockup'ları TEKER TEKER oluşturun.
`$D generate` çağrılarını paralel hale getirmeyin." Bu bir istem-düzeyi talimatıdır,
kod-düzeyi bir kilit değil. Tasarım ikili dosyası sıralı yürütmeyi zorlamaz.

**Risk:** Ajanlar bağımsız işleri paralel hale getirmek için eğitilmiştir. Açık bir
talimat olmadan, 3 oluşturmayı eşzamanlı çalıştırmayı deneyeceklerdir. Bu API çağrılarını
ve parayı israf eder.

### 9. AskUserQuestion Gereksizliği

**Ne olan:** Kullanıcı panodan geri besleme gönderdikten sonra (tercih edilen varyant,
derecelendirmeler, yorumlar JSON'un içindeyken), ajan onlara tekrar sorar: "Hangi
varyantı tercih edersiniz?" Bu sinir bozucudur. Panonun tüm amacı bundan kaçınmaktır.

**Düzeltme:** Beceri şablonu şunu söylemelidir: "Kullanıcının tercihini sormak için
AskUserQuestion KULLANMAYIN. `feedback.json`'u okuyun, seçimlerini içerir. Sadece
doğru anladığınızı onaylamak için AskUserQuestion kullanın, yeniden sormak için değil."

### 10. CORS Sorunu

**Ne olan:** Pano HTML'si harici kaynaklara (yazı tipleri, CDN'den görüntüler) başvurursa,
tarayıcı `Origin: http://127.0.0.1:PORT` ile istekler gönderir. Çoğu CDN buna izin verir,
ancak bazıları engelleyebilir.

**Mevcut durum:** Sunucu CORS başlıkları ayarlamaz. Pano HTML'si kendi-yeterlidir
(görüntüler base64 kodlu, stiller satır içi), bu yüzden pratikte bir sorun olmamıştır.

**Risk:** Mevcut tasarım için düşük. Pano harici kaynaklar yükleseydi önemli olurdu.

### 11. Büyük Yük Sorunu

**Ne olan:** `/api/feedback`'a POST gövde boyut sınırı yok. Pano bir şekilde çok büyük
bir yük gönderirse, `req.json()` hepsini belleğe ayrıştırır.

**Mevcut durum:** Pratikte geri besleme JSON'u ~500 bayt ile ~2KB arasındadır. Risk
teoriktir, pratik değildir. Pano JS'si sabit şekilli bir JSON nesnesi oluşturur.

### 12. fs.writeFileSync Hatası

**Ne olan:** `serve.ts:138`'deki `feedback.json` yazımı try/catch olmadan
`fs.writeFileSync()` kullanır. Disk doluysa veya dizin salt-okunursa, bu hata fırlatır
ve sunucuyu çökertir. Kullanıcı sonsuza kadar döndürücü görür (sunucu ölü, ama pano
bilmiyor).

**Risk:** Pratikte düşük (pano HTML'si az önce aynı dizine yazıldı, yazılabilir
olduğunu kanıtlar). Ama try/catch ile 500 yanıtı daha temiz olur.

## Tam Akış (Adım Adım)

### Mutlu Yol: Kullanıcı İlk Denemede Seçer

```
1. Ajan çalıştırır: $D compare --images "A.png,B.png,C.png" --output board.html --serve &
2. $D serve rastgele bir portta (örn. 54321) Bun.serve() başlatır
3. $D serve http://127.0.0.1:54321'yi kullanıcının tarayıcısında açar
4. $D serve stderr'ye yazdırır: SERVE_STARTED: port=54321 html=/path/board.html
5. $D serve __GSTACK_SERVER_URL enjekte edilmiş pano HTML'si yazar
6. Kullanıcı yan yana 3 varyant ile karşılaştırma panosu görür
7. Kullanıcı Seçenek B'yi seçer, derecelendirir: A: 3/5, B: 5/5, C: 2/5
8. Kullanıcı genel geri bildirimde "B'nin aralığı daha iyi, onunla devam et" yazar
9. Kullanıcı Gönder'e tıklar
10. Pano JS'si http://127.0.0.1:54321/api/feedback adresine POST yapar
    Gövde: {"preferred":"B","ratings":{"A":3,"B":5,"C":2},"overall":"B'nin aralığı daha iyi","regenerated":false}
11. Sunucu feedback.json'ı diske yazar (board.html'nin yanına)
12. Sunucu geri besleme JSON'unu stdout'a yazdırır
13. Sunucu {received:true, action:"submitted"} yanıt verir
14. Pano tüm girdileri devre dışı bırakır, "Kodlama ajanınıza dönün" gösterir
15. Sunucu 100ms sonra kod 0 ile çıkar
16. Ajanın yoklama döngüsü feedback.json'ı bulur
17. Ajan onu okur, kullanıcıya özetler, devam eder
```

### Yeniden Oluşturma Yolu: Kullanıcı Farklı Seçenekler İstiyor

```
1-6.  Yukarıdakiyle aynı
7.  Kullanıcı "Tamamen farklı" çini tıklar
8.  Kullanıcı Yeniden Oluştur'a tıklar
9.  Pano JS'si /api/feedback adresine POST yapar
    Gövde: {"regenerated":true,"regenerateAction":"different","preferred":"","ratings":{},...}
10. Sunucu feedback-pending.json'ı diske yazar
11. Sunucu durumu → "yeniden oluşturuyor" olarak ayarlar
12. Sunucu {received:true, action:"regenerate"} yanıt verir
13. Pano döndürücüyü gösterir: "Yeni tasarımlar oluşturuluyor..."
14. Pano 2sn'de bir GET /api/progress yoklamaya başlar

    Bu arada, ajanda:
15. Ajanın yoklama döngüsü feedback-pending.json'ı bulur
16. Ajan onu okur, siler
17. Ajan çalıştırır: $D variants --brief "tamamen farklı yön" --count 3
    (TEKER TEKER, paralel değil)
18. Ajan çalıştırır: $D compare --images "new-A.png,new-B.png,new-C.png" --output board-v2.html
19. Ajan POST yapar: curl -X POST http://127.0.0.1:54321/api/reload -d '{"html":"/path/board-v2.html"}'
20. Sunucu htmlContent'i yeni panoya değiştirir
21. Sunucu durumu → "sunuyor" olarak ayarlar (yeniden yüklenmeden)
22. Pano'nun bir sonraki /api/progress yoklaması {"status":"serving"} döndürür
23. Pano otomatik yenilenir: window.location.reload()
24. Kullanıcı 3 taze varyant ile yeni pano görür
25. Kullanıcı birini seçer, Gönder'e tıklar → 10. adımdan mutlu yol
```

### "Buna Benzer Daha Fazla" Yolu

```
Yeniden oluşturma ile aynı, şu farklar dışında:
- regenerateAction "more_like_B" olur (varyanta atıfta bulunur)
- Ajan $D variants yerine $D iterate --image B.png --brief "buna benzer, aralığı koru"
  kullanır
```

### Geri Dönüş Yolu: $D serve Başarısız

```
1. Ajan $D compare --serve çalıştırır, başarısız olur (ikili eksik, port hatası, vb.)
2. Ajan geri döner: open file:///path/board.html
3. Ajan AskUserQuestion kullanır: "Tasarım panosunu açtım. Hangi varyantı
   tercih edersiniz? Herhangi bir geri bildirim var mı?"
4. Kullanıcı metin olarak yanıt verir
5. Ajan metin geri bildirimiyle devam eder (yapılandırılmış JSON yok)
```

## Bunu Uygulayan Dosyalar

| Dosya | Rol |
|------|------|
| `design/src/serve.ts` | HTTP sunucusu, durum makinesi, dosya yazma, tarayıcı başlatma |
| `design/src/compare.ts` | Pano HTML oluşturma, derecelendirme/seçim/yeniden oluşturma JS'si, POST mantığı, gönderme-sonrası yaşam döngüsü |
| `design/src/cli.ts` | CLI giriş noktası, `serve` ve `compare --serve` komutlarını bağlar |
| `design/src/commands.ts` | Komut kayıt defteri, `serve` ve `compare`'i argümanlarıyla tanımlar |
| `scripts/resolvers/design.ts` | `generateDesignShotgunLoop()` — yoklama döngüsü ve yeniden yükleme talimatlarını çıktılayan şablon resolver |
| `design-shotgun/SKILL.md.tmpl` | Tam akışı yöneten beceri şablonu: bağlam toplama, varyant oluşturma, `{{DESIGN_SHOTGUN_LOOP}}`, geri besleme onayı |
| `design/test/serve.test.ts` | HTTP uç noktaları ve durum geçişleri için birim testleri |
| `design/test/feedback-roundtrip.test.ts` | E2E testi: tarayıcı tıklama → JS fetch → HTTP POST → diskte dosya |
| `browse/test/compare-board.test.ts` | Karşılaştırma panosu kullanıcı arayüzü için DOM-düzeyi testleri |

## Hala Ne Yanlış Olabilir

### Bilinen Riskler (olasılığa göre sıralı)

1. **Ajan sıralı oluşturma kuralına uymaz** — çoğu LLM paralel hale getirmek ister. İkili dosyada zorlama olmadan bu bir istem-düzeyi talimattır ve göz ardı edilebilir.

2. **Ajan port numarasını kaybeder** — bağlam sıkıştırması stderr çıktısını düşürür. Ajan panoyu yeniden yükleyemez. Azaltma: portu bir dosyaya yazın.

3. **Eski geri besleme dosyaları** — çöken bir oturumdan kalan `feedback-pending.json` bir sonraki çalıştırmayı karıştırır. Azaltma: başlangıçta temizle.

4. **fs.writeFileSync çökmesi** — geri besleme dosya yazımında try/catch yok. Disk doluysa sessiz sunucu ölümü. Kullanıcı sonsuz döndürücü görür.

5. **İlerleme yoklaması sapması** — 5 dakika boyunca `setInterval(fn, 2000)`. Pratikte, JavaScript zamanlayıcıları yeterince hassastır. Ama tarayıcı sekmesi arka plana alınırsa, Chrome aralıkları dakikada bire kadar kısıtlayabilir.

### İyi Çalışan Şeyler

1. **Çift kanallı geri besleme** — ön plan modu için stdout, arka plan modu için dosyalar. Her ikisi de her zaman aktif. Ajan hangisi çalışırsa onu kullanabilir.

2. **Kendi-yeterli HTML** — pano tüm CSS, JS ve base64 kodlu görüntüleri satır içi içerir. Harici bağımlılık yok. Çevrimdışı çalışır.

3. **Aynı sekme yeniden oluşturma** — kullanıcı bir sekmede kalır. Pano `/api/progress` yoklaması + `window.location.reload()` ile otomatik yenilenir. Sekme patlaması yok.

4. **Zarif bozulma** — POST başarısızlığı kopyalanabilir JSON gösterir. İlerleme zaman aşımı net hata mesajı gösterir. Sessiz başarısızlıklar yok.

5. **Gönderme-sonrası yaşam döngüsü** — pano gönderimden sonra salt-okunur olur. Zombi formlar yok. Net "sonra ne yapmalı" mesajı.

## Test Kapsamı

### Test Edilenler

| Akış | Test | Dosya |
|------|------|------|
| Gönder → diskte feedback.json | tarayıcı tıklama → dosya | `feedback-roundtrip.test.ts` |
| Gönderme-sonrası kullanıcı arayüzü kilidi | girdiler devre dışı, başarı gösterildi | `feedback-roundtrip.test.ts` |
| Yeniden oluştur → feedback-pending.json | çini + yeniden oluştur tıklama → dosya | `feedback-roundtrip.test.ts` |
| "Buna benzer" → belirli eylem | JSON'da more_like_B | `feedback-roundtrip.test.ts` |
| Yeniden oluşturmadan sonra döndürücü | DOM yükleme metni gösterir | `feedback-roundtrip.test.ts` |
| Tam yeniden oluştur → yeniden yükle → gönder | 2-tur yolculuk | `feedback-roundtrip.test.ts` |
| Sunucu rastgele portta başlar | port 0 bağlama | `serve.test.ts` |
| Sunucu URL'sinin HTML enjeksiyonu | __GSTACK_SERVER_URL kontrolü | `serve.test.ts` |
| Geçersiz JSON reddi | 400 yanıtı | `serve.test.ts` |
| HTML dosya doğrulama | eksik ise çıkış 1 | `serve.test.ts` |
| Zaman aşımı davranışı | zaman aşımından sonra çıkış 1 | `serve.test.ts` |
| Pano DOM yapısı | radyolar, yıldızlar, çini taşları | `compare-board.test.ts` |

### Test Edilmeyenler

| Boşluk | Risk | Öncelik |
|-----|------|----------|
| Çift tıklama gönder yarış durumu | Düşük — girdiler ilk yanıtta devre dışı kalır | P3 |
| İlerleme yoklaması zaman aşımı (150 yineleme) | Orta — 5 dk testte beklemek uzun | P2 |
| Yeniden oluşturma sırasında sunucu çökmesi | Orta — kullanıcı sonsuz döndürücü görür | P2 |
| POST sırasında ağ zaman aşımı | Düşük — yerel ana bilgisayar hızlıdır | P3 |
| Arka plandaki Chrome sekmesi aralıkları kısıtlar | Orta — 5 dk zaman aşımını 30+ dk uzatabilir | P2 |
| Büyük geri besleme yükü | Düşük — pano sabit şekilli JSON oluşturur | P3 |
| Eşzamanlı oturumlar (iki pano, bir sunucu) | Düşük — her $D serve kendi portunu alır | P3 |
| Önceki oturumdan eski geri besleme dosyası | Orta — yeni yoklama döngüsünü karıştırabilir | P2 |

## Potansiyel İyileştirmeler

### Kısa vadeli (bu dal)

1. **Portu dosyaya yaz** — `serve.ts` başlangıçta `serve.port`'u diske yazar. Ajan her zaman okuyabilir. 5 satır.
2. **Başlangıçta eski dosyaları temizle** — `serve.ts` başlamadan önce `feedback*.json` dosyalarını siler. 3 satır.
3. **Çift tıklamayı koru** — `handleFeedback()`'in başında `state === 'done'` kontrolü. 2 satır.
4. **try/catch dosya yazma** — `fs.writeFileSync`'i try/catch ile sarmala, başarısızlıkta 500 döndür. 5 satır.

### Orta vadeli (takip)

5. **Yoklama yerine WebSocket** — `setInterval` + `GET /api/progress` yerine WebSocket bağlantısı. Pano yeni HTML hazır olduğunda anında bildirim alır. Yoklama sapmasını ve arka plandaki sekme kısıtlamasını ortadan kaldırır. serve.ts'de ~50 satır + compare.ts'de ~20 satır.

6. **Ajan için port dosyası** — `{"port": 54321, "pid": 12345, "html": "/path/board.html"}` bilgisini `$_DESIGN_DIR/serve.json`'a yazar. Ajan stderr'yi ayrıştırmak yerine bunu okur. Sistemi bağlam kaybına karşı daha dayanıklı hale getirir.

7. **Geri besleme şema doğrulaması** — POST gövdesini yazmadan önce bir JSON şemasına karşı doğrula. Hatalı oluşturulmuş geri beslemeyi erken yakala, ajandan aşağıda karıştırmak yerine.

### Uzun vadeli (tasarım yönü)

8. **Kalıcı tasarım sunucusu** — her oturum için `$D serve` başlatmak yerine, uzun ömürlü bir tasarım daemon'u çalıştır (tarama daemon'u gibi). Birden fazla pano bir sunucuyu paylaşır. Soğuk başlangıcı ortadan kaldırır. Ama daemon yaşam döngüsü yönetimi karmaşıklığı ekler.

9. **Gerçek zamanlı işbirliği** — iki ajan (veya bir ajan + bir insan) aynı panoda aynı anda çalışır. Sunucu durum değişikliklerini WebSocket üzerinden yayınlar. Geri besleme üzerinde çakışma çözümü gerektirir.