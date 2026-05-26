# GStack iOS ile iOS Uygulamaları Nasıl Test Edilir

Bu, gstack ile birlikte gelen iOS QA yeteneği için uçtan uca yönlendirmedir:
kanonik Swift şablonlarını uygulamanıza yükleyin, gerçek bir iPhone'u USB üzerinden
bağlayın ve herhangi bir aracıdan (yerel olarak Claude Code veya Tailscale üzerinden
HTTP yapabilen herhangi bir aracı) sürün. Simülatör yok, XCTest donanımı yok,
WebDriverAgent yok.

Aşağıdaki her şey, iOS 26.5 çalıştıran gerçek bir iPhone 17 Pro Max'de uçtan uca
doğrulanmıştır. Aynı akış, iOS 16+ cihazlarda çalışır.

## İhtiyacınız olanlar

- Xcode 16.0+ kurulu macOS (`xcrun devicectl --version` başarılı olmalı). Xcode 16,
  `devicectl` komutunun cihaza USB üzerinden ulaşmak için kullandığı CoreDevice
  tünellerini gönderir.
- iOS 16 veya sonraki bir gerçek iPhone. Kilitli olmayan, Mac'inizle eşleştirilmiş,
  Ayarlar → Gizlilik ve Güvenlik'de **Geliştirici Modu** etkinleştirilmiş.
- Bir Apple geliştirici ekibi — ücretsiz kişisel ekip canlı cihaz hata ayıklama
  dağıtımları için çalışır. Ekip kimliğine (örn. `623FYQ2M88`), sertifika kimliğine
  değil, ihtiyacınız var. Xcode → Ayarlar → Hesaplar → Apple Kimliğiniz → ekip
  listesi'nde bulunur. Kurulum, uygulamayı ilk dağıtımda
  `-allowProvisioningUpdates -allowProvisioningDeviceRegistration` ile cihazınız için
  imzalar.
- gstack kurulu (`./setup` tamamlanmış; `bin/gstack-ios-qa-daemon` diskte olmalı ve
  çalıştırılabilir olmalı).
- PATH üzerinde Bun çalışma zamanı (`bun --version`). Mac tarafındaki artalan süreci
  bir bun sürecidir.

İsteğe bağlı uzak-aracı (Tailscale) modu için ayrıca Mac'te Tailscale kurulu olmalı ve
`/var/run/tailscale.sock` okunabilir olmalıdır.

## Bir nefeste mimari

```
┌─────────────────┐   tailnet (isteğe bağlı)    ┌──────────────────────┐   USB CoreDevice    ┌─────────────────────┐
│ Uzak aracı      │ ─────────────────▶ │ gstack-ios-qa-daemon │ ──────────────────▶ │ iOS uygulaması StateServer │
│ (Claude, GPT,   │  bearer + oturum   │  (Mac, bun/TS)       │  IPv6 ULA tüneli    │  (yalnızca geri döngü)    │
│  OpenClaw, ...) │                    │                      │                     │                     │
└─────────────────┘                    └──────────────────────┘                     └─────────────────────┘
```

- iOS uygulaması, `::1` + `127.0.0.1` 9999 portunda dinleyen bir `StateServer`
  (`DebugBridge` SPM kütüphanesi, yalnızca `#if DEBUG`) içerir. Bearer belirteçli
  geçitli. Önyükleme belirteci, artalan süreci başlatıldıktan sonraki ~5 saniye içinde
  döndürülür, böylece `os_log` kazıyan her şey ölü bir kimlik bilgisi görür.
- Mac artalan süreci, eşleştirilmiş bir cihaz bağlandığında `xcrun devicectl`
  komutunun otomatik olarak açtığı CoreDevice IPv6 tüneli üzerinden trafiği yönlendirir.
- Tailscale modunda, artalan süreci tailnet IP'nize bağlı ayrı bir dinleyici sunar,
  oturum belirteci başına uygulanan yetenek katmanları (gözlem / etkileşim / değişiklik /
  geri yükleme) ile. Belirteçler Mac sahibi tarafından `gstack-ios-qa-mint` aracılığıyla
  açıkça basılır; uzak çağrıcılar asla otomatik olarak izin verilenler listesine eklenmez.

iOS `StateServer` **her zaman** yalnızca geri döngüdür, uzak modda bile. Kimlik
doğrulama Mac tarafında gerçekleşir çünkü iPhone'un bir Tailscale kimliğini doğrulama
yolu yoktur.

## 1. Adım: DebugBridge şablonlarını iOS uygulamanıza ekleyin

Şablonlar `./setup` komutundan sonra `~/.claude/skills/gstack/ios-qa/templates/`
konumundadır. En hızlı kurulum, uygulamanızın kökünden Claude Code'da `/ios-qa`
yeteneğini çağırmaktır — Swift kaynak kodunuzu okur, tipli `@Observable` durum
erişimcileri oluşturur ve şablonları paket kimliğinizle indirir. Ya da el ile yapın:

1. Bunları uygulamanızın çalışma alanı içinde bir `DebugBridge/` SPM paketine kopyalayın:
   - `Sources/DebugBridgeCore/StateServer.swift` (`StateServer.swift.template` dosyasından)
   - `Sources/DebugBridgeCore/DebugBridgeManager.swift` (`DebugBridgeManager.swift.template` dosyasından)
   - `Sources/DebugBridgeTouch/DebugBridgeTouch.m` + `Sources/DebugBridgeTouch/include/DebugBridgeTouch.h` (iki `.template` dosyasından)
   - `Sources/DebugBridgeUI/Bridges.swift` (`Bridges.swift.template` dosyasından)
   - `Sources/DebugBridgeUI/DebugOverlay.swift` (`DebugOverlay.swift.template` dosyasından)
   - `Package.swift` (`Package.swift.template` dosyasından)
2. Paketi uygulamanızın yerel bir bağımlılığı olarak ekleyin. `DebugBridgeUI` ürününe
   `condition: .when(configuration: .debug)` ile bağımlı olun. `DebugBridgeCore` ve
   `DebugBridgeTouch` geçişli olarak gelir.
3. `@main` App init'inizde bağlantıyı `#if DEBUG` ile geçitleyin:

   ```swift
   #if DEBUG
   import DebugBridgeCore
   StateServer.shared.start()
   #if canImport(UIKit)
   import DebugBridgeUI
   DebugBridgeUIWiring.installAll()
   #endif
   #endif
   ```

Üç Swift hedefi şöyle bölünür: `DebugBridgeCore` çapraz platformludur (böylece bir CI
Mac ana bilgisayarında `swift build` UIKit olmadan kodun çoğunu doğrulayabilir),
`DebugBridgeUI` ve `DebugBridgeTouch` yalnızca iOS'tur (UIKit'e bağlantı verirler).
`DebugBridgeTouch` Objective-C'dir — iOS 18+ `_UIHitTestContext` düzeltmesi ile
SwiftUI Düğme dokunuşlarının gerçekten çalışmasını sağlayan KIF kaynaklı UITouch
sentezini taşır.

Yapısal Release-build koruması, `Package.swift` içindeki `.when(configuration: .debug)`
ifadesidir. SwiftPM herhangi bir `DebugBridge*` hedefini bir Release derlemesinde
bağlantılamayı reddeder, bu nedenle köprüyü temizlemeyi unutsanız bile TestFlight'a
gönderilemez.

## 2. Adım: Derle + cihaza yükle

Uygulamanın proje dizininden:

```
xcodebuild \
  -scheme YourAppScheme \
  -configuration Debug \
  -destination 'generic/platform=iOS' \
  -derivedDataPath /tmp/build \
  -allowProvisioningUpdates -allowProvisioningDeviceRegistration \
  CODE_SIGN_STYLE=Automatic \
  DEVELOPMENT_TEAM=YOUR_TEAM_ID \
  build
```

Ardından yükle + başlat:

```
UDID=$(xcrun devicectl list devices 2>/dev/null | awk 'NR>2 && $0!="" {print $(NF-2); exit}')
xcrun devicectl device install app --device "$UDID" /tmp/build/Build/Products/Debug-iphoneos/YourApp.app
xcrun devicectl device process launch --device "$UDID" --terminate-existing your.bundle.id
```

Telefon kilitliyse `FBSOpenApplicationServiceErrorDomain error 1 — Locked` alırsınız.
Kilidi açın ve yeniden deneyin. İlk yükleme, telefonda bir Güven iletişim kutusu gösterir;
Güven'e dokunun, ardından yeniden çalıştırın.

## 3. Adım: Mac tarafındaki artalan sürecini başlatın

İki seçenek.

**Seçenek A — yeteneğin başlatmasına izin verin.** Claude Code'da herhangi bir yerden
`/ios-qa` çalıştırın; yetenek artalan sürecini talep üzerine başlatır, tüneli önyükleme
yapar, önyükleme belirtecini döndürür ve cihazı vekil üzerinden kullanıma sunar.
Yerel USB kullanımı için en temiz yol.

**Seçenek B — kendiniz başlatın.** Çalıştırın:

```
gstack-ios-qa-daemon
```

Artalan süreç, her iki geri döngü dinleyicisi bağlandığında `READY: port=<n> pid=<pid>`
yazdırır. Varsayılan port 9099'dur. Başlatıcılar, hazır olmayı onaylamak için ~5 saniyelik
zaman aşımı ile bu satırı okuyabilir; ayrıca yazdırılan porta `curl` ile işaret
edebilirsiniz.

Her iki şekilde de artalan süreç `~/.gstack/ios-qa-daemon.pid` üzerinde özel bir flock
alır — iki Claude Code oturumundan çalıştırmak güvenlidir; ikinci çağrım, çalışan artalan
sürecinin portunu keşfeder ve katılır.

Belirli bir cihazı veya paketi hedeflemek için bu ortam değişkenlerini ayarlayın:

```
GSTACK_IOS_TARGET_UDID=248C3A58-B843-5BDB-8F5D-89ADB7D7BF6A
GSTACK_IOS_TARGET_BUNDLE_ID=com.yourorg.yourapp
GSTACK_IOS_DAEMON_PORT=9099       # geri döngü dinleyici portu; varsayılan 9099
```

`GSTACK_IOS_TARGET_UDID` ayarlanmamışsa, artalan süreç eşleştirilmiş bağlanan ilk cihazı seçer.

## 4. Adım: Cihazı sürün

Artalan süreç çalıştığında, `http://127.0.0.1:9099` (veya `[::1]:9099`) konumunda bir
HTTP yüzeyiniz var. Yetenek akışı bunu sizin için yapar, ancak ham uç noktalar şunlardır:

| Uç nokta | Ne yapar | Kimlik doğrulama |
|---|---|---|
| `GET /healthz` | Sürüm yoklaması. | yok (geri döngü) |
| `POST /auth/rotate` | Yalnızca artalan süreci; önyükleme belirtecini yalnızca bellekte bir değere döndürür. | önyükleme belirteci |
| `POST /session/acquire` | Cihaz başına oturum kilidini al. `{session_id, ttl_seconds}` döndürür. | bearer |
| `POST /session/release` | Kilidi serbest bırak. | bearer + oturum |
| `GET /screenshot` | Etkin pencerenin PNG ekran görüntüsünü yakala. `{png_base64: "..."}` döndürür. | bearer |
| `GET /elements` | Erişilebilirlik-ağacı anlık görüntüsü. | bearer |
| `GET /state/snapshot` | Her `@Snapshotable` alanını JSON olarak dök. | bearer |
| `POST /state/restore` | Tam bir anlık görüntüyü atomik olarak geri yükle. | bearer + oturum, değişiklik katmanı |
| `POST /tap` `{x,y}` | Pencere koordinatlarında gerçek bir UITouch sentezle. SwiftUI Düğmeleri çalışır. | bearer + oturum, etkileşim katmanı |
| `POST /swipe` `{from_x,from_y,to_x,to_y}` | En yakın kapsayan UIScrollView'u kaydır. | bearer + oturum, etkileşim katmanı |
| `POST /type` `{text}` | Geçerli ilk yanıtlayıcıda metin ayarla. | bearer + oturum, etkileşim katmanı |

Değişiklik yapan istekler hem bir `Authorization: Bearer <token>` başlığı hem de bir
`X-Session-Id` başlığı gerektirir. Okuma uç noktaları (`/screenshot`, `/elements`,
`GET /state/*`) yalnızca bearer gerektirir.

Durum anlık görüntüsü, kurallı durum yapınızdaki `@Snapshotable` özellik sarmalayıcısı
aracılığıyla alan başına katılım tabanlıdır. Açıklama eklemeyen alanlar anlık görüntüde
asla görünmez, bu da belirteçleri, PII ve kimlik doğrulama durumunu varsayılan olarak
kayıtlı sabitlerden uzak tutar.

## 5. Adım: Uzak aracıları çalışır hale getirme (isteğe bağlı)

Başka bir makinedeki bir aracının cihazı sürmesine izin vermek için, artalan süreci
`--tailnet` ile çalıştırın:

```
gstack-ios-qa-daemon --tailnet
```

Artalan süreç önce `/var/run/tailscale.sock` dosyasını yoklar; soket eksikse veya
okunamazsa, tailnet dinleyicisini hiç açmaz (geri döngü hala çalışır). Uzak mod asla
yarım başlamaz.

Ardından bağlanabilmesi gereken kimlik için bir oturum belirteci basın:

```
gstack-ios-qa-mint grant --remote 'alice@example.com' --capability interact
gstack-ios-qa-mint grant --remote 'tag:ci' --capability mutate --ttl 86400 --note 'nightly'
gstack-ios-qa-mint list
```

Yetenek katmanları iç içedir: `observe` (yalnızca okuma uç noktaları) ⊂ `interact`
(dokunuşlar, kaydırmalar, yazma) ⊂ `mutate` (`POST /state/*`) ⊂ `restore`
(`POST /state/restore`). İşi yapan en küçük katmanı seçin. İzin verilenler listesi
dosyası `~/.gstack/ios-qa-allowlist.json` konumundadır (mod 0600) — artalan süreç her
`/auth/mint` isteğinde bunu okur, bu nedenle değişiklikler artalan sürecini yeniden
başlatmadan hemen geçerli olur.

Uzak aracı daha sonra artalan sürecinin tailnet dinleyicisine karşı `POST /auth/mint`
isteğinde bulunur. Artalan süreç, çağrıcının kimliğini tailscaled'nin WhoIs uç noktası
aracılığıyla kurallaştırır, izin verilenler listesini denetler ve kısa ömürlü bir oturum
belirteci döndürür (varsayılan 1 saat, en fazla 24 saat). Her kimliği doğrulanmış
değişiklik yapan istek `~/.gstack/security/ios-qa-audit.jsonl` dosyasına; reddedilen
istekler `~/.gstack/security/attempts.jsonl` dosyasına kaydedilir.

## 6. Adım: Release derlemesi gönderin

TestFlight veya App Store'a göndermeden önce `/ios-clean` çalıştırın. Bu, `DebugBridge`
SPM bağımlılığını kaldırır ve `@main` App'inizdeki `#if DEBUG` bağlantılamasını çıkarır.
`Package.swift`'teki yapısal koruma (`condition: .when(configuration: .debug)`),
temizlemeyi unutsanız bile Release derlemesinin köprüyü bağlantılamayacağı anlamına gelir,
ancak `/ios-clean` size incelemek ve göndermek için düzenli bir diff verir.

## Yaygın hatalar

| Belirti | Ne bozuldu |
|---|---|
| `xcodebuild` `Could not locate device support files for iOS X.Y` ile başarısız oluyor | iPhone'unuzun iOS sürümü için cihaz destek paketini indirmek için `xcodebuild -downloadPlatform iOS` çalıştırın (~8GB). |
| Yükleme başarılı, `process launch` `Locked` ile başarısız oluyor | Telefon kilitli. Kilidi açın ve yeniden deneyin. |
| Eşleştirilmiş bir cihazda ilk yükleme net bir hata olmadan başarısız oluyor | Telefonun Mac'e güvenmesi gerekiyor. Telefonda Ayarlar → Genel → VPN ve Cihaz Yönetimi'ni açın ve onaylayın. |
| `Developer Mode` geçişi Ayarlar → Gizlilik'te yok | Cihazı Xcode → Pencere → Cihazlar ve Simülatörler'e bir kez bağlayın veya ona karşı herhangi bir `devicectl device install` deneyin. iOS, ilk denemeden sonra geçişi gösterir. |
| `xcrun devicectl device copy from` ERROR 7000 döndürüyor | Kaynak yolu yanlış — önyükleme belirteci uygulamanın veri kapsayıcısında `tmp/gstack-ios-qa.token` konumundadır (NSTemporaryDirectory), yolun kökünde değil. |
| `/healthz` 200 döndürüyor ama `/tap` ok:true döndürüyor ve UI'da değişiklik yok | Telefon eşleştirilmiş ama StateServer portu lansmanlar arasında değişmiş olabilir. CoreDevice IPv6'yı yeniden çözün (`dscacheutil -q host -a name '<CihazAdı>.coredevice.local'`). |
| `/auth/mint`'ten `403 identity_not_allowed` | Uzak çağrıcının kimliği Mac'in izin verilenler listesinde yok. Mac'te `gstack-ios-qa-mint grant --remote <kimlik> --capability interact` çalıştırın. |
| Artalan süreç tailnet dinleyicisini açmıyor | Tailscale kurulu değil veya `/var/run/tailscale.sock` okunamaz. Tailscale'i düzeltin, ardından artalan sürecini yeniden başlatın. Bu arada geri döngü hala çalışır. |
| SwiftUI Düğme dokunuşu `ok:true` döndürüyor ama eylem hiç çalışmıyor | iOS 17 veya daha eskisinde `_UIHitTestContext` mevcut değil. DebugBridgeTouch uygulaması, SwiftUI'nin jest konteynerine çözümlenmeyen düz `hitTest:`'e geri döner. Cihazda iOS 18+'ya güncelleyin veya bunun yerine bir UIKit denetimine dokunun. |

## Bu size ne sağlar

HTTP konuşan herhangi bir dilde bir aracı döngüsü yazabilirsiniz. Bir ekran görüntüsü
alın, bir modele ne yapacağını sorun, bir dokunuş gönderin. Belirleyici sabitler kaydetmek
için önce ve sonra durum anlık görüntülerini yakalayın — `/ios-fix` regresyon testleri için.
Bir work arkadaşı izin verilenler listesine ekleyin ve asla donanıma dokunmadan Tailscale
üzerinden dizüstü bilgisayarlarından iPhone'unuzu sürsünler. Aynı artalan sürecini CI'ye
değişiklik katmanı yeteneği ve 24 saatlik TTL ile bir `tag:ci` oturum belirteci basarak
bağlayın.

Tüm yığın, zaten sahip olduğunuz bir Mac, zaten sahip olduğunuz bir iPhone, ücretsiz bir
Apple geliştirici hesabı ve gstack'tir. Ücretli test hizmeti yok. Simülatör kayması yok.
Kullanıcının gördüğü şey, aracının sürdüğü şeydir.