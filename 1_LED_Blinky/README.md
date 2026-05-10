# STM32 Nucleo F446RE — LED Blink

> The "Hello, World!" of embedded systems - simple to run, 
> more to understand than you'd think

---

## Hardware

Nucleo F446RE + USB cable. That's it.

The onboard **LD2 LED is wired to PA5** — no breadboard, no resistors needed.

> **External LED?** Any free GPIO → 330Ω resistor → LED → GND.

---

## Clock Configuration

Default CubeMX config uses **HSI (16 MHz)** through the PLL:

HSI (16 MHz) → /PLLM(16) → xPLLN(336) → /PLLP(4) = 84 MHz SYSCLK

> The MCU configures all of this **before your `main()` runs** — 
> inside `SystemInit()` and `HAL_Init()`.

<img width="1522" height="762" alt="image" src="https://github.com/user-attachments/assets/e53ada39-cc84-4c20-87f2-1a929f489daf" />

## The Code

```c
while (1)
{
    HAL_GPIO_TogglePin(LD2_GPIO_Port, LD2_Pin);
    HAL_Delay(500);
}
```

| Function | What it does |
|---|---|
| `HAL_GPIO_TogglePin` | Flips the PA5 output bit in the GPIO ODR register |
| `HAL_Delay(500)` | Blocks for 500ms using SysTick interrupts |

> ⚠️ `HAL_Delay()` is fine here. In real firmware — a motor controller,
> a communication handler, anything time-sensitive — **it will cause bugs.**
> Use `HAL_GetTick()` for non-blocking delays instead.

---
## Build & Flash

1. Open **STM32CubeIDE** → `File` → `Open Projects from File System`
2. Select the cloned folder → **Build** `Ctrl+B` → **Run** `F11`

ST-Link is built into the Nucleo — no external programmer needed.

---



