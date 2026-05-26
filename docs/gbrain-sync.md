# GBrain eşitleme ile makineler arası bellek

gstack `~/.gstack/` dizinine birçok yararlı durum yazar — öğrenmeler, geribildirimler,
CEO planları, tasarım belgeleri, geliştirici profili. Varsayılan olarak, bunların hepsi
dizüstü değiştirdiğinizde kaybolur. **GBrain eşitleme**, seçilmiş bir alt kümeyi özel
bir git deposuna iter, böylece belleğiniz makineler arasında sizi takip eder ve GBrain
tarafından dizinelenebilir hale gelir.

## Ne elde edersiniz

- Makine A'da çalışın, Makine B'de sorunsuzca devam edin.
- Öğrenmeleriniz, planlarınız ve tasarımlarınız GBrain'de görünür (kullanıyorsanız).
- Verilerinize hiç dokunmayan temiz bir çıkış yolu (`gstack-brain-uninstall`).
- Artalan süreci yok, sistem servisi yok, arka plan işlemi yok.

## Makinenizden Çıkmayanlar

Tasarım gereği, eşitleme açıkken bile bunlar yerel kalır:

- Kimlik bilgileri: `.auth.json`, `auth-token.json`, `sidebar-sessions/`,
  `security/device-salt`, `config.yaml` içindeki tüketici belirteçleri
- Makineye özgü durum: Chromium profilleri, ONNX model ağırlıkları,
  önbellekler, eval-cache, CDP-profile, bir kerelik komut istemi işaretçileri
  (`.welcome-seen`, `.telemetry-prompted`, `.vendoring-warned-*` vb.)
- Soru tercihleri: makine başına UX tercihleri
  (`question-preferences.json`, `question-log.jsonl`, `question-events.jsonl`).

Tam izin verilenler listesi `~/.gstack/.brain-allowlist` dosyasındadır. CLI yönetir;
kendi girdilerinizi işaretçi satırının altına ekleyebilirsiniz.

## İlk çalıştırma kurulumu (30–90 saniye)

```bash
gstack-brain-init
```

Komut şunları yapar:

1. `~/.gstack/` dizimini bir git deposuna dönüştürür.
2. Uzak bir URL sorar (varsayılan: `gh repo create --private
   gstack-brain-$USER`). Herhangi bir git uzak makinesi çalışır — GitHub, GitLab,
   Gitea, kendi sunucunuz.
3. Yalnızca yapılandırmayı içeren bir başlangıç commiti iter.
4. `~/.gstack-brain-remote.txt` dosyasına yazar (yalnızca URL, gizli yok —
   başka bir makineye kopyalamak güvenlidir).
5. gstack-brain deposunu yerel gbrain'a federasyonlu bir kaynak olarak bağlar
   (`gbrain sources add` + `git worktree` aracılığıyla), böylece `gbrain search`
   eşitlenmiş öğrenmelerinizi, planlarınızı ve tasarımlarınızı dizine ekleyebilir.
   Uygulama `bin/gstack-gbrain-source-wireup` dosyasındadır. Eski
   `gstack-brain-reader add --ingest-url ...` HTTP yolu v1.15.1.0 sürümünde
   kaldırıldı — gbrain'ın hiçbir zaman göndermediği bir `/ingest-repo` uç noktasına
   bağımlıydı.

Başlatma sonrası, **çalıştırdığınız sonraki yetenek** size gizlilik modu hakkında
BİR soru sorar:

- **İzin verilenler listesindeki her şey (önerilen)**: öğrenmeler, incelemeler, planlar,
  tasarımlar, geribildirimler, zaman çizelgeleri ve geliştirici profilinin hepsi eşitlenir.
- **Yalnızca yapıtlar**: planlar, tasarımlar, geribildirimler, öğrenmeler — davranışsal
  verileri atlar (zaman çizelgeleri, geliştirici profili).
- **Reddet**: her şeyi yerel tutun. Daha sonra `gstack-config set artifacts_sync_mode full`
  ile eşitlemeyi açabilirsiniz.

Yanıtınız kalıcıdır. Bir daha sorulmaz.

## Makineler arası iş akışı

Makine A'da: `gstack-brain-init` komutunu bir kez çalıştırın. Bu kadar — artık her
yetenek çağrısı, başlangıç ve bitiş sınırlarında eşitleme sırasını boşaltır
(~200–800 ms ağ duraklaması).

Makine B'de:

1. `~/.gstack-brain-remote.txt` dosyasını Makine A'dan Makine B'ye kopyalayın
   (parola yöneticisi, dotfile deposu, USB bellek — size kalmış).
2. Herhangi bir gstack yeteneği çalıştırın. Önsöz, URL dosyasını görür ve şunu yazdırır:
   ```
   BRAIN_SYNC: brain repo detected: <url>
   BRAIN_SYNC: run 'gstack-brain-restore' to pull your cross-machine memory
   ```
3. `gstack-brain-restore` çalıştırın. Bu, depoyu klonlar, öğrenmelerinizi/planlarınızı/
   geribildirimlerinizi yeniden yükler ve git birleştirme sürücülerini yeniden kaydeder.
4. Tüketici belirteçlerini yeniden girin (bunlar makineye özeldir ve eşitlenmez —
   `gstack-config set gbrain_token <belirteciniz>`).
5. Sonraki yetenek: dünkü Makine-A öğrenmeniz ortaya çıkar. İşte o sihirli an.

## Durum, sağlık ve sıra derinliği

```bash
gstack-brain-sync --status
```

Şunları gösterir: son başarılı push, bekleyen sıra derinliği, varsa eşitleme
engelleri ve geçerli gizlilik modu.

Her yetenek çalıştırması, önsöz çıktısının üst kısmında bir `BRAIN_SYNC:` satırı
yazdırır. Sorunlar için bunu tarayın.

## Gizlilik modları ayrıntılı

| Mod | Ne eşitlenir |
|------|------------|
| `off` | Hiçbir şey (varsayılan). |
| `artifacts-only` | Planlar, tasarımlar, geribildirimler, öğrenmeler, incelemeler. Zaman çizelgeleri + geliştirici profilini atlar. |
| `full` | İzin verilenler listesindeki her şey, davranışsal durum dahil. |

İstediğiniz zaman değiştirin:
```bash
gstack-config set artifacts_sync_mode full
gstack-config set artifacts_sync_mode off
```

## Gizli koruma

Her commit, makinenizden ayrılmadan önce kimlik bilgisi biçiminde içerik için taranır.
Engellenen desenler şunları içerir:

- AWS erişim anahtarları (`AKIA…`)
- GitHub belirteçleri (`ghp_`, `gho_`, `ghu_`, `ghs_`, `ghr_`, `github_pat_`)
- OpenAI anahtarları (`sk-…`)
- PEM blokları (`-----BEGIN …-----`)
- JWT'ler (`eyJ…`)
- JSON içindeki bearer belirteçleri (`"authorization": "…"`, `"api_key": "…"`, vb.)

Bir tarama isabet ederse, eşitleme durur, sıra korunur ve önsözünüz şunu yazdırır:

```
BRAIN_SYNC: blocked: <pattern-family>:<snippet>
```

Çözüm için:

1. Sorunlu dosyayı inceleyin.
2. Eşleşme açıkça eşitlemek istediğiniz içerikteki yanlış bir pozitifse,
   o yolu kalıcı olarak dışlamak için `gstack-brain-sync --skip-file <yol>` çalıştırın.
3. Aksi takdirde, gizliyi kaldırmak için dosyayı düzenleyin ve herhangi bir yetenek
   çalıştırın.

`~/.gstack/.git/hooks/pre-commit` konumunda, depoya el ile `git commit` yaparsanız
aynı taramayı çalıştıran bir derinlemesine savunma kancası vardır.

## İki makine çakışmaları

Aynı gün Makine A ve Makine B'de yazarsanız, her ikisi de ekleme commitleri iter.
Git'in varsayılanı dosya kuyruğunda çakışma yaratur, ancak `.jsonl` ve markdown
dosyaları özel birleştirme sürücüleri ile kayıtlıdır:

- JSONL dosyaları, eklemeleri ISO zaman damgasına göre sıralayan ve tekrarları kaldıran
  bir sürücü kullanır (belirleyicilik için her satırın SHA-256 karmasına düşer).
- Markdown yapıtları (geribildirimler, planlar, tasarımlar) her iki tarafı
  birleştiren bir birleşim sürücüsü kullanır.

Çakışma komut istemleri görmemelisiniz. Görürseniz (gerçek anlamsal bir çakışma,
örneğin iki makine aynı planı düzenlemek), git durur ve komut istemi gösterir.

## Makineler arası çekme sıklığı

Önsöz, 24 saatte bir `git fetch` + `git merge --ff-only` çalıştırır
(`~/.gstack/.brain-last-pull` ile önbelleğe alınır). Bunun hakkında düşünmenize
gerek yok — her günün ilk yetenek çağrısında otomatik olarak gerçekleşir.

## Kaldırma

```bash
gstack-brain-uninstall
```

Bu şunları yapar:

- `~/.gstack/.git/` dizinini ve tüm `.brain-*` yapılandırma dosyalarını kaldırır.
- `gstack-config` içindeki `artifacts_sync_mode` ayarını temizler.
- Öğrenmelerinizi, planlarınızı, geribildirimlerinizi veya geliştirici profilinizi
  dokunmaz.

Özel GitHub deposunu da silmek için `--delete-remote` ekleyin (yalnızca GitHub,
`gh repo delete` kullanır).

İstediğiniz zaman `gstack-brain-init` ile yeniden başlatabilirsiniz.

## Sorun giderme

gstack-brain komutunun yazdırabileceği her hata mesajı için, her biri için sorun /
neden / çözüm ile birlikte [gbrain-sync-errors.md](gbrain-sync-errors.md) dosyasına bakın.

## Kaputun altında

Bu özelliğin arkasındaki mimari kararlar (izin verilenler listesi vs ret listesi,
artalan süreci vs önsöz sınırı eşitlemesi, JSONL birleştirme sürücüsü, gizlilik
durdurma kapısı) için gstack planlar dizinindeki
[onaylanmış plana](../system-instruction-you-are-working-jaunty-kahn.md) bakın.