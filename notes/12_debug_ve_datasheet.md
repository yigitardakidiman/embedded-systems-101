# 12. Konu: Debug (Hata Ayıklama) ve Datasheet / Register Okuma

Bu doküman; Gömülü sistemlerde Log tabanlı debug (`printf`) ile Donanımsal Debugger (SWD/JTAG) farkını, Breakpoint ve Adımlama (Step Over/Into) mekanizmalarını, Datasheet ile Reference Manual arasındaki farkları, Pin Çoklama (Alternate Function) yapısını ve Register tablolarından adres/bit maskeleme analizini içerir.

---

## 12.1 Hata Ayıklama Yöntemleri: Log/Serial (`printf`) vs. Donanımsal Debugger (SWD/JTAG)

- **Log Tabanlı Debug (`printf` / Serial Monitor):**
  - UART veya USB üzerinden seri hatta metin basarak yazılımın akışını izleme yöntemidir.
  - **Zamanlama Bozumu (Timing Overhead):** UART seri olarak bit bit veri gönderir. $115200\text{ baud}$ hızında $1$ bit $\approx 8.7\text{ }\mu\text{s}$ sürer. $1$ karakter ($10$ bit) yaklaşık $87\text{ }\mu\text{s}$ gerektirir. $15$–$20$ karakterlik bir log mesajı $1.5\text{ ms}$'nin üzerinde süre tüketir. Bu süre, mikrosaniyeler seviyesinde koşan Timer kesmelerini (ISR) ve gerçek zamanlı kontrol döngülerini altüst eder (*Heisenbug*).
  - **Kilitlenme Zaafı:** İşlemci `HardFault` alıp çöktüğünde UART mesajı basılamaz; hatanın kaynağı yakalanamaz.

- **Donanımsal Debugger (SWD / JTAG):**
  - Çekirdeğin içindeki donanımsal hata ayıklama birimine (ARM Cortex-M Debug Access Port) doğrudan bağlanır.
  - **SWD (Serial Wire Debug):** Yalnızca 2 hat kullanır (`SWDIO`: Veri, `SWCLK`: Saat).
  - **Canlı İzleme (Live Watch):** Koda fazladan hiçbir `printf` satırı yazmadan veya kod zamanlamasını değiştirmeden, debugger harici hat üzerinden işlemci çalışırken RAM'deki değişken adreslerini okuyabilir.

### Soru - Cevap ve Hata Analizi
> **Soru:** Hata ayıklarken `printf` kullanılırsa yazılım kodunda ve zamanlamasında ne gibi etkiler oluşur; donanımsal debugger (SWD) kullanıldığında koda dokunmak gerekir mi?  
> **Öğrenci Yanıtı:** `printf` kullanırsak kodun kesmelerine zaman ayırmak gerekecektir (zamanlamayı bozar), ama SWD kullanırsak kodun zamanlamasında hiçbir değişiklik yapmamıza gerek kalmaz.  
> **Değerlendirme:** Doğru.

---

## 12.2 Debug Mekanizmaları: Breakpoint, Step Over, Step Into ve Watch

1. **Breakpoint (Kesme Noktası):** İlgili kod satırında işlemcinin saat darbelerini donanımsal olarak durdurur (`Halt`). İşlemci durduğunda tüm CPU register'ları (`R0-R15`), RAM bellek ve çevre birimi durumları incelenebilir.
2. **Step Into (İçine Gir):** Bulunulan satırda bir fonksiyon çağrısı varsa (örn: `kalibre_et()`), o fonksiyonun gövdesine atlar ve içeride satır satır ilerler.
3. **Step Over (Üzerinden Geç):** Bulunulan satırdaki fonksiyon çağrısını tek hamlede arka planda çalıştırıp tamamlar ve mevcut dosyadaki bir sonraki satıra geçer.
4. **Watch Window (Değişken Takibi):** RAM'deki değişkenlerin anlık değerlerini listelemek ve izlemek için kullanılır.

### Soru - Cevap
> **Soru:** `float sicaklik = kalibre_et(ham_veri);` satırında durmuşken fonksiyonun iç hesaplamalarını görmek için hangi komut çalıştırılmalıdır? Diğer komuta basılırsa ne olur?  
> **Öğrenci Yanıtı:** `Step Into` kullanılmalıdır; eğer `Step Over` kullanılırsa o kısım atlanır (fonksiyon tek seferde çalışıp bir sonraki satırda durur).  
> **Değerlendirme:** Doğru.

---

## 12.3 Datasheet Anatomisi ve Pinout Okuma

- **Datasheet (Fiziksel & Elektriksel Kimlik):**
  - Çalışma voltajı sınırları ($V_{DD}$), pin başına maksimum akım ($I_{IO}$), toplam akım limiti, güç tüketimi ve sıcaklık aralıkları (**Absolute Maximum Ratings**).
  - Fiziksel kılıf tipleri ve bacak bağlantıları (Pinout).
- **Reference Manual (Yazılımsal & Mimari Kılavuz):**
  - Tüm dahili çevre birimlerinin (Timer, ADC, USART, SPI) çalışma mantığı.
  - Bütün kontrol ve durum yazmaçlarının (`Register`) adresleri, bit tanımları ve reset değerleri.
- **Pin Çoklama (Multiplexing / Alternate Function):**
  - Bacak sayısını optimize etmek için tek bir fiziksel pin birden fazla çevre birimine bağlanabilir (Örn: `PA9` hem GPIO, hem `USART1_TX`, hem `TIM1_CH2` olabilir).
  - Ancak bir fiziksel pin **aynı anda yalnızca tek bir fonksiyona** atanabilir.

### Soru - Cevap
> **Soru:** 1) GPIO maksimum akım sınırı ve çalışma voltajı için hangi dokümana bakılır? 2) `PA2` pini aynı anda hem buton okuma (GPIO) hem de UART TX olarak kullanılabilir mi?  
> **Öğrenci Yanıtı:** 1) Datasheet'e bakılmalıdır. 2) Hayır, sadece tek bir fonksiyon çalıştırılabilir; ya GPIO ya da UART olacaktır.  
> **Değerlendirme:** Doğru.

---

## 12.4 Register Tabloları Okuma ve Bit Maskeleme

Reference Manual'deki yazmaç tablolarında 4 temel unsur yer alır:
1. **Base Adres + Offset:** Fiziksel adres = $\text{Base} + \text{Offset}$. (Örn: `Base: 0x4001 2400`, `Offset: 0x08` $\to$ Adres: `0x4001 2408`).
2. **Reset Değeri:** Çipe elektrik verildiğinde veya reset atıldığında yazmaçtaki bitlerin başlangıç halidir (Örn: `0x0000 0000` tüm bitlerin `0` başladığını gösterir).
3. **Erişim Türü:** `rw` (Okunabilir / Yazılabilir), `r` (Salt Okunur - Durum Bayrakları).
4. **Bit Maskeleme Operatörleri:**
   - Biti `1` yapmak (Set): `YAZMAC |= (1 << BIT_NO);`
   - Biti `0` yapmak (Clear): `YAZMAC &= ~(1 << BIT_NO);`

### Soru - Cevap ve Hata Analizi
> **Soru:** `TIMx_CR1` tablosunda `Reset Değeri: 0x0000 0000` ve `Bit 0: CEN` ($0$: sayaç durduruldu, $1$: sayaç başlatıldı) verilmiştir.  
> 1) Enerji verildiğinde sayaç saymaya başlar mı?  
> 2) `CEN` bitini `1` yapmak için hangi operatör kullanılır?  
> **Öğrenci Yanıtı:** 1) İlk aşamada "evet başlar" yanıtı verildi. `Reset Değeri: 0x0000 0000` olduğu, dolayısıyla `CEN` bitinin başlangıçta `0` olduğu ve sayacın durduğu rehberlikle açıklandı ve öğrenci tarafından "sayaç durur" olarak düzeltildi. 2) `|=` (OR) operatörü kullanılır.  
> **Değerlendirme:** Tam öğrenme sağlandı.

---

## 12.5 Bölüm Sonu Değerlendirme Testi (Endüstriyel Gaz Kaçak Dedektörü Senaryosu)

1. **Soru ($100\text{ }\mu\text{s}$ Timer Döngüsünde `printf` Hatası ve Çözümü):**
   - **Öğrenci Yanıtı:** 1) Mesajın gönderilmesi milisaniyeler alır; $100\text{ }\mu\text{s}$'de bir kesmeye girildiği için kesmeler çakışır ve sistem kilitlenir. 2) Donanımsal debugger ve SWD arayüzü kullanılmalıdır; SWD ile kodun zamanlaması bozulmaz. *(Tam puan)*
2. **Soru (Breakpoint Sonrası `Step Into` vs. `Step Over`):**
   - **Öğrenci Yanıtı:** 1) `Step Into` kullanılmalıdır. 2) `alarm_kontrol` içine girilmez, fonksiyon tek hamlede çalıştırılıp bir alt satırdaki `if` kısmında durulur. *(Tam puan)*
3. **Soru (Datasheet Seçimi ve Pin Çoklama Sınırı):**
   - **Öğrenci Yanıtı:** 1) Akım ve sıcaklık sınırları için Datasheet'e bakılmalıdır. 2) İkisi bir arada olamaz; bir pin aynı anda yalnızca 1 fonksiyon olarak çalışabilir. *(Tam puan)*
4. **Soru (`ADC_CR2` Base + Offset, Reset Değeri ve Bit İşlemi):**
   - **İlk Yanıt:** 1) `0x4001 2400` (Offset eklenmedi), 2) Bilinmiyor, 3) AND operatörü.
   - **Hata Analizi ve Rehberlik:**
     - $\text{Fiziksel Adres} = \text{Base} (0x4001\,2400) + \text{Offset} (0x08)$ formülü hatırlatıldı.
     - `Reset Değeri: 0x0000 0000` olduğu için başlangıçta `ADON = 0` olduğu gösterildi.
     - Biti `1` yapmak için OR ($X \mid 1 = 1$), `0` yapmak için AND ($X \ \& \ 0 = 0$) olduğu hatırlatıldı.
   - **Öğrenci Düzeltmesi:** 1) `0x4001 2408`, 2) Kapalıdır, 3) `|=` (VEYA) operatörü kullanılmalıdır. *(Tam puan - Tam öğrenme doğrulandı)*
