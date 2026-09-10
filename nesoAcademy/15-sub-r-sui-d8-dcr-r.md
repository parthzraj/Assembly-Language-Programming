# 8085 Microprocessor — Arithmetic Group: SUB R, SUI d8, and DCR R

> Continuing the **Arithmetic Group of Instructions** — shifting focus from addition to **subtraction**. This session covers `SUB R`, `SUI d8`, and `DCR R`.

---

## Table of Contents
1. [SUB R — Subtract Register from Accumulator](#sub-r--subtract-register-from-accumulator)
2. [How the 8085 Actually Performs Subtraction (16's Complement Method)](#how-the-8085-actually-performs-subtraction-16s-complement-method)
3. [Worked Example: SUB B](#worked-example-sub-b)
4. [SUI d8 — Subtract Immediate from Accumulator](#sui-d8--subtract-immediate-from-accumulator)
5. [DCR R — Decrement Contents of R](#dcr-r--decrement-contents-of-r)
6. [Session Summary](#session-summary)

---

## SUB R — Subtract Register from Accumulator

### Meaning
- **SUB** = **Sub**tract.
- **SUB R** = Subtract the contents of `R` **from** the accumulator.
- `R` (capital) includes: all GPRs, the accumulator itself, and the memory element `M` (pointed to by HL) — **8 instructions total**.
- One operand is always in the **accumulator**; the result is stored back in the **accumulator**.

### Instruction length
- `SUB R` is a **1-byte long instruction**, just like `ADD R`.

### Interesting fact: `SUB A`
Executing `SUB A` subtracts the accumulator's value **from itself**. Since both operands are identical, the result is always **all zeros** in the accumulator.

---

## How the 8085 Actually Performs Subtraction (16's Complement Method)

Important concept: **the 8085 does not perform "true" subtraction** — it performs subtraction **as addition**, using the **16's complement** of the subtrahend (the hexadecimal equivalent of two's complement in binary).

### How to compute the 16's complement of a hex number (digit-wise shortcut)
- **Least significant digit:** subtract it from **`10H` (16 in decimal)**.
- **All other (more significant) digits:** subtract each from **`F`**.

This digit-wise trick automatically accounts for the "+1" step of two's complement, without needing to compute it separately.

### Why this works
When the 16's complement of the subtrahend is **added** to the minuend:
- If a **carry is generated** out of the most significant digit, it means the result is a **positive value** — the minuend was greater than the subtrahend. This carry is **discarded** (it's not a "real" carry in the arithmetic sense).
- The carry generated during this complement-addition process is fed through an **inverter (NOT gate)** before reaching the **Carry (CY) flag** — so a generated carry here actually **resets** CY to 0, signaling a *positive* result.

> **Key rule:** After a subtraction (via 16's complement addition), if `CY = 0`, the result is **positive**. If `CY = 1` (i.e., no carry was generated during the complement-addition), the result is **negative**, and the Sign flag becomes meaningful. When CY = 0, the Sign flag being set is effectively "ignored" for interpreting positivity, since the carry logic already confirms the result is positive.

---

## Worked Example: SUB B

**Scenario:**
- Accumulator holds `F1H`
- Register B holds `12H`
- Operation: `F1H − 12H` (i.e., `SUB B`)

### Step 1: Compute the 16's complement of the subtrahend (`12H`)
- Least significant digit: `2` → `10H (16) − 2 = 14 = E`
- Most significant digit: `1` → `F − 1 = E`
- **16's complement of `12H` = `EEH`**

### Step 2: Add the minuend and the complement
```
   F1H  (minuend, in accumulator)
 + EEH  (16's complement of subtrahend)
 ------
```
- Least significant digits: `1 + E = F` (no carry)
- Most significant digits: `F + E` → (shortcut: adding to F gives one less than the addend, plus carry) → digit `D`, **carry generated**

**Raw result:** `1DFH` (a 3-digit hex number)

### Step 3: Truncate and handle the carry
- Only the least significant **2 digits (8 bits)** are stored in the accumulator: **`DFH`**
- The generated carry (`1`) passes through the **inverter**, so **CY is reset to 0** — confirming the result is **positive**.

**Final result:** Accumulator = `DFH`, CY = 0

### Effect on the Flags register

| Flag | Result | Reasoning |
|---|---|---|
| **CY** (Carry) | **Reset (0)** | The generated carry was inverted — signals a positive result. |
| **P** (Parity) | **Reset (0)** | `DFH` = `1101 1111` → D (`1101`) has 3 ones + F (`1111`) has 4 ones = **7 ones** (odd) → parity reset. |
| **AC** (Auxiliary Carry) | **Reset (0)** | No carry was generated from the least significant digit addition (`1 + E = F`, no overflow). |
| **Z** (Zero) | **Reset (0)** | `DFH` is not all zeros. |
| **S** (Sign) | **Set (1)** | MSB of `DFH` (`D` = `1101`) is `1` → sign bit set. *(Note: this is effectively ignored here since CY = 0 already confirms a positive result.)* |

---

## SUI d8 — Subtract Immediate from Accumulator

### Meaning
- **SUI** = **S**ubtract **I**mmediate from accumulator.
- Same underlying operation as `SUB R`, but the subtrahend is provided as an **8-bit immediate value** directly within the instruction (immediate addressing mode).

### Instruction length
- Opcode `SUI` → 8 bits (1 byte)
- Immediate data `d8` → 8 bits (1 byte)
- **Total = 2 bytes** → `SUI d8` is a **2-byte long instruction**.
- Unlike `SUB R` (8 variations), **`SUI d8` has only a single instruction/op-code.**

### Worked Example: `SUI 12H`

**Scenario:**
- Accumulator holds `F1H`
- Immediate subtrahend sent via instruction: `12H`
- (Same numbers as the `SUB B` example — the only difference is *how* the subtrahend is supplied: immediately, rather than from register B.)

### Computation
- 16's complement of `12H` = `EEH` (as computed above)
- `F1H + EEH`:
  - Least significant digits: `1 + E = F` (no carry)
  - Most significant digits: `F + E = D`, carry generated
- Raw result: `1DFH` → truncated to `DFH`
- Carry passes through the inverter → **CY reset to 0**

**Final result:** Accumulator = `DFH`, CY = 0

### Effect on the Flags register

| Flag | Result | Reasoning |
|---|---|---|
| **CY** (Carry) | **Reset (0)** | Generated carry inverted — signals a positive result. |
| **P** (Parity) | **Reset (0)** | `DFH` has 7 ones (odd) → parity reset. |
| **AC** (Auxiliary Carry) | **Reset (0)** | No carry from the least significant digit addition. |
| **Z** (Zero) | **Reset (0)** | `DFH` is not all zeros. |
| **S** (Sign) | **Set (1)** | MSB of `DFH` is `1` — sign set, but ignored since CY = 0 confirms positivity. |

---

## DCR R — Decrement Contents of R

### Meaning
- **DCR** = **D**e**cr**ement.
- **DCR R** = Decrement (subtract 1 from) the contents of `R` by 1.
- `R` (capital) includes: the accumulator, all GPRs, and the memory element `M` (pointed to by HL) — **8 instructions total**: `DCR A`, `DCR B`, `DCR C`, `DCR D`, `DCR E`, `DCR H`, `DCR L`, `DCR M`.

### Instruction length
- `DCR R` is a **1-byte long instruction**.

### Flags affected
- Just like `INR R`, **`DCR R` affects all flags except the Carry (CY) flag.**
- Reasoning is the same as for `INR R`: this is a simple decrement (not a two-operand subtraction), so no meaningful carry/borrow signal is generated or tracked by CY.

### Example: `DCR A`
Suppose the accumulator holds `03H`.

**Execution:** The content is decremented by 1 → accumulator becomes `02H`.

### Example: `DCR M`
Suppose memory location `F820H` (pointed to by HL) contains `FFH`.

**Precondition:** The HL register pair must first be loaded with the address `F820H`.

**Execution of `DCR M`:**
- The content at that memory location (`FFH`) is decremented by 1.
- Result: the memory location now contains `FEH`.

### Addressing mode
- `DCR M` does not mention HL directly — it's referenced indirectly via `M`, making it an example of **register indirect addressing mode**.

---

## Session Summary

| Instruction Type | Meaning | Length | Op-Codes | Flags Affected |
|---|---|---|---|---|
| `SUB R` | Subtract register R (GPR/Acc/M) from accumulator | 1 byte | 8 (A,B,C,D,E,H,L,M) | All flags |
| `SUI d8` | Subtract immediate 8-bit data from accumulator | 2 bytes | 1 | All flags |
| `DCR R` | Decrement contents of R (GPR/Acc/M) by 1 | 1 byte | 8 (A,B,C,D,E,H,L,M) | All flags **except CY** |

**Key points to remember:**
- The 8085 performs subtraction **internally as addition**, using the **16's complement** of the subtrahend.
- **16's complement shortcut:** subtract the least significant digit from `10H`; subtract all other digits from `F`.
- After complement-addition, a **generated carry is inverted** before reaching the CY flag:
  - **CY = 0** after subtraction → result is **positive**.
  - **CY = 1** after subtraction → result is **negative**.
- `SUB R` and `SUI d8` follow the same 1-byte / 2-byte and register / immediate addressing pattern as `ADD R` / `ADI d8`.
- `DCR R`, like `INR R`, affects **all flags except CY**, since it's a simple decrement, not a full two-operand subtraction.

---

*Next session: More instructions related to the subtraction operation.*
