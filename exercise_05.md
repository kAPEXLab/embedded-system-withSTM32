# Exercise 5

# 1️⃣ Basic ADC Reading

## What this demo should show
This exercise introduces analog input reading using STM32 ADC.

You will:
- configure ADC input
- read raw analog values
- observe the digital output of the ADC

## What to configure first in CubeMX

### Peripheral
- Enable **ADC1**
- Select analog input pin  
  - Example: **PA0 / ADC1_IN0**

### ADC Configuration
- Resolution: **12-bit**
- Data Alignment: **Right**
- Scan Conversion Mode: **Disable**
- Continuous Conversion Mode: **Disable**
- External Trigger: **Software Start**
- Number of Conversions: **1**

### Channel Configuration
- Rank: **1**
- Sampling Time: **56 cycles** (or similar suitable value)

### Notes
- Connect potentiometer:
  - One end → **3.3V**
  - Other end → **GND**
  - Wiper → **PA0**

---

## Add this helper function above main()

```c
uint32_t adc_read(void)
{
    HAL_ADC_Start(&hadc1);
    HAL_ADC_PollForConversion(&hadc1, 10);

    uint32_t value = HAL_ADC_GetValue(&hadc1);

    HAL_ADC_Stop(&hadc1);
    return value;
}
````

## Replace the while(1) loop with this

```c
while (1)
{
    uint32_t adc_value = adc_read();
    HAL_Delay(500);
}
```

## Observation

* Program runs and continuously reads ADC
* Raw ADC value is not yet visible externally

📌 `ADC converts analog voltage into a digital value.`

***

# 2️⃣ Print ADC Value using UART

## Goal of this step

Make ADC readings visible on terminal.

## Add UART helper function

```c
void uart_print(char *msg)
{
    HAL_UART_Transmit(&huart2, (uint8_t*)msg, strlen(msg), HAL_MAX_DELAY);
}
```

## Replace the while(1) loop with this

```c
char msg[100];

while (1)
{
    uint32_t adc_value = adc_read();

    sprintf(msg, "ADC Raw Value = %lu\r\n", adc_value);
    uart_print(msg);

    HAL_Delay(500);
}
```

## Observation

* Terminal displays ADC raw count values
* Rotating potentiometer changes the values

Typical range:

* near **0** at 0V
* near **4095** at 3.3V

📌 `UART makes invisible sensor values visible for debugging and validation.`

***

# 3️⃣ Convert ADC Counts to Voltage

## Goal of this step

Convert raw ADC counts into a meaningful engineering value.

## Add conversion logic

For 12-bit ADC and 3.3V reference:

```c
uint32_t voltage_mv = (adc_value * 3300UL) / 4095UL;
```

## Replace the while(1) loop with this

```c
char msg[100];

while (1)
{
    uint32_t adc_value = adc_read();
    uint32_t voltage_mv = (adc_value * 3300UL) / 4095UL;

    sprintf(msg, "ADC = %lu, Voltage = %lu mV\r\n", adc_value, voltage_mv);
    uart_print(msg);

    HAL_Delay(500);
}
```

## Observation

* Terminal now shows:
  * raw ADC count
  * approximate voltage in mV

Example:

```text
ADC = 2048, Voltage = 1650 mV
```

📌 `Raw sensor data often needs conversion before it becomes meaningful.`

***

# 4️⃣ Add LED Control Based on ADC Threshold

## Goal of this step

Use sensor input to control output behavior.

## Replace the while(1) loop with this

```c
char msg[100];

while (1)
{
    uint32_t adc_value = adc_read();
    uint32_t voltage_mv = (adc_value * 3300UL) / 4095UL;

    if (voltage_mv > 2000)
    {
        HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
    }
    else
    {
        HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
    }

    sprintf(msg, "ADC = %lu, Voltage = %lu mV\r\n", adc_value, voltage_mv);
    uart_print(msg);

    HAL_Delay(500);
}
```

## Observation

* LED turns ON when voltage crosses threshold
* LED turns OFF when voltage is below threshold

📌 `Sensor input can drive physical control decisions.`

***

# 5️⃣ Classify ADC Value into LOW / MID / HIGH

## Goal of this step

Introduce simple processing before control/logging.

## Replace the while(1) loop with this

```c
char msg[100];

while (1)
{
    uint32_t adc_value = adc_read();
    uint32_t voltage_mv = (adc_value * 3300UL) / 4095UL;

    if (voltage_mv < 1000)
    {
        sprintf(msg, "LOW  : ADC = %lu, Voltage = %lu mV\r\n", adc_value, voltage_mv);
    }
    else if (voltage_mv < 2000)
    {
        sprintf(msg, "MID  : ADC = %lu, Voltage = %lu mV\r\n", adc_value, voltage_mv);
    }
    else
    {
        sprintf(msg, "HIGH : ADC = %lu, Voltage = %lu mV\r\n", adc_value, voltage_mv);
    }

    uart_print(msg);

    HAL_Delay(500);
}
```

## Observation

* Sensor value is now interpreted as a state
* Output becomes easier to understand

Example:

```text
LOW  : ADC = 520, Voltage = 419 mV
MID  : ADC = 1720, Voltage = 1386 mV
HIGH : ADC = 3250, Voltage = 2619 mV
```

📌 `Processing turns raw data into useful system information.`

***

# 6️⃣ Build a Mini Data Pipeline: Read → Process → Control / Logging

## Goal of this step

Create a simple embedded system pipeline using bare-metal logic.

## Replace the while(1) loop with this

```c
char msg[120];

while (1)
{
    /* Read */
    uint32_t adc_value = adc_read();

    /* Process */
    uint32_t voltage_mv = (adc_value * 3300UL) / 4095UL;
    char *state = "LOW";

    if (voltage_mv < 1000)
    {
        state = "LOW";
        HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET);
    }
    else if (voltage_mv < 2000)
    {
        state = "MID";
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
    }
    else
    {
        state = "HIGH";
        HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);
    }

    /* Control / Logging */
    sprintf(msg, "State = %s, ADC = %lu, Voltage = %lu mV\r\n", state, adc_value, voltage_mv);
    uart_print(msg);

    HAL_Delay(500);
}
```

## Observation

This now forms a basic embedded system pipeline:

```text
Read ADC → Convert/Interpret → Actuate LED + Log via UART
```

Examples:

* LOW → LED OFF
* MID → LED toggles
* HIGH → LED ON

📌 `This is a simple but complete sense → process → act / log system.`

***

# 7️⃣ Introduce Fault Injection – Wrong ADC Pin / Wrong Channel

## Goal of this step

Develop sensor debugging mindset.

## Possible fault scenarios

* Wrong ADC channel configured
* Potentiometer not connected properly
* Input pin not configured as analog
* Missing GND reference

## Observation

Symptoms may include:

* ADC always reads 0
* ADC always reads near max
* ADC values remain constant
* Values appear noisy or random

## What to check

1. ADC channel selection
2. Pin mode = analog
3. Potentiometer wiring
4. Common ground connection
5. `MX_ADC1_Init()` called correctly

📌 `Sensor debugging must include both software and hardware checks.`

***

# 8️⃣ Debug ADC Reading with Breakpoints

## Goal of this step

Verify sensor acquisition step-by-step.

## What to do

* Place breakpoint inside:

```c
adc_read()
```

* Step through:
  * `HAL_ADC_Start()`
  * `HAL_ADC_PollForConversion()`
  * `HAL_ADC_GetValue()`

## Observation

* You can confirm conversion is being started
* You can inspect raw ADC value directly in debugger

📌 `Debugger helps verify acquisition path even when UART output is unclear.`

***

# 9️⃣ Timing Dependence of Sensor Reading

## Goal of this step

Understand how sampling interval affects behavior.

## Modify delay

```c
HAL_Delay(100);
```

then try

```c
HAL_Delay(1000);
```

## Observation

* Faster delay → more frequent updates
* Slower delay → less responsive reading

📌 `Sensor update rate is a design decision.`

***

# 🔟 Final Exercise Summary

## What this exercise covered

* ADC peripheral configuration
* Raw analog signal reading
* Conversion from ADC count to voltage
* Threshold-based control
* Data classification (LOW / MID / HIGH)
* Mini pipeline: Read → Process → Control / Logging
* ADC fault diagnosis
* Debugging sensor acquisition

***

## Key Learning

* ADC is the bridge between analog world and digital software
* Raw sensor values must be interpreted before use
* Embedded systems often follow a pipeline:
  * sense
  * process
  * act / log
* Timing and sampling interval matter
* Sensor debugging requires both hardware and software validation

***