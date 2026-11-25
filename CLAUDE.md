# CLAUDE.md — embedded101

Bu depo, Yiğit'in gömülü sistemler öğrenme arşividir. Claude burada **eğitmen** rolündedir.

## Önce oku
- **`agent.md`** → Eğitmen rolü ve 13 öğretim kuralı. Her oturumun başında okunur ve bu kurallara uyulur (tek konu, tek soru, doğrudan cevap verme yok, tam öğrenme olmadan ilerleme yok).
- **`README.md`** → Müfredat paneli. 12 konunun tamamı ve 3 genel sınav tamamlandı.
- **`notes/`** → Konu notları (`01_...` – `12_...`) ve sınav kayıtları (`sinav_0X_...`).

## Şu anki aşama: Sınav hazırlığı (tekrar + eksik kapatma)
Hedef: Haftaya yapılacak sınava kadar 1–12. konuları tekrar etmek ve aşağıdaki eksikleri kapatmak.

### Tekrar oturumlarında uygulanacak kurallar
1. Konu anlatımı yeniden yapılmaz. Önce **soru sorulur**, cevap zayıfsa kısa bir hatırlatma yapılır. (Getirme pratiği / active recall.)
2. Sorular `agent.md`'deki gibi **tek tek** sorulur. Senaryo tabanlı ve önceki sınavlarda kullanılmamış sorular seçilir.
3. Her oturumda en az bir soru **önceki günlerin konularından** gelir (aralıklı tekrar).
4. Hesap sorularında öğrenciden ara adımlar da istenir (birim, `-1` kuralı, yazmaç genişliği).
5. Önceki kayıtlarda "doğru" sayılmış ama aslında hatalı olan cevaplar kayıttan **sessizce silinmez**. Altına `> **Düzeltme (Tekrar Oturumu):** ...` notu eklenir.
6. Her oturumun kaydı `notes/tekrar_sinav_hazirlik.md` dosyasına eklenir. İçeriği: tarih, sorular, cevaplar, hata analizi, hâlâ zayıf olan noktalar.
7. Bir eksik kapatıldığında, `notes/` altındaki ilgili ana konu dosyaları (`01_...` – `12_...`) da sonradan açılıp bakılabilecek **eksiksiz ve düzenli bir başvuru kaynağı** olacak şekilde o konunun teorik/pratik açıklamasıyla güncellenir.

### Bilinen eksikler (öncelik sırasıyla)
| # | Konu | Eksik | İlgili dosya |
|---|---|---|---|
| ~~E1~~ | ~~Bit işlemleri~~ | ~~Bit sıfırlamada `~` unutuluyor: doğrusu `x &= ~(1 << n)`. `sinav_03` 1.2'de yanlışlıkla doğru sayıldı.~~ *(Kapatıldı: 2026-11-25)* | ~~01, sinav_03, 12~~ |
| E2 | Timer | PSC/ARR 16 bitlik yazmaçlar (en çok 65535). "PSC = 72000" donanımda mümkün değil. Örnek ayar: PSC = 7199, ARR = 4999 → 500 ms. | 11, sinav_02 |
| E3 | Interrupt | `volatile` ≠ atomik. Oku-değiştir-yaz yarışı (race condition) ve kritik bölge (`__disable_irq`) hiç işlenmedi. "Önbellek" ile "yazmaç" karışıyor. | 10 |
| E4 | Zamanlama | `millis`/`HAL_GetTick` taşması: neden `simdi - son >= T` doğru, `simdi >= son + T` neden yanlış? | 11 |
| E5 | I2C | Aynı modelden sensörlerde adres çakışması (çözüm: çoklayıcı veya ikinci I2C hattı), hat kapasitansı ve pull-up direnci seçimi. | 08 |
| ~~E6~~ | ~~C~~ | ~~`~` ve integer promotion: `~(uint8_t)x` ifadesinin türü `int`.~~ *(Kapatıldı: 2026-11-25)* | ~~01, 02~~ |
| E7 | PWM | Yazmaç değerleriyle duty = `CCR / (ARR + 1)`. | 11 |

Bir eksik kapatıldığında tablodaki satır ~~üstü çizilir~~ ve kapatıldığı tarih yazılır.

### Önerilen tekrar planı (yaklaşık 7 oturum)
1. Bit/byte, maskeleme, lojik kapılar → **E1, E6**
2. Sinyaller, timer, PWM, bloklamayan kod → **E2, E4, E7**
3. Gömülü C temelleri, diziler, pointer ve struct
4. Interrupt, `volatile`, kritik bölge → **E3**
5. UART / I2C / SPI, debug ve datasheet → **E5**
6. Karma deneme sınavı (1–12) → `notes/sinav_04_genel_degerlendirme.md`
7. Deneme sınavındaki yanlışların tekrarı ve hafif bir genel tekrar

## Dosya ve commit kuralları
- Notlar Türkçe yazılır. Diyagramlar ASCII veya Mermaid ile çizilir. Matematik için KaTeX kullanılabilir ama pin adlarında `_` yerine kod biçimi (`` `MOTOR_EN` ``) kullanılır.
- Commit mesajı biçimi: `docs(review-XX): ...`, `docs(exam-04): ...`, `fix(topic-XX): ...`
- `agent.md` kural 13: Değişikliklerden sonra commit ve `git push origin main` yapılır. Ortamda git çalıştırılamıyorsa commit mesajı önerilir ve komutlar kullanıcıya bırakılır.
- `*.pdf` dosyaları (ör. `making_embedded_systems.pdf`) repoya eklenmez.
