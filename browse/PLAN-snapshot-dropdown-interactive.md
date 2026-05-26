# Plan: Snapshot Dropdown/Otomatik Tamamlama Etkileşimli Öğe Algılama

## Sorun

`snapshot -i`, modern web uygulamalarındaki dropdown/otomatik tamamlama öğelerini kaçırır. Bu öğeler:
1. Genellikle anlamsal ARIA rolleri olmayan, tıklama işleyicilerine sahip `<div>`/`<li>` elementleridir
2. Dinamik olarak oluşturulan portal/popover (yüzen konteynerler) içinde yer alır
3. Playwright'ın erişilebilirlik ağacında (`ariaSnapshot()`) görünmez

`-C` bayrağı (imleç-etkileşimli tarama) bunun için tasarlanmıştır ancak:
- Ayrı bir bayrak gerektirir — `-i` kullanan ajanlar bunu otomatik olarak almaz
- ARIA rolü OLAN öğeleri atlar (ARIA ağacı onları kaçırmış olsa bile)
- Dropdown öğelerinin bulunduğu popover/portal konteynerlerine öncelik vermez

## Temel Neden

Playwright'ın `ariaSnapshot()` işlevi, tarayıcının erişilebilirlik ağacından oluşturulur. Dinamik olarak render edilen popover'lar (React portal'leri, Radix Popover vb.) aşağıdaki durumlarda erişilebilirlik ağacında bulunmayabilir:
- Bileşen ARIA rolleri ayarlamamışsa
- Portal, kapsamlı `body` konumlayıcısının alt ağacı zamanlaması dışında render ediliyorsa
- Tarayıcı, DOM mutasyonundan sonra erişilebilirlik ağacını henüz güncellememişse

## Değişiklikler

### 1. `-i` bayrağı ile imleç-etkileşimli taramayı otomatik etkinleştir

**Dosya:** `browse/src/snapshot.ts`

`-i` (etkileşimli) geçirildiğinde, imleç-etkileşimli taramayı otomatik olarak dahil et. Bu, ajanlar etkileşimli öğeler istediğinde her zaman tıklanabilir ARIA-olmayan öğeleri görebileceği anlamına gelir.

`-C` bayrağı, etkileşimli olmayan snapshot'lar için bağımsız bir seçenek olarak kalır.

```
if (opts.interactive) {
  opts.cursorInteractive = true;
}
```

### 2. Popover/portal öncelikli tarama ekle

**Dosya:** `browse/src/snapshot.ts` (imleç-etkileşimli değerlendirme bloğu içinde)

Genel cursor:pointer taramasından önce, görünür yüzen konteynerleri (popover'lar, dropdown'lar, menüler) özel olarak tara ve tüm doğrudan alt öğelerini etkileşimli olarak dahil et:

Yüzen konteynerler için algılama buluşsal yöntemleri:
- `position: fixed` veya `z-index >= 10` ile `position: absolute`
- `role="listbox"`, `role="menu"`, `role="dialog"`, `role="tooltip"`, `[data-radix-popper-content-wrapper]`, `[data-floating-ui-portal]` vb. içeriğe sahip
- Yakın zamanda DOM'a eklenmiş (ilk sayfa yüklemesinde olmayan)
- Görünür (`offsetParent !== null` veya `position: fixed`)

Her yüzen konteyner için, şu alt öğeleri dahil et:
- Metin içeriğine sahip
- Görünür
- cursor:pointer VEYA onclick VEYA role="option" VEYA role="menuitem" içeriğine sahip
- Netlik için `popover-child` nedeni ile etiketle

### 3. İmleç-etkileşimli taramadaki `hasRole` atlamasını kaldır

**Dosya:** `browse/src/snapshot.ts`

Şu anki durum: `if (hasRole) continue;` — herhangi bir ARIA rolüne sahip öğeyi atlar, ARIA ağacının onu zaten yakaladığını varsayar.

Sorun: ARIA ağacı öğeyi KAÇIRDIYSA (zamanlama, portal, bozuk DOM yapısı), iki sistemden de düşer.

Çözüm: Yalnızca öğenin rolü `INTERACTIVE_ROLES` içindeyse VE aslında ana refMap'te yakalandıysa atla. Aksi takdirde dahil et.

`page.evaluate()` içinden refMap'i kolayca kontrol edemediğimizden, daha basit çözüm: algılanan yüzen konteynerler içindeki öğeler için `hasRole` atlamasını tamamen kaldır. Yüzen konteynerler dışındaki öğeler için, normal sayfa içeriğinde yinelenenleri önlemek adına `hasRole` atlamasını olduğu gibi tut.

### 4. Dropdown test fikstürü ve testleri ekle

**Dosya:** `browse/test/fixtures/dropdown.html`

Şunları içeren HTML sayfası:
- Odak/yazma sırasında dropdown gösteren combobox girdisi
- Tıklama işleyicilerine sahip `<div>` olarak dropdown öğeleri (ARIA rolleri yok)
- `role="option"` ile `<li>` olarak dropdown öğeleri
- React-portal tarzı konteyner (`position: fixed`, yüksek z-index)

**Dosya:** `browse/test/snapshot.test.ts`

Yeni test durumları:
- Dropdown sayfasında `snapshot -i`, imleç taraması ile dropdown öğelerini bulur
- Dropdown sayfasında `snapshot -i`, popover-child öğelerini dahil eder
- Dropdown taramasından `@c` referansları tıklanabilirdir
- ARIA rollerine sahip yüzen konteynerler içindeki öğeler, ARIA ağacı onları kaçırsa bile yakalanır

## Yayına Riski

**Düşük.** `-C` taraması ekleme niteliğindedir — yalnızca `@c` referansları ekler, hiçbir zaman `@e` referanslarını kaldırmaz. `-i` ile otomatik etkinleştirme değişikliği çıktı boyutunu artırır ancak ajanlar zaten karışık referans türlerini işleyebilmektedir.

**Bir endişe:** `-C` taraması TÜM öğeleri sorgular (`document.querySelectorAll('*')`) ve bu ağır sayfalarda yavaş olabilir. Popover'a özel tarama için, algılanan yüzen konteynerler içindeki öğelerle sınırlıyoruz, bu da hızlıdır (küçük alt ağaç).

## Test

```bash
cd /data/gstack/browse && bun test snapshot
```

## Değiştirilen Dosyalar

1. `browse/src/snapshot.ts` — -C'yi -i ile otomatik etkinleştir, popover taraması, yüzen konteynerlerde hasRole atlamasını kaldır
2. `browse/test/fixtures/dropdown.html` — yeni test fikstürü
3. `browse/test/snapshot.test.ts` — yeni dropdown/popover test durumları