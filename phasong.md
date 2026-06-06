# 📡 Hướng Dẫn Chế Tạo Máy Phá Sóng Đa Băng Tần (Multi-Band RF Jammer) V3.0

**Phiên bản:** 3.0  
**Phạm vi:** WiFi 2.4/5GHz, Bluetooth, Cellular 2G/3G/4G LTE, GPS, Camera không dây, Radio, LoRa, IoT  
**Bán kính:** 1m → 100m (tùy chỉnh bằng chiết áp)  
**Nguồn:** Pin Li-Po 12V 5000mAh (hoặc Adapter 12V 3A)

---

## 📋 Mục Lục

1. [Tổng Quan Hệ Thống](#1-tổng-quan-hệ-thống)
2. [Danh Sách Vật Liệu](#2-danh-sách-vật-liệu)
3. [Sơ Đồ Khối](#3-sơ-đồ-khối)
4. [Thiết Kế Mạch Chi Tiết](#4-thiết-kế-mạch-chi-tiết)
5. [Code Điều Khiển Arduino](#5-code-điều-khiển-arduino)
6. [Quy Trình Build Từng Bước](#6-quy-trình-build-từng-bước)
7. [Hiệu Chỉnh & Tối Ưu](#7-hiệu-chỉnh--tối-ưu)
8. [Bảng Tần Số Mục Tiêu](#8-bảng-tần-số-mục-tiêu)
9. [Tính Toán Phạm Vi](#9-tính-toán-phạm-vi)
10. [Sơ Đồ Mạch In (PCB)](#10-sơ-đồ-mạch-in-pcb)
11. [Debug & Khắc Phục](#11-debug--khắc-phục)

---

## 1. Tổng Quan Hệ Thống

### 1.1 Kiến Trúc

Đây là thiết kế **jammer đa băng tần dùng VCO (Voltage Controlled Oscillator) kết hợp Arduino** để quét tần số và điều chỉnh công suất linh hoạt.

**Nguyên lý:** Phát sóng nhiễu (noise) trên cùng tần số với thiết bị cần chặn. Nhiễu được tạo ra bằng cách:
- **Phương pháp chính:** Dùng VCO kết hợp với sóng mang được điều chế nhiễu trắng (AVGN - Audio Gaussian Noise)
- **Phương pháp phụ:** Quét tần số nhanh (Frequency Hopping) dùng PLL

### 1.2 Thông số kỹ thuật

| Tham số | Giá trị | Ghi chú |
|---------|---------|---------|
| Dải tần | 400 MHz → 6 GHz | 6 band riêng biệt |
| Công suất ra | 10mW → 5W mỗi band | Tùy chỉnh bằng biến trở |
| Bán kính | 1m → 100m+ | Phụ thuộc công suất + antenna |
| Nguồn | 12V DC 3A | Pin Li-Po 3S (11.1V) |
| Thời gian chạy | 2-6 giờ | Tùy mức công suất |
| Điều khiển | Arduino Nano + Rotary Encoder + OLED | Giao diện người dùng |

---

## 2. Danh Sách Vật Liệu

### 2.1 Linh kiện điện tử

| STT | Tên linh kiện | Số lượng | Giá (VNĐ) | Mục đích |
|-----|---------------|----------|-----------|----------|
| 1 | Arduino Nano (ATmega328P) | 1 | 60,000 | MCU điều khiển trung tâm |
| 2 | NE555 timer | 3 | 9,000 | Tạo noise tần số thấp |
| 3 | AD9850 DDS Module | 2 | 180,000 | Tạo sóng sin tần số chính xác |
| 4 | MAX2870 PLL Module | 1 | 350,000 | Tạo sóng RF 23.5MHz-6GHz |
| 5 | Transistor BF199 (NPN RF) | 6 | 12,000 | Tầng khuếch đại VCO |
| 6 | Transistor 2SC1971 (NPN RF Power) | 3 | 90,000 | Tầng khuếch đại công suất cuối |
| 7 | Transistor MRF454 (NPN RF Power) | 2 | 120,000 | Khuếch đại công suất lớn (>2W) |
| 8 | RD70HUF1 (MOSFET RF) | 2 | 180,000 | Khuếch đại công suất cực lớn |
| 9 | Varactor Diode BB809 (hoặc SMV1232) | 10 | 20,000 | Điều chỉnh tần số VCO |
| 10 | Pin diode BA479 (hoặc BAR64) | 4 | 16,000 | Chuyển mạch RF (bật/tắt band) |
| 11 | OP-AMP TL072 | 2 | 16,000 | Tạo nhiễu trắng âm thanh |
| 12 | Transistor 2N2222 (NPN) | 3 | 6,000 | Điều khiển relay/công suất |
| 13 | IC ổn áp 7805 (5V 1A) | 1 | 5,000 | Cấp nguồn 5V cho MCU |
| 14 | IC ổn áp 7812 (12V 1A) | 2 | 10,000 | Ổn định nguồn cho tầng RF |
| 15 | LM2596 Buck Converter (điều chỉnh được) | 2 | 45,000 | Điều chỉnh điện áp VCO |

### 2.2 Tụ điện

| STT | Loại | Giá trị | Số lượng | Ghi chú |
|-----|------|---------|----------|---------|
| 16 | Tụ gốm | 1pF, 2.2pF, 4.7pF, 10pF | 20 mỗi loại | Lọc RF |
| 17 | Tụ gốm | 47pF, 100pF, 220pF | 10 mỗi loại | Bypass RF |
| 18 | Tụ gốm | 1nF, 10nF, 100nF | 10 mỗi loại | Lọc nguồn |
| 19 | Tụ hóa | 10μF, 47μF, 100μF, 470μF | 5 mỗi loại | Lọc nguồn DC |
| 20 | Tụ trimmer | 5-50pF (loại ceramic) | 10 | Chỉnh tần số VCO |

### 2.3 Cuộn cảm & Biến áp RF

| STT | Loại | Giá trị | Số lượng | Ghi chú |
|-----|------|---------|----------|---------|
| 21 | Cuộn cảm RF (Wirewound) | 1nH, 2.2nH, 4.7nH, 10nH | 10 mỗi loại | Cộng hưởng VCO |
| 22 | Cuộn cảm RF | 22nH, 47nH, 100nH | 5 mỗi loại | Chặn RF (RFC) |
| 23 | Cuộn cảm RF | 1μH, 10μH, 100μH | 5 mỗi loại | Lọc nguồn |
| 24 | Balun RF 50:75Ω (hoặc tự quấn) | 2-3 vòng dây 0.5mm trên lõi ferrite FT37-43 | 3 | Phối hợp trở kháng |
| 25 | Dây đồng tráng men | 0.5mm, 1.0mm | 2m mỗi loại | Quấn cuộn cảm tự chế |

### 2.4 Antenna & Kết nối

| STT | Tên | Số lượng | Ghi chú |
|-----|-----|----------|---------|
| 26 | Antenna WiFi 2.4GHz 5dBi (RP-SMA) | 2 | Band WiFi/BT |
| 27 | Antenna LTE/GSM đa băng 700-2700MHz 3dBi | 2 | Band Cellular |
| 28 | Antenna GPS 1.5GHz (patch hoặc que) | 1 | Band GPS |
| 29 | Antenna 400-500MHz (que dài 15-17cm) | 1 | Band LoRa/IoT |
| 30 | SMA Female connector (gắn PCB) | 6 | Đầu nối antenna |
| 31 | Cáp RG316 SMA-SMA (20cm) | 6 | Kết nối antenna-PCB |
| 32 | Đầu nối BNC (tùy chọn cho máy đo) | 2 | Test/đo lường |

### 2.5 Hiển thị & Điều khiển

| STT | Tên | Số lượng | Ghi chú |
|-----|-----|----------|---------|
| 33 | OLED SSD1306 128x64 I2C | 1 | Màn hình hiển thị |
| 34 | Rotary Encoder (có nút nhấn) | 1 | Chọn band/chỉnh tần số |
| 35 | Biến trở 10kΩ (log) | 6 | Chỉnh công suất từng band |
| 36 | Biến trở 50kΩ (lin) | 6 | Chỉnh tần số từng VCO |
| 37 | Switch toggle 3 vị trí (ON-OFF-ON) | 6 | Bật/tắt từng band |
| 38 | Nút nhấn 12mm (momentary) | 2 | Menu / Confirm |
| 39 | LED 5mm (đỏ, xanh, vàng) | 6 mỗi màu | Trạng thái hoạt động |

### 2.6 Nguồn & Tản nhiệt

| STT | Tên | Số lượng | Ghi chú |
|-----|-----|----------|---------|
| 40 | Pin Li-Po 3S 11.1V 5000mAh (có BMS) | 1 | Nguồn chính |
| 41 | Adapter nguồn 12V 5A (AC-DC) | 1 | Nguồn dự phòng |
| 42 | Tản nhiệt nhôm (40x40x20mm) | 6 | Cho transistor công suất |
| 43 | Tản nhiệt nhôm (20x20x10mm) | 6 | Cho IC ổn áp |
| 44 | Quạt 12V 40x40x10mm | 2 | Làm mát cưỡng bức |
| 45 | Jack DC 5.5x2.1mm (gắn panel) | 1 | Đầu vào nguồn ngoài |
| 46 | Công tắc nguồn chính (20A) | 1 | Bật/tắt toàn bộ |

### 2.7 PCB & Vỏ hộp

| STT | Tên | Số lượng | Ghi chú |
|-----|-----|----------|---------|
| 47 | PCB FR4 2 lớp (10x15cm) | 3 | Mạch chính + 2 mạch RF |
| 48 | PCB FR4 1 lớp (5x7cm) | 2 | Mạch nguồn |
| 49 | Vỏ hộp nhôm (20x15x8cm) | 1 | Chống nhiễu RF |
| 50 | Giấy đồng (copper tape) | 2m | Che chắn EMI |
| 51 | Ferrite bead (hạt ferrite) | 10 | Lọc nhiễu đường nguồn |
| 52 | Dây điện mềm nhiều màu | 5m mỗi màu | Đấu nối |
| 53 | Dây RF (RG174) | 2m | Tín hiệu RF |
| 54 | Heat shrink (các size) | 1 bộ | Bọc mối hàn |
| 55 | Keo tản nhiệt (thermal paste) | 1 tuýp | Gắn transistor vào tản nhiệt |

### 2.8 Dụng cụ

| STT | Tên | Ghi chú |
|-----|-----|---------|
| 56 | Mỏ hàn xung (60W-80W) + chì hàn | Có đầu nhỏ để hàn SMT |
| 57 | Đồng hồ vạn năng (có đo tần số) | Fluke 15B+ hoặc tương đương |
| 58 | Máy hiện sóng (Oscilloscope) 100MHz | Khuyến nghị Rigol DS1102 |
| 59 | Máy phân tích phổ (Spectrum Analyzer) | TinySA Ultra (~1.5tr) |
| 60 | SWR Meter (VSWR) | NanoVNA (~800k) |
| 61 | Bàn hàn (Helping Hands) | Có kính lúp |
| 62 | Nhíp gắp (ESD-safe) | 2 cái |
| 63 | Kềm cắt dây + Kềm mỏ nhọn | 1 bộ |
| 64 | Khoan mini (Dremel) | Khoan lỗ vỏ hộp |
| 65 | Keo nến (hot glue) | Cố định linh kiện |

**Tổng chi phí ước tính:** 2,500,000 - 4,000,000 VNĐ (tùy lựa chọn linh kiện)

---

## 3. Sơ Đồ Khối

```
┌─────────────────────────────────────────────────────────────────┐
│                        NGUỒN 12V DC                              │
│  (Pin Li-Po 11.1V / Adapter 12V)                                │
└──────────┬──────────────────────────────────────────────────────┘
           │
           ▼
┌──────────────────────┐
│   BUCK CONVERTER      │
│   LM2596 (6-12V)     │◄────── Arduino điều chỉnh (CV) ────┐
│   Điều chỉnh VCC      │                                    │
│   cho tầng khuếch đại  │                                    │
└──────────┬────────────┘                                    │
           │                                                  │
           ▼                                                  │
┌─────────────────────────────────────────────────────────┐   │
│                    ARDUINO NANO                          │   │
│  ┌─────────────────────────────────────────────────┐    │   │
│  │              CHỨC NĂNG                           │    │   │
│  │  • Điều khiển PLL (MAX2870) qua SPI              │    │   │
│  │  • Điều khiển DDS (AD9850)                       │    │   │
│  │  • Đọc Rotary Encoder để chọn band/tần số        │    │   │
│  │  • Điều khiển PWM → LM2596 (chỉnh công suất)    │    │   │
│  │  • Đọc biến trở công suất (Analog)               │    │   │
│  │  • Hiển thị OLED                                 │    │   │
│  │  • Điều khiển relay/mosfet bật/tắt từng band    │    │   │
│  └─────────────────────────────────────────────────┘    │   │
└─────────────────────────────────────────────────────────┘   │
           │                                                  │
           ▼                                                  │
┌─────────────────────────────────────────────────────────┐   │
│                    RF OSCILLATOR MODULES                  │   │
│                                                          │   │
│  ┌─────────────────────┐  ┌─────────────────────┐        │   │
│  │ BAND 1: 400-500MHz  │  │ BAND 2: 700-1000MHz │        │   │
│  │ (VCO + BF199 x2)    │  │ (MAX2870 PLL)       │        │   │
│  │ Ứng: LoRa, IoT,     │  │ Ứng: GSM 900, LTE   │        │   │
│  │      Radio cảnh sát │  │     800/900, 2G      │        │   │
│  └─────────┬───────────┘  └──────────┬───────────┘        │   │
│            │                         │                    │   │
│  ┌─────────▼───────────┐  ┌──────────▼───────────┐        │   │
│  │ BAND 3: 1.5-2.0GHz  │  │ BAND 4: 2.4-2.5GHz   │        │   │
│  │ (VCO + 2SC1971)     │  │ (DDS AD9850 + 2SC1971)│        │   │
│  │ Ứng: GPS, 3G, 4G    │  │ Ứng: WiFi, BT, ZigBee │        │   │
│  └─────────┬───────────┘  └──────────┬───────────┘        │   │
│            │                         │                    │   │
│  ┌─────────▼───────────┐  ┌──────────▼───────────┐        │   │
│  │ BAND 5: 3.5-4.0GHz  │  │ BAND 6: 5.0-6.0GHz   │        │   │
│  │ (MAX2870 PLL)       │  │ (MAX2870 PLL)         │        │   │
│  │ Ứng: 5G NR n77/n78  │  │ Ứng: WiFi 5GHz,      │        │   │
│  │      C-band         │  │      5G NR n79       │        │   │
│  └─────────┬───────────┘  └──────────┬───────────┘        │   │
└─────────────┼─────────────────────────┼────────────────────┘   │
              │                         │                        │
              ▼                         ▼                        │
┌─────────────────────────────────────────────────────────┐      │
│              POWER AMPLIFIER STAGES                       │      │
│                                                          │      │
│  Mỗi band → Pre-amp (BF199) → Driver (2SC1971)          │      │
│           → Final (MRF454 / RD70HUF1)                   │      │
│           → Low-pass filter (LPF)                       │      │
│           → Directional coupler + Detector (SWR)       │      │
│           → Antenna (50Ω SMA)                          │      │
│                                                          │      │
│  Công suất điều chỉnh bằng: Điện áp VCC qua LM2596      │◄────┘
│  (PWM từ Arduino → LM2596 Feedback)                     │
└─────────────────────────────────────────────────────────┘
```

## 4. Thiết Kế Mạch Chi Tiết

### 4.1 Mạch Tạo Nhiễu Trắng (Noise Generator)

Dùng cho điều chế nhiễu (FM noise) lên sóng mang.

```
                    +5V
                     │
                     ├── R1 (10kΩ) ──┐
                     │                │
                    R2 (100kΩ)       C1 (0.1μF)
                     │                │
   Q1 (BC548) ──────┤                ├── R3 (100kΩ) ──┬─── Output
   (NPN, mắc diode)  │                │                  │  (Noise)
                     │                C2 (10μF)          │
                    GND              GND               GND

   Giải thích: Q1 mắc kiểu diode 2 cực, tạo ra nhiễu shot noise.
   Khuếch đại lên bằng OP-AMP TL072 (x100) → Lọc bandpass 100Hz-20kHz.
```

**Mạch khuếch đại và lọc noise:**

```
                    +5V                         +5V
                     │                           │
                     │                           │
   Input (noise) ───┤ R4 (1kΩ)                  │
                     │                           │
                     ├──────────┬── TL072 ──────┤────── Output
                     │          │   (Non-inv)   │      (Noise đã khuếch đại)
                    R5 (100kΩ)  │               │
                     │          │               │
                     │          ├────────────── R6 (10kΩ) ─┬── Vref = 2.5V
                    GND        │                           │
                                C3 (0.1μF)                R7 (10kΩ)
                                 │                          │
                                GND                       GND
```

### 4.2 Mạch VCO 400-500MHz (Band 1)

```
                    +12V
                     │
                    L1 (RFC 100nH)
                     │
                     ├──────────┬──────────────────────┐
                     │          │                      │
                    L2 (22nH)  C1 (10pF)             C2 (10pF)
                     │          │                      │
                     │   D1 (BB809)                   │
                     │          │                      │
   Tune Voltage ────┤ R1 (10kΩ)│                      │
   (0-12V)           │          │                      │
                     │   C3 (4.7pF)                   │
                     │          │                      │
                    Q1 (BF199) │                      │
                     │  (C)    │                      │
                     │  (B)────┘                      │
                     │  (E)                           │
                     │   │                            │
                    R2 (47Ω)                         R3 (100Ω)
                     │   │                            │
                     │  C4 (100pF)                   │
                     │   │                            │
                    GND GND                          │
                                                      │
                   ┌──────────────────────────────────┘
                   │
                  C5 (10pF) ──── To RF Pre-amp Stage

   Tần số cộng hưởng: f = 1/(2π√(L2 × C_v))
   C_v = C1 + C_D1(V)  (điện dung varactor phụ thuộc điện áp)
   
   Với L2=22nH, C1=10pF, BB809 (C=2-15pF):
   • V=12V → C_D=2pF → C_v=12pF → f=310MHz
   • V=0V → C_D=15pF → C_v=25pF → f=214MHz
   
   Chỉnh tần số bằng:
   1. Biến trở VR1 (50kΩ) chia áp từ 12V
   2. Hoặc PWM + RC filter từ Arduino (độ chính xác cao hơn)
```

### 4.3 Mạch PLL MAX2870 (Bands 2, 5, 6 — 700MHz-6GHz)

**Sơ đồ kết nối MAX2870 với Arduino (SPI):**

```
                    MAX2870                          ARDUINO NANO
               ┌──────────────┐
               │              │
   RF_Out_A ───┤ 1  RFoutA   │   D10 ──── CLK ───── D13 (SCK)
               │              │   D11 ──── DATA ──── D11 (MOSI)
               │ 2  RFoutB   │   D12 ──── LE ────── D10 (SS)
               │              │    D9 ──── CE ────── D9
   RF_Out_B ───┤ 3            │    D8 ──── SW_EN ─── D8
               │              │
               │ 4  CE       │◄──── D9 (Arduino)
               │              │
               │ 5  CLK      │◄──── D13 (Arduino)
               │              │
               │ 6  DATA     │◄──── D11 (Arduino)
               │              │
               │ 7  LE       │◄──── D10 (Arduino)
               │              │
               │ 8  VCC      │──── +5V
               │              │
              GND             GND

   Mạch lọc vòng (Loop Filter) cho MAX2870:

   CPOUT ───┬── R1 (1kΩ) ──┬── R2 (10kΩ) ──┬── VTUNE
            │               │               │
           C1 (100nF)     C2 (1nF)         C3 (100pF)
            │               │               │
           GND             GND             GND
```

### 4.4 Mạch DDS AD9850 (Band 4 — 2.4GHz WiFi/Bluetooth)

**Sơ đồ kết nối AD9850:**

```
                    AD9850                          ARDUINO NANO
               ┌──────────────┐
               │              │
               │ 1  D7       │◄──── D7
               │ 2  D6       │◄──── D6
               │ 3  D5       │◄──── D5
               │ 4  D4       │◄──── D4
               │ 5  D3       │◄──── D3
               │ 6  D2       │◄──── D2
               │ 7  D1       │◄──── D1
               │ 8  D0       │◄──── D0
               │ 9  W_CLK   │◄──── A0
               │ 10 FQ_UD   │◄──── A1
               │ 11 RESET   │◄──── A2
               │ 12 VCC     │──── +5V
               │ 13 GND     │──── GND
               │ 14 IOUT    │──── To Filter + RF Amp
               │ 15 IOUTB   │──── (100Ω → GND)
               │ 16 CLKIN   │──── 125MHz Oscillator
               └──────────────┘

   Mạch lọc đầu ra DDS (Low-pass 70MHz):

   IOUT ───┬── R1 (200Ω) ──┬──────────────┬────── Output
           │               │              │        (70MHz LPF)
          C1 (100pF)      L1 (4.7μH)    C2 (47pF)
           │               │              │
          GND             GND            GND

   LƯU Ý: AD9850 chỉ tạo được tối đa ~60MHz. 
   Để đạt 2.4GHz, dùng bộ NHÂN TẦN SỐ (Frequency Multiplier):
   DDS 60MHz → Bộ nhân x40 → 2.4GHz (dùng diode step-recovery hoặc PLL)
   
   CÁCH KHÁC: Dùng AD9850 làm reference cho PLL MAX2870 để tạo 2.4GHz
```

### 4.5 Mạch Khuếch Đại Công Suất RF

**Pre-amp Stage (dùng BF199):**

```
   Input ── C1 (10pF) ────────────┬──── C2 (10pF) ──── To Driver
   (từ VCO/DDS/PLL)               │
                                  L1 (RFC 100nH)
                                   │
                                  +12V (điều chỉnh được)
                                   │
                                  R1 (100Ω) ──── C3 (100nF)
                                   │               │
                                  Q1 (BF199)     GND
                                   │
                                  R2 (47Ω)
                                   │
                                  GND
```

**Driver Stage (dùng 2SC1971 — 1-5W):**

```
   Pre-amp Out ──┬── L1 (3 vòng dây 0.8mm, Φ5mm)
                  │
                  │
                 C1 (100pF)
                  │
                  │
                 L2 (RFC 1μH)
                  │
              +12V (điều chỉnh PWM từ LM2596)
                  │
                 C2 (470μF)
                  │
                 GND
                  │
                 Q1 (2SC1971)
                  │  (C) ──── L3 (5 vòng dây 0.8mm, Φ5mm) ────┬── C3 (100pF) ──→ To Final
                  │  (B) ──── R1 (10Ω) ──── Input (từ Pre-amp)
                  │  (E) ──── R2 (2.2Ω) ──── GND
                  │
                 GND

   Tản nhiệt: Bắt buộc! Keo tản nhiệt + tản nhôm 40x40x20mm
```

**Final Stage (dùng MRF454 / RD70HUF1 — 5-30W):**

```
   Driver Out ──┬── C1 (50pF) ──┬── L1 (3 vòng, Φ8mm)
                │               │
                │               L2 (RFC, 10 vòng ferrite bead)
                │               │
                │           +Supply (7-12V từ LM2596)
                │               │
                │              C2 (1000μF) ── GND
                │               │
                │              Q1 (MRF454 / RD70HUF1)
                │               │  (D) ──── L3 (2 vòng, Φ8mm) ──┬── C3 (100pF) ──→ LPF → Antenna
                │               │  (G) ──── R1 (100Ω) ──── Input
                │               │  (S) ──── GND (qua tản nhiệt)
                │               │
                │              GND
                │
               R2 (10kΩ) ──── GND

   LƯU Ý: Cần có mạch phối hợp trở kháng (matching network) 
   giữa các tầng để đạt hiệu suất tối đa.
```

### 4.6 Mạch Lọc Thông Thấp (LPF) Đầu Ra

Mỗi band cần LPF riêng để lọc hài (harmonic).

**LPF 2.4GHz (cho band WiFi):**

```
  Input ──┬── L1 (3.3nH) ──┬── L2 (3.3nH) ──┬── Output
          │                │                 │
         C1 (1.5pF)      C2 (2.2pF)       C3 (1.5pF)
          │                │                 │
         GND              GND               GND

  Thiết kế Chebyshev 3rd order, cutoff = 2.5GHz
```

**LPF 900MHz (cho band GSM):**

```
  Input ──┬── L1 (8.2nH) ──┬── L2 (8.2nH) ──┬── Output
          │                │                 │
         C1 (3.3pF)      C2 (5.6pF)       C3 (3.3pF)
          │                │                 │
         GND              GND               GND

  Cutoff = 900MHz
```

### 4.7 Mạch Chuyển Mạch RF (Bật/Tắt Band)

Dùng PIN Diode BA479:

```
  RF In ──┬── C1 (100pF) ──┬── RF Out
          │                 │
          │               D1 (BA479)
          │                 │
          │                L1 (RFC 1μH)
          │                 │
         SW Control ◄──────┤ R1 (1kΩ)
                           │
                          Q1 (2N2222)
                           │
                          GND

  SW Control = HIGH (5V) → Q1 dẫn → D1 phân cực thuận → RF đi qua
  SW Control = LOW (0V)  → Q1 tắt → D1 phân cực ngược → Chặn RF
```

### 4.8 Mạch Cảm Biến SWR (Bảo Vệ)

```
  RF Line ──┬─────── C1 (1pF) ──── D1 (HSMS-2850) ──┬── C2 (100nF) ──→ ADC (Arduino)
            │                                         │
           L1 (RFC 1μH)                             R1 (100kΩ)
            │                                         │
           GND                                       GND

  Đo điện áp DC tương ứng với công suất phản xạ.
  Nếu SWR > 2:1 → Arduino tự động giảm công suất hoặc tắt band.
```

### 4.9 Sơ Đồ Kết Nối Tổng Thể

```
                          ARDUINO NANO
              ┌──────────────────────────────┐
              │                              │
              │  A0 ─── Biến trở công suất 1  │
              │  A1 ─── Biến trở công suất 2  │
              │  A2 ─── Biến trở công suất 3  │
              │  A3 ─── Biến trở công suất 4  │
              │  A4 (SDA) ─── OLED SSD1306   │
              │  A5 (SCL) ─── OLED SSD1306   │
              │                              │
              │  A6 ─── SWR Detector Band 1  │
              │  A7 ─── SWR Detector Band 2  │
              │                              │
              │  D0 (RX) ── (UART Debug)     │
              │  D1 (TX) ── (UART Debug)     │
              │                              │
              │  D2 ─── Rotary Encoder SW    │
              │  D3 ─── Rotary Encoder DT    │
              │  D4 ─── Rotary Encoder CLK   │
              │                              │
              │  D5 ─── PWM → LM2596 #1      │
              │  D6 ─── PWM → LM2596 #2      │
              │  D7 ─── PWM → LM2596 #3      │
              │                              │
              │  D8 ─── SW_EN (MAX2870)      │
              │  D9 ─── CE (MAX2870)         │
              │  D10 ── LE (MAX2870)         │
              │  D11 ── MOSI (MAX2870)       │
              │  D12 ── MISO (MAX2870)       │
              │  D13 ── SCK (MAX2870)        │
              │                              │
              │  D14 (A0) ── W_CLK (AD9850)  │
              │  D15 (A1) ── FQ_UD (AD9850)  │
              │  D16 (A2) ── RESET (AD9850)  │
              │  D17 (A3) ── D0 (AD9850)     │
              │  ...                         │
              └──────────────────────────────┘
```

---

## 5. Code Điều Khiển Arduino

### 5.1 Code Chính (`jammer_v3.ino`)

```cpp
/*
 * Multi-Band RF Jammer Controller V3.0
 * Hardware: Arduino Nano + MAX2870 + AD9850 + OLED + Rotary Encoder
 * 
 * Chức năng:
 * - Điều khiển 6 băng tần riêng biệt
 * - Quét tần số (Frequency Hopping)
 * - Điều chỉnh công suất PWM
 * - Giám sát SWR tự động
 * - Hiển thị OLED
 * - Điều khiển bằng Rotary Encoder
 */

#include <SPI.h>
#include <Wire.h>
#include <Adafruit_SSD1306.h>
#include <Encoder.h>

// ==================== ĐỊNH NGHĨA CHÂN ====================

// OLED
#define OLED_SDA A4
#define OLED_SCL A5
#define OLED_ADDR 0x3C
Adafruit_SSD1306 display(128, 64, &Wire, -1);

// Rotary Encoder
#define ENC_CLK 4
#define ENC_DT  3
#define ENC_SW  2
Encoder myEnc(ENC_DT, ENC_CLK);
int lastBtnState = HIGH;
int selectedBand = 0;

// PWM (Công suất) - qua LM2596
#define PWM_BAND1 5
#define PWM_BAND2 6
#define PWM_BAND3 7
#define PWM_BAND4 8   // Dùng chân kỹ thuật số
#define PWM_BAND5 9   // Dùng chân CE (tái sử dụng)
#define PWM_BAND6 10  // Dùng chân LE (tái sử dụng)

// Analog đọc biến trở công suất
#define POT_BAND1 A0
#define POT_BAND2 A1
#define POT_BAND3 A2
#define POT_BAND4 A3

// SWR Detector
#define SWR_BAND1 A6
#define SWR_BAND2 A7

// MAX2870 PLL
#define MAX2870_LE  10
#define MAX2870_CE  9
#define MAX2870_CLK 13
#define MAX2870_DATA 11

// AD9850 DDS
#define DDS_W_CLK A0
#define DDS_FQ_UD A1
#define DDS_RESET A2
#define DDS_DATA  A3  // D0 của AD9850

// Relay bật/tắt từng band
#define RELAY_BAND1 12
#define RELAY_BAND2 11  // Chia sẻ với MAX2870 DATA
#define RELAY_BAND3 10  // Chia sẻ với MAX2870 LE
// Dùng shift register 74HC595 cho các relay còn lại

// ==================== CẤU TRÚC DỮ LIỆU ====================

typedef struct {
  const char* name;
  uint32_t freqMin;       // Hz
  uint32_t freqMax;       // Hz
  uint32_t freqCurrent;   // Hz
  uint8_t  powerPin;      // PWM pin
  uint8_t  potPin;        // Analog pot pin
  uint8_t  relayPin;      // Relay on/off
  uint8_t  swrPin;        // SWR detector
  uint8_t  powerLevel;    // 0-255
  bool     enabled;
  bool     hoppingMode;   // Frequency hopping
  uint32_t hopInterval;   // ms between hops
} BandConfig;

BandConfig bands[6] = {
  {"400-500MHz", 400000000, 500000000, 450000000, PWM_BAND1, POT_BAND1, 14, SWR_BAND1, 0, false, false, 0},
  {"700-1000MHz", 700000000, 1000000000, 850000000, PWM_BAND2, POT_BAND2, 15, SWR_BAND2, 0, false, false, 0},
  {"1.5-2.0GHz", 1500000000, 2000000000, 1575420000, PWM_BAND3, POT_BAND3, 16, A6, 0, false, false, 0},
  {"2.4-2.5GHz", 2400000000, 2500000000, 2450000000, PWM_BAND4, POT_BAND4, 17, A7, 0, false, false, 0},
  {"3.5-4.0GHz", 3500000000, 4000000000, 3700000000, PWM_BAND5, A0, 18, A6, 0, false, false, 0},
  {"5.0-6.0GHz", 5000000000, 6000000000, 5500000000, PWM_BAND6, A1, 19, A7, 0, false, false, 0}
};

// Danh sách tần số cho Frequency Hopping (WiFi channels)
const uint32_t wifiChannels[] = {
  2412000000, 2417000000, 2422000000, 2427000000,
  2432000000, 2437000000, 2442000000, 2447000000,
  2452000000, 2457000000, 2462000000, 2467000000,
  2472000000, 2482000000
};
const int wifiChannelCount = 14;

// ==================== NGUYÊN MẪU HÀM ====================

void setupDisplay();
void setupPins();
void updateDisplay();
void readEncoder();
void readPowerPots();
void setBandPower(uint8_t bandIndex, uint8_t power);
void setBandFrequency(uint8_t bandIndex, uint32_t freq);
void enableBand(uint8_t bandIndex, bool state);
void max2870WriteRegister(uint32_t reg, uint8_t addr);
void max2870SetFrequency(uint32_t freqHz);
void ddsSetFrequency(uint32_t freqHz);
void ddsWriteByte(uint8_t data);
void ddsPulse(uint8_t pin);
void checkSWR();
void frequencyHop(uint8_t bandIndex);
void emergencyShutdown();

// ==================== SETUP ====================

void setup() {
  Serial.begin(115200);
  Serial.println(F("[*] RF Jammer V3.0 khởi động..."));
  
  setupDisplay();
  setupPins();
  
  // Khởi tạo MAX2870
  pinMode(MAX2870_LE, OUTPUT);
  pinMode(MAX2870_CE, OUTPUT);
  pinMode(MAX2870_CLK, OUTPUT);
  pinMode(MAX2870_DATA, OUTPUT);
  SPI.begin();
  
  digitalWrite(MAX2870_CE, HIGH);
  delay(10);
  
  // Khởi tạo AD9850
  pinMode(DDS_W_CLK, OUTPUT);
  pinMode(DDS_FQ_UD, OUTPUT);
  pinMode(DDS_RESET, OUTPUT);
  for (int i = A3; i <= A3 + 7; i++) pinMode(i, OUTPUT);
  
  digitalWrite(DDS_RESET, LOW);
  delay(5);
  digitalWrite(DDS_RESET, HIGH);
  delay(5);
  digitalWrite(DDS_RESET, LOW);
  delay(5);
  
  // Tắt tất cả band ban đầu
  for (int i = 0; i < 6; i++) {
    enableBand(i, false);
    bands[i].powerLevel = 0;
    analogWrite(bands[i].powerPin, 0);
  }
  
  display.clearDisplay();
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 0);
  display.println(F("RF Jammer V3.0"));
  display.println(F("San sang!"));
  display.println(F(" "));
  display.println(F("Chon band: Xoay Encoder"));
  display.println(F("Bat/Tat: Nhan Encoder"));
  display.display();
  
  delay(2000);
  
  Serial.println(F("[+] Khoi dong thanh cong!"));
}

// ==================== MAIN LOOP ====================

void loop() {
  readEncoder();
  readPowerPots();
  checkSWR();
  
  // Frequency Hopping
  for (int i = 0; i < 6; i++) {
    if (bands[i].enabled && bands[i].hoppingMode) {
      frequencyHop(i);
    }
  }
  
  updateDisplay();
  delay(50);  // 20Hz update rate
}

// ==================== SETUP FUNCTIONS ====================

void setupDisplay() {
  Wire.begin();
  if (!display.begin(SSD1306_SWITCHCAPVCC, OLED_ADDR)) {
    Serial.println(F("[-] OLED khong tim thay!"));
    for (;;);  // Dừng nếu không có OLED
  }
  display.clearDisplay();
  display.setTextColor(SSD1306_WHITE);
}

void setupPins() {
  // PWM pins
  pinMode(PWM_BAND1, OUTPUT);
  pinMode(PWM_BAND2, OUTPUT);
  pinMode(PWM_BAND3, OUTPUT);
  pinMode(PWM_BAND4, OUTPUT);
  pinMode(PWM_BAND5, OUTPUT);
  pinMode(PWM_BAND6, OUTPUT);
  
  // Analog pot pins
  pinMode(POT_BAND1, INPUT);
  pinMode(POT_BAND2, INPUT);
  pinMode(POT_BAND3, INPUT);
  pinMode(POT_BAND4, INPUT);
  
  // Relay pins
  pinMode(RELAY_BAND1, OUTPUT);
  pinMode(RELAY_BAND2, OUTPUT);
  pinMode(RELAY_BAND3, OUTPUT);
  
  // Encoder
  pinMode(ENC_CLK, INPUT_PULLUP);
  pinMode(ENC_DT, INPUT_PULLUP);
  pinMode(ENC_SW, INPUT_PULLUP);
  lastBtnState = digitalRead(ENC_SW);
}

// ==================== ENCODER & MENU ====================

void readEncoder() {
  static int lastPos = 0;
  static unsigned long lastDebounce = 0;
  
  long newPos = myEnc.read() / 4;  // Chia 4 vì encoder có 4 xung/vòng
  
  if (newPos != lastPos) {
    selectedBand += (newPos - lastPos);
    if (selectedBand < 0) selectedBand = 0;
    if (selectedBand > 5) selectedBand = 5;
    lastPos = newPos;
  }
  
  // Xử lý nút nhấn encoder
  int btnState = digitalRead(ENC_SW);
  if (btnState == LOW && lastBtnState == HIGH && millis() - lastDebounce > 300) {
    // Nhấn: bật/tắt band hiện tại
    bands[selectedBand].enabled = !bands[selectedBand].enabled;
    enableBand(selectedBand, bands[selectedBand].enabled);
    
    if (bands[selectedBand].enabled) {
      // Bắt đầu với 50% công suất
      bands[selectedBand].powerLevel = 128;
      setBandPower(selectedBand, 128);
      Serial.print(F("[+] Band "));
      Serial.print(bands[selectedBand].name);
      Serial.println(F(": ON"));
    } else {
      setBandPower(selectedBand, 0);
      Serial.print(F("[-] Band "));
      Serial.print(bands[selectedBand].name);
      Serial.println(F(": OFF"));
    }
    
    lastDebounce = millis();
  }
  lastBtnState = btnState;
  
  // Giữ nút 2 giây: chuyển chế độ hopping
  static unsigned long pressStart = 0;
  static bool hoppingToggle = false;
  
  if (btnState == LOW) {
    if (pressStart == 0) pressStart = millis();
    if (millis() - pressStart > 2000 && !hoppingToggle) {
      bands[selectedBand].hoppingMode = !bands[selectedBand].hoppingMode;
      hoppingToggle = true;
      Serial.print(F("[*] Hopping mode: "));
      Serial.println(bands[selectedBand].hoppingMode ? F("ON") : F("OFF"));
    }
  } else {
    pressStart = 0;
    hoppingToggle = false;
  }
}

// ==================== ĐIỀU KHIỂN CÔNG SUẤT ====================

void readPowerPots() {
  for (int i = 0; i < 4; i++) {
    int potVal = analogRead(bands[i].potPin);
    // Map 0-1023 → 0-255 (power level)
    uint8_t newPower = map(potVal, 0, 1023, 0, 255);
    
    if (abs(newPower - bands[i].powerLevel) > 5) {  // Deadband 5 để tránh nhấp nháy
      bands[i].powerLevel = newPower;
      if (bands[i].enabled) {
        setBandPower(i, newPower);
      }
    }
  }
}

void setBandPower(uint8_t bandIndex, uint8_t power) {
  if (bandIndex < 6) {
    // PWM control cho LM2596
    analogWrite(bands[bandIndex].powerPin, power);
    
    // Log công suất
    float powerPercent = (power / 255.0) * 100.0;
    Serial.print(F("[P] Band "));
    Serial.print(bands[bandIndex].name);
    Serial.print(F(": "));
    Serial.print(powerPercent);
    Serial.println(F("%"));
  }
}

// ==================== ĐIỀU KHIỂN TẦN SỐ ====================

void setBandFrequency(uint8_t bandIndex, uint32_t freq) {
  bands[bandIndex].freqCurrent = freq;
  
  switch (bandIndex) {
    case 0: // 400-500MHz: VCO + varactor
      // Điều chỉnh điện áp tune qua PWM + RC filter
      // V = (freq - 400MHz) / (500MHz - 400MHz) * 12V
      {
        float ratio = (float)(freq - bands[bandIndex].freqMin) / 
                      (float)(bands[bandIndex].freqMax - bands[bandIndex].freqMin);
        uint8_t tuneVal = (uint8_t)(ratio * 255.0);
        analogWrite(A4, tuneVal);  // Dùng A4 làm PWM tune
      }
      break;
      
    case 1: // 700-1000MHz: MAX2870
    case 4: // 3.5-4.0GHz: MAX2870
    case 5: // 5.0-6.0GHz: MAX2870
      max2870SetFrequency(freq);
      break;
      
    case 2: // 1.5-2.0GHz: MAX2870
      max2870SetFrequency(freq);
      break;
      
    case 3: // 2.4-2.5GHz: DDS + multiplier
      // DDS tạo ~60MHz, nhân x40 = 2.4GHz
      ddsSetFrequency(freq / 40);
      break;
  }
}

// ==================== MAX2870 PLL CONTROL ====================

void max2870WriteRegister(uint32_t reg, uint8_t addr) {
  digitalWrite(MAX2870_LE, LOW);
  delayMicroseconds(1);
  
  // Gửi 32 bit dữ liệu
  for (int i = 3; i >= 0; i--) {
    uint8_t byteVal = (reg >> (i * 8)) & 0xFF;
    shiftOut(MAX2870_DATA, MAX2870_CLK, MSBFIRST, byteVal);
  }
  
  // Gửi 3 bit địa chỉ
  for (int i = 2; i >= 0; i--) {
    digitalWrite(MAX2870_CLK, LOW);
    digitalWrite(MAX2870_DATA, (addr >> i) & 0x01);
    digitalWrite(MAX2870_CLK, HIGH);
  }
  
  digitalWrite(MAX2870_LE, HIGH);
  delayMicroseconds(1);
}

void max2870SetFrequency(uint32_t freqHz) {
  // Tính toán tham số PLL
  // f_VCO = f_PFD * (N + Frac / Mod)
  // f_PFD = f_REF / R_Divider
  
  uint32_t f_PFD = 10000000;  // 10 MHz reference ÷ 1
  uint32_t N = freqHz / f_PFD;
  uint32_t Frac = (freqHz % f_PFD) * 4095 / f_PFD;  // 12-bit frac
  
  // Register 0: N-divider, frac
  uint32_t reg0 = (N & 0xFFFF) << 15 | (Frac & 0xFFF) << 3;
  max2870WriteRegister(reg0, 0);
  
  // Register 1: Modulus
  uint32_t reg1 = 4095 << 15;  // Mod = 4095
  max2870WriteRegister(reg1, 1);
  
  // Register 2: R-divider, etc
  uint32_t reg2 = 1 << 18;  // R=1
  max2870WriteRegister(reg2, 2);
  
  // Register 3: 
  uint32_t reg3 = 0;
  max2870WriteRegister(reg3, 3);
  
  Serial.print(F("[PLL] Tan so: "));
  Serial.print(freqHz / 1000000);
  Serial.println(F(" MHz"));
}

// ==================== AD9850 DDS CONTROL ====================

void ddsSetFrequency(uint32_t freqHz) {
  // AD9850: 125MHz clock, 32-bit tuning word
  // f_out = (tuning_word * 125MHz) / 2^32
  uint32_t tuningWord = (uint32_t)((double)freqHz * 4294967296.0 / 125000000.0);
  
  digitalWrite(DDS_FQ_UD, LOW);
  
  // Gửi tuning word (32 bit)
  for (int i = 0; i < 4; i++) {
    ddsWriteByte((tuningWord >> (i * 8)) & 0xFF);
  }
  
  // Gửi control byte (0x00: phase = 0, no power down)
  ddsWriteByte(0x00);
  
  // FQ_UD pulse để cập nhật tần số
  digitalWrite(DDS_FQ_UD, HIGH);
  delayMicroseconds(1);
  digitalWrite(DDS_FQ_UD, LOW);
  
  Serial.print(F("[DDS] Tan so: "));
  Serial.print(freqHz / 1000000);
  Serial.println(F(" MHz (base)"));
}

void ddsWriteByte(uint8_t data) {
  for (int i = 0; i < 8; i++) {
    digitalWrite(DDS_DATA, (data >> i) & 0x01);
    ddsPulse(DDS_W_CLK);
  }
}

void ddsPulse(uint8_t pin) {
  digitalWrite(pin, HIGH);
  delayMicroseconds(1);
  digitalWrite(pin, LOW);
}

// ==================== BẬT/TẮT BAND ====================

void enableBand(uint8_t bandIndex, bool state) {
  if (state) {
    // Bật: set tần số, bật relay, tăng dần công suất
    setBandFrequency(bandIndex, bands[bandIndex].freqCurrent);
    
    switch(bandIndex) {
      case 0: digitalWrite(RELAY_BAND1, HIGH); break;
      case 1: digitalWrite(RELAY_BAND2, HIGH); break;
      case 2: digitalWrite(RELAY_BAND3, HIGH); break;
      case 3: 
      case 4:
      case 5:
        // Dùng shift register mở rộng
        break;
    }
    
    // Tăng dần công suất để tránh sốc
    for (int p = 0; p <= bands[bandIndex].powerLevel; p += 5) {
      analogWrite(bands[bandIndex].powerPin, p);
      delay(10);
    }
  } else {
    // Tắt ngay lập tức
    analogWrite(bands[bandIndex].powerPin, 0);
    
    switch(bandIndex) {
      case 0: digitalWrite(RELAY_BAND1, LOW); break;
      case 1: digitalWrite(RELAY_BAND2, LOW); break;
      case 2: digitalWrite(RELAY_BAND3, LOW); break;
      case 3:
      case 4:
      case 5:
        break;
    }
  }
}

// ==================== SWR MONITOR ====================

void checkSWR() {
  for (int i = 0; i < 2; i++) {
    if (bands[i].enabled) {
      int swrVal = analogRead(bands[i].swrPin);
      float voltage = swrVal * (5.0 / 1023.0);
      
      // SWR = (Vforward + Vreflected) / (Vforward - Vreflected)
      // Đơn giản hóa: nếu voltage > 2.5V → SWR cao
      if (voltage > 2.5) {
        Serial.print(F("[!] SWR CAO - Band "));
        Serial.print(bands[i].name);
        Serial.print(F(": "));
        Serial.println(voltage);
        
        // Giảm công suất
        bands[i].powerLevel = max(0, bands[i].powerLevel - 20);
        setBandPower(i, bands[i].powerLevel);
        
        if (bands[i].powerLevel == 0) {
          enableBand(i, false);
          bands[i].enabled = false;
          Serial.println(F("[!] TAT band de bao ve!"));
        }
      }
    }
  }
}

// ==================== FREQUENCY HOPPING ====================

void frequencyHop(uint8_t bandIndex) {
  static unsigned long lastHop[6] = {0};
  
  if (millis() - lastHop[bandIndex] > bands[bandIndex].hopInterval) {
    uint32_t newFreq;
    
    switch (bandIndex) {
      case 3: // WiFi band: quét qua các kênh
        {
          static int chanIdx = 0;
          newFreq = wifiChannels[chanIdx];
          chanIdx = (chanIdx + 1) % wifiChannelCount;
        }
        break;
        
      case 1: // GSM: quét 890-960MHz
        newFreq = 890000000 + random(0, 70) * 100000;
        break;
        
      default:
        // Random trong dải tần
        newFreq = bands[bandIndex].freqMin + 
                  random(0, (bands[bandIndex].freqMax - bands[bandIndex].freqMin) / 1000) * 1000;
        break;
    }
    
    setBandFrequency(bandIndex, newFreq);
    lastHop[bandIndex] = millis();
  }
}

// ==================== EMERGENCY SHUTDOWN ====================

void emergencyShutdown() {
  Serial.println(F("[!!!] EMERGENCY SHUTDOWN!"));
  
  for (int i = 0; i < 6; i++) {
    analogWrite(bands[i].powerPin, 0);
    enableBand(i, false);
    bands[i].enabled = false;
    bands[i].powerLevel = 0;
  }
  
  // Hiển thị cảnh báo
  display.clearDisplay();
  display.setTextSize(2);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 20);
  display.println(F("EMERGENCY"));
  display.println(F("SHUTDOWN!"));
  display.display();
}

// ==================== DISPLAY ====================

void updateDisplay() {
  display.clearDisplay();
  
  // Header
  display.setTextSize(1);
  display.setTextColor(SSD1306_WHITE);
  display.setCursor(0, 0);
  display.print(F("RF JAMMER V3.0"));
  display.setCursor(100, 0);
  
  // Nguồn (ước lượng)
  static unsigned long lastBatCheck = 0;
  if (millis() - lastBatCheck > 10000) {
    int batRaw = analogRead(A7);  // Pin voltage divider qua A7 (tạm thời)
    lastBatCheck = millis();
  }
  display.print(F("12V"));
  
  // Danh sách band
  for (int i = 0; i < 6; i++) {
    int y = 10 + i * 9;
    
    // Con trỏ band được chọn
    if (i == selectedBand) {
      display.drawRect(0, y - 1, 127, 9, SSD1306_WHITE);
    }
    
    // Tên band
    display.setCursor(2, y);
    display.print(bands[i].name);
    
    // Trạng thái
    display.setCursor(72, y);
    if (bands[i].enabled) {
      display.print(F("ON "));
      // Hiển thị % công suất
      int pct = map(bands[i].powerLevel, 0, 255, 0, 100);
      if (pct < 10) display.print(F(" "));
      display.print(pct);
      display.print(F("%"));
      
      // Hopping indicator
      if (bands[i].hoppingMode) {
        display.print(F(" H"));
      }
    } else {
      display.print(F("OFF"));
    }
  }
  
  display.display();
}
```

### 5.2 Code Điều Khiển Từ Xa (USB-Serial)

```python
#!/usr/bin/env python3
"""
Remote Controller cho RF Jammer V3.0
Giao tiếp qua USB-Serial với Arduino
"""

import serial
import serial.tools.list_ports
import time
import sys
import os
from typing import Optional

class JammerController:
    """Remote controller class"""
    
    def __init__(self, port: str = None, baud: int = 115200):
        self.ser = None
        self.port = port
        self.baud = baud
        
    def connect(self) -> bool:
        """Kết nối tới Jammer"""
        if self.port:
            ports = [self.port]
        else:
            ports = [p.device for p in serial.tools.list_ports.comports()]
        
        for port in ports:
            try:
                self.ser = serial.Serial(port, self.baud, timeout=2)
                time.sleep(2)
                self.ser.write(b"\r\n")
                response = self.ser.read(100)
                if b"Jammer" in response:
                    print(f"[+] Kết nối thành công: {port}")
                    return True
            except:
                continue
        
        print("[-] Không tìm thấy Jammer!")
        return False
    
    def send_command(self, cmd: str) -> str:
        """Gửi lệnh và nhận phản hồi"""
        if not self.ser:
            return "Not connected"
        
        self.ser.write((cmd + "\n").encode())
        time.sleep(0.5)
        return self.ser.read(200).decode('utf-8', errors='ignore')
    
    def band_on(self, band: int, power: int = 128) -> str:
        """Bật 1 band (0-5) với công suất (0-255)"""
        return self.send_command(f"BAND{band} ON {power}")
    
    def band_off(self, band: int) -> str:
        """Tắt 1 band"""
        return self.send_command(f"BAND{band} OFF")
    
    def set_frequency(self, band: int, freq_mhz: int) -> str:
        """Chỉnh tần số (MHz)"""
        return self.send_command(f"FREQ {band} {freq_mhz}")
    
    def set_power(self, band: int, power: int) -> str:
        """Chỉnh công suất (0-255)"""
        return self.send_command(f"POWER {band} {power}")
    
    def hopping_on(self, band: int) -> str:
        """Bật chế độ quét tần số"""
        return self.send_command(f"HOP {band} ON")
    
    def hopping_off(self, band: int) -> str:
        """Tắt chế độ quét tần số"""
        return self.send_command(f"HOP {band} OFF")
    
    def emergency_off(self) -> str:
        """Tắt khẩn cấp"""
        return self.send_command("EMERGENCY_OFF")
    
    def status(self) -> str:
        """Xem trạng thái"""
        return self.send_command("STATUS")
    
    def close(self):
        if self.ser:
            self.ser.close()

def print_help():
    print(""" 
RF Jammer Remote Controller
============================
Lệnh:
  connect [port]   - Kết nối tới Jammer
  on BAND [POWER]  - Bật band (0-5), power (0-255)
  off BAND         - Tắt band
  freq BAND MHZ    - Đặt tần số
  power BAND LEVEL - Đặt công suất
  hop BAND ON/OFF  - Bật/tắt frequency hopping
  status           - Xem trạng thái
  emergency        - Tắt khẩn cấp
  help             - Trợ giúp
  exit             - Thoát
""")

def main():
    controller = JammerController()
    
    if not controller.connect():
        print("Thử kết nối thủ công với: python jammer_remote.py COM3")
        return
    
    print_help()
    
    while True:
        try:
            cmd = input("jammer> ").strip()
            if not cmd:
                continue
            
            parts = cmd.split()
            action = parts[0].lower()
            
            if action == "exit":
                break
            elif action == "help":
                print_help()
            elif action == "on":
                band = int(parts[1])
                power = int(parts[2]) if len(parts) > 2 else 128
                print(controller.band_on(band, power))
            elif action == "off":
                band = int(parts[1])
                print(controller.band_off(band))
            elif action == "freq":
                band = int(parts[1])
                freq = int(parts[2])
                print(controller.set_frequency(band, freq))
            elif action == "power":
                band = int(parts[1])
                level = int(parts[2])
                print(controller.set_power(band, level))
            elif action == "hop":
                band = int(parts[1])
                if parts[2].lower() == "on":
                    print(controller.hopping_on(band))
                else:
                    print(controller.hopping_off(band))
            elif action == "status":
                print(controller.status())
            elif action == "emergency":
                print(controller.emergency_off())
            else:
                print("Lệnh không hợp lệ. Gõ 'help' để xem hướng dẫn.")
                
        except KeyboardInterrupt:
            break
        except Exception as e:
            print(f"Lỗi: {e}")
    
    controller.close()

if __name__ == "__main__":
    main()
```

### 5.3 Code Calibration Tự Động

```cpp
/*
 * Calibration Routine - gọi từ Arduino Serial Monitor
 * Gửi 'CAL' để bắt đầu
 */

void calibrationRoutine() {
  Serial.println(F("[CAL] Bat dau hieu chinh..."));
  
  for (int band = 0; band < 6; band++) {
    Serial.print(F("[CAL] Dang hieu chinh band "));
    Serial.println(bands[band].name);
    
    // Quét tần số
    for (uint32_t freq = bands[band].freqMin; freq <= bands[band].freqMax; freq += 1000000) {
      setBandFrequency(band, freq);
      delay(100);
      
      // Đọc công suất phản hồi từ SWR detector
      int swrRaw = analogRead(bands[band].swrPin);
      
      // Log dữ liệu
      Serial.print(F("[CAL] "));
      Serial.print(freq / 1000000);
      Serial.print(F(" MHz, SWR: "));
      Serial.println(swrRaw);
    }
  }
  
  Serial.println(F("[CAL] Hoan thanh!"));
}
```

---

## 6. Quy Trình Build Từng Bước

### Giai Đoạn 1: Chuẩn Bị (Ngày 1)

**Bước 1: Kiểm tra linh kiện**
- [ ] Đo thử từng transistor bằng đồng hồ vạn năng (thang diode)
- [ ] Kiểm tra varactor diode: đo điện dung thay đổi theo điện áp
- [ ] Nạp thử code Blink vào Arduino để kiểm tra
- [ ] Kiểm tra OLED: chạy scan I2C để tìm địa chỉ

**Bước 2: Làm sạch PCB**
- [ ] Dùng cồn 90° lau sạch PCB
- [ ] Đánh nhẹ bề mặt bằng giấy nhám mịn (nếu cần)
- [ ] Tráng 1 lớp thiếc mỏng lên đường mạch

### Giai Đoạn 2: Hàn Mạch Nguồn (Ngày 1-2)

**Bước 3: Hàn mạch ổn áp**
```
Sơ đồ:
    12V In ──┬── C1 (470μF) ── GND
             │
            LM2596 (IN)
             │
            LM2596 (OUT) ──┬── C2 (470μF) ── GND
             │              │
            R1 (10kΩ)      R2 (PWM từ Arduino)
             │              │
            FB             GND
```

- [ ] Hàn LM2596 module lên PCB
- [ ] Cắm jumper cho chân Feedback (bỏ R1, dùng PWM)
- [ ] Hàn tụ lọc đầu vào/vả
- [ ] Kiểm tra: cấp 12V, đo đầu ra: điều chỉnh PWM 0-255 → 1.25V - 12V

**Bước 4: Hàn mạch 5V (cho Arduino)**
- [ ] Hàn 7805 + tụ lọc
- [ ] Kiểm tra: đầu ra 5.00V ±0.1V

### Giai Đoạn 3: Hàn VCO + Pre-amp (Ngày 2-3)

**Bước 5: Hàn VCO 400-500MHz (Band 1)**
- [ ] Hàn cuộn cảm L2 (22nH) lên PCB
- [ ] Hàn tụ C1 (10pF), C2, C3
- [ ] Hàn varactor BB809 (chú ý chiều! dấu gạch là cathode)
- [ ] Hàn transistor BF199 (chú ý chân E-B-C)
- [ ] Hàn điện trở phân cực

**Bước 6: Kiểm tra VCO**
1. Cấp nguồn 12V (qua biến trở để an toàn)
2. Đo điện áp tune: 0-12V
3. Dùng oscilloscope đo tại collector Q1
4. Chỉnh tụ trimmer cho đến khi thấy dao động
5. Xoay biến trở tune → tần số thay đổi

**Lưu ý:** Nếu không có oscilloscope, có thể dùng đài FM + diode detector:
- Quấn 1 vòng dây gần VCO
- Chỉnh đài FM đến khi nghe tiếng rít (nhiễu)
- Khi xoay tune, tần số rít thay đổi → VCO hoạt động

### Giai Đoạn 4: Hàn PLL MAX2870 (Ngày 3-4)

**Bước 7: Hàn MAX2870**
```
CẢNH BÁO: MAX2870 là IC QFN-20 (0.5mm pitch).
Cần có kinh nghiệm hàn SMT hoặc dùng module có sẵn.
```

- [ ] Dùng kẹp IC gắp MAX2870 lên PCB
- [ ] Hàn từng chân (dùng flux, mỏ hàn 300°C, đầu nhọn)
- [ ] Kiểm tra chân hàn bằng kính lúp
- [ ] Hàn loop filter (R1, R2, C1, C2, C3)
- [ ] Hàn tụ bypass 100nF gần chân VCC

**Bước 8: Nạp code test PLL**
```cpp
// Test PLL - gửi tần số 900MHz
void testPLL() {
  max2870SetFrequency(900000000);  // 900 MHz
  delay(100);
  Serial.println(F("PLL test: 900 MHz"));
}
```

- [ ] Nạp code test lên Arduino
- [ ] Mở Serial Monitor → xem output
- [ ] Dùng máy phân tích phổ (TinySA) hoặc đài dò tín hiệu để kiểm tra

**Không có máy phân tích phổ?** Dùng phương pháp "tạm":
- Cắm antenna vào SMA
- Đưa điện thoại gần antenna (cách 5cm)
- Gọi từ điện thoại khác: nếu cuộc gọi bị nhiễu/ngắt → PLL hoạt động

### Giai Đoạn 5: Hàn DDS AD9850 (Ngày 4)

**Bước 9: Hàn AD9850**
- [ ] Hàn AD9850 (DIP16 hoặc module có sẵn)
- [ ] Kết nối 125MHz oscillator (hoặc dùng module có sẵn)
- [ ] Hàn mạch lọc LPF đầu ra (70MHz)

**Bước 10: Kiểm tra DDS**
```cpp
void testDDS() {
  ddsSetFrequency(60000000);  // 60 MHz
  Serial.println(F("DDS test: 60 MHz"));
}
```

- [ ] Đo đầu ra DDS (IOUT) bằng oscilloscope
- [ ] Sóng sin ~1V p-p tại 60MHz
- [ ] Đưa vào bộ nhân tần (hoặc trộn với PLL)

### Giai Đoạn 6: Hàn Tầng Khuếch Đại Công Suất (Ngày 5-6)

**Bước 11: Hàn tầng Driver (2SC1971)**
- [ ] Bôi kem tản nhiệt lên mặt sau transistor
- [ ] Gắn transistor vào tản nhiệt nhôm (vít M3)
- [ ] Hàn chân B-C-E
- [ ] Hàn cuộn cảm RFC và tụ bypass
- [ ] Hàn R1 (10Ω) từ chân B đến Input

**Bước 12: Hàn tầng Final (MRF454 / RD70HUF1)**
- [ ] Tản nhiệt bắt buộc: 2 tản nhôm kẹp 2 mặt
- [ ] Gắn quạt 12V thổi trực tiếp
- [ ] Hàn dây nguồn 1.5mm² (dòng lớn!)
- [ ] Hàn mạch matching network

**Bước 13: Build mạch LPF**
- [ ] Dùng máy tính (sóng RF) hoặc tính tay | Mỗi band 1 LPF riêng
- [ ] Hàn LPF theo thiết kế ở phần 4.6
- [ ] Đo SWR để kiểm tra

### Giai Đoạn 7: Hàn Mạch Điều Khiển & Hiển Thị (Ngày 6-7)

**Bước 14: Hàn Arduino và OLED**
- [ ] Cắm Arduino Nano lên PCB (qua socket)
- [ ] Kết nối I2C: A4(SDA) ↔ SDA, A5(SCL) ↔ SCL
- [ ] Hàn Rotary Encoder (kéo lên 10kΩ)
- [ ] Hàn LED báo + trở hạn dòng 220Ω

**Bước 15: Hàn biến trở + Switch**
- [ ] Gắn 6 biến trở công suất lên mặt trước hộp
- [ ] Hàn dây từ biến trở → A0-A5 (qua chống nhiễu)
- [ ] Hàn 6 switch toggle → D2-D7 (qua pull-down 10kΩ)

### Giai Đoạn 8: Lắp Ráp Vỏ Hộp (Ngày 7-8)

**Bước 16: Chuẩn bị vỏ hộp**
1. Khoan lỗ:
   - Lỗ SMA x6 (Φ8mm)
   - Lỗ biến trở x6 (Φ10mm)
   - Lỗ switch x6 (Φ12mm)
   - Lỗ OLED (Φ25mm hoặc hình chữ nhật)
   - Lỗ LED x6 (Φ5mm)
   - Lỗ quạt (Φ40mm)
   - Lỗ jack DC (Φ8mm)
   - Lỗ công tắc nguồn (Φ12mm)

2. Dán giấy đồng (copper tape) phía trong vỏ
3. Hàn nối các mảnh giấy đồng lại với nhau
4. Nối giấy đồng → GND

**Bước 17: Lắp linh kiện lên vỏ**
1. Bắt vít PCB xuống đáy hộp (dùng standoff)
2. Gắn quạt → hàn vào nguồn 12V (qua relay nhiệt 45°C)
3. Gắn antenna → SMA
4. Đi dây nguồn chính: dây 1.5mm² cho dòng > 2A

**Bước 18: Đi dây kết nối**
Sử dụng sơ đồ dưới đây:

```
┌────────────────────────────────────────────────────────────┐
│                     WIRING DIAGRAM                          │
│                                                             │
│  NGUỒN:                                                     │
│  Pin 12V (+) ──┬── Công tắc chính ──┬───┐                   │
│                │                     │   │                   │
│               Cầu chì 5A            │   LM2596 (VCO Power) │
│                │                     │   │                   │
│                └──┬── 7812 ── 12V RF ──┘ │                   │
│                   │                       │                  │
│                   └── 7805 ── 5V Arduino ─┘                  │
│                                                             │
│  RF (dây RG174, ngắn < 15cm):                               │
│  VCO1 → Pre-amp → Driver → LPF → SWR Det → Antenna 1       │
│  PLL1 → Pre-amp → Driver → LPF → SWR Det → Antenna 2       │
│  VCO2 → Pre-amp → Driver → LPF → SWR Det → Antenna 3       │
│  DDS → Pre-amp → Driver → LPF → SWR Det → Antenna 4        │
│  PLL2 → Pre-amp → Driver → LPF → SWR Det → Antenna 5       │
│  PLL3 → Pre-amp → Driver → LPF → SWR Det → Antenna 6       │
│                                                             │
│  ĐIỀU KHIỂN (dây 0.5mm, dây xoắn):                         │
│  [Encoder] ←20cm dây xoắn→ [Arduino D2/D3/D4]              │
│  [OLED]    ←I2C (SDA/SCL)→  [Arduino A4/A5]                │
│  [Switch]  ←dây→ [Arduino Dx] (mỗi switch 1 chân)         │
│  [Pot]     ←dây có chắn→ [Arduino Ax] (mỗi pot 1 chân)    │
│  [LED]     ←220Ω→ [Arduino Dx] (qua transistor ULN2003)    │
└────────────────────────────────────────────────────────────┘
```

### Giai Đoạn 9: Nạp Code & Chạy Thử (Ngày 8-9)

**Bước 19: Nạp code**
1. Cài đặt Arduino IDE
2. Thêm thư viện: Adafruit SSD1306, Encoder
3. Mở `jammer_v3.ino`
4. Board: Arduino Nano, Processor: ATmega328P (Old Bootloader)
5. Nạp code

**Bước 20: Kiểm tra từng bước**
1. **Không gắn antenna** → Bật từng band 1 → Đo SWR
2. **SWR < 2:1** → Tốt. SWR > 2:1 → Kiểm tra matching
3. **Gắn antenna** → Tăng dần công suất (20% → 50% → 100%)
4. **Kiểm tra từ xa**: 
   - WiFi: Điện thoại cách 5m → kiểm tra mạng WiFi biến mất
   - GSM: Cách 5m → gọi thử, cuộc gọi không kết nối được
   - GPS: Cách 10m → GPS không định vị được

### Giai Đoạn 10: Hiệu Chỉnh & Hoàn Thiện (Ngày 9-10)

**Bước 21: Calibration**
1. Bật chế độ CAL từ Serial Monitor
2. Dùng máy phân tích phổ: đo công suất tại từng tần số
3. Ghi lại tần số cho công suất cao nhất
4. Chỉnh LPF nếu cần

**Bước 22: Kiểm tra tản nhiệt**
1. Chạy full power trong 10 phút
2. Đo nhiệt độ tản nhiệt: < 60°C → OK
3. > 80°C → Thêm tản nhiệt hoặc giảm công suất

**Bước 23: Đóng gói hoàn thiện**
1. Bắt vít nắp hộp (có gioăng cao su)
2. Dán nhãn các nút bấm
3. In hướng dẫn sử dụng (bước 10)

---

## 7. Hiệu Chỉnh & Tối Ưu

### 7.1 Hiệu Chỉnh Tần Số

**Phương pháp 1: Dùng máy phân tích phổ (chính xác nhất)**
1. Kết nối đầu dò của máy phân tích phổ vào đầu ra RF (qua attenuator 30dB để bảo vệ máy)
2. Bật jammer ở mức công suất thấp (10%)
3. Xoay biến trở tune hoặc thay đổi tần số từ code
4. Quan sát đỉnh tần số trên máy

**Phương pháp 2: Dùng đài SDR (RTL-SDR, HackRF)**
1. Gắn antenna SDR gần jammer (cách 1m)
2. Mở phần mềm SDR (GQRX, SDR#)
3. Quét dải tần: xem đỉnh nhiễu xuất hiện đúng tần số

**Phương pháp 3: Dùng đài FM + diode (không có máy đo)**
1. Tự chế "RF Probe": diode 1N4148 + tụ 10pF + 1MΩ → LED
2. LED sáng mạnh nhất khi cộng hưởng đúng tần số

### 7.2 Hiệu Chỉnh Công Suất

**Mối quan hệ PWM - Công suất:**

| PWM | VCC LM2596 | Công suất WiFi | Phạm vi ước tính |
|-----|-----------|----------------|------------------|
| 0   | 1.25V     | 0 mW (tắt)     | 0m               |
| 50  | 3.0V      | ~50 mW         | 2-5m             |
| 100 | 5.0V      | ~200 mW        | 5-10m            |
| 150 | 7.0V      | ~500 mW        | 10-20m           |
| 200 | 9.0V      | ~1 W           | 20-40m           |
| 255 | 12.0V     | ~2-5 W         | 40-100m+         |

**Công thức ước tính bán kính:**
```
R = R₀ × √(P / P₀)
```
Trong đó:
- R₀: bán kính tại công suất tham chiếu (ví dụ: 10m tại 200mW)
- P: công suất hiện tại
- P₀: công suất tham chiếu

Ví dụ: P = 1W (1000mW), P₀ = 200mW, R₀ = 10m
→ R = 10 × √(1000/200) = 10 × √5 = 22.4m

### 7.3 Tối Ưu Antenna

**Tự chế antenna cho từng band:**

**Antenna 1/4 sóng (dễ làm nhất):**
```
L = 7125 / f(MHz)  (cm)

Ví dụ:
- 900 MHz: L = 7125/900 = 7.9 cm
- 2.4 GHz: L = 7125/2400 = 2.97 cm
- 5.8 GHz: L = 7125/5800 = 1.23 cm
```

**Antenna 5/8 sóng (gain cao hơn):**
```
L = 17813 / f(MHz) (cm)
```

---

## 8. Bảng Tần Số Mục Tiêu

### 8.1 Các Băng Tần Cần Chặn

| Band # | Dải tần (MHz) | Mục tiêu | Ghi chú |
|--------|---------------|----------|---------|
| **BAND 1** | **400-500** | LoRa, IoT, Radio cảnh sát | Tần số thấp, xa nhất |
| | 430-440 | Amateur radio | |
| | 460-470 | Trunking radio | |
| **BAND 2** | **700-1000** | **Cellular 2G/3G/4G** | Băng tần chính |
| | 824-894 | GSM 850 / CDMA | |
| | 880-960 | GSM 900 / EGSM | |
| | 703-748 | LTE Band 13/17 (700MHz) | |
| | 832-862 | LTE Band 20 (800MHz) | |
| **BAND 3** | **1500-2000** | **GPS + 3G/4G** | |
| | 1565-1585 | GPS L1 | |
| | 1575.42 | GPS dân sự (C/A) | |
| | 1600-1630 | GLONASS | |
| | 1710-1880 | DCS 1800 (3G) | |
| | 1920-2170 | UMTS 2100 (3G) | |
| | 2010-2025 | LTE TDD Band 34 | |
| **BAND 4** | **2400-2500** | **WiFi + Bluetooth + ZigBee** | |
| | 2400-2483.5 | WiFi 2.4GHz (kênh 1-13) | |
| | 2402-2480 | Bluetooth Classic | |
| | 2400-2483 | BLE (40 kênh) | |
| | 2405-2480 | ZigBee / Thread | |
| | 2441-2480 | Cordless phone | |
| **BAND 5** | **3400-4200** | **5G NR + C-band** | |
| | 3400-3600 | n77 (5G NR) | |
| | 3600-3800 | n78 (5G NR) | |
| | 3800-4200 | C-band satellite | |
| **BAND 6** | **5000-6000** | **WiFi 5GHz + 5G** | |
| | 5150-5250 | WiFi 5GHz (kênh 36-48) | |
| | 5250-5350 | WiFi 5GHz (kênh 52-64) | |
| | 5470-5725 | WiFi 5GHz (kênh 100-140) | |
| | 5725-5850 | WiFi 5GHz (kênh 149-165) | |
| | 4800-4990 | n79 (5G NR) | |

### 8.2 Camera không dây phổ biến

| Loại camera | Tần số (MHz) | Band cần dùng |
|-------------|-------------|---------------|
| Camera WiFi (2.4GHz) | 2400-2483 | Band 4 |
| Camera WiFi (5GHz) | 5150-5850 | Band 6 |
| Camera analog 2.4GHz | 2400-2483 | Band 4 |
| Camera analog 5.8GHz | 5725-5875 | Band 6 |
| Camera 3G/4G | 700-2700 | Bands 2 + 3 |
| Camera không dây 900MHz | 902-928 | Band 2 |
| Camera hồng ngoại nhiệt | (không RF) | Không chặn được |

---

## 9. Tính Toán Phạm Vi

### 9.1 Công Thức

Phạm vi của jammer phụ thuộc vào:

1. **Công suất phát (P_tx)**: Công suất đầu ra của bộ khuếch đại
2. **Độ lợi antenna (G)**: dBi
3. **Tần số (f)**: MHz
4. **Độ nhạy máy thu (RSSI)**: dBm của thiết bị cần chặn
5. **Môi trường**: Trong nhà / ngoài trời / có vật cản

**Phương trình Friis (không gian tự do):**
```
P_rx = P_tx + G_tx + G_rx - 20log(f) - 20log(d) - 32.44
```
Trong đó:
- P_rx: công suất nhận tại thiết bị (dBm)
- P_tx: công suất phát jammer (dBm)
- G_tx, G_rx: độ lợi antenna (dBi)
- f: tần số (MHz)
- d: khoảng cách (km)

**Điều kiện chặn thành công:**
```
P_rx (jammer) > P_rx (tín hiệu thật) + SNR_margin
```
Với SNR_margin = 10-15 dB (tùy modulation)

### 9.2 Bảng Phạm Vi Tham Khảo

| Band | Công suất | Antenna | Trong nhà | Ngoài trời |
|------|-----------|---------|-----------|------------|
| WiFi 2.4GHz | 100mW | 3dBi | 5-10m | 15-25m |
| WiFi 2.4GHz | 500mW | 3dBi | 10-20m | 30-50m |
| WiFi 2.4GHz | 2W | 5dBi | 20-40m | 50-100m |
| GSM 900 | 100mW | 3dBi | 3-8m | 10-20m |
| GSM 900 | 1W | 3dBi | 10-20m | 30-60m |
| GSM 900 | 5W | 5dBi | 30-50m | 80-150m |
| GPS L1 | 1W | Patch 5dBi | 10-20m | 30-50m |
| 4G LTE | 1W | 3dBi | 8-15m | 20-40m |
| 5G NR | 1W | 3dBi | 5-10m | 15-30m |

### 9.3 Cách Tăng/Giảm Phạm Vi

**Tăng phạm vi:**
1. ↑ Công suất: Tăng PWM → Tăng VCC → Tăng P_out
2. ↑ Gain antenna: 2dBi → 5dBi → 8dBi (antenna Yagi)
3. Hiệu chỉnh SWR: SWR 1:1 = công suất ra tối đa
4. Lên cao: Đặt jammer ở vị trí cao, giữa phòng

**Giảm phạm vi:**
1. ↓ Công suất: Giảm PWM (ví dụ 50/255 = ~20%)
2. ↓ Gain antenna: Dùng antenna 0dBi hoặc antenna giả
3. Thêm attenuator: Cắm attenuator 10-20dB giữa amp và antenna
4. Tường/chắn: Đặt sau tường dày

---

## 10. Sơ Đồ Mạch In (PCB)

### 10.1 Mạch Điều Khiển (Arduino + OLED + Encoder)

```
Kích thước: 100x80mm, 2 lớp

Lớp TOP:
┌────────────────────────────────────────────────────────────────┐
│  [SMA1] [SMA2] [SMA3] [SMA4] [SMA5] [SMA6]                    │
│    │       │       │       │       │       │                   │
│  [Filt1] [Filt2] [Filt3] [Filt4] [Filt5] [Filt6]              │
│    │       │       │       │       │       │                   │
│  [Amp6]  [Amp5]  [Amp4]  [Amp3]  [Amp2]  [Amp1]              │
│    │       │       │       │       │       │                   │
│  [SWR1]  [SWR2]  [SWR3]  [SWR4]  [SWR5]  [SWR6]              │
│    │       │       │       │       │       │                   │
│  ┌┴────────┴────────┴────────┴────────┴────────┴┐              │
│  │           GROUND PLANE (Lớp đồng)              │              │
│  │            (Cách ly RF)                        │              │
│  └──────────────┬─────────────────────────────────┘              │
│                 │                                                │
│  ┌──────────────▼─────────────────────────────────┐              │
│  │         ARDUINO NANO + OLED + ENCODER           │              │
│  │                                                 │              │
│  │  ┌─────────────────────────────────────────┐   │              │
│  │  │          OLED SSD1306                    │   │              │
│  │  └─────────────────────────────────────────┘   │              │
│  │                                                 │              │
│  │     [ENC]  [LED1][LED2][LED3][LED4][LED5][LED6]│              │
│  └─────────────────────────────────────────────────┘              │
│                                                                   │
│   [POT1] [POT2] [POT3] [POT4] [POT5] [POT6]                     │
│   [SW1]  [SW2]  [SW3]  [SW4]  [SW5]  [SW6]                     │
│                                                                   │
│   [DC IN] [PWR SW]                                               │
└────────────────────────────────────────────────────────────────┘

Lớp BOTTOM (GND plane):
┌────────────────────────────────────────────────────────────────┐
│  ████████████████████████████████████████████████████████████  │
│  ██  Toàn bộ là GND plane, không có track nào              ██  │
│  ██  (Trừ các via nối xuống GND)                           ██  │
│  ████████████████████████████████████████████████████████████  │
└────────────────────────────────────────────────────────────────┘
```

### 10.2 Bố Trí Mặt Trước (Front Panel)

```
┌─────────────────────────────────────────────────────────────────┐
│  ╔═══════════════════════════════════════════════════════════╗  │
│  ║                    RF JAMMER V3.0                         ║  │
│  ║              Multi-Band Signal Blocker                    ║  │
│  ╚═══════════════════════════════════════════════════════════╝  │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                     OLED 128x64                           │   │
│  │  [400MHz  ████████████░░░░ 80% H]                         │   │
│  │  [700MHz  ████████░░░░░░░░ 50%  ]                         │   │
│  │  [1.5GHz  ░░░░░░░░░░░░░░░░ OFF ] ←                       │   │
│  │  [2.4GHz  ████████████████ 100% ]                         │   │
│  │  [3.5GHz  ░░░░░░░░░░░░░░░░ OFF ]                         │   │
│  │  [5.0GHz  ████████░░░░░░░░ 50%  ]                         │   │
│  └──────────────────────────────────────────────────────────┘   │
│                                                                  │
│  [BAND 1] ○○○○○○ [BAND 2] ○○○○○○ [BAND 3] ○○○○○○               │
│  400-500 MHz       700-1000 MHz     1.5-2.0 GHz                  │
│  ┌─PWR──┐ ┌─SW──┐  ┌─PWR──┐ ┌─SW──┐  ┌─PWR──┐ ┌─SW──┐          │
│  │ ○○○○○│ │ (●) │  │ ○○○○○│ │ ( ) │  │ ○○○○○│ │ ( ) │          │
│  └──────┘ └─────┘  └──────┘ └─────┘  └──────┘ └─────┘          │
│                                                                  │
│  [BAND 4] ○○○○○○ [BAND 5] ○○○○○○ [BAND 6] ○○○○○○               │
│  2.4-2.5 GHz      3.5-4.0 GHz     5.0-6.0 GHz                    │
│  ┌─PWR──┐ ┌─SW──┐  ┌─PWR──┐ ┌─SW──┐  ┌─PWR──┐ ┌─SW──┐          │
│  │ ○○○○○│ │ (●) │  │ ○○○○○│ │ ( ) │  │ ○○○○○│ │ (●) │          │
│  └──────┘ └─────┘  └──────┘ └─────┘  └──────┘ └─────┘          │
│                                                                  │
│           [ROTARY ENCODER]                                       │
│                ┌─────┐                                           │
│                │  ◄ ► │ ← Xoay: chọn band | Nhấn: bật/tắt        │
│                │  ●  │  Giữ 2s: hopping mode                     │
│                └─────┘                                           │
│                                                                  │
│  [DC 12V IN] ●═══════════════● [POWER]                          │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ [SMA1] [SMA2] [SMA3] [SMA4] [SMA5] [SMA6]                │   │
│  │ 400M    700M    1.5G    2.4G    3.5G    5.0G             │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘

Chú thích:
  ○○○○○○ = Biến trở (Potentiometer) chỉnh công suất
  (●) = Switch bật (ON)
  ( ) = Switch tắt (OFF)
  ◄ ► = Rotary Encoder
```

---

## 11. Debug & Khắc Phục

### 11.1 VCO không dao động

| Triệu chứng | Nguyên nhân | Cách khắc phục |
|------------|-------------|-----------------|
| Không thấy sóng trên oscilloscope | Sai phân cực BF199 | Kiểm tra R1, R2 (47Ω-100Ω) |
| | Varactor chết | Đo điện dung varactor (thay đổi theo V) |
| | Cuộn cảm sai giá trị | Kiểm tra L với LCR meter |
| | Thiếu GND plane | Thêm via GND gần VCO |
| Dao động nhưng yếu | Tụ C1-C3 sai | Thay tụ gốm NPO/C0G |
| | BF199 hỏng | Thay BF199 mới |
| Tần số không thay đổi | Varactor ngắn/chập | Đo 2 đầu varactor, thay nếu chập |

### 11.2 Khuếch đại không có công suất

| Triệu chứng | Nguyên nhân | Cách khắc phục |
|------------|-------------|-----------------|
| Pre-amp ra yếu | Sai tụ ghép tầng (C1-C3) | Dùng đúng 10pF-100pF |
| | Mất mass ở tầng | Kiểm tra GND |
| Driver nóng nhưng không ra | Mất matching | Chỉnh L1, C1 |
| | Sai tần số | Dùng máy đo kiểm tra f |
| Final nóng ~70°C, không RF | Quá tải | Giảm công suất driver |
| | SWR cao | Kiểm tra antenna + LPF |

### 11.3 Không kết nối được Arduino

| Vấn đề | Giải pháp |
|--------|-----------|
| Cổng COM không thấy | Cài driver CH340/CP2102 |
| | Dùng cáp USB data (không phải sạc) |
| Nạp code lỗi | Board: Arduino Nano, Old Bootloader |
| | Reset Arduino ngay khi "Uploading" |
| Serial Monitor không ra | Check baud rate (115200) |

### 11.4 OLED không hiển thị

| Vấn đề | Giải pháp |
|--------|-----------|
| Màn hình trắng/đen | Kiểm tra I2C: A4(SDA) ↔ OLED SDA |
| | Chạy scan I2C: `Wire.begin(); scan` |
| | Địa chỉ có thể là 0x3C hoặc 0x3D |
| Hiện chữ nhưng nhấp nháy | Thiếu tụ lọc nguồn gần OLED |
| | Thêm 100nF + 10μF gần OLED |

### 11.5 Jammer hoạt động yếu (bán kính < 1m)

| Nguyên nhân | Kiểm tra | Khắc phục |
|------------|----------|-----------|
| Antenna sai tần số | Đo SWR (cần < 2:1) | Cắt/chỉnh antenna đúng chiều dài |
| LPF quá hẹp | Đo phổ tần | Mở rộng LPF hoặc bỏ qua (nếu cần) |
| Công suất thấp | Đo VCC tại transistor | Chỉnh LM2596 lên 12V |
| Matching kém | Dùng NanoVNA | Thiết kế lại matching network |
| VCC thấp do tụ lọc | Kiểm tra tụ hóa lớn | Thay tụ 1000μF mới |

---

## PHỤ LỤC

### A. Công Thức Tính Toán Nhanh

**Chiều dài antenna 1/4 sóng:**
```
L(cm) = 7125 / f(MHz)
```

**Bước sóng:**
```
λ(m) = 300 / f(MHz)
```

**Công suất dBm → W:**
```
P(W) = 10^((P(dBm) - 30) / 10)
```

**Công suất W → dBm:**
```
P(dBm) = 10 × log10(P(W) × 1000)
```

### B. Tính Công Suất (PWM → VCC → P_out)

```
VCC = 1.25 × (1 + PWM / 255)    // LM2596, ADJ pin qua PWM
P_out ≈ VCC² / (2 × R_load)     // Với R_load ≈ 50Ω (antenna)
```

Ví dụ:
- PWM = 128 (50%): VCC = 1.25 × (1 + 128/255) = 1.88V
  P_out ≈ 1.88² / 100 = 35mW
- PWM = 200 (78%): VCC = 1.25 × (1 + 200/255) = 2.23V
  P_out ≈ 2.23² / 100 = 50mW
- PWM = 255 (100%): VCC = 12V
  P_out ≈ 144 / 100 = 1.44W

### C. Safety Checklist (Trước khi bật)

- [ ] Tất cả transistor đã gắn tản nhiệt + quạt?
- [ ] Antenna đã gắn (không chạy không tải)?
- [ ] SWR < 2:1?
- [ ] Nhiệt độ môi trường < 30°C?
- [ ] Nguồn > 11V (pin còn đầy)?
- [ ] Không có người trong bán kính 2m trước antenna?
- [ ] Công tắc nguồn ở vị trí OFF?

---

**Tài liệu tham khảo:**  
- NE555 Datasheet (Texas Instruments)  
- MAX2870 Datasheet (Maxim Integrated)  
- AD9850 Datasheet (Analog Devices)  
- MRF454 Datasheet (NXP)  
- "RF Circuit Design" - Chris Bowick  
- "Secrets of RF Circuit Design" - Joseph Carr  
- "Experimental Methods in RF Design" - Hayward, Campbell, Larkin

---

*Tài liệu này được tạo ra với mục đích giáo dục và nghiên cứu. Trước khi chế tạo, hãy tìm hiểu quy định pháp luật tại địa phương về thiết bị gây nhiễu sóng.*
