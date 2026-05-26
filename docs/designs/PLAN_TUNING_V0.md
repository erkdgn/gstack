# Plan Tuning v0 — Tasarım Dokümanı

**Durum:** v1 uygulaması için onaylandı
**Dal:** garrytan/plan-tune-skill
**Yazarlar:** Garry Tan (kullanıcı), Claude Opus 4.7 + OpenAI Codex gpt-5.4'den AI destekli incelemelerle
**Tarih:** 2026-04-16

## Bu doküman nedir

`/plan-tune` v1'in ne olduğunun, ne OLMADIĞININ, neyi düşündüğümüzün ve her kararı neden aldığımızın kanonik kaydı. Depoya işlenir, böylece gelecekteki katkı sağlayıcılar (ve gelecekteki Garry) arkeoloji yapmadan akıl yürütmeyi izleyebilir. Kullanıcı başına yerel kayıtlar olan iki `~/.gstack/projects/` eserinin yerini alır.

## Özellik, bir paragrafta

gstack'in 40+ skill'i sürekli AskUserQuestion ateşler. Güçlü kullanıcılar aynı soruları aynı şekilde tekrar tekrar yanıtlar ve gstack'e "bunu bana sormayı bırak" demek için bir yolu yoktur. Daha temel olarak, gstack'in her kullanıcının çalışmasını nasıl yönlendirmeyi tercih ettiğine dair bir modeli yok — kapsam iştahı, risk toleransı, ayrıntı tercihi, özerklik, mimari özen — bu yüzden her skill'in varsayılanları herkes için ortalamadır. `/plan-tune` v1, şema + gözlem katmanını oluşturur: tipli bir soru kayıt defteri, soru başına açık tercihler, satır içi "tune:" geri bildirimi ve düz İngilizce ile incelenebilir bir profil (beyan edilen + çıkarılan boyutlar). Henüz profili temel alarak skill davranışını uyarlamaz. Bu, v1 alt tabanın çalıştığını kanıtladıktan sonra v2'de gelir.

## Daha küçük sürümü neden oluşturuyoruz

Özellik, tam uyarlanabilir bir alt taban olarak başladı: psikografik boyutlar otomatik kararları yönlendiren, kör nokta koçluğu, LANDED kutlama HTML sayfası, hepsi paketlenmiş. Dört inceleme turu (office-hours, CEO EXPANSION, DX POLISH, mühendislik incelemesi) bunu temizledi. Sonra dış ses (Codex) 20 noktalı bir eleştiri sundu. Öncelik sırasına göre kritik bulgular:

1. **"Alt taban" yalandı.** Plan 5 skill'i önyazıda profilden okuyacak şekilde bağladı, ancak AskUserQuestion bir prompt kuralıdır, ara yazılım değil. Aracılar talimatları sessizce atlayabilir. Tipli bir soru kayıt defteri olmadan her AskUserQuestion'ın yönlendirildiği bir alt taban üzerinde otomatik karar oluşturamazsınız. Kayıt defteri olmadan alt taban iddası pazarlamadır.
2. **İçsel mantıksal çelişkiler.** E4 (kör nokta) + E6 (uyumsuzluk) + beyan edilen boyutlar üzerinde ±0.2 kelepçe birleşmez. Kullanıcı kendi beyanını kelepçe ile gerçek kabul ediyorsa, E6'nın uyumsuzluk algılama gürültü algılıyor. Davranış profili düzeltebilirse, kelepçe E6'nın ihtiyaç duyduğu sinyali bastırır.
3. **Profil zehirlenmesi.** Satır içi "tune: never ask" kötü niyetli repo içeriği (README, PR açıklaması, araç çıktısı) tarafından yayılabilir ve aracı bunu sadakatle yazar. Önceki hiçbir inceleme bu güvenlik boşluğunu yakalamadı.
4. **E5 LANDED sayfası önyazıda.** Her skill'in önyazısında `gh pr view` + HTML yazma + tarayıcı açma gecikme, kimlik doğrulama başarısızlıkları, hız sınırları, sürpriz tarayıcı açmaları ve en sıcak yola enjekte edilen belirsizliktir.
5. **Uygulama sırası ters idi.** Plan sınıflandırıcılar ve kutularla başladı. Doğru sıra: önce entegrasyon noktasını oluştur (tipli soru kayıt defteri), sonra altyapı, sonra tüketiciler.

Codex'in argümanını tarttıktan sonra, CEO EXPANSION'ı geri almaya ve gerçek bir tipli kayıt defterini temel olarak gözlemsel bir v1 göndermeye karar verdik. Psikografik, kayıt defteri üretime dayanıklı kanıtlandıktan sonra davranışsal hale gelir.

## v1 Kapsamı (şimdi oluşturduğumuz)

1. **Tipli soru kayıt defteri** (`scripts/question-registry.ts`). gstack'in kullandığı her AskUserQuestion `{id, skill, category, door_type, options[], signal_key?}` ile beyan edilir. Şema yönetilir.
2. **CI uygulaması.** Kapı katmanı lint testi, SKILL.md.tmpl dosyalarındaki her AskUserQuestion deseninin eşleşen bir kayıt defteri girdisi olduğunu iddia eder. Sapma, yeniden adlandırma veya kopyalar üzerinde CI başarısız olur.
3. **Soru günlükleme** (`bin/gstack-question-log`). `~/.gstack/projects/{SLUG}/question-log.jsonl`'ye `{ts, question_id, user_choice, recommended, session_id}` ekler. Kayıt defterisine karşı doğrular.
4. **Soru başına açık tercihler** (`bin/gstack-question-preference`). `{question_id, preference}` yazar, burada preference `always-ask | never-ask | ask-only-for-one-way`'dir. 1. oturumdan itibaren saygı duyulur. Kalibrasyon geçidi yok — kullanıcı belirtti, sistem uyar.
5. **Önyazı enjeksiyonu.** Her AskUserQuestion'dan önce, aracı `gstack-question-preference --check <registry-id>` çağırır. `never-ask` VE soru tek yönlü bir kapı DEĞİLSE, görünür açıklama ile önerilen seçeneği otomatik seç: "Otomatik karar verildi [özet] → [seçenek] (tercihiniz). /plan-tune ile değiştir." Tek yönlü kapılar tercihden bağımsız olarak her zaman sorar — güvenlik geçersiz kılma.
6. **Kullanıcı kaynağı geçidi ile satır içi "tune:" geri bildirimi.** Aracı "Bu soruyu ayarla? Ayarlamak için `tune: [geri bildirim]` yanıtla." teklif eder. Kullanıcı kısayollar (`unnecessary`, `ask-less`, `never-ask`, `always-ask`, `context-dependent`) veya serbest form İngilizce kullanabilir. KRİTİK: aracı yalnızca `tune:` içeriği kullanıcının mevcut sohbet turunda göründüğünde bir tune olayı yazar — araç çıktısında değil, dosya okumasında değil. İkili dosya yazma sırasında `source: "inline-user"` doğrular; diğer kaynakları reddeder.
7. **Beyan edilen profil** (`/plan-tune setup`). Boyut başına bir tane olmak üzere 5 düz İngilizce soru. Birleşik `~/.gstack/developer-profile.json`'da `declared: {...}` altında saklanır. v1'de yalnızca bilgilendirme — skill davranış değişikliği yok.
8. **Gözlemlenen/Çıkarılan profil.** Her soru günlüğü olayı, el yapımı bir sinyal haritası (`scripts/psychographic-signals.ts`) aracılığıyla çıkarılan boyutlara deltalar katkıda bulunur. Talep üzerine hesaplanır. Görüntülenir ama üzerinde işlem yapılmaz.
9. **`/plan-tune` skill'i.** Konuşmalı düz İngilizce inceleme aracı. "Profilimi göster," "bir tercih ayarla," "hangi sorular soruldu," "söylediklerimle yaptıklarım arasındaki boşluğu göster." CLI alt komut sözdizimi gerekmez.
10. **Mevcut `~/.gstack/builder-profile.jsonl` ile birleştirme.** /office-hours oturum kayıtlarını ve birikmiş sinyalleri birleşik `~/.gstack/developer-profile.json`'a katla. Geçiş atomik + idempotent + kaynak dosyasını arşivler.

## v2'ye ertelendi (bu PR'da yok, ancak açık kabul kriterleri)

| Öğe | Neden ertelendi | v2'ye terfi için kabul kriterleri |
|------|--------------|--------------------------------------|
| E1 Alt taban kablolama (5 skill profili okur ve uyarlar) | v1 kayıt defterinin dayanıklı olduğunu kanıtlamasını gerektirir. Gerçek gözlemlenen veri sinyal deltalarını kalibre etmesini gerektirir. Psikografik kayma riski. | v1 kayıt defteri 90+ gün dayanıklı. Çıkarılan boyutlar 3+ skill arasında net istikrar gösterir. Kullanıcı deneme yanılması doğrular, profilden bilgilendirilen varsayılanların doğru hissettirdiğini. |
| E3 `/plan-tune narrative` + `/plan-tune vibe` | Olay-çapalı anlatım kararlı profil gerektirir. v1 verisi olmadan çıktı genel saçmalık olur. | Profil çeşitlilik kontrolü 2+ hafta gerçek kullanım için geçer. Anlatım testinin belirli olayları alıntıladığını, klişeleri değil, kanıtlar. |
| E4 Kör nokta koçu | E1/E6 ile açık etkileşim bütçesi tasarımı olmadan mantıksal olarak çelişir. Genel oturum bütçesi, yükseltme kuralları, uyumsuzluk algılamasından hariç tutma gerektirir. | Etkileşim bütçesi + yükseltme için tasarım şartnamesi. Deneme yanılması zorlamaların koçluk hissettirdiğini, sıkboğazlık değil, doğrular. |
| E5 LANDED kutlama HTML sayfası | Önyazıda yaşayamaz (Codex #9, #10). Terfi edildiğinde, açık komut `/plan-tune show-landed` VEYA gönderi sonrası hook'a taşınır — sıcak yolda pasif algılama değil. | Açık komut veya hook tasarımı. /design-shotgun → /design-html görsel yön için. PR veri toplama için güvenlik + gizlilik incelemesi. |
| E6 Uyumsuzluğa göre otomatik ayarlama | v1'de, /plan-tune beyan edilen ve çıkarılan arasındaki boşluğu gösterir. v2'de, beyan güncellemelerini önerebilir. Çift izli profilin kararlı olmasını gerektirir. | v1'den gerçek uyumsuzluk verisi tutarlı desenler gösterir. Öneri UX'i ayrı olarak tasarlanır. |
| Psikografik odaklı otomatik karar | v1'de sıfır davranış değişikliği. Yalnızca açık tercihler etki eder. | Gerçek kullanım açık tercihlerin çoğu durumu kapsadığını gösterir. Çıkarılan profil güvenilir olacak kadar kararlı. |

## Tamamen reddedildi (Codex haklıydı, bunları yapmıyoruz)

| Öğe | Neden reddedildi |
|------|--------------|
| Alt taban-prompt-kuralı olarak (vs. tipli kayıt defteri) | Codex #1. Aracılar talimatları sessizce atlayabilir. Psikografik üzerine inşa etmek kumdur. |
| Beyan edilen boyutlar üzerinde ±0.2 kelepçe | Codex #6. E6 uyumsuzluk algılama ile mantıksal çelişki yaratır. BİRİNİ seçin: düzenlenebilir tercih VEYA çıkarılan davranış. Şimdi: her ikisi, ayrı ayrı izlenir (çift izli profil). |
| Düz yazı özetlerini ayrıştırarak tek yönlü kapı sınıflandırma | Codex #4. Güvenlik şekillendirmeye bağlı. door_type soru tanım alanında beyan edilmeli (kayıt defteri), çıkarılmamalıdır. |
| Beyanlar + geçersiz kılmalar + kararlar + geri bildirim karıştıran tek olay şema dosyası | Codex #5. Uyumsuz etki alanı nesneleri. Şimdi üç dosyaya ayrıldı: question-log.jsonl, question-preferences.json, question-events.jsonl. |
| /plan-tune katılımı için TTHW telemetrisi | Codex #14. Yerel öncelikli çerçevelme ile çelişiyor. Yalnızca yerel günlükleme. |
| Kullanıcı kaynağı doğrulaması olmadan satır içi tune: yazımları | Codex #16. Profil zehirlenmesi saldırısı. Şimdi: kullanıcı kaynağı geçidi zorunludur. |

## Mimari

```
~/.gstack/
  developer-profile.json            # birleşik: beyan edilen + çıkarılan + oturumlar (office-hours'dan)

~/.gstack/projects/{SLUG}/
  question-log.jsonl                # her AskUserQuestion, yalnızca ekleme, kayıt defteri doğrulanmış
  question-preferences.json         # soru başına açık kullanıcı tercihleri
  question-events.jsonl             # tune: geri bildirim olayları, kullanıcı kaynağı geçitli
```

**Birleşik profil şeması** (hem v0.16.2.0 builder-profile.jsonl hem de önerilen developer-profile.json'un yerini alır):

```json
{
  "identity": {"email": "..."},
  "declared": {
    "scope_appetite": 0.9,
    "risk_tolerance": 0.7,
    "detail_preference": 0.4,
    "autonomy": 0.5,
    "architecture_care": 0.7
  },
  "inferred": {
    "values": {"scope_appetite": 0.72, "risk_tolerance": 0.58, "...": "..."},
    "sample_size": 47,
    "diversity": {
      "skills_covered": 5,
      "question_ids_covered": 14,
      "days_span": 23
    }
  },
  "gap": {"scope_appetite": 0.18, "...": "..."},
  "sessions": [
    {"date": "...", "mode": "builder", "project_slug": "...", "signals": []}
  ],
  "signals_accumulated": {
    "named_users": 1, "taste": 4, "agency": 3, "...": "..."
  }
}
```

**Çeşitlilik kontrolü** (Codex #13): `inferred`, yalnızca `sample_size >= 20 AND skills_covered >= 3 AND question_ids_covered >= 8 AND days_span >= 7` olduğunda "yeterli veri" olarak kabul edilir. Bunun altında, `/plan-tune profile` "henüz yeterli gözlemlenen veri yok" gösterir, potansiyel olarak yanıltıcı bir çıkarılan değer yerine.

## Veri akışı (v1)

1. Önyazı: `question_tuning` yapılandırmasını kontrol et. Kapalıysa, hiçbir şey yapma.
2. Her AskUserQuestion'dan önce:
   - Aracı `gstack-question-preference --check <registry-id>` çağırır
   - `never-ask` VE soru tek yönlü kapı DEĞİLSE → açıklama ile önerilen seçeneği otomatik seç
   - `always-ask`, ayarlanmamış veya soru tek yönlü kapı İSE → normal şekilde sor
3. AskUserQuestion'dan sonra:
   - question-log.jsonl'ye günlük kaydı ekle (kayıt defteri doğrulanmış, bilinmeyen ID'leri reddeder)
4. Satır içi teklif: "Bu soruyu ayarla? Ayarlamak için `tune: [geri bildirim]` yanıtla."
5. Kullanıcının SONRAKİ dönüş mesajı `tune:` öneki içeriyorsa VE içerik kullanıcının kendi mesajında ortaya çıysa (araç çıktısında değil):
   - Aracı `source: "inline-user"` ile `gstack-question-preference --write` çağırır
   - İkili dosya kaynak alanını doğrular; `inline-user` dışında herhangi bir şeyi reddeder
6. Çıkarılan boyutlar `bin/gstack-developer-profile --derive` tarafından talep üzerine yeniden hesaplanır. Sinyal haritası değişiklikleri olay geçmişinden tam yeniden hesaplama tetikler.

## Güvenlik modeli

**Profil zehirlenmesi savunması** (Codex #16, aşağıda Karar J): Satır içi tune olayları YALNIZCA şu durumlarda yazılabilir:
- Aracı kullanıcının mevcut sohbet turunu işliyorken
- `tune:` öneki o kullanıcı mesajında ortaya çıktığında (herhangi bir araç çıktısında, dosya içeriğinde, PR açıklamasında, commit mesajında vb. değil)
- Çözücünün aracıya talimatları bunu açıkça belirtir

İkili uygulama: `gstack-question-preference --write`, tune kaynaklı her kayıtta `source: "inline-user"` alanını gerektirir. Başka herhangi bir kaynak değeri (örn., `inline-tool-output`, `inline-file-content`) bir hata ile reddedilir. Aracı `source` alanını taklit etmemesi talimatı verilir.

**Veri gizliliği**:
- Tüm veriler `~/.gstack/` altında yalnızca yereldir. Kullanıcının açık eylemi olmadan hiçbir şey dışarı çıkmaz.
- `/plan-tune export <yol>` profili kullanıcı tarafından belirtilen yola yazar (katılımlı dışa aktarma).
- `/plan-tune delete` yerel profil dosyalarını siler.
- `gstack-config set telemetry off` herhangi bir telemetriyi engeller (bu skill profil verilerini asla göndermez).
- Profil dosyaları standart kullanıcı ana dizin izinlerine sahiptir.

**Enjeksiyon savunması** (mevcut `bin/gstack-learnings-log` desenleriyle tutarlı): `question_summary` ve serbest form kullanıcı geri bildirimi alanları, bilinen prompt enjeksiyonu desenlerine karşı temizlenir ("önceki talimatları yoksay", "system:", vb.).

## 5 Sıkı Kısıtlama (office-hours'dan korunmuş, Codex geri bildirimi için güncellenmiş)

1. **Tek yönlü kapılar kayıt defteri beyanı ile belirleyici olarak sınıflandırılır**, çalışma zamanı özet ayrıştırması ile DEĞİL. Her kayıt defteri girdisi `door_type: one-way | two-way` beyan eder. Anahtar kelime deseni yedeklemesi (`scripts/one-way-doors.ts`), uç durumlar için bir kemer-ve-yedek koruması ikincil kontroludur.
2. **Profil boyutları incelenebilir VE düzenlenebilir.** `/plan-tune profile` beyan edilen + çıkarılan + boşluğu gösterir. Düz İngilizce ile düzenlemeler yalnızca `declared`'a gider. Sistem `inferred`'ı bağımsız olarak izler.
3. **Sinyal haritası TypeScript'te el yapımı.** `scripts/psychographic-signals.ts`, `{question_id, user_choice} → {dimension, delta}` eşler. Aracı tarafından çıkarılan değil. v1'de yalnızca `inferred.values` gösterimi için tüketilir — kararları yönlendirmek için değil.
4. **v1'de psikografik odaklı otomatik karar yok.** Yalnızca açık soru başına tercihler etki eder. Bu, "kalibrasyon geçidi manipüle edilebilir" eleştirisini (Codex #13) tamamen ortadan kaldırır — v1'in geçecek bir geçidi yok.
5. **Proje başına tercihler genel tercihleri geçersiz kılar.** `~/.gstack/projects/{SLUG}/question-preferences.json`, gelecekteki herhangi bir genel tercih dosyasını geçersiz kılar. Genel profil (`~/.gstack/developer-profile.json`), projeler arasında çeşitlilik için bir başlangıç noktasıdır.

## Neden olay kaynaklı + çift izli

**Çıkarılan profil için neden olay kaynaklı**:
- Sinyal haritası gstack sürümleri arasında değişebilir. Olaylardan yeniden hesapla, veri göçüne gerek yok.
- Denetlenebilir: `/plan-tune profile --trace autonomy` değere katkıda bulunan her olayı gösterir.
- Geleceğe hazır: yeni boyutlar mevcut geçmişten türetilebilir.

**Neden çift izli (beyan edilen + çıkarılan, ayrı ayrı)** (Aşağıda Karar B):
- Codex #6'nın tanımladığı mantıksal çelişkiyi çözer.
- `declared` kullanıcı egemenliğidir. Kullanıcı kim olduğunu belirtir. Sistem, kullanıcı odaklı her şey için uyar (tercihler, beyanlar, geçersiz kılmalar).
- `inferred` gözlemdir. Sistem davranışsal desenleri izler. Görüntülenir ama v1'de üzerinde işlem yapılmaz.
- `gap` ilginç sinyaldir. Büyük boşluklar, kullanıcının kendi tanımının davranışıyla eşleşmediğini önerir — değerli özdeneyim, ama otomatik olarak düzeltilmez.

## Etkileşim modeli — her yerde düz İngilizce

(/plan-devex-review'dan, CLI sözdizimi üzerine kullanıcı düzeltmesi):

`/plan-tune` (argümansız) konuşmalı moda girer. CLI alt komut sözdizimi gerekmez.

Düz İngilizce menü:
- "Profilimi göster"
- "Sorulduğum soruları incele"
- "Bir soru hakkında tercih ayarla"
- "Profilimi güncelle — bir şeyi değiştirdim"
- "Söylediklerimle yaptıklarım arasındaki boşluğu göster"
- "Kapat"

Kullanıcı konuşmalı olarak yanıt verir. Aracı yorumlar, amaçlanan değişikliği onaylar, sonra yazar. Örneğin:
- Kullanıcı: "0.5'in öne sürdüğünden daha bir okyanus kaynatma kişisiyim"
- Aracı: "Anlaşıldı — `declared.scope_appetite`'ı 0.5'ten 0.8'e güncelle? [E/h]"
- Kullanıcı: "Evet"
- Aracı güncellemeyi yazar

Serbest form girdiden `declared`'ın herhangi bir mutasyonu için onay adımı gereklidir (Codex #15 güven sınırı).

Güçlü kullanıcılar kısayollar yazabilir (`narrative`, `vibe`, `reset`, `stats`, `enable`, `disable`, `diff`). Hiçbiri gerekli değil. Her ikisi de çalışır.

## Oluşturulacak Dosyalar

### Çekirdek şema
- `scripts/question-registry.ts` — tipli kayıt defteri. Tüm SKILL.md.tmpl AskUserQuestion çağrılarının denetiminden tohumlanır.
- `scripts/one-way-doors.ts` — ikincil anahtar kelime yedeklemesi. Birincil: kayıt defterinde `door_type`.
- `scripts/psychographic-signals.ts` — çıkarılan hesaplama için el yapımı sinyal haritası.

### İkili dosyalar
- `bin/gstack-question-log` — günlük kaydı ekle, kayıt defterine karşı doğrula.
- `bin/gstack-question-preference` — açık tercihleri oku/yaz/kontrol/temizle.
- `bin/gstack-developer-profile` — `bin/gstack-builder-profile`'ın yerini alır. Alt komutlar: `--read` (eski uyum), `--derive`, `--gap`, `--profile`.

### Çözücüler
- `scripts/resolvers/question-tuning.ts` — üç üreteç: `generateQuestionPreferenceCheck(ctx)` (soru öncesi kontrol), `generateQuestionLog(ctx)` (soru sonrası günlük), `generateInlineTuneFeedback(ctx)` (soru sonrası tune: kullanıcı kaynağı geçit talimatları ile prompt).

### Skill
- `plan-tune/SKILL.md.tmpl` — konuşmalı, düz İngilizce inceleme ve tercih aracı.

### Testler
- `test/plan-tune.test.ts` — kayıt defteri tamlılığı, kopya ID kontrolü, tercih önceliği (never-ask + tek yönlü değil → OTOMATİK_KARAR; never-ask + tek yönlü → NORMAL_SOR), kullanıcı kaynağı geçidi (inline-user olmayan kaynakları reddeder), türetme + yeniden hesaplama, birleşik profil şeması, 7 oturumlu fixture ile geçiş regresyonu.

## Değiştirilecek Dosyalar

- `scripts/resolvers/index.ts` — 3 yeni çözücü kaydet.
- `scripts/resolvers/preamble.ts` — `_QUESTION_TUNING` yapılandırma okuma; tier >= 2 için 3 çözücü enjekte et.
- `bin/gstack-builder-profile` — eski shim `bin/gstack-developer-profile --read`'e devreder.
- Geçiş betiği — mevcut builder-profile.jsonl'ı birleşik developer-profile.json'a katlar. Atomik, idempotent, kaynağı `.migrated-YYYY-MM-DD` olarak arşivler.

## v1'de dokunulmayanlar

Açıkça değişmeden — `{{PROFILE_ADAPTATION}}` yer tutucuları yok, profil temelinde davranış değişikliği yok:

- `ship/SKILL.md.tmpl`, `review/SKILL.md.tmpl`, `office-hours/SKILL.md.tmpl`, `plan-ceo-review/SKILL.md.tmpl`, `plan-eng-review/SKILL.md.tmpl`

Bu skill'ler yalnızca günlükleme / tercih kontrolü / tune geri bildirimi için önyazı enjeksiyonu kazanır. Profil temelli varsayılanlar yok. v2 işi.

## Karar günlüğü (her biri için artıları/eksileri)

### Karar A: Üçünü birden paketle (soru günlüğü + duyarlılık + psikografik) vs. daha küçük kama gönder — İLK YANIT: PAKETLE; DÜZELTİLMİŞ: KAYIT DEFTERİ İLKİ GÖZLEMSEL

İlk kullanıcı pozisyonu (office-hours): "Psikografik farklılaştırma. Geri bildirim döngüsü davranışı gerçekten ayarlayabilsin diye tümünü gönder." Bu CEO EXPANSION'ı yönlendirdi.

**Paketlemenin artıları:** Hırs. Öğrenme katmanı bunu yapılandırmadan daha fazlasını yapar. Psikografik olmadan bu süslü bir ayarlar menüsü.

**Paketlemenin eksileri (Codex tarafından ortaya çıkarıldı):** Alt taban mevcut değildi. Psikografik prompt kuralı üzerinde kumdur. E1/E4/E6 tutarsız birleşir. Profil zehirlenmesi ele alınmamıştı. E5 önyazıda gizli sıcak yol yan etkisi. Uygulama sırası uygulanamaz bir kural çevresinde makine inşa etti.

**Düzeltilmiş yanıt:** Kayıt defteri ilk gözlemsel v1 (bu doküman). Hırssı açık kabul kriterleri ile v2 hedefi olarak korur. Savunulabilir bir temel gönderir. Kullanıcı Codex'in 20 noktalı eleştirisini gördükten sonra bunu kabul etti.

### Karar B: Olay kaynaklı vs. saklanan boyutlar vs. karma — YANIT: OLAY KAYNAKLI + KULLANICI BEYANI ÇAPASI (B+C)

**Yaklaşım A (saklanan boyutlar):** Yerinde mutasyon. Basit.
- Artıları: En küçük veri modeli. Akıl yürütmesi kolay.
- Eksileri: Kayıplı. Geçmiş yok. Sinyal haritası değişiklikleri göç gerektirir. Profil değişiklikleri kullanıcı için opak.

**Yaklaşım B (olay kaynaklı):** Ham olayları sakla, boyutları türet.
- Artıları: Denetlenebilir. Sinyal haritası değişikliklerinde yeniden hesaplanabilir. Asla veri göçü yok. Mevcut learnings.jsonl deseniyle eşleşir.
- Eksileri: Daha karmaşık türetme. Olaylar dosyası zamanla büyür (sıkıştırma v2'ye ertelendi).

**Yaklaşım C (karma — kullanıcı beyanı çapası, olaylar rafine eder):** İlk profil kullanıcı tarafından belirtilir; olaylar ±0.2 içinde rafine eder.
- Artıları: 1. gün değeri. Kullanıcı egemenliği. Sıfırdan başlamak yerine kalibrasyon çapası.
- Eksileri: ±0.2 kelepçe uyumsuzluk algılama ile mantıksal çelişki yaratır (Codex #6 bunu yakaladı).

**Seçilen: ±0.2 KELEPÇE KALDIRILARAK B+C birleştirildi.** Altta olay kaynaklı, beyan edilen profil birinci sınıf ayrı alan. Kelepçe yok. Beyan edilen ve çıkarılan bağımsız değerler olarak yaşar. Aralarındaki boşluk görüntülenir ama v1'de otomatik olarak düzeltilmez.

### Karar C: Tek yönlü kapı sınıflandırma — çalışma zamanı düzyazı ayrıştırma vs. kayıt defteri beyanı — YANIT: KAYIT DEFTERİ BEYANI (Codex sonrası)

**Çalışma zamanı düzyazı ayrıştırma (orijinal):** `isOneWayDoor(skill, category, summary)` artı anahtar kelime desenleri.
- Artıları: Skill yazarları için minimum sürtünme. Bakımı şema yok.
- Eksileri (Codex #4): Güvenlik şekillendirmeye bağlı. Yıkıcı işlem sorusu hafifçe ifade edilirse yanlış sınıflandırılabilir. Bir güvenlik geçidi için kabul edilemez.

**Kayıt defteri beyanı (düzeltilmiş):** Her kayıt defteri girdisi `door_type` beyan eder.
- Artıları: Belirleyici. Denetlenebilir. CI ile uygulanabilir (tüm sorular beyan etmeli).
- Eksileri: Bakım yükü. Her yeni skill sorusu sınıflandırmalıdır.

**Seçilen: birincil olarak kayıt defteri beyanı, yedek olarak anahtar kelime desenleri.** Şema yönetimi güvenliğin bedelidir.

### Karar D: Satır içi tune geri bildirim grameri — yapılandırılmış anahtar kelimeler vs. serbest form doğal dil — YANIT: YAPILANDIRILMIŞ VE SERBEST FORM YEDEK

**Yalnızca yapılandırılmış anahtar kelimeler:** `tune: unnecessary | ask-less | never-ask | always-ask | context-dependent`.
- Artıları: Belirsiz değil. Temiz profil verisi.
- Eksileri: Kullanıcılar ezberlemeli.

**Yalnızca serbest form:** Aracı kullanıcının söylediği her şeyi yorumlar.
- Artıları: Doğal. Öğrenilecek sözdizimi yok.
- Eksileri: Tutarsız profil verisi. Bir tune'un neden etkili olmadığını hata ayıklamak zor.

**Seçilen: her ikisi.** Güçlü kullanıcılar için belgelenen kısayollar; aracı serbest İngilizce'yi kabul eder ve normalleştirir. Düz İngilizce etkileşim varsayılan; yapılandırılmış anahtar kelimeler isteğe bağlı hızlı yoldur.

### Karar E: /plan-tune için CLI alt komut yapısı — YANIT: DÜZ İNGILIZCE KONUŞMALI (alt komut sözdizimi gerekmez)

**`/plan-tune profile`, `/plan-tune profile set autonomy 0.4`, vb.** (orijinal):
- Artıları: Güçlü kullanıcılar için hızlı. --help ile kendi kendini belgeleyen.
- Eksileri: Kullanıcılar ezberlemeli. Her çağırma bir CLI oturumu gibi hissettirir, bir konuşma gibi değil.

**Düz İngilizce konuşmalı (kullanıcı düzeltmesinden sonra düzeltilmiş):** `/plan-tune` bir menüye girer. Kullanıcı doğal dilde ne istediğini söyler.
- Artıları: Sıfır ezberleme. Bir kabukla değil, bir koçla konuşmak gibi hissettirir.
- Eksileri: Güçlü kullanıcılar için daha yavaş. İyi aracı yorumlama gerektirir.

**Seçilen: isteğe bağlı kısayollarla konuşmalı.** Hiçbir yol gerekmez. Çoğu kullanıcı kısayolları hiç görmez. Beyan edilen profili mutasyona uğratmadan önce onay adımı gerekir (aracı yanlış yorumlamaya karşı güven — Codex #15 güven sınırı).

### Karar F: LANDED kutlama — pasif önyazı algılama vs. açık komut vs. gönderi sonrası hook — YANIT: v2'YE ERTELENDİ; TERFİ EDİLDİĞİNDE, ÖNYAZIDA DEĞİL

**Önyazıda pasif algılama (orijinal):** Her skill'in önyazısı yakın birleştirmeleri algılamak için `gh pr view` çalıştırır.
- Artıları: Hangı skill çalıştırılırsa çalıştırılsın çalışır. Kullanıcının özel bir şey yapması gerekmez.
- Eksileri (Codex #9): Gecikme, kimlik doğrulama başarısızlıkları, hız sınırları, sürpriz tarayıcı açmaları, her skill'in önyazısına enjekte edilen belirsizlik. Sıcak yolda yan etki.

**Açık komut (`/plan-tune show-landed`):** Kullanıcı katılır.
- Artıları: Sıcak yol yan etkisi yok. Kullanıcı ne zaman göreceğini kontrol eder.
- Eksileri: Kullanıcı keşfi gerektirir. "Kazandığınızda sizi şaşırt" büyüsü kaybolur.

**Gönderi sonrası hook (`/ship` PR oluşturmadan sonra algılamayı tetikler):** /ship'e bağlı.
- Artıları: Doğal zamanlama. Önyazı maliyeti yok.
- Eksileri: /ship her zaman lan etme olayı değildir (manuel birleştirmeler, takım üyelerinin birleştirmesi, vb.).

**Seçilen: TAMAMEN ERTELENDİ.** v2 bunu düzgün tasarlayacak. Terfi edildiğinde, önyazıdan çıkar. Kullanıcı Codex'in kutlama sayfasının zaten riskli bir özellik için stratejik olarak uymadığı argümanını kabul etti.

### Karar G: Kalibrasyon geçidi — 20 olay vs. çeşitlilik kontrolü — YANIT: ÇEŞİTLİLİK KONTROLÜ

**"20 olay" (orijinal):** Basit sayı.
- Artıları: Uygulama açısından önemsiz.
- Eksileri (Codex #13): Manipüle edilebilir. TEK soruya 20 satır içi "gereksiz" yanıt beş boyutu kalibre etmemeli.

**Çeşitlilik kontrolü (düzeltilmiş):** `sample_size >= 20 AND skills_covered >= 3 AND question_ids_covered >= 8 AND days_span >= 7`.
- Artıları: Profil gerçekten sistem genelinde egzersiz yapılmış before it's trusted.
- Eksileri: Biraz daha karmaşık.

**Seçilen: çeşitlilik kontrolü.** v1'de yalnızca "görüntülemek için yeterli veri" eşiği için kullanılır. v2'de psikografik odaklı otomatik karar için geçit olacak.

### Karar H: Uygulama sırası — önce sınıflandırıcılar vs. önce entegrasyon noktası — YANIT: ÖNCE ENTEGRASYON NOKTASI (kayıt defteri + CI lint)

**Önce sınıflandırıcılar (orijinal):** İkili araçları oluştur, sonra çözücüler, sonra skill şablonu.
- Artıları: Atomik yapı taşları. Entegrasyondan önce birim test edebilirsin.
- Eksileri (Codex #19): Uygulanamaz bir kural çevresinde makine inşa eder. Kural tutmazsa, tüm iş boşa gider.

**Önce entegrasyon noktası (düzeltilmiş):** Önce tipli kayıt defteri + CI lint oluştur. Altyapıyı üzerine inşa etmeden önce entegrasyonun çalıştığını kanıtla.
- Artıları: Temel kanıtlanır. Altyapının dayanabileceği bir şey var.
- Eksileri: gstack'teki her mevcut AskUserQuestion'ı denetlemeyi gerektirir — önemli ön çalışma.

**Seçilen: önce entegrasyon noktası.** Codex'in argümanı belirleyici oldu. Denetim tam olarak amaç — uyarlamayı üzerine inşa etmeden önce gerçekten neye sahip olduğumuzu kataloglamamızı zorlar.

### Karar I: TTHW için telemetri — katılımlı telemetri vs. yalnızca yerel — YANIT: YALNIZCA YEREL

**Katılımlı telemetri (orijinal, DX incelemesinde önerildi):** TTHW'yi telemetri olayı üzerinden araçlandır.
- Artıları: Tüm kullanıcılar arasında katılım deneyiminin nicel ölçüsü.
- Eksileri (Codex #14): Yerel öncelikli OSS çerçevelmesi ile çelişiyor. Bu skill için özel olarak telemetri yüzeyi ekler.

**Yalnızca yerel (düzeltilmiş):** Günlükleme yereldir. Mevcut `telemetry` yapılandırmasına saygı duyar; skill yeni telemetri kanalları eklemez.
- Artıları: gstack'in yerel öncelikli etosuyla tutarlı.
- Eksileri: Katılım zamanının toplu görünümü yok.

**Seçilen: yalnızca yerel.** Daha sonra TTHW verisine ihtiyacımız varsa, skill'e özel biri yerine gstack genelinde bir telemetri olayı olarak mevcut katılımlı bayrağın arkasına ekleriz.

### Karar J: Profil zehirlenmesi savunması — savunma yok vs. onay geçidi vs. kullanıcı kaynağı geçidi — YANIT: KULLANICI KAYNAĞI GEÇİDİ

**Savunma yok (orijinal — Codex tarafından yakalandı):** Aracı gördüğü her tune olayını yazar.
- Artıları: En basit. Ek güven kontrolü yok.
- Eksileri (Codex #16): Kötü niyetli repo içeriği, PR açıklamaları, araç çıktısı `tune: never ask` enjekte edebilir ve profili zehirleyebilir. Bu gerçek bir saldırı yüzeyidir.

**Onay geçidi:** Her tune yazımı "Onaylıyor musunuz? [E/h]" sorar.
- Artıları: Evrensel savunma.
- Eksileri: Her meşru kullanımda sürtünme.

**Kullanıcı kaynağı geçidi:** Aracı yalnızca `tune:` öneki kullanıcının mevcut sohbet turundaki mesajında ortaya çıktığında tune olaylarını yazar (araç çıktısında değil, dosya içeriğinde değil). İkili dosya `source: "inline-user"` doğrular.
- Artıları: Saldırıyı meşru kullanımda sürtünme olmadan engeller.
- Eksileri: Aracının kaynağı doğru tanımlamasına bağlıdır. İkili düzey doğrulama, uygulamadır.

**Seçilen: kullanıcı kaynağı geçidi.** Tehdit modeline (otomatik girdilerdeki kötü niyetli içerik) normal akışı bozmadan karşı gelir.

## Başarı Kriterleri

- `bun test` yeni `test/plan-tune.test.ts` dahil geçer.
- Her SKILL.md.tmpl'deki her AskUserQuestion çağırmasının bir kayıt defteri girdisi var. CI lint uygular.
- `~/.gstack/builder-profile.jsonl`'dan geçiş, oturumların %100'ünü + signals_accumulated'ı korur. 7 oturumlu fixture ile regresyon testi.
- Tek yönlü kapı kayıt defteri beyan edilen girdileri: yıkıcı işlemlerin, mimari çatalların, > 1 gün CC çabası kapsam eklemelerin, güvenlik/uyumluluk seçimlerinin %100'ü `one-way` olarak sınıflandırılır.
- Kullanıcı kaynağı geçidi testi: `source: "inline-tool-output"` ile bir tune olayı yazmaya çalışmak reddedilir.
- Deneme yanılmaları: Garry 2+ hafta `/plan-tune` kullanır. Geri bildirimde bulunur:
  - `tune: never-ask` yazmak doğal hissettirdi mi yoksa göz ardı edildi mi
  - Kayıt defteri bakımı (yeni sorular eklemek) makul bir disiplin mi yoksa şema bürokrasisi mi hissettirdi mi
  - Çıkarılan boyutlar oturumlar arasında kararlı mı yoksa gürültülü mü
  - Düz İngilizce etkileşim bir koç gibi mi yoksa bir sohbet botuyla tartışmak gibi mi hissettirdi mi

## Uygulama Sırası

1. gstack'teki her SKILL.md.tmpl'deki her `AskUserQuestion` çağırmasını denetle. ID'ler, kategoriler, door_types, seçenekler ile ilk `scripts/question-registry.ts`'yi oluştur. Bu temeldir; diğer her şey bunun üzerinde oturur.
2. Kayıt defteri tamlılık testi (geçit katmanı) yaz `test/plan-tune.test.ts`. Sapmayı yakaladığını doğrula — geçici olarak bir kayıt defteri girdisini kaldır, CI'nın başarısız olduğunu onayla.
3. Anahtar kelime deseni yedekleme sınıflandırıcı ile `scripts/one-way-doors.ts`'yi tohumla.
4. İlk `{question_id, user_choice} → {dimension, delta}` eşlemeleri ile `scripts/psychographic-signals.ts`'yi tohumla. Sayılar geçici — v1 gönderilir, v2 yeniden kalibre eder.
5. Gelecekteki v2 `/plan-tune vibe` tarafından referans verilen arketip tanımları ile `scripts/archetypes.ts`'yi tohumla.
6. `bin/gstack-question-log` — kayıt defterine karşı doğrular, bilinmeyen ID'leri reddeder.
7. `bin/gstack-question-preference` — tüm alt komutlar + testler.
8. `bin/gstack-developer-profile` — `--read` (eski), `--derive`, `--gap`, `--profile`.
9. Geçiş betiği — builder-profile.jsonl → birleşik developer-profile.json. Atomik, idempotent, kaynağı arşivler. Fixture ile regresyon testi.
10. `scripts/resolvers/question-tuning.ts` — üç üreteç (tercih kontrolü, günlük, kullanıcı kaynağı geçit talimatları ile satır içi tune).
11. 3 çözücüyü `scripts/resolvers/index.ts`'ye kaydet.
12. `scripts/resolvers/preamble.ts`'yi güncelle — `_QUESTION_TUNING` yapılandırma okuma; tier >= 2 skill'ler için koşullu enjekte et.
13. `plan-tune/SKILL.md.tmpl` — konuşmalı düz İngilizce skill.
14. `bun run gen:skill-docs` — tüm SKILL.md dosyaları yeniden oluşturulur; her birinin 100KB token tavanının altında kaldığını doğrula.
15. `bun test` — tüm 45+ test durumu yeşil.
16. 2+ hafta deneme yanılmaları. Gerçek soru günlüğü + tercih verisi topla. Başarı kriterlerine karşı ölç.
17. `/ship` v1. Deneme yanılmalarından sonra v2 kapsam tartışması.

## Açık Sorular (v2 kapsam kararları, gerçek verilere kadar ertelendi)

1. Kesin sinyal haritası deltaları. v1 ilk tahminlerle gönderilir; v2 gözlemlenen verilerden yeniden kalibre eder.
2. `inferred` ve `declared` boşluğu büyük olduğunda, `declared`'ı güncellemeyi otomatik önermeli miyiz? Yoksa yalnızca görüntülemeli miyiz?
3. Bir sinyal haritası sürümü değiştiğinde, otomatik yeniden hesaplamalı mıyız yoksa kullanıcıyı bilgilendirmeli miyiz? Varsayılan: diff gösterimi ile otomatik yeniden hesaplama.
4. Çapraz proje profil mirası vs. izolasyon. v1 proje başına tercihler + genel profil; v2 açık çapraz proje öğrenme katılımı ekleyebilir.
5. `/plan-tune` paylaşık bir developer-profile'ın işbirliğini bilgilendirdiği bir "takım profili" modu desteklemeli mi? v2+.

## Dahil edilen incelemeler

- **/office-hours (2026-04-16, 1 oturum):** 5 sıkı kısıtlama belirledi, olay kaynaklı + kullanıcı beyanı mimarisini seçti.
- **/plan-ceo-review (2026-04-16, EXPANSION modu):** 6 genişleme kabul edildi, daha sonra Codex incelemesinden sonra geri alındı.
- **/plan-devex-review (2026-04-16, POLISH modu):** Düz İngilizce etkileşim modeli; bu v1'e kadar hayatta kaldı.
- **/plan-eng-review (2026-04-16):** Test planı ve tamlık kontrolleri; kısmen kayıt defteri ilk yeniden yazımı tarafından geçersiz kılındı.
- **/codex (2026-04-16, gpt-5.4 yüksek akıl yürütme):** 20 noktalı eleştiri geri almaya yönelti. Claude incelemelerinin kaçırdığı 15+ meşru bulgu.

## Katkılar ve uyarılar

Bu plan ~6 saatlik planlama boyunca yinelemeli bir AI işbirliği döngüsü aracılığıyla geliştirildi. Yazar (Garry Tan) her kapsam kararını yönlendirdi; AI sesleri (Claude Opus 4.7 ve OpenAI Codex gpt-5.4) planı sorguladı ve rafine etti. Codex'in dış sesi olmadan, çok daha büyük ve daha az savunulabilir bir plan gönderilirdi. Yüksek riskli mimari değişikliklerde çapraz model incelemenin değeri gerçek ve ölçülebilirdir.