# Exercise 6

# 1️⃣ I2C Basics: Bus Scan / Device Communication Setup

## What this demo should show
This exercise introduces serial peripheral communication using I2C.

You will:
- configure I2C peripheral
- understand start, address, read/write concept
- communicate with an I2C device (sensor / EEPROM / mock device)
- observe data exchange through UART logs

## What to configure first in CubeMX

### Peripheral
- Enable **I2C1**
- Standard mode: **100 kHz**

### Pins
Typical default pins on STM32F446RE:
- **PB8** → I2C1_SCL
- **PB9** → I2C1_SDA

### Also keep UART enabled
- **USART2**
- 115200 baud

### Notes
If using an external I2C device:
- connect SDA
- connect SCL
- connect GND
- connect VCC
- ensure pull-up resistors are present if required


## Add UART helper function above main()

```c
void uart_print(char *msg)
{
    HAL_UART_Transmit(&huart2, (uint8_t*)msg, strlen(msg), HAL_MAX_DELAY);
}
````

## Add a simple I2C probe example in while(1)

> Example device address: `0x50 << 1` for EEPROM-style demo  
> Replace with your actual sensor address if needed.

```c
char msg[100];
HAL_StatusTypeDef status;

while (1)
{
    status = HAL_I2C_IsDeviceReady(&hi2c1, (0x50 << 1), 2, 100);

    if (status == HAL_OK)
    {
        uart_print("I2C device detected\r\n");
    }
    else
    {
        uart_print("I2C device not responding\r\n");
    }

    HAL_Delay(1000);
}
```

## Observation

* UART shows whether the I2C device responds
* Confirms bus and addressing are correct

📌 `I2C communication starts with correct addressing and device response.`

***

# 2️⃣ Read a Register from I2C Device

## Goal of this step

Perform a basic register read over I2C.

> This example assumes the device supports register-based access.  
> Replace device address and register address according to your sensor.

## Add this code inside while(1)

```c
char msg[100];
uint8_t reg_addr = 0x00;
uint8_t reg_data = 0;
HAL_StatusTypeDef status;

while (1)
{
    status = HAL_I2C_Mem_Read(&hi2c1,
                              (0x50 << 1),
                              reg_addr,
                              I2C_MEMADD_SIZE_8BIT,
                              &reg_data,
                              1,
                              100);

    if (status == HAL_OK)
    {
        sprintf(msg, "I2C Read OK: Reg[0x%02X] = 0x%02X\r\n", reg_addr, reg_data);
        uart_print(msg);
    }
    else
    {
        uart_print("I2C Read Failed\r\n");
    }

    HAL_Delay(1000);
}
```

## Observation

* UART displays register value if communication is correct

📌 `I2C is commonly used for register-based sensor communication.`

***

# 3️⃣ Write Then Read Back (I2C)

## Goal of this step

Understand write + verification sequence.

> Use this only if your test device supports writable registers.

## Replace while(1) loop with this

```c
char msg[100];
uint8_t reg_addr = 0x01;
uint8_t write_data = 0x25;
uint8_t read_data = 0;
HAL_StatusTypeDef status;

while (1)
{
    status = HAL_I2C_Mem_Write(&hi2c1,
                               (0x50 << 1),
                               reg_addr,
                               I2C_MEMADD_SIZE_8BIT,
                               &write_data,
                               1,
                               100);

    if (status == HAL_OK)
    {
        uart_print("I2C Write OK\r\n");
    }
    else
    {
        uart_print("I2C Write Failed\r\n");
    }

    status = HAL_I2C_Mem_Read(&hi2c1,
                              (0x50 << 1),
                              reg_addr,
                              I2C_MEMADD_SIZE_8BIT,
                              &read_data,
                              1,
                              100);

    if (status == HAL_OK)
    {
        sprintf(msg, "Read Back = 0x%02X\r\n", read_data);
        uart_print(msg);
    }
    else
    {
        uart_print("I2C Read Back Failed\r\n");
    }

    HAL_Delay(1000);
}
```

## Observation

* Read-back confirms whether write operation succeeded

📌 `Read-back verification is a common embedded debugging practice.`

***

# 4️⃣ Basic SPI Communication Demo

## What this demo should show

This step introduces SPI communication with a simple transmit/receive example.

You will:

* configure SPI peripheral
* send data
* receive response
* observe communication through UART logs

***

## What to configure first in CubeMX

### Peripheral

* Enable **SPI1**
* Mode: **Full Duplex Master**

### Typical pins

* **PA5** → SPI1\_SCK
* **PA6** → SPI1\_MISO
* **PA7** → SPI1\_MOSI

### Optional chip select

Use one GPIO pin as manual chip select:

* Example: **PB6** → CS output


## Add SPI transaction code inside while(1)

```c
char msg[100];
uint8_t tx_data[2] = {0xA5, 0x5A};
uint8_t rx_data[2] = {0};
HAL_StatusTypeDef status;

while (1)
{
    HAL_GPIO_WritePin(GPIOB, GPIO_PIN_6, GPIO_PIN_RESET);   // CS LOW

    status = HAL_SPI_TransmitReceive(&hspi1, tx_data, rx_data, 2, 100);

    HAL_GPIO_WritePin(GPIOB, GPIO_PIN_6, GPIO_PIN_SET);     // CS HIGH

    if (status == HAL_OK)
    {
        sprintf(msg, "SPI TX=[0x%02X 0x%02X], RX=[0x%02X 0x%02X]\r\n",
                tx_data[0], tx_data[1], rx_data[0], rx_data[1]);
        uart_print(msg);
    }
    else
    {
        uart_print("SPI Transfer Failed\r\n");
    }

    HAL_Delay(1000);
}
```

## Observation

* UART shows transmitted and received bytes
* Demonstrates synchronous data exchange

📌 `SPI exchanges data in full duplex mode during clocked transfer.`

***

# 5️⃣ Compare I2C and SPI Communication

## Goal of this step

Understand protocol differences at system level.

## Summary

### I2C

* 2 wires
* address-based
* multiple devices on same bus
* lower pin count
* simpler sensor integration

### SPI

* faster
* full duplex
* no address field
* requires chip select
* typically more pins

📌 `Choice of protocol depends on device type, speed, and system constraints.`

***

# 6️⃣ Use UART for Protocol Debugging

## Goal of this step

Use UART as a visibility channel for protocol experiments.

## Add structured logs

### For I2C

```c
sprintf(msg, "I2C Reg[0x%02X] = 0x%02X\r\n", reg_addr, reg_data);
uart_print(msg);
```

### For SPI

```c
sprintf(msg, "SPI RX Byte0 = 0x%02X\r\n", rx_data[0]);
uart_print(msg);
```

## Observation

* UART logs help diagnose communication correctness

📌 `Protocol debugging often depends on careful logging and visibility.`

***

# 7️⃣ Fault Injection – Wrong Device Address / Wrong Wiring

## Goal of this step

Introduce communication debugging mindset.

## I2C fault examples

* wrong device address
* SDA / SCL not connected correctly
* missing pull-ups
* wrong voltage supply

## SPI fault examples

* chip select not toggled
* MISO/MOSI swapped
* wrong clock polarity / phase
* slave device not powered

## Observation

Symptoms may include:

* no response
* timeout
* all 0xFF / all 0x00 data
* repeated transfer failure

📌 `Peripheral debugging requires checking both software configuration and hardware signals.`

***

# 8️⃣ Mini Protocol-to-Data-Flow View

## Goal of this step

Connect communication with embedded data flow thinking.

### Example flow

```text
I2C Sensor Read → Process Data → UART Log
```

or

```text
SPI Transfer → Interpret Response → Display / Log Result
```

## Why this matters

Protocols are not isolated topics. In real systems they are part of the full chain:

```text
Read → Interpret → Act / Log
```

📌 `Peripheral communication is one stage in a larger embedded system pipeline.`

***

# 9️⃣ Final Exercise Summary

## What this exercise covered

* I2C bus setup and device probing
* I2C register read/write sequence
* SPI transfer basics
* UART-assisted protocol debugging
* Protocol comparison (I2C vs SPI)
* Communication fault diagnosis
* Viewing communication as part of a data flow pipeline

***

## Key Learning

* I2C and SPI are fundamental embedded communication protocols
* Correct peripheral configuration is essential
* UART is useful for observing protocol behavior
* Read-back verification helps validate transactions
* Communication is typically just one stage in a larger embedded system flow
* Hardware debugging is as important as software debugging

***