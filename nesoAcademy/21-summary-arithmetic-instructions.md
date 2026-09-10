# 21 - Summary of Arithmetic Instructions (8085 Microprocessor)

> A consolidated summary of **all 14 instruction types** in the **Arithmetic Group of Instructions**, covered across the previous sessions (including `ADD R`, `ADI`, `INR R`, `ADC R`, `ACI`, `SUB R`, `SUI`, `DCR R`, `SBB R`, `SBI`, `INX RP`, `DCX RP`, `DAD RP`, and `DAA`).
> This session recaps each instruction type and tallies up the total **opcode count = 62**.

---

## 1. Purpose of This Session

- All 14 arithmetic instruction types have already been studied individually in earlier sessions.
- This session is a **review/summary**, going through each type briefly and **counting the total number of opcodes** across the entire arithmetic group.

---

## 2. The 14 Instruction Types (In Order)

### Type 1: `ADD R`
- **Meaning:** Add contents of `R` to accumulator.
- **Rule:** In 8085, one operand of any 8-bit addition **must** be in the accumulator. After addition, the result is also stored back in the accumulator.
- The second operand comes from **register or memory location specified by capital `R`**.
- **`R` (capital)** represents: accumulator, all 6 GPRs (B, C, D, E, H, L), and **M** (memory location pointed to by the HL register pair).
- **Opcodes: 8**

### Type 2: `ADI` (Add Immediate)
- Performs the same operation as `ADD R`, but the 8-bit data is provided **directly within the instruction** (immediate addressing mode) instead of via a register.
- **Opcodes: 1**

### Type 3: `INR R` (Increment)
- Increments the content of `R` (accumulator, GPRs, or memory location `M` pointed by HL) by **1**.
- **Opcodes: 8**

### Type 4: `ADC R` (Add with Carry)
- **Meaning:** Add with carry — contents of `R` to accumulator.
- **Why needed:** The 8085 is an 8-bit microprocessor, so it cannot add numbers larger than 8 bits (e.g., 16-bit numbers) in a single step. Multi-byte addition must be done in parts:
  1. Add the lower 8 bits first.
  2. If a carry is generated, it must be accounted for when adding the upper 8 bits — this is where `ADC` comes in.
- **Example walkthrough:**
  - `45` loaded into accumulator
  - `33` (addend) loaded into register B
  - Carry was already **set** from a previous operation
  - Instruction used: `ADC B` → adds B **+ carry** to accumulator
  - Final result: **79** stored in accumulator
- **Opcodes: 8** (accumulator, GPRs, and memory location M)

### Type 5: `ACI` (Add with Carry Immediate)
- Same as `ADC R`, but the addend is provided directly in the instruction (immediate addressing mode).
- **Opcodes: 1**

> ✅ **Types 1–5 complete the "addition" instructions.**

---

### Type 6: `SUB R` (Subtract)
- Subtracts the content of `R` (accumulator, GPRs, or memory location M) **from** the accumulator.
- **Opcodes: 8**

### Type 7: `SUI` (Subtract Immediate)
- Subtracts an 8-bit value, provided directly in the instruction, from the accumulator.
- **Opcodes: 1**

### Type 8: `DCR R` (Decrement)
- Opposite of `INR R` — decrements the content of `R` by 1.
- **Opcodes: 8**

### Type 9: `SBB R` (Subtract with Borrow)
- **Meaning:** Subtract with borrow — contents of `R` from accumulator.
- **Why needed:** Similar reasoning to `ADC` — for multi-byte subtraction, the 8085 can't process more than 8 bits at once. The subtraction is done in two parts:
  1. Subtract the lower byte first.
  2. If a **borrow** occurs (indicated by the carry flag being set), that borrow must be accounted for in the next part using `SBB`.
- **Opcodes: 8** (accumulator, GPRs, and memory location M)

### Type 10: `SBI` (Subtract with Borrow Immediate)
- Same as `SBB R`, but the subtrahend is provided directly within the instruction (immediate addressing mode).
- **Opcodes: 1**

> ✅ **Types 6–10 complete the "subtraction" instructions.**

---

### Type 11: `INX RP` (Increment Register Pair / "Extended Register")
- Increments the content of an **extended register (register pair)** by 1.
- **Note on naming:** Only the *first* register of the pair is mentioned in the instruction (e.g., `B` implies the pair `BC`, since `C` is treated as an extension of `B`). Same logic applies to `D`→`DE` and `H`→`HL`.
- **Opcodes: 3** (for register pairs: BC, DE, HL)

### Type 12: `DCX RP` (Decrement Register Pair)
- Opposite of `INX RP` — decrements the content of the register pair by 1.
- **Opcodes: 3**

### Type 13: `DAD RP` (Double Add Register Pair)
- **Meaning:** Double Add — adds the contents of a register pair (`RP`) with the **HL register pair**.
- **Why "double":** Using the accumulator alone, only 8-bit ("single") addition is possible. `DAD` adds **16 bits at once** by combining two register pairs — hence "double add."
- Note: HL is the implied destination/second operand, even though it isn't written explicitly in the mnemonic.
- **Opcodes: 3** (for register pairs: BC, DE, HL — added to HL)

### Type 14: `DAA` (Decimal Adjust Accumulator)
- Used **after a BCD addition** has been performed, to adjust/correct the result stored in the accumulator into valid BCD form.
- Based on whether correction is needed at the least significant digit, most significant digit, both, or neither, `DAA` adds one of:
  - **`00`** → no correction needed (both digits already valid BCD)
  - **`06`** → correction needed only at the **least significant digit** (it was invalid BCD)
  - **`60`** → correction needed only at the **most significant digit** (it was invalid BCD, indicated also by the carry flag being set)
  - **`66`** → correction needed at **both digits** (both were invalid BCD)
- Recall: A **valid BCD digit** is any value **≤ 9**.
- **Opcodes: 1**

> ✅ **Types 11–14 cover register-pair arithmetic and the special decimal-adjust instruction.**

---

## 3. Opcode Count — Running Tally

| # | Instruction Type | Opcodes | Running Total |
|:---:|---|:---:|:---:|
| 1 | ADD R | 8 | 8 |
| 2 | ADI | 1 | 9 |
| 3 | INR R | 8 | 17 |
| 4 | ADC R | 8 | 25 |
| 5 | ACI | 1 | 26 |
| 6 | SUB R | 8 | 34 |
| 7 | SUI | 1 | 35 |
| 8 | DCR R | 8 | 43 |
| 9 | SBB R | 8 | 51 |
| 10 | SBI | 1 | 52 |
| 11 | INX RP | 3 | 55 |
| 12 | DCX RP | 3 | 58 |
| 13 | DAD RP | 3 | 61 |
| 14 | DAA | 1 | **62** |

**Total: 14 instruction types → 62 opcodes**

---

## 4. Quick Reference Table

| Instruction Type | Full Meaning | Addressing Mode | Opcodes |
|---|---|---|:---:|
| ADD R | Add register/memory to accumulator | Register/Memory | 8 |
| ADI | Add immediate | Immediate | 1 |
| INR R | Increment register/memory/accumulator | Register/Memory | 8 |
| ADC R | Add with carry | Register/Memory | 8 |
| ACI | Add with carry, immediate | Immediate | 1 |
| SUB R | Subtract register/memory | Register/Memory | 8 |
| SUI | Subtract immediate | Immediate | 1 |
| DCR R | Decrement register/memory/accumulator | Register/Memory | 8 |
| SBB R | Subtract with borrow | Register/Memory | 8 |
| SBI | Subtract with borrow, immediate | Immediate | 1 |
| INX RP | Increment register pair | Implied | 3 |
| DCX RP | Decrement register pair | Implied | 3 |
| DAD RP | Double add (register pair + HL) | Implied | 3 |
| DAA | Decimal adjust accumulator | Implied | 1 |

---

## 5. Key Points to Remember

1. The **Arithmetic Group** has **14 instruction types** and **62 total opcodes**.
2. Instructions using **capital `R`** always refer to: Accumulator + 6 GPRs (B, C, D, E, H, L) + memory location `M` (pointed by HL pair) → typically **8 opcodes**.
3. Instructions ending in **immediate addressing mode** (`ADI`, `ACI`, `SUI`, `SBI`) always have just **1 opcode**, since the data comes from the instruction itself, not a register.
4. Instructions on **register pairs** (`INX RP`, `DCX RP`, `DAD RP`) apply to only **3 pairs** (BC, DE, HL) → **3 opcodes** each.
5. `ADC`/`SBB` exist specifically to handle **multi-byte arithmetic**, propagating carry/borrow between the lower and upper bytes.
6. `DAA` is unique — it has only **1 opcode** and works based on the **accumulator content + Auxiliary Carry flag + Carry flag**, applying one of four corrections: `00`, `06`, `60`, or `66`.

---

## 6. What's Next

- Next session: **Solving practice problems** based on everything learned in the Arithmetic Group of Instructions.

---
*Notes prepared from video lecture transcript — "Summary of Arithmetic Instructions"*
