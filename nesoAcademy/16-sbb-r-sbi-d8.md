# 8085 Microprocessor — Arithmetic Group: SBB R and SBI d8

> Continuing the subtraction-related arithmetic instructions. This session covers `SBB R` and `SBI d8` — the **"subtract with borrow"** variants, essential for multi-byte subtraction.

---

## Table of Contents
1. [Why "Subtract With Borrow" Is Needed](#why-subtract-with-borrow-is-needed)
2. [SBB R — Subtract Register with Borrow from Accumulator](#sbb-r--subtract-register-with-borrow-from-accumulator)
3. [Worked Example: Multi-Byte Subtraction Using SUB B and SBB C](#worked-example-multi-byte-subtraction-using-sub-b-and-sbb-c)
4. [SBI d8 — Subtract Immediate with Borrow from Accumulator](#sbi-d8--subtract-immediate-with-borrow-from-accumulator)
5. [Session Summary](#session-summary)

---

## Why "Subtract With Borrow" Is Needed

Just as `ADC R` handles **carry propagation** for multi-byte addition, `SBB R` handles **borrow propagation** for multi-byte subtraction.

### Illustration: Subtracting `12F2H` from `3456H`

Both numbers are 4-digit hex (16-bit) values. Manual subtraction:
```
  3456
- 12F2
------
```
- `6 − 2 = 4`
- `5 − F` → since `5 < F`, we must **borrow 1** from the next digit place, making it `15 (decimal)`. Then `15 − F(15) = 0`... 

Wait — let's follow the transcript's actual working:
- `5 − F`: borrow needed. Borrowing makes this digit `15` (decimal, i.e., `1` "borrowed" as 16 + 5). Converting `15H` to decimal: `1×16¹ + 5×16⁰ = 16 + 5 = 21` (decimal). `F` in decimal is `15`. `21 − 15 = 6` → result digit `6`.
- The digit we borrowed from (`3`) becomes `2` (i.e., `3 − 1`).
- `2 − 1 = 1`
- `3 − 1 = 2` *(using the now-reduced digit `2`, minus the next subtrahend digit `1`... following through the full borrow chain)*

**Final result: `3456H − 12F2H = 2164H`**

Since the 8085 can only operate on **8 bits (2 hex digits) at a time**, this subtraction must be split into two parts:
1. **Lower byte:** `56H − F2H` (using `SUB B`) — this may require a borrow.
2. **Upper byte:** `34H − 12H`, **also accounting for any borrow generated from step 1** — this is exactly what `SBB R` is for.

> This is the core motivation for `SBB R` and `SBI d8`: correctly propagating **borrows** across successive 8-bit subtractions when working with multi-byte numbers.

---

## SBB R — Subtract Register with Borrow from Accumulator

### Meaning
- **SBB** = **S**u**b**tract with **B**orrow.
- **SBB R** = Subtract the contents of `R`, **plus any existing borrow (Carry flag)**, from the accumulator.
- `R` (capital) includes: all GPRs, the accumulator itself, and the memory element `M` (pointed to by HL) — **8 instructions total**: `SBB A`, `SBB B`, `SBB C`, `SBB D`, `SBB E`, `SBB H`, `SBB L`, `SBB M`.
- The result is stored back in the **accumulator**.

### Instruction length
- `SBB R` is a **1-byte long instruction**, following the same pattern as other `R`-type instructions.

### Important concept: Carry flag doubles as the "Borrow" flag
During subtraction operations, the **Carry (CY) flag** is reinterpreted as the **Borrow flag**:
- If, after a subtraction, **CY is set (1)** → it means a **borrow was needed** (the result was negative).
- If **CY is reset (0)** → no borrow was needed (the result was positive).

When performing the **next** (higher-byte) subtraction in a multi-byte operation, this borrow must be **factored in** — either by:
- Subtracting 1 more from the minuend, **or**
- Adding 1 to the subtrahend (mathematically equivalent).

`SBB R` automatically incorporates the current Carry/Borrow flag into the subtraction.

---

## Worked Example: Multi-Byte Subtraction Using SUB B and SBB C

**Goal:** Compute `3456H − 12F2H` using the 8085, split into two 8-bit operations.

### Step 1 — Lower byte: `56H − F2H` using `SUB B`

- Accumulator = `56H` (minuend)
- Register B = `F2H` (subtrahend)

**16's complement of `F2H`:**
- Least significant digit: `2` → `10H (16) − 2 = 14 = E`
- Most significant digit: `F` → `F − F = 0`
- Complement = `0EH`

**Addition:** `56H + 0EH`
- LSD: `6 + E = 20 (decimal) = 0x14` → digit `4`, carry `1`
- MSD: `5 + 0 + 1(carry) = 6`
- Raw result: `64H` — **no overall carry generated** (sum stays within 8 bits)

**Result:** Accumulator = `64H`. Since **no carry was generated**, the result is treated as **negative**, so after passing through the inverter, the **Carry (Borrow) flag is SET**.

> This matches the manual calculation: the lower byte result should be `64H`, and a borrow was indeed needed (since `56H < F2H`).

### Step 2 — Upper byte: `34H − 12H`, accounting for the borrow, using `SBB C`

- Accumulator = `34H` (minuend)
- Register C = `12H` (subtrahend)
- **Carry (Borrow) flag is already SET** from Step 1 → this borrow must be added to the subtrahend: `12H + 1 = 13H`

**16's complement of `13H`:**
- Least significant digit: `3` → `10H (16) − 3 = 13 = D`
- Most significant digit: `1` → `F − 1 = E`
- Complement = `EDH`

**Addition:** `34H + EDH`
- LSD: `4 + D = 17 (decimal) = 0x11` → digit `1`, carry `1`
- MSD: `3 + E + 1(carry) = 18 (decimal) = 0x12` → digit `2`, carry `1` (**overall carry generated**)
- Raw result: `121H` → truncated to **`21H`**

**Result:** Accumulator = `21H`. Since a **carry WAS generated** this time, the result is **positive**, so after the inverter, the **Carry (Borrow) flag is RESET**.

### Final Combined Result
- Upper byte: `21H`, Lower byte: `64H`
- **Combined result: `2164H`** ✅ — matches the manual subtraction performed earlier!

---

## SBI d8 — Subtract Immediate with Borrow from Accumulator

### Meaning
- **SBI** = **S**ubtract with **B**orrow **I**mmediate from accumulator.
- Same operation as `SBB R`, but the subtrahend is provided as an **8-bit immediate value** directly within the instruction (immediate addressing mode).

### Instruction length
- Opcode `SBI` → 8 bits (1 byte)
- Immediate data `d8` → 8 bits (1 byte)
- **Total = 2 bytes** → `SBI d8` is a **2-byte long instruction**.
- Like `SUI d8` and `ACI d8`, there is only a **single instruction/op-code** for this type.

### Worked Example: `SBI F3H`

**Scenario:**
- Accumulator holds `33H`
- Immediate subtrahend sent via instruction: `F3H`
- **Carry (Borrow) flag is already SET** before this instruction executes (a borrow is needed from a prior operation)

**Step 1: Incorporate the existing borrow into the subtrahend**
- `F3H + 1 (borrow) = F4H`

**Step 2: Compute the 16's complement of `F4H`**
- Least significant digit: `4` → `10H (16) − 4 = 12 = C`
- Most significant digit: `F` → `F − F = 0`
- Complement = `0CH`

**Step 3: Add the minuend and the complement**
- `33H + 0CH`:
  - LSD: `3 + C = 15 (decimal) = F`
  - MSD: `3 + 0 = 3`
- Raw result: `3FH` — **no carry generated**

**Result:** Accumulator = `3FH`. Since **no carry was generated**, the result is **negative**, so after the inverter, the **Carry (Borrow) flag is SET**.

### Key takeaway
> `SBI d8` is a 2-byte instruction that subtracts an 8-bit immediate value (plus any existing borrow) from the accumulator, storing the result back in the accumulator and updating the Flags register — following the same 16's complement addition method as `SBB R`, `SUB R`, and `SUI d8`.

---

## Session Summary

| Instruction Type | Meaning | Length | Op-Codes | Addressing Mode |
|---|---|---|---|---|
| `SBB R` | Subtract register R (GPR/Acc/M) + borrow from accumulator | 1 byte | 8 (A,B,C,D,E,H,L,M) | Register / Register Indirect (for M) |
| `SBI d8` | Subtract immediate 8-bit data + borrow from accumulator | 2 bytes | 1 | Immediate |

**Key points to remember:**
- `SBB R` and `SBI d8` are the **"subtract with borrow"** counterparts of `SUB R` and `SUI d8`, respectively — just as `ADC R`/`ACI d8` are to `ADD R`/`ADI d8`.
- Their primary purpose is enabling **correct multi-byte subtraction**, propagating any borrow generated from a lower-byte subtraction into the next (higher-byte) subtraction.
- The **Carry (CY) flag doubles as the Borrow flag** during subtraction operations:
  - **CY = 0** after a subtraction → result is **positive** (no borrow needed).
  - **CY = 1** after a subtraction → result is **negative** (a borrow was needed).
- To incorporate an existing borrow into the next subtraction step, **add 1 to the subtrahend** before computing its 16's complement (equivalent to subtracting 1 more from the minuend).
- Both instructions follow the same **16's complement addition method** used throughout the subtraction-related instructions in this series.

---

*Next session: A couple more arithmetic instructions.*
