# 8085 Microprocessor — Arithmetic Group Instructions

This document explains the **Arithmetic Group** of instructions in the Intel 8085 microprocessor instruction set. Before diving into the instructions, a few core concepts are explained so the instructions make full sense.

---

## 1. Background Concepts

### 1.1 Registers Used
- **Accumulator (A)** — The primary 8-bit register used for all arithmetic and logic operations. Every arithmetic instruction either uses A as an operand or stores its result in A.
- **General purpose registers** — B, C, D, E, H, L (each 8-bit). These can hold data temporarily.
- **Register pair M** — This is **not a real register**. "M" refers to the memory location whose address is stored in the **HL register pair**. So `ADD M` means "add the contents of the memory location pointed to by HL."
- **Flags affected** — Arithmetic instructions typically affect the **Carry (CY)**, **Sign (S)**, **Zero (Z)**, **Auxiliary Carry (AC)**, and **Parity (P)** flags.

### 1.2 Types of Operands
Arithmetic instructions can take their second operand from three places:
| Operand type | Example | Meaning |
|---|---|---|
| Register | `ADD B` | Uses the value in register B |
| Memory (via HL) | `ADD M` | Uses the value at the memory address in HL |
| Immediate data (8-bit) | `ADI 25H` | Uses a constant value written directly in the instruction |

### 1.3 Carry vs No Carry
Many instructions come in pairs — one that includes the **Carry flag** in the operation, and one that doesn't:
- **ADD** = Add without carry
- **ADC** = Add **with** carry (adds the previous carry flag value too)
- **SUB** = Subtract without borrow
- **SBB** = Subtract **with** borrow (subtracts the previous borrow/carry flag value too)

This is useful for multi-byte (multi-precision) arithmetic, where you need to carry over the result from a lower byte into a higher byte.

---

## 2. Addition Instructions

### 17) `ADD B`
**Operation:** A ← A + B  
Adds the contents of register B to the Accumulator. Result is stored in A. Flags are updated.

### 18) `ADD M`
**Operation:** A ← A + [HL]  
Adds the contents of the memory location pointed to by HL to the Accumulator.

### 19) `ADI 25H`
**Operation:** A ← A + 25H  
Adds the 8-bit immediate value (here, 25H) directly to the Accumulator. "ADI" = **Add Immediate**.

### 23) `ADC B`
**Operation:** A ← A + B + CY  
Adds register B **and** the current Carry flag value to A. Used when adding the lower byte already produced a carry that needs to be included in this addition.

### 24) `ADC M`
**Operation:** A ← A + [HL] + CY  
Same as above, but the second operand comes from the memory location pointed to by HL.

### 25) `ACI 25H`
**Operation:** A ← A + 25H + CY  
Adds the immediate value **and** the Carry flag to A. "ACI" = **Add Immediate with Carry**.

---

## 3. Subtraction Instructions

### 20) `SUB B`
**Operation:** A ← A − B  
Subtracts the contents of register B from the Accumulator.

### 21) `SUB M`
**Operation:** A ← A − [HL]  
Subtracts the contents of the memory location pointed to by HL from A.

### 22) `SUI 25H`
**Operation:** A ← A − 25H  
Subtracts the immediate value from A. "SUI" = **Subtract Immediate**.

### 26) `SBB B`
**Operation:** A ← A − B − CY (borrow)  
Subtracts register B **and** the existing borrow (Carry flag) from A. Used in multi-byte subtraction where a borrow was generated from a previous (lower-byte) subtraction.

### 27) `SBB M`
**Operation:** A ← A − [HL] − CY  
Same as above, but subtracts the value at the memory location pointed to by HL, along with the borrow.

### 28) `SBI 25H`
**Operation:** A ← A − 25H − CY  
Subtracts the immediate value and the borrow from A. "SBI" = **Subtract Immediate with Borrow**.

---

## 4. Increment / Decrement Instructions

These instructions change a value by exactly 1. **Important:** unlike ADD/SUB, these do **not** affect the Carry flag (but they do affect Sign, Zero, Auxiliary Carry, and Parity).

### 29) `INR B`
**Operation:** B ← B + 1  
Increments (adds 1 to) the contents of register B.

### 30) `DCR B`
**Operation:** B ← B − 1  
Decrements (subtracts 1 from) the contents of register B.

### 31) `INR M`
**Operation:** [HL] ← [HL] + 1  
Increments the contents of the memory location pointed to by HL.

### 32) `DCR M`
**Operation:** [HL] ← [HL] − 1  
Decrements the contents of the memory location pointed to by HL.

### 33) `INX B`
**Operation:** BC ← BC + 1  
Increments the entire **register pair** BC (16-bit increment), not just one register. Used for pointer/counter operations. Does **not** affect any flags.

### 34) `DCX B`
**Operation:** BC ← BC − 1  
Decrements the register pair BC (16-bit decrement). Does not affect flags.

---

## 5. 16-bit Addition

### 35) `DAD D`
**Operation:** HL ← HL + DE  
"DAD" = **Double Add**. Adds the 16-bit value in the DE register pair to the 16-bit value in HL, storing the result back in HL. Only the **Carry flag** is affected (set if there's a carry out of the 16-bit addition); other flags are unaffected. Commonly used for address/pointer arithmetic.

---

## 6. Decimal Adjust

### 36) `DAA`
**"Decimal Adjust Accumulator."**  
After performing binary addition on two **BCD (Binary Coded Decimal)** numbers, the result in the Accumulator may not be a valid BCD digit (since BCD only uses 0–9 per nibble, but binary addition can produce 0–15). `DAA` corrects/adjusts the value in A back into valid packed BCD format based on the AC and CY flags. It's typically used right after an `ADD`/`ADC` instruction when working with BCD numbers.

---

## 7. Quick Reference Table

| # | Mnemonic | Operation | Type |
|---|----------|-----------|------|
| 17 | ADD B | A ← A + B | Register add |
| 18 | ADD M | A ← A + [HL] | Memory add |
| 19 | ADI 25H | A ← A + 25H | Immediate add |
| 20 | SUB B | A ← A − B | Register subtract |
| 21 | SUB M | A ← A − [HL] | Memory subtract |
| 22 | SUI 25H | A ← A − 25H | Immediate subtract |
| 23 | ADC B | A ← A + B + CY | Add with carry |
| 24 | ADC M | A ← A + [HL] + CY | Add with carry |
| 25 | ACI 25H | A ← A + 25H + CY | Add immediate with carry |
| 26 | SBB B | A ← A − B − CY | Subtract with borrow |
| 27 | SBB M | A ← A − [HL] − CY | Subtract with borrow |
| 28 | SBI 25H | A ← A − 25H − CY | Subtract immediate with borrow |
| 29 | INR B | B ← B + 1 | Register increment |
| 30 | DCR B | B ← B − 1 | Register decrement |
| 31 | INR M | [HL] ← [HL] + 1 | Memory increment |
| 32 | DCR M | [HL] ← [HL] − 1 | Memory decrement |
| 33 | INX B | BC ← BC + 1 | Register pair increment (16-bit) |
| 34 | DCX B | BC ← BC − 1 | Register pair decrement (16-bit) |
| 35 | DAD D | HL ← HL + DE | 16-bit add |
| 36 | DAA | Adjust A to valid BCD | Decimal adjust |

---

## 8. Key Takeaways
- **ADD/SUB** = simple 8-bit add/subtract, no carry/borrow involved.
- **ADC/SBB** = same, but factor in the previous Carry/Borrow flag — essential for multi-byte arithmetic.
- **INR/DCR** = ±1 on a register or memory location; does not touch Carry flag.
- **INX/DCX** = ±1 on a full 16-bit register pair; touches no flags at all.
- **DAD** = 16-bit addition between register pairs, result in HL.
- **DAA** = fixes up binary addition results so they remain valid BCD digits.
