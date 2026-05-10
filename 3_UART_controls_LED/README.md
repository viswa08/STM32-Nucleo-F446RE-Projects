# STM32 Nucleo F446RE — UART Controlled LED

> Type `1`. LED on. Type `0`. LED off.
> Your laptop just became a hardware controller.

This project introduces **UART interrupts** — the right way to receive data
without freezing your MCU waiting for input.

---

## What's New Here?

The previous project used polling — the MCU sat and waited for UART events.
This project uses **interrupt-driven UART receive**, meaning:

- The MCU does its own work in the main loop
- When a byte arrives, the hardware interrupts and handles it instantly
- No data is missed. No time is wasted waiting.

This is how real firmware handles communication.

---

## Hardware

Nucleo F446RE + USB cable. Same as before — ST-Link bridges USART2 to your PC.

| Pin | Function |
|---|---|
| PA2 | USART2 TX |
| PA3 | USART2 RX |
| PA5 | LD2 LED (output) |

---

## CubeMX Configuration

Same USART2 settings as the Hello World project (115200, 8N1) with one addition:

**NVIC Settings → USART2 global interrupt → Enabled ✓**

This tells the MCU to fire an interrupt when a byte is received,
instead of making you poll for it.

<img width="1467" height="857" alt="uart_interrupt" src="https://github.com/user-attachments/assets/3d6ca2b2-7198-4e3d-80e5-5c3d5c8df398" />


---

## How It Works

### Starting interrupt reception

Before the main loop, arm the receiver with:

```c
HAL_UART_Receive_IT(&huart2, &rx_data, 1);
```

This tells HAL: *"When 1 byte arrives on USART2, fire the callback."*
It returns immediately — non-blocking.

---

### The callback

When a byte arrives, HAL calls `HAL_UARTRxCpltCallback` automatically:

```c
if (rx_data == '1')
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_SET);   // LED on
else if (rx_data == '0')
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_5, GPIO_PIN_RESET); // LED off

HAL_UART_Receive_IT(&huart2, &rx_data, 1); // re-arm for next byte
```

Two things to notice:
- **Re-arming is mandatory.** HAL disables the interrupt after each callback —
  you must call `Receive_IT` again or you'll only ever receive one byte.
- **The callback echoes back `'1'` or `'0'`** so you can confirm in the
  terminal what the MCU received.

---

### The main loop

```c
while (1)
{
    HAL_UART_Transmit(&huart2, msg, sizeof(msg)-1, HAL_MAX_DELAY);
    HAL_Delay(10000);
}
```

While waiting 10 seconds between transmits, the MCU is still fully responsive
to incoming bytes. **The LED reacts instantly** regardless of where the main
loop is. This is the point — interrupts don't wait for your loop.

---

## Try It

Open Tera Term at 115200 baud on your Nucleo's COM port:

- Type `1` → LED turns on, terminal echoes `1`
- Type `0` → LED turns off, terminal echoes `0`
- Every 10 seconds the MCU prints its status message unprompted

Notice the LED responds **immediately** even during the 10 second delay.
That's interrupt-driven IO in action.

---

## Interrupt vs Polling — When to Use What

| | Polling | Interrupt |
|---|---|---|
| MCU blocks? | Yes | No |
| Misses data? | Possible | No |
| Code complexity | Simple | Slightly more |
| Use when | Learning, one-shot transfers | Real firmware, always-on receive |

---

## What's Next

| # | Project | Concepts |
|---|---|---|
| 01 | LED Blink | GPIO, HAL_Delay |
| 02 | Button Input | GPIO Input, debouncing |
| 03 | Non-blocking Blink | HAL_GetTick, state machine |
| 04 | UART Hello World | UART, polling transmit |
| 05 | **UART LED Control** ← you are here | UART RX interrupt, callback |
| 06 | PWM LED Dimming | Timers, duty cycle |

---

*Found this useful? A ⭐ helps others find this repo.*
