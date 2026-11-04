# 8. Konu: Haberleşme Protokolleri (UART, I2C, SPI)

Bu doküman; gömülü sistemlerde donanımlar, çipler ve sensörler arası veri iletiminde kullanılan temel haberleşme protokollerini, elektriksel sinyalleşme kurallarını, paket yapılarını (frame anatomy) ve derste gerçekleştirilen tüm interaktif soru-cevap / hata analizi kayıtlarını sıfırdan, eksiksiz bir şekilde içerir.

---

## 📌 Alt Konu Dağılımı ve İlerleme Planı

1. **8.1 Seri Haberleşmeye Giriş:** Paralel vs Seri İletişim, Saat (Clock) İhtiyacı, Senkron vs Asenkron Kavramı.
2. **8.2 UART Protokolü:** Asenkron yapı, Baud Rate matematiği, Çerçeve (Frame) yapısı (Start, Data, Parity, Stop), Donanımsal bağlantı (TX/RX/GND), TTL vs RS-232 seviyeleri.
3. **8.3 I2C Protokolü:** 2 hatlı senkron mimari (SDA/SCL), Open-Drain ve Pull-Up dirençlerinin fiziksel zorunluluğu, Paket anatomisi (Start, 7-Bit Adres, R/W biti, 9. saat ACK/NACK, Data baytları, Stop), Multi-Master ve Tahkimat (Arbitration).
4. **8.4 SPI Protokolü:** 4 hatlı mimari (MOSI, MISO, SCK, CS), Full-Duplex shift register yapısı, Saat Modları (CPOL ve CPHA), Fiziksel çip seçimi (Active-LOW).
5. **8.5 Karşılaştırma ve Sistem Tasarımı:** Hız, pin maliyeti, mesafe ve karmaşıklık açısından protokollerin seçimi.

---
