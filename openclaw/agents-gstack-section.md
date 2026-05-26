## Kodlama Görevleri (gstack)

### Kurallar (pazarlıksız)

1. **Her zaman spawn edin, asla yönlendirmeyin.** Kullanıcı HERHANGİ bir gstack becerisi kullanmayı istediğinde, sessions_spawn aracılığıyla HER ZAMAN bir Claude Code oturumu spawn edin. Kullanıcıya Claude Code'u kendisinin açmasını söylemeyin. Asla "bunun Claude Code'da çalışması gerekiyor" demeyin. Asla "bunun için Claude Code'u açmanız gerekecek" demeyin. Sadece yapın.

2. **Depoyu çözün.** Kullanıcı bir depo veya proje adlandırırsa, çalışma dizinini o depo yoluna ayarlayın. Depo yolu bilinmiyorsa, hangi depo olduğunu sorun — kullanıcıya Claude Code'u açmasını söylemek için ertelemeyin.

3. **Autoplan uçtan uca çalışır.** Özellikle /autoplan için: oturumu spawn edin, tam inceleme boru hattını (CEO → tasarım → mühendislik) çalıştırmasına izin verin ve bittiğinde planı burada sohbette raporlayın. Kullanıcı daha sonra bulabilsin diye planı belleğe yazın. Kullanıcının asla Telegram'dan ayrılması gerekmemeli.

### Sevk Yönlendirmesi

Kodlama işi istendiğinde, sevk katmanını seçin:

**BASİT:** "bu yazım hatasını düzelt," "o yapılandırmayı güncelle," tek dosya değişiklikleri
→ sessions_spawn(runtime: "acp", prompt: "<just the task>")

**ORTA:** çok dosyalı özellikler, yeniden düzenlemeler, beceri düzenlemeleri
→ sessions_spawn(runtime: "acp", prompt: "<gstack-lite content>\n\n<task>")

**AĞIR:** belirli bir gstack metodolojisi gerektirir
→ sessions_spawn(runtime: "acp", prompt: "Load gstack. Run /qa https://...")
  Beceriler: /cso, /review, /qa, /ship, /investigate, /design-review, /benchmark, /gstack-upgrade

**TAM:** eksiksiz bir özellik oluştur, çok günlük kapsam, planlama + inceleme gerektirir
→ sessions_spawn(runtime: "acp", prompt: "<gstack-full content>\n\n<task>")
  Claude Code çalıştırır: /autoplan → uygula → /ship → raporla

**PLAN:** kullanıcı bir Claude Code projesi planlamak, bir özelliğin şartnamesini çıkarmak veya herhangi bir kod yazılmadan önce bir şey tasarlamak istiyor
→ sessions_spawn(runtime: "acp", prompt: "<gstack-plan content>\n\n<task>")
  Claude Code çalıştırır: /office-hours → /autoplan → plan dosyasını kaydeder → raporlar
  Plan bağlantısını bellek/bilgi deposuna kaydedin.
  Kullanıcı uygulamaya hazır olduğunda, planı işaret eden yeni bir TAM oturum spawn edin.

### Karar Hevristiği

- <10 satır kodda yapılabiliyor mu? → **BASİT**
- Birden fazla dosyaya dokunuyor ama yaklaşık açık mı? → **ORTA**
- Kullanıcı belirli bir beceri adlandırıyor mu (/cso, /review, /qa)? → **AĞIR**
- "Upgrade gstack", "update gstack" → `Run /gstack-upgrade` ile **AĞIR**
- Bir özellik, proje veya hedef mi (bir görev değil)? → **TAM**
- Kullanıcı henüz uygulamadan bir şey PLANLAMAK istiyor mu? → **PLAN**