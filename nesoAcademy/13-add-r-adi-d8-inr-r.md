# 8085 Microprocessor — Arithmetic Group: ADD R, ADI d8, and INR R

> Beginning the **Arithmetic Group of Instructions** — the second of 7 instruction groups in the 8085 microprocessor. This group has **14 instruction types**, totaling **62 op-codes**. This session covers the first three: `ADD R`, `ADI d8`, and `INR R`.

---

## Table of Contents
1. [Recap & Introduction to the Arithmetic Group](#recap--introduction-to-the-arithmetic-group)
2. [The Flags Register — Recap](#the-flags-register--recap)
3. [ADD R — Add Register to Accumulator](#add-r--add-register-to-accumulator)
4. [ADI d8 — Add Immediate to Accumulator](#adi-d8--add-immediate-to-accumulator)
5. [INR R — Increment Contents of R](#inr-r--increment-contents-of-r)
6. [Session Summary](#session-summary)

---

## Recap & Introduction to the Arithmetic Group

- The **Data Transfer Group** (13 instruction types, 83 op-codes) has been fully covered.
- Starting this session: the **Arithmetic Group**, which contains:
  - **14 instruction types**
  - **62 op-codes** (cumulative, to be covered across upcoming sessions)
- This session covers the first three instruction types: **`ADD R`**, **`ADI d8`**, and **`INR R`**.

---

## The Flags Register — Recap

Since **all arithmetic instructions affect the Flags register**, it's important to recall its structure. The Flags register is **8 bits**, with the following bit assignments (from LSB to MSB):

| Bit Position | Flag | Meaning |
|---|---|---|
| 0 (LSB) | **CY** (Carry) | Set when an operation produces a carry/borrow out of the most significant bit |
| 1 | — | Don't care (unused) |
| 2 | **P** (Parity) | Set to 1 if the number of 1s in the accumulator (after the operation) is **even** |
| 3 | — | Don't care (unused) |
| 4 | **AC** (Auxiliary Carry) | Set when a carry occurs out of the **least significant 4 bits** (nibble) during addition |
| 5 | — | Don't care (unused) |
| 6 | **Z** (Zero) | Set to 1 if the accumulator's content (after the operation) is **all zeros** |
| 7 (MSB) | **S** (Sign) | Reflects the **most significant bit** of the accumulator (two's complement sign convention) |

> With the introduction of arithmetic instructions, the **Flags register** is now added to the programmer's view of the 8085, alongside the GPRs and accumulator.

---

## ADD R — Add Register to Accumulator

### Meaning
- **ADD R** = Add the contents of register `R` to the **accumulator**.
- `R` refers to any of the **6 GPRs**, the **accumulator itself**, or the **memory location pointed to by HL** (denoted `M`).

### Why one operand must be in the accumulator
For any binary operation (an operation involving two operands, such as addition):
- **One operand must always reside in the accumulator.**
- **The other operand** can reside anywhere — specified by `R`.
- **The result is always stored back in the accumulator.**

### Instruction length
- `ADD R` is a **1-byte long instruction**.

### Example: `ADD B`
(Previously seen in the Flags Register session.)
- One operand resides in the **accumulator**; the other resides in register **B**.
- After execution, the sum (e.g., `38H`) is stored back in the **accumulator**.
- Any carry generated is stored in the **Flags register**.

### Instruction variations
The `ADD R` type covers 8 specific instructions:
- `ADD A` — adds the accumulator to itself (**doubles** its content).
- `ADD B`, `ADD C`, `ADD D`, `ADD E`, `ADD H`, `ADD L` — adds the respective GPR to the accumulator.
- `ADD M` — adds the content of the memory location pointed to by the **HL register pair** to the accumulator.

### Why "R" is capitalized (vs. lowercase "r" in Data Transfer instructions)
- In earlier **Data Transfer** instruction types, lowercase `r` referred only to the **accumulator and GPRs**.
- Here, **capital `R`** is used because this instruction type **also includes `ADD M`** — i.e., it includes the **memory element** (pointed to by HL), not just registers.

> **Convention:** Lowercase `r` = accumulator + GPRs only. Capital `R` = accumulator + GPRs + memory element (M, via HL).

### Addressing mode of ADD M
- `ADD M` does not mention the HL register pair directly — it's referenced **indirectly** via `M`.
- This makes `ADD M` an example of **register indirect addressing mode**.

### Key takeaway
> `ADD R` is a 1-byte instruction that adds the content of `R` (a GPR, the accumulator, or memory via HL) to the accumulator, storing the result back in the accumulator, and updating the Flags register.

---

## ADI d8 — Add Immediate to Accumulator

### Meaning
- **ADI** = **A**dd **I**mmediate to accumulator.
- One operand is in the **accumulator**; the other operand is the **8-bit immediate data** (`d8`) sent directly within the instruction.

### Instruction length
- Opcode `ADI` → 8 bits (1 byte)
- Immediate data `d8` → 8 bits (1 byte)
- **Total = 2 bytes** → `ADI d8` is a **2-byte long instruction**.

> Note: Unlike `ADD R` (which has 8 variations), `ADI d8` has **only a single op-code** — the second byte of the instruction simply carries whatever 8-bit data is being added.

### Worked Example: `ADI 12H`

Suppose the accumulator currently holds `F1H`.

**Execution:** `F1H + 12H`
- Adding the least significant digits: `1 + 2 = 3`
- Adding the most significant digits: `F + 1` → (shortcut: adding to `F`, the largest hex digit, gives a result **one less than the addend**, plus a carry) → result digit = `0`, with a **carry generated**.

**Result:** Accumulator = `03H`, and a **carry** is generated.

### Effect on the Flags register (for this example)

| Flag | Result | Reasoning |
|---|---|---|
| **CY** (Carry) | **Set (1)** | A carry was generated from the most significant digit addition. |
| **P** (Parity) | **Set (1)** | `03H` = `00000011` in binary — has **2 ones** (even) → parity set. |
| **AC** (Auxiliary Carry) | **Reset (0)** | No carry occurred from the least significant 4 bits (`1 + 2 = 3`, no overflow). |
| **Z** (Zero) | **Reset (0)** | The accumulator's result (`03H`) is **not** all zeros. |
| **S** (Sign) | **Reset (0)** | `03H` = `00000011` — the most significant bit is `0`, so sign is reset. |

### Key takeaway
> `ADI d8` is a 2-byte instruction that adds an 8-bit immediate value to the accumulator (immediate addressing mode), storing the result back in the accumulator and updating all Flags register bits based on the outcome.

---

## INR R — Increment Contents of R

### Meaning
- **INR** = **In**crement.
- **INR R** = Increment the contents of `R` by **1**.
- `R` here (capital, as with `ADD R`) refers to any GPR, the accumulator, **or** the memory element `M` (pointed to by HL).

### Instruction length
- `INR R` is a **1-byte long instruction**, just like `ADD R`.

### Example: `INR A`
Suppose the accumulator holds `03H`.

**Execution:** The content is incremented by 1 → accumulator becomes `04H`.

### Instruction variations
- `INR A`, `INR B`, `INR C`, `INR D`, `INR E`, `INR H`, `INR L`
- `INR M` — increments the content of the memory location pointed to by HL.

> After execution, the incremented result is stored back **in the same location** it came from (register or memory).

### Effect on the Flags register — Why Carry (CY) Is NOT Affected

`INR R` affects **all flags except the Carry (CY) flag.**

**Reasoning, illustrated with `INR M`:**

Suppose memory location `F820H` (pointed to by HL) contains `FFH` (all 1s in 8 bits).

**Execution of `INR M`:**
- `FFH` is the largest possible 2-digit hex value achievable within 8 bits.
- Incrementing `FFH` by 1 would logically produce `100H` — but this is a **3-digit** hex number, which **cannot fit** in an 8-bit memory location.
- As a result, the value **wraps around/resets to `00H`**.

**Why CY is unaffected:** This operation is an **increment**, not an **addition** of two independent operands. Since it's not a true two-operand addition, **no carry flag is generated or affected** — the value simply resets/overflows without setting CY.

> **Key distinction:** `ADD`/`ADI` operations *can* set the carry flag because they involve genuine two-operand addition. `INR` does **not** affect the carry flag, even when the value overflows (e.g., `FFH → 00H`), because it's treated as a simple increment operation.

### Key takeaway
> `INR R` is a 1-byte instruction that increments the content of `R` (a GPR, accumulator, or memory via HL) by 1, storing the result back in the same location, and affecting **all Flags register bits except Carry (CY)**.

---

## Session Summary

| Instruction Type | Meaning | Length | Op-Codes | Flags Affected |
|---|---|---|---|---|
| `ADD R` | Add register R (GPR/Acc/M) to accumulator | 1 byte | 8 (A,B,C,D,E,H,L,M) | All flags |
| `ADI d8` | Add immediate 8-bit data to accumulator | 2 bytes | 1 | All flags |
| `INR R` | Increment contents of R (GPR/Acc/M) by 1 | 1 byte | 8 (A,B,C,D,E,H,L,M) | All flags **except CY** |

**Key points to remember:**
- All arithmetic instructions **affect the Flags register**, which is now part of the programmer's view.
- For any binary operation, **one operand is always in the accumulator**, and the **result is stored back in the accumulator**.
- **Capital `R`** (vs. lowercase `r` in Data Transfer instructions) signals that the memory element `M` (via HL) is included as a valid operand location.
- `ADD M` and other `M`-based instructions use **register indirect addressing mode**.
- `INR R` does **not** affect the Carry flag — because it's a simple increment, not a two-operand addition — even when the value overflows (e.g., `FFH → 00H`).

---

*Next session: Two more instruction types related to the addition operation.*
