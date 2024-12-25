# TIM6, UART2, and SLEEPONEXIT Feature Demonstration

This project demonstrates the usage of TIM6, UART2, and the SLEEPONEXIT feature on an STM32 microcontroller. It involves configuring TIM6 to generate an update interrupt every 10ms, sending data over UART2 in the interrupt service routine (ISR), and measuring current consumption in two modes: without sleep mode and with the SLEEPONEXIT feature enabled. Additionally, a GPIO pin (PA12) is toggled to verify system behavior using a logic analyzer.

---

## Key Features

### TIM6 Configuration
- **Purpose:** Triggers an update interrupt every 10ms.
- **Usage:** The ISR of TIM6 is responsible for sending data via UART2.

### UART2 Configuration
- **Purpose:** Transmits data asynchronously.
- **Data:** The message `"testing SLEEPONEXIT feature\r\n"` is sent every 10ms.

### SLEEPONEXIT Feature
- **Purpose:** Reduces current consumption by entering sleep mode automatically after ISR execution.
- **Implementation:** Enabled using the `HAL_PWR_EnableSleepOnExit()` API. The system wakes only when an interrupt occurs.

---

## Project Workflow

### 1. Initialization
- **System Clock:** Configured to support TIM6 and UART2 operation.
- **Peripheral Initialization:**
  - GPIO: Configured for PA12 for debugging with the logic analyzer.
  - TIM6: Configured to generate interrupts every 10ms.
  - UART2: Configured for asynchronous communication.

### 2. Main Function
- **HAL Initialization:** Initializes HAL and resets peripherals.
- **Enable SLEEPONEXIT:** Configured using `HAL_PWR_EnableSleepOnExit()`.
- **Timer Start:** Starts TIM6 in interrupt mode using `HAL_TIM_Base_Start_IT()`.
- **Infinite Loop:** Maintains system in an idle state, relying on interrupts for operation.

### 3. Interrupt Handlers
#### TIM6 Update Interrupt
- **Function:** `HAL_TIM_PeriodElapsedCallback`.
- **Action:** Sends the predefined string over UART2 using `HAL_UART_Transmit_IT()`.
#### UART Transmission Complete Callback
- **Function:** `HAL_UART_TxCpltCallback`.
- **Action:** Toggles PA12 to create a visible signal for the logic analyzer.

---

## Results and Observations

### Current Consumption
- **Without Sleep Mode:** Higher current consumption due to the CPU remaining active in the infinite loop.
- **With Sleep Mode (SLEEPONEXIT):** Significantly reduced current consumption. The CPU sleeps after ISR execution, only waking upon interrupts.

### Logic Analyzer Observations
- **PA12 Pulse:** Verified UART transmission timing and sleep mode behavior. The pin toggles briefly after each UART transmission completes.

---

## Additional Exercise: UART Communication on Button Interrupt

### Description
This exercise involves sending data over UART2 whenever a button interrupt is received. Current consumption is measured in two modes:
1. **Without Sleep Mode:** The CPU remains active in the infinite loop.
2. **With Sleep Mode (using WFI instruction):** The CPU enters sleep mode, waking only on interrupt.

### Implementation

#### GPIO and UART Configuration
- **Button GPIO Pin:** Configured as an external interrupt source.
- **UART2:** Configured for asynchronous communication to send messages when the button is pressed.

#### Main Function Overview
1. **System Initialization:** Initializes HAL, system clock, GPIOs, and UART2.
2. **Infinite Loop:** Enters sleep mode using the `__WFI()` instruction.
   - Wakes up when the button interrupt is triggered.

#### Interrupt Handler
##### Button Interrupt Callback
- **Function:** `HAL_GPIO_EXTI_Callback`.
- **Action:** Transmits data over UART2 using `HAL_UART_Transmit`.

---

### Key Code Snippet

```c
void HAL_GPIO_EXTI_Callback(uint16_t GPIO_Pin)
{
    const char *message = "Button Press Detected!\r\n";
    if (HAL_UART_Transmit(&huart2, (uint8_t*)message, strlen(message), HAL_MAX_DELAY) != HAL_OK)
    {
        Error_Handler();
    }
}
