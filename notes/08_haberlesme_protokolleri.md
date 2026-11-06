# 8. Konu: Haberleşme Protokolleri (UART, I2C, SPI)

Bu doküman; seri ve paralel haberleşme temellerini, senkron/asenkron iletişim farklarını, dupleks modlarını ve gömülü sistemlerin üç temel haberleşme protokolü olan **UART**, **I2C** ve **SPI** protokollerinin donanım mimarilerini, paket anatomilerini ve karşılaştırmalı seçim kriterlerini içerir.

---

## 8.1 Temel Haberleşme Kavramları

1. **Seri vs. Paralel Haberleşme:**
   - **Paralel:** Bir baytlık ($8\text{ bit}$) verinin $8$ ayrı hat üzerinden aynı anda gönderilmesidir. Çok fazla pin işgal eder; mesafe arttıkça hatlar arası parazit (crosstalk) ve zamanlama kayması (clock skew) nedeniyle veri bozulur.
   - **Seri:** Bitlerin tek bir hat üzerinden ardışık (arka arkaya) iletilmesidir. Az pin kullanır, uzun mesafelerde sinyal bütünlüğü korunur.

2. **Senkron vs. Asenkron Haberleşme:**
   - **Senkron (Synchronous):** Veri hattının yanında ortak bir saat sinyali hattı (**SCL / SCK**) bulunur. Alıcı ve verici bu saat darbelerine göre bitleri okur/yazar (Örn: I2C, SPI).
   - **Asenkron (Asynchronous):** Bağımsız bir saat hattı yoktur. İki taraf önceden aynı iletim hızına (**Baud Rate**) ayarlanır ve paket başı/sonu **Start/Stop** bitleriyle belirlenir (Örn: UART).

3. **Veri Akış Yönleri (Dupleks Modları):**
   - **Simplex:** Tek yönlü iletişim (Örn: Sadece konum basan GPS modülü).
   - **Half-Duplex:** Çift yönlü fakat sırayla iletişim (Telsiz mantığı; Örn: I2C SDA hattı, RS-485).
   - **Full-Duplex:** Aynı anda hem gönderme hem alma yapılabilen iletişim (Örn: UART TX/RX, SPI MOSI/MISO).

### Soru - Cevap
> **Soru:**  
> 1) $1\text{ metre}$ mesafedeki bir sensör ve pin sayısı kısıtlı bir MCU için neden paralel yerine seri haberleşme tercih edilir?  
> 2) Senkron haberleşmede zamanlama uyumu ne ile sağlanır; asenkronda bunun yerine ne kullanılır?  
> **Öğrenci Cevabı:**  
> 1) Seri haberleşmede hem daha az pin kullanılır hem de mesafe arttıkça parazitler yüzünden veri kaybı daha az olur.  
> 2) Senkronda SCL hattı ile haberleşme zamanı kontrol edilir, asenkronda verici ve alıcı arasındaki Baud Rate aynı olmalıdır.  
> **Değerlendirme:** Doğru ve eksiksiz.

---

## 8.2 UART (Universal Asynchronous Receiver-Transmitter)

- **Donanımsal Bağlantı:** Asenkron ve Full-Duplex çalışır. Verici pin (**TX - Output**) karşı tarafın alıcı pinine (**RX - Input**) çapraz bağlanır ($TX \rightarrow RX$, $RX \leftarrow TX$). Sinyallerin aynı referansa göre ölçülebilmesi için **ortak GND** bağlantısı zorunludur.
- **Paket Anatomisi (Frame):**
  1. **Boşta (Idle):** Hat sürekli **HIGH ($3.3\text{V}$)** seviyesindedir.
  2. **Start Biti:** Hattın $1\text{ bit}$ süresi boyunca **LOW ($0\text{V}$)** seviyesine çekilmesidir (Alıcının zamanlayıcısını uyandırır).
  3. **Veri Bitleri (Data):** Genellikle $8\text{ bit}$ gönderilir (**LSB First** - en anlamsız bitten başlayarak).
  4. **Parity Biti (Opsiyonel):** Basit hata kontrol biti.
  5. **Stop Biti:** Paket sonunda hattın en az $1\text{ bit}$ süresince **kesinlikle HIGH ($3.3\text{V}$)** seviyesine çekilmesidir.
- **Baud Rate ve Çerçeve Hatası (Framing Error):**
  - Örneğin $9600\text{ baud}$ hızında $1\text{ bit}$ süresi $T_{\text{bit}} = \frac{1}{9600} \approx 104.2\ \mu\text{s}$ iken, $115200\text{ baud}$ hızında $T_{\text{bit}} \approx 8.7\ \mu\text{s}$'dir.
  - Hızlar uyuşmazsa veya kablo kopup hat sürekli LOW kalırsa; alıcı, Stop bitinin olması gereken (10. adım) anda HIGH yerine LOW okur. Donanım paketin çerçevesinin bozulduğunu anlayarak **Framing Error** bayrağını kaldırır ve veriyi reddeder.

### Soru - Cevap ve Hata Analizi
> **Soru:**  
> 1) $TX \rightarrow TX$ ve $RX \rightarrow RX$ bağlanırsa neden haberleşme olmaz?  
> 2) Bir taraf $115200$, diğer taraf $9600\text{ baud}$ olursa veri neden bozulur?  
> **Öğrenci İlk Yanıtı:** 1) TX pinleri alıcıdır, iki alıcı arasında iletişim olmaz. 2) Baud rate farklı olduğunda bozuk datalar meydana gelir.  
> **Hata Analizi ve Rehberlik:**  
> - **TX (Transmitter)** alıcı değil **gönderici (çıkış)**, **RX (Receiver)** ise **alıcı (giriş)** pindir. İki çıkış ($TX-TX$) birbirine bağlanırsa ikisi de hatta voltaj basar, elektriksel çakışma olur ve $RX$ pinleri boşta kalır.  
> - Zamanlama açısından: $115200\text{ baud}$ tarafı tüm paketi $\sim 87\ \mu\text{s}$'de bitirirken, $9600\text{ baud}$ tarafı ilk biti okumak için $\sim 156\ \mu\text{s}$ bekler; yani paket çoktan bitmiş olur.  
> - **Framing Error Analojisi:** UART paketi 'A' (Start-LOW) ile başlayıp 'Z' (Stop-HIGH) ile bitmek zorunda olan 10 harflik bir cümle gibidir. Alıcı 10. adımda HIGH ('Z') görürse paketi **geçerli**, LOW görürse **hatalı (Framing Error)** kabul eder.  
> **Öğrenci Kontrol Cevabı:** Stop bitinde $3.3\text{V}$ (HIGH) görülürse mikrodenetleyici paketi **geçerli** kabul eder. *(Doğru)*

---

## 8.3 I2C (Inter-Integrated Circuit): Donanım Mimarisi

- **Yapı:** Sadece **2 tel (SDA ve SCL)** üzerinden onlarca cihazın haberleşmesini sağlayan **senkron** ve **Half-Duplex** veri yolu protokolüdür.
  - **SCL (Serial Clock):** Master tarafından üretilen saat hattı.
  - **SDA (Serial Data):** Çift yönlü (sırayla kullanılan) veri hattı.
- **Open-Drain ve Pull-Up Dirençleri:**
  - Aynı hatta bağlı birden fazla cihazın aynı anda hem HIGH hem LOW basarak kısa devre yapmasını önlemek için tüm I2C pinleri **Open-Drain** modundadır.
  - Cihazlar hatta asla aktif $3.3\text{V}$ basamaz; yalnızca hattı toprağa çekebilir (**LOW - $0\text{V}$**) veya serbest bırakır.
  - Hattın mantıksal $1$ (**HIGH - $3.3\text{V}$**) olabilmesi için SDA ve SCL hatlarına dışarıdan **Pull-Up dirençleri** (örn: $4.7\text{ k}\Omega$) bağlanması zorunludur.

### Soru - Cevap
> **Soru:** I2C devresinde harici Pull-Up dirençleri unutulursa neden haberleşme sağlanamaz?  
> **Öğrenci Cevabı:** Pull-up direnci koymadığımız için Open-Drain yapıda hat asla $3.3\text{V}$ (HIGH) olamaz, bu da verilerin aktarılmamasına yol açar.  
> **Değerlendirme:** Tamamen doğru.

---

## 8.4 I2C Paket Anatomisi: START/STOP, Adresleme ve ACK/NACK

- **START Koşulu:** SCL **HIGH** iken SDA'nın **HIGH $\rightarrow$ LOW** çekilmesi.
- **STOP Koşulu:** SCL **HIGH** iken SDA'nın **LOW $\rightarrow$ HIGH** bırakılması.
- **Adres Baytı Yapısı (`(Adres << 1) | R/W`):**
  - İlk $7\text{ bit}$ ($b_7..b_1$): Sensörün donanımsal adresi.
  - Son $1\text{ bit}$ ($b_0$ - R/W): **`0` = Yazma (Write)**, **`1` = Okuma (Read)**.
- **ACK / NACK (9. Saat Darbesi):**
  - Her $8\text{ bitlik}$ paketten sonra 9. saat darbesinde alıcı cihaz SDA'yı **LOW** yaparsa **ACK (Onay / Cihaz Hatta Mevcut)**, hat **HIGH** kalırsa **NACK (Red / Cihaz Yok veya Hata)** anlamına gelir.

### Soru - Cevap ve Hata Analizi
> **Soru:** $7\text{ bit}$ adresi `0x3C` olan ekrana Yazma (Write) yapmak için gönderilecek bayt nedir ve NACK gelirse ne anlama gelir?  
> **Öğrenci İlk Yanıtı:** 1) `0011 110`, 2) `0011 1101`, 3) Sensör veriyi okuyamadı hatasıdır.  
> **Hata Analizi ve Rehberlik:**  
> - `0x3C` $\rightarrow$ `3` (`011`) ve `C` (`1100`) $\rightarrow$ $7\text{ bit}$ olarak **`011 1100`**.  
> - Sola 1 bit kaydırılıp sonuna Write (`0`) eklendiğinde: `0111 1000` (**`0x78`**).  
> - Adres aşamasında NACK gelmesi; o adrese sahip cihazın hatta hiç cevap vermediğini (kablosunun kopuk, beslemesinin olmadığını veya adresin yanlış olduğunu) gösterir.  
> **Pekiştirme Sorusu:** Adresi `0x68` (`110 1000`) olan jiroskoptan **Okuma (Read)** yapmak için basılacak $8\text{ bitlik}$ değer nedir ve sensör sağlam bağlıysa 9. saatte ne döner?  
> **Öğrenci Cevabı:** `1101 0001` (`0xD1`). ACK döneceği için hat LOW olur, işlem başarılı olur.  
> **Değerlendirme:** Tamamen doğru.

---

## 8.5 SPI (Serial Peripheral Interface) Protokolü

- **Yapı:** Yüksek hız ($10-80\text{ MHz}$) gerektiren birimler (SD kart, renkli TFT ekran, Flash bellek) için kullanılan **senkron**, **Full-Duplex** ve **Push-Pull** çalışan protokoldür.
- **Donanım Hatları ($3 + N$ Pin Mimari):**
  1. **SCK (Serial Clock):** Master tarafından üretilen saat sinyali *(Ortak/Paralel)*.
  2. **MOSI (Master Out Slave In):** Master'dan Slave'e veri hattı *(Ortak/Paralel)*.
  3. **MISO (Master In Slave Out):** Slave'den Master'a veri hattı *(Ortak/Paralel)*.
  4. **CS / SS (Chip Select):** Her sensör için **ayrı (bağımsız)** çekilen seçim hattıdır. **Active-LOW** çalışır; Master hangi sensörle konuşacaksa yalnızca onun CS pinini **LOW ($0\text{V}$)** yapar, diğer tüm sensörlerin CS pinlerini **HIGH ($3.3\text{V}$)** tutar.
- **Shift Register Mimarisi:** Master ve Slave içindeki $8\text{ bitlik}$ kaydırmalı yazmaçlar halka şeklinde çalışır; her saat darbesinde 1 bit gönderilirken aynı anda 1 bit alınır. Adresleme baytı ve ACK/NACK beklemesi olmadığı için çok hızlıdır.

### Soru - Cevap ve Hata Analizi
> **Soru:** 5 adet SPI sensör için kaç pin gerekir ve boştayken CS pinleri hangi seviyede tutulur?  
> **Öğrenci İlk Yanıtı:** 1) Paralel bağlanacağı için 4 pin. 2) HIGH olur, çünkü haberleşmek istediği zaman CS'i LOW'a çeker.  
> **Hata Analizi ve Rehberlik:**  
> - 2. yanıt tamamen doğrudur.  
> - 1. yanıtta: Eğer 5 sensörün CS pini de tek pine paralel bağlanırsa, pin LOW yapıldığında 5 sensör aynı anda uyanıp MISO hattında çakışma (kısa devre) yaratır. Bu nedenle **SCK, MOSI, MISO ortaktır ($3\text{ pin}$)**, ancak her sensöre **bağımsız bir CS pini ($5\text{ pin}$)** çekilir: $3 + 5 = \mathbf{8\text{ pin}}$ ($3 + N$ kuralı).

---

## 8.6 Protokollerin Karşılaştırılması ve Seçim Kriterleri

| Özellik | UART | I2C | SPI |
|:---|:---:|:---:|:---:|
| **Pin Sayısı** | $2$ ($TX, RX$) | $2$ ($SDA, SCL$) | $3 + N$ ($SCK, MOSI, MISO + N \times CS$) |
| **Saat (Senkronizasyon)**| Asenkron (Baud Rate) | Senkron ($SCL$) | Senkron ($SCK$) |
| **Dupleks Modu** | Full-Duplex | Half-Duplex | Full-Duplex |
| **Hız** | Düşük / Orta | Orta ($100-400\text{ kHz}$) | Çok Yüksek ($10-80\text{ MHz}$) |
| **Çıkış Yapısı** | Push-Pull | Open-Drain (Pull-Up Şart) | Push-Pull |
| **Çoklu Cihaz Seçimi** | Yok (Birebir / 1-to-1) | Yazılımsal ($7\text{-bit}$ Adres) | Donanımsal (Her cihaza ayrı $CS$ pini) |

### Final Senaryo Sorusu ve Mühendislik Gerekçeleri
> **Senaryo Eşleştirmesi:**  
> 1) **Renkli TFT Ekran (60 FPS Grafik) $\rightarrow$ SPI:** Saniyede milyonlarca piksel verisi aktarılacağı için en yüksek bant genişliğine ve hıza sahip protokol olan SPI seçilir.  
> 2) **3 Adet Çevre Sensörü (Sıcaklık, Nem, Basınç - Pin Kısıtlı) $\rightarrow$ I2C:** Veriler yavaş değişir ve hız kritik değildir. Yazılımsal adresleme sayesinde 3 sensörün tamamı sadece **2 pin** (SDA, SCL) harcanarak bağlanır.  
> 3) **Bilgisayara Metin Loglama $\rightarrow$ UART:** İletişim iki cihaz arasında birebirdir (Point-to-Point) ve harici kabloda saat hattı (Clock) taşımadan en sade şekilde asenkron haberleşme sağlar.
