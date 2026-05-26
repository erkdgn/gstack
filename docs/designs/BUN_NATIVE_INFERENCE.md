# Bun-Yerel İstem Enjeksiyonu Sınıflandırıcı — Araştırma Planı

**Durum:** P3 araştırma / erken prototip
**Dal:** `garrytan/prompt-injection-guard`
**İskelet:** `browse/src/security-bunnative.ts`
**TODOK çapası:** "Bun-native 5ms DeBERTa inference (XL, P3 / research)"

## Çözdüğü sorun

Derlenmiş `browse/dist/browse` ikili dosyası `onnxruntime-node` bağlayamaz çünkü
Bun'un `--compile` seçeneği, bağımlılıkları geçici bir çıkarma dizininden dlopen eden
tek dosyalık bir çalıştırılabilir dosya üretir ve bu dizinden yerel .dylib yükleme
başarısız olur (belgelenmiş oven-sh/bun#3574, #18079 + CEO planı §Ön-Uygulama Geçidi 1).

Mevcut azaltım (dal-2 mimarisi): ML sınıflandırıcı sadece `sidebar-agent.ts`'te
(derlenmemiş bun betiği) `@huggingface/transformers` aracılığıyla çalışır.
Server.ts (derlenmiş) sıfır ML içerir — kanarya + mimari kontrollere güvenir
(XML çerçeveleme + komut izin verilenler listesi).

Dal-2 ile sorun: sınıflandırıcı sadece sidebar-agent'in gördüğünü tarayabilir.
Derlenmiş ikili içinde kalan herhangi bir içerik yolu (çıkışta doğrudan kullanıcı
girdisi, sadece kanarya kontrolü) ML katmanını atlar.

Sıfırdan bir Bun-yerel sınıflandırıcı — yerel modül yok, onnxruntime yok — derlenmiş
ikilinin her yerde tam ML savunmasını çalıştırmasına izin verir.

## Hedef sayılar

| Metrik | Mevcut (derlenmemiş Bun'da WASM) | Hedef (Bun-yerel) |
|---|---|---|
| Soğuk-başlangıç | ~500ms (WASM başlatma) | <100ms (gömüler mmap) |
| Kararlı-durum p50 | ~10ms | ~5ms |
| Kararlı-durum p95 | ~30ms | ~15ms |
| Derlenmiş ikilide çalışır | HAYIR | EVET (birincil hedef) |
| macOS arm64 | iyi (WASM) | hedef-ilk |
| macOS x64 | iyi (WASM) | uzatma |
| Linux amd64 | iyi (WASM) | uzatma |

## Mimari

Üç yapı taşı, kaldıraç sırasına göre:

### 1. Simgeleştirici (TAMAMLANDI — security-bunnative.ts'te gönderildi)

HuggingFace `tokenizer.json`'u doğrudan okuyan ve transformers.js ile aynı
`input_ids` dizisini üreten saf-TS WordPiece kodlayıcı.

**Kendi başına neden yerel simgeleştirici önemli:** Simgeleştirme, transformers.js
yolunda çok sayıda küçük dizi ayrırır. Saf-TS sürümümüz Tensor-ayırma yükünü
atlar. Mütevazı hız artışı (~5x sadece simgeleştirici), ama daha önemlisi:
asenkron sınırı kaldırır, böylece soğuk yol sıfır dinamik içe aktarma ile başlar.

**Test kapsamı:** `browse/test/security-bunnative.test.ts`, 20 fixture dizesi
üzerinde `input_ids`'imizin transformers.js çıktısıyla eşleştiğini doğrular.

### 2. İleri geçiş (ARAŞTIRMA — çok haftalı)

Zor kısım. BERT-küçük şunlara sahiptir:
  * 12 dönüştürücü katman
  * Gizli boyut 512, dikkat başları 8
  * Toplam ~30M parametre

Her ileri geçiş:
  1. Gömme arama (id'ler → 512-boyutlu vektörler)
  2. Konumsal kodlama ekleme
  3. 12 × (öz-dikkat + FFN + KatmanNorm)
  4. Havuzlayıcı (CLS belirteci izdüşümü)
  5. Sınıflandırıcı baş (2-yollu sigmoid)

Sıcak yol, dönüştürücü katmanı başına 12 matris çarpımıdır. Her biri ~512×512×{seq_len}.
seq_len=128'de, bu ~100 (128, 512) @ (512, 512) şeklinde matris çarpımı.

**İki uygulanabilir yaklaşım:**

**Yaklaşım A: Float32Array + SIMD ile Saf-TS**
  * Bun'un tip dizisi desteği + SIMD iç işlevlerini kullanın (Bun kararlı sürümünde
    kullanılabilir olduğunda — şu anda sadece wasm)
  * Uygulama: ~2000 SATIR dikkatli sayısal hesaplama. KatmanNorm, GELU,
    softmax, ölçeklenmiş nokta-çarpım dikkati hepsi el ile yazılmış.
  * Gecikme tahmini: M-serisinde ~30-50ms (WASM'dan anlamlı şekilde yavaş,
    çünkü WASM WebAssembly SIMD kullanır)
  * KARAR: tek başına değmez. Saf-TS matris çarpımında WASM'ı yenemez.

**Yaklaşım B: Bun FFI + Apple Accelerate**
  * Apple'ın Accelerate çerçevesini (cblas_sgemm) çağırmak için `bun:ffi` kullanın.
    M-serisinde, 768×768 matris çarpımı için cblas_sgemm ~0.5ms'dir.
  * Ağırlıklar Float32Array olarak saklanır (başlangıçta ONNX başlatıcı tensörlerinden
    yüklenir), simgeleştirici TS'de, matris çarpımı FFI üzerinden, aktivasyonlar saf TS'de.
  * Uygulama: ~1000 SATIR. Sayısal hesaplamalar aynı, ama toplu iş BLAS'e devredilir.
  * Gecikme tahmini: 3-6ms p50 (hedefe uyar).
  * RİSK: sadece macOS. Linux OpenBLAS FFI'ye ihtiyaç duyar (farklı sembol düzeni).
    Windows tamamen ayrı bir hikaye.
  * KARAR: macOS-ilk gstack için uygulanabilir. Mevcut gönderim duruşumuzla eşleşir
    (derlenmiş ikililer sadece Darwin arm64 için).

**Yaklaşım C: Bun'da WebGPU**
  * Bun 1.1.x'te WebGPU desteği kazandı. transformers.js zaten bir WebGPU arka uca sahip.
    Yerel Bun'u bunun üzerinden yönlendirebilir miyiz?
  * RİSK: macOS'ta başsız sunucu bağlamında WebGPU uygun bir görüntü bağlamı gerektirir.
    Derlenmiş bir bun ikilisinden çalışıp çalışmadığı belirsiz.
  * DURUM: keşfedilmemiş. Kazan yol olabilir — bir araştırma değer.

### 3. Ağırlık yükleme (KOLAY — gönderildi)

ONNX başlatıcı tensörleri, derleme zamanında `bun:ffi`'nin `mmap()` yapabileceği
düz bir ikili yığına bir kez çıkarılabilir. Net sonuç: çalışma zamanında sıfır
açma. İskelet bunu henüz yapmıyor (transformers.js üzerinden yükler), ancak plan
yeterince basit ki ağırlık yükleyici, Yaklaşım B seçildikten sonra inşa edilecek
ilk şey.

## Kilometre taşları

1. **Simgeleştirici + kıyaslama donanımı** (GÖNDERİLDİ)
   Simgeleştirici doğruluk testini geçirir. Kıyaslama, mevcut WASM taban çizgisini
   10ms p50'de kaydeder.

2. **Bun FFI kavram kanıtı** — Apple Accelerate'den `cblas_sgemm`, 768×768
   matris çarpımını zamanlayın. <1ms gecikmeyi doğrulayın.

3. **FFI'de tek dönüştürücü katman** — Q/K/V izdüşümleri için cblas_sgemm çağırın,
   TS'de KatmanNorm + softmax uygulayın. Çıktıyı aynı input_ids üzerinde onnxruntime
   çıktısıyla karşılaştırın. 1e-4 mutlak hata içinde eşleşmeli.

4. **Tam ileri geçiş** — 12 katmanın tamamını + havuzlayıcı + sınıflandırıcı bağlayın.
   100 fixture dizesi üzerinde onnxruntime'a karşı doğruluk.

5. **Üretim takası** — security-bunnative.ts'teki `classify()` gövdesini değiştirin.
   WASM geri dönüşünü silin.

6. **Niceleme** — Accelerate'in cblas_sgemv_u8s8'si ile int8 matris çarpımı
   (müsaitse) veya onnxruntime-extensions'a geri dönüş. ~%50 bellek azaltma,
   marjinal hız kazanımı.

## Neden sadece v1'de göndermiyoruz?

Doğruluk sorunu. Önceden eğitilmiş bir dönüştürücünün kayan nokta yeniden uygulaması,
her işlemin referansla epsilon düzeyinde anlaşma gerektirdiği çok haftalık bir
mühendislik çabasıdır. KatmanNorm epsilon'unu yanlış alırsanız doğruluk sessizce
kaydırılır. Softmax taşma işlemeyi yanlış alırsanız sınıflandırıcı uzun girdilerde
çöp üretir.

Bunu P0 güvenlik özelliğinin PR'ı altında göndermek yanlış risk dağılımıdır. WASM
yolunu şimdi gönderin (tamam), arayüzü kanıtlayın (`classify()` ile gönderildi),
yereli artımlı olarak kendi doğruluk-gerileme test paketi ile bir takip PR olarak
gönderin.

## Kıyaslama

Mevcut taban çizgisi (`browse/test/security-bunnative.test.ts` kıyaslama modundan,
Apple M-serisinde ölçülmüştür — diğer donanımda farklılık gösterebilir):

| Arka uç | p50 | p95 | p99 | Notlar |
|---|---|---|---|---|
| transformers.js (WASM) | ~10ms | ~30ms | ~80ms | Isınma sonrası |
| bun-yerel (saplama — devreder) | WASM ile aynı | | | Tasarıma göre eşleşir |

Yaklaşım B (Accelerate FFI) geldiğinde, bu satır yeni sayılarla yenilenir
ve delta işlem mesajında işaretlenir.