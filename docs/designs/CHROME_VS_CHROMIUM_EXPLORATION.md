# Chrome vs Chromium: Playwright'ın Paketlenmiş Chromium'unu Neden Kullanıyoruz

## Orijinal Vizyon

`$B connect`'i inşa ettiğimizde, plan kullanıcının **gerçek Chrome tarayıcısına**
bağlanmaktı — çerezleri, oturumları, uzantıları ve açık sekmeleri olan tarayıcıya.
Daha fazla çerez içe aktarma yok. Tasarım şunları öngörüyordu:

1. `chromium.connectOverCDP(wsUrl)` çalışan bir Chrome'a CDP üzerinden bağlanma
2. Chrome'u zarifçe kapatma, `--remote-debugging-port=9222` ile yeniden başlatma
3. Kullanıcının gerçek tarama bağlamına erişim

Bu nedenle `chrome-launcher.ts` mevcuttu (361 SATIR tarayıcı ikili keşfi, CDP port
araştırması ve çalışma zamanı algılama) ve yöntem `connectCDP()` olarak adlandırılmıştı.

## Gerçekte Ne Oldu

Gerçek Chrome, Playwright'ın `channel: 'chrome'` üzerinden başlatıldığında sessizce
`--load-extension`'ı engeller. Uzantı yüklenmezdi. Yan panel (aktivite akışı,
referanslar, sohbet) için uzantıya ihtiyacımız vardı.

Uygulama, Playwright'ın paketlenmiş Chromium'u ile `chromium.launchPersistentContext()`
üzerine düştü — bu, `--load-extension` ve `--disable-extensions-except` aracılığıyla
uzantıları güvenilir bir şekilde yükler. Ama isimler kaldı: `connectCDP()`,
`connectionMode: 'cdp'`, `BROWSE_CDP_URL`, `chrome-launcher.ts`.

Orijinal vizyon (kullanıcının gerçek tarayıcı durumuna erişim) hiçbir zaman
uygulanmadı. Her seferinde yeni bir tarayıcı başlattık — işlevsel olarak Playwright'ın
Chromium'u ile aynı, ama 361 satır ölü kod ve yanıltıcı isimlerle.

## Keşif (2026-03-22)

Bir `/office-hours` tasarım oturumu sırasında, mimariyi izledik ve şunları keşfettik:

1. `connectCDP()` CDP kullanmıyor — `launchPersistentContext()`'i çağırıyor
2. `connectionMode: 'cdp'` yanıltıcı — sadece "görsel mod"
3. `chrome-launcher.ts` ölü kod — tek içe aktarımı ulaşılamaz bir `attemptReconnect()` yöntemindeydi
4. `preExistingTabIds`, hiçbir zaman bağlanmadığımız gerçek Chrome sekmelerini korumak için tasarlanmıştı
5. `$B handoff` (başsız → görsel), uzantıları yükleyemeyen farklı bir API (`launch()` + `newContext()`) kullanıyordu, iki farklı "görsel" deneyim yaratıyordu

## Düzeltme

### Yeniden adlandırılanlar
- `connectCDP()` → `launchHeaded()`
- `connectionMode: 'cdp'` → `connectionMode: 'headed'`
- `BROWSE_CDP_URL` → `BROWSE_HEADED`

### Silinenler
- `chrome-launcher.ts` (361 SATIR)
- `attemptReconnect()` (ölü yöntem)
- `preExistingTabIds` (ölü kavram)
- `reconnecting` alanı (ölü durum)
- `cdp-connect.test.ts` (silinen kodun testleri)

### Birleştirilenler
- `$B handoff` artık `launchPersistentContext()` + uzantı yükleme kullanıyor (`$B connect` ile aynı)
- Tek görsel mod, iki değil
- Handoff uzantıyı + yan paneli ücretsiz veriyor

### Kapı altına alınanlar
- Yan panel sohbeti `--chat` bayrağının arkasında
- `$B connect` (varsayılan): sadece aktivite akışı + referanslar
- `$B connect --chat`: + deneysel bağımsız sohbet ajanı

## Mimari (sonra)

```
Tarayıcı Durumları:
  BAŞSIZ (varsayılan) ←→ GÖRSEL ($B connect veya $B handoff)
     Playwright            Playwright (aynı motor)
     launch()              launchPersistentContext()
     görünmez              görünür + uzantı + yan panel

Yan panel (dikey eklenti, sadece görsel):
  Aktivite sekmesi    — her zaman açık, canlı tarama komutlarını gösterir
  Referanslar sekmesi — her zaman açık, @ref katmanlarını gösterir
  Sohbet sekmesi      — --chat ile katılım, deneysel bağımsız ajan

Veri Köprüsü (yan panel → çalışma alanı):
  Yan panel .context/sidebar-inbox/*.json dizinine yazar
  Çalışma alanı $B inbox aracılığıyla okur
```

## Neden Gerçek Chrome Değil?

Gerçek Chrome, Playwright tarafından başlatıldığında `--load-extension`'ı engeller.
Bu bir Chrome güvenlik özelliğidir — komut satırı argümanlarıyla yüklenen uzantılar,
kötü niyetli uzantı enjeksiyonunu önlemek için Chromium-tabanlı tarayıcılarda
kısıtlanır.

Playwright'ın paketlenmiş Chromium'unda bu kısıtlama yoktur çünkü test ve otomasyon
için tasarlanmıştır. `ignoreDefaultArgs` seçeneği, Playwright'ın kendi uzantı-engelleme
bayraklarını atlamamıza izin verir.

Kullanıcının gerçek çerezlerine/oturumlarına erişmek isteseydik, yol şu olurdu:
1. Çerez içe aktarma (zaten `$B cookie-import` ile çalışıyor)
2. Conductor oturum enjeksiyonu (gelecek — yan panel çalışma alanı ajanına mesajlar gönderir)

Gerçek Chrome'a yeniden bağlanmak değil.