# 1-7. Konular Genel Değerlendirme Sınavı

Bu doküman; 1'den 7'ye kadar işlenen tüm konuların pekiştirilmesi amacıyla gerçekleştirilen kapsamlı senaryo tabanlı sınavın sorularını, öğrenci cevaplarını ve değerlendirmesini içerir.

---

## 📋 Sınav Senaryosu
**Proje:** Akıllı Endüstriyel Sıvı Dozajlama ve Karıştırıcı Cihazı (STM32 Tabanlı).

```mermaid
flowchart LR
    subgraph GIRDILER["Güvenlik ve Sensörler"]
        A["Acil Stop (A)"] --> OR["OR Kapısı"]
        B["Kazan Kapağı (B)"] --> OR
        OR --> NOT["NOT (Evirici)"]
        T["Sıcaklık (T > 60°C)"] --> MCU
    end

    subgraph MCU["STM32 MCU (Cross-Compiler / STM32CubeIDE)"]
        REG["VALVE_REG = 0x4A\nconst Sabitler (Flash)\nuint16_t Dizi (RAM)"]
    end

    NOT -- "/SHUTDOWN (Active-LOW)" --> MOT["Motor Sürücü"]
    MCU -- "PWM (AF Modu - 250 Hz)" --> KAR["Karıştırıcı Motor"]
    MCU -- "STEP (1600 Hz) + DIR" --> DOZ["Dozajlama Step Motoru"]
    MCU -- "GPIOB->ODR (PB0=3.3V)" --> VANA["Sıvı Valfi"]
```

---

## 1. Soru: Sayı Sistemleri ve Bit İşlemleri
- **Senaryo:** Valf kontrol yazmacı `VALVE_REG = 0x4A`.
- **Soru:**
  1. `0x4A` binary karşılığı ve hangi bitlerin HIGH olduğu?
  2. Diğer bitleri bozmadan Bit 2'yi HIGH yapacak operatör, `(1 << ?)` maskesi ve yeni Hex değeri?
- **Öğrenci Yanıtı:**
  1. `0100 1010` $\rightarrow$ Pin 1, 3, 6 HIGH. *(Doğru)*
  2. OR işlemi, `(1 << 2)` maskesi, yeni değer `0x4E`. *(Doğru)*
- **Değerlendirme:** Tam puan.

---

## 2. Soru: Lojik Kapılar ve Active-LOW Mantığı
- **Senaryo:** Acil Stop Butonu (A) veya Kazan Kapağı (B) açıldığında tehlike ($Y=1$) üreten devre ve motor sürücünün $\overline{\text{SHUTDOWN}}$ pini.
- **Soru:** Hangi kapı kullanılmalıdır? Tehlike anında motor sürücünün kapanması için araya ne eklenmelidir?
- **Öğrenci Yanıtı:**
  - OR kapısı kullanılmalıdır ($Y = A + B$).
  - Active-LOW pini $0\text{V}$ ile tetiklendiği için araya NOT (evirici) kapısı eklenmelidir. *(Doğru)*
- **Değerlendirme:** Tam puan.

---

## 3. Soru: Elektriksel Sinyaller (PWM ve Step Motor)
- **Hesaplamalar:**
  1. $250\text{ Hz}$ PWM sinyali periyodu: $T = \frac{1}{250} = \mathbf{4\text{ ms}}$. *(Öğrenci: 4 ms - Doğru)*
  2. $3.3\text{V}$, $\%60$ Duty Cycle ortalama voltaj: $3.3 \times 0.60 = \mathbf{1.98\text{V}}$. *(Öğrenci: 1.98 V - Doğru)*
  3. $200$ adımlık motorun $4$ tur dönmesi için darbe adedi: $4 \times 200 = \mathbf{800\text{ darbe}}$. *(Öğrenci: 800 - Doğru)*
  4. Bu hareketin $0.5\text{ saniyede}$ bitmesi için frekans: $\frac{800}{0.5} = \mathbf{1600\text{ Hz}}$. *(Öğrenci: 1600 Hz - Doğru)*
- **Değerlendirme:** Tam puan.

---

## 4. Soru: Gömülü C Programlama Temelleri ve Bellek Güvenliği
- **Soru:**
  1. `uint8_t sayac = 250;` binlerce kez sayarsa $255$'ten sonra ne olur? Taşmayı önleyecek en az yer kaplayan tip nedir?
  2. `uint16_t dizi[4];` RAM'de kaç bayt yer tutar? `dizi[4] = 65;` yazılırsa ne olur?
- **Öğrenci Yanıtı:**
  1. Hata vermez, $0$ olur. `uint16_t` kullanılmalıdır. *(Doğru)*
  2. $4 \times 2\text{ bayt} = \mathbf{8\text{ bayt}}$ yer tutar. Sınır aşımında hata vermez, bellek bozulması (Memory Corruption) olur. *(Doğru)*
- **Değerlendirme:** Tam puan.

---

## 5. Soru: Algoritma Kurma ve Karar Yapıları
- **Soru:** Sıcaklık kontrolü ($60^\circ C$) akış diyagramında hangi şekille gösterilir?
- **Öğrenci Yanıtı:** Baklava dilimi (Eşkenar Dörtgen). *(Doğru)*
- **Değerlendirme:** Tam puan.

---

## 6. Soru: Geliştirme Ortamları ve İşletim Sistemleri
- **Soru:** x86'dan ARM'a kod derleyen derleyici türü, STM32 resmi IDE'si ve kamera/ağ için tercih edilecek işletim sistemi türü nedir?
- **Öğrenci Yanıtı:**
  1. Cross Compiler. *(Doğru)*
  2. STM32CubeIDE. *(Doğru)*
  3. Gömülü Linux. *(Doğru)*
- **Değerlendirme:** Tam puan.

---

## 7. Soru: Mikrodenetleyici Mimarisi ve Donanım Kontrolü
- **Soru:** Sabit verileri Flash'ta saklayıp RAM tasarrufu sağlayan anahtar kelime nedir? PWM çıkışı için GPIO modu ne olmalıdır ve `GPIOB->ODR |= (1 << 0);` donanımda ne yapar?
- **Öğrenci Yanıtı:**
  1. `const` kullanılır, Flash'ta depolanır. *(Doğru)*
  2. Timer bağlantısı için Alternate Function (AF) seçilir. PB0 pini $0\text{V}$'tan $3.3\text{V}$'a çıkar (HIGH olur). *(Doğru)*
- **Değerlendirme:** Tam puan.

---

## 🏆 Sonuç: BAŞARILI (100 / 100)
Öğrenci donanımsal mantığı, sayı sistemlerini, bellek mimarisini ve elektriksel hesaplamaları ezberlemeden, mühendislik mantığıyla kavramıştır.
