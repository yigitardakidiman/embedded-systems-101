# 9. Konu: Pointer ve Struct

Bu doküman; gömülü sistemlerde donanım adreslerini ve register yapılarını doğrudan yönetmeyi sağlayan Pointer mantığını, adres (`&`) ve değer (`*`) operatörlerini, Struct yapılarını, Ok (`->`) operatörünü ve Memory-Mapped I/O yazmaç eşlemesini içerir.

---

## 9.1 Bellek Adresleri ve Pointer Mantığı (`&` ve `*` Operatörleri)

- **Pointer (İşaretçi):** RAM'deki bir verinin değerini değil, o verinin saklandığı **bellek adresini** tutan özel bir değişkendir.
- **İki Temel Operatör:**
  - **Adres Operatörü (`&`):** Bir değişkenin RAM'deki fiziksel adresini verir (`&degisken`).
  - **İçerik / Değer Operatörü (`*` - Dereference):** Bir adrese gidip o adresin içindeki hücrenin değerini okur veya yazar (`*p_degisken = 50;`).

### Soru - Cevap
> **Soru:**  
> `uint8_t voltaj = 40; uint8_t *p_voltaj = &voltaj;` için:  
> 1) `p_voltaj` kendi içinde ne tutar?  
> 2) `*p_voltaj` yazdırılırsa ne görünür?  
> 3) `*p_voltaj = 80;` satırından sonra `voltaj` değişkeninin değeri ne olur?  
> **Öğrenci Cevabı:**  
> 1) `p_voltaj` voltaj değişkeninin adresini tutar.  
> 2) O adreste 40 olduğu için 40 yazdırılır.  
> 3) Değer 80 olur çünkü `*p_voltaj` ile doğrudan o adresteki değer 80 yapılmıştır, voltaj da aynı adreste olduğu için 80 olur.  
> **Değerlendirme:** Tamamen doğru.

---

## 9.2 Fonksiyonlarda Pointer Kullanımı (Değer vs Adres ile Çağırma)

- **Değer ile Çağırma (Pass-by-Value):** Fonksiyona değişkenin kopyası gider. Fonksiyon içinde yapılan değişiklikler `main`'deki orijinal değişkeni etkilemez.
- **Adres ile Çağırma (Pass-by-Reference):** Fonksiyona değişkenin RAM adresi (`&degisken`) gönderilir. Fonksiyon `*ptr` ile doğrudan o adrese yazarak orijinal veriyi değiştirir.
- **Gömülü Sistemlerdeki Avantajları:**
  1. *Birden Fazla Değer Döndürme:* C dilinde tek bir `return` sınırı vardır. Fonksiyona adresler gönderilerek birden fazla sensör verisi (Örn: `dht11_oku(&sicaklik, &nem)`) aynı anda doldurulabilir.
  2. *Stack Tasarrufu ve Yüksek Hız:* 200 baytlık bir tamponu Stack'e kopyalamak yerine sadece 4 baytlık başlangıç adresi gönderilir.

### Soru - Cevap
> **Soru:**  
> 1) 200 baytlık veri dizisini fonksiyona kopyalamak yerine pointer ile göndermenin Stack ve hız açısından faydası nedir?  
> 2) `sicaklik` ve `nem` değişkenlerini `dht11_oku(?, ?)` fonksiyonuna gönderirken çağrı nasıl yazılır?  
> **Öğrenci Cevabı:**  
> 1) 200 baytın hepsini tek tek kopyalamak yerine başlangıç adresini göndermek hız kazandırır ve Stack belleği dolmamış olur.  
> 2) Adres (`&`) işareti kullanılır (`dht11_oku(&sicaklik, &nem);`).  
> **Değerlendirme:** Tamamen doğru.

---

## 9.3 Yapılar (Structs)

- **Dizi (Array) vs Yapı (Struct):**
  - Diziler yalnızca **aynı türden** verileri ardışık saklayabilir.
  - Struct'lar ise birbirleriyle ilişkili **farklı türdeki** verileri (örn: `uint8_t id`, `uint16_t hiz`, `char isim[10]`) tek bir çatı altında toplar.
- **Erişim:** Struct'ın doğrudan kendisine sahip olunduğunda elemanlara **Nokta (`.`)** operatörüyle erişilir.

### Soru - Cevap
> **Soru:**  
> 1) Dizi ile Struct arasındaki fark nedir?  
> 2) `struct PilDurumu ana_batarya;` içindeki `gerilim_mv` alanına 3700 değeri nasıl atanır?  
> **Öğrenci Cevabı:**  
> 1) Dizilerde sadece benzer türdeki verileri tutabiliriz, struct içerisinde farklı türlerde değişkenler tanımlayabiliriz.  
> 2) `ana_batarya.gerilim_mv = 3700;`  
> **Değerlendirme:** Tamamen doğru.

---

## 9.4 Struct Pointer'ları, Ok (`->`) Operatörü ve Donanım Yazmaçlarının Sırrı

- **Nokta (`.`) vs Ok (`->`) Operatörü:**
  - Struct'ın **kendisi** varsa: `.` kullanılır (`motor.hiz = 100;`).
  - Struct'ın **adresi (pointer'ı)** varsa: `->` kullanılır (`p_motor->hiz = 100;`).
- **Gömülü Sistemlerde Memory-Mapped I/O Yazmaç Eşlemesi:**
  - Mikrodenetleyicilerde çevre birimlerinin (GPIO, Timer, UART) yazmaçları bellekte ardışık yerleşmiştir. Üreticiler bu yazmaçları bir `struct` olarak modeller:
    ```c
    typedef struct {
        volatile uint32_t MODER;
        volatile uint32_t ODR;
    } GPIO_TypeDef;
    ```
  - `GPIOA` veya `TIMER2` ifadeleri donanımın fiziksel başlangıç adresini gösteren birer **Struct Pointer'ıdır**:
    ```c
    #define GPIOA ((GPIO_TypeDef *) 0x40020000)
    ```
  - Bu sebeple yazmaçlara `GPIOA.ODR` ile değil, pointer erişim operatörü olan **`GPIOA->ODR`** ile erişilir.

### Soru - Cevap
> **Soru 1:** Struct'ın kendisi varken ve adresi varken elemana erişim sembolleri nedir?  
> **Öğrenci Cevabı:** Struct varsa `.`, adresi varsa `->` ile erişilir. *(Doğru)*  
> 
> **Soru 2:** `TIMER2->CNT = 0;` kodunda `TIMER2` nedir ve neden `->` kullanılmıştır?  
> **Öğrenci Yanıtı:** TIMER bir adrestir (pointer) ve bu yüzden `CNT`'ye erişmek için `->` işareti kullanılmıştır. *(Doğru)*  
> **Değerlendirme:** Harika. Mikrodenetleyici kütüphanelerinin donanım erişim mantığı tam olarak kavranmıştır.
