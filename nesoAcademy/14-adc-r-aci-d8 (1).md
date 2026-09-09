# 8085 Microprocessor — Arithmetic Group: ADC R and ACI d8

> Continuing the **Arithmetic Group of Instructions**. This session covers `ADC R` and `ACI d8` — the "add with carry" variants of the addition instructions.

---

## Table of Contents
1. [Why "Add With Carry" Is Needed](#why-add-with-carry-is-needed)
2. [ADC R — Add Register with Carry to Accumulator](#adc-r--add-register-with-carry-to-accumulator)
3. [ACI d8 — Add Immediate with Carry to Accumulator](#aci-d8--add-immediate-with-carry-to-accumulator)
4. [Session Summary](#session-summary)

---

## Why "Add With Carry" Is Needed

The 8085 microprocessor can only add 8-bit numbers at a time. However, sometimes we need to add **multi-byte numbers** (numbers larger than 8 bits, e.g., 16-bit values represented as 4-digit hex numbers).

### Illustration: Adding `4556H` + `33F3H`

Recall: each hexadecimal digit = 4 bits, so a 4-digit hex number = **16 bits**.

**Manual addition (for reference):**
```
   4556
 + 33F3
 ------
```
- `6 + 3 = 9`
- `5 + F` → (shortcut: adding to F gives one less than the addend, plus a carry) → result digit `4`, **carry generated**
- `1 (carry) + 5 = 6`... plus next digit `4 + 3 = 7`

Since the 8085 can only handle **8 bits (2 hex digits) at a time**, this addition must be broken into two steps:
1. **Add the lower byte first:** `56H + F3H = 49H` (using a normal `ADD` instruction) — this step generates a **carry**.
2. **Add the upper byte, including that carry:** `45H + 33H + carry` — this is where **"add with carry"** instructions are needed, since a plain `ADD` would ignore the carry from step 1.

> This is the core motivation for `ADC R` and `ACI d8`: performing **multi-byte (multi-precision) addition** correctly by propagating the carry between successive 8-bit additions.

---

## ADC R — Add Register with Carry to Accumulator

### Meaning
- **ADC** = **A**dd with **C**arry.
- **ADC R** = Add the contents of `R` **plus the current carry flag** to the accumulator.
- `R` (capital, as established previously) includes: all GPRs, the accumulator itself, **and** the memory element `M` (pointed to by HL).
- The result is stored back in the **accumulator**.

### Instruction length
- `ADC R` is a **1-byte long instruction**.

### Instruction variations
8 specific instructions, following the same pattern as `ADD R`:
- `ADC A`, `ADC B`, `ADC C`, `ADC D`, `ADC E`, `ADC H`, `ADC L`, `ADC M`

### Worked Example: `ADC B`

**Scenario:** Continuing the multi-byte addition example from above.
- Accumulator holds `45H` (the upper byte of the first number).
- Register B holds `33H` (the upper byte of the second number).
- The **Carry flag is already set** (from the previous lower-byte addition, which generated a carry).

**Execution of `ADC B`:**
```
45H (accumulator)
+ 33H (register B)
+ 1  (carry)
------
79H
```
Result: Accumulator = `79H`

### Effect on the Flags register (for this example)

| Flag | Result | Reasoning |
|---|---|---|
| **CY** (Carry) | **Reset (0)** | No carry was generated from this addition (`45 + 33 + 1 = 79`, no overflow past 8 bits). |
| **P** (Parity) | **Reset (0)** | `79H` = `01111001` in binary → 5 ones (**odd**) → parity reset (only set for **even** count). |
| **AC** (Auxiliary Carry) | **Reset (0)** | No carry occurred from the addition of the least significant 4 bits. |
| **Z** (Zero) | **Reset (0)** | The result (`79H`) is not all zeros. |
| **S** (Sign) | **Reset (0)** | `79H` = `01111001` — most significant bit is `0`. |

### Key takeaway
> `ADC R` is a 1-byte instruction that adds the content of `R` (GPR, accumulator, or memory via HL) **plus the current carry** to the accumulator, storing the result back in the accumulator. It's essential for correctly performing **multi-byte addition**.

---

## ACI d8 — Add Immediate with Carry to Accumulator

### Meaning
- **ACI** = **A**dd with **C**arry **I**mmediate to accumulator.
- Same concept as `ADC R`, but the second operand is provided as an **8-bit immediate value** directly within the instruction (immediate addressing mode), rather than from a register.

### Instruction length
- Opcode `ACI` → 8 bits (1 byte)
- Immediate data `d8` → 8 bits (1 byte)
- **Total = 2 bytes** → `ACI d8` is a **2-byte long instruction**.

### Worked Example: `ACI 12H`

**Scenario:**
- Accumulator holds `F1H`.
- The **Carry flag is already set** (assume 1, from a prior operation).
- Immediate data sent via the instruction: `12H`.

**Execution:**
```
 F1H (accumulator)
+ 12H (immediate data)
+  1  (carry)
------
```
- Least significant digits: `1 + 2 + 1(carry) = 4`
- Most significant digits: `F + 1` → (shortcut: adding to F gives one less than the addend, plus a carry) → result digit `0`, with a **carry generated**.

**Result:** Accumulator = `04H`, and a **carry** is generated (carry flag set).

### Effect on the Flags register (for this example)

| Flag | Result | Reasoning |
|---|---|---|
| **CY** (Carry) | **Set (1)** | A carry was generated from the most significant digit addition (`F + 1`). |
| **P** (Parity) | **Reset (0)** | `04H` = `00000100` in binary → only **1 one** (odd) → parity reset. |
| **AC** (Auxiliary Carry) | **Reset (0)** | No carry occurred from the least significant 4 bits (`1 + 2 + 1 = 4`, no overflow). |
| **Z** (Zero) | **Reset (0)** | The result (`04H`) is not all zeros. |
| **S** (Sign) | **Reset (0)** | `04H` = `00000100` — most significant bit is `0`. |

### Key takeaway
> `ACI d8` is a 2-byte instruction that adds an 8-bit immediate value **plus the current carry** to the accumulator, storing the result back in the accumulator. Like `ADC R`, it's used for correctly propagating carry across multi-byte additions — but with an immediate operand instead of a register/memory operand.

---

## Session Summary

| Instruction Type | Meaning | Length | Op-Codes | Addressing Mode |
|---|---|---|---|---|
| `ADC R` | Add register R (GPR/Acc/M) + carry to accumulator | 1 byte | 8 (A,B,C,D,E,H,L,M) | Register / Register Indirect (for M) |
| `ACI d8` | Add immediate 8-bit data + carry to accumulator | 2 bytes | 1 | Immediate |

**Key points to remember:**
- Both `ADC R` and `ACI d8` are the **"add with carry"** counterparts of `ADD R` and `ADI d8`, respectively.
- Their primary purpose is enabling **correct multi-byte (multi-precision) addition** — where a carry generated from adding the lower bytes must be included when adding the upper bytes.
- Like all arithmetic instructions, both **affect the entire Flags register** (CY, P, AC, Z, S) based on the outcome of the operation.
- `ADC R` uses capital `R`, meaning it includes GPRs, the accumulator, and the memory element `M` (via HL) — just like `ADD R`.

---

*Next session: Instructions related to the subtraction operation.*
