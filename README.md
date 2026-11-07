# STM32 UART · Điều khiển LED qua terminal

## Tổng quan

Demo **USART1 trên STM32F103C8T6 (Blue Pill)**. Gửi `1` để bật LED PC13, `0` để tắt. Firmware echo ký tự và gửi trạng thái để có thể kiểm tra cả hai chiều UART.

## Phần cứng

![Sơ đồ đấu nối Blue Pill và USB–TTL 3,3 V](docs/images/uart-wiring.svg)

| Blue Pill | USB–TTL | Vai trò |
| --- | --- | --- |
| PA9 | RXD | USART1 TX |
| PA10 | TXD | USART1 RX |
| GND | GND | Mass chung |

Dùng adapter USB–TTL mức logic 3,3 V. Cấp nguồn cho Blue Pill riêng phù hợp với board.

## Build và nạp firmware

Cần **Python 3.9+**, **GNU Arm Embedded** (`arm-none-eabi-gcc`, `objcopy`, `size`) và gói **STM32CubeF1 v1.8.7**. Script dùng HAL và startup từ gói ST, cùng linker script 64 KiB flash / 20 KiB SRAM của STM32F103C8.

```sh
python build.py
```

Đặt GNU Arm Embedded vào `PATH`, hoặc chỉ định thư mục `bin` của toolchain. Nếu gói HAL nằm ngoài thư mục mặc định `~/STM32Cube/Repository/STM32Cube_FW_F1_V1.8.7`, truyền đường dẫn tới thư mục chứa `Drivers`.

```sh
python build.py --sdk "<STM32CubeF1 root>" --toolchain "<GNU Arm bin>"
```

Build tạo `build/firmware.elf`, `build/firmware.hex` và `build/firmware.bin`. Nạp `.hex` bằng STM32CubeProgrammer qua ST-LINK/SWD, rồi reset board. Có thể dùng `.elf` để nạp symbol và debug. Firmware mẫu dùng HSE 8 MHz và SYSCLK 72 MHz.

Tài liệu công cụ: [STM32CubeF1](https://github.com/STMicroelectronics/STM32CubeF1) · [STM32CubeProgrammer](https://www.st.com/en/development-tools/stm32cubeprog.html).

## Chạy thử

1. Mở cổng serial của USB–TTL ở **115200 bit/s, 8 data bit, no parity, 1 stop bit**, tắt flow control.
2. Mở terminal trước khi reset board để thấy chuỗi `So: 1` đến `So: 30` và `Done!`.
3. Gửi `1` và kiểm tra LED sáng cùng phản hồi `1 -> LED ON!`. Gửi `0` và kiểm tra LED tắt. Phản hồi kèm bộ đếm lệnh.

## Cấu trúc dự án

| Đường dẫn | Vai trò |
| --- | --- |
| [`Core/Inc/`](Core/Inc/) | Header ứng dụng và thư viện. |
| [`Core/Src/`](Core/Src/) | USART1, hàng đợi RX và điều khiển LED PC13. |
| [`firmware/`](firmware/) | Khởi tạo phần cứng, SysTick và linker script. |
| [`docs/`](docs/) | Hướng dẫn kỹ thuật và sơ đồ SVG trong `docs/images/`. |
| [`build.py`](build.py) | Build firmware từ mã nguồn. |

## Tài liệu

- [Hướng dẫn kỹ thuật](docs/CODE_GUIDE.md).
