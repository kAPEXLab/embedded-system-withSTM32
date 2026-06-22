# STM32 Foundations & Peripheral Interfacing Exercises

This repository contains a structured set of hands-on exercises designed for the Program on **STM32 Microcontrollers**.

The exercises focus on building a strong foundation in:

- STM32CubeIDE and CubeMX workflow
- GPIO and low-level understanding
- Interrupts and event handling
- Debugging fundamentals
- UART communication
- ADC-based sensor interfacing
- I2C and SPI communication basics
- Data flow thinking in embedded systems

The overall goal is to help participants move from **basic hardware bring-up** to **simple embedded data pipelines** using STM32.

---

## Learning Progression

The exercises are organized as a step-by-step progression:

1. STM32 project setup and GPIO basics  
2. Button handling using polling and interrupts  
3. Debugging, timing observation, and fault injection  
4. UART communication and serial logging  
5. ADC interfacing and simple data processing  
6. I2C / SPI communication and peripheral data flow  

Each exercise is designed as a **single README with multiple steps**, so that participants can build concepts incrementally.

---

## Repository Structure

| Folder | Description |
|--------|------------|
| [01_stm32_project_setup_gpio](./exercise_01.md/) | STM32CubeIDE setup, GPIO output, HAL vs register access |
| [02_button_polling_interrupts](./exercise_02.md/) | Button interfacing using polling and EXTI interrupts |
| [03_debugging_timing_fault_injection](./exercise_03.md/) | Breakpoints, variable watch, timing analysis, fault injection |
| [04_uart_communication](./exercise_04.md/) | UART transmission, formatted logging, button-triggered serial output |
| [05_adc_data_flow](./exercise_05.md/) | ADC reading, voltage conversion, threshold logic, read → process → log |
| [06_i2c_spi_protocols](./exercise_06.md/) | I2C and SPI basics, protocol verification, communication as system data flow |

---

## Key Concepts Covered

- STM32CubeIDE project creation
- CubeMX peripheral configuration
- GPIO control using HAL and registers
- Polling vs interrupt-based event handling
- Breakpoints and variable watch
- Timing observation using `HAL_GetTick()`
- Fault injection and debugging mindset
- UART-based runtime visibility
- ADC acquisition and analog signal interpretation
- Threshold-based decision logic
- I2C and SPI communication fundamentals
- Embedded data flow thinking: **Read → Process → Control / Logging**

---

## Hardware Platform

These exercises are designed for:

- **STM32F446RE Nucleo Board**
- Onboard LED (**LD2 / PA5**)
- User Button (**B1 / PC13**)
- UART via **USART2** and onboard virtual COM port
- ADC input via **PA0** (potentiometer recommended)
- External I2C / SPI sensor or test device (optional for protocol exercises)

---

## Software Environment

- **STM32CubeIDE**
- **STM32CubeMX**
- **STM32 HAL drivers**
- Serial terminal tool such as:
  - TeraTerm
  - PuTTY
  - minicom

---

## Design Philosophy

These exercises are designed with the following principles:

- Build confidence through **small, observable steps**
- Emphasize **both hardware understanding and software discipline**
- Use UART logs and debugger to make system behavior visible
- Focus on **real engineering reasoning**, not just API usage
- Progress naturally from:
  - hardware bring-up
  - peripheral interfacing
  - data interpretation
  - system-level thinking

---

## How to Use This Repository

1. Follow the exercises in sequence from `01` to `06`
2. Open each folder and read the corresponding `README.md`
3. Complete all steps in order within each exercise
4. Build, flash, and observe behavior on hardware
5. Use debugger and UART output wherever suggested
6. Compare behavior before and after each modification

---

## Expected Learning Outcome

By completing these six exercises, participants should be able to:

- Create and configure STM32 projects independently
- Control GPIO pins using both HAL and register-level access
- Design and debug polling and interrupt-based input handling
- Use USART for runtime logging and debugging
- Acquire and interpret analog sensor data
- Build simple embedded data processing pipelines
- Perform basic I2C and SPI communication
- Develop stronger confidence in embedded bring-up and debugging

---