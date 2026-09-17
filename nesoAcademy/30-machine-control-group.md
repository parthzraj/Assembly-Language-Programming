# 8085 Microprocessor — Machine Control Group Instructions

This document explains the **Machine Control Group** of instructions in the Intel 8085 — these control the processor's overall operation, interrupts, and a couple of flag-only operations. Background concepts are explained first so the instructions make full sense.

---

## 1. Background Concepts

### 1.1 Interrupts
An **interrupt** is a signal that tells the microprocessor to pause what it's currently doing and jump to a special routine (an **Interrupt Service Routine**, or ISR) to handle some urgent event — then return to where it left off. The 8085 has several hardware interrupt pins (like `RST 7.5`, `RST 6.5`, `RST 5.5`, `TRAP`, and `INTR`), and whether these interrupts are "listened to" or "ignored" is controlled by software using some of the instructions below.

### 1.2 The Interrupt Enable Flip-Flop
Internally, the 8085 has a flip-flop (a 1-bit memory element) that determines whether maskable interrupts are currently allowed. `EI` sets it, `DI` clears it.

### 1.3 Serial I/O Pins
The 8085 has two special pins, **SOD** (Serial Output Data) and **SID** (Serial Input Data), used for simple bit-by-bit serial communication. `SIM` and `RIM` are the instructions used to control/read these, along with interrupt masking.

---

## 2. Interrupt Control Instructions

### `EI` — Enable Interrupts
Sets the interrupt enable flip-flop, allowing the processor to **respond to maskable hardware interrupts** (RST 5.5, RST 6.5, RST 7.5, INTR). By default, interrupts are automatically disabled after a `RST` or after the processor is reset, so `EI` is used to turn them back on when the program is ready to accept them.

### `DI` — Disable Interrupts
Clears the interrupt enable flip-flop, **blocking** all maskable hardware interrupts from being recognized. Used when the processor is doing something critical/time-sensitive that shouldn't be interrupted. (Note: `TRAP`, the non-maskable interrupt, **cannot** be disabled by `DI` — it always gets through.)

### `SIM` — Set Interrupt Mask
"Set Interrupt Mask." A more advanced/flexible instruction than plain `EI`/`DI` — it uses the value **currently in the Accumulator** to:
- Individually mask/unmask the three specific restart interrupts: RST 5.5, RST 6.5, RST 7.5
- Reset the pending status of RST 7.5
- Optionally output a bit through the **SOD** (Serial Output Data) pin

Essentially, `SIM` is programmed by first loading a specific bit pattern into A, then executing `SIM` to apply that configuration.

### `RIM` — Read Interrupt Mask
"Read Interrupt Mask." Reads back the **current interrupt mask status** (which interrupts are enabled/pending) **and** the value on the **SID** (Serial Input Data) pin, loading this information into the Accumulator so the program can inspect it.

---

## 3. Processor Control Instructions

### `NOP` — No Operation
Does absolutely nothing except consume one instruction cycle (advances PC, takes up time). Commonly used for:
- Small timing delays
- Reserving space in memory (e.g., for later patching of code)
- Padding

### `HLT` — Halt
Stops the processor from executing any further instructions. The 8085 enters a halt state (PC does not advance) until it is reset or an interrupt occurs. Used to intentionally stop program execution, e.g., at the end of a program.

### `RST` — Restart
**Operation:** Push PC onto the stack, then PC ← fixed restart address  
A special, shorter form of `CALL` used for interrupt handling. There are 8 restart instructions (`RST 0` through `RST 7`), each jumping to a **fixed, predefined address** in memory (spaced 8 bytes apart: 0000H, 0008H, 0010H, ... 0038H). It works like `CALL` (it saves the return address on the stack first) but is a single byte instruction, making it much faster — this is why it's typically used as the response to hardware interrupts, which need quick handling.

---

## 4. Carry Flag Control Instructions

These two instructions directly manipulate the **Carry flag (CY)** without doing any arithmetic — no operand needed.

### `CMC` — Complement Carry
**Operation:** CY ← NOT(CY)  
**Toggles/inverts** the current value of the Carry flag: if CY was 1, it becomes 0, and vice versa. "CMC" = **Complement Carry**.

### `STC` — Set Carry
**Operation:** CY ← 1  
**Forces** the Carry flag to 1, regardless of its previous value. "STC" = **Set Carry**. Useful for deliberately establishing a known Carry value before an operation like `ADC`/`SBB` that depends on it.

---

## 5. Quick Reference Table

| Mnemonic | Operation | Category |
|---|---|---|
| `RIM` | Read interrupt mask & SID pin into A | Interrupt/serial control |
| `SIM` | Set interrupt mask & SOD pin from A | Interrupt/serial control |
| `EI` | Enable maskable interrupts | Interrupt control |
| `DI` | Disable maskable interrupts | Interrupt control |
| `NOP` | Do nothing (1 cycle delay) | Processor control |
| `HLT` | Halt/stop the processor | Processor control |
| `RST` | Push PC, jump to fixed restart address | Restart / interrupt handling |
| `CMC` | CY ← NOT(CY) | Carry flag control |
| `STC` | CY ← 1 | Carry flag control |

---

## 6. Key Takeaways
- **EI/DI** are the simple on/off switches for maskable interrupts; **SIM/RIM** give finer, per-interrupt control (and also handle serial I/O via SOD/SID).
- **NOP** does nothing useful computationally — it's a timing/placeholder tool.
- **HLT** stops execution entirely until reset or interrupt.
- **RST** is a fast, fixed-address version of `CALL`, mainly used to respond to hardware interrupts.
- **CMC/STC** are the only instructions that directly manipulate the Carry flag on their own, without any accompanying arithmetic — handy for controlling the input to `ADC`/`SBB`/rotate-through-carry instructions.
