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
