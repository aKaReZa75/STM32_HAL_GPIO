## Complete Examples

### Example 1: Simple LED Control

```c
#include "main.h"

int main(void) {
    // Initialize HAL
    HAL_Init();
    SystemClock_Config();
    MX_GPIO_Init(); // CubeMX-generated
    
    while(1) {
        // Blink LED
        HAL_GPIO_TogglePin(LED_GPIO_Port, LED_Pin);
        HAL_Delay(500);
    }
}
```

### Example 2: Button-Controlled LED

```c
#include "main.h"

int main(void) {
    HAL_Init();
    SystemClock_Config();
    MX_GPIO_Init();
    
    while(1) {
        // Read button state (active low)
        if(HAL_GPIO_ReadPin(KEY_GPIO_Port, KEY_Pin) == GPIO_PIN_RESET) {
            HAL_GPIO_WritePin(LED_GPIO_Port, LED_Pin, GPIO_PIN_SET);
        } else {
            HAL_GPIO_WritePin(LED_GPIO_Port, LED_Pin, GPIO_PIN_RESET);
        }
    }
}
```

---

## Important Notes

### Output Modes Explained

#### Push-Pull Output (`GPIO_MODE_OUTPUT_PP`)

**Description:**  
The pin can actively drive both HIGH and LOW states.

**Characteristics:**
- Strong HIGH output (VDD)
- Strong LOW output (GND)
- Can source and sink current
- Most common output mode

**Use Cases:**
- LEDs
- General digital outputs
- Logic signals

---

#### Open-Drain Output (`GPIO_MODE_OUTPUT_OD`)

**Description:**  
The pin can only actively drive LOW. HIGH state requires external pull-up resistor.

**Characteristics:**
- Strong LOW output (GND)
- HIGH state is high-impedance (floating)
- Requires external pull-up resistor
- Allows multiple devices on same line

**Use Cases:**
- I2C communication lines (SDA, SCL)
- 1-Wire communication
- Wired-OR configurations
- Level shifting to higher voltages


> [!NOTE]
> - Push-pull is faster than open-drain
> - Open-drain allows voltage level shifting
> - Always use pull-up resistor with open-drain outputs

---

### Pull Resistors

#### Internal Pull-Up (`GPIO_PULLUP`)

**Characteristics:**
- Typical resistance: 30-50kΩ
- Pulls pin to VDD when floating
- Weak pull (can be overridden)

**Use Cases:**
- Button inputs (active low)
- I2C lines (as backup, external preferred)
- Default HIGH state for inputs

---

#### Internal Pull-Down (`GPIO_PULLDOWN`)

**Characteristics:**
- Typical resistance: 30-50kΩ
- Pulls pin to GND when floating
- Weak pull (can be overridden)

**Use Cases:**
- Button inputs (active high)
- Default LOW state for inputs
- Preventing floating inputs

---

#### No Pull (`GPIO_NOPULL`)

**Characteristics:**
- Pin floats when not driven
- No internal resistor connected
- Minimum power consumption

**Use Cases:**
- Outputs (don't need pull resistors)
- Inputs with external pull resistors
- Analog inputs
- High-speed signals

> [!TIP]
> **Choosing Pull Resistor Type:**
> - **Active LOW button:** Use `GPIO_PULLUP`
> - **Active HIGH button:** Use `GPIO_PULLDOWN`
> - **Outputs:** Use `GPIO_NOPULL`
> - **I2C lines:** Use external pull-ups (2.2-4.7kΩ)

---

### Output Speed Selection

The output speed setting affects the slew rate (how fast the pin can change state).

| Speed Setting | Frequency Range | Rise/Fall Time | Power | Use Case |
|---------------|-----------------|----------------|-------|----------|
| `GPIO_SPEED_FREQ_LOW` | Up to 8 MHz | Slower | Lower | LEDs, buttons, low-speed signals |
| `GPIO_SPEED_FREQ_MEDIUM` | Up to 50 MHz | Medium | Medium | General purpose, UART, I2C |
| `GPIO_SPEED_FREQ_HIGH` | Up to 100 MHz | Fast | Higher | SPI, fast signals |
| `GPIO_SPEED_FREQ_VERY_HIGH` | Up to 180 MHz | Very fast | Highest | High-speed interfaces, SDIO |

**Guidelines:**
- Use the **lowest speed** that meets your requirements
- Higher speed = more EMI (electromagnetic interference)
- Higher speed = more power consumption
- For most applications, LOW or MEDIUM is sufficient

```c
// LED example - LOW speed is sufficient
GPIO_InitStruct.Pin = LED_Pin;
GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;

// SPI communication - HIGH speed may be needed
GPIO_InitStruct.Pin = SPI_SCLK_Pin;
GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;
```

---

### Alternate Function Mode

Some pins can be configured as alternate functions to be controlled by peripherals (UART, SPI, I2C, TIM, etc.).

```c
// Example: Configure PA9 as USART1_TX
GPIO_InitTypeDef GPIO_InitStruct = {0};

GPIO_InitStruct.Pin = GPIO_PIN_9;
GPIO_InitStruct.Mode = GPIO_MODE_AF_PP;        // Alternate function push-pull
GPIO_InitStruct.Pull = GPIO_NOPULL;
GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_HIGH;
GPIO_InitStruct.Alternate = GPIO_AF7_USART1;   // AF7 = USART1
HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
```

> [!NOTE]
> - Alternate function mapping varies by STM32 family
> - Refer to datasheet for AF mapping tables
> - CubeMX automatically configures AF when you enable peripherals

---

## FAQ (Frequently Asked Questions)

**Q: How much current can a GPIO pin source/sink?**  
A: Typically 20-25mA per pin, with a maximum total per port. Check your specific STM32 datasheet for exact limits. Always use current-limiting resistors for LEDs.

**Q: What happens if I exceed the current limit?**  
A: The pin or entire MCU may be damaged. Always calculate and limit current using resistors or transistors for high-current loads.

**Q: Can I connect 5V signals to GPIO pins?**  
A: Most STM32 GPIO pins are 5V-tolerant when configured as inputs (check datasheet). However, outputs operate at 3.3V. Use level shifters for proper 5V interfacing.

**Q: Why do I need pull resistors for buttons?**  
A: Without pull resistors, inputs float and read random values. Pull resistors ensure a defined state when the button is not pressed.

**Q: How do I detect a button press without missing it?**  
A: Use external interrupts (EXTI) instead of polling. Configure the pin with `GPIO_MODE_IT_FALLING` or `GPIO_MODE_IT_RISING` and implement the interrupt callback.

**Q: Can I read the output state of an output pin?**  
A: Yes, `HAL_GPIO_ReadPin()` reads the Output Data Register (ODR) for output pins, showing the current output state.

**Q: What's the best way to control many LEDs?**  
A: For more than 8-10 LEDs, consider using LED driver ICs (like 74HC595 shift register) or multiplexing techniques to save GPIO pins.

---

# 🌟 Support Me
If you found this repository useful:
- Subscribe to my [YouTube Channel](https://www.youtube.com/@aKaReZa75).
- Share this repository with others.
- Give this repository and my other repositories a star.
- Follow my [GitHub account](https://github.com/aKaReZa75).

# ✉️ Contact Me
Feel free to reach out to me through any of the following platforms:
- 📧 [Email: aKaReZa75@gmail.com](mailto:aKaReZa75@gmail.com)
- 🎥 [YouTube: @aKaReZa75](https://www.youtube.com/@aKaReZa75)
- 💼 [LinkedIn: @akareza75](https://www.linkedin.com/in/akareza75)