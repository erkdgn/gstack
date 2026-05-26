# Adımlama Güncellemeleri v0 — Tasarım Dokümanı

**Durum:** V1.1 planı (henüz uygulanmadı).
**Çıkarıldığı yer:** İnceleme titizliği, adımlama iş kolunun plan-metin düzenlemesiyle düzeltilemeyen yapısal boşluklar olduğunu ortaya çıkardığında, uygulama sırasında [PLAN_TUNING_V1.md](./PLAN_TUNING_V1.md)'den çıkarıldı.
**Yazarlar:** Garry Tan (kullanıcı), Claude Opus 4.7 + OpenAI Codex gpt-5.4'den AI destekli incelemelerle.
**İnceleme planı:** CEO + Codex + DX + Mühendislik döngüsü, V1 ile aynı titizlik.

## Teşekkür

Bu plan **[Louise de Sadeleer](https://x.com/LouiseDSadeleer/status/2045139351227478199)** sayesinde var. Mimari inceleme sırasındaki "evet evet evet"i yalnızca jargonla ilgili değildi (V1 bunu ele alıyor) — adımlama ve ajansla ilgiliydi. Çok uzun bir inceleme boyunca çok fazla kesintiye uğratıcı karar. Teknik olmayan kullanıcılar ~10-15 kesintide kontrol dışı kalır. **Bu V1.1.**

## Sorun

Louise'in gstack inceleme çıktısını okurken yaşadığı yorgunluk iki kaynaktan geliyordu:

1. **Jargon yoğunluğu** — teknik terimler açıklama olmadan ortaya çıkıyordu. *V1'de ele alındı (ELI10 yazımı).*
2. **Kesinti hacmi** — `/autoplan` 4 aşama (CEO + Tasarım + Mühendislik + DX) çalıştırdı, her biri 5-10 AskUserQuestion promptuyla. Toplam ~45 dakika boyunca ~30-50 prompt. Teknik olmayan kullanıcılar ~10-15 kesintide kontrol dışı kalır. **Bu V1.1.**

Sadece çeviri kesinti hacmini düzeltmez. Çevrilmiş bir kesinti hala bir kesintidir. Düzeltme, bulguların NE ZAMAN ortaya çıktığını değiştirmeli, yalnızca NASIL ifade edildiklerini değil.

## Neden çıkarıldı (V1'in üçüncü mühendislik incelemesinden + Codex 2. geçişinden yapısal boşluklar)

V1 planlaması sırasında, bir adımlama iş kolu taslağı hazırlandı: bulguları sırala, iki yönlü kapıları otomatik kabul et, aşama başına en fazla 3 AskUserQuestion promptu, otomatik kabul edilen öğeler için Sessiz Kararlar bloğu, kararı sonradan yeniden açmak için `flip <id>` komutu. Üçüncü mühendislik inceleme geçişi + ikinci Codex geçişi, plan-metin düzenlemeleriyle kapatılamayacak 10 boşluk ortaya çıkardı:

1. **Oturum-durum modeli tanımsız.** Adımlama, aşama başına durum gerektirir (hangı bulgular ortaya çıktı, hangileri otomatik kabul edildi, hangileri kullanıcı döndürebilir). V1'in glossing için aşama başına durumu var ama aşama başına adımlama belleği için destek deposu yok.
2. **Soru günlüğünden aşama tanımlayıcısı eksik.** Sessiz Mühendislik #8, bir aşama içinde >3 prompt olduğunda uyarmak istedi. V0'ın `question-log.jsonl`'inde `phase` alanı yok. V1 "şema değişikliği yok" iddia etti — uygulama hedefiyle çelişiyor.
3. **Soru kayıt defteri ≠ bulgu kayıt defteri.** V0'ın `scripts/question-registry.ts` *soruları* kapsar (skill tanım zamanında kaydedilir). İnceleme bulguları *dinamiktir* (çalışma zamanında keşfedilir). Kayıt defteri aracılığıyla `door_type: one-way` uygulaması, aracının inceleme sırasında oluşturduğu geçici bulguları kapsamaz. Tek yönlü kapı güvenliği, aracının inceleme ortasında oluşturduğu bulgular için uygulanamaz.
4. **Adımlama düzyazı olarak mevcut kontrol akışını tersine çeviremez.** V1, önyazı düzyazısına "bulguları sırala, sonra sor" kuralı eklemeyi planladı. Ancak `plan-eng-review/SKILL.md.tmpl` gibi mevcut skill şablonlarında aşama başı STOP/AskUserQuestion dizileri var. Önyazı düzyazısındaki bir kural, kalıplaştırılmış aşama başı STOP'u güvenilir şekilde geçersiz kılamaz. Davranışsal değişiklik sıralamadır, prompt şekillendirmesi değil.
5. **Döndürme mekanizmasının uygulaması yok.** "Değiştirmek için `flip <id>` yanıtla" düzyazıydı. Komut ayrıştırıcı yok, durum deposu yok, yeniden oynatma davranışı yok. Konuşma sıkışırsa ve Sessiz Kararlar bloğu bağlamdan ayrılırsa, orijinal karar kaybolur.
6. **Geçiş promptunun kendisi bir kesintidir.** V1'in yükseltme sonrası geçiş promptu (V0 düzyazısını geri yüklemeyi teklif eden), V1.1'in azaltmaya çalıştığı kesinti bütçesine karşı sayılır. V1.1 karar vermeli: bütçeden muaf mı, yoksa kesinti-1/N olarak mı dahil?
7. **İlk çalıştırma önyazı promptları da sayılır.** Lake tanıtımı, telemetri, proaktif, yönlendirme enjeksiyonu — Louise hepsini ilk çalıştırmada gördü. Bunlar ilk gerçek skill çalışmadan önceki kesintiler. V1.1, bunların yeni kullanıcılar için hangilerinin yük taşıyıcı, hangilerinin N. oturuma kadar ertelenebilir olduğunu denetlemeli.
8. **Sıralama formülü gerçek verilere göre kalibre edilmemiş.** V1, `product 0-8`'i (bozuk: `{0,1,2,4,8}` dağılımı), ardından `sum 0-6` ile eşik ≥ 4'ü düşündü. Ama ikisi de gerçek bulgu dağılımına karşı doğrulanmadı. V1.1, gerçek bulguların neye benzediğini ölçmek için V0 soru günlüğünü araçlandırmalı, sonra kalibre etmeli.
9. **"Her tek yönlü kapı ortaya çıkar" vs "aşama başına en fazla 3" çelişiyor.** Tek yönlü kapı sınırı = sınırısız (güvenlik); iki yönlü sınır = 3. Ama plan, açık öncelik olmadan her iki kurala da sahipti. V1.1 belirtmeli: tek yönlü kapılar aşama bütçesinden bağımsız olarak sınırısız şekilde ortaya çıkar.
10. **Tanımsız doğrulama değerleri.** V1 planında "Sessiz Kararlar bloğu ≥ N giriş" vardı, N hiçbir zaman tanımlanmadı, ve throughput JSON'ındaki `active: true` alanı hiçbir zaman tanımlanmadı. V1.1 somut değerler alır.

## V1.1 Kapsamı

1. **Oturum-durum modelini tanımla.** Skill çağırma başına vs aşama başına vs konuşma başına. Destek deposu: muhtemelen hangı bulguların ortaya çıktığını vs. aşama başına otomatik kabul edildiğini kaydeden `~/.gstack/sessions/<session_id>/pacing-state.json`. Temizlik: önyazıdaki mevcut oturum izleme ile aynı TTL.

2. **`phase` alanını question-log.jsonl şemasına ekle.** Her AskUserQuestion'ı hangi inceleme aşamasından geldiğini sınıflandır (CEO / Tasarım / Mühendislik / DX / diğer). Geçiş: mevcut girdiler varsayılan olarak `"unknown"`. Bozucu olmayan şema uzantısı.

3. **Dinamik bulgular için kayıt defteri kapsamını genişlet.** İki seçenek, CEO incelemesi sırasında seç:
   - (a) `scripts/question-registry.ts`'yi çalışma zamanı kaydına izin verecek şekilde genişlet (geçici ID'ler hala günlüğe kaydedilir + sınıflandırılır).
   - (b) Bulgu metnini → risk katmanına eşleyen ikincil bir çalışma zamanı sınıflandırıcı `scripts/finding-classifier.ts` ekle (desen eşleştirme kullanarak).

4. **Adımlamayı önyazı düzyazısından skill şablonu kontrol akışına taşı.** Her inceleme skill şablonunu güncelle: (i) aşamayı dahili olarak tamamla, (ii) bulguları `gstack-pacing-rank` ikili dosyası ile sırala, (iii) en fazla 3 AskUserQuestion promptu yayın, (iv) geri kalanını Sessiz Kararlar bloğu olarak yayın. Önyazı kuralı değil — her şablonda açık sıralama.

5. **Döndürme mekanizması uygulaması.** Yeni ikili dosya `bin/gstack-flip-decision`. Komut ayrıştırıcı kullanıcı mesajından `flip <id>` kabul eder. Orijinal kararı pacing-state.json'da arar. Açık AskUserQuestion olarak yeniden açar. Yeni seçim kalıcı olur.

6. **Geçiş promptu bütçe kararı.** Açık kural: tek seferlik geçiş promptları aşama başına kesinti bütçesinden muaftır. Gerekçe: inceleme aşamaları başlamadan önce ateşlenir, sırasında değil.

7. **İlk çalıştırma önyazı denetimi.** Lake tanıtımı, telemetri, proaktif, yönlendirme enjeksiyonunu denetle. Her biri için: ilk kez kullanıcı için yük taşıyıcı mı, yoksa ertelenebilir mi? Olası sonuç: lake tanıtımı dışında hepsini 2. oturuma kadar bastır. Kalanlarını kullanıcıların isteğe bağlı olarak çaırabileceği bir `/plan-tune first-run` komutu ile sun.

8. **Sıralama eşiği kalibrasyonu.** V0'ın soru günlüğünü araçlandır (zaten çalışıyor, geçmişi var). Son CEO + Mühendislik + DX + Tasarım incelemelerindeki `severity × irreversibility × user-decision-matters` gerçek dağılımını ölç. Eşiği gerçek verilere göre seç. Hedef: bulguların ~%20'si ortaya çıkar, ~%80'i otomatik kabul edilir.

9. **Açık kural: tek yönlü kapılar sınırısız.** Skill şablonu düzyazısında sabit kodlanmış: "tek yönlü kapılar aşama kesinti bütçesinden bağımsız olarak ortaya çıkar." İki yönlü bulgular aşama başına 3 ile sınırlıdır.

10. **Somut doğrulama değerleri.** Sessiz Kararlar için `N`'yi tanımla (örn., önemsiz olmayan bir plan için ≥ 5 giriş beklenir), somut alan adları ile throughput JSON şemasını tanımla.

## V1.1 için kabul kriterleri

- **Kesinti sayısı:** Louise (veya benzeri teknik olmayan bir işbirlikçi) V0 temeli ile karşılaştırılabilir bir plan üzerinde uçtan uca `/autoplan`'ı yeniden çalıştırır. AskUserQuestion sayısı V0 temelinin ≤%50'si. (V1, V1.1 kalibrasyonu için bu temel transkripti yakalar.)
- **Tek yönlü kapı kapsamaı:** Güvenlik kritik kararlarının %100'ü (`door_type: one-way` VEYA sınıflandırıcı tarafından işaretlenmiş dinamik bulgular) tam teknik ayrıntıyla bireysel olarak ortaya çıkar. Sınırısız.
- **Döndürme gidiş-dönüşü:** Kullanıcı `flip test-coverage-bookclub-form` yazar. Orijinal otomatik kabul edilen karar AskUserQuestion olarak yeniden açılır. Kullanıcının yeni seçimi Sessiz Kararlar bloğunda kalıcı olur (veya kullanıcı açık ortaya çıkarmaya çevirirse kaldırılır).
- **Aşama başına gözlemlenebilirlik:** `/plan-tune`, herhangi bir oturum için aşama başına AskUserQuestion sayılarını görüntüleyebilir, question-log.jsonl'nin yeni `phase` alanından okuyarak.
- **İlk çalıştırma azaltma:** Yeni kullanıcılar ilk gerçek skill çalışmadan önce ≤1 meta-prompt (lake tanıtımı) görür, V1'in 4'üne karşı (lake + telemetri + proaktif + yönlendirme).
- **İnsan yeniden çalıştırma:** Louise + Garry bağımsız nitel incelemeler, V1 ile aynı desen.

## V1'e bağımlılıklar

V1.1, V1'in altyapısına dayanır:
- `explain_level` yapılandırma anahtarı + önyazı yankı deseni (A4).
- Jargon listesi + Yazım Stili bölümü (V1.1'in kesinti dili ELI10 kurallarına saygı duymalıdır).
- V0 uyku negatif testleri (V1.1 de 5D psikografik makineriyi uyandırmaz).
- V1'in yakalanan Louise transkripti (kabul kriteri kalibrasyonu için temel).

V1.1 hiçbir V2 öğesine bağlı değildir (E1 alt taban kablolama, anlatım/vibe, vb.).

## İnceleme planı

- **Ön çalışma:** mevcut V0 verilerinden gerçek soru günlüğü dağılımını yakala. Kapsam #8 için kalibrasyon girdisi olarak kullan.
- **CEO incelemesi.** Önerme meydan okuması: adımlama doğru düzeltme mi, yoksa V1.1 aşamaları tamamen kaldırmayı mı düşünmeli? (Örn., CEO + Tasarım + Mühendislik + DX'yi tek bir birleştirilmiş inceleme geçişine daralt.) Kapsam modu: muhtemelen SEÇİCİ GENİŞLEME (adımlama çekirdek, ilgili iyileştirmeler seçme).
- **Codex incelemesi.** V1.1 planında bağımsız geçiş. V1'in zorlandığı kontrol akışı değişikliği (Kapsam #4) üzerinde özellikle dikkatli olunması beklenir.
- **DX incelemesi.** Döndürme mekanizmasının DX'ine odaklan — `flip <id>` keşfedilebilir mi, komut sözdizimi doğal mı, hata yolu net mi?
- **Mühendislik incelemesi ×N.** V1 ile aynı şekilde birden fazla geçiş beklenir.

## V1.1'de dokunulmayanlar

V2 öğeleri ertelenmiş olarak kalır:
- Karışıklık sinyali algılama
- 5D psikografik odaklı skill uyarlaması (V0 E1)
- /plan-tune anlatım + /plan-tune vibe (V0 E3)
- Skill başına veya konu başına açıklama seviyeleri
- Takım profilleri
- AST tabanlı "teslim edilen özellikler" metriği