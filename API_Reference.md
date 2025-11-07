# STM32 HAL GPIO - API Reference

This document provides a complete reference for STM32 HAL GPIO (General Purpose Input/Output) driver functions. The GPIO peripheral allows microcontroller pins to be configured as digital inputs or outputs, enabling interaction with external hardware components like LEDs, buttons, sensors, and other digital devices.

## Overview

The STM32 HAL GPIO driver provides a hardware abstraction layer for controlling GPIO pins across all STM32 microcontroller families. It offers a consistent API regardless of the specific STM32 series you're using (F0, F1, F4, L4, H7, etc.).

### Key Features

- **Flexible Pin Configuration:** Configure pins as input, output, alternate function, or analog
- **Multiple Output Modes:** Push-pull or open-drain output configurations
- **Configurable Pull Resistors:** Internal pull-up, pull-down, or no pull
- **Speed Control:** Adjustable output speed (Low, Medium, High, Very High)
- **Pin Locking:** Prevent accidental pin reconfiguration
- **Interrupt Support:** External interrupt/event configuration (covered separately)
- **Hardware Abstraction:** Consistent API across all STM32 families

---

## GPIO Ports and Pins

### Port Naming Convention

STM32 microcontrollers organize GPIO pins into ports, each containing up to 16 pins:

```
Port A: PA0, PA1, PA2, ..., PA15
Port B: PB0, PB1, PB2, ..., PB15
Port C: PC0, PC1, PC2, ..., PC15
Port D: PD0, PD1, PD2, ..., PD15
Port E: PE0, PE1, PE2, ..., PE15
Port F: PF0, PF1, PF2, ..., PF15
...
```

**Available Ports Vary by Device:**
- Entry-level MCUs: Typically GPIOA-GPIOC
- Mid-range MCUs: Typically GPIOA-GPIOE
- High-performance MCUs: May include GPIOA-GPIOK

---

## STM32CubeMX Configuration

Before using GPIO functions in code, you **MUST** configure the pins using STM32CubeMX or manually in your initialization code.

### CubeMX GPIO Configuration Steps:

1. **Open STM32CubeMX Project**
   - Create new project or open existing one
   - Select your target STM32 microcontroller

2. **Navigate to Pinout & Configuration**
   - Click on "Pinout & Configuration" tab
   - View the chip pinout diagram

3. **Select GPIO Pin**
   - Click on the desired pin in the chip diagram
   - A dropdown menu will appear

![GPIO_INTPUT](Images/GPIO_INPUT.png)

4. **Set GPIO Mode**
   - For output pins: Select `GPIO_Output`
   - For input pins: Select `GPIO_Input`
   - For alternate functions: Select appropriate peripheral function
   - For analog: Select `ADC`, `DAC`, or `Analog`

![GPIO_OUTTPUT](Images/GPIO_OUTPUT.png)

5. **Configure GPIO Settings**
   - Click on "System Core" → "GPIO" in the left panel
   - Select the configured pin
   - Set the following parameters:

![CUBE_GPIO](Images/CUBE_GPIO.png)

   **For Output Pins:**
   - GPIO mode: `Output Push Pull` or `Output Open Drain`
   - GPIO Pull-up/Pull-down: `No pull-up and no pull-down`, `Pull-up`, or `Pull-down`
   - Maximum output speed: `Low`, `Medium`, `High`, or `Very High`
   - Initial output level: `Low` or `High`

![CUBE_OUTPUT](Images/CUBE_OUTPUT.png)

   **For Input Pins:**
   - GPIO Pull-up/Pull-down: `No pull-up and no pull-down`, `Pull-up`, or `Pull-down`
   - GPIO mode: `Input mode` (with or without interrupts)

![CUBE_INPUT](Images/CUBE_INPUT.png)

6. **Assign User Label (Optional but Recommended)**
   - Enter a descriptive name (e.g., `LED_RED`, `BUTTON_USER`, `SENSOR_DATA`)
   - This creates readable macros in generated code

7. **Generate Code**
   - Click "Project" → "Generate Code"
   - CubeMX generates initialization code in `main.c` and `gpio.c`

### Example Configuration

**LED on PB5 (Output):**
- GPIO mode: Output Push Pull
- GPIO Pull-up/Pull-down: No pull-up and no pull-down
- Maximum output speed: Low
- User Label: `LED`

**Button on PA10 (Input):**
- GPIO mode: Input mode
- GPIO Pull-up/Pull-down: Pull-up
- User Label: `KEY`

---

## GPIO Pin Definitions

After CubeMX code generation, GPIO pins are defined in `main.h` as macros:

```c
/* User Label Definitions */
#define LED_Pin GPIO_PIN_5
#define LED_GPIO_Port GPIOB

#define KEY_Pin GPIO_PIN_10
#define KEY_GPIO_Port GPIOA
```

These macros are used with HAL GPIO functions throughout your code.

---

## API Functions

### Initialization and Configuration

#### `void HAL_GPIO_Init(GPIO_TypeDef *GPIOx, GPIO_InitTypeDef *GPIO_Init)`

**Description:**  
Initializes GPIO pins according to the specified parameters in the `GPIO_InitTypeDef` structure.

**Parameters:**
- `GPIOx` - GPIO Port (GPIOA, GPIOB, GPIOC, etc.)
- `GPIO_Init` - Pointer to `GPIO_InitTypeDef` structure containing configuration parameters

**Returns:**  
None

**GPIO_InitTypeDef Structure:**
```c
typedef struct {
    uint32_t Pin;       // GPIO pins to be configured (GPIO_PIN_x)
    uint32_t Mode;      // Operating mode (Input, Output, AF, Analog)
    uint32_t Pull;      // Pull-up/Pull-down activation
    uint32_t Speed;     // Output speed (Low, Medium, High, Very High)
    uint32_t Alternate; // Alternate function selection (for AF mode)
} GPIO_InitTypeDef;
```

**Pin Configuration Options:**

| Parameter | Options | Description |
|-----------|---------|-------------|
| **Pin** | `GPIO_PIN_0` to `GPIO_PIN_15`<br>`GPIO_PIN_All` | Pin number(s) to configure |
| **Mode** | `GPIO_MODE_INPUT`<br>`GPIO_MODE_OUTPUT_PP`<br>`GPIO_MODE_OUTPUT_OD`<br>`GPIO_MODE_AF_PP`<br>`GPIO_MODE_AF_OD`<br>`GPIO_MODE_ANALOG` | Input, Output Push-Pull, Output Open-Drain, Alternate Function, Analog |
| **Pull** | `GPIO_NOPULL`<br>`GPIO_PULLUP`<br>`GPIO_PULLDOWN` | No pull, Pull-up, Pull-down |
| **Speed** | `GPIO_SPEED_FREQ_LOW`<br>`GPIO_SPEED_FREQ_MEDIUM`<br>`GPIO_SPEED_FREQ_HIGH`<br>`GPIO_SPEED_FREQ_VERY_HIGH` | Output speed selection |

**Examples:**

```c
// Example 1: Configure PB5 as output (LED)
GPIO_InitTypeDef GPIO_InitStruct = {0};

GPIO_InitStruct.Pin = GPIO_PIN_5;
GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
GPIO_InitStruct.Pull = GPIO_NOPULL;
GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
HAL_GPIO_Init(GPIOB, &GPIO_InitStruct);

// Example 2: Configure PA10 as input with pull-up (KEY)
GPIO_InitStruct.Pin = GPIO_PIN_10;
GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
GPIO_InitStruct.Pull = GPIO_PULLUP;
HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);

// Example 3: Configure multiple pins at once
GPIO_InitStruct.Pin = GPIO_PIN_0 | GPIO_PIN_1 | GPIO_PIN_2;
GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
GPIO_InitStruct.Pull = GPIO_NOPULL;
GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_MEDIUM;
HAL_GPIO_Init(GPIOB, &GPIO_InitStruct);

// Example 4: Using CubeMX-generated labels
GPIO_InitStruct.Pin = LED_Pin;
GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
GPIO_InitStruct.Pull = GPIO_NOPULL;
GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
HAL_GPIO_Init(LED_GPIO_Port, &GPIO_InitStruct);
```

> [!IMPORTANT]
> - Enable GPIO port clock before calling `HAL_GPIO_Init()` using `__HAL_RCC_GPIOx_CLK_ENABLE()`
> - CubeMX automatically generates this in `MX_GPIO_Init()` function
> - Manual initialization requires explicit clock enabling

**Clock Enabling Examples:**
```c
// Enable clock for GPIO ports
__HAL_RCC_GPIOA_CLK_ENABLE();
__HAL_RCC_GPIOB_CLK_ENABLE();
__HAL_RCC_GPIOC_CLK_ENABLE();
```

---

#### `void HAL_GPIO_DeInit(GPIO_TypeDef *GPIOx, uint32_t GPIO_Pin)`

**Description:**  
De-initializes the specified GPIO pins, resetting them to their default state (analog mode with no pull resistors).

**Parameters:**
- `GPIOx` - GPIO Port (GPIOA, GPIOB, GPIOC, etc.)
- `GPIO_Pin` - GPIO pin(s) to be de-initialized (GPIO_PIN_x or combination)

**Returns:**  
None

**Default State After De-initialization:**
- Mode: Analog (high impedance)
- Pull: No pull-up/pull-down
- Speed: Low
- Output level: Undefined

**Examples:**

```c
// Example 1: De-initialize single pin
HAL_GPIO_DeInit(GPIOA, GPIO_PIN_5);

// Example 2: De-initialize multiple pins
HAL_GPIO_DeInit(GPIOB, GPIO_PIN_0 | GPIO_PIN_1 | GPIO_PIN_2);

// Example 3: De-initialize entire port
HAL_GPIO_DeInit(GPIOC, GPIO_PIN_All);

// Example 4: Using CubeMX-generated labels
HAL_GPIO_DeInit(LED_GPIO_Port, LED_Pin);
```

**Use Cases:**
- Releasing pins before entering low-power mode
- Reconfiguring pins for different functionality
- Disabling unused peripherals to save power
- Troubleshooting pin configuration issues

> [!NOTE]
> - De-initialization does NOT disable the GPIO port clock
> - After de-initialization, pins are in analog mode (lowest power consumption)
> - Re-initialization requires calling `HAL_GPIO_Init()` again

---

### Output Control

#### `void HAL_GPIO_WritePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin, GPIO_PinState PinState)`

**Description:**  
Sets or clears the selected GPIO pin(s).

**Parameters:**
- `GPIOx` - GPIO Port (GPIOA, GPIOB, GPIOC, etc.)
- `GPIO_Pin` - GPIO pin(s) to be written (GPIO_PIN_x or combination)
- `PinState` - Pin state to set
  - `GPIO_PIN_RESET` (0): Clear pin (logic LOW, 0V)
  - `GPIO_PIN_SET` (1): Set pin (logic HIGH, VDD)

**Returns:**  
None

**Execution Time:**  
Very fast (single register write operation)

**Examples:**

```c
// Example 1: Turn LED ON (Set pin HIGH)
HAL_GPIO_WritePin(GPIOB, GPIO_PIN_5, GPIO_PIN_SET);

// Example 2: Turn LED OFF (Set pin LOW)
HAL_GPIO_WritePin(GPIOB, GPIO_PIN_5, GPIO_PIN_RESET);

// Example 3: Using CubeMX-generated labels
HAL_GPIO_WritePin(LED_GPIO_Port, LED_Pin, GPIO_PIN_SET);

// Example 4: Control multiple pins simultaneously
// Turn on LEDs on PB0, PB1, PB2
HAL_GPIO_WritePin(GPIOB, GPIO_PIN_0 | GPIO_PIN_1 | GPIO_PIN_2, GPIO_PIN_SET);
```

**Complete Example:**
```c
#include "main.h"

int main(void) {
    HAL_Init();
    SystemClock_Config();
    MX_GPIO_Init();  // CubeMX-generated initialization
    
    while(1) {
        // LED ON
        HAL_GPIO_WritePin(LED_GPIO_Port, LED_Pin, GPIO_PIN_SET);
        HAL_Delay(1000);
        
        // LED OFF
        HAL_GPIO_WritePin(LED_GPIO_Port, LED_Pin, GPIO_PIN_RESET);
        HAL_Delay(1000);
    }
}
```

> [!TIP]
> - For controlling multiple pins on the same port, use bitwise OR: `GPIO_PIN_0 | GPIO_PIN_1`
> - This function is atomic and thread-safe (uses hardware bit set/reset registers)
> - When writing to multiple pins, all specified pins are set to the same state

---

#### `void HAL_GPIO_TogglePin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin)`

**Description:**  
Toggles the state of the specified GPIO pin(s). If the pin is HIGH, it becomes LOW, and vice versa.

**Parameters:**
- `GPIOx` - GPIO Port (GPIOA, GPIOB, GPIOC, etc.)
- `GPIO_Pin` - GPIO pin(s) to be toggled (GPIO_PIN_x or combination)

**Returns:**  
None

**Execution Time:**  
Very fast (read-modify-write operation)

**Examples:**

```c
// Example 1: Simple LED toggle
HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_5);

// Example 2: Using CubeMX-generated labels
HAL_GPIO_TogglePin(LED_GPIO_Port, LED_Pin);

// Example 3: LED blinking with toggle (cleaner code)
while(1) {
    HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_5);
    HAL_Delay(500);
}

// Example 4: Toggle multiple LEDs
while(1) {
    HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_0 | GPIO_PIN_1 | GPIO_PIN_2);
    HAL_Delay(200);  // Fast blinking effect
}
```

> [!TIP]
> - `TogglePin` is more efficient than reading, inverting, and writing manually
> - Perfect for LED blinking, debug signals, and square wave generation
> - When toggling multiple pins, all specified pins toggle simultaneously
> - The current pin state is automatically detected and inverted

---

### Input Reading

#### `GPIO_PinState HAL_GPIO_ReadPin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin)`

**Description:**  
Reads the state of the specified GPIO pin.

**Parameters:**
- `GPIOx` - GPIO Port (GPIOA, GPIOB, GPIOC, etc.)
- `GPIO_Pin` - GPIO pin to be read (single pin only)

**Returns:**
- `GPIO_PIN_RESET` (0): Pin is LOW (0V)
- `GPIO_PIN_SET` (1): Pin is HIGH (VDD)

**Return Type:**  
`GPIO_PinState` (enum with values 0 or 1)

**Execution Time:**  
Very fast (single register read operation)

**Examples:**

```c
// Example 1: Read button state
GPIO_PinState buttonState = HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_10);

if(buttonState == GPIO_PIN_RESET) {
    // Button pressed (assuming active low)
}

// Example 2: Using CubeMX-generated labels
GPIO_PinState buttonState = HAL_GPIO_ReadPin(KEY_GPIO_Port, KEY_Pin);

// Example 3: Control LED based on button
while(1) {
    if(HAL_GPIO_ReadPin(KEY_GPIO_Port, KEY_Pin) == GPIO_PIN_RESET) {
        HAL_GPIO_WritePin(LED_GPIO_Port, LED_Pin, GPIO_PIN_SET);
    } else {
        HAL_GPIO_WritePin(LED_GPIO_Port, LED_Pin, GPIO_PIN_RESET);
    }
}

// Example 4: Button press detection with debouncing
void checkButton(void) {
    static uint32_t lastDebounceTime = 0;
    static GPIO_PinState lastButtonState = GPIO_PIN_SET;
    static GPIO_PinState buttonState = GPIO_PIN_SET;
    
    GPIO_PinState reading = HAL_GPIO_ReadPin(KEY_GPIO_Port, KEY_Pin);
    
    if(reading != lastButtonState) {
        lastDebounceTime = HAL_GetTick();
    }
    
    if((HAL_GetTick() - lastDebounceTime) > 50) {
        if(reading != buttonState) {
            buttonState = reading;
            
            if(buttonState == GPIO_PIN_RESET) {
                // Button pressed action
                HAL_GPIO_TogglePin(LED_GPIO_Port, LED_Pin);
            }
        }
    }
    
    lastButtonState = reading;
}
```

> [!NOTE]
> - `ReadPin` can only read one pin at a time
> - To read multiple pins, call the function multiple times
> - For interrupt-based button handling, use EXTI (External Interrupt) configuration
> - Always implement debouncing for mechanical buttons

---

### Pin Protection

#### `HAL_StatusTypeDef HAL_GPIO_LockPin(GPIO_TypeDef *GPIOx, uint16_t GPIO_Pin)`

**Description:**  
Locks the configuration of the specified GPIO pin(s), preventing accidental reconfiguration until the next system reset.

**Parameters:**
- `GPIOx` - GPIO Port (GPIOA, GPIOB, GPIOC, etc.)
- `GPIO_Pin` - GPIO pin(s) to be locked (GPIO_PIN_x or combination)

**Returns:**
- `HAL_OK`: Lock operation successful
- `HAL_ERROR`: Lock operation failed

**Locking Mechanism:**
- Once locked, pin configuration cannot be changed
- Lock remains active until system reset or power cycle
- Prevents accidental `HAL_GPIO_Init()` calls from modifying configuration
- Does NOT prevent `WritePin`, `TogglePin`, or `ReadPin` operations

**What Gets Locked:**
- Pin mode (Input, Output, AF, Analog)
- Output type (Push-pull, Open-drain)
- Pull-up/Pull-down configuration
- Output speed
- Alternate function selection

**What Does NOT Get Locked:**
- Pin output state (can still write HIGH/LOW)
- Pin reading capability
- Interrupt configuration

**Examples:**

```c
// Example 1: Lock critical output pin
HAL_GPIO_LockPin(GPIOB, GPIO_PIN_5);

// Example 2: Lock with error checking
if(HAL_GPIO_LockPin(LED_GPIO_Port, LED_GPIO_Pin) == HAL_OK) {
    // Lock successful
} else {
    // Lock failed
}

// Example 3: Lock multiple pins
HAL_GPIO_LockPin(GPIOB, GPIO_PIN_0 | GPIO_PIN_1 | GPIO_PIN_2);
```

> [!IMPORTANT]
> - Lock is permanent until system reset - use with caution
> - Locking does NOT affect pin read/write operations
> - Lock only prevents configuration changes via `HAL_GPIO_Init()`
> - No way to unlock pins without resetting the microcontroller
> - Check return value to ensure lock was successful

> [!WARNING]
> - Do NOT lock pins that need runtime reconfiguration
> - Do NOT lock pins used by peripherals that reinitialize them
> - Debugging can be difficult if pins are locked unintentionally
> - Document all locked pins clearly in your code

---

## Function Summary Table

| Function | Purpose | Returns | Typical Use Case |
|----------|---------|---------|------------------|
| `HAL_GPIO_Init()` | Initialize GPIO pin(s) | None | Initial pin configuration |
| `HAL_GPIO_DeInit()` | Reset GPIO pin(s) to default | None | Release pins, low-power mode |
| `HAL_GPIO_WritePin()` | Set pin HIGH or LOW | None | Control LEDs, relays, outputs |
| `HAL_GPIO_TogglePin()` | Toggle pin state | None | LED blinking, debug signals |
| `HAL_GPIO_ReadPin()` | Read pin state | `GPIO_PinState` | Read buttons, sensors, inputs |
| `HAL_GPIO_LockPin()` | Lock pin configuration | `HAL_StatusTypeDef` | Protect critical pins |

---

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

**Circuit Example:**
```
MCU Pin (PP) ──┬── 330Ω ──┬── LED ── GND
               │          │
              VDD        GND
```

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

**Circuit Example:**
```
VDD ── 4.7kΩ ── MCU Pin (OD) ── Device
```

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