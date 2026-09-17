# 8085 Microprocessor — Logic Group Instructions

This document explains the **Logic Group** of instructions in the Intel 8085 instruction set, along with the background concepts needed to understand them.

---

## 1. Background Concepts

### 1.1 What This Group Does
The Logic Group performs **bitwise logical operations** (AND, OR, XOR, Compare) and **rotate operations** on the Accumulator (A). Almost every instruction here either:
- combines A with another value bit-by-bit, or
- shifts/rotates the bits within A.

### 1.2 Operand Sources (same pattern as Arithmetic Group)
| Operand type | Example | Meaning |
|---|---|---|
| Register | `ANA B` | Uses the value in register B |
| Memory (via HL) | `ANA M` | Uses the value at the memory address in HL |
| Immediate data | `ANI 25H` | Uses a constant value written directly in the instruction |

### 1.3 Flags Involved
- **CY (Carry)** — used heavily in rotate instructions; also cleared by AND/OR/XOR operations (except in special cases).
- **Z (Zero)** — set if the result is zero.
- **S (Sign)**, **P (Parity)** — set based on the result.
- **AC (Auxiliary Carry)** — set by `ANA`/`ANI` (logical AND) operations specifically; cleared by OR/XOR.

---

## 2. Logical AND Instructions

### 37) `ANA B`
**Operation:** A ← A AND B  
Performs a bitwise AND between the Accumulator and register B. A bit in the result is 1 only if **both** corresponding bits in A and B are 1. Commonly used to **mask off (clear) specific bits** while keeping others.

### 38) `ANA M`
**Operation:** A ← A AND [HL]  
Same AND operation, but the second operand is the value at the memory location pointed to by HL.

### 39) `ANI 25H`
**Operation:** A ← A AND 25H  
ANDs the Accumulator with the 8-bit immediate value 25H. "ANI" = **AND Immediate**.

---

## 3. Logical OR Instructions

### 40) `ORA B`
**Operation:** A ← A OR B  
Performs a bitwise OR between A and register B. A bit in the result is 1 if **either** bit in A or B is 1. Commonly used to **set specific bits** without disturbing others.

### 41) `ORA M`
**Operation:** A ← A OR [HL]  
Same OR operation, with the second operand taken from memory (address in HL).

### 42) `ORI 25H`
**Operation:** A ← A OR 25H  
ORs the Accumulator with the immediate value 25H. "ORI" = **OR Immediate**.

---

## 4. Logical XOR (Exclusive OR) Instructions

### 43) `XRA B`
**Operation:** A ← A XOR B  
Performs a bitwise XOR (exclusive OR) between A and B. A bit in the result is 1 only if the corresponding bits in A and B are **different**. `XRA A` (A XORed with itself) is a common trick to **clear the Accumulator to zero**.

### 44) `XRA M`
**Operation:** A ← A XOR [HL]  
Same XOR operation, second operand from memory.

### 45) `XRI 25H`
**Operation:** A ← A XOR 25H  
XORs A with the immediate value 25H. "XRI" = **XOR Immediate**. Often used to **toggle** specific bits.

---

## 5. Rotate Instructions

Rotate instructions shift all bits in the Accumulator left or right by one position. The bit that "falls off" one end is placed both into the **Carry flag** and wrapped around to the opposite end (for RLC/RRC), or the behavior involves the Carry flag directly (for RAL/RAR). These do **not** need a second operand — they only operate on A.

### 46) `RLC` — Rotate Left (no carry involved in the rotation itself)
Each bit in A shifts one position **left**. The **MSB (bit 7)** is moved into **both** the Carry flag **and** bit 0 (it wraps around).
```
CY ← A7,  A(n+1) ← An,  A0 ← A7
```

### 47) `RRC` — Rotate Right
Each bit in A shifts one position **right**. The **LSB (bit 0)** is moved into **both** the Carry flag **and** bit 7 (it wraps around).
```
CY ← A0,  An ← A(n+1),  A7 ← A0
```

### 48) `RAL` — Rotate Accumulator Left through Carry
Each bit shifts left, but instead of wrapping around directly, the **old Carry flag** value moves into bit 0, and bit 7 moves into the Carry flag.
```
CY ← A7,  A(n+1) ← An,  A0 ← old CY
```
This effectively rotates through a 9-bit loop (8 bits of A + the Carry flag).

### 49) `RAR` — Rotate Accumulator Right through Carry
Same idea as RAL but to the right: bit 0 moves into Carry, and the **old Carry flag** value moves into bit 7.
```
CY ← A0,  An ← A(n+1),  A7 ← old CY
```

**RLC/RRC vs RAL/RAR — the key difference:**
- RLC/RRC: the bit that falls off wraps directly back into A (Carry is just a copy/spectator).
- RAL/RAR: the bit that falls off goes into Carry, and the *previous* Carry value is what enters A. This links rotations across multiple instructions/bytes.

---

## 6. Compare Instruction

### 50) `CMP B`
**Operation:** A − B (result discarded, only flags are set)  
Compares the Accumulator with register B by internally subtracting B from A, **without changing A**. Only the flags are updated:
- If A = B → **Zero flag = 1**
- If A < B → **Carry flag = 1**
- If A > B → **Carry flag = 0**, **Zero flag = 0**

### 51) `CPI 25H`
**Operation:** A − 25H (result discarded, only flags are set)  
Same comparison, but against the immediate value 25H instead of a register. "CPI" = **Compare Immediate**. Very commonly used before conditional jump instructions (like `JZ`, `JC`) to make decisions.

---

## 7. Quick Reference Table

| # | Mnemonic | Operation | Purpose |
|---|----------|-----------|---------|
| 37 | ANA B | A ← A AND B | Logical AND (register) |
| 38 | ANA M | A ← A AND [HL] | Logical AND (memory) |
| 39 | ANI 25H | A ← A AND 25H | Logical AND (immediate) |
| 40 | ORA B | A ← A OR B | Logical OR (register) |
| 41 | ORA M | A ← A OR [HL] | Logical OR (memory) |
| 42 | ORI 25H | A ← A OR 25H | Logical OR (immediate) |
| 43 | XRA B | A ← A XOR B | Logical XOR (register) |
| 44 | XRA M | A ← A XOR [HL] | Logical XOR (memory) |
| 45 | XRI 25H | A ← A XOR 25H | Logical XOR (immediate) |
| 46 | RLC | Rotate A left, MSB → CY & bit 0 | Rotate left |
| 47 | RRC | Rotate A right, LSB → CY & bit 7 | Rotate right |
| 48 | RAL | Rotate A left through Carry | Rotate left through carry |
| 49 | RAR | Rotate A right through Carry | Rotate right through carry |
| 50 | CMP B | A − B (flags only) | Compare with register |
| 51 | CPI 25H | A − 25H (flags only) | Compare with immediate |

---

## 8. Key Takeaways
- **ANA / ORA / XRA** (and their `I` immediate, `M` memory variants) do bitwise logic on the Accumulator — AND masks bits off, OR sets bits, XOR toggles bits (and clears A when XORed with itself).
- **RLC/RRC** rotate bits within A and wrap the outgoing bit back in, also copying it to Carry.
- **RAL/RAR** rotate through the Carry flag, chaining rotations across the Carry bit — useful for multi-byte shifts.
- **CMP/CPI** compare values by subtracting internally, but only update flags — the Accumulator itself is untouched. This is the standard way to set up conditional branching in 8085 programs.
