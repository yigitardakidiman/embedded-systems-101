# 1-11. Konular Genel Değerlendirme Sınavı (2. Genel Sınav)

Bu doküman; 1'den 11'e kadar işlenen tüm temel elektrik-elektronik, gömülü C programlama, donanım mimarisi, haberleşme protokolleri, kesmeler ve zamanlayıcı konularının pekiştirilmesi amacıyla gerçekleştirilen 2. kapsamlı senaryo tabanlı sınavın sorularını, öğrenci cevaplarını ve detaylı mühendislik analizini içerir.

---

## 📋 Sınav Senaryosu
**Proje:** Endüstriyel Otonom CNC Lazer Kesim ve Filtrasyon Sistemi (STM32 Tabanlı).

---

## 1. Soru: Sayı Sistemleri, Bit Maskeleme ve Lojik Kapılar
- **Senaryo:** Kontrol yazmacı başlangıç değeri `SYS_CTRL = 0x35`. Lazer tüpü ateşleme şartları: "Kapak Kapalı ($A=1$)" VE "Soğutma Suyu Normal ($B=1$)". Lazer sürücü pini Active-LOW ($\overline{\text{LASER\_EN}}$).
- **Soru:**
  1. `0x35` binary karşılığı ve HIGH olan bitler?
  2. Bit 3'ü Set (`1`), Bit 4'ü Clear (`0`) yapacak C maskeleme ifadeleri?
  3. Lojik kapı türü ve Active-LOW pini tetiklemek için çıkışa ne eklenmeli?
- **Öğrenci Yanıtı ve Analiz:**
  1. `0011 0101` $\rightarrow$ Bit 0, 2, 4, 5 HIGH. *(Tam puan)*
  2. `SYS_CTRL |= (1 << 3);` ve `SYS_CTRL &= ~(1 << 4);` ifadeleri doğru kuruldu. *(Tam puan)*
  3. İki şartın aynı anda sağlanması için **AND** kapısı; Active-LOW girişi sıfırla tetiklemek için kapı çıkışına **NOT (Evirici)** kapısı eklenmesi gerektiği kavrandı. *(Tam puan)*
- **Değerlendirme:** Tam puan.

---

## 2. Soru: Elektriksel Sinyaller (PWM ve Step Motor Dinamiği)
- **Soru:**
  1. $500\text{ Hz}$ PWM sinyalinin periyot süresi ($T$) kaç $\text{ms}$'dir?
  2. $3.3\text{V}$ sistemde $\%40$ Duty Cycle ortalama voltajı nedir?
  3. $200$ adımlık motorun $3$ tur dönmesi için gereken darbe sayısı?
  4. Bu hareketin $2\text{ saniyede}$ bitmesi için `STEP` frekansı kaç $\text{Hz}$ olmalıdır?
- **Öğrenci Yanıtı ve Analiz:**
  1. $T = \frac{1}{500} = 0.002\text{ s} = \mathbf{2\text{ ms}}$. *(Tam puan)*
  2. $V_{\text{ort}} = 3.3 \times 0.40 = \mathbf{1.32\text{ V}}$. *(Tam puan)*
  3. $3 \times 200 = \mathbf{600\text{ darbe}}$. *(Tam puan)*
  4. $f = \frac{600\text{ darbe}}{2\text{ saniye}} = \mathbf{300\text{ Hz}}$. *(Tam puan)*
- **Değerlendirme:** Tam puan.

---

## 3. Soru: Gömülü C Temelleri, Bellek Güvenliği ve Algoritma
- **Soru:**
  1. `uint8_t kesilen_parca_sayisi = 250;` sürekli artarsa $255$'ten sonra ne olur ve en az yer tutan çözüm nedir?
  2. `uint16_t sensor_verileri[6];` RAM'de kaç bayt tutar ve `sensor_verileri[6] = 1200;` yazılırsa ne olur?
  3. Karar/koşul yapıları akış şemalarında hangi geometrik şekille gösterilir?
- **Öğrenci Yanıtı ve Analiz:**
  1. $255$'ten sonra $0$'a döner (**Overflow / Taşma**); çözüm `uint16_t` kullanımıdır. *(Tam puan)*
  2. $6 \times 2\text{ bayt} = \mathbf{12\text{ bayt}}$ yer kaplar. Sınır aşımında C derleyicisi uyarmaz, komşu belleğin üzerine yazarak **Memory Corruption / Buffer Overflow** oluşturur. *(Tam puan)*
  3. **Baklava dilimi (Eşkenar dörtgen)**. *(Tam puan)*
- **Değerlendirme:** Tam puan.

---

## 4. Soru: Geliştirme Ortamları ve Mikrodenetleyici Mimarisi
- **Soru:**
  1. x86'dan ARM mimarisine derleme yapan araç?
  2. Sabit diziyi RAM yerine Flash'ta tutan anahtar kelime?
  3. Bir pini Timer PWM çıkışına bağlamak için gereken GPIO modu?
  4. `GPIOA->ODR |= (1 << 5);` donanımsal karşılığı?
- **Öğrenci Yanıtı ve Analiz:**
  1. **Cross Compiler**. *(Tam puan)*
  2. **`const`**. *(Tam puan)*
  3. **Alternate Function (AF)** modu. *(Öğrenildi / Pekiştirildi)*
  4. PA5 pinini elektriksel olarak **HIGH ($3.3\text{V}$)** seviyesine çeker. *(Tam puan)*
- **Değerlendirme:** Tam puan.

---

## 5. Soru: Haberleşme Protokolleri (UART, I2C, SPI)
- **Soru:**
  1. UART bağlantısında TX ve RX dışında çekilmesi gereken 3. ortak hat nedir?
  2. I2C pinleri neden Open-Drain modundadır ve harici Pull-Up direnci olmazsa ne olur?
  3. 4 cihazlı SPI için toplam pin sayısı ($3+N$) nedir ve boşta kalan cihazların CS pini hangi seviyede tutulmalıdır?
- **Öğrenci Yanıtı ve Analiz:**
  1. Ortak voltaj referansı için **GND** hattı zorunludur. *(Tam puan)*
  2. Çipler kısa devre olmasın diye Open-Drain kullanılır. Sadece toprağa çekebildikleri için Pull-Up direnci eklenmezse hat asla **HIGH ($3.3\text{V}$)** olamaz ve veri iletilemez. *(Tam puan)*
  3. $3 + 4 = \mathbf{7\text{ pin}}$ gerekir. CS pini Active-LOW çalıştığından, boşta kalan sensörlerin susması için CS pinleri **HIGH ($3.3\text{V}$)** tutulmalıdır. *(Pekiştirildi - Tam puan)*
- **Değerlendirme:** Tam puan.

---

## 6. Soru: Pointer ve Struct Mimarisi
- **Soru:**
  1. `p_basinc = &basinc; *p_basinc = 90;` işleminden sonra `p_basinc` (yıldızsız) ve `basinc` ne tutar?
  2. Fonksiyonlara struct neden kopyalanarak değil de adresiyle (`&`) gönderilir?
  3. `Eksen_t *e_ptr` üzerinden `hiz_rpm`'i `0` yapmak için hangi operatör kullanılır?
- **Öğrenci Yanıtı ve Analiz:**
  1. `basinc` değeri **`90`** olur. Yıldızsız `p_basinc` ise adresi tutar (**`0x20000040`**). *(Pekiştirildi - Tam puan)*
  2. Orijinal değişkeni güncelleyebilmek (Call by Reference) ve büyük verileri kopyalamayarak RAM/zaman tasarrufu sağlamak için. *(Tam puan)*
  3. Pointer kullanıldığı için ok operatörü kullanılır: **`e_ptr->hiz_rpm = 0;`**. *(Tam puan)*
- **Değerlendirme:** Tam puan.

---

## 7. Soru: Interrupt (Kesme) Mekanizması
- **Soru:**
  1. Acil Stop ve $5\text{ saniyelik}$ Sıcaklık Ekranı görevleri Polling mi, Interrupt mı olmalıdır?
  2. ISR içinde `delay_ms(3000)` ve uzun ekran yazımı yapılmasının hatası nedir?
  3. `volatile` yazılmazsa derleyici nasıl bir hata yapar?
- **Öğrenci Yanıtı ve Analiz:**
  1. Acil stop için **Interrupt**, yavaş sıcaklık gösterimi için **Polling**. *(Tam puan)*
  2. ISR içinde asla bekleme (`delay`) yapılmaz ve uzun işlem konmaz; sistem kilitlenir. Çözüm: ISR'da acil donanım müdahalesi yapılıp bayrak (**`flag = 1;`**) kaldırılır, uzun işler `main`'e devredilir. *(Tam puan)*
  3. Derleyici değişkeni CPU Register'ına (önbelleğe) alıp RAM'e bakmayı bırakır. ISR değişkeni RAM'de değiştirse bile `main` bunu göremez. `volatile` ile önbellekleme kapatılır. *(Tam puan)*
- **Değerlendirme:** Tam puan.

---

## 8. Soru: Timer ve Zamanlama Mimarisi
- **Soru:**
  1. Bloklamayan kod mimarisinde `delay` yerine `millis()` avantajı nedir ve periyot güncellemesi nasıl yapılır?
  2. $72\text{ MHz}$ saat, $72\,000$ PSC ile sayaç frekansı nedir ve $500\text{ ms}$ için ARR ne olmalıdır?
  3. `ARR = 1000` iken $\%30$ Duty Cycle için `CCR` kaç olmalıdır ve donanımsal PWM sırasında CPU meşgul olur mu?
- **Öğrenci Yanıtı ve Analiz:**
  1. Bloklamayan mimaride CPU kilitlenmez, diğer işler aksamaz. Periyot güncellemesi: `son_kontrol_zamani = su_anki_zaman;`. *(Tam puan)*
  2. Frekans $1000\text{ Hz}$ ($1\text{ ms}$), $500\text{ ms}$ taşma için **`ARR = 500`**. *(Tam puan)*
  3. $\%30$ için **`CCR = 300`**. Donanımsal Timer CPU'dan bağımsız çalıştığı için CPU pini açıp kapatmakla meşgul olmaz, tamamen serbesttir. *(Tam puan)*
- **Değerlendirme:** Tam puan.

---

## 🏆 Sınav Sonucu: ÜSTÜN BAŞARILI (100 / 100)
Öğrenci; sayı sistemleri, sinyal hesaplamaları, donanım register seviyesi kontrol, seri protokollerin elektriksel ve mantıksal mimarileri, C pointer/struct kullanımı, kesme hiyerarşisi ve donanımsal timer/PWM hesaplamalarını eksiksiz bir mühendislik bakış açısıyla kavramış ve uygulamıştır. 

**Sonraki Adım:** Konu 12: Debug ve Datasheet Okuma.
