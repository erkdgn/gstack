# gstack-plan: Tam İnceleme Süzgeci

Kullanıcı bir Claude Code projesi planlamak istediğinde orkestratör tarafından enjekte edilir. Mevcut CLAUDE.md'ye ekleyin.

## Planlama Boru Hattı
1. CLAUDE.md dosyasını okuyun ve proje bağlamını anlayın.
2. Bir tasarım belgesi üretmek için /office-hours çalıştırın (problem bildirimi, ön koşullar, alternatifler).
3. Tasarımı incelemek için /autoplan çalıştırın (CEO + mühendislik + tasarım + DX incelemeleri + codex adversiyel).
4. Orkestratörün daha sonra referans verebileceği bir dosyaya son incelenen planı kaydedin.
   Şuraya yazın: geçerli depoda plans/<project-slug>-plan-<date>.md.
   Tasarım belgesini, tüm inceleme kararlarını ve uygulama sırasını dahil edin.
5. Orkestratöre raporlayın:
   - Plan dosya yolu
   - Tasarlanan şeylerin ve ana kararların tek paragraflık özeti
   - Kabul edilen kapsam genişletmeleri listesi (varsa)
   - Önerilen sonraki adım (genellikle: uygulamak için gstack-full ile yeni bir oturum spawn edin)

Hiçbir şey uygulamayın. Bu yalnızca planlamadır.
Orkestratör plan bağlantısını kendi bellek/bilgi deposuna kaydedecektir.