---
name: gstack-upgrade
version: 1.1.0
description: |
  gstack'i en son sürüme yükseltir. Global ve vendored kurulumu algılar,
  yükseltmeyi çalıştırır ve yenilikleri gösterir. "gstack'i yükselt",
  "gstack'i güncelle" veya "en son sürümü al" isteklerinde kullanılır.
  Ses tetikleyicileri (konuşmadan metne takma adlar): "araçları yükselt", "araçları güncelle", "gee stack yükselt", "g stack yükselt".
triggers:
  - gstack yükselt
  - gstack sürümünü güncelle
  - en son gstack'i al
allowed-tools:
  - Bash
  - Read
  - Write
  - AskUserQuestion
---
<!-- AUTO-GENERATED from SKILL.md.tmpl — doğrudan düzenlemeyin -->
<!-- Yeniden oluştur: bun run gen:skill-docs -->

# /gstack-upgrade

gstack'i en son sürüme yükselt ve yenilikleri göster.

## Satır içi yükseltme akışı

Bu bölüm, tüm skill önsözleri tarafından `UPGRADE_AVAILABLE` algıladıklarında referans alınır.

### Adım 1: Kullanıcıya sor (veya otomatik yükselt)

Önce otomatik yükseltmenin etkin olup olmadığını kontrol edin:
```bash
_AUTO=""
[ "${GSTACK_AUTO_UPGRADE:-}" = "1" ] && _AUTO="true"
[ -z "$_AUTO" ] && _AUTO=$(~/.claude/skills/gstack/bin/gstack-config get auto_upgrade 2>/dev/null || true)
echo "AUTO_UPGRADE=$_AUTO"
```

**Eğer `AUTO_UPGRADE=true` veya `AUTO_UPGRADE=1`:** AskUserQuestion'ı atlayın. "gstack v{old} → v{new} otomatik olarak yükseltiliyor..." loglayın ve doğrudan Adım 2'ye geçin. Otomatik yükseltme sırasında `./setup` başarısız olursa, yedekten (`.bak` dizini) geri yükleyin ve kullanıcıyı uyarın: "Otomatik yükseltme başarısız oldu — önceki sürüm geri yüklendi. Tekrar denemek için `/gstack-upgrade` komutunu manuel olarak çalıştırın."

**Aksi takdirde**, AskUserQuestion kullanın:
- Soru: "gstack **v{new}** mevcut (v{old} sürümündesiniz). Şimdi yükseltmek ister misiniz?"
- Seçenekler: ["Evet, şimdi yükselt", "Beni her zaman güncel tut", "Şimdi değil", "Bir daha sorma"]

**"Evet, şimdi yükselt" seçilirse:** Adım 2'ye geçin.

**"Beni her zaman güncel tut" seçilirse:**
```bash
~/.claude/skills/gstack/bin/gstack-config set auto_upgrade true
```
Kullanıcıya şunu söyleyin: "Otomatik yükseltme etkinleştirildi. Gelecekteki güncellemeler otomatik olarak kurulacak." Ardından Adım 2'ye geçin.

**"Şimdi değil" seçilirse:** Artan geri bildirimle snooze durumu yazın (ilk snooze = 24 saat, ikinci = 48 saat, üçüncü ve sonrası = 1 hafta), ardından mevcut skill ile devam edin. Yükseltmeyi tekrar belirtmeyin.
```bash
_SNOOZE_FILE="$HOME/.gstack/update-snoozed"
_REMOTE_VER="{new}"
_CUR_LEVEL=0
if [ -f "$_SNOOZE_FILE" ]; then
  _SNOOZED_VER=$(awk '{print $1}' "$_SNOOZE_FILE")
  if [ "$_SNOOZED_VER" = "$_REMOTE_VER" ]; then
    _CUR_LEVEL=$(awk '{print $2}' "$_SNOOZE_FILE")
    case "$_CUR_LEVEL" in *[!0-9]*) _CUR_LEVEL=0 ;; esac
  fi
fi
_NEW_LEVEL=$((_CUR_LEVEL + 1))
[ "$_NEW_LEVEL" -gt 3 ] && _NEW_LEVEL=3
echo "$_REMOTE_VER $_NEW_LEVEL $(date +%s)" > "$_SNOOZE_FILE"
```
Not: `{new}`, `UPGRADE_AVAILABLE` çıktısındaki uzak sürümdür — güncelleme kontrol sonucundan değiştirin.

Kullanıcıya snooze süresini söyleyin: "Sonraki hatırlatma 24 saat sonra" (veya seviyeye göre 48 saat veya 1 hafta). İpucu: "Otomatik yükseltmeler için `~/.gstack/config.yaml` dosyasında `auto_upgrade: true` ayarlayın."

**"Bir daha sorma" seçilirse:**
```bash
~/.claude/skills/gstack/bin/gstack-config set update_check false
```
Kullanıcıya şunu söyleyin: "Güncelleme kontrolleri devre dışı bırakıldı. Yeniden etkinleştirmek için `~/.claude/skills/gstack/bin/gstack-config set update_check true` çalıştırın."
Mevcut skill ile devam edin.

### Adım 2: Kurulum türünü algıla

```bash
if [ -d "$HOME/.claude/skills/gstack/.git" ]; then
  INSTALL_TYPE="global-git"
  INSTALL_DIR="$HOME/.claude/skills/gstack"
elif [ -d "$HOME/.gstack/repos/gstack/.git" ]; then
  INSTALL_TYPE="global-git"
  INSTALL_DIR="$HOME/.gstack/repos/gstack"
elif [ -d ".claude/skills/gstack/.git" ]; then
  INSTALL_TYPE="local-git"
  INSTALL_DIR=".claude/skills/gstack"
elif [ -d ".agents/skills/gstack/.git" ]; then
  INSTALL_TYPE="local-git"
  INSTALL_DIR=".agents/skills/gstack"
elif [ -d ".claude/skills/gstack" ]; then
  INSTALL_TYPE="vendored"
  INSTALL_DIR=".claude/skills/gstack"
elif [ -d "$HOME/.claude/skills/gstack" ]; then
  INSTALL_TYPE="vendored-global"
  INSTALL_DIR="$HOME/.claude/skills/gstack"
else
  echo "ERROR: gstack bulunamadı"
  exit 1
fi
echo "Install type: $INSTALL_TYPE at $INSTALL_DIR"
```

Yukarıda yazdırılan kurulum türü ve dizin yolu sonraki tüm adımlarda kullanılacaktır.

### Adım 3: Eski sürümü kaydet

Adım 2'nin çıktısındaki kurulum dizinini kullanın:

```bash
OLD_VERSION=$(cat "$INSTALL_DIR/VERSION" 2>/dev/null || echo "unknown")
```

### Adım 4: Yükselt

Adım 2'de algılanan kurulum türü ve dizini kullanın:

**Git kurulumları için** (global-git, local-git):
```bash
cd "$INSTALL_DIR"
STASH_OUTPUT=$(git stash 2>&1)
git fetch origin
git reset --hard origin/main
./setup
```
Eğer `$STASH_OUTPUT` "Saved working directory" içeriyorsa, kullanıcıyı uyarın: "Not: yerel değişiklikler stash edildi. Geri yüklemek için skill dizininde `git stash pop` çalıştırın."

**Vendored kurulumlar için** (vendored, vendored-global):
```bash
PARENT=$(dirname "$INSTALL_DIR")
TMP_DIR=$(mktemp -d)
git clone --depth 1 https://github.com/garrytan/gstack.git "$TMP_DIR/gstack"
mv "$INSTALL_DIR" "$INSTALL_DIR.bak"
mv "$TMP_DIR/gstack" "$INSTALL_DIR"
cd "$INSTALL_DIR" && ./setup
rm -rf "$INSTALL_DIR.bak" "$TMP_DIR"
```

### Adım 4.5: Yerel vendored kopyayı işle

Adım 2'deki kurulum dizinini kullanın. Ayrıca yerel bir vendored kopya olup olmadığını ve team modunun etkin olup olmadığını kontrol edin:

```bash
_ROOT=$(git rev-parse --show-toplevel 2>/dev/null)
LOCAL_GSTACK=""
if [ -n "$_ROOT" ] && [ -d "$_ROOT/.claude/skills/gstack" ]; then
  _RESOLVED_LOCAL=$(cd "$_ROOT/.claude/skills/gstack" && pwd -P)
  _RESOLVED_PRIMARY=$(cd "$INSTALL_DIR" && pwd -P)
  if [ "$_RESOLVED_LOCAL" != "$_RESOLVED_PRIMARY" ]; then
    LOCAL_GSTACK="$_ROOT/.claude/skills/gstack"
  fi
fi
_TEAM_MODE=$(~/.claude/skills/gstack/bin/gstack-config get team_mode 2>/dev/null || echo "false")
echo "LOCAL_GSTACK=$LOCAL_GSTACK"
echo "TEAM_MODE=$_TEAM_MODE"
```

**Eğer `LOCAL_GSTACK` boş değilse VE `TEAM_MODE` `true` ise:** Vendored kopyayı kaldırın. Team modu, tek bilgi kaynağı olarak global kurulumu kullanır.

```bash
cd "$_ROOT"
git rm -r --cached .claude/skills/gstack/ 2>/dev/null || true
if ! grep -qF '.claude/skills/gstack/' .gitignore 2>/dev/null; then
  echo '.claude/skills/gstack/' >> .gitignore
fi
rm -rf "$LOCAL_GSTACK"
```
Kullanıcıya şunu söyleyin: "`$LOCAL_GSTACK` konumundaki vendored kopya kaldırıldı (team modu aktif — global kurulum bilgi kaynağıdır). `.gitignore` değişikliğini hazır olduğunuzda commit edin."

**Eğer `LOCAL_GSTACK` boş değilse VE `TEAM_MODE` `true` DEĞİLSE:** Yeni yükseltilmiş birincil kurulumdan kopyalayarak güncelleyin (README vendored kurulumuyla aynı yaklaşım):
```bash
mv "$LOCAL_GSTACK" "$LOCAL_GSTACK.bak"
cp -Rf "$INSTALL_DIR" "$LOCAL_GSTACK"
rm -rf "$LOCAL_GSTACK/.git"
cd "$LOCAL_GSTACK" && ./setup
rm -rf "$LOCAL_GSTACK.bak"
```
Kullanıcıya şunu söyleyin: "`$LOCAL_GSTACK` konumundaki vendored kopya da güncellendi — hazır olduğunuzda `.claude/skills/gstack/` dizinini commit edin."

Eğer `./setup` başarısız olursa, yedekten geri yükleyin ve kullanıcıyı uyarın:
```bash
rm -rf "$LOCAL_GSTACK"
mv "$LOCAL_GSTACK.bak" "$LOCAL_GSTACK"
```
Kullanıcıya şunu söyleyin: "Eşitleme başarısız oldu — `$LOCAL_GSTACK` konumundaki önceki sürüm geri yüklendi. Tekrar denemek için `/gstack-upgrade` komutunu manuel olarak çalıştırın."

### Adım 4.75: Sürüm geçişlerini çalıştır

`./setup` tamamlandıktan sonra, eski ve yeni sürüm arasındaki geçiş betiklerini çalıştırın. Geçişler, `./setup`'ın tek başına kapsayamayacağı durum düzeltmelerini (eski yapılandırma, yetim dosyalar, dizin yapısı değişiklikleri) işler.

```bash
MIGRATIONS_DIR="$INSTALL_DIR/gstack-upgrade/migrations"
if [ -d "$MIGRATIONS_DIR" ]; then
  for migration in $(find "$MIGRATIONS_DIR" -maxdepth 1 -name 'v*.sh' -type f 2>/dev/null | sort -V); do
    # Dosya adından sürümü çıkar: v0.15.2.0.sh → 0.15.2.0
    m_ver="$(basename "$migration" .sh | sed 's/^v//')"
    # Bu geçiş sürümü eski sürümden yenise çalıştır
    # (Aynı segment sayısına sahip noktalı sürümler için basit dize karşılaştırması çalışır)
    if [ "$OLD_VERSION" != "unknown" ] && [ "$(printf '%s\n%s' "$OLD_VERSION" "$m_ver" | sort -V | head -1)" = "$OLD_VERSION" ] && [ "$OLD_VERSION" != "$m_ver" ]; then
      echo "Running migration $m_ver..."
      bash "$migration" || echo "  Warning: migration $m_ver had errors (non-fatal)"
    fi
  done
fi
```

Geçişler, `gstack-upgrade/migrations/` dizinindeki idempotent bash betikleridir. Her biri `v{SÜRÜM}.sh` olarak adlandırılır ve yalnızca daha eski bir sürümden yükseltme yapılırken çalışır. Yeni geçişler eklemek için CONTRIBUTING.md sayfasına bakın.

### Adım 5: İşaretleyici yaz + önbelleği temizle

```bash
mkdir -p ~/.gstack
echo "$OLD_VERSION" > ~/.gstack/just-upgraded-from
rm -f ~/.gstack/last-update-check
rm -f ~/.gstack/update-snoozed
```

### Adım 6: Yenilikleri göster

`$INSTALL_DIR/CHANGELOG.md` dosyasını okuyun. Eski sürüm ve yeni sürüm arasındaki tüm sürüm girdilerini bulun. Tema tarafından gruplandırılmış 5-7 madde olarak özetleyin. Fazla detaya girmeyin — kullanıcıya yönelik değişikliklere odaklanın. Önemsiz iç yeniden yapılandırmaları atlayın (çok önemli değillerse).

Format:
```
gstack v{new} — v{old} sürümünden yükseltildi!

Yenilikler:
- [madde 1]
- [madde 2]
- ...

Mutlu göndermeler!
```

### Adım 7: Devam et

Yenilikleri gösterdikten sonra, kullanıcının özgün olarak çağırdığı skill ile devam edin. Yükseltme tamamlandı — başka bir işlem gerekmez.

---

## Bağımsız kullanım

Doğrudan `/gstack-upgrade` olarak çağrıldığında (bir önsözden değil):

1. Yeni bir güncelleme kontrolü zorla (önbelleği atla):
```bash
~/.claude/skills/gstack/bin/gstack-update-check --force 2>/dev/null || \
.claude/skills/gstack/bin/gstack-update-check --force 2>/dev/null || true
```
Bir yükseltme olup olmadığını belirlemek için çıktıyı kullanın.

2. Eğer `UPGRADE_AVAILABLE <old> <new>` varsa: yukarıdaki Adım 2-6'yı izleyin.

3. Çıktı yoksa (birincil güncel ise): eski bir yerel vendored kopya olup olmadığını kontrol edin.

Yukarıdaki Adım 2 bash bloğunu çalıştırarak birincil kurulum türünü ve dizinini (`INSTALL_TYPE` ve `INSTALL_DIR`) algılayın. Ardından yukarıdaki Adım 4.5 algılama bash bloğunu çalıştırarak yerel vendored kopya (`LOCAL_GSTACK`) ve team modu durumunu (`TEAM_MODE`) kontrol edin.

**Eğer `LOCAL_GSTACK` boşsa** (yerel vendored kopya yok): kullanıcıya "Zaten en son sürümdesiniz (v{version})." deyin.

**Eğer `LOCAL_GSTACK` boş değilse VE `TEAM_MODE` `true` ise:** Yukarıdaki Adım 4.5 team-modu kaldırma bash bloğunu kullanarak vendored kopyayı kaldırın. Kullanıcıya şunu söyleyin: "Global v{version} güncel. Eski vendored kopya kaldırıldı (team modu aktif). `.gitignore` değişikliğini hazır olduğunuzda commit edin."

**Eğer `LOCAL_GSTACK` boş değilse VE `TEAM_MODE` `true` DEĞİLSE**, sürümleri karşılaştırın:
```bash
PRIMARY_VER=$(cat "$INSTALL_DIR/VERSION" 2>/dev/null || echo "unknown")
LOCAL_VER=$(cat "$LOCAL_GSTACK/VERSION" 2>/dev/null || echo "unknown")
echo "PRIMARY=$PRIMARY_VER LOCAL=$LOCAL_VER"
```

**Sürümler farklıysa:** Yerel kopyayı birincilden güncellemek için yukarıdaki Adım 4.5 eşitleme bash bloğunu izleyin. Kullanıcıya şunu söyleyin: "Global v{PRIMARY_VER} güncel. Yerel vendored kopya v{LOCAL_VER} → v{PRIMARY_VER} olarak güncellendi. Hazır olduğunuzda `.claude/skills/gstack/` dizinini commit edin."

**Sürümler aynıysa:** kullanıcıya "En son sürümdesiniz (v{PRIMARY_VER}). Global ve yerel vendored kopyanın ikisi de güncel." deyin.