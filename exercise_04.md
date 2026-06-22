# Exercise 4

# 1️⃣ Basic UART Transmission

## What this demo should show
This exercise introduces UART-based serial communication using STM32.

You will:
- configure UART
- transmit text to a serial terminal
- verify communication between STM32 and PC


## What to configure first in CubeMX

### Peripheral
- Enable **USART2**
- Mode: **Asynchronous**
- Baud rate: **115200**
- Word length: **8 Bits**
- Stop bits: **1**
- Parity: **None**
- Hardware flow control: **None**

### Note
Use the board virtual COM port through USB connection.


## Add this helper function above main()

```c
void uart_print(char *msg)
{
    HAL_UART_Transmit(&huart2, (uint8_t*)msg, strlen(msg), HAL_MAX_DELAY);
}
````

## Replace the while(1) loop with this

```c
while (1)
{
    uart_print("UART Hello from STM32\r\n");
    HAL_Delay(1000);
}
```

## Observation

* Terminal should display the message every 1 second

📌 `UART is one of the most useful debugging and monitoring tools in embedded systems.`

***

# 2️⃣ Transmit a Counter Value

## Goal of this step

Move from fixed string output to dynamic formatted output.

## Replace the while(1) loop with this

```c
char msg[100];
uint32_t count = 0;

while (1)
{
    sprintf(msg, "Message Count = %lu\r\n", count++);
    uart_print(msg);

    HAL_Delay(1000);
}
```

## Observation

* Terminal displays incrementing counter values
* Data changes every second

📌 `Formatted UART messages are useful for observing runtime behavior.`

***

# 3️⃣ Add LED + UART Together

## Goal of this step

Demonstrate simultaneous GPIO action and serial logging.

## Replace the while(1) loop with this

```c
char msg[100];
uint32_t count = 0;

while (1)
{
    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);

    sprintf(msg, "LED toggled, count = %lu\r\n", count++);
    uart_print(msg);

    HAL_Delay(500);
}
```

## Observation

* LED blinks
* UART logs each blink event

📌 `Embedded systems often combine physical output with communication output.`

***

# 4️⃣ Button-Controlled UART Message

## Goal of this step

Connect input handling with communication.

## Replace the while(1) loop with this

```c
while (1)
{
    if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == GPIO_PIN_SET)
    {
        uart_print("Button Pressed\r\n");
    }

    HAL_Delay(200);
}
```

## Observation

* Terminal shows message when button is pressed

## Important note

Depending on board wiring, button logic may be inverted.  
If needed, use:

```c
if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == GPIO_PIN_RESET)
```

📌 `UART helps visualize system events that are otherwise not physically visible.`

***

# 5️⃣ Transmit Multi-Line Status Block

## Goal of this step

Show how UART can be used for structured status reporting.

## Replace the while(1) loop with this

```c
char msg[100];
uint32_t count = 0;

while (1)
{
    sprintf(msg, "\r\n--- STATUS BLOCK START ---\r\n");
    uart_print(msg);

    sprintf(msg, "Loop Count = %lu\r\n", count++);
    uart_print(msg);

    sprintf(msg, "LED State Toggled\r\n");
    uart_print(msg);

    sprintf(msg, "--- STATUS BLOCK END ---\r\n");
    uart_print(msg);

    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
    HAL_Delay(1000);
}
```

## Observation

* Messages appear in grouped block format
* Easier to read in terminal

📌 `Well-structured logs are easier to debug than isolated messages.`

***

# 6️⃣ Introduce Delay Impact on Communication

## Goal of this step

Understand that UART output timing is controlled by application timing.

## Modify delay

```c
HAL_Delay(2000);
```

## Observation

* UART updates become slower
* LED also changes more slowly

📌 `Communication rate is determined by software timing and scheduling.`

***

# 7️⃣ Fault Injection – Wrong UART Handle or Missing Init

## Goal of this step

Introduce communication debugging mindset.

## Example issue

If `uart_print()` is called but nothing appears on terminal, possible reasons include:

* USART not enabled in CubeMX
* wrong COM port selected on PC
* wrong baud rate in terminal
* wrong UART handle used
* board not connected correctly

## Suggested verification steps

1. Check `MX_USART2_UART_Init()` is called
2. Verify terminal baud rate = 115200
3. Check board COM port
4. Confirm `huart2` is valid
5. Rebuild and reflash program

📌 `When debugging UART issues, always check both MCU configuration and terminal settings.`

***

# 8️⃣ Observe UART Transmission in Debugger

## Goal of this step

Use debugger to inspect message transmission flow.

## What to do

* Place breakpoint inside:

```c
uart_print()
```

* Run in debug mode
* Step over `HAL_UART_Transmit(...)`

## Observation

* You can verify that code is actually reaching UART transmit function
* Helps separate software issue from terminal/hardware issue

📌 `Debugger confirms whether transmission is being attempted by software.`

***

# 9️⃣ Polling-Based UART Discussion

## What this step demonstrates

In this exercise, UART transmission uses:

```c
HAL_UART_Transmit(..., HAL_MAX_DELAY)
```

This is:

* simple
* blocking
* easy to understand

But it also means:

* CPU waits until transmission completes
* not ideal for high-performance systems

📌 `Polling mode is useful for learning and debugging, but scalable systems often use interrupt or DMA-based communication.`

***

# 🔟 Final Exercise Summary

## What this exercise covered

* UART peripheral configuration
* Basic serial text transmission
* Dynamic message formatting
* Combining GPIO and UART
* Button-triggered serial event logging
* Structured multi-line status output
* Communication timing dependency
* Basic UART debugging techniques

***

## Key Learning

* UART provides a basic communication channel between STM32 and PC
* Serial logs are extremely useful for debugging embedded systems
* Communication timing depends on software structure
* Polling-based UART is simple but blocks CPU
* Embedded systems often use UART for runtime observability

***