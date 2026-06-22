# Exercise 3

# 1️⃣ Basic Debugging with Breakpoints

## What this demo should show
This exercise introduces debugging fundamentals using STM32CubeIDE.

You will:
- Pause execution using breakpoints
- Step through code
- Observe program flow

## Replace the while(1) loop

```c
while (1)
{
    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
    HAL_Delay(500);
}
````

## Debugging Steps

1. Start Debug Mode
2. Place breakpoint on:

```c
HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
```

3. Run code


## Observation

* Execution stops at breakpoint
* Each step continues execution line-by-line

📌 `You can now control program execution manually`

***

# 2️⃣ Step Execution (Step Into / Step Over)

## Goal of this step

Understand how code executes line-by-line.

***

## Perform

* Use **Step Over** → executes function
* Use **Step Into** → goes inside function
* Use **Resume** → continues execution

## Observation

* You see how HAL functions are called
* Execution is no longer a “black box”

📌 `Debugging reveals what CPU is actually doing`

***

# 3️⃣ Variable Watch & Observation

## Goal of this step

Track runtime values using debugger


## Add a variable

```c
uint32_t counter = 0;
```

## Modify loop

```c
while (1)
{
    counter++;

    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
    HAL_Delay(500);
}
```

## Debug Steps

1. Add `counter` to Watch window
2. Run in debug mode


## Observation

* Counter keeps increasing
* You can observe variables in real time

📌 `You can observe internal system state without printing`

***

# 4️⃣ Measure Timing in Code

## Goal of this step

Understand execution timing behavior


## Modify code

```c
uint32_t counter = 0;
uint32_t prev_tick = 0;
uint32_t curr_tick = 0;
uint32_t delta = 0;

while (1)
{
    curr_tick = HAL_GetTick();

    delta = curr_tick - prev_tick;

    prev_tick = curr_tick;

    counter++;

    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
    HAL_Delay(500);
}
```

## Observation

* `delta` value \~500 ms
* Timing is measurable

📌 `Timing should be measured, not assumed`

***

# 5️⃣ Introduce Processing Load (Important Step)

## Goal of this step

Observe how additional load affects timing

## Add this function above main()

```c
void simulate_load(void)
{
    for (volatile uint32_t i = 0; i < 1000000; i++)
    {
    }
}
```

## Modify loop

```c
while (1)
{
    curr_tick = HAL_GetTick();
    delta = curr_tick - prev_tick;
    prev_tick = curr_tick;

    simulate_load();

    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
    HAL_Delay(500);
}
```

## Observation

* `delta` increases
* LED timing slightly shifts

📌 `Execution time affects system timing`

***

# 6️⃣ Fault Injection – Wrong Pin

## Goal of this step

Understand real debugging process

## Introduce bug

```c
HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_6);
```

## Observation

* Code runs correctly (no errors)
* LED does not blink

## What to check

* Correct GPIO pin
* Hardware mapping
* CubeMX configuration

📌 `Code can be logically correct but still fail on hardware`

***

# 7️⃣ Fault Injection – Wrong Delay

## Modify

```c
HAL_Delay(2000);
```

## Observation

* LED blinks slowly
* System appears “unresponsive”

📌 `Timing affects perceived system behavior`

***

# 8️⃣ Add Debug Logging (Compare with Debugger)

## Add helper function

```c
void uart_print(char *msg)
{
    HAL_UART_Transmit(&huart2, (uint8_t*)msg, strlen(msg), HAL_MAX_DELAY);
}
```

## Modify loop

```c
char msg[100];

while (1)
{
    sprintf(msg, "delta = %lu ms\r\n", delta);
    uart_print(msg);

    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
    HAL_Delay(500);
}
```

## Observation

* Data visible in terminal
* Compare with debugger values

📌 `Debugger → internal view`
📌 `UART → external observation`

***

# 9️⃣ Understand Timing Dependency

## Combined behavior

```text
Execution time = code + processing + delay
```

***

## Observation

* Delay alone does not define timing
* Actual loop timing depends on execution

📌 `Timing = delay + execution overhead`

***

# 🔟 Final Exercise Summary

## What this exercise covered

* Breakpoints and stepping
* Variable watch
* Observing execution flow
* Measuring timing using HAL\_GetTick()
* Effect of processing load
* Fault injection and debugging
* UART logging vs debugger observation

***

## Key Learning

* Debugging is essential in embedded systems
* Execution must be verified, not assumed
* Timing depends on total system behavior
* Faults can come from configuration, not just code
* Observability is critical for system validation

***