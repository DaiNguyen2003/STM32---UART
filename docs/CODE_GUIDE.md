# Hướng dẫn Code UART với STM32 HAL Library

## 📚 Các hàm UART trong HAL Library

### 1. Hàm truyền dữ liệu (Blocking Mode)

```c
HAL_StatusTypeDef HAL_UART_Transmit(
    UART_HandleTypeDef *huart,  // Con trỏ đến UART handle
    uint8_t *pData,             // Con trỏ đến dữ liệu cần gửi
    uint16_t Size,              // Số byte cần gửi
    uint32_t Timeout            // Thời gian chờ (ms)
);
```

**Ví dụ:**
```c
// Gửi chuỗi đơn giản
char msg[] = "Hello World!\r\n";
HAL_UART_Transmit(&huart1, (uint8_t*)msg, strlen(msg), 100);

// Gửi số
char buffer[32];
int value = 123;
sprintf(buffer, "Value: %d\r\n", value);
HAL_UART_Transmit(&huart1, (uint8_t*)buffer, strlen(buffer), 100);
```

---

### 2. Hàm nhận dữ liệu (Blocking Mode)

```c
HAL_StatusTypeDef HAL_UART_Receive(
    UART_HandleTypeDef *huart,  // Con trỏ đến UART handle
    uint8_t *pData,             // Buffer lưu dữ liệu nhận
    uint16_t Size,              // Số byte cần nhận
    uint32_t Timeout            // Thời gian chờ (ms)
);
```

**Ví dụ:**
```c
uint8_t rx_data[10];

// Chờ nhận 1 byte, timeout 1 giây
if (HAL_UART_Receive(&huart1, rx_data, 1, 1000) == HAL_OK) {
    // Nhận thành công
    HAL_UART_Transmit(&huart1, rx_data, 1, 100);  // Echo lại
}
```

---

### 3. Hàm nhận dữ liệu (Interrupt Mode) ⭐

```c
HAL_StatusTypeDef HAL_UART_Receive_IT(
    UART_HandleTypeDef *huart,  // Con trỏ đến UART handle
    uint8_t *pData,             // Buffer lưu dữ liệu nhận
    uint16_t Size               // Số byte cần nhận
);
```

**Ưu điểm**: Không block CPU, nhận dữ liệu trong nền

**Ví dụ:**
```c
// Khai báo buffer toàn cục
uint8_t rx_buffer[64];

int main(void) {
    // ... khởi tạo ...
    
    // Bắt đầu nhận 1 byte bằng interrupt
    HAL_UART_Receive_IT(&huart1, rx_buffer, 1);
    
    while (1) {
        // CPU rảnh làm việc khác
    }
}
```

---

### 4. Callback khi nhận xong (Interrupt Mode) ⭐

```c
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart);
```

**Ví dụ đầy đủ:**
```c
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart) {
    if (huart->Instance == USART1) {
        uint8_t received = rx_buffer[0];
        
        // Echo lại ký tự nhận được
        HAL_UART_Transmit(&huart1, &received, 1, 10);
        
        // Xử lý lệnh
        if (received == '1') {
            HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_RESET);
        }
        else if (received == '0') {
            HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_SET);
        }
        
        // QUAN TRỌNG: Gọi lại để nhận tiếp
        HAL_UART_Receive_IT(&huart1, rx_buffer, 1);
    }
}
```

---

### 5. Hàm truyền dữ liệu (Interrupt Mode)

```c
HAL_StatusTypeDef HAL_UART_Transmit_IT(
    UART_HandleTypeDef *huart,
    uint8_t *pData,
    uint16_t Size
);
```

**Callback khi truyền xong:**
```c
void HAL_UART_TxCpltCallback(UART_HandleTypeDef *huart) {
    // Truyền xong, có thể gửi dữ liệu tiếp
}
```

---

### 6. Hàm DMA Mode (Hiệu suất cao)

```c
// Truyền bằng DMA
HAL_UART_Transmit_DMA(&huart1, tx_buffer, size);

// Nhận bằng DMA
HAL_UART_Receive_DMA(&huart1, rx_buffer, size);
```

---

## 💡 Code mẫu trong main.c

### Code điều khiển LED qua UART

```c
/* USER CODE BEGIN Includes */
#include <stdio.h>
#include <string.h>
/* USER CODE END Includes */

/* USER CODE BEGIN PV */
uint8_t rx_buffer[64];
volatile uint32_t led_on_count = 0;
volatile uint32_t led_off_count = 0;
/* USER CODE END PV */

/* USER CODE BEGIN PFP */
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart) {
    if (huart->Instance == USART1) {
        char msg[64];
        uint8_t received = rx_buffer[0];
        
        // Echo ký tự
        HAL_UART_Transmit(&huart1, &received, 1, 10);
        
        // Xử lý lệnh
        switch (received) {
            case '1':
                led_on_count++;
                HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_RESET);
                sprintf(msg, " -> LED ON! (%lu)\r\n", led_on_count);
                HAL_UART_Transmit(&huart1, (uint8_t*)msg, strlen(msg), 100);
                break;
                
            case '0':
                led_off_count++;
                HAL_GPIO_WritePin(GPIOC, GPIO_PIN_13, GPIO_PIN_SET);
                sprintf(msg, " -> LED OFF! (%lu)\r\n", led_off_count);
                HAL_UART_Transmit(&huart1, (uint8_t*)msg, strlen(msg), 100);
                break;
                
            case '\r':
            case '\n':
                HAL_UART_Transmit(&huart1, (uint8_t*)"\r\n", 2, 10);
                break;
        }
        
        // Nhận tiếp
        HAL_UART_Receive_IT(&huart1, rx_buffer, 1);
    }
}
/* USER CODE END PFP */

int main(void) {
    HAL_Init();
    SystemClock_Config();
    MX_GPIO_Init();
    MX_USART1_UART_Init();
    
    /* USER CODE BEGIN 2 */
    // Bắt đầu nhận UART
    HAL_UART_Receive_IT(&huart1, rx_buffer, 1);
    
    // Gửi thông báo khởi động
    char *welcome = "=== UART LED Control ===\r\n";
    char *help = "Send '1' to turn ON, '0' to turn OFF\r\n";
    HAL_UART_Transmit(&huart1, (uint8_t*)welcome, strlen(welcome), 100);
    HAL_UART_Transmit(&huart1, (uint8_t*)help, strlen(help), 100);
    /* USER CODE END 2 */
    
    while (1) {
        // CPU rảnh
    }
}
```

---

## 📊 So sánh các Mode

| Mode | Ưu điểm | Nhược điểm | Khi nào dùng |
|------|---------|------------|--------------|
| **Blocking** | Đơn giản | Block CPU | Debug, gửi ít dữ liệu |
| **Interrupt** | Không block | Phức tạp hơn | Nhận dữ liệu bất kỳ lúc nào |
| **DMA** | Hiệu suất cao | Cần setup DMA | Truyền lượng lớn dữ liệu |

---

## ⚡ Tips & Tricks

### 1. Redirect printf() qua UART
```c
// Thêm vào USER CODE BEGIN 0
int __io_putchar(int ch) {
    HAL_UART_Transmit(&huart1, (uint8_t*)&ch, 1, 10);
    return ch;
}

// Bây giờ có thể dùng printf()
printf("Hello %d\r\n", 123);
```

### 2. Nhận chuỗi có kết thúc Enter
```c
uint8_t rx_buffer[64];
uint8_t rx_index = 0;

void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart) {
    uint8_t received = rx_buffer[rx_index];
    
    if (received == '\r' || received == '\n') {
        rx_buffer[rx_index] = '\0';  // Kết thúc chuỗi
        
        // Xử lý chuỗi hoàn chỉnh
        process_command((char*)rx_buffer);
        
        rx_index = 0;  // Reset
    } else {
        rx_index++;
        if (rx_index >= 64) rx_index = 0;  // Tránh overflow
    }
    
    HAL_UART_Receive_IT(&huart1, &rx_buffer[rx_index], 1);
}
```

### 3. Kiểm tra lỗi UART
```c
void HAL_UART_ErrorCallback(UART_HandleTypeDef *huart) {
    if (huart->ErrorCode & HAL_UART_ERROR_ORE) {
        // Overrun error - đọc quá chậm
    }
    if (huart->ErrorCode & HAL_UART_ERROR_FE) {
        // Frame error - sai baud rate
    }
    
    // Reset và nhận lại
    HAL_UART_Receive_IT(&huart1, rx_buffer, 1);
}
```

---

## 🔗 Xem thêm
- [README.md](../README.md) - Hướng dẫn tổng quan
- [main.c](../Core/Src/main.c) - Code thực tế
