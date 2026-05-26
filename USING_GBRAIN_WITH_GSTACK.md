# GBrain'i GStack ile Kullanmak

Kodlama aracınız, gerçekten sakladığı bir belleğe sahip.

[GBrain](https://github.com/garrytan/gbrain), yapay zeka aracıları için tasarlanmış kalıcı bir bilgi tabanıdır. Aracınızın öğrendiklerini, verdiğiniz kararları, nelerin işe yaradığını ve nelerin yaramadığını saklar ve aracının tümüne talep üzerine arama yapmasını sağlar. GStack, sıfırdan "gbrain çalışıyor ve aracım onu çağırabiliyor" durumuna tek komutla geçiş yolu sunar — deneme-yerel, takımla-paylaşım ve aradaki her şey için yollar mevcuttur.

Bu tam kılavuzdur: her senaryo, her bayrak, her yardımcı komut, her sorun giderme adımı. Hızlı özet için [README'nin GBrain bölümüne](README.md#gbrain--persistent-knowledge-for-your-coding-agent) bakın. Hata kodları ve eşzamanlama sorunları için [docs/gbrain-sync.md](docs/gbrain-sync.md) belgesine bakın.

---

## Tek komutla kurulum

```bash
/setup-gbrain
```

Bu kadar. Yetenek (skill) mevcut durumunuzu algılar, en fazla üç soru sor ve kurulum, başlatma, Claude Code için MCP kaydı ve depo başına güven politikası adımlarında size rehberlik eder. Hiçbir şey kurulmamış temiz bir Mac'te beş dakikadan kısa sürede tamamlanır. Zaten bir şeyler kurulmuş bir Mac'te saniyeler sürer (mevcut durumu algılar ve yapılmış işleri atlar).

## Kurulum sonrası elde ettikleriniz

`/setup-gbrain` tamamlandığında, kodlama aracınız daha önce sahip olmadığı iki erişim yüzeyi olur:

- **Bu depoda anlamsal kod araması.** `gbrain search "browser security canary"` tam eşlemeli grep sonuçları yerine sıralanmış dosya bölgeleri döndürür. `gbrain code-def`, `code-refs`, `code-callers`, `code-callees` sembol üzerinden çağrı grafiğini takip eder — hangi dosyada uygulama olduğunu bilmediğiniz ama ne yaptığını bildiğiniz durumlarda kullanışlıdır. Aracı, soru anlamsal olduğunda bunları Grep yerine tercih eder; CLAUDE.md dosyasına yönlendirme kurallarını öğreten bir `## GBrain Search Guidance` bloğu eklenir.
- **Oturumlar arası bellek.** Geçmiş oturumlardan planlar, retrospektifler, kararlar ve öğrenimler `~/.gstack/` içinde yaşar ve (artefakt eşzamanlamasını etkinleştirdiyseniz) gbrain'in dizine eklediği özel bir git deposuna gönderilir. `gbrain search "auth hakkında ne karar vermiştik?"` sorgusu eski CEO planını gerçekten bulur — her oturumda bağlamı yeniden açıklamanız gerekmez.

Uzaktan MCP'yi de etkinleştirdiyseniz (aşağıdaki Yol 4), beyin sorguları diğer makinelerin yazabileceği paylaşılan bir beyin sunucusuna yönlendirilir — dizüstü bilgisayarınız, masaüstünüz ve bir takım arkadaşınızın makinesi aynı belleği görür.

## Dört yol

Yetenek "Beyininiz nerede yaşamalı?" diye sorduğunda birini seçersiniz.

### Yol 1: Supabase, zaten bir bağlantı dizginiz var

En uygun olan: siz (veya bir takım arkadaşınızın bulut aracısı) zaten bir Supabase beyini hazırlamış ve bu yerel makinenin aynı veriyi kullanmasını istiyorsunuz.

**Ne olur:** Session Pooler URL'sini yapıştırın (Settings → Database → Connection Pooler → Session → URI kopyala, port 6543). Yetenek URL'yi echo kapalı olarak okur, sansürlenmiş bir önizleme gösterir (`aws-0-us-east-1.pooler.supabase.com:6543/postgres` — ana bilgisayar görünür, parola maskelenmiş), `GBRAIN_DATABASE_URL` ortam değişkeni aracılığıyla `gbrain init`'e iletir ve URL hiçbir zaman argv'ye veya kabuk geçmişinize yazılmaz.

**Güvenlik uyarısı:** Bu URL'yi yapıştırmak, yerel Claude Code'unuza paylaşılan beyindeki her sayfa için tam okuma/yazma erişimi verir. İstediğiniz güven düzeyi bu değilse, bunun yerine PGLite yerel (Yol 3) seçin ve beyinlerin ayrı kalmasını kabul edin.

### Yol 2a: Supabase, yeni proje otomatik hazırlama

En uygun olan: yeni Supabase hesabı, tıklama olmadan temiz yeni bir proje istiyorsunuz.

**Ne olur:** Bir Supabase Personal Access Token (PAT) yapıştırırsınız. Yetenek önce kapsam bilgilendirmesini gösterir — *bu token, oluşturacağımız proje dahil Supabase hesabınızdaki her projeye tam erişim verir*. Kuruluşlarınızı listeler, hangi kuruluş ve hangi bölgeyi sorar (varsayılan `us-east-1`), bir veritabanı parolası oluşturur, `POST /v1/projects` çağırır, proje `ACTIVE_HEALTHY` olana kadar her 5 snyede `GET /v1/projects/{ref}` yoklar (180 s zaman aşımı), pooler URL'sini alır, `gbrain init`'e iletir. Uçtan uca: ~90 saniye.

Sonunda: PAT'yi https://supabase.com/dashboard/account/tokens adresinden iptal etmeniz için açık hatırlatma. Yetenek PAT'yi belleğinden zaten çıkardı.

**Hazırlama sırasında Ctrl-C yaparsanız:** SIGINT tuzağı devam eden proje referansınızı ve bir sürdürme komutunu yazdırır. Yetim projeyi Supabase panosundan silebilir veya `/setup-gbrain --resume-provision <ref>` ile kaldığınız yerden devam edebilirsiniz.

### Yol 2b: Supabase, manuel oluşturma

En uygun olan: PAT yapıştırmak yerine supabase.com'da kendiniz tıklamayı tercih ediyorsunuz.

**Ne olur:** Yetenek dört manuel adımda size rehberlik eder (kayıt → yeni proje → ~2 dk bekle → Session Pooler URL'sini kopyala), ardından Yol 1'in yapıştırma adımından devralır. Yol 1 ile aynı güvenlik uygulaması.

### Yol 3: PGLite yerel

En uygun olan: önce deneyim, hesap yok, bulut yok, paylaşım yok. Veya herhangi bir bulut aracından yalıtılmış, "bu Mac'in beyni" kalmalı.

**Ne olur:** `gbrain init --pglite`. Beyin `~/.gbrain/brain.pglite` konumunda yaşar. Başlatma için ağ çağrısı yok. 30 saniyede biter.

**Gömme modeli.** `VOYAGE_API_KEY` ayarlandığında, gstack PGLite'ı `voyage-code-3` (1024-boyut) ile başlatır — Voyage'in koda özel gömme modeli, bu kod tabanının sembol sorgularında genel amaçlı `voyage-4-large` ve OpenAI `text-embedding-3-large` modellerini bire bir geride bırakır. `VOYAGE_API_KEY` olmadığında, gbrain otomatik seçim yapar (`OPENAI_API_KEY` varsa OpenAI 1536-boyut, yoksa sağlayıcı zincirinde aşağı iner). Her durumda, gömmeler eşzamanlama sırasında seçilen sağlayıcının API'sine çağrı yapar — `/sync-gbrain` çalıştırmadan önce istediğiniz sağlayıcının anahtarını ayarlayın.

Yalnızca gbrain'in nasıl hissettirdiğini buluta bağlanmadan görmek istiyorsanız bu en iyi ilk tercihtir. Daha sonra `/setup-gbrain --switch` ile taşınabilirsiniz.

### Yol 4: Uzaktan gbrain MCP (bölünmüş motor)

En uygun olan: beyniniz kontrol ettiğiniz başka bir makinede çalışıyor (Tailscale, ngrok, iç LAN) veya bir takım arkadaşınızın sunucusu. Yerel bir veritabanı kurmadan makineler arası bellek avantajı istiyorsunuz ve yine bu Mac'te sembol duyarlı kod araması istiyorsunuz.

**Ne olur:** Bir MCP URL'si (örn. `https://wintermute.tail554574.ts.net:3131/mcp`) ve bir taşıyıcı (bearer) token yapıştırırsınız. Yetenek URL'yi ağ üzerinden doğrular, gbrain'i kullanıcı kapsamında `~/.claude.json` dosyasında HTTP MCP olarak kaydeder ve kod araması için küçük bir yerel PGLite da kurmayı önerir (~30 saniye, ~120 MB disk).

Yerel PGLite'ı kabul ederseniz, **bölünmüş motor modunda** sonuçlanırsınız:

- **Beyin/bağlam sorguları** (`mcp__gbrain__search`, `mcp__gbrain__query`, `mcp__gbrain__get_page`) uzaktan MCP'ye yönlendirilir. Planlar, retrospektifler, öğrenimler, makineler arası bellek — hepsi paylaşılan sunucuda.
- **Kod sorguları** (`gbrain code-def`, `code-refs`, `code-callers`, `code-callees`, kod için `gbrain search`) her çalışma ağacındaki `.gbrain-source` sabitlemesi aracılığıyla yerel PGLite'a yönlendirilir. Yerel olarak dizine eklenir, hızlıdır, asla makineden ayrılmaz.

İki motor birbirinden bağımsızdır. Yerel PGLite'ı silmek uzaktan beyni etkilemez; uzaktan MCP taşıyıcı token'ını döndürmek yerel kod aramasını etkilemez. Bu ayrıca, uzaktan beyin yöneticinizin her geliştiricinin checkout'unu dizine ekleyememesi (veya eklememesi) gerektiğinde de doğru yapılandırmadır — yerel kod yerel kalır.

## Claude Code için MCP kaydı

Varsayılan olarak yetenek "Claude Code'a gbrain için yazılı araç yüzeyi verilsin mi?" diye sorar. Evet derseniz, şu komutu çalıştırır:

```bash
claude mcp add gbrain -- gbrain serve
```

Bu, gbrain'in stdio MCP sunucusunu Claude Code'a kaydeder. Artık `gbrain search`, `gbrain put`, `gbrain get` vb. her oturumda birinci sınıf araçlar olarak görünür, kabuk dışı komutlar değil.

**`claude` PATH üzerinde değilse**, yetenek MCP kaydını manuel kayıt ipucuyla zarifçe atlar. CLI çözücüsü yine de gbrain'e kabuk dışı komut veren herhangi bir yetenekten çalışır — MCP bir yükseltmedir, önkoşul değil.

**Diğer yerel aracılar** (Cursor, Codex CLI, vb.) kendi MCP kayıtlarına ihtiyaç duyar. Yetenek v1 için Claude Code hedeflidir; diğer sunucular `gbrain serve` komutunu kendi MCP yapılandırmalarında manuel olarak kaydedebilir.

### Depo başına güven politikası (üçlü)

Makinenizdeki her depo bir politika kararına sahiptir: **okuma-yazma**, **salt-okunur** veya **reddet**.

- **okuma-yazma** — aracınız bu deponun bağlamından `gbrain search` yapabilir VE yeni sayfalar beyine yazabilir. Kendi projeleriniz için varsayılan.
- **salt-okunur** — aracınız beyni arayabilir ama bu deponun oturumlarından yeni sayfa yazamaz. Çok müşterili danışmanlar için ideal: paylaşılan beyni arayın, Müşteri A'nın kodunu Müşteri B'nin deposunda çalışırken beyine bulaştırmayın.
- **reddet** — hiçbir gbrain etkileşimi yok. Depo gbrain araçlarına görünmezdir.

Yetenek, bir gstack yeteneğini ilk kez çalıştırdığınızda depo başına bir kez sorar. Bundan sonra karar yapışkandır — aynı git uzak deposunun her çalışma ağacı + dalı aynı politikayı paylaşır, bu yüzden bir kez ayarlayın ve sizi takip eder.

SSH ve HTTPS uzak varyantları aynı anahtara çöküşür: `https://github.com/foo/bar.git` ve `git@github.com:foo/bar.git` aynı depodur.

**Politika değiştirmek için:**

```bash
/setup-gbrain --repo      # yalnızca bu depo için yeniden sor

# Veya doğrudan:
~/.claude/skills/gstack/bin/gstack-gbrain-repo-policy set "github.com/foo/bar" read-only
```

**Tüm politikaları görmek için:**

```bash
~/.claude/skills/gstack/bin/gstack-gbrain-repo-policy list
```

Depolama: `~/.gstack/gbrain-repo-policy.json`, mod 0600, şema sürümlüdür, bu nedenle gelecekteki geçişler deterministik kalır.

## Beyini `/sync-gbrain` ile güncel tutma

`/setup-gbrain` tek seferlik katılımdır. `/sync-gbrain`, gbrain'in bu deponun kodundaki değişiklikleri görmesini istediğiniz her seferinde çalıştıracağınız fiildir.

```bash
/sync-gbrain                # artımlı: mtime hızlı yolu, temiz bir ağaçta ~saniyeler
/sync-gbrain --full         # tam yeniden dizine ekleme (~25-35 dakika, büyük bir Mac'te)
/sync-gbrain --code-only    # yalnızca kod aşaması; bellek + beyin eşzamanlamasını atla
/sync-gbrain --dry-run      # ne eşzamanlanacağını önizle; yazma yok
```

Yetenek üç aşama çalıştırır — kod, bellek, beyin eşzamanlaması — bağımsız olarak. Biri başarısız olursa diğerlerini engellemez. Durum `~/.gstack/.gbrain-sync-state.json` dosyasında kalıcıdır, bu yüzden yeniden çalıştırmak temiz bir şekilde devam eder.

**Yeni bir çalışma ağacında ne yapar:**

1. **Ön kontrol.** `gbrain_local_status` kontrol eder (yerel motorun sağlığı). Motor `broken-db` veya `broken-config` ise, yetenek bir düzeltme menüsüyle DURUR — sessizce düşük kalitede çalışmayı reddeder. Yerel motor eksikse ve uzaktan MCP modundaysanız (Yol 4), kod aşaması temizçe ATLANIR ve yalnızca beyin eşzamanlaması çalışır.
2. **Kod aşaması.** Çalışma dizinini `gbrain sources add` ile birleştirilmiş kaynak olarak kaydeder, depo köküne bir `.gbrain-source` sabitleme dosyası yazar (kubectl tarzı bağlam — her çalışma ağacı kendi sabitlemesini alır, bu yüzden Conductor kardeş çalışma ağaçları çarpışmaz), `gbrain sync --strategy code` çalıştırır.
3. **Bellek aşaması.** `~/.gstack/` aktarımlarınızı + seçilmiş belleği hazırlar. Yerel-stdio MCP modunda, yerel motora alır. Uzaktan-http MCP modunda, hazırlanan markdown'ı uzaktan beyin yöneticisinin çekme hattı için `~/.gstack/transcripts/run-<pid>-<ts>/` konumunda saklar.
4. **Beyin eşzamanlama aşaması.** Seçilmiş artefaktları (planlar, tasarımlar, retrospektifler) yapılandırılmışsa özel artefakt deponuza gönderir.
5. **CLAUDE.md kılavuzu.** Gidiş-dönüşü yetenek denetimler (bir sayfa yaz → ara → bul). Yeşilse, projenizin CLAUDE.md'sine `## GBrain Search Guidance` bloğunu yazar. Kırmızıysa, bloğu KALDIRIR — araca asla kurulmamış bir araç kullanması söylenmemelidir.

**Filigran.** Eşzamanlama durumu işlem karması ile ilerler. gbrain dizine ekleyemediği bir dosyaya denk gelirse (dosya başına 5 MB sabit sınır veya dosya eşzamanlama sırasında kayboldu), filigran yerinde kalır ve sonraki eşzamanlamalar yeniden dener. Düzeltilemez bir başarısızlığı kabul edip geçmek için:

```bash
gbrain sync --source <source-id> --skip-failed
```

Yeniden çalıştırılabilir, etkisiz (idempotent), aynı makinede birden fazla terminalden çalıştırmak güvenlidir (`~/.gstack/.sync-gbrain.lock` dosyasında kilitlenir).

## Daha sonra motor değiştirme

PGLite seçtiniz ve artık bir takım beynine katılmak mı istiyorsunuz? Tek komut:

```bash
/setup-gbrain --switch
```

Yetenek `timeout 180s` ile sarılmış `gbrain migrate --to supabase --url "$URL"` çalıştırır. Geçiş çift yönlüdür (Supabase → PGLite de çalışır) ve kayıpsızdır — sayfalar, parçalar, gömmeler, bağlantılar, etiketler ve zaman çizelgesinin hepsi kopyalanır. Orijinal beyininiz yedek olarak korunur.

**Geçiş askıda kalırsa:** başka bir gstack oturumu kaynak beyinde bir kilit tutuyor olabilir. 3 dakikadan sonra zaman aşımı eyleme geçirilebilir bir mesajla tetiklenir. Diğer çalışma alanlarını kapatın ve yeniden çalıştırın.

## GStack bellek eşzamanlaması (ayrı bir konu)

Bu gbrain'in kendisinden farklıdır. gstack durumunuz (`~/.gstack/` — öğrenimler, planlar, retrospektifler, zaman çizelgesi, geliştirici profili) varsayılan olarak makineye yereldir. "GStack bellek eşzamanlaması", isteğe bağlı olarak seçilmiş, gizli taramalı bir alt kümeyi özel bir git deposuna gönderir, böylece belleğiniz makineler arası sizi takip eder — ve gbrain çalıştırıyorsanız, o git deposu da orada dizine eklenebilir hale gelir.

Şu şekilde etkinleştirin:

```bash
gstack-brain-init
```

Tek seferlik bir gizlilik istemi alırsınız: **her şey izin listesinde** / **yalnızca artefaktlar** (planlar, tasarımlar, retrospektifler, öğrenimler — zaman çizelgeleri gibi davranışsal verileri atla) / **kapalı**. Her yetenek çalışması kuyruğu başlangıç ve bitişte eşzamanlar — arka plan programı yok, arka plan süreci yok.

Gizli biçimli içerik (AWS anahtarları, GitHub token'ları, PEM blokları, JWT'ler, taşıyıcı token'lar) makinenizden ayrılmadan önce eşzamanlamadan engellenir.

**Yeni bir makinede:** `~/.gstack-brain-remote.txt` dosyasını kopyalayın, `gstack-brain-restore` çalıştırın ve dünün öğrenimleri bugünün dizüstü bilgisayarında yüzeye çıkar.

Tam kılavuz: [docs/gbrain-sync.md](docs/gbrain-sync.md). Hata dizini: [docs/gbrain-sync-errors.md](docs/gbrain-sync-errors.md).

`/setup-gbrain`, ilk kurulumun sonunda bunu bağlamayı önerir — bir AskUserQuestion daha ve aynı özel depo altyapısıyla tümleşir.

### Yetim projeleri temizleme

Hazırlama sırasında Ctrl-C yaptıysanız, karar vermeden önce üç farklı adım denediyseniz veya başka şekilde kullanmadığınız gbrain biçimli Supabase projeleri biriktirdiyseniz, bunun için bir alt komut vardır:

```bash
/setup-gbrain --cleanup-orphans
```

Yetenek bir PAT'yi yeniden toplar (tek seferlik, sonrasında çıkardı), Supabase hesabınızdaki adı `gbrain` ile başlayan ve referansı etkin `~/.gbrain/config.json` pooler URL'nizle eşleşmeyen her projeyi listeler. Her yetim proje için şunu sorar: *"Yetim proje `<ref>` silinsin mi (`<name>`, oluşturma `<date>`)?"* — toplu işlem yok, "tümünü sil" kısayolu yok. Etkin beyin asla silme için sunulmaz.

## Komut + bayrak referansı

### `/setup-gbrain` giriş modları

| Çağrı | Ne yapar |
|---|---|
| `/setup-gbrain` | Tam akış: durumu algıla, yol seç, kur, başlat, MCP, politika, isteğe bağlı bellek eşzamanlaması |
| `/setup-gbrain --repo` | Yalnızca geçerli depo için uzak başına güven politikasını değiştir |
| `/setup-gbrain --switch` | Motor geçişi (PGLite ↔ Supabase), diğer adımları yeniden çalıştırmadan |
| `/setup-gbrain --resume-provision <ref>` | Yoklama sırasında kesintiye uğrayan yol-2a otomatik hazırlamasını sürdür |
| `/setup-gbrain --cleanup-orphans` | Yetim Supabase projelerini listele + proje başına sil |

### Yardımcı komutlar (betikleme için)

| Komut | Amacı |
|---|---|
| `gstack-gbrain-detect` | Mevcut durumu JSON olarak yazdır: gbrain PATH üzerinde, sürüm, yapılandırma motoru, doctor durumu, eşzamanlama modu |
| `gstack-gbrain-install` | Algılama öncelikli kurucu (`~/git/gbrain`, `~/gbrain`, ardından taze klonu araştırır). `--dry-run` ve `--validate-only` bayrakları vardır. PATH gölgeleme kontrolü çıkış kodu 3 ile düzeltme menüsü gösterir. |
| `gstack-gbrain-lib.sh` | Kaynak olarak eklenir, çalıştırılmaz. `read_secret_to_env DEĞİŞKEN_ADI "istem" [--echo-redacted "<sed-ifadesi>"]` sağlar |
| `gstack-gbrain-supabase-verify` | Yapısal URL denetimi. Doğrudan bağlantı URL'lerini (`db.*.supabase.co:5432`) çıkış kodu 3 ile reddeder |
| `gstack-gbrain-supabase-provision` | Yönetim API sarmalayıcısı. Alt komutlar: `list-orgs`, `create`, `wait`, `pooler-url`, `list-orphans`, `delete-project`. Tümüü ortamda `SUPABASE_ACCESS_TOKEN` gerektirir. `create` ve `pooler-url` ayrıca `DB_PASS` gerektirir. Her alt komutta `--json` modu mevcuttur. |
| `gstack-gbrain-repo-policy` | Uzak başına güven üçlüsü. Alt komutlar: `get`, `set`, `list`, `normalize` |
| `gstack-gbrain-source-wireup` | `~/.gstack/` beyin deponuzu `gbrain sources add` + `git worktree` aracılığıyla gbrain'e birleştirilmiş kaynak olarak kaydeder, ardından başlangıç `gbrain sync` çalıştırır. Etkisiz (idempotent). v1.12.x'teki ölü `consumers.json + /ingest-repo` HTTP bağlantısının yerini alır. Bayraklar: `--strict`, `--source-id <id>`, `--no-pull`, `--uninstall`, `--probe`. |

### gbrain CLI (yukarı akış aracı)

Gbrain'in kendisi gstack'in sardığı şu komutları içerir:

| Komut | Amacı |
|---|---|
| `gbrain init --pglite` | Yerel bir PGLite beyini başlat |
| `gbrain init --non-interactive` | Ortam değişkenleriyle başlat (`GBRAIN_DATABASE_URL` veya `DATABASE_URL`). URL'yi argv olarak asla geçmeyin — kabuk geçmişine sızar. |
| `gbrain doctor --json` | Sağlık kontrolü. `{status: "ok"|"warnings"|"error", health_score: 0-100, checks: [...]}` döndürür |
| `gbrain migrate --to supabase --url ...` | PGLite beyinini Supabase'e taşı (kayıpsız, kaynağı yedek olarak korur) |
| `gbrain migrate --to pglite` | Ters geçiş |
| `gbrain search "sorgu"` | Beyinde ara |
| `gbrain put "<slug>" --content "<markdown-with-frontmatter>"` | Bir sayfa yaz (başlık/etiketler `--content` içindeki YAML ön maddesinde olur) |
| `gbrain get "<slug>"` | Bir sayfa getir |
| `gbrain serve` | MCP stdio sunucusunu başlat (`claude mcp add` tarafından kullanılır) |

### Yapılandırma dosyaları + durum

| Yol | Ne yaşar orada |
|---|---|
| `~/.gbrain/config.json` | Motor (pglite/postgres), veritabanı URL'si veya yolu, API anahtarları. Mod 0600. `gbrain init` tarafından yazılır. |
| `~/.gstack/gbrain-repo-policy.json` | Uzak başına güven üçlüsü. Şema v2. Mod 0600. |
| `~/.gstack/.setup-gbrain.lock.d` | Eşzamanlı çalıştırma kilidi (atomik mkdir). Normal çıkış + SIGINT üzerinde serbest bırakılır. |
| `~/.gstack/.brain-queue.jsonl` | gstack bellek eşzamanlaması için bekleyen eşzamanlama girdileri |
| `~/.gstack/.brain-last-push` | Son eşzamanlama gönderiminin zaman damgası (`/health` puanlaması için) |
| `~/.gstack-brain-remote.txt` | gstack bellek eşzamanlama uzak deponuzın URL'si (makineler arası kopyalama için güvenli) |
| `~/.gstack/.setup-gbrain-inflight.json` | Gelecekteki `--resume-provision` kalıcı durumu için ayrılmış |

### Ortam değişkenleri

| Değişken | Nerede okunur | Ne yapar |
|---|---|---|
| `SUPABASE_ACCESS_TOKEN` | `gstack-gbrain-supabase-provision` | Yönetim API çağrıları için PAT. Her kurulum çalıştırmasından sonra çıkardılır. |
| `DB_PASS` | `gstack-gbrain-supabase-provision` (create, pooler-url) | Oluşturulan veritabanı parolası. Asla argv'de değil. |
| `GBRAIN_DATABASE_URL` | `gbrain init`, `gbrain doctor`, vb. | Postgres bağlantı dizgisi (bizim için Supabase pooler URL'si). Ortam, `~/.gbrain/config.json` üzerinde önceliklidir. |
| `DATABASE_URL` | `gbrain init` (yedek) | `GBRAIN_DATABASE_URL` ile aynı anlambilim; ikinci olarak kontrol edilir. |
| `SUPABASE_API_BASE` | `gstack-gbrain-supabase-provision` | Yönetim API ana bilgisayarını geçersiz kılar. Testler tarafından sahte sunucuya yönlendirmek için kullanılır. |
| `GBRAIN_INSTALL_DIR` | `gstack-gbrain-install` | Varsayılan kurulum yolunu geçersiz kılar (`~/gbrain`) |
| `GSTACK_HOME` | her yardımcı komut | `~/.gstack` durum dizinini geçersiz kılar. Yoğun test kullanımı. |
| `VOYAGE_API_KEY` | `gbrain embed` alt süreci; gstack PGLite başlatması | Ayarlandığında, gstack PGLite'ı `voyage-code-3` (1024-boyut) ile başlatır, Voyage'in koda özel gömme modeli. Bu kod tabanının sembol sorgularında `voyage-4-large` ve OpenAI `text-embedding-3-large` modellerini bire bir geride bırakır. A/B rakamları için CHANGELOG v1.43.1.0'a bakın. |
| `OPENAI_API_KEY` | `gbrain embed` alt süreci | `VOYAGE_API_KEY` ayarlı değilken `gbrain sync` / `/sync-gbrain` sırasında gömmeler için kullanılır (gbrain'in otomatik seçtiği yedek, `text-embedding-3-large` 1536-boyut). Her iki anahtar da yoksa, sayfalar yapısal olarak içe aktarılır (sembol tabloları, parçalar) ancak anlamsal arama düşer — eşzamanlama günlüğünde `[gbrain] embedding failed for code file ...` satırları görürsünüz. |
| `ANTHROPIC_API_KEY` | `claude-agent-sdk`, ücretli değerlendirmeler | `bun run test:evals` ve Claude'a doğrudan `query()` çağrısı için gerekli. |
| `GSTACK_OPENAI_API_KEY` | `lib/conductor-env-shim.ts` | Conductor tarafından eklenen yedek. Kanonik ad boş olduğunda `OPENAI_API_KEY`'e yükseltilir. |
| `GSTACK_ANTHROPIC_API_KEY` | `lib/conductor-env-shim.ts` | Anthropic için yukarıdakiyle aynı kalıp. |

## Conductor + GSTACK_* ortam değişkenleri

gstack'i bir [Conductor](https://conductor.build) çalışma alanı içinde çalıştırıyorsanız, **Conductor açıkça `ANTHROPIC_API_KEY` ve `OPENAI_API_KEY`'i çalışma alanı ortamından çıkarır.** Bunları `~/.zshrc` veya `.env` dosyasında ayarlamak işe yaramaz — çıkarma ortam devralmasından sonra gerçekleşir. Çalışma alanına kullanılabilir bir API anahtarı getirmek için, bunları Conductor'un çalışma alanı ortam yapılandırmasında `GSTACK_ANTHROPIC_API_KEY` ve `GSTACK_OPENAI_API_KEY` olarak ayarlayın. Conductor bunları dokunulmadan geçirir.

`lib/conductor-env-shim.ts` gstack tarafında boşluğu köprüler: bir yan etki olarak içe aktarıldığında (`import "../lib/conductor-env-shim";`), kanonik adı görmeyen her alt süreç için `GSTACK_FOO_API_KEY`'yi `FOO_API_KEY`'e yükseltir. Shim şu dosyalara zaten bağlanmıştır:

- `bin/gstack-gbrain-sync.ts` — böylece `/sync-gbrain` gömmeler için OpenAI'yi alır
- `bin/gstack-model-benchmark` — böylece `--judge` çalışmaları manuel ortam eşlemesi olmadan çalışır
- `scripts/preflight-agent-sdk.ts` — böylece ücretli değerlendirme kimlik doğrulama yoklamaları çalışır
- `test/helpers/e2e-helpers.ts` — böylece `bun run test:evals` Anthropic'i bulur

Ücretli bir API'ye ulaşan veya gbrain gömmeleri gerektiren yeni bir TS giriş noktası eklerseniz, en üste aynı tek satırlık içe aktarmayı ekleyin. Katkıda bulunan kontrol listesi için [CONTRIBUTING.md "Conductor workspaces"](CONTRIBUTING.md#conductor-workspaces) bölümüne bakın.

`bin/gstack-codex-probe` bash tabanlıdır ve bunları doğrudan okumaz — Codex CLI tarafından yönetilen `~/.codex/` kimlik doğrulamasına güvenir.

## Güvenlik modeli

Bu yeteneğin dokunduğu her gizli bilgi için bir kural: **yalnızca ortam değişkeni, asla argv, asla günlüğe yazılmaz, asla bize tarafından diske yazılmaz.** Tek kalıcı depolama, gbrain'in kendi disiplini olan, mod 0600 olan `~/.gbrain/config.json` dosyasıdır; bizimki değil.

**Kodda zorunlu kılınan:**

- `test/skill-validation.test.ts` dosyasındaki CI grep testi, `$SUPABASE_ACCESS_TOKEN` veya `$GBRAIN_DATABASE_URL` bir argv konumunda görünürse yapılandırmayı başarısız kılar
- CI grep testi, `--insecure`, `-k` veya `NODE_TLS_REJECT_UNAUTHORIZED=0` `bin/gstack-gbrain-supabase-provision` dosyasında görünürse başarısız kılar
- Hazırlama yardımcısının en üstündeki `set +x`, hata ayıklama izlemelerinin PAT sızdırmasını önler
- Telemetri yükü yalnızca numaralandırılmış kategorik değerler içerir (senaryo, kurulum sonucu, MCP katılımı, güven katmanı) — asla gizli bilgi içerebilecek serbest biçimli dizgiler yok

**Testlerle zorunlu kılınan:**

- `test/secret-sink-harness.test.ts`, her gizli bilgi işleyen komutu ekilmiş bir gizli bilgiyle çalıştırır ve ekimin hiçbir yakalanan kanalda (stdout, stderr, `$HOME` altındaki dosyalar, telemetri JSONL) görünmediğini iddia eder. Her ekim için dört eşleşme kuralı: tam, URL-kodçözümlü, ilk-12-karakter öneki, base64.
- Aynı test dosyasındaki pozitif kontroller kasıtlı olarak her kapsanan kanalda ekim sızdırır ve donanımın her birini yakaladığını iddia eder. Pozitif kontroller olmadan, sessizce az raporlayan bir donanım, çalışan bir donanımdan ayırt edilemez olur.

**Hala sızdırabileceğiniz şeyler** (v1'in dürüst sınırları):

- Bir gizli bilgiyi `read -s` dışında normal bir sohbet mesajına yapıştırırsanız, sohbet transkriptinde ve herhangi bir sunucu tarafı günlüğünde olur
- Sızdırma donanımı alt süreç ortamını dökmez — `env >> ~/.log` çalıştıran bir komut algılamayı atlardı (v1'de böyle bir komut yok; grep testleri bunu önler)
- Kabuğunuzun kendi `HISTFILE` davranışı sizin kabuğunuzundur, bizim değil — gizli bilgileri argv'ye geçmediğimiz için onlara kodumuz aracılığıyla geçmezler, ancak ham bir `curl` komutuna yapıştırmanızı engelleyen bir şey yok

## Sorun giderme

### Kurulum sırasında "PATH SHADOWING DETECTED"

Başka bir `gbrain` çalıştırılabiliri kurucunun yeni bağladığından daha önce PATH üzerinde. Kurucunun sürüm kontrolü bunu yakaladı. Şunlardan birini düzeltin:

- Diğerine ihtiyacınız yoksa `rm $(which gbrain)`
- Bağlanan çalıştırılabilir dosyanın kazanması için kabuk rc'nizde `~/.bun/bin`'i PATH'e ekleyin
- Gölgeleyen çalıştırılabilir dosyanın kurulum dizinine `GBRAIN_INSTALL_DIR` ayarlayın ve yeniden çalıştırın

Ardından `/setup-gbrain` komutunu yeniden çalıştırın.

### "rejected direct-connection URL"

Bir `db.<ref>.supabase.co:5432` URL'si yapıştırdınız. Bunlar yalnızca IPv6'dır ve çoğu ortamda başarısız olur. Bunun yerine Session Pooler URL'sini kullanın: Supabase panosu → Settings → Database → Connection Pooler → **Session** → URI kopyala (port 6543).

### Otomatik hazırlama 180 s'de zaman aşımına uğruyor

Supabase projesi hâlâ başlatılıyor. Referansınız çıkış mesajında yazdırıldı. Bir dakika bekleyin, ardından:

```bash
/setup-gbrain --resume-provision <ref>
```

Yetenek bir PAT'yi yeniden toplar, proje oluşturmayı atlar, yoklamayı sürdürür.

### "Başka bir `/setup-gbrain` örneği çalışıyor"

Eski bir kilit dizininiz var. Başka bir örneğin gerçekten çalışmadığından eminseniz:

```bash
rm -rf ~/.gstack/.setup-gbrain.lock.d
```

Ardından yeniden çalıştırın.

### Politika dosyasında "No crossmodel tension"

`~/.gstack/gbrain-repo-policy.json` dosyasını eski `allow` değerleriyle elle mi düzenlediniz? Sorun değil. Sonraki okumada, gstack `allow` → `read-write` dönüşümünü otomatik olarak yapar ve `_schema_version: 2` ekler. Stderr'de bir günlük satırı, etkisiz (idempotent), deterministik.

### `gbrain doctor` "warnings" diyor

`/health` bunu kırmızı değil sarı olarak değerlendirir. Hangi alt kontrollerin uyarı verdiğini görmek için `gbrain doctor --json | jq .checks` çalıştırın. Tipik nedenler: çözücü MECE örtüşmesi (yetenek adları çakışması) veya veritabanı bağlantısının henüz yapılandırılmamış olması.

### `/sync-gbrain` `OK` raporluyor ama `gbrain search` anlamsal hiçbir şey döndürmüyor

İçe aktarma sırasında gömmeler muhtemelen başarısız oldu. Sembol sorguları (`code-def`, `code-refs`) gömme gerektirmediği için hâlâ çalışır, ancak `gbrain search "<terimler>"` düşürülmüş bir BM25 yoluna geri döner. Eşzamanlama çıktısında şu satırları arayın:

```
[gbrain] embedding failed for code file <name>: OpenAI embedding requires OPENAI_API_KEY
```

Çözüm, yeniden çalıştırmadan önce süreç ortamına bir sağlayıcı API anahtarı koymaktır. Kod için `VOYAGE_API_KEY` tercih edilir (gstack ayarlandığında PGLate varsayılan olarak `voyage-code-3` kullanır); aksi takdirde `OPENAI_API_KEY` `text-embedding-3-large` seçeneğine geri döner. Çıplak bir Mac kabuğunda, çağırmadan önce anahtarı `~/.zshrc` dosyasından kaynaklayın. Conductor'da, `lib/conductor-env-shim.ts` shim'i `GSTACK_ANTHROPIC_API_KEY` / `GSTACK_OPENAI_API_KEY`'yi otomatik olarak kanonik adlarına yükseltir; `VOYAGE_API_KEY` için doğrudan Conductor çalışma alanı ortamınızda ayarlayın. Zaten içe aktarılmış sayfalarda gömmeleri geri doldurmak için `/sync-gbrain --code-only` yeniden çalıştırın.

### `gbrain sync` bir işlem karmasında bloklandı — `FILE_TOO_LARGE`

Deponuzdaki bir dosya gbrain'in 5 MB sabit sınırını aşıyor (`gbrain/src/core/import-file.ts` dosyasındaki `MAX_FILE_SIZE`). Yaygın suçlular: yanıt yeniden oynatma önbellekleri, yakalanan ekran görüntüleri, büyük JSON örnek dosyaları. gbrain kod eşzamanlaması için `.gitignore` tarzı hariç tutme listelerine saygı göstermez; tek düğme başarısızlığı kabul etmektir:

```bash
gbrain sync --source <source-id> --skip-failed
```

Filigran sorunlu işlemi geçer. Aynı dosya değişirse tekrar başarısız olur; bu olduğunda yeniden atlayın.

### PGLite → Supabase geçişi askıda kalıyor

Kardeş bir Conductor çalışma alanındaki başka bir gstack oturumu, yerel PGLite dosyanızda çalıştırma önbelleği `gstack-brain-sync` çağrısı aracılığıyla bir kilit tutuyor olabilir. Diğer çalışma alanlarını kapatın, `/setup-gbrain --switch` yeniden çalıştırın. Zaman aşımı 180 s ile sınırlıdır, bu yüzden sonsuza kadar beklemeyeceksiniz.

## Neden bu tasarım

**Neden uzak başına güven üçlüsü ve ikili izin/reddet değil?** Çok müşterili danışmanlar geri yazma olmadan arama gerektirir. Sabah Müşteri A üzerinde, öğleden sonra Müşteri B üzerinde çalışan serbest bir geliştirici, A'nın kod içgörülerinin Müşteri B'nin arayabileceği bir beyne sızmasına izin veremez. Salt-okunur bunu temiz bir şekilde çözer.

**Neden gbrain'i gstack'e paketlemek yerine ayrı tutmak?** Gbrain kendi sürüm döngüsü, şema geçişleri ve MCP yüzeyi olan ayrı, aktif olarak geliştirilen bir projedir. Paketlemek, gstack'in gbrain güncellemelerini geçit olarak kullanması gerektirir, bu da gbrain iyileştirmelerinin kullanıcılara ulaşmasını yavaşlatır. Ayrı-ama-tümleşik, her birinin kendi döngüsünde göndermesine izin verir.

**Neden bayrak yerine ortam değişkeni ile `gbrain init --non-interactive`?** Bağlantı dizgileri veritabanı parolaları içerir. Bunları argv olarak geçmek, parolayı `ps`, kabuk geçmişi ve süreç listelerine yerleştirir. Ortam değişkeni aktarımı gizli bilgiyi yalnızca süreç belleğinde tutar. gbrain hem `GBRAIN_DATABASE_URL` hem de `DATABASE_URL` destekler; çarpışmaları önlemek için önceki kullanırız.

**Neden uyar-and-devam yerine PATH gölgelemesinde sıkı-başarısız?** Gölgelenen bir `gbrain`, sonraki her komutun yeni kurduğumuzdan farklı bir çalıştırılabilir dosyayı çağırması anlamına gelir. Bu, haftalar sonra gizemli özellik boşlukları olarak ortaya çıkan sessiz bir sürüm-kayma hatasıdır. Kurulum yeteneklerinin bir görevi vardır — çalışan bir ortam kurmak. Bozuk bir ortama kurulum yapmayı reddetmek, kurulum yeteneği açısından doğru davranıştır.

**Neden her depoyu otomatik olarak içe aktarmak yerine?** Gizlilik + gürültü. Dokunduğunuz her depoyu içe aktaran bir otomatik içe aktarma önbelleği girişi: (a) izin olmadan iş kodunu paylaşılan bir beyne sızdırır ve (b) geçici depolarla aramayı tıkar. Uzak başına politika, içe aktarmayı açık, depo başına bir karar yapar. `/setup-gbrain` bugün herhangi bir otomatik içe aktarma girişi kurmaz — ancak politika deposu daha sonra biri için ileriye uyumludur.

## İlgili yetenekler + sonraki adımlar

- `/health` — 0-10 bileşik puanında bir GBrain boyutu içerir (doctor durumu, eşzamanlama kuyruk derinliği, son gönderim yaşı). gbrain kurulmadığında boyut atlanır; gbrain olmayan bir makinede `/health` çalıştırmak bu seçimi cezalandırmaz.
- `/gstack-upgrade` — gstack'i güncel tutar. gbrain'i bağımsız olarak yükseltmez. gbrain'i güncellemek için `bin/gstack-gbrain-install` dosyasında `PINNED_COMMIT`'i güncelleyin ve `/setup-gbrain` yeniden çalıştırın.
- `/retro` — haftalık retrospektif, bellek eşzamanlaması açıkken gbrain'inizden öğrenimleri ve planları çeker, retrospektifin makineler arası geçmişe başvurmasına olanak tanır.

`/setup-gbrain` çalıştırın ve neyin işe yaradığını görün.