# 4. Konu: Programlama Temelleri (Gömülü C)

Bu doküman; gömülü sistemlerde donanıma doğrudan temas eden C programlama temellerini, bellek yönetimini (RAM, Flash, Stack), taşma mekanizmalarını, dizileri ve soru-cevap kayıtlarını içerir.

---

## 4.1 Bellek ve Sabit Genişlikli Veri Tipleri (`stdint.h`)

Farklı mimarilerde `int` boyutlarının değişmesi (8-bit MCU'da 16 bit, 32-bit MCU'da 32 bit) riskine karşı standart **`<stdint.h>`** kütüphanesi kullanılır:

| Tip | Anlamı | Bit / Bayt | Değer Aralığı |
|---|---|---|---|
| **`uint8_t`** | Unsigned 8-bit | 8 bit (1 Bayt) | $0$ ile $255$ |
| **`int8_t`** | Signed 8-bit | 8 bit (1 Bayt) | $-128$ ile $+127$ |
| **`uint16_t`** | Unsigned 16-bit | 16 bit (2 Bayt) | $0$ ile $65.535$ |
| **`int16_t`** | Signed 16-bit | 16 bit (2 Bayt) | $-32.768$ ile $+32.767$ |
| **`uint32_t`** | Unsigned 32-bit | 32 bit (4 Bayt) | $0$ ile $4.294.967.295$ |

### Taşma (Overflow) ve Alttan Taşma (Underflow) Mekanizması
- $255$ (`1111 1111`) sayısına $1$ eklendiğinde $256$ (`1 0000 0000`) olur. 9. bit 8 bitlik RAM hücresine sığmaz (elde/carry olur), geriye `0000 0000` kalır $\rightarrow$ Sonuç **$0$** olur.
- C dilinde ve CPU donanımında **çalışma zamanında hiçbir hata veya uyarı verilmez; program durmaz.**
- $0$ değerinden $1$ çıkarıldığında ise geriye doğru takla atar ve sonuç **$255$** olur (Underflow).

```text
  Bellek Genişliği Karşılaştırması:
  uint8_t  : [  1 Bayt  ] (8 bit)
  uint16_t : [  1 Bayt  ][  1 Bayt  ] (16 bit)
  uint32_t : [  1 Bayt  ][  1 Bayt  ][  1 Bayt  ][  1 Bayt  ] (32 bit)

  uint8_t Overflow (Taşma) & Underflow (Kilometre Sayacı Mantığı):
    255:    [1][1][1][1][1][1][1][1]
         +1
    --------------------------------
    256: [1][0][0][0][0][0][0][0][0]  --> Sonuç: 0  (9. Carry biti 8-bitlik kutuya sığmaz, düşer!)
          ^-- Taşan Bit

    0 - 1  --->  Geriye sarar  --->  [1][1][1][1][1][1][1][1] = 255 (Hata verilmez!)
```

### Soru - Cevap
> **Soru:**  
> 1) $0-1000$ arası analog sensör değerini saklamak için en az yer kaplayan doğru `<stdint.h>` tipi hangisidir?  
> 2) `uint8_t sayac = 255; sayac = sayac + 1;` sonucu ne olur?  
> **Öğrenci Cevabı:** 1) `uint16_t` kullanılmalı. 2) $0$ olur (Bit seviyesinde açıklama istendi).  
> **Açıklama:** 9 bitlik taşma ve donanım ALU davranışı detaylandırıldı.  
> **Pekiştirme Sorusu:** `uint8_t deger = 0; deger = deger - 1;` yapılırsa hata verir mi, ne olur?  
> **Öğrenci Yanıtı:** "Hata vermez mi, uint8 en küçük 0 olabilir?"  
> **Rehberlik:** C ve işlemcinin runtime hata fırlatmadığı, kilometre sayacı gibi geriye sarıp `1111 1111` olacağı açıklandı.  
> **Öğrenci Düzeltmesi:** **255 olur.**

---

## 4.2 Değişken İsimlendirme, Bileşik Atamalar ve `const`

- **`const` Belirteci:** Değeri değiştirilemeyen sabitlerdir.
  - Normal değişkenler kısıtlı **RAM**'de saklanırken; `const` değişkenler **Flash (ROM / Program Belleği)** içinde saklanır. Kısıtlı RAM korunmuş olur.
- **Bileşik Atamalar:**
  - `x |= (1 << n);` $\rightarrow$ $n$. biti HIGH yapar.
  - `x &= ~(1 << n);` $\rightarrow$ $n$. biti LOW yapar.

```text
  +-----------------------------------+     +-----------------------------------+
  |  FLASH BELLEK (ROM - Kalıcı)      |     |  RAM (SRAM - Geçici / Kısıtlı)    |
  |  -------------------------------  |     |  -------------------------------  |
  |  * Derlenmiş Program Kodu         |     |  * Normal Değişkenler (sayac)     |
  |  * const uint8_t PIN = 5;         |     |  * Diziler / Tamponlar (Buffer)   |
  |  * Değiştirilemez Sabit Tablolar  |     |  * Stack (Fonksiyon Çağrıları)    |
  +-----------------------------------+     +-----------------------------------+
```

### Soru - Cevap
> **Soru:** 1) `uint8_t PIN = 5;` yerine `const uint8_t PIN = 5;` yazmanın donanımsal avantajı nedir? 2) `bayrak = 0b00000001; bayrak |= (1 << 2);` binary sonucu ne olur?  
> **Öğrenci Cevabı:** 1) `const` yapıldığında ROM'da yer tutar ve RAM'den yemez. 2) `0000 0101` olur.  
> **Değerlendirme:** Doğru.

---

## 4.3 Karar Yapıları (`if-else` ve `switch-case`)

- `=` operatörü atamadır (değer yazar). `==` operatörü eşitlik karşılaştırmasıdır.
- `if (buton = 1)` yazılırsa değişkene 1 atanır ve ifade her zaman TRUE (Doğru) kabul edilir!
- `switch-case` gömülü sistemlerde Durum Makinelerini (Finite State Machine - FSM) kurmak için kullanılır. `break;` unutulursa alttaki case'e doğrudan akar (fall-through).

```mermaid
stateDiagram-v2
    [*] --> BEKLEME: Sistem Açıldı
    BEKLEME --> CALISMA: Başlat Butonu (break var)
    CALISMA --> HATA: Sensör Arızası
    HATA --> BEKLEME: Reset
```

```text
  switch-case'de "break;" Unutulursa (Fall-Through Hatası):
    case 1:
        motor_ac();
        // break; UNUTULDU!  --+
    case 2:                    | Akış doğrudan aşağı sızar!
        alarm_cal();  <--------+ (İstenmeyen ikinci işlem de çalışır)
        break;
```

### Soru - Cevap
> **Soru:** 1) `if (buton = 1)` ile `if (buton == 1)` farkı nedir? 2) `case` sonuna `break;` konulmazsa ne olur?  
> **Öğrenci Cevabı:** 1) `=` kullanınca buton durumu 1 olarak atanır (if her zaman çalışır), `==` karşılaştırır. 2) `break` olmazsa 2. case'den devam eder.  
> **Değerlendirme:** Doğru.

---

## 4.4 Döngüler ve Sonsuz Döngü (`while(1)`)

- Mikrodenetleyicilerde işletim sistemi olmadığı için program `main()` dışına çıkamaz. Çıkarsa tanımsız belleğe atlar ve çöker.
- Bu yüzden her gömülü program bir **Sonsuz Döngü (Super-Loop / Ana Döngü)** içerir: `while(1) { ... }`.
- **Bloklayan Kod (Blocking Code):** `while (sensor_hazir == 0);` gibi donanım yanıtı bekleyen kodlar, donanım arızalanırsa işlemciyi sonsuza kadar kilitler.

```mermaid
flowchart TD
    A(["Güç Verildi / Reset"]) --> B["main() Başlar"]
    B --> C["Donanım Ayarları (GPIO, Timer, UART Init)"]
    C --> D{"while(1) Super-Loop"}
    D --> E["Sensör Oku"]
    E --> F{"Bloklayan Satır: while(hazir == 0);"}
    F -- "Kablo Koptu (Hep 0)" --> F
    F -- "Veri Geldi (1)" --> G["Çıkışları Güncelle"]
    G --> D
```

### Soru - Cevap
> **Soru:** 1) `main()` içinde neden mutlaka `while(1)` bulunur? 2) `while(veri_geldi_mi() == 0);` satırında kablo koparsa sistem ne yapar?  
> **Öğrenci Cevabı:** 1) `main()` 1 kere çalışıp biterse kod tanımsız yerde biter. 2) `while` içi sürekli true olacağı için sonsuz döngüye girip sistemi dondurur.  
> **Değerlendirme:** Doğru.

---

## 4.5 Fonksiyonlar ve Yığın (Stack) Bellek Mantığı

- Bir fonksiyon çağrıldığında işlemci o anki dönüş adresini ve fonksiyonun yerel değişkenlerini RAM'deki **Stack (Yığın)** alanına kaydeder.
- Mikrodenetleyicilerde Stack alanı çok dardır (örn: 512B - 1KB).
- Kendi kendini çağıran fonksiyonlar (**Recursion**) Stack'i çok hızlı tüketerek **Stack Overflow** yaratır ve sistemi çökertir. Bu nedenle gömülü sistem standartlarında (MISRA-C) özyineleme yasaktır.

```text
  RAM Yerleşimi ve Stack Overflow (Yığın Taşması):
  +-------------------------------------+  Düşük Adres
  | Global Değişkenler / Diziler (Data) |
  +-------------------------------------+
  |         Boş RAM Alanı               |  <-- ÇARPIŞMA! (Stack aşağı taşarsa
  |               ^                     |      global değişkenleri ezer!)
  |               |  (Stack Büyür)      |
  +---------------+---------------------+
  | fonk_B() yerel değişken + dönüş adr |
  | fonk_A() yerel değişken + dönüş adr |
  | main()   yerel değişkenler          |  Yüksek Adres (Stack Tabanı)
  +-------------------------------------+
```

### Soru - Cevap
> **Soru:** 1) Girdi almayan ve değer döndürmeyen `buzzer_sustur` fonksiyon başlığı nedir? 2) Gömülüde recursive fonksiyonların yasaklanmasının donanımsal sebebi nedir?  
> **Öğrenci Cevabı:** 1) `void buzzer_sustur(void)`. 2) Mikrodenetleyicilerde stack boyutu çok küçük olduğu için her çağrıda stack overflow oluşur.  
> **Değerlendirme:** Doğru.

---

## 4.6 Diziler (Arrays) ve Buffer Mantığı

- Bellekte aynı türden verilerin **ardışık (yan yana)** dizildiği yapılardır.
  $$\text{RAM Alanı} = \text{Eleman Sayısı} \times \text{Tip Boyutu}$$
- Haberleşmede gelen baytlar dizilere (Buffer / Tampon) toplanır.
- C dili dizi sınır denetimi (bounds check) yapmaz. 10 elemanlı dizinin 10. indeksine (`[10]`) yazılırsa yanındaki bellek hücresinin üzerine yazar (**Buffer Overflow / Memory Corruption**).

```text
  uint16_t adc_kayitlari[10];  (10 eleman x 2 Bayt = 20 Bayt ardışık RAM)
  +--------+--------+--------+-----+--------+ | +------------------+
  | [0] 2B | [1] 2B | [2] 2B | ... | [9] 2B | | | Komşu Değişken   |
  +--------+--------+--------+-----+--------+ | +------------------+
  0x2000   0x2002   0x2004         0x2012     | 0x2014 (Sınır Dışı!)
                                              |
     adc_kayitlari[10] = 500;  ---------------+---> Komşu veriyi ezer!
```

### Soru - Cevap
> **Soru:** 1) `uint16_t adc_kayitlari[10];` RAM'de kaç bayt yer tutar? İlk ve son eleman indeksleri nedir? 2) `adc_kayitlari[10] = 500;` yazılırsa ne olur?  
> **Öğrenci Cevabı:** 1) 20 bayt, ilk eleman `[0]`, son eleman `[9]`. 2) Hata fırlatmaz, dizinin hemen yanındaki adrese yazar; bu da bellek bozulması olur.  
> **Değerlendirme:** Doğru.

---

## 🎯 4. Bölüm Genel Değerlendirme (Sıcaklık Tamponu Senaryosu)

**Senaryo:** `uint16_t sicaklik_tamponu[5] = {24, 26, 25, 28, 27};` dizisinin ortalamasını alıp $30^\circ C$'nin üzerindeyse fanı açacak (`fan_ac()`), değilse kapatacak (`fan_kapat()`) algoritma.

1. **Soru:** Toplam değişkeni neden `uint8_t` olamaz, hangi tip seçilmelidir?
   - **Öğrenci Yanıtı:** Bilmiyorum $\rightarrow$ 5 elemanın $70^\circ C$ olduğu senaryoda $5 \times 70 = 350 > 255$ taşma (overflow) durumu analiz edildi.
   - **Öğrenci Düzeltmesi:** **`uint16_t`** kullanılmalıdır. *(Doğru)*
2. **Soru:** Bu akışı kuran döngü ve if-else taslağını yazınız.
   - **Öğrenci İlk Kodu:**
     ```c
     uint16_t toplam = 0;
     for (int i = 0; i <= 5; i++) { // DİKKAT: i <= 5
         toplam += arr[i];
     }
     ortalama = toplam / 5;
     if (ortalama > 30) { fan_ac(); } else { fan_kapat(); }
     ```
   - **Rehberlik / Hata Analizi:** `i <= 5` yazıldığında döngünün 6 kez döndüğü ve `i = 5` olduğunda `arr[5]` ile dizi sınırının aşıldığı (buffer overflow) gösterildi.
   - **Öğrenci Düzeltmesi:** `for (int i = 0; i < 5; i++)` olarak sınır düzeltildi. *(Doğru)*
