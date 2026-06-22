# Exercise 2

# 1️⃣ Button Polling: Control LED

## What this demo should show
This exercise introduces input handling using GPIO.

- Read button state
- Control LED based on input
- Understand polling-based design



## What to configure first in CubeMX

### Pins
- **PA5** → LED output
- **PC13** → Button input (GPIO Input mode)


## Replace the while(1) loop with this

```c
while (1)
{
    if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == GPIO_PIN_SET)
    {
        HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
    }
    else
    {
        HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
    }
}
````

## Observation

* LED turns ON when button is pressed
* LED turns OFF when button is released

📌 `CPU continuously checks the button state (polling)`

----

# 2️⃣ Add UART Logging to Polling Loop

## Goal of this step

Observe system behavior through UART logs

## Add helper function

```c
void uart_print(char *msg)
{
    HAL_UART_Transmit(&huart2, (uint8_t*)msg, strlen(msg), HAL_MAX_DELAY);
}
```

## Modify while loop

```c
while (1)
{
    if (HAL_GPIO_ReadPin(GPIOC, GPIO_PIN_13) == GPIO_PIN_SET)
    {
        uart_print("Button Pressed\r\n");
        HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
    }
    else
    {
        HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
    }

    HAL_Delay(200);
}
```

## Observation

* Serial output shows button events
* LED behavior remains same

📌 `Polling + logging increases CPU load`

***

# 3️⃣ Limitation of Polling (Important Step)

## Goal of this step

Expose timing limitation of polling

## Modify delay

```c
HAL_Delay(1000);
```

## Observation

* Button response becomes slower
* Quick presses may be missed

## What this demonstrates

```text
Polling interval = responsiveness
```

📌 `Polling depends on loop timing`
📌 `Long delays → missed events`

***

# 4️⃣ Convert to Interrupt-Based Input (EXTI)

## Goal of this step

Move from polling → event-driven design

***

## Update CubeMX Configuration

### Change PC13 mode:

* From GPIO Input → **EXTI (External Interrupt)**

### Enable interrupt:

* EXTI line (PC13)
* Configure trigger (Rising/Falling based on board)

### Enable NVIC:

* EXTI15\_10\_IRQn

***

## Add ISR Callback

```c
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
    if (GPIO_Pin == GPIO_PIN_13)
    {
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
    }
}
```

***

## Observation

* LED toggles only when button is pressed
* No continuous polling required

📌 `System becomes event-driven`

***

# 5️⃣ Add Event Counter in ISR

## Goal of this step

Track number of button events

## Add global variable

```c
volatile uint32_t g_button_count = 0;
```

## Modify ISR

```c
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
    if (GPIO_Pin == GPIO_PIN_13)
    {
        g_button_count++;
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
    }
}
```

***

## Observation

* Each button press increments counter

📌 `ISR should remain lightweight`

***

# 6️⃣ Print Event Count Using UART

## Goal

Combine ISR + communication

## Add to main loop

```c
char msg[50];

while (1)
{
    sprintf(msg, "Button Count = %lu\r\n", g_button_count);
    uart_print(msg);

    HAL_Delay(1000);
}
```

***

## Observation

* Button count is printed every 1 second
* ISR handles event, main loop handles logging

📌 `Separation of responsibilities`

***

# 7️⃣ Debugging Interrupt Execution

## Goal

Understand interrupt execution flow

## Steps

* Place breakpoint in:

```c
HAL_GPIO_EXTI_Callback()
```

## Observation

* Code execution jumps to ISR on button press
* ISR interrupts main loop execution

📌 `Interrupt preempts main execution`

***

# 8️⃣ Common Mistake (Fault Injection)

## Introduce bug

```c
if(GPIO_Pin = GPIO_PIN_13)
```

## Observation

* Incorrect behavior
* ISR triggers unexpectedly

## Fix

```c
if(GPIO_Pin == GPIO_PIN_13)
```

📌 `Assignment vs comparison bug`

***

# 9️⃣ Polling vs Interrupt Comparison

## Polling

```text
Loop checks continuously
↓
CPU usage high
↓
Events may be missed
```

## Interrupt

```text
Event triggers ISR
↓
Immediate response
↓
Efficient CPU usage
```

***

# 🔟 Final Exercise Summary

## What this exercise covered

* Button input using polling
* Limitation of polling
* Event-driven design using EXTI
* ISR implementation
* Debugging interrupt flow
* Integration with UART logging

***

## Key Learning

* Polling is simple but inefficient
* Interrupts provide immediate response
* ISR must be short and efficient
* Debugging interrupt systems is essential
* Embedded systems often use event-driven design

***