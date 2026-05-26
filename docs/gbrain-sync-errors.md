# gbrain-sync hata arama

`gstack-brain-*` komutunun yazdırabileceği her hata mesajı, sorun, neden ve
çözüm ile birlikte.

Bu dosyayı `BRAIN_SYNC:` sonrasındaki öneke veya komut çıktısındaki ikili dosya
adına göre arayın.

---

## `BRAIN_SYNC: brain repo detected: <url>`

**Sorun.** `~/.gstack-brain-remote.txt` dosyasına sahip (başka bir makineden
kopyalanmış) bir makinedesiniz ancak `~/.gstack/.git` konumunda yerel bir git
deposu yok.

**Neden.** GBrain eşitlemesini başka bir yerde kurmuşsunuz ve gstack bu makinede
henüz geri yüklenmemiş.

**Çözüm.**
```bash
gstack-brain-restore
```
Bu, depoyu `~/.gstack/` konumuna çeker ve birleştirme sürücülerini yeniden kaydeder.

Burada geri yüklemek istemiyorsanız, ipucunu şu şekilde reddedin:
```bash
gstack-config set artifacts_sync_mode_prompted true
```

---

## `BRAIN_SYNC: blocked: <pattern-family>:<snippet>`

**Sorun.** Eşitleme, gizli tarayıcının hazırlanmış bir dosyada kimlik bilgisi
biçiminde içerik algılaması nedeniyle durdu. Sıra korundu; hiçbir şey
gönderilmedi.

**Neden.** Hazırlık öncesi gizli desenlerden biri dosya içeriğiyle eşleşti —
muhtemelen bir AWS anahtarı, GitHub belirteci, OpenAI anahtarı, PEM bloğu,
JWT veya JSON içindeki bearer belirteci.

**Çözüm (üç seçenek).**

1. **Gerçek bir gizliyse**: sorunlu dosyayı düzenleyerek gizliyi çıkarın,
   ardından eşitlemeyi yeniden denemek için herhangi bir yetenek çalıştırın.

2. **Desen yanlış pozitifse** (örneğin, öğrenmenizde yayınlamak *istediğiniz*
   bir örnek dizede GitHub belirteç örüntüsü varsa):
   ```bash
   gstack-brain-sync --skip-file <yol>
   ```
   Bu, yolu gelecekteki eşitlemelerden kalıcı olarak dışlar.

3. **Bu eşitleme grubunun tamamını iptal etmek istiyorsanız** (temiz başlangıç):
   ```bash
   gstack-brain-sync --drop-queue --yes
   ```
   Bu, sırayı commit yapmadan temizler. Gelecekteki yazmalar sırayı normal
   şekilde yeniden doldurur.

---

## `BRAIN_SYNC: push failed: auth.`

**Sorun.** Git push, uzak makineyle kimlik doğrulamanızın süresi dolduğu veya
eksik olduğu için reddedildi.

**Neden.** Uzak makine mevcut kimlik bilgileriyle erişilemez.

**Çözüm.** Uak makinenize göre kimlik doğrulamayı yenileyin:

- **GitHub**: `gh auth status` (ardından gerekirse `gh auth refresh`)
- **GitLab**: `glab auth status`
- **Diğer**: `git remote -v` + SSH anahtarlarını veya kimlik bilgisi yardımcısını denetleyin

Kimlik doğrulamayı düzelttikten sonra, eşitlemeyi otomatik olarak yeniden denemek için
herhangi bir yetenek çalıştırın.

---

## `BRAIN_SYNC: push failed: <hatanın-ilk-satırı>`

**Sorun.** Push, kimlik doğrulama dışındaki bir nedenle başarısız oldu. Git
hatasının ilk satırı iki noktadan sonra görünür.

**Neden.** Ağ sorunu, reddedilen push (uzak makine önde), sunucu 500 veya depo
erişiminin iptal edilmesi olabilir.

**Çözüm.** Daha fazla ayrıntı için `~/.gstack/.brain-sync-status.json` dosyasına bakın
veya şunu çalıştırın:
```bash
cd ~/.gstack && git status && git push origin HEAD
```
git'in tam hatasını görmek için. Sıra herhangi bir push denemesinden sonra temizlenir,
ancak yerel commitiniz hala var — sonraki yetenek çalıştırması push'u yeniden deneyecektir.

---

## `gstack-brain-init: ~/.gstack/.git zaten <url> konumuna işaret eden bir git deposu`

**Sorun.** Var olanla eşleşmeyen bir uzak URL ile başlatmaya çalıştınız.

**Neden.** `gstack-brain-init` komutunu zaten farklı bir uzak makineyle çalıştırdınız.

**Çözüm.** Şunlardan birini yapın:

- Var olan uzak makineyi kullanın: eşleşen URL ile `gstack-brain-init` komutunu
  `--remote` olmadan çalıştırın.
- Uak makineyi değiştirin: önce `gstack-brain-uninstall`, ardından yeni URL ile
  yeniden başlatın. Bu, verilerinizi silmez.

---

## `Remote not reachable: <url>`

**Sorun.** Başlatma, bağlantıyı doğrulamak için git uzak makinesine erişemedi.

**Neden.** Yanlış URL, eksik kimlik doğrulama, ağ sorunu.

**Çözüm.** El ile test edin:
```bash
git ls-remote <url>
```
Bu başarısız olursa, şunları denetleyin:
- URL yazımı
- GitHub: `gh auth status`
- GitLab: `glab auth status`
- Özel ağ / VPN / DNS

---

## `gstack-brain-init: '<name>' oluşturulamadı veya bulunamadı`

**Sorun.** `gh repo create` ile otomatik depo oluşturma başarısız oldu ve depo
`gh repo view` ile de bulunamıyor.

**Neden.** `gh` kimliği doğrulanmamış, o isimde bir depo zaten başka biriye ait
veya GitHub hesabınız kota sınırına ulaştı.

**Çözüm.**
```bash
gh auth status
```
Kimlik doğrulamasızsa, `gh auth login` çalıştırın. Depo adı çakışıyorsa, farklı bir
isim geçirin:
```bash
gstack-brain-init --remote git@github.com:KULLANICI/custom-name.git
```

---

## `gstack-brain-restore: ~/.gstack/.git zaten <url> konumuna işaret ediyor`

**Sorun.** Var olan git yapılandırmasıyla eşleşmeyen bir URL'den geri yüklemeye
çalıştınız.

**Neden.** Farklı bir uzak makineyle önceki bir başlatmadan kalan eski `.git`.

**Çözüm.** `gstack-brain-uninstall`, ardından `gstack-brain-restore <url>` komutunu
yeniden çalıştırın.

---

## `gstack-brain-restore: ~/.gstack/ dizininde üzerine yazılacak izin verilen dosyalar var`

**Sorun.** Geri yüklemeye çalışıyorsunuz ancak `~/.gstack/` dizininde zaten üzerine
yazılacak öğrenmeler veya planlar var.

**Neden.** Ya (a) bu makine eşitleme öncesi bir gstack oturumundan durum biriktirmiş
veya (b) önceki başarısız bir geri yükleme kısmi durum bırakmış.

**Çözüm (üç seçenek).**

1. **Bu makinenin durumu yeni gerçek olmalıysa**: `gstack-brain-init` komutunu
   çalıştırın — bu, bu makinenin durumundan yepyeni bir brain deposu oluşturur.

2. **Uak makineyi benimsemek ve bu makinenin durumunu atmak istiyorsanız**:
   önce `~/.gstack/projects/` dizinini yedekleyin, ardından sorunlu dosyaları kaldırın
   ve geri yüklemeyi yeniden çalıştırın.

3. **Birleştirmek istiyorsanız**: bunun için otomatik birleştirme yoktur. Öğrenmeleri
   `~/.gstack/` dizininden zaten eşitlemenin açık olduğu bir makinedeki çalışan gstack'e
   el ile kopyalayın, ardından burada geri yükleyin.

---

## `gstack-brain-restore: <url> bir gstack-brain deposu gibi görünmüyor`

**Sorun.** Klonlama başarılı oldu ancak depoda `.brain-allowlist` ve
`.gitattributes` eksik.

**Neden.** Geri yüklemeyi rastgele bir git deposuna yönlendirdiniz veya biri brain
deposundan kanonik yapılandırma dosyalarını silmiş.

**Çözüm.** URL'yi doğrulayın. Doğruysa, kanonik yapılandırmayı yeniden tohumlamak
için `gstack-brain-init --remote <url>` çalıştırın.

---

## Hiçbir şey eşitlenmiyor ama etmesini bekliyorum

**Bir hata değil, ama yaygın bir tuzak.** Şu sırayla denetleyin:

1. `gstack-brain-sync --status` — mod `off` mu?
2. `~/.gstack/.git` var mı?
3. `gstack-config get artifacts_sync_mode` — `full` veya `artifacts-only` olmalı.
4. Eşitlemesini beklediğiniz dosya — izin verilenler listesinde mi?
   `cat ~/.gstack/.brain-allowlist`
5. Gizlilik sınıfı filtresi — mod `artifacts-only` ise, davranışsal dosyalar
   (zaman çizelgeleri, geliştirici profili) bilerek atlanır.

Hepsi doğru görünüyorsa, bir boşaltımı zorlamak için şunu çalıştırın:
```bash
gstack-brain-sync --discover-new
gstack-brain-sync --once
```