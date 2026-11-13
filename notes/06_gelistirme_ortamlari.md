# 6. Konu: Geliştirme Ortamları ve İşletim Sistemleri

Bu doküman; gömülü yazılım geliştirme sürecinde kullanılan metin editörleri, derleyiciler, IDE'ler ile Linux çekirdeği, dağıtımlar ve Gömülü Linux kullanım alanlarını içerir.

---

## 6.1 Metin Editörü, Derleyici (Compiler) ve IDE Kavramları

Mikrodenetleyiciler sadece $0$ ve $1$'lerden oluşan **Makine Kodunu (Binary)** çalıştırır. C kodunu bu seviyeye taşımak için üç temel araç kullanılır:

1. **Metin Editörü (Text Editor):** Sadece kod yazılan metin düzenleyicidir (Saf VS Code, Sublime Text). Kodu derleyemez veya donanıma yükleyemez.
2. **Derleyici (Compiler):** C kodunu işlemcinin makine diline çeviren programdır.
   - **Native Compiler:** Kendi bilgisayarında çalışıp yine kendi bilgisayarın için kod üreten derleyicidir (`gcc`).
   - **Cross-Compiler (Çapraz Derleyici):** Kendi bilgisayarında (x86) çalışıp, hedef mikrodenetleyici (ARM Cortex-M, AVR vb.) için makine kodu üreten derleyicidir (`arm-none-eabi-gcc`). Gömülü sistemlerde kural olarak çapraz derleyiciler kullanılır.
3. **IDE (Tümleşik Geliştirme Ortamı):** Editör, derleyici, hata ayıklayıcı (debugger) ve kod yükleme araçlarını tek bir arayüzde toplayan yazılımdır (STM32CubeIDE, Keil MDK, MPLAB X).

```mermaid
flowchart LR
    C["main.c / driver.h\n(Metin Editörü / IDE)"] --> CC["Cross-Compiler\n(arm-none-eabi-gcc)"]
    CC --> OBJ["Makine Kodu\n(.elf / .bin / .hex)"]
    OBJ --> PRG["Debug Probe\n(ST-Link / J-Link)"]
    PRG --> MCU["Hedef MCU\n(Flash Bellek)"]
```

### Soru - Cevap
> **Soru:**  
> 1) Bilgisayarında yazılan kodu ARM mikrodenetleyicinin makine koduna çeviren derleyici türü nedir?  
> 2) Saf VS Code kurup içine C kodu yazarsan harici bir derleyici olmadan bunu MCU'da çalıştırabilir misin?  
> **Öğrenci Cevabı:** 1) Cross compiler. 2) Çalıştıramam çünkü saf VS Code bir metin editörüdür, makine diline çevirmek için compiler gereklidir.  
> **Değerlendirme:** Tamamen doğru.

---

## 6.2 Çekirdek (Kernel), Linux ve Ubuntu Arasındaki Fark

- **Çekirdek (Kernel):** Donanım (CPU, RAM, depolama, çevre birimleri) ile üst seviye yazılımlar arasında köprü olan en alt seviye işletim sistemi bileşenidir.
- **Linux:** 1991'de Linus Torvalds tarafından yazılan bir **Çekirdektir (Kernel)**.
- **Dağıtım (Distribution):** Linux çekirdeğinin etrafına paket yöneticisi, masaüstü arayüzü ve sistem araçları eklenerek oluşturulan tam işletim sistemidir (Örn: Ubuntu, Debian, Fedora).
- **Gömülü Linux (Embedded Linux):** Yüksek işlem gücü, ağ desteği (Ethernet, WiFi), dosya sistemleri ve grafik arayüz gerektiren güçlü işlemcilerde (Raspberry Pi, akıllı TV, araç eğlence sistemleri) tercih edilir.
- **Determinizm ve Gerçek Zamanlılık (Real-Time):** Can güvenliği ve anlık tepki gerektiren sistemlerde (ABS fren, hava yastığı, motor kontrol) Linux'un zamanlama belirsizliklerinden kaçınmak için doğrudan **Bare-Metal (çıplak C)** veya **RTOS** çalıştıran mikrodenetleyiciler kullanılır.

```text
  GÖMÜLÜ LİNUX MİMARİSİ (Navigasyon, Akıllı TV)    BARE-METAL / RTOS (ABS Fren, Motor)
  +--------------------------------------------+    +----------------------------------+
  | Kullanıcı Uygulamaları (Arayüz, Ağ, Dosya) |    | Doğrudan C Uygulama Kodu         |
  +--------------------------------------------+    | (Kesin Zamanlama / Determinizm)  |
  | Dağıtım Araçları (Ubuntu / Yocto Paketler) |    +----------------------------------+
  +--------------------------------------------+    | Register / HAL Sürücü Katmanı    |
  | Linux Çekirdeği (Kernel)                   |    +----------------------------------+
  +--------------------------------------------+    | MCU Donanımı (STM32, C2000)      |
  | Güçlü İşlemci (MPU + Harici RAM / eMMC)    |    +----------------------------------+
  +--------------------------------------------+
```

### Soru - Cevap
> **Soru:**  
> 1) Linux ile Ubuntu arasındaki teknik ilişki nedir?  
> 2) Bir otomobilde ABS Fren Sistemi ile Navigasyon ekranında sırasıyla Gömülü Linux mu yoksa Bare-Metal MCU mu kullanılır? Neden?  
> **Öğrenci Cevabı:** 1) Linux bir kernel'dir (çekirdektir). Ubuntu ise Linux tabanlı bir dağıtımdır. 2) Navigasyonda Gömülü Linux, ABS'de Bare-Metal MCU kullanılır.  
> **Değerlendirme:** Doğru. ABS'de gecikmeye (jitter) tahammül yoktur; gerçek zamanlı (real-time) kesin determinizm şarttır.

---

## 6.3 Sektörde Kullanılan Popüler Gömülü IDE'ler ve Araçlar

Gömülü sistemler endüstrisinde kullanılan IDE'ler iki ana kategoriye ayrılır:

### 1. Üreticiye Özel (Vendor-Specific) ve Endüstri Standardı IDE'ler

| IDE Adı | Geliştirici | Hedef Çipler | Öne Çıkan Özellikleri |
|---|---|---|---|
| **STM32CubeIDE** | STMicroelectronics | STM32 (ARM Cortex-M) | Ücretsizdir. Dahili **STM32CubeMX** aracıyla pin ve clock ayarlarını grafiksel olarak yapıp C kodu üretir. GCC derleyicisi kullanır. |
| **Keil MDK-ARM (µVision)** | Arm / Keil | Neredeyse tüm ARM çipleri | Savunma sanayii ve otomotivde uzun yıllardır altın standarttır. Gelişmiş simülatörü ve register seviyesinde hızlı debugger'ı vardır. Lisanslıdır (32 KB kod limiti olan ücretsiz demo sürümü bulunur). |
| **IAR Embedded Workbench** | IAR Systems | ARM, MSP430, AVR, Renesas, RISC-V | Dünyanın en gelişmiş kod optimizasyonu (en küçük kod boyutu ve en yüksek hız) sunan tescilli C/C++ derleyicisine sahiptir. Medikal, havacılık ve otomotiv güvenlik standartları (MISRA-C, ISO 26262) projelerinde sıkça tercih edilir. Ücretlidir. |
| **MPLAB X IDE** | Microchip | PIC, dsPIC, AVR (ATmega, ATtiny) | NetBeans tabanlıdır. Microchip ve Atmel çiplerini kodlamak için resmi ortamdır. XC8, XC16, XC32 derleyicileri ile kullanılır. |
| **Code Composer Studio (CCS)** | Texas Instruments | MSP430, C2000, Sitara | TI'ın özellikle motor kontrolü ve dijital güç elektroniğinde lider C2000 DSP/MCU serisi için Eclipse tabanlı resmi geliştirme ortamıdır. |
| **ESP-IDF / Espressif IDE** | Espressif | ESP32, ESP8266 | Nesnelerin İnterneti (IoT), Wi-Fi ve Bluetooth tabanlı projelerde yaygın ESP32 ailesinin resmi geliştirme ortamıdır. FreeRTOS tabanlıdır. |

---

### 2. Modern ve Çoklu Platform (Cross-Platform) Ortamlar

- **VS Code + PlatformIO (Sektörün Yeni Trendi):**
  - VS Code'a yüklenen **PlatformIO** eklentisi, farklı çip üreticilerinin (STM32, ESP32, AVR, NRF52 vb.) kütüphanelerini, derleyicilerini ve yükleyicilerini tek bir çatı altında toplar.
  - Proje konfigürasyonunu `platformio.ini` dosyasında tutarak ekip çalışmasını ve Git sürüm kontrolünü inanılmaz kolaylaştırır.
- **VS Code + Cortex-Debug:**
  - `arm-none-eabi-gcc`, `CMake` ve `OpenOCD` gibi açık kaynaklı araç zincirlerini kullanarak tamamen özelleştirilmiş profesyonel bir ARM geliştirme ortamı kurmayı sağlar.
- **Arduino IDE:**
  - Öğrenmesi ve prototip geliştirmesi çok hızlıdır. Ancak arka planda donanımı aşırı soyutladığı, register kontrolünü gizlediği ve profesyonel donanımsal hata ayıklama (hardware debugging / breakpoint) yetenekleri zayıf olduğu için ticari ürün geliştirmede tercih edilmez.

---

### 3. Donanımsal Hata Ayıklayıcılar (Hardware Debuggers / Probes)

Kodun mikrodenetleyiciye yüklenmesini (Flash) ve çalışırken adım adım izlenmesini (Breakpoint, Register okuma) sağlayan fiziksel ara donanımlardır:
- **ST-Link (ST):** STM32 geliştirme kartlarının üzerinde entegre gelen resmi programlayıcı.
- **J-Link (Segger):** Dünya çapında endüstri standardı olan, son derece hızlı ve neredeyse tüm mikrodenetleyici ailelerini destekleyen profesyonel hata ayıklayıcı.
- **DAPLink / CMSIS-DAP:** ARM'ın açık kaynaklı standart debug donanımı.
- **PICkit:** Microchip PIC/AVR mikrodenetleyicileri için donanımsal programlayıcı.

```text
  +-------------------+    USB     +--------------------+  SWDIO (Veri)   +----------------+
  |  Geliştirme PC    |===========>|  ST-Link / J-Link  |---------------->|   Hedef MCU    |
  |  (IDE / Debugger) |            |  (Hardware Probe)  |---------------->|  (STM32 / ARM) |
  +-------------------+            +--------------------+  SWCLK (Saat)   |  Flash + RAM   |
                                                           GND (Ortak)    +----------------+
```

