# UART Communication với STM32

## 📖 Giới thiệu về UART

**UART (Universal Asynchronous Receiver/Transmitter)** là giao thức truyền thông nối tiếp bất đồng bộ, được sử dụng rộng rãi trong các hệ thống nhúng để giao tiếp giữa vi điều khiển và các thiết bị ngoại vi.

### Đặc điểm chính:
- **Bất đồng bộ**: Không cần tín hiệu clock chung
- **Full-duplex**: Có thể truyền và nhận đồng thời
- **Đơn giản**: Chỉ cần 2 dây (TX, RX) + GND

### Ứng dụng thực tế:
| Ứng dụng | Mô tả |
|----------|-------|
| Debug/Console | In log, debug qua terminal |
| GPS Module | Nhận dữ liệu vị trí |
| Bluetooth (HC-05) | Giao tiếp không dây |
| GSM Module | Gửi SMS, gọi điện |
| RS-232/RS-485 | Giao tiếp công nghiệp |

---

## 🔧 UART với STM32

### Các UART trên STM32F103C8T6 (Blue Pill):
| UART | TX Pin | RX Pin | Ghi chú |
|------|--------|--------|---------|
| USART1 | PA9 | PA10 | Thường dùng cho debug |
| USART2 | PA2 | PA3 | Kết nối ngoại vi |
| USART3 | PB10 | PB11 | Kết nối ngoại vi |

### Sơ đồ kết nối với USB-TTL:
```
STM32          USB-TTL
─────          ───────
PA9 (TX)  ───► RX
PA10 (RX) ◄─── TX
GND       ──── GND
```

> ⚠️ **Lưu ý**: TX của STM32 nối với RX của USB-TTL và ngược lại!

---

## ⚙️ Cách Config trong STM32CubeMX

### Bước 1: Chọn USART
1. Mở **Pinout & Configuration**
2. Tìm `USART1` trong danh sách Connectivity
3. Chọn Mode: **Asynchronous**

### Bước 2: Cấu hình Parameters
| Parameter | Giá trị thường dùng |
|-----------|---------------------|
| Baud Rate | 115200 |
| Word Length | 8 Bits |
| Stop Bits | 1 |
| Parity | None |
| Hardware Flow Control | None |

### Bước 3: Enable Interrupt (nếu cần)
1. Vào tab **NVIC Settings**
2. Tick ✅ **USART1 global interrupt**

### Bước 4: Generate Code
1. **Project Manager** → Đặt tên project
2. **Generate Code**

---

## ✅ Task List - Các bước thực hiện

### Setup Project
- [ ] Tạo project mới trong STM32CubeIDE
- [ ] Cấu hình System Clock (HSE, PLL)
- [ ] Cấu hình USART1 Asynchronous mode
- [ ] Enable USART1 interrupt (nếu dùng IT mode)
- [ ] Generate code

### Coding
- [ ] Include thư viện `<stdio.h>` và `<string.h>`
- [ ] Khai báo buffer nhận dữ liệu
- [ ] Viết hàm callback `HAL_UART_RxCpltCallback()`
- [ ] Gọi `HAL_UART_Receive_IT()` để bắt đầu nhận
- [ ] Xử lý lệnh trong callback

### Testing
- [ ] Kết nối USB-TTL với máy tính
- [ ] Mở terminal (Hercules, Putty, Arduino Serial Monitor)
- [ ] Cấu hình đúng COM port và Baud rate
- [ ] Test gửi/nhận dữ liệu

---

## ❌ Lỗi thường gặp và cách Fix

### 1. Không nhận được dữ liệu
| Nguyên nhân | Cách fix |
|-------------|----------|
| TX/RX đấu ngược | Đổi dây TX ↔ RX |
| Baud rate không khớp | Kiểm tra cả 2 phía cùng baud rate |
| Chưa gọi `HAL_UART_Receive_IT()` | Gọi trong `main()` trước vòng while |
| Interrupt chưa enable | Tick USART1 interrupt trong CubeMX |

### 2. LED chỉ nháy rồi tắt (không giữ trạng thái)
```c
// ❌ Sai - Toggle LED
HAL_GPIO_TogglePin(GPIOC, GPIO_PIN_13);

// ✅ Đúng - Set/Reset cụ thể
if (received_char == '1') {
    HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_RESET);  // LED ON
}
else if (received_char == '0') {
    HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_SET);    // LED OFF
}
```

### 3. LED sáng mờ hoặc không ổn định
- **Nguyên nhân**: Điện áp 3.3V không đủ cho LED
- **Fix**: Cấp nguồn 5V cho LED qua điện trở hạn dòng (330Ω - 1kΩ)

### 4. Nhận sai ký tự hoặc ký tự lạ
| Nguyên nhân | Cách fix |
|-------------|----------|
| Sai baud rate | Đồng bộ baud rate 2 phía |
| Sai Word Length | Kiểm tra 8-bit hay 9-bit |
| Nhiễu tín hiệu | Dùng dây ngắn, thêm tụ lọc |

### 5. UART chỉ nhận 1 lần
```c
// ❌ Sai - Không gọi lại Receive_IT
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart) {
    // Xử lý dữ liệu...
}

// ✅ Đúng - Gọi lại để nhận tiếp
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart) {
    // Xử lý dữ liệu...
    HAL_UART_Receive_IT(&huart1, &rx_buffer[0], 1);  // Nhận tiếp!
}
```

---

## 📁 Cấu trúc Project

```
UART/
├── Core/
│   ├── Inc/
│   │   └── main.h
│   └── Src/
│       └── main.c          ← Code chính
├── docs/
│   └── CODE_GUIDE.md       ← Hướng dẫn code
└── README.md               ← File này
```

---

## 🔗 Tài liệu tham khảo
- [STM32 HAL UART Documentation](https://www.st.com/resource/en/user_manual/um1785-description-of-stm32f0-hal-and-lowlayer-drivers-stmicroelectronics.pdf)
- [STM32CubeIDE User Guide](https://www.st.com/resource/en/user_manual/um2609-stm32cubeide-user-guide-stmicroelectronics.pdf)
